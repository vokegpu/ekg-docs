# Overview

- Abstract widget, and abstract UI data

- Type

- Level

# Abstract widget, and abstract UI data

EKG has two layers of objects: `ekg::ui::abstract_widget`, and `ekg::ui::abstract`. Both are used together but with different purposes.

`ekg::ui::abstract` objects work as intermediary data between the user-programmer, and the `ekg::ui::abstract_widget`, which is used for processing the logic, and rendering.

This philosophy allows more control, and memory security for the application. While backing service tools handle the `ekg::ui::abstract_widget`, the user only has access to `ekg::ui::abstract`.

# Type

Type represents different widgets, and helps check the instance of the current context widget object class, `ekg::type` contains the following enum(s) constant:

- `ekg::type::abstract` [refers to the abstract UI element](Abstract)

- `ekg::type::frame` [refers to the frame UI element](Frame)

- `ekg::type::button` [refers to the button UI element](Button) 

- `ekg::type::label` [refers to the label UI element](Label)

- `ekg::type::slider` [refers to the slider UI element](Slider)

- `ekg::type::slider2d` [refers to the slider 2D UI element](Slider2d) 

- `ekg::type::checkbox` [refers to the frame UI element](Checkbox) 

- `ekg::type::textbox` [refers to the textbox UI element](Textbox)

- `ekg::type::combobox` [refers to the combobox UI element](Combobox) 

- `ekg::type::listbox` [refers to the listbox UI element](Listbox) 

- `ekg::type::tab` [refers to the tab UI element](Tab)

- `ekg::type::popup` [refers to the popup UI element](Popup)

- `ekg::type::scroll` [refers to the scroll UI element](Scroll)

# Level

A level-constant `ekg::level::bottom_level`, and `ekg::level::top_level`defines how the widget should be processed and placed. Most of the widgets except popup(s), by default sets `ekg::level::bottom_level`.

Normal placement(s) `ekg::level::bottom_level` is followed by [layout service](Layout), and can not interrupt the current processing widgets IO (input, events calling) context.

Regular placement(s) `ekg::level::top_level` is followed by a private invocation of [layout service](Layout) features, and has the ability to interrupt the current processing widgets IO (input, events calling) context. Top-level widgets can not be a child, this rule allows separating IO events processing between top-level and bottom-level widgets.
