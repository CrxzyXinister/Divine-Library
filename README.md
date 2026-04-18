--[[
════════════════════════════════════════════════════════════════
           DIVINE UI LIBRARY — DOCUMENTATION v1.0
════════════════════════════════════════════════════════════════

OVERVIEW
────────
ZyphrotUI is a self-contained Roblox UI library extracted from
the Zyphrot Hub script. Drop it in and call four lines to get a
fully-featured, themed, draggable hub window with tabs, toggles,
sliders, and buttons — all mobile-aware.

LOADING
────────
Put ZyphrotUI.lua in ReplicatedStorage (or use loadstring):

    -- Option A: ModuleScript in ReplicatedStorage
    local ZyphrotUI = require(game:GetService("ReplicatedStorage").ZyphrotUI)

    -- Option B: loadstring (executor)
    local ZyphrotUI = loadstring(game:HttpGet("YOUR_RAW_URL"))()

════════════════════════════════════════════════════════════════
  QUICK START — copy-paste example
════════════════════════════════════════════════════════════════

    local ZyphrotUI = require(game.ReplicatedStorage.ZyphrotUI)

    -- 1. Create a window
    local Window = ZyphrotUI.new("My Hub", {
        theme      = "Neon Green",   -- optional
        background = "Matrix",       -- optional
    })

    -- 2. Add a tab
    local Tab = Window:AddTab("Movement")

    -- 3. Add a section header
    Tab:AddSection("Speed")

    -- 4. Add a toggle
    local setSpeedVisual = Tab:AddToggle("Speed Boost", false, function(state)
        print("Speed Boost is now:", state)
    end)

    -- 5. Add a slider
    local setSpeedVal = Tab:AddSlider("Walk Speed", 16, 100, 30, function(val)
        -- val updates live while dragging
        local hum = game.Players.LocalPlayer.Character
            and game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = val end
    end)

    -- 6. Add a button
    Tab:AddButton("Teleport to Spawn", function()
        -- your logic here
    end)

    -- 7. Change theme at runtime
    Window:SetTheme("Royal Purple")

    -- 8. Drive toggle/slider externally (e.g. from a keybind)
    game:GetService("UserInputService").InputBegan:Connect(function(input, gpe)
        if gpe then return end
        if input.KeyCode == Enum.KeyCode.V then
            local newState = not mySpeedEnabled
            mySpeedEnabled = newState
            setSpeedVisual(newState)     -- updates switch visuals
        end
    end)

════════════════════════════════════════════════════════════════
  API REFERENCE
════════════════════════════════════════════════════════════════

──────────────────────────────────────────
  ZyphrotUI.new(title, opts?) → Window
──────────────────────────────────────────
Creates the hub window and its ScreenGui.

Parameters:
  title  string   Text shown in the top-left of the sidebar.
  opts   table?   Optional settings:
    .width      number   Window width in pixels (default 650)
    .height     number   Window height in pixels (default 420)
    .theme      string   Starting theme name (see SetTheme)
    .background string   Starting background effect (see SetBackground)

Returns: Window object (see Window API below)

Notes:
  • The window is draggable out-of-the-box.
  • Press [U] or the "ZH" button to hide/show the window.
  • The window auto-scales on mobile (65 % of PC size).


──────────────────────────────────────────
  Window:AddTab(name) → Tab
──────────────────────────────────────────
Adds a tab button to the sidebar and returns a Tab object
you use to add content rows.

Parameters:
  name  string   Label shown on the sidebar button.

Returns: Tab object (see Tab API below)

Example:
    local tabMovement = Window:AddTab("Movement")
    local tabCombat   = Window:AddTab("Combat")


──────────────────────────────────────────
  Window:SetTheme(name)
──────────────────────────────────────────
Switches the colour theme. All existing UI elements recolour
instantly via the internal ThemeUpdateFuncs registry.

Available themes:
  "Crimson Red"       -- default, deep red
  "Cyberpunk Yellow"  -- gold / yellow
  "Neon Green"        -- bright green
  "Royal Purple"      -- purple
  "Rainbow"           -- animated rainbow (runs every frame)

Example:
    Window:SetTheme("Cyberpunk Yellow")


──────────────────────────────────────────
  Window:SetBackground(effectName)
──────────────────────────────────────────
Starts an animated particle background inside the window frame.
Calling it again replaces the current effect.

Available effects:
  "Stars"    -- rising glowing dots (tinted by theme colour)
  "Matrix"   -- falling random ASCII characters
  "Grid"     -- slowly scrolling grid lines
  "None"     -- clears the background

Example:
    Window:SetBackground("Stars")


──────────────────────────────────────────
  Window:Destroy()
──────────────────────────────────────────
Destroys the ScreenGui and all its contents.


════════════════════════════════════════════════════════════════
  TAB API  (returned by Window:AddTab)
════════════════════════════════════════════════════════════════

──────────────────────────────────────────
  Tab:AddSection(text)
──────────────────────────────────────────
Inserts a non-interactive category header between rows.

Parameters:
  text  string   Header label (typically ALL CAPS).

Example:
    Tab:AddSection("MOVEMENT OPTIONS")


──────────────────────────────────────────
  Tab:AddToggle(label, default, callback, keybindKey?, KEYBINDS?) → setVisual
──────────────────────────────────────────
Adds a toggle row with an animated on/off switch.

Parameters:
  label       string    Row label text.
  default     boolean   Starting state (true = on).
  callback    function  Called with (state: boolean) when toggled.
  keybindKey  string?   Key into your KEYBINDS table — shows a
                        clickable [KEY] rebind button next to the switch.
  KEYBINDS    table?    Your keybinds table. Required if keybindKey given.

Returns:
  setVisual  function(state: boolean, skipCallback?: boolean)
             Call this to drive the toggle from outside (e.g. keybinds).
             Pass true as the second arg to skip firing the callback.

Example:
    local KEYBINDS = { SPEED = Enum.KeyCode.V }
    local Enabled  = { SpeedBoost = false }

    local setSpeedVis = Tab:AddToggle(
        "Speed Boost",
        Enabled.SpeedBoost,
        function(state)
            Enabled.SpeedBoost = state
            if state then startSpeed() else stopSpeed() end
        end,
        "SPEED",
        KEYBINDS
    )

    -- Later, from a keybind handler:
    Enabled.SpeedBoost = not Enabled.SpeedBoost
    setSpeedVis(Enabled.SpeedBoost)


──────────────────────────────────────────
  Tab:AddSlider(label, min, max, default, callback, isFloat?) → setVal
──────────────────────────────────────────
Adds a slider row with a draggable thumb and a live text box.
The text box also accepts typed input (validated on focus lost).

Parameters:
  label     string    Row label text.
  min       number    Minimum value.
  max       number    Maximum value.
  default   number    Starting value.
  callback  function  Called with (value: number) while dragging.
  isFloat   boolean?  true = 2 decimal places, false = integers (default).

Returns:
  setVal  function(value: number)
          Programmatically set the slider value and update visuals.

Example:
    local setWalkSpeed = Tab:AddSlider(
        "Walk Speed", 16, 200, 30,
        function(val)
            local hum = game.Players.LocalPlayer.Character
                and game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum.WalkSpeed = val end
        end
    )

    -- Reset to default:
    setWalkSpeed(30)

    -- Float example (gravity percentage):
    Tab:AddSlider("Gravity %", 0.1, 2.0, 1.0, function(val)
        workspace.Gravity = 196.2 * val
    end, true)  -- isFloat = true


──────────────────────────────────────────
  Tab:AddButton(text, callback, color?) → TextButton
──────────────────────────────────────────
Adds a full-width clickable button with a ripple effect.

Parameters:
  text      string    Button label.
  callback  function  Called with (button: TextButton) on click.
  color     Color3?   Custom background colour. Omit for default.

Returns: the raw TextButton instance.

Example:
    Tab:AddButton("Nuke Nearest Player", function()
        local nearest = getNearestPlayer()
        if nearest then nukePlayer(nearest) end
    end)

    -- Custom coloured button:
    Tab:AddButton("Save Config", saveConfig, Color3.fromRGB(40, 160, 80))


──────────────────────────────────────────
  Tab:AddLabel(text, color?) → TextLabel
──────────────────────────────────────────
Inserts a passive informational text row.

Parameters:
  text   string   Content to display.
  color  Color3?  Text colour (defaults to muted grey).

Returns: the raw TextLabel instance (you can update .Text later).

Example:
    local statusLabel = Tab:AddLabel("Status: Idle")

    -- Update later:
    statusLabel.Text = "Status: Active"


════════════════════════════════════════════════════════════════
  INTERNAL UTILITIES (available for advanced use)
════════════════════════════════════════════════════════════════

These are used internally but you can call them directly if you
build your own UI on top of the library:

  MakeDraggable(frame: Frame)
      Makes any Frame draggable on both PC and mobile.

  attachRipple(btn: TextButton, target?: Frame)
      Attaches a material-ripple click effect.
      `target` is where the ripple renders (defaults to btn).

  playSound(id?: string, vol?: number)
      Plays a short UI sound. Default id is the Zyphrot click.

  UpdateTheme(primary, light, dark: Color3)
      Manually sets theme colours and calls all registered
      recolor callbacks.

  StartBackgroundEffect(container: Frame, effectName: string)
      Runs a particle background in any clipped Frame.

════════════════════════════════════════════════════════════════
  FULL EXAMPLE — multi-tab hub
════════════════════════════════════════════════════════════════

    local ZyphrotUI = require(game.ReplicatedStorage.ZyphrotUI)

    local Enabled = {
        SpeedBoost = false,
        InfJump    = false,
        ESP        = false,
    }
    local Values = {
        WalkSpeed = 30,
        JumpPower = 50,
    }
    local KEYBINDS = {
        SPEED = Enum.KeyCode.V,
        JUMP  = Enum.KeyCode.N,
    }

    local Window = ZyphrotUI.new("My Hub", {
        theme      = "Crimson Red",
        background = "Stars",
    })

    -- ── Movement tab ──
    local tabMove = Window:AddTab("Movement")

    tabMove:AddSection("SPEED")
    local setSpeedVis = tabMove:AddToggle(
        "Speed Boost", Enabled.SpeedBoost,
        function(state)
            Enabled.SpeedBoost = state
            -- apply speed logic here
        end,
        "SPEED", KEYBINDS
    )

    local setSpeedVal = tabMove:AddSlider(
        "Walk Speed", 16, 200, Values.WalkSpeed,
        function(val)
            Values.WalkSpeed = val
            local c = game.Players.LocalPlayer.Character
            local h = c and c:FindFirstChildOfClass("Humanoid")
            if h then h.WalkSpeed = val end
        end
    )

    tabMove:AddSection("JUMP")
    tabMove:AddToggle(
        "Infinite Jump", Enabled.InfJump,
        function(state) Enabled.InfJump = state end
    )
    tabMove:AddSlider(
        "Jump Power", 10, 200, Values.JumpPower,
        function(val)
            Values.JumpPower = val
            local c = game.Players.LocalPlayer.Character
            local h = c and c:FindFirstChildOfClass("Humanoid")
            if h then h.JumpPower = val end
        end
    )

    -- ── Visuals tab ──
    local tabVis = Window:AddTab("Visuals")

    tabVis:AddToggle(
        "Player ESP", Enabled.ESP,
        function(state)
            Enabled.ESP = state
            toggleESP(state)
        end
    )

    -- ── Settings tab ──
    local tabSettings = Window:AddTab("Settings")

    tabSettings:AddButton("Theme: Crimson Red", function(btn)
        local themes = {"Crimson Red","Cyberpunk Yellow","Neon Green","Royal Purple","Rainbow"}
        -- (cycle through themes — your own logic)
    end)

    tabSettings:AddButton("Background: Stars", function(btn)
        local bgs = {"Stars","Matrix","Grid","None"}
        -- (cycle through bgs — your own logic)
    end)

    -- ── Keybind handler ──
    game:GetService("UserInputService").InputBegan:Connect(function(input, gpe)
        if gpe then return end
        if input.KeyCode == KEYBINDS.SPEED then
            Enabled.SpeedBoost = not Enabled.SpeedBoost
            setSpeedVis(Enabled.SpeedBoost)
        end
        if input.KeyCode == KEYBINDS.JUMP then
            Enabled.InfJump = not Enabled.InfJump
        end
    end)

════════════════════════════════════════════════════════════════
  DESIGN NOTES
════════════════════════════════════════════════════════════════

• Scale factor `s` (0.65 on mobile, 1.0 on PC) is applied to
  every size, position, offset, and TextSize value so you never
  need to think about mobile separately.

• All colours are stored in the `C` table.  Theme changes update
  `C` in place and call every registered ThemeUpdateFuncs entry,
  so elements recolour live without recreation.

• The sliding tab-switch animation uses Quart easing and a 35ms
  offset so the incoming page slides from the right.

• Ripple effects use absolute mouse/touch position so they always
  originate from the exact click point.

• Keybind rebind: click the [KEY] button on any toggle, then press
  a key. The library handles listening and updating automatically.

════════════════════════════════════════════════════════════════
]]
