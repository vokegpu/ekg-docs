# Input-Binding Tag Style

## Preface 🐄

EKG needs standard a way for describing binding tag(s).

🐈

---

## Fundamentals

### Case

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
