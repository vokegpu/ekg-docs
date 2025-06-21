# Formatting

## Preface

This topic is dedicated for any-formatting model: the package-pattern, input-binding or whatever needs a text.

## Package-Formatting Model

### Notation

```
ekg/
  /**
   * For any type of feature.
   **/
  <package>/
    <impl>/
      <impl-from-base.hpp>
    <headers-base.hpp>
  /**
   * Internal features may not need the previously approach.
   **/
  <package>/
    <internal-feature.hpp>
```

As said, not all features may need a base-header, since any-header can be a base without implementation.

### Implementation

An example of how it should look, note: the EKG differ from this for obvious reasons, it is a fixed-example of notation.

```
ekg/
  core/
    runtime.hpp
    callback.hpp
    pools.hpp
  ui/
    theme/
      design.hpp
      manager.hpp
    button/
    ...
    abstract.hpp
    property.hpp
    action.hpp
    layer.hpp
  draw/
    typrography/
      font_renderer.hpp
      glyph.hpp
    shape/
      shape.hpp
    allocator.hpp
  gpu/
    vulkan/
      model.hpp
      vk.hpp
    opengl/
      model.hpp
      gl.hpp
    sdl3/
      model.hpp
      sdl3.hpp
    model.hpp
    api.hpp
  input/
    handler.hpp
    input.hpp
  platform/
    sdl/
      sdl2.hpp
      sdl3.hpp
    glfw/
      glfw.hpp
    platform.hpp
```

View from user-programmer:

```cpp
/* example of sdl3 + vulkan */

#include <ekg.hpp>
#include <ekg/platform/sdl/sdl3.hpp>
#include <ekg/gpu/vulkan/vk.hpp>

new ekg::platform::sdl3(/* etc */);
new ekg::gpu::vulkan(/* etc */);

/* example of sdl2 + opengl */

#include <ekg.hpp>
#include <ekg/platform/sdl/sdl2.hpp>
#include <ekg/gpu/opengl/gl.hpp>

new ekg::platform::sdl2(/* etc */);
new ekg::gpu::opengl(/* etc */);
```

## Input-Binding Tag Style Model

EKG needs standard a way for describing binding tag(s).

### Fundamentals

All mapped input(s) for custom tag(s) must follow a specific-case and text-description.

`<where*>-<action*>-<description>`

E.g a button action:
```c++
ekg::bind("button-active", "mouse-1");
```

E.g a frame action:
```c++
ekg::bind("frame-drag", "mouse-1");
```

E.g a listbox action:
```c++
ekg::bind("listbox-action-break-line", {"return", "keypad enter"});
```

As shown `<description>` is not necessary all the time.

### Notation

- `active` a desired functionality of something, like of a button press-and-release-over is named `active`, or a slider bar-dragging is `active`.
- `action` a specific generic-case where needs a description.

Others notations not written here are specialized, no notation-information is necessary.

### Examples

```c++
ekg::bind("textbox-action-select-all", {"rctrl+a", "lctrl+a"});
ekg::bind("textbox-action-select", {"rshift", "lshift"});
```

## Conclusions

Formats allow a standardnization of EKG environment.
