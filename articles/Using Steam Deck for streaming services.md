Since I acquired my Steam Deck, I couldn't shake the feeling that something is missing from it. It appears to be the ideal solution for utilizing streaming services while traveling. However, it lacks any dedicated streaming app. While one can resort to using a browser, the only reasonable way to do so is by utilizing the touchscreen, which, to me, interrupts the handheld experience.

To prove my point, I attempted to develop a Chrome extension that would enable you to navigate YouTube entirely using Steam Deck controller. Initially, I thought I would need to find a method to read controller inputs; however, it turned out that Steam Deck controls are essentially just mapped keyboard keys.

<img src="https://cms.filipizydorczyk.pl/api/v1/media/Steamdeck layout.png">

So, the only thing I had to do was create "mappings" for each key, and then I could simply use standard JavaScript keyboard events.

```js
const LEFT_JOYSTICK_UP = "ArrowUp";
const LEFT_JOYSTICK_DOWN = "ArrowDown";
const LEFT_JOYSTICK_RIGHT = "ArrowRight";
const LEFT_JOYSTICK_LEFT = "ArrowLeft";

// Ive added all keys in steamdeck some of them are responsible for mouse so I left them empty
const RIGHT_JOYSTICK_UP = "";
const RIGHT_JOYSTICK_DOWN = "";
const RIGHT_JOYSTICK_RIGHT = "";
const RIGHT_JOYSTICK_LEFT = "";

const LEFT_TRACK_PAD = "";
const RIGHT_TRACK_PAD = "";

const DIRECTIONAL_PAD_UP = "ArrowUp";
const DIRECTIONAL_PAD_DOWN = "ArrowDown";
const DIRECTIONAL_PAD_RIGHT = "ArrowRight";
const DIRECTIONAL_PAD_LEFT = "ArrowLeft";

const L1 = "Control";
const L2 = "";
const L4 = "Shift";
const L5 = "";

const R1 = "Alt";
const R2 = "";
const R4 = "PageUp";
const R5 = "PageDown";

const Y = " ";
const B = "Escape";
const X = ""; // Its "show keyboard", no action for this key at all in browser
const A = "Enter";

const ACTION_KEY = "Escape";
const TAB_KEY = "Tab";
```

Whenever there was an event from a specific key, I would handle it by controlling the DOM and performing actions on various parts of the website.

```js

document.addEventListener("keydown", (event) => { console.log(event.key) });

```

This is how it looked like

<img src="https://cms.filipizydorczyk.pl/api/v1/media/Steamdeck YT.gif">

There are some issues, though:
1. Sometimes, the same actions would require different handlers on different parts of the website.
2. It's very sensitive to website updates. If the structure of HTML changes, then it would be possible that our extension will stop working correctly.
3. If I wanted to use multiple streaming platforms, each one would require a new extension.

So even though it's a cool proof of concept, we need something more than that. At the moment, the only way I see is to use Android TV apps with some kind of emulation/translation layer and create mappings from Steam Deck controls to TV remote controls. This would undoubtedly provide a much smoother experience, and we would have a standardized set of inputs to map to... However, I am not aware of an easy and reliable way to run Android apps on Linux, which is what Steam Deck really is.