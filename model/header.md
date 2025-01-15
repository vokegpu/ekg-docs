# Architecture-Model

## Preface

As known for creating GUI at user-programmer side is currently complex and old, to change it, we will discuss here.

EKG is a significant project of my life, I wasted lot of time coding EKG, but it is what I love to do, graphics, GUIs, and GPU-accelerated softwares.

Many years coding EKG I made a lot of mistakes decisions and now after 3 years, I need to think better about my library. I am alone in this journey, but I believe we need sometimes do two-steps backward and re-write, re-do, re-think the architecture of us projects.

This article contains some base-references created for prove everything written by me here, take a look on the basics of this article, the new-rules/code-of-conduct for this project, and code-safety references for write this article:  
~ [[1] Project-rules](./project-rules.md)  
~ [[2] Code-safety](./code-safety.md)

❤

`all enumerated-morals is subjectively-sacred for us, all is insanely lovely fast and deathly, but the knowledge is ethic-objective as the fact of love existence`,

Thank you Astah 🐈‍⬛,  
Thank you God.

---

## The Problem

#### Status-Quo

I was reading the code for create widgets and I noticed one thing, I mean, is not hard to notice that; the GUI creation method is horrible.

You can stack widgets inside a frame/container bla-bla.
```c++
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
NO useless OO (Object-Oriented) features.

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
  abstract *do_some_stupid_thing();
  abstract *do_meows();
  abstract *bla_bla_setts();
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

And an abstract widget:
```c++
class popup_widget : public ekg::ui::abstract_widget {
public: // always public
  // internal reserved case fields
public:
  // virtuals method(s)
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

Now EKG must follow this way too, also as necessary for a better between GPU-API(s) standard compatibility.  
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

Now we must take all the previous knowneldge and implement for the user-programmer widget creation.

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

#### Benefits

Now some benefits we gain with this new model:

* Stack-based system for better query performance and widget(s) internal-logic.
* Memory-safety.
* Easily by-widget theme, and options settings.
* Recycle descriptors (properties, widgets-descriptors, etc).

---

## Runtime Fundamentals

As all explained previous, we have one problem and one solution, now we need to discuss how to implement implement, exploiting memory-safe points.

#### Summary

* Stack is where all widgets are store, used on application-side as GUI context.
```c++
// namespace ekg (ekg/ui/stack.hpp)
struct stack_t {
public:
  const char *p_tag {};
  std::vector<ekg::ui::abstract*> children {};
  uint64_t counter {};
}
``` 

* Properties is a descriptor for abstract widgets, which address a generic content (by-widget), children, and user context-flags.
```c++
// namespace ekg (ekg/ui/properties.hpp)
struct properties_t {
public:
  const char *p_tag {};
  ekg::type type {};
  ekg::id unique_id {};
  void *p_descriptor {};
  void *p_widget {};
  void *p_stack {};
  ekg::properties_t *p_abs_parent {};
  ekg::properties_t *p_parent {};
  std::vector<ekg::properties_t*> children {};
  bool is_parentable {};
}
```

* Result is an enum type that contains useful generic flags for any-operations.
```c++
// namespace ekg (ekg/io/memory.hpp)
enum result {
  success,
  failed,
  widget_not_found
};
```

* Flags is an uint64_t number type reserved for bits storage or enum storage.
```c++
// namespace ekg (ekg/io/memory.hpp)
typedef uint64_t flags;
```

* ID is an uint64_t number type reserved for unique ID.
```c++
typedef uint64_t id;
```

#### Memory-Safety Implementation

First `std::unique_ptr` must be used for current updating widget(s), making safety the memory management for runtime. References are raw ptr(s), EKG does not care about it.

* [`std::vector<std::unique_ptr<ekg::ui::abstract>>`](./code-safety.md#Memory-Safety) code-safety proof.
```c++
class runtime {
protected:
  std::vector<std::unique_ptr<ekg::ui::abstract>> loaded_widget_list {};
  // fields etc
};
```

For collection operations, EKG make a between raw and smart-ptr unsafe space, but user-programmer must follow the EKG standard for no-problems.

* [`new_widget_instance<t>()`](./code-safety.md#Safe-Instance-Creation) code-safety proof.
```c++
// namespace ekg (ekg/io/safety.hpp)
template<typename t>
t *new_widget_instance() {
  return dynamic_cast<t*>(
    ekg::core->push_back_widget_safety(
      std::unique_ptr<abstract>(
        dynamic_cast<abstract*>(new t {})
      )
    ).get()
  );
}
```
```c++
// namespace ekg (ekg/core/runtime.hpp)
ekg::ui::abstract *ekg::runtime::push_back_widget_safety(ekg::ui::abstract *p_widget) {
  return this->loaded_widget_list.emplace_back(
    std::unique_ptr<ekg::ui::abstract>(p_widget)
  );
}
```

#### Make

Many widget-descriptors are not the same, then it is required to generic safety `static_cast` to any as ptr.

```c++
// namespace * (ekg/io/memory.hpp)
#define ekg_cast_to_any_as_ptr(t, any) (t*)((void*)&any)
```

Make function must be a template function `ekg::make<t>`, which create a descriptor named data-descriptor and generate what we will call here an `ekg::ui::abstract`; strictly created under some rules:

1- A widget descriptor must be created under a internal-cache, and no application-side, we want make sure that we are not handling trash memory.

2- The widget must implement OO features, and use of addressed data-descriptor, initialized at once.

3- The current `ekg::make<t>` invoke must be inside a valid stack-instace.

```c++
// namespace ekg (ekg/io/safety.hpp)
template<typename t>
ekg::ui::abstract *ekg::make(t descriptor) {
  ekg::stack_t *p_stack {
    ekg::core->get_current_stack()
  };

  if (p_stack == nullptr) { // rule 3
    ekg::log(ekg::log::error) << "Failed failed to create a widget instance, the current stack instance is null"
    return nullptr;
  } 

  ekg::ui::abstract *p_created_widget {
    nullptr
  };

  ekg::properties_t properties {
    .p_tag = descriptor.p_tag,
    .type = descriptor.type,
    .unique_id = ekg::core->generate_unique_id()
  };

  switch (descriptor.type) {
    case ekg::type::frame: {
      ekg::frame_t *p_descriptor {
        ekg_cast_to_any_as_ptr(ekg::frame_t, descriptor)
      };

      ekg::ui::frame *p_frame {
        ekg::core->new_widget_instance<ekg::ui::frame>() // rule 1
      };

      p_frame->descriptor = *p_descriptor;

      properties.p_descriptor = &p_frame->descriptor; // rule 2
      properties.p_widget = &p_frame;

      p_created_widget = p_frame;
      break;
    }
    /* etc */
  }

  p_created_widget->properties = properties;

  ekg::properties_t *p_current_parent_properties {
    ekg::core->get_current_parent_properties()
  };

  if (
    p_created_widget->properties.is_parentable
    &&
    p_current_parent_properties != nullptr
  ) {
    ekg::add_child_to_parent(
      &p_current_parent_properties,
      &p_created_widget
    );
  }

  return p_created_widget;
}
```

#### Register Properties and Descriptors

EKG previous used an ID system for hash mapping, but there is no reason for use that, as known the new stack-based system, we can separate each element by chunks of stacks, and then apply an AABB for stack query(s).

For collecting, `ekg::properties` contains a field named `children`, which is used on most of widgets.

```c++
// namespace ekg (ekg/io/algorithm.hpp)
ekg::flags ekg::add_child_to_parent(
  ekg::properties_t *p_parent_properties,
  ekg::properties_t *p_child_properties
) {
  if (p_children == nullptr || p_properties == nullptr) {
    ekg::log() << "Failed to add child to parent, `null`, `null`";
    return ekg::result::failed;
  }

  if (
    p_child_properties->p_parent != nullptr
    &&
    p_child_properties->p_parent->unique_id == p_parent_properties->unique_id
  ) {
    return ekg::result::success;
  }

  p_child_properties->p_parent = p_parent_properties;
  p_child_properties->p_abs_parent = p_parent_properties->p_abs_parent;
  p_parent_properties->children.push_back(p_properties);

  return ekg::result::success;
}
```

The unique ID is generated increasing a global ID.
```c++
// ekg::runtime::generate_unique_id
ekg::id ekg::runtime::generate_unique_id() {
  return ++this->global_id;
}
```

#### Widget Use-Case Reserved

For handling objects inside a widget, we can iterate over the children directly without searching for a hash.

```c++
for (ekg::properties_t *&p_child_properties : this->properties.children) {
  // do here any operation
}
```

#### Widget Lifetime Memory-Safety

Operations like destroy, all are managed by the recycler, we do not directly delete the `ekg::ui::abstract`, but set a flag, and make sure all unused are cleaned, for example:

```c++
// namespace ekg (ekg/io/algorithm.hpp)
ekg::flag ekg::destroy(ekg::stack_t *p_stack, ekg::properties_t *p_destroy_widget_properties) {
  if (p_stack == nullptr) {
    ekg::log(ekg::log::error) << "Failed to destroy widget, `null` p_stack";
    return ekg::result::failed;
  }

  if (p_destroy_widget_properties == nullptr) {
    ekg::log(ekg::log::error) << "Failed to destroy widget, `null` p_destroy_widget_properties";
    return ekg::result::failed;
  }

  uint64_t counter {
    p_stack->counter++
  };

  p_destroy_widget_properties->was_destroy = true;

  for (ekg::properties_t *&p_properties : p_destroy_widget_properties->children) {
    ekg::destroy(p_stack, p_properties);
  }

  if (counter == 0) {
    p_stack->counter = 0;

    if (p_destroy_widget_properties->p_parent != nullptr) {
      std::vector<ekg::properties_t*> &parent_children {
        p_destroy_widget_properties->p_parent->children
      };

      parent_children.erase(
        std::remove_if(
          parent_children.begin(),
          parent_children.end(),
          [](ekg::properties_t *&p_properties) {
            return p_properties->unique_id == p_destroy_widget_properties->unique_id
          }
        )
      );
    }
  }

  return ekg::result::success;
}

// namespace ekg (ekg/io/algorithm.hpp)
ekg::flag ekg::find_and_destroy(
  ekg::stack_t *p_stack,
  std::string_view widget_tag
) {
  if (p_stack == nullptr) {
    ekg::log(ekg::log::error) << "Failed to destroy widget, `null` stack";
    return ekg::result::failed;
  }

  return ekg::destroy(p_stack, ekg::find(p_stack, widget_tag));
}
```

```c++
ekg::find_and_destroy(&this->my_gui, "frame-widget");
ekg::destroy(&this->my_gui, ekg::find("frame-widget"));
```

After destroying everything recursively, we can safety delete from the memory using the recycler.

```c++
// ekg::runtime::initialize_internal_tasks
this->service_handler.allocate() = ekg::task {
  .info = {
    .tag = "gc",
    .p_data = nullptr
  },
  .function = [this](ekg::info &info) {
    std::erase(
      std::remove_if(
        this->loaded_widget_list.begin(),
        this->loaded_widget_list.end(),
        [](std::unique_ptr<ekg::ui::abstract> &p_widget) {
          return p_widget->properties.was_destroy;
        }
      ),
      this->loaded_widget_list.end()
    );
  }
};
```

---

# Copyright

Copyright (c) 2022-2025 Rina Wilk / vokegpu@gmail.com