---
language: javascript
type: function
tags:
  - snippet
  - javascript
  - function
status: tested
---

# JavaScript - Debounce Function

## Purpose
Delay calling a function until a specified time has passed without another call.

## Code

```javascript
function debounce(fn, delay = 250) {
  let timerId;

  return function (...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

## Usage

```javascript
const onSearchInput = debounce((event) => {
  console.log("Search:", event.target.value);
}, 300);

document.querySelector("#search").addEventListener("input", onSearchInput);
```

## Dependencies
- Modern browser or JavaScript runtime with `setTimeout`

## Notes
- This is a trailing-edge debounce.
- The function runs after calls stop for the configured delay.
