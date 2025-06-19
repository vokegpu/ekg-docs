# Performance

## Preface

Both performance sides CPU and GPU must be detailed developed, for a real performance gain, here we will discuss everything about.

## Model

Two topics will be defined here:

* [CPU](./performance.md#CPU) --- about how buffers should be generated (technique) and overall-runtime execution.
* [GPU](./performance.md#GPU) --- rendering part where how buffers should be used and overall APIs-video used.

### CPU

#### Allocator

The `ekg::service::allocator` stands for CPU-side geometry handling, while not related directly to the GPU-API used, we need make sure for capture each part of draw before send to the GPU its-self.
`ekg::io::gpu_data_t` is the draw-call information, which map the stride of geometry resources located in VRAM. The CPU-side geometry resources use a simple vector to cache vertices.

```cpp
namespace ekg::service {
  class allocator {
  public:
    static std::vector<voke::io::gpu_data_t> *p_current_gpu_data_buffer;
    static std::vector<float> *p_current_geometry_buffer;
  public:
    void invoke() {
      /* initialize default geometries */
      /* reset GPU-data index */
    }

    voke::io::gpu_data_t &generate_data() {
      /* etc */

      this->gpu_data_instance++;

      if (ekg::service::allocator::p_current_gpu_data_buffer) {
        ekg::service::allocator::p_current_gpu_data_buffer->push_back(gpu_data);
      }

      return gpu_data;
    }

    void push_back_geometry(
      float x,
      float y,
      float u,
      float v
    ) {
      if (ekg::service::allocator::p_current_gpu_data_buffer) {
        ekg::service::allocator::p_current_gpu_data_buffer->push_back(x);
        ekg::service::allocator::p_current_gpu_data_buffer->push_back(y);
        ekg::service::allocator::p_current_gpu_data_buffer->push_back(u);
        ekg::service::allocator::p_current_gpu_data_buffer->push_back(v);
      }

      this->geometry_buffer.push_back(x);
      this->geometry_buffer.push_back(y);
      this->geometry_buffer.push_back(u);
      this->geometry_buffer.push_back(v);

      this->geometry_instance += 4;
    }

    void pass_gpu_data_buffer(std::vector<voke::io::gpu_data_t> &gpu_data_buffer) {
      /* tweaks on current data index for prevent issues like out of bouding index */
      this->gpu_data_buffer.insert(
        this->gpu_data_buffer.begin() + this->gpu_data_instance,
        gpu_data_buffer.begin(),
        gpu_data_buffer.end()
      );

      this->gpu_data_instance += gpu_data_buffer.size();
    }

    void pass_geometry_buffer(std::vector<float> &geometry_buffer) {
      /* tweaks on current data index for prevent issues like out of bouding index */

      this->geometry_buffer.insert(
        this->geometry_buffer.begin() + this->geometry_instance,
        geometry_buffer.begin(),
        geometry_buffer.end()
      );

      this->geometry_instance += geometry_buffer.size();
    }

    void revoke() {
      /* send to GPU-VRAM all the CPU-side geometry resources */
      /* reduce GPU-data size */
    }
  };
}
```

The legacy EKG allocator does not cover how the GPU-VRAM is handled, there is no capacity-system like a `std::vector`, but this will be discussed later, not here. 

#### Smart-Caching

The purpose for smart-caching is skiping unncessary complete redraws, reducing the CPU-side useless-pain wasting cycles.

Local buffers is important for skip useless geometry re-calculations, as defined here:
```cpp
struct buffer_t {
public:
  std::vector<float> geometry_buffer {};
  std::vector<voke::gpu::data_t> gpu_data_buffer {};
  /* etc */
};
```

The main rendering loop uses of smart-cache for improve CPU-usage.

```cpp
this->allocator.invoke();

for (ekg::at_t &at : this->stack) {
  ekg::property_t &property {ekg::property(at)};
  if (property == ekg::property_t::not_found) {
    continue;
  }

  /**
   * The smart-cache occurs here, with two possible cases:
   * 1- if should update: the redraw will recalculate the entire local geometry buffer (geometry_buffer and gpu_data_buffer) and update at end.
   * 2- if should not update: the redraw will copy the latest geometry buffer (geometry_buffer and gpu_data_buffer). 
   **/

  ekg_core_abstract_todo(
    property.descriptor_at.flags,
    property.descriptor_at,

    ekg::ui::pass(property, descriptor);
  );

  if (!property.widget.should_update_geometry) {
    continue;
  }

  ekg_core_abstract_todo(
    property.descriptor_at.flags,
    property.descriptor_at,
    ekg::ui::buffering(property, descriptor);
  );
}

this->allocator.revoke();
```

The `ekg::ui::pass` check for possibles low-latency changes. For example, check if an internal flag was changed or one size is different. Ultimately the efficient part <goes> here with low-latency checks.

#### High-Frequency 

High-frequency operations is designed to perform fixed-animations, active widgets and focused widgets. EKG should update synced with current framerate or a fixed chosen framerate. 

```cpp
size_t size {this->high_frequency_list.size()};
for (size_t it {}; it < size; it++) {
  ekg::at_t &at {this->high_frequency_list.at(it)};
  ekg::property_t &property {ekg::property(at)};

  if (property == ekg::property_t::not_found) {
    this->this->high_frequency_list.erase(this->high_frequency_list.begin() + it);
    size = this->high_frequency_list.size();
    continue;
  }

  ekg_core_abstract_todo(
    property.descriptor_at.flags,
    property.descriptor_at,
    ekg::ui::high_frequency(property, descriptor);
  );

  if (!property.widget.is_high_frequency) {
    this->this->high_frequency_list.erase(this->high_frequency_list.begin() + it);
    size = this->high_frequency_list.size();
  }
}
```

The high-frequency rate-update can be fixed if `ekg::update()` is called under a fixed-runtime.

#### CPU-Overall

All others parts of runtime, likely `ekg::runtime::poll_events` and callback-feature may affect performance but not dicussed here, because this topic is focused on how handle draw(s) in CPU-side and GPU-side.

### GPU

#### Font-Rendering

Registry glyph info on GPU and access it from the UTF-8 sequence (x 2 3 4) each `n` number is 1 byte, then the sampler is dynamic resized. Not yet done.

#### Capacity-System for VRAM

Likely, a `std::vector` cover the memory with capacity-system, where `n` is always larger than the current filled memory-block, this make possible inserting without re-allocating.

Possible implementation:
```cpp
template<typename t>
struct vector {
protected:
  t *p {};
  size_t current_size {};
  size_t current_capacity {};
  bool shuld_resize {};
public:
  vector() {
    this->current_capacity = 100;
    this->p = new t[this->current_capacity];
  }

  ~vector() {
    delete this->p;
  }

  void reserve(size_t size) {
    this->current_capacity = size;
    this->shuld_resize = true;
  }

  size_t size() {
    return this->current_size;
  }

  size_t capacity() {
    return this->current_capacity;
  }

  void check_memory_boundies() {
    if (
        this->shuld_resize
        ||
        this->current_size >= this->current_capacity
    ) {
      this->shuld_resize = false;
      this->current_capacity *= 2;
      size_t it {};

      t *p {new t[this->current_capacity]};
      memcpy(p, this->p, this->current_size);

      delete this->p;
      this->p = p;
    }
  }

  void push_back(const t &copy) {
    this->check_memory_boundies();
    this->p[this->current_size++] = copy;
  }

  t &emplace_back() {
    this->check_memory_boundies();
    return this->p[this->current_size++];
  }
};

vector<float> meow {};
```

That is it, a vector, the `t *p` is a block of memory, which increases if necessary, sadly the GPU is not cappable of do it, because we are limited to the simultaneosly limite of GPU.
For bypass it we can reserve a specific amount of VRAM and fill it, OpenGL is able to resize, because the driver handle the buffers and desfragment it for us, but Vulkan no.

For Vulkan we can enjoy of low-level mapping memory-buffer `t *p` pointing to a block of memory at VRAM, passing (correct-syncing) direct to PCIe, reducing the copy process at CPU-side and CPU-side cycles.

Topics:  
| - | [OpenGL](./performance.md#OpenGL)  
| - | [Vulkan](./performance.md#Vulkan)

##### OpenGL

For OpenGL (3, 4, ES3, WebGL2/ES2) we will cover the block of memory by allocating initianlly a capacity.

```cpp
virtual void pass_geometry_buffer(
  std::vector<float> &geometry_buffer
) {
  if (geometry_buffer.size() > this->gbuffer_capacity) {
    glBufferData(
      GL_ARRAY_BUFFER,
      sizeof(float) * this->gbuffer_capacity,
      geometry_buffer.data(),
      GL_STATC_DRAW
    );
  } else {
    glBufferSubData(
      GL_ARRAY_BUFFER,
      0,
      sizeof(float) * geometry_buffer.size(),
      geometry_buffer.data()
    );
  }

  /* check for vram usage limit */
  /* soon mapped buffers should be used instead this */
}
```

##### Vulkan

Not implemented yet.

## Conclusion

Dealing with CPU-cycles and GPU-memory is not a pain, as covered in this topic. Smart-caching, advanced usage of low-level Vulkan and the old OpenGL.
