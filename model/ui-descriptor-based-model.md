# UI Descriptor-Based Model

## Preface

Descriptor describe a state of something, EKG enjoy of descriptor-oriented programming for cover memory-safety and flexibility, here we will discuss how EKG define each important descriptor.

Note: this topic always update because new added widgets.

## Runtime

### Descriptor Definition

A descriptor can be anything: stack, callback, button, label, etc.

Strictly definitions:  
| - | For elabore complex node-descriptors, each descriptor virtual-address must end with suffix `_at` and each descriptor must have own unique `ekg::at_t at`.  
| - | Of course the type of descriptor should be `static const ekg::type`.  
| - | The not found option `static t not_found`.  
| - | Ultimately the logic operators `==` `!=`, also, making sure `not_found` is always 'not-found', for prevent bypass risks.  
| - | Cast operator to `ekg::at_t` using the own at.

If is not defined like this, EKG does not consider a property descriptor, because we can have many descriptors-like, but not descriptor concept from EKG.

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
  /* mandator fields */
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

  operator &ekg::at_t() {
    return this->at;
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

The nameclature pattern `*_t` allows to define a function `ekg::descriptor(/* etc */)` using the descriptor name, for querying a descriptor as defined here:
```cpp
namespace ekg {
  descriptor_t &descriptor(ekg::at_t &at) {
    /* etc */
    return descriptor_t::not_found;
  }

  /**
   * This is optionally may some descriptors will not have
   * this type of query wrapper.
   * 
   * This may implement algorithms to reduce linear querying,
   * but is not the best approach and does not allow direct
   * branch prediction, as virtual-address do.
   **/
  descriptor_t &descriptor(std::string_view tag) {
    /* etc */
    return descriptor_t::not_found;
  }
}
```

As pointed in memory handling architecture model paper, we need make the virtual-address `ekg::at_t` as reference for prevent pointless behavior.

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

The `ekg::make<ekg::stack_t>` as defined here:
```cpp
case ekg::type::stack:
  ekg::stack_t &temp {ekg::any_static_cast<ekg::stack_t>(&descriptor)};
  ekg::at_t at {ekg::pools.stack.push_back(temp)};
  ekg::stack_t &ref {ekg::pools.query(at)};
  ref.at = at;
  return ref;
```

Stack query options:  
| - | `ekg::stack_t &ekg::stack(ekg::at_t &at)` for virtual-address branch-prediction querying.  
| - | `ekg::stack_t &ekg::stack(std::string_view tag)` for non-branch-prediction linear querying.

### Callback

A callback describe any task: internal-events, actions or free-use-case.

As defined here:
```cpp
// ekg/task/callback.hpp

namespace ekg {
  struct info_t {
  public:
    std::string tag {};
    void *p_data {nullptr}; // user-programmer address, not EKG-related
  };

  using callback_function_t = void(*)(ekg::info_t&)

  struct callback_t {
  public:
    /* optional fields */
    ekg::info_t info {};
    std::function(void(ekg::info_t&)) lambda {};
    ekg::callback_function_t function {nullptr};
  };
}
```

Note that we use a raw-ptr in `ekg::info_t`, yeah, but we can not cover memory safe here, using pools does not solve the problem here because the user-programmer must use of pools for things, and is not a standard, likely, the user-programmer must safety work by-self with it, since it is not related to EKG, EKG does not use this `p_data` for nothing, unless declare.

The `ekg::make<ekg::callback_t>` as defined here:
```cpp
case ekg::type::callback:
  ekg::callback_t &temp {ekg::any_static_cast<ekg::callback_t>(&descriptor)};
  ekg::at_t at {ekg::pools.callback.push_back(temp)};
  ekg::callback_t &ref {ekg::pools.query(at)};
  ref.at = at;
  return ref;
```

Callback query options:  
| - | `ekg::callback_t &ekg::callback(ekg::at_t &at)` for virtual-address branch-prediction querying.

### Button

#### Notations

Actions:

```cpp
// ekg/ui/action.hpp
namespace ekg {
  enum class action {
    press,
    release,
    drag,
    scroll,
    focus,
    unfocus
  };

  constexpr size_t enum_action_size {static_cast<size_t>(ekg::action::unfocus) + 1};

  template<size_t element_size>
  using actions_t = std::array<std::array<ekg::at_t, element_size>, ekg::enum_action_size>
}
```

Layers:

```cpp
// ekg/ui/action.hpp
namespace ekg {
  enum class layer {
    background,
    highlight,
    outline,
    active
  };

  constexpr size_t enum_action_size {static_cast<size_t>(ekg::layer::active) + 1};

  template<size_t element_size>
  using layers_t = std::array<std::array<ekg::at_t, element_size>, ekg::enum_action_size>
}
```

#### UI descriptors

Button can be defined as:

```cpp
// ekg/ui/button/button.hpp
namespace ekg {
  struct button_t {
  public:
    struct check_t {
    public:
      ekg::value<std::string> text {};
      ekg::value<bool> value {};
      ekg::flags_t dock {ekg::dock::left | ekg::dock::center};
      bool box {};
    };

    constexpr static size_t rect {0};
    constexpr static size_t box {1};
    constexpr static size_t element_size {2};
  public:
    /* etc */

    /* optional fields */
    std::string tag {};
    std::vector<ekg::button_t::check_t> checks {};
    ekg::flags_t dock {ekg::dock::left};
    ekg::actions_t<ekg::button_t::element_size> actions {};
    ekg::layers_t<ekg::button_t::element_size> layers {};
  public:
    /* etc */
  };
}
```

The `ekg::make<ekg::button_t>` can be defined as:
```cpp
case ekg::type::button:
  ekg::button_t &ref {ekg::any_static_cast<ekg::button_t>(&descriptor)};
  break;
```

** NEW WIDGETS WILL BE PLACED HERE SOON, THANKS **

## Conclusions

EKG is a GUI library, that is it.
