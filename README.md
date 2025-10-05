-- ReplicatedFirst > LocalScript
-- Longer loader with place info + key gate (24HOURS) + animated text + Blur + main GUI:
-- TextBoxes: WalkSpeed, JumpPower; Teleport by exact Username or exact Display Name (case-insensitive).

-- Services
local Players = game:GetService("Players")
local ReplicatedFirst = game:GetService("ReplicatedFirst")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local MarketplaceService = game:GetService("MarketplaceService")

-- Player / UI containers
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Aesthetic blur helper (client-side post-processing)
local blur = Instance.new("BlurEffect")
blur.Name = "HTO_Blur"
blur.Size = 0
blur.Parent = Lighting

local function tweenBlur(target, dur)
    TweenService:Create(blur, TweenInfo.new(dur or 0.35, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {Size = target}):Play()
end

-- Helper: flowing black/white UIGradient for TextLabel
local function addFlowingGradient(textLabel)
    local grad = Instance.new("UIGradient")
    grad.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0.0, Color3.fromRGB(0,0,0)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,255,255)),
        ColorSequenceKeypoint.new(1.0, Color3.fromRGB(0,0,0))
    }
    grad.Rotation = 0
    grad.Offset = Vector2.new(-1, 0)
    grad.Parent = textLabel

    task.spawn(function()
        while textLabel.Parent do
            local tween = TweenService:Create(grad, TweenInfo.new(2.0, Enum.EasingStyle.Linear), {Offset = Vector2.new(1, 0)})
            tween:Play()
            tween.Completed:Wait()
            grad.Offset = Vector2.new(-1, 0)
        end
    end)
end

-- Fetch current place/game info safely
local placeName = "Unknown Place"
local success, info = pcall(function()
    return MarketplaceService:GetProductInfo(game.PlaceId)
end)
if success and info and info.Name then
    placeName = tostring(info.Name)
end
local placeIdStr = tostring(game.PlaceId)
local gameIdStr = tostring(game.GameId)

----------------------------------------------------------------
-- Custom Loading Screen (longer + animated + “Made By Steve” + game info)
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

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 60)
title.Position = UDim2.new(0, 0, 0.18, 0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "Speed Hax"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Parent = bg
addFlowingGradient(title)

local ring = Instance.new("ImageLabel")
ring.Size = UDim2.fromOffset(128, 128)
ring.AnchorPoint = Vector2.new(0.5, 0.5)
ring.Position = UDim2.new(0.5, 0, 0.52, 0)
ring.BackgroundTransparency = 1
ring.Image = "rbxassetid://4965945816"
ring.ImageColor3 = Color3.fromRGB(120, 170, 255)
ring.Parent = bg

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, 0, 0, 28)
subtitle.Position = UDim2.new(0, 0, 0.67, 0)
subtitle.BackgroundTransparency = 1
subtitle.Font = Enum.Font.Gotham
subtitle.Text = "Loading experience…"
subtitle.TextColor3 = Color3.fromRGB(255, 255, 255)
subtitle.TextScaled = true
subtitle.Parent = bg
addFlowingGradient(subtitle)

local madeBy = Instance.new("TextLabel")
madeBy.Size = UDim2.new(1, 0, 0, 24)
madeBy.Position = UDim2.new(0, 0, 0.76, 0)
madeBy.BackgroundTransparency = 1
madeBy.Font = Enum.Font.GothamMedium
madeBy.Text = "Made By Steve"
madeBy.TextColor3 = Color3.fromRGB(255, 255, 255)
madeBy.TextScaled = true
madeBy.Parent = bg
addFlowingGradient(madeBy)

-- Side info panel: game name, IDs, Supported
local infoPanel = Instance.new("Frame")
infoPanel.Size = UDim2.fromOffset(360, 80)
infoPanel.Position = UDim2.new(1, -380, 0, 20)
infoPanel.BackgroundColor3 = Color3.fromRGB(20, 24, 34)
infoPanel.BackgroundTransparency = 0.1
infoPanel.Parent = bg
Instance.new("UICorner", infoPanel).CornerRadius = UDim.new(0, 10)
local infoStroke = Instance.new("UIStroke", infoPanel)
infoStroke.Thickness = 1
infoStroke.Color = Color3.fromRGB(90, 130, 255)

local gameLine = Instance.new("TextLabel")
gameLine.Size = UDim2.new(1, -16, 0, 24)
gameLine.Position = UDim2.new(0, 8, 0, 8)
gameLine.BackgroundTransparency = 1
gameLine.Font = Enum.Font.GothamMedium
gameLine.TextXAlignment = Enum.TextXAlignment.Left
gameLine.Text = ("Game: %s"):format(placeName)
gameLine.TextColor3 = Color3.fromRGB(255,255,255)
gameLine.TextSize = 14
gameLine.Parent = infoPanel
addFlowingGradient(gameLine)

