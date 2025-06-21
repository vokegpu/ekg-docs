# Design

## Preface

Why EKG-programming model and what decisions of EKG-programming model.

## Legacy EKG-programming Model

### Excessive Object-Oriented

Legacy EKG excessively used OO (Object-Oriented) for everything: UI-interfaces, widgets and hardware-interfaces.

Abstract base of an UI for user-programmer side.
```cpp
class abstract {
protected:
  ekg::id_t id {}; // like wtf
public:
  abstract *do_some_stupid_thing();
  abstract *do_meows();
  abstract *bla_bla_setts();
  bla_t getters_bla();
};
```

A frame widget in legacy EKG contains two classes: `frame` for user-programmer side and `frame_widget` for update/render logic.

```cpp
class frame : public ekg::ui::abstract {
protected:
  int32_t wtf_number_for_something {};
public:
  popup *may_we_boom();
}

class frame_widget : public ekg::ui::abstract_widget {
public: // always public
  // internal reserved case fields
public:
  // virtuals method(s)
}
```

For using the frame in user-programmer, the final code looks like this:

```cpp
ekg::frame("meow", {.x = 0.0f, .y = 0.0f, .w = 200.0f}, ekg::dock::none)
  ->set_drag(ekg::dock::top)
  ->set_resize(ekg::dock::left | ekg::dock::bottom | ekg::dock::right)
  ->lol_really_builder_pattern("?");
```

At this point, EKG legacy model resulted in many problems at side of user-programer. 
Getters/setters is a cool way to improve security, but sometimes there is no reason for getters/setters. Like why a setter for set the frame dimensions? just no. EKG must re-write this part and write a new model pattern.

### Descriptors

Vulkan, DirectX 11/12, system libraries, wayland-clients (compositors), and more others technologies implements the object-state descriptors.

Now EKG must follow this way too, also as necessary for a better between GPU-API(s) standard compatibility.  
EKG already implemented descriptors for samplers objects. Lets look better.

```cpp
uint32_t previous_size {f_renderer.font_size};
f_renderer.set_size(512);

FT_Load_Char(
  typography_font_face.ft_face,
  ekg::utf8_to_utf32("🐮"),
  FT_LOAD_RENDER | FT_LOAD_COLOR | FT_LOAD_DEFAULT
);

f_renderer.set_size(previous_size);

sampler_alloc_info.w = static_cast<int32_t>(typography_font_face.ft_face->glyph->bitmap.width);
sampler_alloc_info.h = static_cast<int32_t>(typography_font_face.ft_face->glyph->bitmap.rows);

sampler_alloc_info.gl_wrap_modes[0] = GL_REPEAT;
sampler_alloc_info.gl_wrap_modes[1] = GL_REPEAT;
sampler_alloc_info.gl_parameter_filter[0] = GL_LINEAR;
sampler_alloc_info.gl_parameter_filter[1] = GL_LINEAR;
sampler_alloc_info.gl_internal_format = GL_RGBA;
sampler_alloc_info.gl_format = GL_BGRA;
sampler_alloc_info.gl_type = GL_UNSIGNED_BYTE;
sampler_alloc_info.gl_generate_mipmap = GL_TRUE;
sampler_alloc_info.pv_data = typography_font_face.ft_face->glyph->bitmap.buffer;

return ekg::allocate_sampler(
  &sampler_alloc_info,
  p_sampler
);
```

### Memory-Unsafe and Verbose-STDnization

It is not an unknown knowneldge that any-raw-ptr is insanely potentially unsafe, you can track or use smart-ptr(s) but it is not enough for confirm that is really safety.

The legacy EKG abuses of raw-ptr(s), making it smart-ptr(s) does not make it totally-safety, for lot of reasons. Also the std can be verbose alot, for example `std::option<t, s>` or `std::shared_ptr<t>` or `std::unique_ptr<t>`.

While overall softwares can be unsafe if used raw-ptr(s), you can not track if a desired memory-space exists until you touch, and this touch can cost the entire application to crash.

As proven [here](../proofs/proofs.md#raw-pointers-unsafety) and [here](../proofs/proofs.md#raw-pointers-crazy-unsafety), it is too dangerous. Let this for memory-model topic.

## The Solution

Today most of programmers do not want to hardcode or write large codes for small output(s), so imagine a large bloated code to output graphical user interface, a mess.

### Design

Now we must take all the previous knowneldge and implement a new model, we expect to the final result be like this:

```cpp
ekg::make<ekg::frame_t>(
  { 
    .tag = "meow",
    .rect = {.w = 50.0f, .h = 50.0f},
    .resize = ekg::dock::none,
    .drag = ekg::dock::full
  }
);

ekg::make<ekg::label_t>(
  {
    .tag = "moo",
    .text = "a tewt",
    .dock_text = ekg::dock::left,
    .dock = ekg::dock::fill
  }
);

ekg::make<ekg::button_t>(
  {
    .tag = "buttons and checks",
    .dock = ekg::dock::next | ekg::dock::fill,
    .checks = {
      {.text = "check for meow", .box = ekg::dock::left, .dock = ekg::left}
      /* etc */
    }
  }
);
```

And voila^2, new protype model. But it keeps bloated, how can we simplify this?

Well we can, here is:

```cpp
ekg::stack_t my_stack {
  .tag = "my-gui-context"
};

ekg::make<ekg::stack_t>(my_stack);

ekg::button_t button {
  .tag = "my-button",
  .dock = ekg::dock::next | ekg::dock::fill,
  .checks = {
    {.text = "text"}
  }
};

ekg::button_t checkbox {
  .tag = "my-button",
  .dock = ekg::dock::next | ekg::dock::fill,
  .checks = {
    {.text = "text", .box = ekg::dock::left}
  }
};

button.checks[0].text = "click-here";
ekg::make<ekg::button_t>(button);

button.checks[0].text = "click-here to!!o";
ekg::make<ekg::button_t>(button);

checkbox.checks[0].text = "check hewe";
ekg::make<ekg::button_t>(checkbox);
```

Of course, this topic details the reasons for this design-decision. In next topics we will discuss very detailed about the implementation and rules.

## Conclusion

Now some benefits we gain with this new model:
``
* Memory-safe descriptors:  
  | - | Recyclable descriptors with many possible designs.  
  | - | Easily by-widget theme and many options settings.  
  | - | No raw-ptr(s) or smart-ptr(s) for descriptors.  
  | - | Efficiently stack-memory usage.

* Stack-system for better query performance and widgets internal-logic.
