# Overview

- [Runtime](/ekg-docs/task/#runtime)

- [Task](/ekg-docs/task/#task)

# Runtime

The event system and handler service exist for two different purposes, the handler is a task manager that works with callback events (tasks) and executes the main tasks of the EKG runtime core.
The event system exists to release UI events. 

# Task

The structure of a [task](/ekg-docs/handler-service/#task) is simple.

```cpp
ekg::cpu::event task {
    .p_tag = "task event cat 1",
    .p_callback = nullptr,
    .function = [](void *p_callback) {};
};
```

For dispatching a [task](/ekg-docs/handler-service/#task) is simple, reference or copy `ekg::core->service_handler.generate()`

```cpp
auto &task = ekg::core->service_handler.generate();
task.p_tag = "cat";
task.p_callback = nullptr;
task.function = [](void *p_callback) {};

// It is already dispatched :)!
```