local idLine = Instance.new("TextLabel")
idLine.Size = UDim2.new(1, -16, 0, 22)
idLine.Position = UDim2.new(0, 8, 0, 34)
idLine.BackgroundTransparency = 1
idLine.Font = Enum.Font.Gotham
idLine.TextXAlignment = Enum.TextXAlignment.Left
idLine.Text = ("PlaceId: %s  |  GameId: %s"):format(placeIdStr, gameIdStr)
idLine.TextColor3 = Color3.fromRGB(200,220,255)
idLine.TextSize = 13
idLine.Parent = infoPanel
addFlowingGradient(idLine)

local sup = Instance.new("TextLabel")
sup.Size = UDim2.new(1, -16, 0, 22)
sup.Position = UDim2.new(0, 8, 0, 56)
sup.BackgroundTransparency = 1
sup.Font = Enum.Font.GothamBold
sup.TextXAlignment = Enum.TextXAlignment.Left
sup.Text = "Supported"
sup.TextColor3 = Color3.fromRGB(180,255,200)
sup.TextSize = 13
sup.Parent = infoPanel
addFlowingGradient(sup)

-- Replace default loader and animate
ReplicatedFirst:RemoveDefaultLoadingScreen()
local spin = TweenService:Create(ring, TweenInfo.new(2, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1), {Rotation = 360})
spin:Play()
tweenBlur(12, 0.25)

-- Longer minimum duration
local MIN_DURATION = 8.0
local startTime = time()
if not game:IsLoaded() then game.Loaded:Wait() end
local remain = math.max(0, MIN_DURATION - (time() - startTime))
task.wait(remain)

-- Fade out loader
TweenService:Create(bg, TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {BackgroundTransparency = 1}):Play()
task.wait(0.5)
loaderGui:Destroy()

----------------------------------------------------------------
-- Key Gate UI (accepts 24HOURS, then shows "Valid Key!")
----------------------------------------------------------------
local keyGui = Instance.new("ScreenGui")
keyGui.Name = "HTO_KeyGate"
keyGui.IgnoreGuiInset = true
keyGui.ResetOnSpawn = false
keyGui.Parent = playerGui

local panel = Instance.new("Frame")
panel.Size = UDim2.fromOffset(360, 180)
panel.Position = UDim2.new(0.5, 0, 0.5, 0)
panel.AnchorPoint = Vector2.new(0.5, 0.5)
panel.BackgroundColor3 = Color3.fromRGB(18, 22, 30)
panel.Parent = keyGui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 12)
local stroke = Instance.new("UIStroke", panel)
stroke.Thickness = 1
stroke.Color = Color3.fromRGB(70, 120, 255)

local header = Instance.new("TextLabel")
header.Size = UDim2.new(1, -24, 0, 30)
header.Position = UDim2.new(0, 12, 0, 10)
header.BackgroundTransparency = 1
header.Font = Enum.Font.GothamBold
header.TextXAlignment = Enum.TextXAlignment.Left
header.Text = "Key System"
header.TextColor3 = Color3.fromRGB(255, 255, 255)
header.TextSize = 18
header.Parent = panel
addFlowingGradient(header)

local prompt = Instance.new("TextLabel")
prompt.Size = UDim2.new(1, -24, 0, 22)
prompt.Position = UDim2.new(0, 12, 0, 50)
prompt.BackgroundTransparency = 1
prompt.Font = Enum.Font.Gotham
prompt.TextXAlignment = Enum.TextXAlignment.Left
prompt.Text = "Enter key to continue (hint: 24HOURS)"
prompt.TextColor3 = Color3.fromRGB(255, 255, 255)
prompt.TextSize = 14
prompt.Parent = panel
addFlowingGradient(prompt)

local keyBox = Instance.new("TextBox")
keyBox.Size = UDim2.new(1, -24, 0, 36)
keyBox.Position = UDim2.new(0, 12, 0, 80)
keyBox.BackgroundColor3 = Color3.fromRGB(30, 36, 48)
keyBox.TextColor3 = Color3.fromRGB(230, 240, 255)
keyBox.Font = Enum.Font.GothamBold
keyBox.TextSize = 16
keyBox.ClearTextOnFocus = false
keyBox.PlaceholderText = "Type key here"
keyBox.Text = ""
keyBox.Parent = panel
Instance.new("UICorner", keyBox).CornerRadius = UDim.new(0, 8)

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -24, 0, 22)
status.Position = UDim2.new(0, 12, 0, 124)
status.BackgroundTransparency = 1
status.Font = Enum.Font.Gotham
status.TextXAlignment = Enum.TextXAlignment.Left
status.Text = ""
status.TextColor3 = Color3.fromRGB(180, 200, 255)
status.TextSize = 14
status.Parent = panel

