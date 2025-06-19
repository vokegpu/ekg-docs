# Architecture-Model

## Preface

As previously discussed [here](./the-problem.md), now  we need take all advantages of descriptors and modern C++ to create a new memory handling-model for EKG. 

## Runtime

### Model Fundamentals

The argument is:
> If usage of pointers are potentially unsafe, pointers must be not used, instead, a memory pool must be used.

A replacement for [Bjarne Stroustrup](https://en.wikipedia.org/wiki/Bjarne_Stroustrup) definition of [object-oriented](https://en.wikipedia.org/wiki/Kristen_Nygaard) in favor of descriptors design: Object-state oriented.

![image](https://github.com/user-attachments/assets/fc643e9f-fe9c-4226-97e1-6d51ca7f68e4)

As proven [here](./proofs.md#safety-descriptor-pool-query) pools are the most safety use-case for EKG.

### C++-Style Reference

When executing programs in a OS, the memory accessed from the program is not directly the RAM, but a virtual place, reference is a pointer that points to the virtual place of something.

There is two ways to describing a reference in C++:  
  | - | C-style: `meow_t *p`.  
  | - | C++-style: `meow_t &v`.  

Both C and C++ are the same, as shown:
```cpp
void meow(int *p_meow) {
  *p_meow = 40;
}

void meow(int &meow) {
  meow = 40;
}

// x86-64 gcc 14.2
meow(int*):
  push rbp
  mov  rbp, rsp
  mov  QWORD PTR [rbp-8], rdi
  mov  rax, QWORD PTR [rbp-8]
  mov  DWORD PTR [rax], 40
  nop
  pop  rbp
  ret
meow(int&):
  push rbp
  mov  rbp, rsp
  mov  QWORD PTR [rbp-8], rdi
  mov  rax, QWORD PTR [rbp-8]
  mov  DWORD PTR [rax], 40
  nop
  pop  rbp
  ret
```

C-style reference is complete dangerous in most of cases, requires strictly validations and memory-handling, if not, undefined behaviors occurs and there is no way to eficiently work on it.

```cpp
struct meow_t {
public:
  bool was_created {};
};

ekg::flags_t create(meow_t *p_meow) {
  if (p_meow == nullptr) { // validation needed;
    // BUT it does not save you from your own mistakes: UNDETECTED DELETED MEMORY
    return ekg::result::failed;
  }

  p_meow->was_created = true; // <- sign that was created
  return ekg::result::success;
}
```

Of course, we can do that, but nothing prevents you from any programming mistakes, handling memory in this way is hard, now imagine it totally safe, it is impossible to set `nullptr` automatically all the time.

```cpp
meow_t meow {};
create(&meow); // ok, created

meow_t *p_meow {new meow_t {}};
created(p_meow); // hm dangerous but ok, created

delete p_meow;
p_meow = nullptr; // (?) wha, how could you explicit set as nullptr always?
create(p_meow); // (??)

delete p_meow;
create(p_meow); // (??) hbasjdhsbjhbdjh Biwbeihd

// etc

meow_t a {};
meow_t b { .p_meow = &a }; // (?) this is horrible

```

Unlike this, C++ reference allows compile-time type-safe programming for this model, so, here no mistakes occur, because you are building with known types.

```cpp
std::vector<meow_t> meows(2);
meow_t &meow {meows.at(0)}; // safe
meow.text = "must meow?";

{
  meow_t &humm_meow {meows.at(0)};
  humm_meow.text = "yes";

  meow_t &second_meow {meows.at(1)};
  second_meow.text = humm_meow.text;
}

meows.clear();

if (!meows.empty()) {
  // do here, safe
}
```

May you think, of course, it is safe, so, this is the way for handling descriptors in EKG.

### Memory-Model Simplification

A memory pool is a space where `n` size of memory block is reserved (dynamic or not), and occuped when neeeds. This block of memory is index-based, so picking descriptors from the pool require a known index. Allowing branch prediction.

Pool-concept is defined as:
```cpp
#include <vector>

namespace ekg {
  template<typename t>
  using pool = std::vector<t>;
}
```

For defining how map-indices from a pool:
```cpp
#include <cstdint>

namespace ekg {
  typedef uint64_t id_t;
  typedef uint64_t flags_t;

  struct at_t {
  public:
    ekg::flags_t type {};
    ekg::id_t id {};
    size_t index {};
  };
}
```

When defining the descriptor, an empty-case must be defined, where point to a safety controlled not-found behavior.

```cpp
struct meow_t {
public:
  static meow_t not_found { .is_error_reserved = true };
public:
  bool is_error_reserved {};
  ekg::at_t at_next {};
public:
  bool operator == (ekg::meow_t &other) {
    return this->is_error_reserved == other.is_error_reserved;
  }

  bool operator != (ekg::meow_t &other) {
    return this->is_error_reserved != other.is_error_reserved;
  }
};
```

With this descriptor-base done, we need now query it, and safety say if it was found or not.

```cpp
meow_t &query(ekg::at_t at, ekg::pool<meow_t> &pool) {
  return at.index >= pool.size() ? meow_t::not_found : pool.at(index);
}
```

This pool system is totally safe, no raw-ptr, no any kinda of smart-ptr to "prevent" memory issues.

```cpp
ekg::pool<meow_t> meow_pool {};
meow_pool.emplace_back();
meow_t &a {meow_pool.at(0)};

meow_pool.emplace_back();
meow_t &b {meow_pool.at(1)};
b.at_next.index = 0;

meow_t &search_meow {query({.index = 3654}, meow_pool)};
if (search_meow == ekg::meow_t::not_found) { 
  return; // there is no possible crash here, unless you force
}
```

### Definition of Virtual Address, Descriptor and Memory-Pool

#### Virtual Address

A virtual address `ekg::at_t` points to a safety space, as defined:
```cpp
namespace ekg {
  constexpr ekg::id_t not_found {29426662939}; // broken-heart hash

  struct at_t {
  public:
    static ekg::at_t not_found;
  public:
    ekg::id_t unique_id {ekg::not_found};
    size_t index {ekg::not_found};
    ekg::flags_t flags {ekg::not_found};
  public:
    bool operator == (ekg::at_t &at) {
      return this->flags == at.flags && this->unique_id == at.unique_id;
    }

    bool operator != (ekg::at_t &at) {
      return !(*this == at);
    }
  };
}
```

#### Descriptor

A complete technical details about descriptor will be discussed in some nexts topics, [here](./ui-descriptor-based-model.md), but now, for a simple definition, this is enough:

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
    descriptor_t::not_found.at = ekg::at_t::not_found; // assert
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

#### Memory-Pool

We have a pool concept, now we can increase the complexity of pool for the use case of EKG.  
`ekg::pool<t>`, where `t` expect a descriptor (as defined before), as defined here:

```cpp
template<typename t>
class pool {
protected:
  std::vector<t> loaded {};
  ekg::id_t highest_unique_id {};
  size_t dead_virtual_address_count {};
  size_t trash_capacity {10};
public:
  pool() {};
};
```

Inserting, as defined:

```cpp
t &push_back(const t &copy) {
  this->loaded.push_back(copy);

  size_t index {this->loaded.size() - 1};
  t &descriptor {this->loaded.at(index)};

  descriptor.at.unique_id = this->highest_unique_id++;
  descriptor.at.flags = static_cast<ekg::flags_t>(t::type);
  descriptor.at.index = index;

  return descriptor;
}
```

Querying specified `t` is defined as:
```cpp
t &query(ekg::at_t &at) {
  if (
    at.index >= this->loaded.size()
    ||
    this->loaded.at(at.index).at.unique_id != at.unique_id
  ) {
    size_t size {this->loaded.size()};
    for (size_t it {}; it < size; it++) {
      t &descriptor {this->loaded.at(it)};
      descriptor.at.index = it;
      if (descriptor.at.unique_id == at.unique_id) {
        at.index = it;
        return descriptor;
      }
    }

    return t::not_found;
  }
  t &descriptor {this->loaded.at(at.index)};
  return descriptor;
}
```


`ekg::at_t` must be a reference at query, because if an element is not found, the pool try to re-index and ultimately if nothing helps just return as `not_found`.

Any dead element is marked with `is_dead` as defined here:

```cpp
bool kill(ekg::at_t &at) {
  t &element {this->query(at)};
  if (element == t::not_found) {
    return false;
  }

  element.is_dead = true;
  return static_cast<bool>(++this->dead_virtual_address_count);
}
```

Dealing with deadly virtual memory is safety, cleaning is not a priority because of erasing performance, we need to priority inserting, instead of immediate delete.

So GC should be called at end of main program loop.

```cpp
void gc() {
  this->trash_capacity = 0; // for this example
  if (this->dead_virtual_address_count < this->trash_capacity) {
    return;
  }

  size_t size {this->loaded.size()};
  for (size_t it {}; it < size; it++) {
    t &element {this->loaded.at(it)}; 
    if (!element.is_dead) {
      continue;
    }

    this->loaded.erase(this->loaded.begin() + it);
    size = this->loaded.size();
  }

  this->dead_virtual_address_count = 0;
}
```

All these definitions must be followed for a safety-strictly memory-handling control.

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

Querying descriptors is memory-safe, since the type of descriptor is static constexpr defined, any invalid virtual address will return `t::not_found`.

```cpp
template<typename t>
t &query(
  ekg::at_t &at
) {
  switch (t::type) {`
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

Usage example:

```cpp
ekg::at_t find_my_checkbox { .id = 20, .index = 64 };
ekg::button_t &meow_check {ekg::query<ekg::button_t>(find_my_checkbox)};

/* brute force will occur if unique id 20 is not found */
/* and store the last index, allowing branch prediction */

if (meow_check == ekg::button_t::not_found) {
  ekg::log() << "not found :c";
} else {
  ekg::log() << "found :3";
}
```

Of course, the user-programmer (programmer who uses EKG library) will not even touch on this, this is for EKG developers.

### The Ultimately Proof

You can run this code:

https://github.com/vokegpu/ekg-docs/blob/master/model/proofs.md#safety-descriptor-pool-query

Pick STD17.

## Conclusion

Likely, if someone says "a program is memory-safe" you can make some questions if is or not, but EKG, you can just read the argument principle, understand the model, and confirm: it is memory-safe.

Of course, EKG is not done yet, but it is how we will cover the memory address for descriptors, soon, we will remove ALL object-oriented from EKG.
