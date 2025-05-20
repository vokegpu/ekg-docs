# Performance-Model

## Preface

Both performance sides CPU and GPU must be detailed developed, for a real performance gain, here we will discuss everything about.

## CPU

### Necessary-Macros

The property give to us the type of widget, so we can call any methods for each type of widget, this feels a bit ugly, but we need make memory-safe each part of code.

```cpp
#define EKG_WIDGET_CALL(abstract, property, callable)
  switch (property.type) {
  case ekg::type::button:
    ekg::button_t &button {ekg::button(property.descriptor_at)};
    if (button == ekg::button_t::not_found) break;
    ekg::ui::button::callable(property, abstract, button);
    break;
  case ekg::type::checkbox:
    ekg::checkbox_t &checkbox {ekg::button(property.descriptor_at)};
    if (checkbox == ekg::checkbox_t::not_found) break;
    ekg::ui::checkbox::callable(property, abstract, checkbox);
    break;
  case ekg::type::listbox:
    ekg::listbox_t &listbox {ekg::button(property.descriptor_at)};
    if (listbox == ekg::listbox_::not_found) break;
    ekg::ui::listbox::callable(property, abstract, listbox);
    break;
  /* etc */
  }
```

### Smart-Caching

CPU-side should not waste cycles for calculate new geometry resources, I always thought about smart-caching geometry in CPU.

Local geometry-buffer is important for skip useless geometry re-calculations, as defined here:
```cpp
// ekg/ui/abatract.hpp
// ekg::ui

struct abstract_t {
public:
  std::vector<float> gbuffer {};
  /* etc */
};
```

The main rendering loop uses of smart-cache for improve CPU-usage.

```cpp
// ekg/core/runtime.hpp
// ekg::runtime::render

this->allocator.invoke();

for (ekg::at_t &at : this->stack_list) {
  ekg::ui::abstract_t &widget {ekg::ui::abstract(at)}; 
  if (widget == ekg::ui::abstract_t::not_found || widget.is_dead || widget.is_invisible) {
    continue;
  }

  ekg::property_t &property {ekg::property(widget.property_at)};
  if (property == ekg::property_t::not_found) {
    continue;
  }

  /**
   * The smart-cache occurs here, with two possible cases:
   * 1- if should update: the redraw will recalculate the entire local geometry buffer (gbuffer) and update at end.
   * 2- if should not update: the redraw will copy the latest geometry buffer (gbuffer). 
   **/

  EKG_WIDGET_CALL(
    abstract,
    property,
    on_pre_redraw
  );

  if (!widget.should_update_geometry) {
    continue;
  }

  EKG_WIDGET_CALL(
    abstract,
    property,
    on_redraw
  );
}

this->allocator.revoke();
```

The `ekg::ui::*::on_pre_redraw` check for possibles low-latency changes. For example, check if an internal flag was changed or one size is different. Ultimately the efficient part <goes> here with low-latency checks.

### High-Frequency 

High-frequency operations is designed to perform fixed-animations, active widgets and focused widgets. EKG should update synced with current framerate or a fixed chosen framerate. 

```cpp
// ekg/core/runtime.hpp
// ekg::runtime::update

size_t size {this->high_frequency_list.size()};
for (size_t it {}; it < size; it++) {
  ekg::at_t &at {this->high_frequency_list.at(it)};
  ekg::property_t &property {ekg::property(at)};

  if (property == ekg::property_t::not_found) {
    this->this->high_frequency_list.erase(this->high_frequency_list.begin() + it);
    size = this->high_frequency_list.size();
    continue;
  }

  ekg::ui::abstract_t &abstract {ekg::abstract(property.abstract_at)};
  if (abstract == ekg::ui::abstract_t::not_found) {
    this->this->high_frequency_list.erase(this->high_frequency_list.begin() + it);
    size = this->high_frequency_list.size();
    continue;
  }

  EKG_WIDGET_CALL(
    abstract,
    property,
    update
  );

  if (!property.is_high_frequency) {
    this->this->high_frequency_list.erase(this->high_frequency_list.begin() + it);
    size = this->high_frequency_list.size();
  }
}
```

### Event

It is the same as legacy code, but with memory-safe cover.

```cpp

```

## GPU
