# The Problem

## Preface

As known for creating GUI at user-programmer side is currently complex and old, to change it, we will discuss here.

EKG is a significant project of my life, I wasted lot of time coding EKG, but it is what I love to do, graphics, GUIs, and GPU-accelerated softwares.

Many years coding EKG I made a lot of mistakes decisions and now after 3 years, I need to think better about my library. I am alone in this journey, but I believe we need sometimes do two-steps backward and re-write, re-do, re-think the architecture of us projects.

Code-safety references for write this article:  
~ [[2] Code-safety](./code-safety.md)

❤

`all enumerated-morals are subjectively-sacred for us, all is insanely lovely fast and deathly, but the knowledge is ethic-objective as the fact of love existence`,

Thank you Astah 🐈‍⬛,  
Thank you God.

---

## The Problem

#### Status-Quo

I was reading the code for create widgets and I noticed one thing, I mean, is not hard to notice that; the GUI creation method is horrible.

You can stack widgets inside a frame/container bla-bla.
```cpp
ekg::frame("meow", {.x = 0.0f, .y = 0.0f, .w = 200.0f}, ekg::dock::none);
ekg::label("Meow!", ekg::dock::fill);
ekg::button("Meow Here", ekg::dock::fill | ekg::dock::next);
```

And voila we have this simple GUI frame.  
![splash-meow-gui](../splash/splash-meow-gui.png?raw=true)

I do not think it is a simple-easy way to create GUIs, imagine increasing the complexity of GUI context, a lot of worse code to just create GUI. So the point is not rewrite all the entire library but only focus on the important parts, the user-programmer side.

#### User-Programmer Side

`User-programmer` means you, the one who use the EKG library, so you are called user-programmer, for almost 3 years I did not even think about you, I was only focusing on the runtime and widgets. Because of this, some stupids decisions were made for EKG, like the part of creating widgets.

May you think, how we can improve it? And the anwser is:  
NO useless [OO](https://en.wikipedia.org/wiki/Object-oriented_programming) (Object-Oriented) features.

3 things:  
-- user-programmer  
-- ui-object  
-- widget-object  

When a widget is created, the user-programmer only has access to the ui-object, then the runtime internally generate from ui-object a widget, linked to this ui-object.

The point obvious here:  
-- Why [OO](https://en.wikipedia.org/wiki/Object-oriented_programming) is used here?   
-- Why are there lot of bloat setters/getters?

#### Answering All These Questions

Normally a GUI runtime [OO](https://en.wikipedia.org/wiki/Object-oriented_programming)-based have naturally overheads, there is no problem, like we need focus on the algorithm and not only in [branchless programming](https://en.algorithmica.org/hpc/pipelining/branchless/). EKG created the UI-object concept, if the user-programmer created an element, user-programmer only has access to the UI-object, a simple rule, but bloated.

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

If we want create a new element like a popup, we must create a class based from the `ekg::ui::abstract`.

```cpp
class popup : public ekg::ui::abstract {
protected:
  int32_t wtf_number_for_something {};
public:
  popup *may_we_boom();
}
```

And an abstract widget:
```cpp
class popup_widget : public ekg::ui::abstract_widget {
public: // always public
  // internal reserved case fields
public:
  // virtuals method(s)
}
```

Doing like this way, is extremly bloated. The user-programmer side projection is actually worse.  

```cpp
ekg::frame("meow", {.x = 0.0f, .y = 0.0f, .w = 200.0f}, ekg::dock::none)
  ->set_drag(ekg::dock::top)
  ->set_resize(ekg::dock::left | ekg::dock::bottom | ekg::dock::right)
  ->lol_really_builder_pattern("?");
```

UI-object concept needs many getters/setters, this was a stupid decision made in the past, Rina, why did you made this? God.

Getters/setters is a cool way to improve security, but sometimes there is no reason for getters/setters. Like why a setter for set the frame dimensions? just no. EKG must re-write this part and write a new model pattern.

#### Descriptors

Vulkan, DirectX 11/12, system libraries, wayland-clients (compositors), and more others technologies implements the object-state descriptors.

Now EKG must follow this way too, also as necessary for a better between GPU-API(s) standard compatibility.  
EKG already implemented descriptors for samplers objects. Lets look better.

```cpp
uint32_t previous_size {f_renderer.font_size};
f_renderer.set_size(512);

FT_Load_Char(
  typography_font_face.ft_face,
  ekg::utf_string_to_char32("🐮"),
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
sampler_alloc_info.p_data = typography_font_face.ft_face->glyph->bitmap.buffer;

return ekg::allocate_sampler(
  &sampler_alloc_info,
  p_sampler
);
```

Cool?

---

## The Solution

Today most of programmers do not want to hardcode or write large codes for small output(s), so imagine a large bloated code to output graphical user interface, a mess.

The C++ std version used in EKG is 17, but EKG does not even implement any of features, rarely `constexpr` or new `std::` features; may you find only `std::string_view`, which you can think okay, we do not need to be forced to use alternative ways, as example, you can have a macro instead the use of `template`:

```cpp
#define ekg_lerp(a, b, t) a + (b - a) * t

namespace ekg {
  template<typename t>
  constexpr t lerp(t a, t b, t delta) {
    return a + (b - a) * delta;
  }
}

// both are the same, so any-way

ekg_lerp(1.0f, 2.0f, 0.5); // 1.5f
ekg::lerp<float>(1.0f, 2.0f, 0.5); // 1.5f
```

If we can macro for these operations, why we need C++17 then?

At sametime we have consistency of macro but we have a bad image for the community, "hey you must use `insert-modern-cpp-features`". Now, I will think better, instead making confuses decisions.

#### Design

Now we must take all the previous knowneldge and implement for the user-programmer widget creation.

Descriptors for create each UI element, descriptors for options, descriptors for details, descriptors for internal functions, descriptors for themes, and all descriptors for user-programmer application-side-input(s).

```cpp
bool do_something {};

ekg::make<ekg::frame_t>(
  { 
    .tag = "idk a frame?",
    .options = {
    .rect = {.w = 50.0f, .h = 50.0f},
    .resize = ekg::dock::none,
    .drag = ekg::dock::full
  }
);

ekg::make<ekg::label_t>(
  {
    .tag = "idk meow?",
    .text = "tijolo",
    .dock = ekg::dock::fill,
    .options = {
    .text_dock = ekg::dock::left
  }
);


ekg::make<ekg::checkbox_t>(
  {
    .tag = "idk ?<>?<>?>> meow?",
    .text = "click here if u brain",
    .value = ekg::value<bool>(&do_something),
    .text = {
      .value =
    }
    .options = {
      .text_dock = ekg::dock::left,
      .
    }
  }
);

```

And voila^2, new protype model. But it keeps bloated, how can we simplify this?

Well we can, here is:

```cpp
ekg::checkbox_t check {
  /* add stuff */
};

ekg::make(check);

check.text = "bla";
ekg::make(bla);

bla.text = "boo";
```

## Conclusion

Now some benefits we gain with this new model:
``
* Memory-safe descriptors:  
  | - | Recyclable descriptors with many possible designs.  
  | - | Easily by-widget theme and many options settings.  e 
  | - | No raw-ptr(s) or smart-ptr(s) for descriptors. As proven [here](./proofs#Raw-Pointers-Unsafety) and (here)[./proofs/#Raw-PointersCrazyUnsafety], it is too dangerous for an interface context.

* Stack-system for better query performance and widgets internal-logic.