local submit = Instance.new("TextButton")
submit.Size = UDim2.new(0, 110, 0, 32)
submit.Position = UDim2.new(1, -122, 1, -42)
submit.BackgroundColor3 = Color3.fromRGB(40, 60, 95)
submit.Text = "Unlock"
submit.Font = Enum.Font.GothamBold
submit.TextSize = 14
submit.TextColor3 = Color3.fromRGB(230, 240, 255)
submit.Parent = panel
Instance.new("UICorner", submit).CornerRadius = UDim.new(0, 8)

local function validateKey()
    local entered = (keyBox.Text or ""):gsub("^%s*(.-)%s*$", "%1")
    if string.upper(entered) == "24HOURS" then
        status.Text = "Valid Key!"
        addFlowingGradient(status)
        TweenService:Create(panel, TweenInfo.new(0.35, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 1}):Play()
        task.wait(0.4)
        keyGui:Destroy()
        return true
    else
        status.Text = "Invalid key"
        status.TextColor3 = Color3.fromRGB(255, 120, 120)
        return false
    end
end

submit.MouseButton1Click:Connect(validateKey)
keyBox.FocusLost:Connect(function(enter)
    if enter then validateKey() end
end)

----------------------------------------------------------------
-- After key accepted: build main GUI and reduce blur slightly
----------------------------------------------------------------
while keyGui.Parent do task.wait() end
tweenBlur(2, 0.35)

-- Character helpers
local function getHumanoid()
    local char = player.Character or player.CharacterAdded:Wait()
    return char:WaitForChild("Humanoid")
end
local function getRoot(hum)
    return hum and hum.Parent and hum.Parent:FindFirstChild("HumanoidRootPart") or nil
end

local humanoid = getHumanoid()
player.CharacterAdded:Connect(function()
    humanoid = getHumanoid()
end)

-- Toast helper
local function showToast(msg)
    local toastGui = Instance.new("ScreenGui")
    toastGui.Name = "HTO_Toast"
    toastGui.IgnoreGuiInset = true
    toastGui.ResetOnSpawn = false
    toastGui.Parent = playerGui

    local frame = Instance.new("Frame")
    frame.Size = UDim2.fromOffset(260, 36)
    frame.Position = UDim2.new(1, -280, 1, -60)
    frame.BackgroundColor3 = Color3.fromRGB(20, 24, 34)
    frame.BackgroundTransparency = 0.1
    frame.Parent = toastGui
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
    local s = Instance.new("UIStroke", frame)
    s.Thickness = 1
    s.Color = Color3.fromRGB(90, 130, 255)

    local text = Instance.new("TextLabel")
    text.Size = UDim2.new(1, -20, 1, 0)
    text.Position = UDim2.new(0, 10, 0, 0)
    text.BackgroundTransparency = 1
    text.Font = Enum.Font.GothamMedium
    text.TextXAlignment = Enum.TextXAlignment.Left
    text.Text = msg
    text.TextColor3 = Color3.fromRGB(255, 255, 255)
    text.TextSize = 14
    text.Parent = frame
    addFlowingGradient(text)

    frame.BackgroundTransparency = 1
    TweenService:Create(frame, TweenInfo.new(0.25, Enum.EasingStyle.Sine), {BackgroundTransparency = 0.1}):Play()
    task.wait(1.4)
    TweenService:Create(frame, TweenInfo.new(0.25, Enum.EasingStyle.Sine), {BackgroundTransparency = 1}):Play()
    task.wait(0.3)
    toastGui:Destroy()
end

-------------------------------------------------------
-- Main GUI: animated text + Speed/Jump/Teleport
-------------------------------------------------------
local NORMAL_SPEED = 16
local DEFAULT_JUMPPOWER = 50
local MIN_SPEED, MAX_SPEED = 1, 200
local MIN_JP, MAX_JP = 0, 300

-- Smooth proxies
local speedValue = Instance.new("NumberValue")
speedValue.Value = NORMAL_SPEED
speedValue.Changed:Connect(function(v)
    if humanoid and humanoid.Parent then
        humanoid.WalkSpeed = v
    end
end)

local jumpValue = Instance.new("NumberValue")
jumpValue.Value = DEFAULT_JUMPPOWER
jumpValue.Changed:Connect(function(v)
    if humanoid and humanoid.Parent then
        humanoid.UseJumpPower = true
        humanoid.JumpPower = v
    end
end)

