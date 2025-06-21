# Architecture

## Preface

With design decisions and memory-model complete explained we can now elabore how EKG is internally projected.

## Pools-Usage Model 

Pools should be used ideally in EKG for a better syntaxy-result.

### Pools

With pools we can store an unique specific descriptor, then, direct access by it is own index position.
So we can store every single type of descriptors in each designed pool:

```cpp
namespace ekg::io {
  extern struct pools_t {
  public:
    ekg::pool<ekg::stack_t> stack {ekg::stack_t::not_found};
    ekg::pool<ekg::callback_t> callback {ekg::callback_t::not_found};
    ekg::pool<ekg::sampler_t> sampler {ekg::sampler_t::not_found};
    ekg::pool<ekg::property> button_property {ekg::property::not_found};
    ekg::pool<ekg::button_t> button {ekg::button_t::not_found};
    /* etc */
  } pools;
}
```

### Make

For registry decriptors, EKG define a function `make<t>` where `t` is the descriptor. Each descriptor must be handled by a pool.

```cpp
namespace ekg {
  /**
   * User-programmer
   **/
  template<typename t>
  t &make(t descriptor) {
    switch (t::type) {
    case ekg::type::*:
      /**
       * A safety illegal cast: we know the type, we do not care about.
       * Thanks for blessing every night.
       **/
      ekg::*_t &temp {ekg::any_static_cast<ekg::*_t>(&descriptor)};
      ekg::at_t at {ekg::pool.*.push_back(temp)};
      ekg::*_t &ref {ekg::pool.query(at)};

      ref.at = at;

      /* ... */

      return ref;
    case ekg::type::*:
      /* etc */
      return ref;
    }

    return t::not_found;
  }
}

namespace ekg::io {
  t &make(t descriptor) {
    /* etc */
  }
}
```

Possible implementation:
```cpp
namespace ekg {
  template<typename t>
  t &make(
    t descriptor
  ) {
    switch (t::type) {
    case ekg::type::stack:
      return ekg::pools.stack.push_back(
        ekg::io::any_static_cast<t>(&descriptor)
      );
    case ekg::type::callback:
      return ekg::pools.callback.push_back(
        ekg::io::any_static_cast<t>(&descriptor)
      );
    case ekg::type::sampler:
      return ekg::pools.sampler.push_back(
        ekg::io::any_static_cast<t>(&descriptor)
      );
    case ekg::type::button:
      ekg::button_t &button {
        ekg::pools.button.push_back(
          ekg::io::any_static_cast<ekg::button_t>(&descriptor)
        )
      };

      ekg::property_t &property {
        ekg::pools.button_property.push_back({})
      };

      property.is_childnizate = false;
      property.is_children_docknizable = false;

      button.at.flags = t::type;
      property.descriptor_at = button.at;

      property.at.flags = t::type;
      button.property_at = property.at;

      ekg::core::registry(property);
      return button;
    }

    return t::not_found;
  }
}
```

### Query

Also the nameclature pattern `*_t` allows to define a function `ekg::descriptor(/* etc */)` using the descriptor name, but the query should be `query<t>` where `t` must be a valid descriptor, as defined here:
```cpp
namespace ekg {
  template<typename t>
  t &query(ekg::at_t &at) {
    /* etc */
    return t::not_found;
  }

  /**
   * This is optionally may some descriptors will not have
   * this type of query wrapper.
   * 
   * This may implement algorithms to reduce linear querying,
   * but is not the best approach and does not allow direct
   * branch prediction, as virtual-address do.
   **/
  template<typename t>
  t &query(const std::string_view &tag) {
    /* etc */
    return t::not_found;
  }
}
```

Possible implementation:
```cpp
template<typename t>
t &query(
  ekg::at_t &at
) {
  switch (t::type) {
    return ekg::io::any_static_cast<ekg::sampler_t>(
      &ekg::pools.sampler.query(at)
    );
  case ekg::type::callback:
    return ekg::io::any_static_cast<ekg::callback_t>(
      &ekg::pools.callback.query(at)
    );
  case ekg::type::sampler:
    return ekg::io::any_static_cast<ekg::sampler_t>(
      &ekg::pools.sampler.query(at)
    );
  case ekg::type::button:
    return ekg::io::any_static_cast<ekg::button_t>(
      &ekg::pools.button.query(at)
    );
  }

  return t::not_found;
}
```

As pointed in memory-handling model articles, we need make the virtual-address `ekg::at_t` as reference for prevent pointless behavior.

## Overall-Runtime Model

### Widgets-Logic

Each UI element contains a descriptor and a widget place where logic is made.

Can be simplified in:
```cpp
namespace ekg::ui {
  /**
   * Where metrics are re-calculated or extensive work is made.
   **/
  void reload(
    ekg::property_t &property,
    ekg::descriptor_t &descriptor
  );

  /**
   * Where input events are invoked:
   * - pre: checks if a widget is being hovering
   * - process: process all behavior stuff
   * - post: reset hovering state --- highlight is a different state but uses of hovered state
   **/
  void event(
    ekg::property_t &property,
    ekg::descriptor_t &descriptor,
    const ekg::io::stage &stage
  );

  /**
   * A high-frequency fixed-framerate update, used for smooth animations.
   **/
  void high_frequency(
    ekg::property_t &property,
    ekg::descriptor_t &descriptor
  );

  /**
   * Check if a widget should rebuffering or just pass the previous buffers
   * content to the allocator.
   * 
   * The smart-caching.
   **/
  void pass(
    ekg::property_t &property,
    ekg::descriptor_t &descriptor
  );

  /**
   * Redraw and pass the new buffers to GPU.
   **/
  void buffering(
    ekg::property_t &property,
    ekg::descriptor_t &descriptor
  );
}
```

May widgets contains custom functions to be able embed. For example scrollbar, which can be reused any where.

### Abstract

The property give to us the type of widget, so we can call any methods for each type of widget, this feels a bit ugly, but we need make memory-safe each part of code.

```cpp
#define ekg_core_abstract_todo(type, at, todo) \
  switch (type) { \
    case ekg::type::button: { \
      ekg::button_t &descriptor { \
        ekg::query<ekg::button_t>(at) \ 
      }; \
      if (descriptor == ekg::button_t::not_found) { \
        break; \
      } \
      todo \
      break; \
    /* etc */ \
    } \
  } \
```

### High-Frequency 

High-frequency operations is designed to perform fixed-animations, active widgets and focused widgets. EKG should update synced with current framerate or a fixed chosen framerate. 

```cpp
size_t size {this->high_frequency_list.size()};
for (size_t it {}; it < size; it++) {
  ekg::at_t &at {this->high_frequency_list.at(it)};
  ekg::property_t &property {ekg::property(at)};

  if (property == ekg::property_t::not_found) {
    this->this->high_frequency_list.erase(this->high_frequency_list.begin() + it);
    size = this->high_frequency_list.size();
    continue;
  }

  ekg_core_abstract_todo(
    property.descriptor_at.flags,
    property.descriptor_at,
    ekg::ui::high_frequency(property, descriptor);
  );

  if (!property.widget.is_high_frequency) {
    this->this->high_frequency_list.erase(this->high_frequency_list.begin() + it);
    size = this->high_frequency_list.size();
  }
}
```

## Conclusions

EKG architecture use of everything possible to fit the memory-model and low-latency behavior.
