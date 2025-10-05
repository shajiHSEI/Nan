-- ReplicatedFirst > LocalScript
-- One-file version: custom loading screen + "Hax Take Over" UI + sprint system (Studio-safe)

-- Services
local Players = game:GetService("Players")
local ReplicatedFirst = game:GetService("ReplicatedFirst")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- Player / UI containers
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

----------------------------------------------------------------
-- Custom Loading Screen (Fluxus-like vibe, but original assets)
----------------------------------------------------------------
local loaderGui = Instance.new("ScreenGui")
loaderGui.Name = "Hax Take Over Loader"
loaderGui.IgnoreGuiInset = true
loaderGui.ResetOnSpawn = false
loaderGui.Parent = playerGui

local bg = Instance.new("Frame")
bg.Size = UDim2.fromScale(1, 1)
bg.BackgroundColor3 = Color3.fromRGB(14, 17, 23)
bg.Parent = loaderGui

local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 30, 50)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 15, 25))
}
grad.Rotation = 45
grad.Parent = bg

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 60)
title.Position = UDim2.new(0, 0, 0.2, 0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "Speed Hax"
title.TextColor3 = Color3.fromRGB(220, 230, 255)
title.TextScaled = true
title.Parent = bg

local ring = Instance.new("ImageLabel")
ring.Size = UDim2.fromOffset(128, 128)
ring.AnchorPoint = Vector2.new(0.5, 0.5)
ring.Position = UDim2.new(0.5, 0, 0.55, 0)
ring.BackgroundTransparency = 1
-- Simple ring asset; replace with any permitted asset id if desired
ring.Image = "rbxassetid://4965945816"
ring.ImageColor3 = Color3.fromRGB(120, 170, 255)
ring.Parent = bg

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, 0, 0, 28)
subtitle.Position = UDim2.new(0, 0, 0.72, 0)
subtitle.BackgroundTransparency = 1
subtitle.Font = Enum.Font.Gotham
subtitle.Text = "Loading experience…"
subtitle.TextColor3 = Color3.fromRGB(180, 190, 210)
subtitle.TextScaled = true
subtitle.Parent = bg

-- Remove default loader and animate
ReplicatedFirst:RemoveDefaultLoadingScreen()

local spin = TweenService:Create(
    ring,
    TweenInfo.new(2, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1),
    { Rotation = 360 }
)
spin:Play()

task.spawn(function()
    while loaderGui.Parent do
        local t1 = TweenService:Create(title, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {TextTransparency = 0.3})
        t1:Play(); t1.Completed:Wait()
        local t2 = TweenService:Create(title, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {TextTransparency = 0.1})
        t2:Play(); t2.Completed:Wait()
    end
end)

-- Keep minimum time, then wait until fully loaded
task.wait(2)
if not game:IsLoaded() then game.Loaded:Wait() end

-- Fade out and remove loader
local fade = TweenService:Create(bg, TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {BackgroundTransparency = 1})
fade:Play()
fade.Completed:Wait()
loaderGui:Destroy()

-------------------------------------------------------
-- Character utilities (wait for Humanoid to be ready)
-------------------------------------------------------
local function getHumanoid()
    local char = player.Character or player.CharacterAdded:Wait()
    return char:WaitForChild("Humanoid")
end

local humanoid = getHumanoid()
player.CharacterAdded:Connect(function()
    humanoid = getHumanoid()
end)

-------------------------------------------------------
-- "Hax Take Over" UI + Sprint/Speed Controller (Studio)
-------------------------------------------------------
-- Constants
local NORMAL_SPEED = 16            -- default Roblox walk speed
local sprintSpeed = 32             -- adjustable sprint target
local boostEnabled = true
local shiftDown = false

-- Smooth speed tween via NumberValue proxy
local speedValue = Instance.new("NumberValue")
speedValue.Value = NORMAL_SPEED
speedValue.Changed:Connect(function(v)
    if humanoid and humanoid.Parent then
        humanoid.WalkSpeed = v
    end
end)

