# jdui

The menu used by my Grand Blue and Find a Needle scripts, on its own so it can be dropped into other scripts. Everything is drawn with the Drawing API, so it works on external executors (written against Matcha).

## Loading it

```lua
local ui = loadstring(game:HttpGet("https://raw.githubusercontent.com/jdev-studio/jdui/refs/heads/main/JDUI"))()
```

It comes with a Home and a Settings tab (theme, menu key, test notification). Right Shift shows and hides the menu.

## Example

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

main:AddButton({ Title = "Say hi", ButtonText = "Run", Callback = function()
	ui:Notify({ Title = "Hi", Content = "Button pressed", Type = "success", Duration = 4 })
end })

main:AddLabel({ Title = "Status", Description = "idle", Icon = "info" })
```

## Reference

**Menu**

| | |
| --- | --- |
| `ui:AddTab({ Title, Icon })` | Adds a tab to the sidebar |
| `ui:Notify({ Title, Content, Type, Duration })` | Type is `"info"`, `"success"` or `"error"` |
| `ui:SetTheme(name)` | `"Purple"`, `"Green"`, `"Blue"` or `"Black"` |
| `ui:SetKeybind(vk)` | Menu key as a VK code, e.g. `0xA3` for Right Ctrl |
| `ui.Visible` | Set to show or hide the menu |
| `ui.Effects` | Glow and trail effects on or off |
| `ui:Destroy()` | Removes the menu |
| `ui.Home`, `ui.Settings` | The built-in tabs, so you can add to them |

**Controls** (all take `Title`, `Description`, `Icon` and `Callback`)

| | |
| --- | --- |
| `tab:AddToggle({ Default })` | Callback gets true / false |
| `tab:AddSlider({ Min, Max, Step, Default })` | Callback gets the number |
| `tab:AddDropdown({ Options, Default })` | Callback gets the chosen option |
| `tab:AddButton({ ButtonText })` | Callback runs on click |
| `tab:AddLabel({ Title, Description })` | Text only |

Every control has `:GetValue()` and `:SetValue(value, silent)`. Passing `silent` as true skips the callback.

Icons: `bolt`, `layers`, `shield`, `spark`, `power`, `sliders`, `palette`, `home`, `gear`, `script`, `info`, `check`.

Running it again closes the old menu first, so reloading a script doesn't leave two menus on screen.
