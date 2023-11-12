# # Overview

- [Fundamentals](/ekg-docs/listbox/#fundamentals)

- [Methods](/ekg-docs/listbox/#methods)

- [Container](/ekg-docs/listbox/#container)

# Fundamentals

The [listbox](/ekg-docs/listbox/#listbox) [widget](/ekg-docs/widget/) displays text fields as a 2D list. [Item](/ekg-docs/item/) stores each text field, representing a geometry component.

The first [items](/ekg-docs/item/) represent the category column, and the sub-[items](/ekg-docs/item/) from the category represent the rows.

```cpp
ekg::item &item = p_listbox->item();

item.insert("Cats");   // category: Cats
item.insert("Dogs");   // category: Dogs
item.insert("Plants"); // category: Plants

item.at(0).insert("Potato");
item.at(1).insert("Toto");
item.at(2).insert("Flower");
```

Each [container](/ekg-docs/listbox/#container) category is independent of the others, however, EKG [listbox](/ekg-docs/listbox/) contains one mode to link all [items](/ekg-docs/item/), named, single column mode.

```cpp
ekg::item &item = p_listbox->item();

item.insert("Name");        // category: Name
item.insert("Description"); // category: Description
item.insert("Status");      // category: Status

p_listbox->set_single_column_mode(true);

item.at(0).insert("Potato");
item.at(1).insert("Cat");
item.at(2).insert("Playing with kitty-friends!");

item.at(0).insert("Chiquinha");
item.at(1).insert("Cat");
item.at(2).insert("Carrying of children kitties!");
```

The single-column mode can also reserve empty spaces, to prevent issues in processing, but it is possible to perform a filling.

```cpp
item.fill(0, ekg::attr::box | ekg::attr::unselectable); // edit attributes
item.fill(0, "Name"); // edit name
```

# Methods

### Listbox

Set single-column mode state.

```cpp
ekg::ui::listbox *set_single_column_mode(bool state);
```

Get single-column mode state.

```cpp
bool is_single_column_mode();
```

Set dimension width in pixels, see details [here](/ekg-docs/widget/#dimension).

```cpp
ekg::ui::listbox *set_width(float w);
```

Get dimension width in pixels, see details [here](/ekg-docs/widget/#dimension).

```cpp
float get_width();
```

Set dimension height in factor-height, see details [here](/ekg-docs/widget/#dimension).

```cpp
ekg::ui::listbox *set_scaled_height(int32_t factor);
```

Get dimension height in factor-height, see details [here](/ekg-docs/widget/#dimension).

```cpp
int32_t get_scaled_height();
```

Get dimension height in pixels, see details [here](/ekg-docs/widget/#dimension).

```cpp
float get_height();
```

Set [layout dock](/ekg-docs/layout/#dock) position.

```cpp
ekg::ui::listbox *set_place(uint16_t dock);
```

Get the [item](/ekg-docs/item/) data.

```cpp
ekg::item &item();
```

Set the category [font](/ekg-docs/font/) used.

```cpp
ekg::ui::listbox *set_category_font_size(ekg::font font);
```

Get the category [font](/ekg-docs/font/) used.

```cpp
ekg::font get_category_font_size();
```

Set the [item](/ekg-docs/item/) [font](/ekg-docs/font/) used.

```cpp
ekg::ui::listbox *set_item_font_size(ekg::font font);
```

Get the [item](/ekg-docs/item/) [font](/ekg-docs/font/) used.

```cpp
ekg::font get_item_font_size();
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

# Container

Each row is a component container, a [listbox](/ekg-docs/listbox/) container allocates all [Items](/ekg-docs/item/) cursively opened and dynamically calculates the visible index, based on the opened components count.

When opening an [item](/ekg-docs/item/) component, the container increases the open counter, and when an [item](/ekg-docs/item/) component is closed, it decreases the open counter. This simple arithmetic system is useful for estimating container height.

The rendering optimization is possible with a unique height, and by default, it is [font](/ekg-docs/font/) normal size. Calculating a dynamically visible index requires normalizing the scroll value with the estimated height. `scroll` means the scrolling value, `len` is the container components' length, and `height` is the container's estimated height.

![](https://cdn.discordapp.com/attachments/1064693858245546045/1170436090063228939/848372603294974024.png?ex=6559088d&is=6546938d&hm=ecdad3b26c03adcd8c8242fb44d9be96cc461356831cfa59a7038cb73696def1&)

The dynamic index calculation inverts the signal of the `scroll` because the scroll vector must subtract the position of the rectangle, and for the formula, it is not required, since we want to get normalized.

All closed components are skipped in the rendering section, the [item](/ekg-docs/item/) component contains the count of total children, and it is used to increase the current index iteration.