local function tweenSpeed(target, dur)
    TweenService:Create(
        speedValue,
        TweenInfo.new(dur or 0.25, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
        {Value = target}
    ):Play()
end

local function updateSprint()
    local target = (boostEnabled and shiftDown) and sprintSpeed or NORMAL_SPEED
    tweenSpeed(target, 0.25)
end

-- Build the control UI
local ui = Instance.new("ScreenGui")
ui.Name = "Hax Take Over"
ui.IgnoreGuiInset = true
ui.ResetOnSpawn = false
ui.Parent = playerGui

local panel = Instance.new("Frame")
panel.Size = UDim2.fromOffset(280, 160)
panel.Position = UDim2.new(0, 20, 0, 80)
panel.BackgroundColor3 = Color3.fromRGB(18, 22, 30)
panel.Active = true
panel.Draggable = true
panel.Parent = ui

local stroke = Instance.new("UIStroke", panel)
stroke.Color = Color3.fromRGB(70, 120, 255)
stroke.Thickness = 1

local corner = Instance.new("UICorner", panel)
corner.CornerRadius = UDim.new(0, 10)

local title2 = Instance.new("TextLabel")
title2.Size = UDim2.new(1, -40, 0, 28)
title2.Position = UDim2.new(0, 12, 0, 8)
title2.BackgroundTransparency = 1
title2.Font = Enum.Font.GothamBold
title2.TextXAlignment = Enum.TextXAlignment.Left
title2.Text = "Hax Take Over"
title2.TextColor3 = Color3.fromRGB(220, 230, 255)
title2.TextSize = 18
title2.Parent = panel

local close = Instance.new("TextButton")
close.Size = UDim2.fromOffset(24, 24)
close.Position = UDim2.new(1, -30, 0, 8)
close.BackgroundColor3 = Color3.fromRGB(30, 35, 45)
close.Text = "×"
close.Font = Enum.Font.GothamBold
close.TextSize = 18
close.TextColor3 = Color3.fromRGB(200, 210, 230)
Instance.new("UICorner", close).CornerRadius = UDim.new(0, 6)
close.Parent = panel
close.MouseButton1Click:Connect(function()
    panel.Visible = not panel.Visible
end)

local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, -24, 0, 22)
label.Position = UDim2.new(0, 12, 0, 48)
label.BackgroundTransparency = 1
label.Font = Enum.Font.Gotham
label.TextXAlignment = Enum.TextXAlignment.Left
label.Text = "Speed Hax (Studio)"
label.TextColor3 = Color3.fromRGB(180, 190, 210)
label.TextSize = 16
label.Parent = panel

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.fromOffset(90, 28)
toggle.Position = UDim2.new(0, 12, 0, 78)
toggle.BackgroundColor3 = Color3.fromRGB(40, 60, 95)
toggle.Text = "Boost: ON"
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 14
toggle.TextColor3 = Color3.fromRGB(230, 240, 255)
Instance.new("UICorner", toggle).CornerRadius = UDim.new(0, 8)
toggle.Parent = panel

local minus = Instance.new("TextButton")
minus.Size = UDim2.fromOffset(28, 28)
minus.Position = UDim2.new(0, 112, 0, 78)
minus.BackgroundColor3 = Color3.fromRGB(30, 35, 45)
minus.Text = "−"
minus.Font = Enum.Font.GothamBold
minus.TextSize = 18
minus.TextColor3 = Color3.fromRGB(200, 210, 230)
Instance.new("UICorner", minus).CornerRadius = UDim.new(0, 6)
minus.Parent = panel

local plus = Instance.new("TextButton")
plus.Size = UDim2.fromOffset(28, 28)
plus.Position = UDim2.new(0, 146, 0, 78)
plus.BackgroundColor3 = Color3.fromRGB(30, 35, 45)
plus.Text = "+"
plus.Font = Enum.Font.GothamBold
plus.TextSize = 18
plus.TextColor3 = Color3.fromRGB(200, 210, 230)
Instance.new("UICorner", plus).CornerRadius = UDim.new(0, 6)
plus.Parent = panel

local speedText = Instance.new("TextLabel")
speedText.Size = UDim2.fromOffset(100, 28)
speedText.Position = UDim2.new(0, 180, 0, 78)
speedText.BackgroundTransparency = 1
speedText.Font = Enum.Font.GothamBold
speedText.TextXAlignment = Enum.TextXAlignment.Left
speedText.Text = ("Sprint: %d"):format(sprintSpeed)
speedText.TextColor3 = Color3.fromRGB(190, 210, 255)
speedText.TextSize = 14
speedText.Parent = panel

local tip = Instance.new("TextLabel")
tip.Size = UDim2.new(1, -24, 0, 20)
tip.Position = UDim2.new(0, 12, 0, 118)
tip.BackgroundTransparency = 1
tip.Font = Enum.Font.Gotham
tip.TextXAlignment = Enum.TextXAlignment.Left
tip.Text = "Hold LeftShift to sprint"
tip.TextColor3 = Color3.fromRGB(150, 160, 180)
tip.TextSize = 12
tip.Parent = panel

local function refreshLabels()
    toggle.Text = boostEnabled and "Boost: ON" or "Boost: OFF"
    speedText.Text = ("Sprint: %d"):format(sprintSpeed)
end

toggle.MouseButton1Click:Connect(function()
    boostEnabled = not boostEnabled
    refreshLabels()
    updateSprint()
end)

minus.MouseButton1Click:Connect(function()
    sprintSpeed = math.max(18, sprintSpeed - 2)
    refreshLabels()
    updateSprint()
end)

plus.MouseButton1Click:Connect(function()
    sprintSpeed = math.min(60, sprintSpeed + 2)
    refreshLabels()
    updateSprint()
end)

-- Input handling: LeftShift sprint
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.LeftShift then
        shiftDown = true
        updateSprint()
    end
end)

UserInputService.InputEnded:Connect(function(input, gp)
    if input.KeyCode == Enum.KeyCode.LeftShift then
        shiftDown = false
        updateSprint()
    end
end)

-- Initialize speeds
if humanoid then
    humanoid.WalkSpeed = NORMAL_SPEED
end
speedValue.Value = NORMAL_SPEED
refreshLabels()
