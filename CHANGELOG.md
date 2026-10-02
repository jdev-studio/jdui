# Changelog

The current version is in `ui.Version`.

**Numbering:** each new feature or fix gets the next `1.0.x` number. If it takes more than one try to get right, the follow ups get a letter: `1.0.1`, `1.0.1b`, `1.0.1c` and so on. The next feature moves on to `1.0.2`.

## 1.0.5 - 2026-10-02

- Settings save themselves. `ui:SetConfig("myscript")` saves every toggle, slider, dropdown and keybind box, plus the theme and menu key, to `myscript.json`, and loads them back next time. It saves half a second after a change and when the menu closes. Callbacks run when values are restored.
- New `Flag` option on controls to give them a fixed name in the file (otherwise `Tab/Title`), plus `ui:SaveConfig()` and `ui.ConfigFile`.
- Docs: new Saving settings section.

## 1.0.4 - 2026-10-02

- Clicking the menu no longer clicks the game behind it. While the cursor is over the menu (or you're dragging it or a slider), game input is turned off with `setrobloxinput`, and turned back on when the cursor leaves, the menu is hidden or closed, or Roblox isn't focused. Matcha's own UI does the same.
- New `ui.CaptureInput` (on by default) and `ui.CaptureGuard` (return `false` to let the game keep its input, e.g. while your script is clicking). While `ui.InputGuard` returns `true` the game keeps its input as well.
- Docs: new sections on the order of tabs, hiding tabs and tab icons, removing and showing controls, changing dropdown options, how callbacks run, key codes and troubleshooting, plus more detail on sliders, keybinds and input.

## 1.0.3 - 2026-10-01

Controls can now be resized. Thanks to the person on Discord who asked for smaller toggles and sliders. Everything is documented in [Sizes and layout](README.md#sizes-and-layout).

**New**

- `ui:SetLayout("Compact")` makes every control a slim single line row, so 7 fit on a page instead of 4. `ui:SetLayout("Default")` goes back.
- `ui:SetLayout({ ... })` changes individual sizes for the whole menu, on top of whichever preset is active. The current values are in `ui.Layout`.
- `Style = { ... }` option on `AddTab` and on every `Add...` control, plus `tab:SetStyle()` and `control:SetStyle()`. A control's style beats its tab's, and a tab's beats the menu layout.
- 17 style keys: `Height`, `Gap`, `Width`, `Padding`, `Corner`, `Border`, `IconSize`, `TitleSize`, `DescriptionSize`, `ShowDescription`, `ToggleWidth`, `ToggleHeight`, `SliderWidth`, `BoxWidth`, `BoxHeight`, `ButtonWidth`, `ButtonHeight`.
- Controls can sit side by side: `Width = 0.5` puts two in a row and `Width = 1/3` puts three.
- Misspelled style keys, wrong types and bad `Width` values raise a clear error. Numbers out of range are clamped instead of breaking the menu.

**Changed**

- Pages are filled by height instead of "4 per page". At the default size that's still 4.
- Titles and descriptions are cut to fit the space each card actually has, so toggles and buttons show more of a long title than before (toggle titles used to stop at 28 characters).
- Rows too short for two lines show just the title, centred. Short slider rows show the value to the left of the track.
- The dropdown list is as wide as its box.
- Buttons sit 1 pixel lower because of how the new layout rounds positions. You won't notice it.

**Unchanged:** with no style set, every control is drawn at the same size and position as in 1.0.2b, so scripts already using JDUI look the same.

## 1.0.2b - 2026-09-30

- The menu is also stored in `_G.JDUI`. Matcha drops the value a loadstring returns, so `loadstring(...)()` came back empty there. Use `loadstring(...)() or _G.JDUI`.

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
