# Proofs

## Raw-Pointers Unsafety

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

## Safety-Descriptor Pool Query

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <functional>
#include <array>

struct at_t {
public:
    size_t id {};
    size_t index {};
};

struct meow_t { // descriptor
public:
    size_t id {};
    std::string text {};
public:    
    operator std::string() {
        return this->text;
    }
    
    bool operator == (meow_t &other) {
        return this->id == other.id;
    }

    bool operator != (meow_t &other) {
        return this->id != other.id;
    }
};

#define NOT_FOUND 34553453453

static std::vector<meow_t> meow_pool {};
static meow_t not_found_meow {.id = NOT_FOUND};

static std::array<std::function<void*(size_t)>, 1> query_function {
    [](size_t index) { 
        return index >= meow_pool.size() ? &not_found_meow : &meow_pool.at(index);
    }
};

enum type {
    MEOW = 0
};

template<typename t>
constexpr t &query(size_t index, size_t id) {
    return *static_cast<t*>(query_function[id](index));
}

class abstract {
public:
    at_t at_descriptor { .id = type::MEOW, .index = 0 };
};

int main() {
    meow_pool.emplace_back() = {.text = "MEOW-DESCRIPTOR"};
    abstract boo {};

    meow_t &slot0 {query<meow_t>(boo.at_descriptor.index, boo.at_descriptor.id)};
    if (slot0 != not_found_meow) {
        std::cout << "found: " << slot0.text << "\n";
    }

    meow_t &slot1 {query<meow_t>(1, type::MEOW)};
    if (slot1 == not_found_meow) {
        std::cout << "not found";
    }

    return 0;
}
```

## Renderable-Widget Safety

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