local function tweenNumberValue(valObj, target, dur)
    TweenService:Create(valObj, TweenInfo.new(dur or 0.25, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {Value = target}):Play()
end

local ui = Instance.new("ScreenGui")
ui.Name = "Hax Take Over"
ui.IgnoreGuiInset = true
ui.ResetOnSpawn = false
ui.Parent = playerGui

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(380, 240)
main.Position = UDim2.new(0, 20, 0, 80)
main.BackgroundColor3 = Color3.fromRGB(18, 22, 30)
main.Active = true
main.Draggable = true
main.Parent = ui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)
local stroke2 = Instance.new("UIStroke", main)
stroke2.Thickness = 1
stroke2.Color = Color3.fromRGB(70, 120, 255)

local header2 = Instance.new("TextLabel")
header2.Size = UDim2.new(1, -24, 0, 30)
header2.Position = UDim2.new(0, 12, 0, 8)
header2.BackgroundTransparency = 1
header2.Font = Enum.Font.GothamBold
header2.TextXAlignment = Enum.TextXAlignment.Left
header2.Text = "Hax Take Over"
header2.TextColor3 = Color3.fromRGB(255, 255, 255)
header2.TextSize = 18
header2.Parent = main
addFlowingGradient(header2)

-- Speed
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0, 160, 0, 22)
speedLabel.Position = UDim2.new(0, 12, 0, 50)
speedLabel.BackgroundTransparency = 1
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Text = "Walk speed (stud/s)"
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 14
speedLabel.Parent = main
addFlowingGradient(speedLabel)

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0, 130, 0, 32)
speedBox.Position = UDim2.new(0, 170, 0, 44)
speedBox.BackgroundColor3 = Color3.fromRGB(30, 36, 48)
speedBox.TextColor3 = Color3.fromRGB(230, 240, 255)
speedBox.Font = Enum.Font.GothamBold
speedBox.TextSize = 16
speedBox.ClearTextOnFocus = false
speedBox.PlaceholderText = "e.g. 16"
speedBox.Text = tostring(NORMAL_SPEED)
speedBox.Parent = main
Instance.new("UICorner", speedBox).CornerRadius = UDim.new(0, 8)

local speedNow = Instance.new("TextLabel")
speedNow.Size = UDim2.new(0, 120, 0, 22)
speedNow.Position = UDim2.new(0, 310, 0, 50)
speedNow.BackgroundTransparency = 1
speedNow.Font = Enum.Font.Gotham
speedNow.TextXAlignment = Enum.TextXAlignment.Left
speedNow.Text = ("Now: %d"):format(NORMAL_SPEED)
speedNow.TextColor3 = Color3.fromRGB(150, 200, 255)
speedNow.TextSize = 14
speedNow.Parent = main

-- JumpPower
local jpLabel = Instance.new("TextLabel")
jpLabel.Size = UDim2.new(0, 160, 0, 22)
jpLabel.Position = UDim2.new(0, 12, 0, 92)
jpLabel.BackgroundTransparency = 1
jpLabel.Font = Enum.Font.Gotham
jpLabel.TextXAlignment = Enum.TextXAlignment.Left
jpLabel.Text = "Jump power"
jpLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
jpLabel.TextSize = 14
jpLabel.Parent = main
addFlowingGradient(jpLabel)

local jpBox = Instance.new("TextBox")
jpBox.Size = UDim2.new(0, 130, 0, 32)
jpBox.Position = UDim2.new(0, 170, 0, 86)
jpBox.BackgroundColor3 = Color3.fromRGB(30, 36, 48)
jpBox.TextColor3 = Color3.fromRGB(230, 240, 255)
jpBox.Font = Enum.Font.GothamBold
jpBox.TextSize = 16
jpBox.ClearTextOnFocus = false
jpBox.PlaceholderText = "e.g. 50"
jpBox.Text = tostring(DEFAULT_JUMPPOWER)
jpBox.Parent = main
Instance.new("UICorner", jpBox).CornerRadius = UDim.new(0, 8)

local jpNow = Instance.new("TextLabel")
jpNow.Size = UDim2.new(0, 120, 0, 22)
jpNow.Position = UDim2.new(0, 310, 0, 92)
jpNow.BackgroundTransparency = 1
jpNow.Font = Enum.Font.Gotham
jpNow.TextXAlignment = Enum.TextXAlignment.Left
jpNow.Text = ("Now: %d"):format(DEFAULT_JUMPPOWER)
jpNow.TextColor3 = Color3.fromRGB(150, 200, 255)
jpNow.TextSize = 14
jpNow.Parent = main

