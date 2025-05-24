# UI Descriptor-Based Model

## Preface

Descriptor describe a state of something, EKG enjoy of descriptor-oriented programming for cover memory-safety and flexibility, here we will discuss how EKG define each important descriptor.

## Runtime

### Descriptor Definition

A descriptor can be anything: stack, callback, button, label, etc.

Strictly definitions:  
| - | For elabore complex node-descriptors, each descriptor virtual-address must end with suffix `_at` and each descriptor must have own unique `ekg::at_t at`.  
| - | Of course the type of descriptor should be `static const ekg::type`.  
| - | The not found option `static t not_found`.  
| - | Ultimately the logic operators `==` `!=`, also, making sure `not_found` always is 'not-found', for prevent bypass risks.

As defined here:

```cpp
// ekg/io/memory.hpp
namespace ekg {
  typedef size_t id_t;
  static const ekg::id_t not_found {333666999};
}

// ekg/whatever/descriptor.hpp
struct descriptor_t {
public:
  static const ekg::type type {/* type */};
  static descriptor_t not_found;
public:
  /* mandatory fields */
  ekg::id_t unique_id {};
  ekg::at_t other_descriptor_at {};
  ekg::at_t at {};
public:
  bool operator == (ekg::descriptor_t &descriptor) {
    descriptor_t::not_found.unique_id = ekg::not_found; // assert
    return this->unique_id == descriptor.unique_id;
  }

  bool operator != (ekg::descriptor_t &descriptor) {
    return !(*this == descriptor);
  }
};
```

For registry decriptors, EKG define a function `make<t>` where `t` is the descriptor. Each descriptor must be handled by a pool.

```cpp
// ekg/io/make.hpp
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

Now let's cover the important descriptors from EKG.

### Stack

A stack describe GUI-context: widgets.

As defined here:
```cpp
// ekg/ui/stack.hpp
namespace ekg {
  struct stack_t {
  public:
    /* optional fields */
    std::string tag {};
    ekg::vector<ekg::at_t> widgets {};
  };
}
```

The make as defined here:
```cpp
namespace ekg
```

### Widgets

## Conclusions
