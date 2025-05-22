# UI Descriptor-Based Model

## Preface

Descriptors works describe a state of something, here the UI, we will discuss about how EKG handle it.

## Runtime

### Stack

A stack describe GUI-context: callbacks and widgets.

As defined here:
```cpp
// ekg/ui/stack.hpp

namespace ekg {
  struct stack_t {
  public:
    /* fixed fields */
    ekg::type type {ekg::type::STACK};
    ekg::id_t unique_id {};

    /* custom fields */
    std::string tag {};
    ekg::vector<ekg::at_t> callbacks {};
    ekg::vector<ekg::at_t> widgets {};
    /* etc if necessary */
  };
}
``` 

When a stack is created with `ekg::make<ekg::stack_t>` it is stored in a pool. For querying specific widgets you must specify the stack in first parameter of `ekg::make<t>`, as defined:
```cpp
// ekg/io/make.hpp
namespace ekg {
  ekg::stack_t &make(const ekg::stack_t &descriptor) {
    ekg::at_t at {
      ekg::pools.stack.push_back(descriptor)
    };
  }

  ekg::callback_t &make(const ekg::callback_t &descriptor) {
    ekg::at_t at {
      ekg::pools.callbacks.push_back(descriptor)
    };
  }

  /* etc */
}
```

### Widgets

## Conclusions

