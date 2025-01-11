# Architecture Model

As known for creating GUI at user-programmer side is currently complex and old, to change it, we will discuss here.

The new-rules for this project:
* [project-rules](./project-rules.md)

---

## The Problem

#### Status-Quo

I was looking the code for create widgets and I noticed one thing, I mean, is not hard to notice that; the GUI creation method is horrible.

You can stack widgets inside a frame/container bla-bla.
```c++
ekg::frame("meow", {.x = 0.0f, .y = 0.0f, .w = 200.0f}, ekg::dock::none);
ekg::label("Meow!", ekg::dock::fill);
ekg::button("Meow Here", ekg::dock::fill | ekg::dock::next);
```

And voila we have this simple GUI frame.  
[splash-meow-gui](../splash/splash-meow-gui.png?raw=true)

I do not think it is a simple-easy way to create GUIs, increasing the complexity make worse to create. So the point is not rewrite all the entire library but only focus on the important parts, the user-programmer side.

#### User-Programmer Side

`User-programmer` means you, the one who use the EKG library, so you are called user-programmer, for almost 3 years I did not even think about you, I was only focusing on the runtime and widgets. Because of this, some stupids decisions were made for EKG, like the part of creating widgets.

May you think, how we can improve it? And the anwser is:  
NO MORE useless OO (Object-Oriented) features.

One obvious feature, the user-programmer interface with 3 layers concept:  
-- user-programmer  
-- ui-object  
-- widget-object  

When a widget is created, the user-programmer only has access to the ui-object, then the runtime internally generate from ui-object a widget, linked to this ui-object.

