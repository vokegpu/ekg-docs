# Packages Pattern Model

## Preface

As far EKG is getting headers, the stupid package structure will affect the new features.

## Source

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

## Conclusion

This is important as quack.