-- Teleport by exact Username or exact DisplayName (case-insensitive)
local tpLabel = Instance.new("TextLabel")
tpLabel.Size = UDim2.new(0, 240, 0, 22)
tpLabel.Position = UDim2.new(0, 12, 0, 134)
tpLabel.BackgroundTransparency = 1
tpLabel.Font = Enum.Font.Gotham
tpLabel.TextXAlignment = Enum.TextXAlignment.Left
tpLabel.Text = "Teleport to username or display name"
tpLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
tpLabel.TextSize = 14
tpLabel.Parent = main
addFlowingGradient(tpLabel)

local tpBox = Instance.new("TextBox")
tpBox.Size = UDim2.new(0, 240, 0, 32)
tpBox.Position = UDim2.new(0, 120, 0, 128)
tpBox.BackgroundColor3 = Color3.fromRGB(30, 36, 48)
tpBox.TextColor3 = Color3.fromRGB(230, 240, 255)
tpBox.Font = Enum.Font.GothamBold
tpBox.TextSize = 16
tpBox.ClearTextOnFocus = false
tpBox.PlaceholderText = "Exact Username or Display Name"
tpBox.Text = ""
tpBox.Parent = main
Instance.new("UICorner", tpBox).CornerRadius = UDim.new(0, 8)

-- Handlers and parsing
local function clampNumber(txt, minV, maxV, fallback)
    local n = tonumber(txt)
    if not n then return fallback end
    n = math.floor(n + 0.5)
    return math.clamp(n, minV, maxV)
end

speedBox.FocusLost:Connect(function()
    local n = clampNumber(speedBox.Text, MIN_SPEED, MAX_SPEED, math.floor(speedValue.Value + 0.5))
    speedBox.Text = tostring(n)
    tweenNumberValue(speedValue, n, 0.25)
    speedNow.Text = ("Now: %d"):format(n)
end)

jpBox.FocusLost:Connect(function()
    local n = clampNumber(jpBox.Text, MIN_JP, MAX_JP, math.floor(jumpValue.Value + 0.5))
    jpBox.Text = tostring(n)
    tweenNumberValue(jumpValue, n, 0.25)
    jpNow.Text = ("Now: %d"):format(n)
end)

local function findPlayerByExact(input)
    if not input or input == "" then return nil end
    local needle = string.lower(input)
    local candidates = Players:GetPlayers()
    -- Prefer exact username matches, then exact display name matches (case-insensitive)
    for _, plr in ipairs(candidates) do
        if string.lower(plr.Name) == needle then
            return plr
        end
    end
    for _, plr in ipairs(candidates) do
        if string.lower(plr.DisplayName) == needle then
            return plr
        end
    end
    return nil
end

tpBox.FocusLost:Connect(function()
    local raw = (tpBox.Text or ""):gsub("^%s*(.-)%s*$", "%1")
    if raw == "" then return end
    local target = findPlayerByExact(raw)
    if not target or not target.Character then
        showToast("Player not found")
        return
    end
    local myHum = humanoid
    local myRoot = getRoot(myHum)
    local tHum = target.Character:FindFirstChildOfClass("Humanoid")
    local tRoot = getRoot(tHum)
    if myRoot and tRoot then
        -- Try PivotTo for smoother whole-model move, fallback to HRP CFrame
        local char = myHum.Parent
        local dest = tRoot.CFrame * CFrame.new(0, 5, 0)
        if char and char:IsA("Model") and char.PrimaryPart then
            char:PivotTo(dest)
        elseif char and char:IsA("Model") then
            -- Temporarily set PrimaryPart if missing
            char.PrimaryPart = myRoot
            char:PivotTo(dest)
            char.PrimaryPart = nil
        else
            myRoot.CFrame = dest
        end
        showToast("Done!")
    else
        showToast("Teleport failed")
    end
end)

-- Defaults
local function applyDefaults()
    if humanoid then
        humanoid.WalkSpeed = NORMAL_SPEED
        humanoid.UseJumpPower = true
        humanoid.JumpPower = DEFAULT_JUMPPOWER
    end
    speedValue.Value = NORMAL_SPEED
    jumpValue.Value = DEFAULT_JUMPPOWER
    speedBox.Text = tostring(NORMAL_SPEED)
    jpBox.Text = tostring(DEFAULT_JUMPPOWER)
    speedNow.Text = ("Now: %d"):format(NORMAL_SPEED)
    jpNow.Text = ("Now: %d"):format(DEFAULT_JUMPPOWER)
end

applyDefaults()
player.Cha
