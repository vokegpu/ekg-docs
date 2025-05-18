# Archtecture-Model

## Praface

As previously discussed [here](./the-problem.md), now  we need take all advantages of descriptors and modern C++ to create a new memory handling-model for EKG. 

## Runtime

### Model Fundamentals

This applies to the descriptors, not the renderable widgets. Until descriptors memory-model are done, internal EKG systems will not implement pools for renderable widgets.

The argument is:
> If usage of pointers are potentially unsafe, pointers must be not used, instead, a memory pool must be used.

A replacement for [Bjarne Stroustrup](https://en.wikipedia.org/wiki/Bjarne_Stroustrup) definition of [object-oriented](https://en.wikipedia.org/wiki/Kristen_Nygaard) in the favor of descriptors: Object-state oriented.

![image](https://github.com/user-attachments/assets/130bc32f-f75b-491e-a2fe-ca11ac148fe9)

### C++-Style Reference

When executing programs in a OS, the memory accessed from the program is not directly the RAM, but a virtual place, reference is a pointer that points to the virtual place of something.

There is two ways to describing a reference in C++:  
  | - | C-style: `meow_t *p`.  
  | - | C++-style: `meow_t &v`.  

Both C and C are the same, as shown:
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

### Pool Concept

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

### Pool Definition

We have a pool concept, now we can increase the complexity of pool for the use case of EKG.

```cpp
namespace ekg {
  template<typename t>
  class pool {
  protected:
    t not_found;
    std::vector<t> pool {};
  public:
    pool(t not_found) : not_found(not_found) {}
  };
}
```

Quering specified `t` is defined as:
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

`ekg::at_t` must be a reference at query, because here, the pool can re-index the entire elements, and MUST be synced with `ekg::at_t` if not found, everything should pre-declared to increase the memory-handling safety. 

### Resources

With pools we can store an unique specific descriptor, then, direct access by it is own index position.
So we can store every single type of descriptors in each designed pool, with safe query functions:

```cpp
// ekg/io/resources.hpp

#include <array>

namespace ekg::io {
  extern struct resources_t {
  public:
    ekg::pool<ekg::checkbox_t> checkbox_pool {};
    ekg::pool<ekg::property_t> checkbox_property_pool {};
    ekg::pool<ekg::button_t> button_pool {};
    ekg::pool<ekg::property_t> button_property_pool {};
  } resources;
}

namespace ekg {
  ekg::checkbox_t &checkbox(ekg::at_t &at) {
    if (at.index >= ekg::resources.checkbox_pool.size()) return;

    ekg::checkbox_t &ref {
      ekg::resources.checkbox_pool.at(at.index)
    };

    return (
      ref.unique_id == at.id
      ?
      ekg::resources.checkbox_pool.at(at.index);
      :
      ekg::checkbox_t::not_found
    );
  }

  ekg::property_t &property(ekg::at_t &at) {
    ekg::pool<ekg::property_t> *p_property_pool {nullptr};
    switch (at.type) {
    case ekg::type::checkbox:
      p_property_pool = &ekg::resources.checkbox_property_pool;
      break;
    case ekg::type::textbox:
      p_property_pool = &ekg::resources.textbox_property_pool;
      break;
    }

    if (!p_property_pool) {
      return ekg::property_t::not_found;
    }

    bool found {};
    if (
        at.index >= p_property_pool->pool.size()
        ||
        p_property_pool->query(at).unique_id != at.unique_id
    ) {
      p_property_pool->index(at);
    }

    return (
      at.index < p_property_pool->pool.size()
      &&

      ?

      :
      ekg::property_t::not_found
    );
  }
}
```

Querying descriptors is totally safe.

```cpp
ekg::at_t find_my_checkbox { .id = 0, .index = 64 };
ekg::checkbox_t &checkbox {ekg::checkbox(find_my_checkbox)};

if (checkbox == ekg::checkbox_t::not_found) {
  ekg::log() << "not found :c";
} else {
  ekg::log() << "found :3";
}
```

Generic querying is safe, a necessary MACRO is defined:
```cpp
#define EKG_QUERY_DESCRIPTOR(at) \
  ekg::checkbox &checkbox {ekg::checkbox(at)}; \
  ekg::button_t &button {ekg::button(at)}; \
  /* all descriptors */ \
```

An example of generic-querying use for children property:
```cpp
struct frame_t {
public:
  constexpr static ekg::frame_t not_found {/* reserved behavior-case */};
public:
  std::vector<ekg::at_t> children {};
};

// etc

frame_t &frame {ekg::frame(my_stack, "window")}; // find for a frame tagged with 'window'
if (frame == ekg::frame_t::not_found) {
  return;
}

ekg::button_t &my_button {ekg::make(ekg::button_t { /* properties */ })};
frame.children.push_back(my_button); // added

ekg::checkbox_t &my_check {ekg::make(ekg::checkbox_t { /* properties */ })};
frame.children.push_back(my_check); // added

// etc

ekg::frame_t &descriptor {ekg::frame(this->at_descriptor)};
if (descriptor == ekg::frame_t::not_found) return;
for (ekg::at_t &at : descriptor.children) {
  EKG_QUERY_DESCRIPTOR(at);
  height += child.rect.y + child.rect.h;
}
```

## Conclusion

Now EKG is pool-safety, there is way to leak memory unless you force it.
