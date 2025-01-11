# Project Rules

1- All user-programmer side features, must be no longer by-package namespace separated.
2- Only internal-objects and internal-features must be by-package namespace separated.

As example:

```c++
// ekg/ui/frame
namespace ekg {
  struct frame_t {
  public:
    // descriptor fields
  }
}

namespace ekg::ui {
  struct frame : public ekg::ui::abstract {
  public:
    // frame widget fields
  }
}
```

3- Non-OO features as descriptors, must end as a type, `*_t`.