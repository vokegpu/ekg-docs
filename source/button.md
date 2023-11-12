# Overview

- [Fundamentals](/ekg-docs/button/#fundamentals)

- [Examples](/ekg-docs/button/#examples)

- [Methods](/ekg-docs/button/#methods)

# Fundamentals

The [button](/ekg-docs/button/) [widget](/ekg-docs/widget/) works fundamentally simply, you have a text, and the alignment of text. Each button has it is own optional `ekg::cpu::event` callback event [task](/ekg-docs/handler-service/#task), where one lambda function is optional.

The [button](/ekg-docs/button/) [widget](/ekg-docs/widget/) contains two string sets `set_text` and `set_tag`, `set_tag` is reserved for user purposes, while `set_text` is the visible text of the [widget](/ekg-docs/widget/) [button](/ekg-docs/button/).

# Examples

For creating a simple [button](/ekg-docs/button/) with no callback event.

```cpp
auto p_button = ekg::button("cat", ekg::dock::fill | ekg::next);
```

The first parameter is the visible text of a [button](/ekg-docs/button/). The second parameter is the docking alignment in the frame widget.

To create a callback button, with [task](/ekg-docs/handler-service/#task).

```cpp
p_button->set_callback(new ekg::cpu::event("cat", nullptr, [](void *p_callback) {
    ekg::log() << "Cat!";
}));
```

This callback event is not deleted after execution, because it is batched, which means that all clicks will execute this [task](/ekg-docs/handler-service/#task).

# Methods

### Button

Set the [font-size](/ekg-docs/font/#sizes) of [button](/ekg-docs/button/) text.

```cpp
ekg::ui::button *set_font_size(ekg::font font);
```

Get the [font-size](/ekg-docs/font/#sizes) of [button](/ekg-docs/button/) text.

```cpp
ekg::font get_font_size();
```

Set [layout dock](/ekg-docs/layout/#dock) position.

```cpp
ekg::ui::button *set_place(uint16_t dock);
```

Set dimension width in pixels, see details [here](/ekg-docs/widget/#dimension).

```cpp
ekg::ui::button *set_width(float w);
```

Get dimension width in pixels, see details [here](/ekg-docs/widget/#dimension).

```cpp
float get_width();
```

Set dimension height in scale factor, see details [here](/ekg-docs/widget/#dimension).

```cpp
ekg::ui::button *set_scaled_height(int32_t h);
```

Get dimension height in scale factor, see details [here](/ekg-docs/widget/#dimension).

```cpp
int32_t get_scaled_height();
```

Get dimension height in pixels.

```cpp
float get_height();
```

Set callback event [task](/ekg-docs/handler-service/#task).

```cpp
ekg::ui::button *set_callback(ekg::cpu::event *p_callback);
```

Get callback event [task](/ekg-docs/handler-service/#task).

```cpp
ekg::cpu::event *get_callback();
```

Set display text.

```cpp
ekg::ui::button *set_text(std::string_view text);
```

Get display text.

```cpp
std::string_view get_text();
```

Set [button](/ekg-docs/button/) state pressed/callback.

```cpp
ekg::ui::button *set_value(bool state);
```

Get [button](/ekg-docs/button/) state.

```cpp
bool get_value();
```

Set text [alignment dock](/ekg-docs/layout/#mask alignment).

```cpp
ekg::ui::button *set_text_align(uint16_t dock);
```

Get text [alignment dock](/ekg-docs/layout/#mask alignment).

```cpp
uint16_t get_text_align();
```

### Abstract

Set tag (reserved purposes).

```cpp
ekg::ui::abstract *set_tag(std::string_view tag);
```

Get the tag (reserved purposes).

```cpp
std::string_view get_tag();
```

Set the [widget](/ekg-docs/widget/) ID, see details [here](/ekg-docs/widget/#family).

```cpp
ekg::ui::abstract *set_id(int32_t id);
```

Get the [widget](/ekg-docs/widget/) ID, see details [here](/ekg-docs/widget/#family).

```cpp
int32_t get_id();
```

Set parent mother [widget](/ekg-docs/widget/) ID, see details [here](/ekg-docs/widget/#family).

```cpp
ekg::ui::abstract *set_parent_id(int32_t parent_id);
```

Get parent mother [widget](/ekg-docs/widget/) ID, see details [here](/ekg-docs/widget/#family).

```cpp
int32_t get_parent_id();
```

Add a child to the [widget](/ekg-docs/widget/), see details [here](/ekg-docs/widget/#family).

```cpp
ekg::ui::abstract *add_child(int32_t id);
```

Get the [widget](/ekg-docs/widget/) child id list, see details [here](/ekg-docs/widget/#family).

```cpp
std::vector<int32_t> &get_child_id_list();
```

Remove a child [widget](/ekg-docs/widget/) from the mother [widget](/ekg-docs/widget/#family), see details [here](/ekg-docs/widget/#family).

```cpp
ekg::ui::abstract *remove_child(int32_t id);
```

Check if the [widget](/ekg-docs/widget/) has a parent mother [widget](/ekg-docs/widget/), see details [here](/ekg-docs/widget/#family).

```cpp
bool has_parent();
```

Check if the [widget](/ekg-docs/widget/) has children, see details [here](/ekg-docs/widget/#family).

```cpp
bool has_children();
```

Set [widget](/ekg-docs/widget/) alive state.

```cpp
ekg::ui::abstract *set_alive(bool state);
```

Get [widget](/ekg-docs/widget/) alive state.

```cpp
bool is_alive();
```

Destroy the [widget](/ekg-docs/widget/#family).

```cpp
void destroy();
```

Set the [widget state](/ekg-docs/widget/#state).

```cpp
ekg::ui::abstract *set_state(const ekg::state &_state);
```

Get the [widget state](/ekg-docs/widget/#state).

```cpp
ekg::state get_state();
```

Set the [widget type](/ekg-docs/widget/#type).

```cpp
ekg::ui::abstract *set_type(const ekg::type &_type);
```

Get the [widget type](/ekg-docs/widget/#type).

```cpp
ekg::type get_type();
```

Set the [widget level](/ekg-docs/widget/#level).

```cpp
ekg::ui::abstract *set_level(const ekg::type &_level);
```

Get the [widget level](/ekg-docs/widget/#level).

```cpp
ekg::type get_level();
```

Get [layout dock](/ekg-docs/layout/#dock) position.

```cpp
uint16_t get_place_dock();
```

Get the sync flags reference.

```cpp
uint16_t &get_sync();
```

Reset the UI data front-end with the [widget](/ekg-docs/widget/) back-end.

```cpp
void reset();
```

Access to absolute [widget](/ekg-docs/widget/) back-end rectangle.

```cpp
ekg::rect &widget();
```

Get the UI data front-end rectangle.

```cpp
ekg::rect &ui();
```