# Value Low-Frequency-Based Mmodel

## Preface

The way of taking information from user-programmer side to EKG is a dangerous discussion.

## Runtime

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

This was a critic topic, because many devs can consider unsafe, but it is how EKG standard allows for make a safety program. Safe and low-latency.
