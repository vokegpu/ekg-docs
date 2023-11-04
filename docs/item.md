# Overview

- Fundamentals
- Examples
- Methods

# Fundamentals

The EKG library contains an item collector named `ekg::item`, the item is a recursive vector collection.

# Examples

The item contains the following fields:

- `value` display-text/tag of item. 
- `attr` the attribute flag defines the rendering, and processing behavior.
- `child_list` child list to other items.
- `component` geometry data of item component.

Setting a value to an item is simple: 

```cpp
ekg::item item("animals"); // constructor explicit
ekg::item item {"animals"}; // initialization-list implicit
ekg::item item = {"animals"}; // assign initialization-list implicit

ekg::item item {}; // initialization-list implicit
item.value = "animals"; // accessing the field, but is not too recommendedd
```

Item contains attr (attributes) flags and state flags, both manually configurable or automatically by setting the value component with special chars.

Special chars:

- `\t` add automatically the attribute `ekg::attr::separator`.
- `\\` add automatically the attribute `ekg::attr::box`.
- `\1` add automatically the attribute `ekg::attr::category`.
- `\2` add automatically the attribute `ekg::attr::row`.
- `\3` add automatically the attribute `ekg::attr::row_member`.
- `\4` add automatically the attribute `ekg::attr::unselectable`.

```cpp
ekg::item item {"Animals"};
item.set_value("\t\\Select All");

// insert a std::vector<std::string> to item.
item.insert({"\\Dogs"});
item.insert({"\\Cats"});
item.insert({"\\Humans"});
```

You can iterate over the item children using for loop:

```cpp
ekg::item item("animals");
item.insert({"dogs", "cats", "humans"});

for (ekg::item &items : item) {
    std::cout << items.value << std::endl;
}
```

# Methods

Set value with attributes check, use this instead of setting directly the field. 

```cpp
void set_value(std::string_view _value);
```

Initialize an item child without generating a copy.

```cpp
ekg::item &emplace_back();
```

Insert an item copy in the child list.

```cpp
void push_back(const ekg::item &item);
```

Insert an item by setting the value.

```cpp
void push_back(std::string_view item_value);
```

Insert a `std::vector<ekg::item>` child list.

```cpp
void insert(const std::vector<std::string> &item_value_list);
```

Get the child item by index position.

```cpp
ekg::item &at(uint64_t index);
```

Check if has item children.

```cpp
bool empty() const;
```

Get the size of the child list.

```cpp
uint64_t size() const;
```

Get begin iterator point.

```cpp
std::vector<ekg::item>::iterator begin();
```

Get end iterator point.

```cpp
std::vector<ekg::item>::iterator end();
```

Get const begin iterator point.

```cpp
std::vector<ekg::item>::const_iterator cbegin() const;
```

Get const end iterator point.

```cpp
std::vector<ekg::item>::const_iterator cend() const;
```