# Overview

- [Abstract widget, and abstract UI data](/ekg-docs/widget/#abstract)

- [Type](/ekg-docs/widget/#type)

- [Level](/ekg-docs/widget/#level)

- [Dimension](/ekg-docs/widget/#dimension)

# Abstract widget, and abstract UI data

EKG has two layers of objects: `ekg::ui::abstract_widget`, and `ekg::ui::abstract`. Both are used together but with different purposes.

`ekg::ui::abstract` objects work as intermediary data between the user-programmer, and the `ekg::ui::abstract_widget`, which is used to process the logic and render.

This philosophy allows more control, and memory security for the application. While backing service tools handle the `ekg::ui::abstract_widget`, the user can only access `ekg::ui::abstract`.

# Type

Type represents different widgets, and helps check the instance of the current context widget object class.

| Type                  | Widget                          |
| --------------------- | ------------------------------- |
| `ekg::type::abstract` | [Abstract](/ekg-docs/abstract/) |
| `ekg::type::frame`    | [Frame](/ekg-docs/frame/)       |
| `ekg::type::label`    | [Label](/ekg-docs/label/)       |
| `ekg::type::slider`   | [Slider](/ekg-docs/slider/)     |
| `ekg::type::slider2d` | [Slider2D](/ekg-docs/slider2d/) |
| `ekg::type::checkbox` | [Checkbox](/ekg-docs/checkbox/) |
| `ekg::type::textbox`  | [Textbox](/ekg-docs/textbox/)   |
| `ekg::type::combobox` | [Combobox](/ekg-docs/combobox/) |
| `ekg::type::listbox`  | [Listbox](/ekg-docs/listbox/)   |
| `ekg::type::tab`      | [Tab](/ekg-docs/tab/)           |
| `ekg::type::popup`    | [Popup](/ekg-docs/popup/)       |
| `ekg::type::scroll`   | [Scroll](/ekg-docs/scroll/)     |

# Level

A level-constant `ekg::level::bottom_level`, and `ekg::level::top_level`defines how the widget should be processed and placed. Most of the widgets except popup(s), by default sets `ekg::level::bottom_level`.

Normal placement(s) `ekg::level::bottom_level` is followed by [layout service](/ekg-docs/layout/), and can not interrupt the current processing widgets IO (input, events calling) context.

Regular placement(s) `ekg::level::top_level` is followed by a private invocation of [layout service](/ekg-docs/layout/) features, and can interrupt the current processing widgets IO (input, events calling) context. Top-level widgets can not be a child, this rule allows separating IO events processing between top-level and bottom-level widgets.

# Dimension

The dimension of a [widget](/ekg-docs/widget/) contains differences between the horizontal and vertical axes. The width is based on pixels, and the height is based on a factor. The height factor is a multiple of [font-size](/ekg-docs/font/#sizes).

If a [widget](/ekg-docs/widget/) height factor is 2, and the [widget](/ekg-docs/widget/) [font](/e) setting is `ekg::font::normal`, then the size is `2*font_size`.
