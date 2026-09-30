# Changelog

The current version is in `ui.Version`.

**Numbering:** each new feature or fix gets the next `1.0.x` number. If it takes more than one try to get right, the follow-ups get a letter: `1.0.1`, `1.0.1b`, `1.0.1c` and so on. The next feature moves on to `1.0.2`.

## 1.0.2 - 2026-09-30

- New logo in the top left of the menu: a sombrero with two maracas crossed behind it, drawn with lines like the other icons so it follows the theme colour.
- All icons are rounded: circles (power, gear, info) are smooth instead of 7 to 12-sided, and corners are softened.
- The open and close animations spell JDUI (they still said GBS).

## 1.0.1 - 2026-09-30

- Added `tab:AddKeybind` for your own key boxes (click it, press a key, the callback gets the key code). Escape cancels.
- Added `:SetText`, `:SetDescription` and `:Set` (short for `:SetValue`) on controls.
- Added `ui.InputGuard`: if it returns true the menu ignores mouse clicks, so a script that clicks by itself doesn't press menu buttons.

## 1.0.0 - 2026-09-30

- First release as its own library. Same menu as the Grand Blue and Find a Needle scripts: sidebar tabs, toggles, sliders, dropdowns, buttons, labels, notifications, four themes and a rebindable menu key, with the open/close animations and effects.
- Loading it returns the menu, so `loadstring(...)()` can be used straight away.
