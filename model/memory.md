# Memory

## Preface

As previously discussed [here](./design.md), now we need take all advantages of descriptors and modern C++ to create a new memory handling-model for EKG. 

## Memory-Pool Model

This model derive from an objective argument with complex restrictions to be affirmed as memory-handling safe model.

The proof of memory-safe contains two articles:  
| - | This document with code definitions and affirmations (done).  
| - | Logical mathematical proof (not done yet) in a paper to more complex afirmations.

Both contains buildable and runnable code for showcase.

### Fundamentals

The argument is:
> If any-pointer is potentially unsafe, no-one pointer must be used, instead, a memory pool must be used.

A replacement for [Bjarne Stroustrup](https://en.wikipedia.org/wiki/Bjarne_Stroustrup) definition of [object-oriented](https://en.wikipedia.org/wiki/Kristen_Nygaard):
```cpp
class widget {};
class button : public widget {};
class label : public widget {};
```

In favor of descriptors design: Object-state oriented
```cpp
struct property_t { .descriptor_at = /* virtual-address */ };
ekg::pool<property_t> property_pool {};

struct button_t { .property_at = /* virtual-address */ };
ekg::pool<button_t> button_pool {};

struct label_t { .property_at = /* virtual-address */ };
ekg::pool<label_t> label_pool {};
```

We will explains step-by-step the thinking until define the pool, descriptors, and virtual-address.

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

### Definition of Virtual-Address, Descriptor and Memory-Pool

#### Virtual-Address

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

### Descriptor

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

#### Memory-Pool

We have a pool concept, now we can increase the complexity of pool for the use case of EKG.  
`ekg::pool<t>`, where `t` expect a descriptor (as defined before), as defined here:

```cpp
template<typename t>
class pool {
protected:
  std::vector<t> loaded {};
  std::vector<t> cached {};
  ekg::id_t highest_unique_id {};
  size_t dead_virtual_address_count {};
  size_t trash_capacity {10};
  size_t virtual_memory_capacity {100};
public:
  /**
   * A virtual-memory space should initialize a capacity first.
   * 
   * Some cases:
   * - If memory is not enough intialliy, you can reserve more before use.
   * - If memory required is out of reserved-space, a `cached` memory is used
   * and only when GC runs that is inserted onto `loaded` --- Allowing dynamic
   * virtual safety-memory inserting.
   * 
   * The GC should always run at end of any program loop.
   **/
  pool() { this->loaded.reserve(this->virtual_memory_capacity); };
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

Of course, the user-programmer (programmer who uses EKG library) will not even touch on this, this is for EKG developers.

### The Ultimately Proof

You can run this code:

https://github.com/vokegpu/ekg-docs/blob/master/proofs/proofs.md#safety-descriptor-pool-query

Pick STD17.

For an objective proof with more dense mathematical proofs, you can read the paper here. (not done yet)

## Low-Latency Value Model

The way of taking information from user-programmer side to EKG is a dangerous discussion.

### The Concept

The value is not a simple structure, but a structure that connects general-purpose from user-programmer side to EKG side, because of this, EKG also called 'low-latency' must do it right.

First: raw-ptr reference if it should or not be unsafety used?
```cpp
template<typename t>
class value {
protected:
  t value {};
  t *p {};
  t previous {};
  bool changed {};
public:
  value(t *p_address) {
    this->ownership(p_address);
    this->changed = true;
  }

  value(t value) {
    this->get() = value;
    this->changed = true;
  }

  value(const char *p_char) {
    this->get() = p_char;
    this->changed = true;
  }

  void set(const p &value) {
    this->get() = p;
    this->changed = true;
  }

  t &get() {
    return p ? *p : value;
  }

  void ownership(t *p_address) {
    if (p_address == nullptr) {
      return;
    }

    this->p = p_address;
  }

  bool was_changed() {
    if (this->was_changed) {
      this->was_changed = false;
      return true;
    }

    t &get {this->get()};
    if (this->previous != get) {
      this->previous = get;
      return true;
    }

    return false;
  }
};
```

This works great but too unsafe, for example:
```cpp
ekg::value<int32_t> bla {};

{
  int32_t b {};
  bla.ownership(&b);
}

bla.set(2666); // death
```

It is too hard to find a way to improve the `ekg::value<t>`, EKG is a safety library, allowing ONLY this may affect the argument used for memory model. This should be strictly used.

### Mapping Memory Address

For working with `ekg::value<t>`, EKG must track all the virtual-address widgets where the ownership is under, as defeined here:

```cpp
namespace ekg {
  struct mapped_address_sign_info_t {
  public:
    std::vector<ekg::at_t> ats {};
    void *pv_address {};
  };

  extern struct signed_address_info_t {
  public:
    std::vector<ekg::mapped_address_sign_info_t> list {};
    size_t current {};
  } sign;
}
```

Signed address info contains the information about all current signed actions and the current bound. When mapping we should care of address but DO NOT use them, we will not use, just compare, so it is safe. When unmapping tracked ownerships, if the current address is signed in EKG, EKG will reset all ownerships to `nullptr`, making the GUI safety, note: it does not disable the GUI or something, this only reset the onwership address.

```cpp
namespace ekg {
  void map(void *pv_address) {
    if (pv_address == nullptr) {
      ekg::sign.current = ekg::not_found;
      return;
    } 

    ekg::stack_t &current_stack {
      ekg::query<ekg::stack_t>(ekg::gui.bind.stack_at)
    };

    if (current_stack == ekg::stack_t::not_found) {
      ekg::sign.current = ekg::not_found;
      return;
    }

    size_t size {ekg::sign.list.size()};
    for (size_t it {}; it < size; it++) {
      ekg::mapped_address_sign_info_t &info {ekg::sign.list.at(it)};
      if (info.pv_address == pv_address) {
        ekg::sign.current = it;
        return;
      }
    }

    ekg::sign.current = ekg::sign.list.size();
    ekg::sign.list.push_back({.ats = {}, .p = pv_address});
  }

  void unmap(void *pv_address) {
    if (pv_address) {
      return;
    }

    size_t size {ekg::sign.list.size()};
    for (size_t it {}; it < size; it++) {
      ekg::mapped_address_sign_info_t &info {ekg::sign.list.at(it)};
      if (info.pv_address == pv_address) {
        for (ekg::at_t &at : info.ats) {
          ekg_core_abstract_todo(
            at.flags,
            at,
            ekg::ui::unmap(descriptor); // all UIs must have this
          );
          /* etc */
        }

        ekg::sign.list.erase(ekg::sign.list.begin() + it);
        break;
      }
    }
  }
}
```

With this system, EKG allows safety low-latency memory coverage.

### Value Standard Usage

Strictly usages of `ekg::value<t>`:  
| - | Ownership actions must be used under `ekg::map(void *pv_address)` and `ekg::unmap(void *pv_address)`.

Let's cover an example of how ownership SHOULD be used.

```cpp
struct entity_info_t {
public:
  std::string tag {};
  size_t unique_id {};
};

struct entity_state_info_t {
public:
  bool is_alive {};
  bool is_invisible {};
};

class entity_base {
protected:
  entity_info_t entity_info {};
  entity_state_info_t state_info {};
public:
  entity_base(entity_info_t info, entity_state_info_t state_info)
    : entity_info(info), state_info(state_info) {}

  ~entity_base() {
    // dead
  }

  entity_info_t &get_info() {
    return this->entity_info;
  }

  entity_state_info_t &get_state_info() {
    return this->entity_state_info_t;
  }
};

// etc

entity_base *p_entity {
  new entity_base(
    {.unique_id = 20, .tag = "cow"},
    {.is_alive = false, .is_invisible = false}
  )
};

void io::events::on_pick_entity(
  entity_base *p_entity
) {
  if (p_entity == nullptr) {
    return;
  }

  ekg::stack_t &entity_pick_popup {ekg::context("entity-pick-popup")};
  if (entity_pick_popup == ekg::stack_t::not_found) {
    return;
  }

  ekg::invoke(entity_pick_popup); // invoke stack
  ekg::map(p_entity); // map this stack

  ekg::checkbox("is-alive")
    .check
    .ownership(&p_entity->get_state_info().is_alive);

  ekg::checkbox("is-invisible")
    .check
    .ownership(&p_entity->get_state_info().invisible);

  ekg::map(); // ends but do not unmap
}
```

This world UI example is simple and is safe, the user-programmer should be right about the address of `entity_base`, unless, the safety EKG system will not cover this. For example, if this entity does not exists anymore, the GUI-context ownerships should be reseted, if not, the crash will occur in GUI. 

```cpp
void world::memory::kill_entity(
  entity_base *p_entity
) {
  if (p_entity == nullptr) {
    return;
  }

  // this will unmap IF previously mapped, reseting all onwerships
  ekg::unmap(p_entity);

  delete p_entity;
  p_entity = nullptr;
}
```

## Conclusion

Likely, if someone says "a program is memory-safe" you can make some questions if is or not, but EKG, you can just read the argument principle, understand the model, and confirm: it is memory-safe.

Of course, EKG is not done yet, but it is how we will cover the memory address for descriptors, soon, we will remove ALL object-oriented from EKG.

This was a critic topic, it is how EKG standard allows for make a safety program. Safe and low-latency.
