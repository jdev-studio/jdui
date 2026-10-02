# jdui

The menu used by my Grand Blue and Find a Needle scripts, on its own so it can be dropped into other scripts. Everything is drawn with the Drawing API, so it works on external executors (written against Matcha). No images, about 45 KB.

- [Loading it](#loading-it)
- [Quick start](#quick-start)
- [How the menu works](#how-the-menu-works)
- [Tabs](#tabs)
- [Controls](#controls)
- [Changing controls later](#changing-controls-later)
- [Sizes and layout](#sizes-and-layout)
- [Saving settings](#saving-settings)
- [Notifications](#notifications)
- [Menu settings](#menu-settings)
- [The built in tabs](#the-built-in-tabs)
- [Icons](#icons)
- [Key codes](#key-codes)
- [Recipes](#recipes)
- [Troubleshooting](#troubleshooting)
- [Limits](#limits)
- [Executor requirements](#executor-requirements)

## Loading it

```lua
local ui = loadstring(game:HttpGet("https://raw.githubusercontent.com/jdev-studio/jdui/refs/heads/main/JDUI"))() or _G.JDUI
```

Loading it gives you the menu. Some executors (Matcha included) drop the value a loadstring returns, so JDUI also puts the menu in `_G.JDUI`; the `or _G.JDUI` covers both. It plays a short intro, then the menu shows up on screen. Running it again closes the old menu first, so reloading your script doesn't leave two menus on screen.

If the download can fail (no internet, GitHub down), wrap it so the rest of your script still runs:

```lua
local ok, ui = pcall(function()
	return loadstring(game:HttpGet("https://raw.githubusercontent.com/jdev-studio/jdui/refs/heads/main/JDUI"))() or _G.JDUI
end)
if not ok or type(ui) ~= "table" then
	warn("menu failed to load: " .. tostring(ui))
	ui = nil
end
```

## Quick start

```lua
local ui = loadstring(game:HttpGet("https://raw.githubusercontent.com/jdev-studio/jdui/refs/heads/main/JDUI"))() or _G.JDUI

local main = ui:AddTab({ Title = "Main", Icon = "bolt" })

main:AddToggle({ Title = "Auto farm", Description = "Hotkey: F1", Default = false, Callback = function(on)
	print("auto farm", on)
end })

main:AddSlider({ Title = "Speed", Min = 0, Max = 100, Step = 5, Default = 50, Callback = function(v)
	print("speed", v)
end })

main:AddDropdown({ Title = "Mode", Options = { "Safe", "Fast" }, Default = "Safe", Callback = function(v)
	print("mode", v)
end })

main:AddKeybind({ Title = "Weapon slot", Description = "Click, then press a key", Callback = function(vk)
	print("weapon key", vk)
end })

main:AddButton({ Title = "Say hi", ButtonText = "Run", Callback = function()
	ui:Notify({ Title = "Hi", Content = "Button pressed", Type = "success" })
end })

local status = main:AddLabel({ Title = "Status", Description = "idle", Icon = "info" })

main:Select()
```

## How the menu works

- **Right Shift** shows and hides the menu (changeable, see [Menu settings](#menu-settings)).
- Drag the window by its top bar. It scales down on small screens.
- The sidebar opens when you hover it and shows the tab names. Up to 5 tabs are visible at once; arrows appear to scroll through more.
- Controls fill each page from top to bottom: 4 per page at the default size, 7 with the Compact layout, 14 with compact half width toggles (see [Sizes and layout](#sizes-and-layout)). When they don't all fit, arrows and a page number appear in the bottom right.
- The **X** in the top right asks "Are you sure?" and then closes the menu for good (`ui:Destroy()`).
- Callbacks run in their own thread, so a slow callback doesn't freeze the menu. If a callback errors, the error shows up as a red notification instead of breaking the menu.

## Tabs

```lua
local tab = ui:AddTab({ Title = "Main", Icon = "bolt" })
```

| Option | Default | |
| --- | --- | --- |
| `Title` | `"Tab"` | Name in the sidebar and at the top of the page |
| `Icon` | `"script"` | See [Icons](#icons) |
| `Style` | none | Sizes for every control in this tab, see [Sizes and layout](#sizes-and-layout) |

`ui:AddTab("Main")` works too.

`tab:Select()` switches to that tab. The menu opens on Home unless you select something else, so call `:Select()` on the tab you want to open first. Selecting a tab replays its controls' entrance animation and scrolls the sidebar so the tab is in view.

A tab keeps its controls in `tab.Controls` (in the order they were added) and its current page in `tab.Page`.

### Order of the tabs

The sidebar draws `ui.Tabs` from top to bottom. Every `ui:AddTab` adds the new tab to the end of that list, and JDUI creates Home and Settings as soon as it loads, so without any changes the order is:

1. Home
2. Settings
3. your tabs, in the order you made them

To choose the order yourself, replace the list:

```lua
ui.Tabs = { main, other, ui.Settings } -- hides Home, keeps Settings last
main:Select()
```

You can change `ui.Tabs` at any time, even while the menu is open. The sidebar picks it up on the next frame.

**Moving one tab** without writing out the whole list:

```lua
for i, t in ipairs(ui.Tabs) do
	if t == other then table.remove(ui.Tabs, i) break end
end
table.insert(ui.Tabs, 1, other) -- 1 = top, leave the number out for the bottom
```

**Keeping Settings at the bottom** when you add tabs later (new tabs go after it otherwise):

```lua
for i, t in ipairs(ui.Tabs) do
	if t == ui.Settings then table.remove(ui.Tabs, i) break end
end
table.insert(ui.Tabs, ui.Settings)
```

### Hiding tabs

Leave a tab out of `ui.Tabs` and it's gone from the sidebar. It still exists with all its controls, so you can put it back later. There's no way to delete a tab outright, and you don't need one.

If the menu is currently on a tab you hide, call `:Select()` on one that's still in the list, or the page of the hidden tab stays on screen.

**A tab without an icon.** An icon is a list of lines, so an empty list draws nothing:

```lua
ui.Home.Icon = {}
local quiet = ui:AddTab({ Title = "Misc", Icon = {} })
```

The name still shows when the sidebar opens on hover.

**Renaming a tab** later: `tab.Title = "New name"`. Keep it to 18 characters; longer names aren't cut on the page title.

## Controls

Every control takes these options:

| Option | |
| --- | --- |
| `Title` | Bold first line |
| `Description` | Smaller second line |
| `Icon` | Icon in the badge on the left (each type has its own default) |
| `Callback` | Function called when the value changes, or when a button is clicked |
| `Style` | Sizes for just this control, see [Sizes and layout](#sizes-and-layout) |

Titles are kept up to 52 characters and descriptions up to 68. The default icons are `power` for toggles, `sliders` for sliders, `layers` for dropdowns, `bolt` for buttons, `key` for keybinds and `spark` for labels.

**How callbacks run.** A callback only runs when the value actually changes (setting a toggle that's already on doesn't call it again), or every time a button is clicked. Each call runs in its own thread, so a slow callback (one that waits or teleports) doesn't freeze the menu. If a callback errors, the menu stays up and shows a red "Script error" notification with the message.

### Toggle

```lua
local t = tab:AddToggle({ Title = "Auto farm", Default = false, Callback = function(on) end })
```

`Default` is `false` if left out. The callback gets `true` or `false`.

### Slider

```lua
local s = tab:AddSlider({ Title = "Speed", Min = 0, Max = 100, Step = 1, Default = 50, Callback = function(v) end })
```

| Option | Default | |
| --- | --- | --- |
| `Min` | `0` | |
| `Max` | `100` | Must be bigger than `Min` |
| `Step` | `1` | Values snap to this. Use `0.1` or `0.05` for decimals |
| `Default` | `Min` | |

Click or drag along the track. The value shows above it with up to 2 decimals (trailing zeros dropped, so `50` not `50.00`). Values from dragging or `:SetValue` are always snapped to `Step` and kept between `Min` and `Max`, and the callback only runs when the snapped value changes, so dragging a slider with `Step = 5` calls it once per step, not every frame.

### Dropdown

```lua
local d = tab:AddDropdown({ Title = "Mode", Options = { "Safe", "Fast", "Insane" }, Default = "Safe", Callback = function(v) end })
```

`Options` is required. `Default` falls back to the first option. `:SetValue` only accepts one of the options and raises an error for anything else.

The list shows 4 options at a time, with arrows at the bottom when there are more. Clicking outside the list closes it. Options are shown with `tostring`, so numbers work too.

To change the options later, replace the list and pick a value from the new one:

```lua
d.Options = { "Nearest", "Lowest HP", "Random" }
d:SetValue("Nearest", true)
```

### Button

```lua
tab:AddButton({ Title = "Teleport", ButtonText = "Go", Callback = function() end })
```

`ButtonText` defaults to `"Run"` (up to 9 characters).

### Keybind

```lua
local k = tab:AddKeybind({ Title = "Weapon slot", Default = 0x31, Callback = function(vk) end })
```

Click the box, then press a key. The callback gets the key as a VK code (a number, e.g. `0x31` for 1, `0x46` for F, see [Key codes](#key-codes)). Escape cancels. The box shows "None" until a key is picked. `Default` is optional.

Shift, Ctrl and Alt are recorded as their left or right versions (Left Shift is `0xA0`, Right Shift `0xA1` and so on), because those are the codes `iskeypressed` understands.

This only records the key. Checking whether it's pressed is up to your script, for example with `iskeypressed(k:GetValue())`.

### Label

```lua
local l = tab:AddLabel({ Title = "0 found", Description = "targets nearby", Icon = "info" })
```

Text only. Labels get the full width, so longer titles fit. `tab:AddLabel("Some text")` works too.

Labels have no callback. They're meant to be updated from your script with `:SetText` and `:SetDescription` (see [Changing controls later](#changing-controls-later) and the live status label in [Recipes](#recipes)).

## Changing controls later

Every `Add...` call returns the control.

| | |
| --- | --- |
| `c:GetValue()` | Current value |
| `c:SetValue(value, silent)` | Changes the value. With `silent` as `true` the callback isn't called |
| `c:Set(value, silent)` | Same as `SetValue` |
| `c:SetText(text)` | Changes the title |
| `c:SetDescription(text)` | Changes the description |
| `c:SetStyle(style)` | Changes this control's sizes, see [Sizes and layout](#sizes-and-layout) |
| `c.Title`, `c.Description` | Can also be set directly |

`SetText`, `SetDescription` and `SetStyle` return the control, so they can be chained: `c:SetText("On"):SetStyle({ Height = 36 })`.

Use `silent` when your script changes the value itself, e.g. a hotkey turning a feature off, so the callback doesn't run twice:

```lua
if iskeypressed(0x70) then -- F1
	farming = not farming
	farmToggle:Set(farming, true)
end
```

**Removing a control.** Take it out of its tab's `Controls` list. The page fills the gap on the next frame:

```lua
for i, c in ipairs(tab.Controls) do
	if c == oldButton then table.remove(tab.Controls, i) break end
end
```

**Showing a control only sometimes.** Remove it as above, and add it back with `table.insert(tab.Controls, position, control)` when you need it again. The control keeps its value and callback in between.

## Sizes and layout

All controls start at the sizes the menu always had. You can make them smaller (or bigger), change the borders, padding and corners, and put several controls side by side in one row.

### The quick way

```lua
ui:SetLayout("Compact")
```

This makes every control a slim single line row: 7 per page instead of 4. It works before or after you add controls and applies straight away. `ui:SetLayout("Default")` goes back.

### Where sizes can be set

Sizes are set with a style table. There are three places to put one, and the most specific one wins:

1. **One control:** `tab:AddToggle({ Title = "Fly", Style = { Height = 36 } })` or `control:SetStyle({ Height = 36 })`
2. **One tab:** `ui:AddTab({ Title = "Visuals", Style = { Height = 36 } })` or `tab:SetStyle({ Height = 36 })`
3. **The whole menu:** `ui:SetLayout({ Height = 36 })` or `ui:SetLayout("Compact")`

For each key JDUI checks the control first, then its tab, then the menu layout, then the built in default. A style only needs the keys you want to change.

### All style keys

Sizes are in menu pixels. The menu is 800 by 450 and scales down on small screens, so everything keeps its proportions.

| Key | Default | Compact | Allowed | What it changes |
| --- | --- | --- | --- | --- |
| `Height` | `65` | `36` | 20 to 296 | Height of the control's card |
| `Gap` | `12` | `6` | 0 to 60 | Space below a row, and between cards in the same row |
| `Width` | `1` | `1` | above 0, up to 1 | Share of the row the card takes. `0.5` puts two side by side, `1/3` puts three |
| `Padding` | `13` | `9` | 0 to 40 | Inner space between the card's edge and what's inside it (also the space between the icon and the text) |
| `Corner` | `12` | `9` | 0 to 30 | How round the card corners are. `0` is square |
| `Border` | `1` | `1` | 0 to 6 | Thickness of the card's outer border. `0` removes it |
| `IconSize` | `31` | `22` | 0 to 60 | Size of the icon badge on the left. `0` hides the icon and moves the text to the edge |
| `TitleSize` | `14` | `13` | 8 to 30 | Title text size |
| `DescriptionSize` | `11` | `11` | 8 to 24 | Description text size |
| `ShowDescription` | `true` | `true` | `true` / `false` | `false` always hides the description and centres the title |
| `ToggleWidth` | `46` | `34` | 20 to 120 | Toggle switch width |
| `ToggleHeight` | `24` | `18` | 10 to 40 | Toggle switch height. The knob grows with it |
| `SliderWidth` | `156` | `130` | 40 to 400 | Slider track length |
| `BoxWidth` | `165` | `140` | 60 to 400 | Dropdown and keybind box width. The dropdown list is as wide as the box (at least 120) |
| `BoxHeight` | `35` | `26` | 14 to 60 | Dropdown and keybind box height |
| `ButtonWidth` | `94` | `80` | 30 to 300 | Button width |
| `ButtonHeight` | `34` | `26` | 14 to 60 | Button height |

The current menu values are in `ui.Layout` (for example `ui.Layout.Height`).

### How rows and pages are filled

- Controls are placed in the order they were added. A row takes controls while their `Width`s add up to 1 or less. A control that doesn't fit starts the next row.
- A row is as tall as its tallest control, and every card in the row stretches to that height. The space below the row is the biggest `Gap` in it.
- A page has 296 pixels of room (between the title line and the page arrows). A row that doesn't fit goes on the next page.

So at the default size 4 rows fit (4 × 65 + 3 × 12 = 296), and at the Compact size 7 fit (7 × 36 + 6 × 6 = 288).

### What adjusts by itself

- **Descriptions** are drawn only when the row is tall enough: at least `TitleSize + DescriptionSize + 22` (47 by default). Shorter rows show just the title, centred. This is why Compact rows have no description.
- **Slider values** sit above the track when the row is at least 50 tall, and to the left of the track in shorter rows.
- **Switches, boxes and buttons** are never taller than the row minus 4, so they always fit inside their card.
- **Long text** is cut with `...` to fit the space left of the switch, box or slider. Narrow cards show shorter titles, and toggles get more room than dropdowns because the switch is smaller.

### Examples

**Small toggles only.** Keep the other controls normal and pass one shared style to the toggles:

```lua
local small = { Height = 36, IconSize = 22, TitleSize = 13, ToggleWidth = 34, ToggleHeight = 18 }

tab:AddToggle({ Title = "Fly", Style = small })
tab:AddToggle({ Title = "Noclip", Style = small })
tab:AddSlider({ Title = "Speed", Min = 0, Max = 100 }) -- normal size
```

Each control copies the style table when it's made, so changing `small` afterwards doesn't affect controls that already exist. Use `:SetStyle` for that.

**Two toggles per row.**

```lua
ui:SetLayout("Compact")
local half = { Width = 0.5 }
for _, name in ipairs({ "Fly", "Noclip", "ESP", "Tracers", "Fullbright", "No fog" }) do
	tab:AddToggle({ Title = name, Style = half })
end
tab:AddSlider({ Title = "Walk speed", Min = 16, Max = 100 }) -- full width row under them
```

Toggles, buttons and labels work well at half width. Sliders, dropdowns and keybinds need room for their track or box, so leave them at full width or make the track or box narrower too (`SliderWidth`, `BoxWidth`).

**One compact tab, the rest normal.**

```lua
local visuals = ui:AddTab({ Title = "Visuals", Icon = "spark", Style = { Height = 36, Gap = 6, IconSize = 22 } })
```

**One bigger control in a compact menu.**

```lua
ui:SetLayout("Compact")
tab:AddDropdown({ Title = "Target", Description = "Who to lock on to", Options = { "Nearest", "Lowest HP" }, Style = { Height = 65, BoxWidth = 165, BoxHeight = 35 } })
```

**Start from Compact and tweak it.** A name resets the layout to that preset. A table changes only the keys in it and keeps the rest:

```lua
ui:SetLayout("Compact")
ui:SetLayout({ Gap = 4, Border = 0, Corner = 6 })
```

**Flat look with no borders or icons.**

```lua
ui:SetLayout({ Border = 0, Corner = 4, IconSize = 0 })
```

**Let players pick the size** in the Settings tab:

```lua
ui.Settings:AddDropdown({ Title = "Menu size", Options = { "Default", "Compact" }, Callback = function(v)
	ui:SetLayout(v)
end })
```

**Changing values directly.** `ui.Layout`, `tab.Style` and `control.Style` are plain tables and are read every frame, so `ui.Layout.Height = 50` works. To remove one override, set it to `nil`: `control.Style.Height = nil`.

### Mistakes are reported

`SetLayout`, `SetStyle`, `AddTab` and every `Add...` check the style you pass. A misspelled key (`"Unknown style key 'Hieght'"`), a wrong type (`Height = "big"`), a `Width` of 0 or more than 1, or an unknown layout name raises an error, so the problem shows up right away instead of being silently ignored. Numbers outside the allowed range are clamped to it. That includes values you put straight into `ui.Layout` or `control.Style`, which skip the check.

## Saving settings

One line makes the menu remember everything the player sets:

```lua
ui:SetConfig("myscript")
```

From then on, every toggle, slider, dropdown and keybind box is saved to `myscript.json` in the executor's workspace folder (on Matcha that's `C:\matcha\workspace`), along with the theme and the menu key. Next time the script runs, the same line loads them back.

- **When it saves:** half a second after the last change, so dragging a slider writes the file once, not every frame. It also saves when the menu is closed.
- **When it loads:** straight away for controls that already exist, and for controls you add later, as soon as they're made. So you can call `SetConfig` before or after adding your controls.
- **Callbacks run on load.** A saved toggle that was on calls its callback with `true` when it's restored, so your feature starts the same way as if the player had clicked it. If you don't want that for a feature, check a flag in the callback.
- **Buttons and labels** aren't saved, since they have no value.
- **Saved values that no longer fit** are skipped: a dropdown option that was removed, or a keybind on a control that's gone. Slider values are snapped to the slider's current `Min`, `Max` and `Step`.

**How controls are named in the file.** Each control is saved under `Tab/Title`, for example `Main/Auto farm`. If you rename a control or its tab, its old value isn't found any more. To keep a value across renames, or when two controls in a tab share a title, give the control a fixed `Flag`:

```lua
main:AddToggle({ Title = "Auto farm", Flag = "autoFarm", Callback = function(on) end })
```

| | |
| --- | --- |
| `ui:SetConfig(name)` | Turns saving on with `name.json` (or `name` as is if it already ends in `.json`) and loads what's in it |
| `ui:SaveConfig()` | Writes the file right now |
| `ui.ConfigFile` | The file in use, `nil` until `SetConfig` is called |
| `Flag` option | Fixed name for a control in the file, instead of `Tab/Title` |

Saving needs `writefile` and `readfile`. Without them the menu works normally and just doesn't save.

## Notifications

```lua
ui:Notify({ Title = "Done", Content = "Caught a fish", Type = "success", Duration = 5 })
ui:Notify("Done", "Caught a fish") -- short form
```

| Option | Default | |
| --- | --- | --- |
| `Title` | `"JDUI"` | Up to 32 characters |
| `Content` | `""` | Up to 48 characters |
| `Type` | `"info"` | `"info"`, `"success"` (green) or `"error"` (red) |
| `Duration` | `4` | Seconds, 1 to 20 |

They slide in at the top left, show a progress bar and stack up to 4 at a time. They show even while the menu is hidden.

## Menu settings

| | |
| --- | --- |
| `ui:SetKeybind(vk)` | Menu key as a VK code, e.g. `0xA3` for Right Ctrl, `0x23` for End. Escape isn't allowed |
| `ui.Keybind` | Current menu key |
| `ui:SetTheme(name)` | `"Purple"`, `"Green"`, `"Blue"` or `"Black"`. Fades to the new colours. Returns `false` for an unknown name |
| `ui.Theme` | Current theme name |
| `ui:SetLayout(name or style)` | `"Default"` or `"Compact"` resets every size to that preset. A style table changes just those keys. See [Sizes and layout](#sizes-and-layout) |
| `ui.Layout` | Current menu sizes |
| `ui.Visible` | `true` / `false` to show or hide the menu from code |
| `ui.Effects` | Glow, dust and trail effects. `false` turns them off |
| `ui.EffectStrength` | How strong the effects are, `0.8` by default |
| `ui.ReducedMotion` | `true` skips animations (the intro, easing, sliding) |
| `ui.StartupSound` | `false` turns off the ping when the intro plays |
| `ui.InputGuard` | Optional function. While it returns `true` the menu ignores mouse clicks |
| `ui.CaptureInput` | `true` by default: while the cursor is over the menu, clicks only go to the menu and don't reach the game behind it. `false` turns that off |
| `ui.CaptureGuard` | Optional function. While it returns `false` the game keeps getting clicks even over the menu, e.g. while your script is clicking by itself |
| `ui.Version` | JDUI version |
| `ui.Alive` | `false` after the menu has been closed |
| `ui:Destroy()` | Removes the menu and all its drawings |
| `ui:RequestClose()` | Opens the "Are you sure?" close prompt |

**Clicks over the menu.** On Matcha the menu is drawn on top of the game, so a click on it would normally also reach the game (your character swings or casts). JDUI turns game input off with `setrobloxinput` while the cursor is over the menu, and back on as soon as it leaves, the menu is hidden or the script is closed. Executors without `setrobloxinput` just skip this.

If your script clicks by itself, keep game input on while it does, or its clicks are lost whenever they land under the menu:

```lua
ui.CaptureGuard = function()
	return not autoFarming -- false = let the game have the clicks
end
```

While `InputGuard` returns `true` the game keeps its input too.

`InputGuard` is for scripts that click the mouse themselves. Without it, an auto clicker could press menu buttons by accident:

```lua
ui.InputGuard = function()
	return autoClicking
end
```

To start with the menu hidden, set `ui.Visible = false` right after loading. Pressing the menu key opens it.

A few details about input:

- The menu key does nothing during the intro, while the close prompt is open, or while a keybind box is waiting for a key.
- Clicks and the menu key are ignored while Roblox isn't the focused window (on executors with `isrbxactive`), and game input is always given back then.
- Game input is also kept off while you drag the window or a slider, even if the cursor slips off the menu, so a drag never ends in a swing.
- Matcha's own UI uses `setrobloxinput` the same way. If two menus that both do this are open at once, the last one to change it wins.

## The built in tabs

- `ui.Home` has a "Welcome." label.
- `ui.Settings` has a theme dropdown, the menu keybind, and a "Test notification" button.

Add your own controls to them like any other tab:

```lua
ui.Settings:AddLabel({ Title = "Version 1.2", Description = "My script", Icon = "info" })
```

To remove one of the defaults, take it out of the tab's `Controls` list:

```lua
for i = #ui.Settings.Controls, 1, -1 do
	if ui.Settings.Controls[i].Title == "Test notification" then
		table.remove(ui.Settings.Controls, i)
	end
end
```

## Icons

`brand` (the sombrero logo), `bolt`, `layers`, `shield`, `spark`, `power`, `sliders`, `palette`, `home`, `gear`, `script`, `key`, `info`, `check`, `close`, `up`, `down`, `left`, `right`.

Icons are line drawings on a 20 by 20 grid, so they follow the theme colour. You can pass your own as a list of lines, each `{ x1, y1, x2, y2 }`:

```lua
local arrow = { { 4, 10, 16, 10 }, { 11, 5, 16, 10 }, { 16, 10, 11, 15 } }
tab:AddButton({ Title = "Next", Icon = arrow })
```

## Key codes

Keybind boxes, `ui:SetKeybind` and `iskeypressed` all use Windows VK codes. The common ones:

| Keys | Codes |
| --- | --- |
| `0` to `9` | `0x30` to `0x39` |
| `A` to `Z` | `0x41` to `0x5A` |
| `F1` to `F12` | `0x70` to `0x7B` |
| Left / Right Shift | `0xA0` / `0xA1` |
| Left / Right Ctrl | `0xA2` / `0xA3` |
| Left / Right Alt | `0xA4` / `0xA5` |
| Space, Tab, Enter | `0x20`, `0x09`, `0x0D` |
| Insert, Delete | `0x2D`, `0x2E` |
| Home, End | `0x24`, `0x23` |
| Page Up, Page Down | `0x21`, `0x22` |
| Arrow keys (left, up, right, down) | `0x25`, `0x26`, `0x27`, `0x28` |

Escape (`0x1B`) can't be the menu key, because it's what cancels a keybind box.

## Recipes

**Saving extra values of your own** next to the menu's (for things that aren't controls). For the controls themselves, `ui:SetConfig` already does this; see [Saving settings](#saving-settings). Needs `writefile` / `readfile`:

```lua
local FILE = "myscript_settings.json"
local HttpService = game:GetService("HttpService")
local saved = {}
pcall(function() saved = HttpService:JSONDecode(readfile(FILE)) end)
local function save() pcall(writefile, FILE, HttpService:JSONEncode(saved)) end

tab:AddToggle({ Title = "Auto farm", Default = saved.autoFarm == true, Callback = function(on)
	saved.autoFarm = on
	save()
end })

if saved.theme then ui:SetTheme(saved.theme) end
if saved.menuKey then ui:SetKeybind(saved.menuKey) end
```

**Saving the theme and menu key when they change**:

```lua
local setTheme, setKeybind = ui.SetTheme, ui.SetKeybind
ui.SetTheme = function(self, name)
	local ok = setTheme(self, name)
	saved.theme = self.Theme
	save()
	return ok
end
ui.SetKeybind = function(self, vk)
	setKeybind(self, vk)
	saved.menuKey = vk
	save()
end
```

**A live status label**:

```lua
local status = tab:AddLabel({ Title = "Idle", Icon = "info" })
game:GetService("RunService").Heartbeat:Connect(function()
	status:SetText(farming and "Farming" or "Idle")
	status:SetDescription(string.format("%d collected", collected))
end)
```

**Cleaning up your script when the menu is closed**:

```lua
local destroy = ui.Destroy
ui.Destroy = function(self)
	destroy(self)
	stopEverything()
end
```

## Troubleshooting

**`ui` is nil after loading.** Your executor dropped the value `loadstring(...)()` returns. Add `or _G.JDUI` to the end of the line, as in [Loading it](#loading-it).

**Nothing shows up.** The intro takes about 3 seconds. After that, check `ui.Visible` and press the menu key (Right Shift unless you changed it). The console prints "UI loaded. Right Shift toggles the menu." when it starts.

**The menu vanished and the console says "UI stopped: ...".** Something errored while the menu was being drawn, so it closed itself cleanly instead of freezing. The message says what. Usually it's a bad value put straight into a control or style table; report it if not.

**A red "Script error" notification.** One of your callbacks errored. The notification shows the error; the menu keeps working.

**Clicking the menu still clicks the game.** Your executor has no `setrobloxinput`, or `ui.CaptureInput` is `false`, or `ui.CaptureGuard` returns `false`, or `ui.InputGuard` returns `true` at that moment.

**My script's own clicks get lost.** They're landing under the menu while it has game input switched off. Return `false` from `ui.CaptureGuard` while your script is clicking (see [Menu settings](#menu-settings)).

**My script presses menu buttons by itself.** Set `ui.InputGuard` to return `true` while your script holds the mouse.

**"Dropdown value must be one of its Options".** `:SetValue` (or `Default`) got something that isn't in `Options`. Check spelling and capitals.

**"Unknown style key ..." or another style error.** See [Mistakes are reported](#mistakes-are-reported).

**Two menus on screen.** Can't happen with JDUI: loading it again closes the old one first. If you see an old style menu next to it, that's a different library left over from an earlier version of your script.

## Limits

- Text is cut off with `...` when it doesn't fit its card. Titles are stored up to 52 characters and descriptions up to 68. At the default size a full width label shows about that much (52 and 66), and other controls show less depending on how wide their switch, box or slider is. Tab names are cut at 18.
- As many rows per page as fit in 296 pixels (4 at the default size), and 5 tabs visible at a time. The rest are reached with the arrows.
- Only one JDUI menu can be open at a time. Loading a second one closes the first.

## Executor requirements

JDUI needs `Drawing.new` (Square, Line, Text), `iskeypressed`, `ismouse1pressed`, `game:HttpGet` and `loadstring`, plus `RunService.RenderStepped`. `isrbxactive` is used if it exists, so the menu ignores input while Roblox isn't focused. Written for Matcha.
