# UI Descriptor-Based Model

## Preface

Descriptor describe a state of something, EKG enjoy of descriptor-oriented programming for cover memory-safety and flexibility, here we will discuss how EKG define each important descriptor.

Note: this topic always update because new added widgets.

## Runtime

### Descriptor Definition

A descriptor can be anything: stack, callback, button, label, etc.

Strictly definitions:  
| - | For elabore complex node-descriptors, each descriptor virtual-address must end with suffix `_at` and each descriptor must have own unique `ekg::at_t at`.  
| - | Of course the type of descriptor should be `static constexpr ekg::type`.  
| - | The not found option `static t not_found`.  
| - | Ultimately the logic operators `==` `!=`, also, making sure `not_found` is always 'not-found', for prevent bypass risks.  
| - | Cast operator to `ekg::at_t` using the own at.

If is not defined like this, EKG does not consider a property descriptor, because we can have many descriptors-like, but not descriptor concept from EKG.

As defined here:

```cpp
namespace ekg {
  typedef size_t id_t;
}

struct descriptor_t {
public:
  static constexpr ekg::type type {/* type */};
  static descriptor_t not_found;
public:
  /* mandator field */
  ekg::at_t at {
    .unique_id = ekg::not_found,
    .index = ekg::not_found,
    .flags = ekg::not_found
  };
  bool is_dead {};
public:
  bool operator == (ekg::descriptor_t &descriptor) {
    return (
      (this->is_dead && descriptor_t::not_found.at == descriptor.at)
      ||
      (!this->is_dead && this->at == at)
    );
  }

  bool operator != (ekg::descriptor_t &descriptor) {
    return !(*this == descriptor);
  }

  operator ekg::at_t() {
    return this->at;
  }
};
```

Implementation:
- [ekg/io/descriptor.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/io/descriptor.hpp)

### `ekg::make<t>` and `ekg::query<t>`

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

As pointed in memory handling architecture model paper, we need make the virtual-address `ekg::at_t` as reference for prevent pointless behavior.

Implementations:
- [ekg/core/pools.hpp;ekg::query<t>](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/core/pools.hpp)
- [ekg/core/pools.hpp;ekg::make<t>](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/core/pools.hpp)

### Necessary-Macros

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

Implementation:
- [ekg/core/pools.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/core/pools.hpp)

### Descriptors

There is no a fixed place for declaring descriptors, they can be found implemented any-where under EKG source. 

Implementations:
- [ekg/handler/callback.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/handler/callback.hpp)
- [ekg/gpu/sampler.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/gpu/sampler.hpp)
- [ekg/ui/property.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/ui/property.hpp)
- [ekg/ui/stack.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/ui/stack.hpp)
- [ekg/ui/button/buton.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/ui/button/button.hpp)
- [ekg/ui/frame/frame.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/ui/frame/frame.hpp)
- [ekg/ui/label/label.hpp](https://github.com/vokegpu/ekg/blob/version-core/include/ekg/ui/label/label.hpp)

## Conclusions

EKG is a GUI library, that is it.
