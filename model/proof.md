# Memory-Safety

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