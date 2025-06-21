# Hello World 🐈

## Preface

Making GUIs with EKG can be easy and at same time not friendly. EKG is mainly object-state based and offers two modes for creating GUIs context(s).

---

## Fundamental

### High-Level Mode

For creating widgets without a large descriptor configuration, EKG simplifies using a declared-specified function.

```c++
ekg::stack_t my_context {
  .tag = "meow"
};

ekg::frame({.tag = "frame", .rect = {20.0f, 20.0f, 100.0f, 200.0f}});
ekg::button({.tag = "oi", .text = "click here for meow", .dock = ekg::dock::fill | ekg::dock::next});

```

### Low-Level Mode

May you need configure directly the children stack.

```c++
ekg::stack_t my_context {
  .tag = "meow",
  .children = {
    ekg::make<ekg::frame_t>(
      {
        .tag = "frame",
        .rect = {20.0f, 20.0f, 100.0f, 200.0f}
      }
    ),
    ekg::make<ekg::button_t>(
      {
        .tag = "button",
        .text = "click here for meow",
        .dock = ekg::dock::fill | ekg::dock::next
      }
    )
  }
};
```

### Valuable Ownership

EKG works with ownership-reference for storing an UI element value, as exampled here:

```c++
bool my_ref {};
ekg::checkbox(
  {
    .tag = "idk",
    .text = "should terminal meow?",
    .value = ekg::value<bool>(&my_ref)
  }
);
```

Once created, the reference address is the actual read-write value by widget , if `ekg::value<t>` was not directly set, an internal cache is used.

```c++
bool my_new_ref {};
ekg::checkbox_t &checkbox {
  ekg::find("idk")
};

checkbox.value.move(&my_new_ref);
checkbox.value = ekg::value<bool>(&my_new_ref); // or
```

Setting a new reference address affect the internal value. 

```c++
ekg::checkbox_t checkbox {
  ekg::checkbox(
    {
      .tag = "idk",
      .text = "should terminal meow?",
      .value = ekg::value<bool>(true)
    }
  )
};

if (
    checkbox.value.get()
    ||
    (bool) checkbox.value
    ||
    static_cast<bool>(checkbox.value)
  ) {
  ekg::log() << "meow";
}

bool new_ref {};
checkbox.value.move(&new_ref); // now is not anymore `true`
```