The point obvious here:  
-- Why [OO](https://en.wikipedia.org/wiki/Object-oriented_programming) is used here?   
-- Why is there lot of bloat setters/getters?

#### Answering All These Questions

Normally a GUI runtime OO-based have naturally overheads, there is no problem, like we need focus on the algorithm and not only in [branchless programming](https://en.algorithmica.org/hpc/pipelining/branchless/). EKG created the UI-object concept, if the user-programmer created an element, user-programmer only has acess to the UI-object, a simple rule, but bloated.

```c++
class abstract {
protected:
  ekg::id id {}; // like wtf
public:
  popup *do_some_stupid_thing();
  popup *do_meows();
  popup *bla_bla_setts();
  bla_t getters_bla();
};
```

If we want create a new element like a popup, we must create a class based from the `ekg::ui::abstract`.

```c++
class popup : public ekg::ui::abstract {
protected:
  int32_t wtf_number_for_something {};
public:
  popup *may_we_boom();
}
```

Doing like this way, is extremly bloated. The user-programmer side projection is actually worse.  

```c++
ekg::frame("meow", {.x = 0.0f, .y = 0.0f, .w = 200.0f}, ekg::dock::none)
  ->set_drag(ekg::dock::top)
  ->set_resize(ekg::dock::left | ekg::dock::bottom | ekg::dock::right)
  ->lol_really_builder_pattern("?");
```

UI-object concept needs many getters/setters, this was a stupid decision made in the past, Rina, why did you made this? God.

Getters/setters is a cool way to improve security, but sometimes there is no reason for getters/setters. Like why a setter for set the frame dimensions? just no. EKG must re-write this part and write a new model pattern.

#### Descriptors

Vulkan, DirectX 11/12, system libraries, wayland-clients (compositors), and more others technologies implements the object-state descriptors.

Now EKG must follow this way too, also as necessary for a better GPU-API(s) standard compatibility.  
EKG already implemented descriptors for samplers objects. Lets look better.

```c++
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

Cool ya?

---

## The Solution

Today most of programmers do not want to hardcode or write large codes for small output(s), so imagine a large bloated code to output graphical user interface, a mess.

The C++ std version used in EKG is 17, but EKG does not even implement any of features, rarely `constexpr` or new `std::` features; may you find only `std::string_view`, which you can think okay, we do not need to be forced to use alternative ways, as example, you can have a macro instead the use of `template`:

```c++
#define ekg_lerp(a, b, t) a + (b - a) * t

namespace ekg {
  template<typename t>
  t lerp(t a, t b, t delta) {
    return a + (b - a) * delta;
  }
}

// both are the same, so any-way

ekg_lerp(1.0f, 2.0f, 0.5); // 1.5f
ekg::lerp<float>(1.0f, 2.0f, 0.5); // 1.5f
```

If we can macro for these operations, why we need C++17 then?

At sametime we have consistency of macro but we have a bad image for the community, "hey you must use `insert-modern-cpp-features`". Now, I will think better, instead making confuses decisions.

#### Widget Creation

Now we must take all the previous knowneldge and apply for the user-programmer widget creation.

Descriptors for create each UI element, descriptors for options, descriptors for details, descriptors for internal functions, descriptors for themes, and all descriptors for user-programmer application-side-input(s).

```c++
bool do_something {};

ekg::stack_t my_window {
  .p_tag = "window-meow",
  .children = {
    ekg::make<ekg::frame_t>(
      { 
        .p_tag = "idk a frame?",
        .options = {
          .rect = {.w = 50.0f, .h = 50.0f},
          .resize = ekg::dock::none,
          .drag = ekg::dock::full
        }
      }
    ),
    ekg::make<ekg::label_t>(
      {
        .p_tag = "idk meow?",
        .p_text = "tijolo",
        .dock = ekg::dock::fill,
        .options = {
          .text_dock = ekg::dock::left
        }
      }
    ),
    ekg::make<ekg::checkbox_t>(
      {
        .p_tag = "idk ?<>?<>?>> meow?",
        .p_text = "click here if u brain",
        .value = ekg::value<bool>(&do_something),
        .options = {
          .text_dock = ekg::dock::left
        }
      },
    )
  }
};
```

And voila^2, new protype model. But it keeps bloated, how can we simplify this?

Well we can, here is:

```c++
ekg::stack_t my_window {
  .p_tag = "window-meow",
  .children = {
    ekg::frame("idomeow", /* blabla */),
    ekg::label("blabla", ekg::dock::fill),
    // ...
  }
};
```

# Runtime

As all explained previous, we have one problem and one solution, now we need to discuss how to implement implement, exploiting memory-safe points.

## Fundamentals

#### Summary

* Properties is a descriptor for abstract widgets, which address a generic content (by-widget descriptors), also children.
```c++
// namespace ekg::ui
struct properties_t {
public:
  const char *p_tag {};
  ekg::type type {};
  void *p_descriptor {};
  void *p_widget {};
  ekg::ui::properties_t *p_abs_parent_properties {};
  ekg::ui::properties_t *p_parent {};
  std::vector<ekg::ui::properties_t*> children {};
  bool is_parentable {};
}
```

#### Memory Creation Instace `ekg::make`

`ekg::make` must be a template function, which create a descriptor named data-descriptor and generate what we will call here a `widget`; strictly created under some rules:

1- A widget descriptor must be created under a internal-cache, and no application-side, we want make sure that we are not handling trash memory.

2- The widget must implement OO features, and use of addressed data-descriptor, initialized at once.

3- The current `ekg::make<t>` invoke must be inside a valid stack-instace.

```c++
#define ekg_cast_to_any_as_ptr(t, any) (t*)((void*)&any)

template<typename t>
ekg::ui::abstract *ekg::make(t descriptor) {
  if (ekg::runtime::p_current_stack == nullptr) { // rule 3
    ekg::log(ekg::log::error) << "Failed failed to create a widget instance, the current stack instance is null"
    return nullptr;
  } 

  ekg::ui::abstract *p_created_widget {
    nullptr
  };

  ekg::ui::properties_t properties {
    .p_tag = descriptor.p_tag,
    .type = descriptor.type,
    .id = (descriptor.id = ekg::core->generate_new_unique_id())
  };

  switch (descriptor.type) {
    case ekg::type::frame: {
      ekg::frame_t *p_descriptor {
        ekg_cast_to_any_as_ptr(ekg::frame_t, descriptor)
      };

      ekg::ui::frame *p_frame {
        ekg::core->new_frame_instance()
      };

      p_created_widget = p_frame;
      p_frame->descriptor = *p_descriptor;
      properties.p_data = &p_frame->descriptor; // rule 2

      break;
    }
    /* etc */
  }

  p_created_widget->properties = properties;
  ekg::core->map_new_widget(p_created_widget); // rule 1

  return p_created_widget;
}
```

#### Register Properties and Descriptors

By default, all widgets are able to collect other widgets. EKG must implement some internally methods to update and handle all these info(s). First, the ID for each widget, it needs to be generated with no collisions, but EKG does not need a complex ID system, just increasing one number is okay. 

```C++
ekg::id ekg::runtime::generate_new_unique_id() {
  return ++this->global_id;
}
```

For collecting, `ekg::ui::properties` contains a field named `children`, which is used on most of widgets.

```C++
void ekg::runtime::map_new_widget(
  ekg::ui::abstract *p_widget
) {
  p_widget->properties.p_widget = p_widget;
  this->mapped_widget[p_widget->properties.id] = p_widget;

  if (this->p_current_parent && p_widget->is_parentable) {
    ekg::ui::add_child_to_parent(
      this->p_current_parent.properties.children,
      this->p_widget->properties
    );
  }
}
```

#### Widget Use-Case Reserved

For handling objects inside a widget, we can iterate over the children directly without searching for a hash.

```c++
for (ekg::ui::properties_t *&)
```