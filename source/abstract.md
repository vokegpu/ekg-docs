# Overview

- [Fundamentals](/ekg-docs/abstract/#fundamentals)

- [Methods](/ekg-docs/abstract/#methods)

# Fundamentals

The abstract [widget](/ekg-docs/widget/) is not used for the user-programmer part and is not recommended for any purpose. [Abstract widget](/ekg-docs/widget/#abstract widget, and abstract ui data) is used by the runtime core as a polymorphism heritage base for handling everything. 

# Methods

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