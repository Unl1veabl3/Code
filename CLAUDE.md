# CLAUDE.md — AI Assistant Guide for This Repository

## Repository Overview

This is a **Roblox Lua scripting toolkit** consisting of two primary open-source libraries and two obfuscated scripts. The code runs inside the Roblox game engine, typically executed via exploit executors or the Roblox console. There is no build system, package manager, or test runner — scripts are loaded directly into the Roblox environment at runtime.

---

## File Structure

```
/
├── lib.lua          # Orion UI Library — GUI framework for Roblox scripts (~1,700 lines)
├── spy.lua          # SimpleSpy — Remote event/function logger and spy tool (~2,500 lines)
├── Sky.lua          # Obfuscated script (WeAreDevs obfuscator v1.0.0, ~92 KB)
├── Applefarm.lua    # Obfuscated script (WeAreDevs obfuscator v1.0.0, ~134 KB)
└── CLAUDE.md        # This file
```

**Sky.lua** and **Applefarm.lua** are single-line obfuscated files. Do not attempt to modify, deobfuscate, or augment them.

---

## lib.lua — Orion UI Library

### Purpose
A GUI framework that creates draggable, themeable windows inside Roblox games. It is used as a dependency loaded by other scripts via `loadstring(game:HttpGet(...))`.

### Core Object: `OrionLib`

```lua
local OrionLib = {
    Elements     = {},   -- all registered GUI elements
    ThemeObjects = {},   -- objects bound to theme colors
    Connections  = {},   -- all RBXScriptConnections for cleanup
    Flags        = {},   -- keyed element state (for save/load)
    Themes       = { Default = { Main, Second, Stroke, Divider, Text, TextDark } },
    SelectedTheme = "Default",
    Folder       = nil,  -- filesystem folder name for config persistence
    SaveCfg      = false -- enable config auto-save
}
```

### Public API

```lua
-- Initialize library (must be called first)
OrionLib:Init()

-- Create a window; returns a Window object
local Window = OrionLib:MakeWindow({
    Name         = "Title",
    HidePremium  = false,
    SaveConfig   = false,
    ConfigFolder = "OrionTest",
    IntroEnabled = true,
    IntroText    = "...",
    IntroIcon    = "rbxassetid://...",
    Icon         = "rbxassetid://...",
    CloseCallback = function() end,
})

-- Show a notification
OrionLib:MakeNotification({
    Name     = "Title",
    Content  = "Body text",
    Image    = "rbxassetid://...",
    Time     = 5,
})

-- Destroy all GUI and disconnect all connections
OrionLib:Destroy()
```

### Window → Tab → Section → Element hierarchy

```lua
local Tab = Window:MakeTab({ Name = "Tab", Icon = "rbxassetid://...", PremiumOnly = false })

Tab:AddSection({ Name = "Section" })

-- Available element methods on Tab (or section reference):
Tab:AddButton({ Name = "...", Callback = function() end })
Tab:AddToggle({ Name = "...", Default = false, Save = false, Flag = "flagKey", Callback = function(v) end })
Tab:AddSlider({ Name = "...", Min = 0, Max = 100, Default = 50, Color = Color3, Increment = 1, ValueName = "units", Callback = function(v) end })
Tab:AddTextBox({ Name = "...", Default = "", TextDisappear = false, Callback = function(v) end })
Tab:AddDropdown({ Name = "...", Default = "Option", Options = {"A","B"}, Callback = function(v) end })
Tab:AddColorpicker({ Name = "...", Default = Color3.fromRGB(255,0,0), Callback = function(c) end })
Tab:AddLabel("text")
Tab:AddParagraph("Title", "Body")
Tab:AddBind({ Name = "...", Default = Enum.KeyCode.E, Hold = false, Callback = function() end })
```

### Key Internal Patterns

- **`CreateElement(ClassName, Properties)`** — wrapper around `Instance.new` with bulk property assignment.
- **`MakeElement(ClassName, ...)`** — creates common composite UI objects (Frame, Button, etc.) with defaults.
- **`AddThemeObject(GuiObject, ThemeColorKey)`** — registers an object to automatically recolor on theme change.
- **`AddConnection(Signal, Function)`** — wraps `:Connect()` and stores the connection in `OrionLib.Connections` for cleanup.
- **Config persistence** — uses Roblox filesystem functions `isfile()`, `readfile()`, `writefile()` to store JSON configs per `GameId` inside the configured `Folder`.

### Theme System

Themes are defined as tables of `Color3` values under `OrionLib.Themes`. The `Default` theme keys are:

| Key        | Description              |
|------------|--------------------------|
| `Main`     | Primary background color |
| `Second`   | Secondary background     |
| `Stroke`   | Border/outline color     |
| `Divider`  | Divider line color       |
| `Text`     | Primary text color       |
| `TextDark` | Secondary/muted text     |

---

## spy.lua — SimpleSpy

### Purpose
A remote event/function interception and logging tool. It hooks into Roblox's namecall mechanism to capture all `RemoteEvent:FireServer()` and `RemoteFunction:InvokeServer()` calls, then displays them in a GUI panel with generated Lua code for replay.

### Configuration (top of file)

