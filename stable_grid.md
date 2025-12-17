# Refactoring Guide: Stable ipyaggrid Tables in Solara Chat Application

## Overview

This guide provides instructions for refactoring the current DataGrid implementation to eliminate flickering and maintain stable column state when:
- New messages arrive in the chat
- The expand button is clicked to open the grid in a dialog

**Current Architecture:**
```
ChatBot Component (manages messages list, uses thread)
    └── render_message() (called for each message)
            └── MemoizedComponent (in datagrid_display.py)
                    └── setupDataGrid
```

**Target Outcome:** Minimal changes to existing structure while fixing stability issues.

---

## Key Solara Concepts (Required Reading)

Before implementing, understand these documented Solara behaviors:

### 1. ipyaggrid Does Not Auto-Update via Reacton

From the official Solara ipyaggrid example (https://py.cafe/maartenbreddels/ipyaggrid-solara):

> `grid_data` and `grid_options` are not traits, so letting them update by reacton/solara has no effect. Instead, we need to get a reference to the widget and call `.update_grid_data` in a `use_effect`.

**Implication:** You cannot simply pass new props to ipyaggrid and expect updates. You must use `solara.get_widget()` inside `use_effect` to imperatively update the widget.

### 2. use_effect Executes After Render

From Solara docs (https://solara.dev/documentation/api/hooks/use_effect):

> `use_effect` is executed after page render, letting us fetch the actual underlying widget object using `solara.get_widget` on an element. `dependencies` should be a list of variables, which when changed trigger re-execution of effect. If left empty, effect will never re-execute.

**Implication:** 
- `dependencies=[]` → Effect runs once on mount only
- `dependencies=[some_var]` → Effect re-runs when `some_var` changes
- `dependencies=None` → Effect runs on every render (avoid this)

### 3. use_memo for Expensive Computations

From Reacton API docs (https://reacton.solara.dev/en/latest/api/):

> Pass an empty list (or any fixed value that supports comparison) to only execute the function once for the lifetime of the component.

**Implication:** Use `solara.use_memo(fn, dependencies=[])` to compute grid options once per component instance.

### 4. The .key() Method Forces Widget Recreation

From Solara Discord discussion:

> `.key(f"theme-{theme}")` forces the recreation of the Widget, so any state of the widget not captured by a state outside the widget might get lost.

**Implication:** Do NOT use `.key()` unless you explicitly want to destroy and recreate the widget. This will cause flickering.

---

## What NOT To Do

| Anti-Pattern | Why It Causes Problems |
|--------------|------------------------|
| `dependencies=[id(df)]` | `id()` changes on every render even if DataFrame content is identical |
| `dependencies=[df]` | DataFrame objects don't implement stable equality comparison |
| Creating grid inside render without memoization | New widget instance created on every parent re-render |
| Using `.key()` on the grid element | Forces widget recreation, losing column state |
| Rendering same grid element in two places simultaneously | Widget can only exist in one DOM location |
| Using `use_memo` with `dependencies=None` | Auto-detection can be unreliable with complex objects |

---

## Refactoring Instructions

### Step 1: Update MemoizedComponent in `datagrid_display.py`

Replace the current implementation with the following pattern:

```python
import ipyaggrid
import solara
from typing import cast, List, Optional

@solara.component
def MemoizedDataGrid(
    message_id: str,  # REQUIRED: unique identifier for this message
    df,               # pandas DataFrame
    display_height: str = "400px",
    date_columns: Optional[List[str]] = None,
    hidden_columns: Optional[List[str]] = None,
):
    """
    Stable ipyaggrid component that prevents flickering on parent re-renders.
    
    Key principles applied:
    1. Grid element created once via use_memo with empty dependencies
    2. Widget updates done imperatively via use_effect + get_widget
    3. Height changes trigger use_effect, not widget recreation
    """
    
    # State for expand dialog
    expanded = solara.use_reactive(False)
    
    # Compute grid options ONCE per component lifetime
    # Docs: https://solara.dev/documentation/api/hooks/use_memo
    grid_options = solara.use_memo(
        lambda: _build_grid_options(df, date_columns, hidden_columns),
        dependencies=[]  # Empty list = compute once, never recompute
    )
    
    # Determine current height based on expanded state
    current_height = "80vh" if expanded.value else display_height
    
    # Create the element ONCE
    # This returns an Element (virtual), not a Widget (real)
    # Docs: https://solara.dev/documentation/advanced/howto/ipywidget-libraries
    el = solara.use_memo(
        lambda: ipyaggrid.Grid.element(
            grid_data=df,
            grid_options=grid_options,
            license="community",
        ),
        dependencies=[]  # Create element once per component lifetime
    )
    
    def update_grid():
        """
        Imperatively update the widget after render.
        
        This is REQUIRED for ipyaggrid because grid_data and grid_options
        are not reactive traits - Reacton/Solara cannot auto-update them.
        
        Docs: https://py.cafe/maartenbreddels/ipyaggrid-solara
        """
        widget = cast(ipyaggrid.Grid, solara.get_widget(el))
        
        # Update height (changes when expanded state changes)
        widget.height = current_height
        
        # Auto-size columns after a short delay to ensure DOM is ready
        # This fixes the "columns go very small" issue
        widget.js_post_grid = """
            setTimeout(function() {
                if (gridOptions.api) {
                    gridOptions.api.sizeColumnsToFit();
                }
            }, 150);
        """
    
    # Run update_grid when expanded state changes
    # Docs: https://solara.dev/documentation/api/hooks/use_effect
    solara.use_effect(update_grid, dependencies=[expanded.value])
    
    # RENDER UI
    with solara.Column():
        # Expand/collapse button
        with solara.Row(justify="end"):
            solara.Button(
                icon_name="mdi-arrow-expand-all" if not expanded.value else "mdi-arrow-collapse-all",
                on_click=lambda: expanded.set(not expanded.value),
                icon=True,
                text=True,
            )
        
        # Inline grid display (when NOT expanded)
        if not expanded.value:
            solara.display(el)
        
        # Dialog display (when expanded)
        # Note: Dialog renders the same element, which moves it to the dialog
        with solara.lab.ConfirmationDialog(
            open=expanded.value,
            on_close=lambda: expanded.set(False),
            ok=None,
            cancel="Close",
            title="Data View",
            max_width="95vw",
            persistent=False,
        ):
            if expanded.value:
                solara.display(el)


def _build_grid_options(df, date_columns: Optional[List[str]], hidden_columns: Optional[List[str]]) -> dict:
    """
    Build ag-grid column definitions from DataFrame.
    Called once per component via use_memo.
    """
    date_columns = date_columns or []
    hidden_columns = hidden_columns or []
    
    column_defs = []
    for col in df.columns:
        col_def = {
            "field": col,
            "filter": True,
            "sortable": True,
            "resizable": True,
            "minWidth": 100,  # Prevents columns from collapsing too small
        }
        if col in hidden_columns:
            col_def["hide"] = True
        if col in date_columns:
            col_def["filter"] = "agDateColumnFilter"
        column_defs.append(col_def)
    
    return {
        "columnDefs": column_defs,
        "defaultColDef": {
            "sortable": True,
            "filter": True,
            "resizable": True,
            "minWidth": 100,
        },
        "animateRows": True,
    }
```

### Step 2: Update render_message() to Pass message_id

Ensure every message has a stable unique identifier:

```python
import uuid

def render_message(message: dict):
    """
    Render a single chat message.
    
    CRITICAL: Each message MUST have a stable 'id' field.
    This id must NOT change between renders.
    """
    # Ensure message has stable ID
    # If messages come from backend, use their ID
    # If generated client-side, generate once when message is created
    message_id = message.get("id")
    if message_id is None:
        raise ValueError("Message must have an 'id' field for stable rendering")
    
    # ... existing message rendering logic ...
    
    if message.get("dataframe") is not None:
        MemoizedDataGrid(
            message_id=message_id,  # Pass the stable ID
            df=message["dataframe"],
            display_height=message.get("display_height", "400px"),
            date_columns=message.get("date_columns"),
            hidden_columns=message.get("hidden_columns"),
        )
```

### Step 3: Update ChatBot Component Message Creation

When creating messages (user or assistant), always assign a stable ID:

```python
import uuid

# When user sends a message:
user_message = {
    "id": str(uuid.uuid4()),  # Generate once, never changes
    "role": "user",
    "content": user_input,
}

# When assistant responds with DataFrame:
assistant_message = {
    "id": str(uuid.uuid4()),  # Generate once, never changes
    "role": "assistant",
    "content": "Here is the data:",
    "dataframe": result_df,
    "date_columns": ["date_field"],
    "hidden_columns": ["internal_id"],
}

# Append to messages list
messages.set([*messages.value, assistant_message])
```

### Step 4: Remove Old Memoization Patterns

Remove any existing code that uses these anti-patterns:

```python
# REMOVE patterns like:
grid = solara.use_memo(
    lambda: setupDataGrid(...),
    dependencies=[id(df), display_height]  # ❌ id(df) is unstable
)

# REMOVE patterns like:
grid = solara.use_memo(
    lambda: setupDataGrid(...),
    dependencies=None  # ❌ Auto-detection unreliable
)

# REMOVE patterns like:
return grid.key(f"grid-{message_id}")  # ❌ Forces recreation
```

---

## Verification Checklist

After refactoring, verify the following:

| Test Case | Expected Behavior |
|-----------|-------------------|
| Send new message | Existing tables remain stable, no flicker |
| Click expand button | Grid moves to dialog smoothly, columns retain size |
| Close dialog | Grid returns inline, columns retain size |
| Multiple tables in chat | Each table is independent, no cross-contamination |
| Resize columns manually | Resized state persists across expand/collapse |
| Pivot table interactions | Pivot state persists across parent re-renders |

---

## Troubleshooting

### Issue: Columns still go small after expand

**Cause:** `sizeColumnsToFit` called before DOM is ready.

**Fix:** Increase timeout in `js_post_grid`:
```python
widget.js_post_grid = """
    setTimeout(function() {
        if (gridOptions.api) {
            gridOptions.api.sizeColumnsToFit();
        }
    }, 300);  // Increase from 150 to 300
"""
```

### Issue: Grid flickers when new message arrives

**Cause:** `dependencies` array contains unstable reference.

**Fix:** Ensure `dependencies=[]` for both `use_memo` calls.

### Issue: Grid doesn't update height when expanding

**Cause:** `expanded.value` not in `use_effect` dependencies.

**Fix:** Verify:
```python
solara.use_effect(update_grid, dependencies=[expanded.value])
```

### Issue: "get_widget returned None" error

**Cause:** Trying to get widget before element is rendered.

**Fix:** `use_effect` automatically runs after render, so this shouldn't happen. If it does, check that `el` is being returned/displayed in the component.

### Issue: Multiple grids share state unexpectedly

**Cause:** Missing or duplicate `message_id`.

**Fix:** Verify each message has a unique, stable `id` field generated at creation time.

---

## Summary of Changes

| File | Change |
|------|--------|
| `datagrid_display.py` | Replace `MemoizedComponent` with new `MemoizedDataGrid` using `use_effect` + `get_widget` pattern |
| `render_message.py` (or equivalent) | Pass `message_id` to `MemoizedDataGrid` |
| ChatBot component | Ensure all messages have stable `id` field assigned at creation |
| Remove | Any use of `id(df)` in dependencies, any `.key()` on grid elements |

---

## Reference Links

- Solara use_effect: https://solara.dev/documentation/api/hooks/use_effect
- Solara use_memo: https://solara.dev/documentation/api/hooks/use_memo
- Official ipyaggrid + Solara pattern: https://py.cafe/maartenbreddels/ipyaggrid-solara
- Solara state management: https://solara.dev/documentation/getting_started/fundamentals/state-management
- ipywidget libraries in Solara: https://solara.dev/documentation/advanced/howto/ipywidget-libraries
