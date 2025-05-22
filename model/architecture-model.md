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

Unlike this, C++ reference allow compile-time type-safe programming, so, no mistakes occur, because you are building with known types.

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

### Pool-Concept

A memory pool is a space where `n` size of memory block is reserved (dynamic or not), and occuped when neeeds. This block of memory is index-based, so picking descriptors from the pool require a known index. Allowing branch prediction.

Pool-concept is defined as:
```cpp
// ekg/io/memory.hpp

#include <vector>

namespace ekg {
  template<typename t>
  using pool = std::vector<t>;
}
```

For defining how map-indices from a pool:
```cpp
// ekg/io/memory.hpp

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

### Pool-Definition

We have a pool concept, now we can increase the complexity of pool for the use case of EKG.

```cpp
namespace ekg {
  template<typename t>
  class pool {
  protected:
    t not_found;
    std::vector<t> pool {};
    size_t dead_virtual_address_count {};
    ekg::id_t highest_unique_id {};
  public:
    size_t trash_capacity {20};
  public:
    pool(t not_found) : not_found(not_found) {}

    ekg::at_t push_back(const t &copy) {
      ekg::at_t at {.index = this->pool.size(), .unique_id = ++this->highest_unique_id};
      this->pool.push_back(copy);
      return at;
    }
  };
}
```

Querying specified `t` is defined as:
```cpp
t &query(ekg::at_t &at) {
  if (
      at.index >= this->pool.size()
      ||
      this->pool.at(at.index).unique_id != at.unique_id
  ) {
    size_t size {this->pool.size()};
    for (size_t it {}; it < size; it++) {
      t &element {this->pool.at(it)};
      if (element.unique_id == at.unique_id) {
        at.index = it;
        return element;
      }
    }

    return this->not_found;
  }

  return this->pool.at(at.index);
}
```

`ekg::at_t` must be a reference at query, because if an element is not found, the pool try to re-index and ultimately if nothing helps just return as `not_found`.

Any dead element is marked with `is_dead` as defined here:

```cpp
bool kill(ekg::at_t &at) {
  t &element {this->query(at)};
  if (element == this->not_found) {
    return false;
  }

  element.is_dead = true;
  return static_cast<bool>(++this->dead_virtual_address_count);
}
```

Dealing with deadly virtual memory is safety, cleaning is not a priority because of erasing/inserting performance, we need to priority inserting, instead of immediate delete.

```cpp
void gc() {
  if (this->dead_virtual_address_count < this->trash_capacity) {
    return;
  }

  for (size_t it {}; it < size; it++) {
    t &element {this->pool.at(it)}; 
    if (!element.is_dead) {
      continue;
    }

    this->pool.erase(this->pool.begin() + it);
    size = this->pool.size();
  }

  this->dead_virtual_address_count = 0;
}
```

All these definitions must be followed for a safety-strictly memory-handling control.

### Resources

With pools we can store an unique specific descriptor, then, direct access by it is own index position.
So we can store every single type of descriptors in each designed pool:

```cpp
// ekg/core/pools.hpp

namespace ekg::io {
  extern struct pools_t {
  public:
    ekg::pool<ekg::checkbox_t> checkbox {ekg::checkbox_t::not_found};
    ekg::pool<ekg::property_t> checkbox_property {ekg::property_t::not_found};
    ekg::pool<ekg::button_t> button {ekg::button_t::not_found};
    ekg::pool<ekg::property_t> button_property {ekg::property_t::not_found};
    /* etc */
  } pools;
}

namespace ekg {
  ekg::checkbox_t &checkbox(ekg::at_t &at) {
    return ekg::io::pools.checkbox.query(at);
  }

  ekg::button_t &button(ekg::at_t &at) {
    return ekg::io::pools.button.query(at);
  }

  /* etc */

  ekg::property_t &property(ekg::at_t &at) {
    ekg::pool<ekg::property_t> *p_property_pool {nullptr};
    switch (at.type) {
    case ekg::type::checkbox:
      p_property_pool = &ekg::core::pools.checkbox_property;
      break;
    case ekg::type::textbox:
      p_property_pool = &ekg::core::pools.textbox_property;
      break;
    }

    if (!p_property_pool) {
      return ekg::property_t::not_found;
    }

    return p_property_pool->query(at);
  }
}
```

Querying descriptors is totally safe.

```cpp
ekg::at_t find_my_checkbox { .id = 20, .index = 64 };
ekg::checkbox_t &checkbox {ekg::checkbox(find_my_checkbox)};

/* brute force will occur if unique id 20 is not found */

if (checkbox == ekg::checkbox_t::not_found) {
  ekg::log() << "not found :c";
} else {
  ekg::log() << "found :3";
}
```

### Generic-Query

Okay here a big deal with descriptors and pools, generic is possible but limited to the architecture of EKG, while not a EKG standard, you can make unsafe everything.

```cpp

// ekg/io/memory.hpp
template<typename t>
constexpr t &any_static_cast(void *p_any) {
  return *static_cast<t*>(p_any); // illegal-casting
}

// ekg/io/resource.hpp
template<typename t>
t &query(ekg::at_t &at) {
  if (at.type == ekg::type::frame) {
    return ekg::any_static_cast<t>(&ekg::pools.frame.query(at));
  } else if (at.type == ekg::type::checkbox) {
    return ekg::any_static_cast<t>(&ekg::pools.checkbox.query(at));
  } /* else if etc */
  
  return ekg::any_static_cast<t>(&ekg::pools.callback.query(at));
}
```

Not EKG standard, but you are able to implement at your own risk, that is it.

## Conclusion

Likely, if someone says "a program is memory-safe" you can make some questions if is or not, but EKG, you can just read the argument principle, understand the model, and confirm: it is memory-safe.

Of course, EKG is not done yet, but it is how we will cover the memory address for descriptors, soon, we will remove ALL object-oriented from EKG.
