The cross-platform functions are not implicit, it is required
to explicit set-up the current window-lib-desired IO events.
`.p_os_platform` in EKG runtime property specify which
library platform is being currently used by application;
it is important to initialize the window-lib-desired,
as same the GPU API. 

The GLFW does not has a simple event system, in part, you
must call EKG in all input events.

```cpp
// glfw
new ekg::os::glfw(p_glfw_win)

ekg::os::glfw_keyboard_input(...)
ekg::os::glfw_mouse_input(...)
ekg::os::glfw_text_input(...)
```

The SDL, only requires a unique call.

```cpp
// sdl
new ekg::os::sdl(p_sdl_win)

ekg::os::sdl_poll_event(...)
```

A callback is purely a task, each callback created can be reused in any widget,
each time a widget is interacted by an target action;
the callback is fired, as-defined in widget.

The first `tag` argument, means the tag of callback.
The second `p_data` argument, means the user-programmer reserved ptr.
The last `function` argument, is the callable lambda used in the execution of callback. 

```cpp
p_slider->get_value() // ok
p_button->get_value() // ok
p_textbox->get_text() // ok

ekg::slider("oiii queria ser totosa", 2.0f, 0.0f, 3.0f, ekg::dock::fill)
  ->set_callback(ekg::action::press,   new ekg::callback(...))
  ->set_callback(ekg::action::release, new ekg::callback(...))
  ->set_callback(ekg::action::focus,   new ekg::callback(...));

ekg::button("oiii beijo", ekg::dock::fill)
  ->set_callback(
    ekg::action::press,
    new ekg::callback("oi", nullptr, [](ekg::callback::info info) {
      std::cout << info.tag << std::endl;
      info.id; // int
      info.type; // ekg::type
      info.slider; // slider
      info.button; // bool
      info.popup; // string
      info.checkbox; // bool
      info.combobox; // str
      info.p_data; // user-programmer purpose 
    }));
```

All internal IO events, are processed by the EKG update,
not all time, only when inputs are fired.

```cpp
ekg::update();
```