# jdui

The menu used by my Grand Blue and Find a Needle scripts, on its own so it can be dropped into other scripts. Everything is drawn with the Drawing API, so it works on external executors (written against Matcha). No images, about 40 KB.

- [Loading it](#loading-it)
- [Quick start](#quick-start)
- [How the menu works](#how-the-menu-works)
- [Tabs](#tabs)
- [Controls](#controls)
- [Changing controls later](#changing-controls-later)
- [Notifications](#notifications)
- [Menu settings](#menu-settings)
- [The built-in tabs](#the-built-in-tabs)
- [Icons](#icons)
- [Recipes](#recipes)
- [Limits](#limits)
- [Executor requirements](#executor-requirements)

## Loading it

```lua
local ui = loadstring(game:HttpGet("https://raw.githubusercontent.com/jdev-studio/jdui/refs/heads/main/JDUI"))()
```

Loading it returns the menu. It plays a short intro, then the menu shows up on screen. Running it again closes the old menu first, so reloading your script doesn't leave two menus on screen.

If the download can fail (no internet, GitHub down), wrap it so the rest of your script still runs:

```lua
local ok, ui = pcall(function()
	return loadstring(game:HttpGet("https://raw.githubusercontent.com/jdev-studio/jdui/refs/heads/main/JDUI"))()
end)
if not ok or type(ui) ~= "table" then
	warn("menu failed to load: " .. tostring(ui))
	ui = nil
end
```

## Quick start

```lua
local ui = loadstring(game:HttpGet("https://raw.githubusercontent.com/jdev-studio/jdui/refs/heads/main/JDUI"))()

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
- Each tab shows 4 controls per page. With more than 4, arrows and a page number appear in the bottom right.
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

`ui:AddTab("Main")` works too.

`tab:Select()` switches to that tab. The menu opens on Home unless you select something else, so call `:Select()` on the tab you want to open first.

Tabs are shown in the order of `ui.Tabs`. You can reorder it or leave tabs out:

```lua
ui.Tabs = { main, other, ui.Settings } -- hides Home, keeps Settings last
```

## Controls

Every control takes these options:

| Option | |
| --- | --- |
| `Title` | Bold first line |
| `Description` | Smaller second line |
| `Icon` | Icon in the badge on the left (each type has its own default) |
| `Callback` | Function called when the value changes, or when a button is clicked |

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

Click or drag along the track. The value shows above it.

### Dropdown

```lua
local d = tab:AddDropdown({ Title = "Mode", Options = { "Safe", "Fast", "Insane" }, Default = "Safe", Callback = function(v) end })
```

`Options` is required. `Default` falls back to the first option. `:SetValue` only accepts one of the options.

### Button

```lua
tab:AddButton({ Title = "Teleport", ButtonText = "Go", Callback = function() end })
```

`ButtonText` defaults to `"Run"` (up to 9 characters).

### Keybind

```lua
local k = tab:AddKeybind({ Title = "Weapon slot", Default = 0x31, Callback = function(vk) end })
```

Click the box, then press a key. The callback gets the key as a VK code (a number, e.g. `0x31` for 1, `0x46` for F). Escape cancels. The box shows "None" until a key is picked. `Default` is optional.

This only records the key. Checking whether it's pressed is up to your script, for example with `iskeypressed(k:GetValue())`.

### Label

```lua
local l = tab:AddLabel({ Title = "0 found", Description = "targets nearby", Icon = "info" })
```

Text only. Labels get the full width, so longer titles fit. `tab:AddLabel("Some text")` works too.

## Changing controls later

Every `Add...` call returns the control.

| | |
| --- | --- |
| `c:GetValue()` | Current value |
| `c:SetValue(value, silent)` | Changes the value. With `silent` as `true` the callback isn't called |
| `c:Set(value, silent)` | Same as `SetValue` |
| `c:SetText(text)` | Changes the title |
| `c:SetDescription(text)` | Changes the description |
| `c.Title`, `c.Description` | Can also be set directly |

Use `silent` when your script changes the value itself, e.g. a hotkey turning a feature off, so the callback doesn't run twice:

```lua
if iskeypressed(0x70) then -- F1
	farming = not farming
	farmToggle:Set(farming, true)
end
```

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
| `ui.Visible` | `true` / `false` to show or hide the menu from code |
| `ui.Effects` | Glow, dust and trail effects. `false` turns them off |
| `ui.EffectStrength` | How strong the effects are, `0.8` by default |
| `ui.ReducedMotion` | `true` skips animations (the intro, easing, sliding) |
| `ui.StartupSound` | `false` turns off the ping when the intro plays |
| `ui.InputGuard` | Optional function. While it returns `true` the menu ignores mouse clicks |
| `ui.Version` | JDUI version |
| `ui.Alive` | `false` after the menu has been closed |
| `ui:Destroy()` | Removes the menu and all its drawings |
| `ui:RequestClose()` | Opens the "Are you sure?" close prompt |

`InputGuard` is for scripts that click the mouse themselves. Without it, an auto-clicker could press menu buttons by accident:

```lua
ui.InputGuard = function()
	return autoClicking
end
```

To start with the menu hidden, set `ui.Visible = false` right after loading. Pressing the menu key opens it.

## The built-in tabs

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

## Recipes

**Saving settings between sessions** (needs `writefile` / `readfile`):

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

## Limits

- Text is cut off with `...` when it's too long: control titles at 28 characters (52 for labels), descriptions at 38 (66 for labels), tab names at 18.
- 4 controls per page and 5 tabs visible at a time; the rest are reached with the arrows.
- Only one JDUI menu can be open at a time. Loading a second one closes the first.

## Executor requirements

JDUI needs `Drawing.new` (Square, Line, Text), `iskeypressed`, `ismouse1pressed`, `game:HttpGet` and `loadstring`, plus `RunService.RenderStepped`. `isrbxactive` is used if it exists, so the menu ignores input while Roblox isn't focused. Written for Matcha.
