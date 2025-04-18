# Raw-Pointers Unsafety Proof

```cpp
#include <iostream>
#include <vector>
#include <string>

struct meow_t {
public:
    std::string text {};
public:    
    operator std::string() {
        return this->text;
    }
};

int main() {
    meow_t *p_meow {new meow_t {.text = "meow, meow, meow, meow"}};
    std::vector<meow_t*> meow {};
    meow.emplace_back() = p_meow;
    
    delete p_meow;
    p_meow = nullptr;
    
    if (meow.at(0) != nullptr) {
        std::cout << "meow = " << (std::string)(*meow.at(0)); // output: ???
    }

    // UB

    return 0;
}
```

# Renderable-Widget Proof

```c++
#include <iostream>
#include <vector>
#include <memory>
#include <map>

struct abstract_t {
public:
  const char *p_tag {};
};

void add_to_list(
  std::vector<std::unique_ptr<abstract_t>> *p_list,
  abstract_t *p_widget
) {
  p_list->emplace_back(
    std::unique_ptr<abstract_t>(p_widget)
  );
}

int main() {
  std::vector<std::unique_ptr<abstract_t>> loaded_abstract_list {};
  abstract_t *p_raw {};
    
  {
    add_to_list(
      &loaded_abstract_list,
      (p_raw = new abstract_t {.p_tag = "meow"})
    );
  }
    
  std::cout << loaded_abstract_list.at(0)->p_tag << std::endl; // assert meow
    
  for (std::unique_ptr<abstract_t> &p_widget : loaded_abstract_list) {
    std::cout << p_widget->p_tag << std::endl; // assert meow
  }

  std::cout << p_raw->p_tag << std::endl; // assert meow
  loaded_abstract_list.clear();
    
  std::cout << p_raw->p_tag << std::endl; // assert segmentation fault
    
  return 0;
}
```

# Safe-Instance Creation

```c++
#include <iostream>
#include <memory>
#include <vector>

struct abstract { const char *p_tag {}; virtual void meow() {};};
struct meow : public abstract {};

template<typename t>
t *new_widget_instance(
    std::vector<std::unique_ptr<abstract>> *p_core
) {
  return dynamic_cast<t*>(
    p_core->emplace_back(
      std::unique_ptr<abstract>(
        dynamic_cast<abstract*>(new t {})
      )
    ).get()
  );
}

int main() {
    std::vector<std::unique_ptr<abstract>> core {};
    abstract *p_meow {new_widget_instance<meow>(&core)};
    p_meow->p_tag = "meow";
    std::cout << p_meow->p_tag << std::endl; // assert meow
    return 0;
}
```