```lua
local realconfigs = {
    logcheckcaller   = false,  -- log calls from checking scripts
    autoblock        = false,  -- auto-block repeated remotes
    funcEnabled      = true,   -- enable RemoteFunction logging
    advancedinfo     = false,  -- show extra debug info
    supersecretdevtoggle = false
}

local DISCORD_WEBHOOK_URL = "" -- set to send logs to a Discord webhook
```

### Key Features
- Hooks `__namecall` metamethod on the Roblox game object to intercept remote calls.
- Left panel: scrolling log of remote calls with timestamps.
- Right panel: generated Lua code viewer for the selected call (supports copy-to-clipboard).
- Blacklist/whitelist system for filtering specific remote names.
- Discord webhook integration: sends log data as JSON to the configured URL.
- Mobile-aware UI with touch input support.

### Executor API Compatibility
spy.lua probes for multiple executor APIs and uses whichever is available:

```lua
get_thread_identity / getidentity / getthreadidentity
set_thread_identity / setidentity
hookmetamethod / hookfunction
islclosure / is_l_closure
getupvalues / debug.getupvalues
getconstants / debug.getconstants
setclipboard / toclipboard / set_clipboard
request / syn.request
newcclosure / clonefunction / cloneref
```

Graceful fallbacks are provided via `blankfunction` when APIs are absent.

---

## Roblox Execution Environment

These scripts run inside the Roblox engine. Key globals available:

| Global/API            | Description                                 |
|-----------------------|---------------------------------------------|
| `game`                | Root DataModel; access services via `:GetService()` |
| `game:GetService()`   | Service locator (Players, RunService, etc.) |
| `game.CoreGui`        | Top-level GUI container                     |
| `getgenv()`           | Executor global environment table           |
| `task.spawn(fn)`      | Non-blocking coroutine spawn                |
| `task.delay(t, fn)`   | Delayed execution                           |
| `pcall(fn, ...)`      | Protected call (error handling)             |
| `isfile/readfile/writefile` | Roblox filesystem (executor-provided) |
| `loadstring(src)()`   | Dynamically compile and run Lua source      |

---

## Code Conventions

### Naming
- **PascalCase** for module objects, classes, and config keys (`OrionLib`, `MakeWindow`, `SaveCfg`).
- **camelCase** for local variables and helper functions (`addConnection`, `tweenObject`).
- **SCREAMING_SNAKE_CASE** for top-level constants (`DISCORD_WEBHOOK_URL`).
- Service references are declared at the top of the file as locals: `local TweenService = game:GetService("TweenService")`.

### Error Handling
- Use `pcall()` for any operation that might fail (HTTP requests, file I/O, GUI creation).
- Do not let unhandled errors crash the entire script; log or warn instead.

### Connections and Cleanup
- Always register event connections through `AddConnection()` (lib.lua) so they are disconnected on `OrionLib:Destroy()`.
- In spy.lua, track connections in a local table and disconnect on `SimpleSpyShutdown()`.

### Async / Concurrency
- Use `task.spawn()` for fire-and-forget coroutines.
- Use `task.delay()` for timed callbacks.
- Use `coroutine.create/resume/yield` only when fine-grained control over thread lifecycle is needed.

### GUI Best Practices
- Always parent GUI to `game.CoreGui` (or via `gethui()` if available).
- Use `TweenService:Create()` for animations; store tween references if they need to be cancelled.
- Destroy existing duplicate GUIs before creating new ones (check `Interface.Name == Orion.Name`).

---

## Development Workflow

### Loading Scripts
Scripts are not "built" — they are fetched and executed at runtime inside Roblox:

```lua
-- Load Orion Library
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/.../lib.lua"))()

-- Load SimpleSpy
loadstring(game:HttpGet("https://raw.githubusercontent.com/.../spy.lua"))()
```

### Making Changes
1. Edit the `.lua` file directly.
2. Re-execute the script in your Roblox executor to test.
3. Verify GUI renders, interactions work, and no console errors appear.

### No Automated Tests
There is no test framework. Manual in-game testing is the only verification method:
- Open Roblox Studio or use a supported executor.
- Execute the script.
- Exercise all UI elements (buttons, sliders, toggles, dropdowns).
- Check the Roblox output/console for errors.

---

## Git Workflow

- Active development branch: `claude/add-claude-documentation-b5Lzh`
- Main integration branch: `origin/main`
- Stable branch: `master`

Commit messages should be short and descriptive, e.g.:
```
fix: resolve toggle callback not firing on second click
feat: add ColorPicker reset button
docs: update CLAUDE.md with spy.lua config reference
```

---

## What NOT to Do

- **Do not modify Sky.lua or Applefarm.lua.** They are obfuscated and their purpose is unknown.
- **Do not add a build system, bundler, or package manager** — this is a plain Lua project with no Node/npm/cargo tooling.
- **Do not add TypeScript, type annotations, or transpilation** — the target runtime is stock Roblox Lua (Luau).
- **Do not introduce external dependencies** beyond what Roblox provides natively.
- **Do not attempt to deobfuscate** Sky.lua or Applefarm.lua.
- **Do not commit executor API keys, webhook URLs, or credentials.** The `DISCORD_WEBHOOK_URL` variable in spy.lua should remain an empty string in the repository.
