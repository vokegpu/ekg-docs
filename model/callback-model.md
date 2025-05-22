# Callback-Model

## Preface

We will discuss here about the callback-system model, the callback-system is used for EKG engine internal-events; widgets actions likely press, release, drag etc.

## Run

```cpp

// 120fps

ekg::stack_t stack {};
ekg::make<ekg::checkbox_t>(stack, {.tag = "god-mode", .text = "oi quica aqui pra ficar gowd mode"});
ekg::callback_t &my {
  ekg::make<ekg::callback_t>(
    stack,
    {
      .function = [](ekg::infot_t &info) {
        ekg::stack_t &stack {ekg::stack(info.stack_at)};
        if (stack == ekg::stack_t::not_found) {
          return;
        }

        ekg::checkbox_t &check {ekg::checkbox(stack, "god-mode")};
        if (check == ekg::checkbox_t::not_found) {
          return;
        }

        if (info.p_data == nullptr) {
          return;
        } 

        // user-programmer, not related to EKG it-self, so: unsafe
        world::entity *p_entity {static_cast<world::entity*>(info.p_data)};
        p_entity->is_god_mode = check.value;
      }
    }
  )
};

/* etc */

if (m_is_current_meow) {
  ekg::checkbox_t &checkbox {ekg::checkbox(this->popup.query("meow"))};
  if (checkbox != ekg::checkbox_t::not_found) {
    checkbox.actions[ekg::checkbox_t::check] = static_cast<ekg::at_t>(ekg::callback(stack, "god-mode"));
  }
}
```

