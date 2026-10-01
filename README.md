-- Ria (Hood Customs Edition - Cinnamoroll Sanrio Theme)
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- Prevent duplicate UI instances & cleanup old connections
if CoreGui:FindFirstChild("RiaHoodCustomsHub") then
    CoreGui.RiaHoodCustomsHub:Destroy()
end
if CoreGui:FindFirstChild("PastelSilentAimUI") then
    CoreGui.PastelSilentAimUI:Destroy()
end

local RiaHub = Instance.new("ScreenGui")
RiaHub.Name = "RiaHoodCustomsHub"
RiaHub.ResetOnSpawn = false
RiaHub.Parent = CoreGui

-- ========================================================
-- ☁️ CUTE CINNAMOROLL LOADING SCREEN (5 SECONDS)
-- ========================================================
local LoadingScreen = Instance.new("Frame")
LoadingScreen.Size = UDim2.new(0, 340, 0, 170)
LoadingScreen.Position = UDim2.new(0.5, -170, 0.5, -85)
LoadingScreen.BackgroundColor3 = Color3.fromRGB(240, 248, 255)
LoadingScreen.BorderSizePixel = 0
LoadingScreen.Parent = RiaHub

local LoadCorner = Instance.new("UICorner")
LoadCorner.CornerRadius = UDim.new(0, 16)
LoadCorner.Parent = LoadingScreen

local LoadStroke = Instance.new("UIStroke")
LoadStroke.Color = Color3.fromRGB(135, 206, 250)
LoadStroke.Thickness = 3
LoadStroke.Parent = LoadingScreen

local LoadTitle = Instance.new("TextLabel")
LoadTitle.Size = UDim2.new(1, 0, 0, 45)
LoadTitle.Position = UDim2.new(0, 0, 0, 15)
LoadTitle.BackgroundTransparency = 1
LoadTitle.Font = Enum.Font.GothamBold
LoadTitle.Text = "☁️ Loading Ria... ☁️"
LoadTitle.TextColor3 = Color3.fromRGB(70, 130, 180)
LoadTitle.TextSize = 15
LoadTitle.Parent = LoadingScreen

local LoadSub = Instance.new("TextLabel")
LoadSub.Size = UDim2.new(1, 0, 0, 30)
LoadSub.Position = UDim2.new(0, 0, 0, 55)
LoadSub.BackgroundTransparency = 1
LoadSub.Font = Enum.Font.GothamMedium
LoadSub.Text = "✨ Press Right Control to toggle UI ✨"
LoadSub.TextColor3 = Color3.fromRGB(100, 149, 237)
LoadSub.TextSize = 12
LoadSub.Parent = LoadingScreen

local BarBg = Instance.new("Frame")
BarBg.Size = UDim2.new(0, 280, 0, 12)
BarBg.Position = UDim2.new(0.5, -140, 0.8, -25)
BarBg.BackgroundColor3 = Color3.fromRGB(225, 238, 250)
BarBg.BorderSizePixel = 0
BarBg.Parent = LoadingScreen

local BarBgCorner = Instance.new("UICorner")
BarBgCorner.CornerRadius = UDim.new(1, 0)
BarBgCorner.Parent = BarBg

local BarFill = Instance.new("Frame")
BarFill.Size = UDim2.new(0, 0, 1, 0)
BarFill.BackgroundColor3 = Color3.fromRGB(135, 206, 250)
BarFill.BorderSizePixel = 0
BarFill.Parent = BarBg

local BarFillCorner = Instance.new("UICorner")
BarFillCorner.CornerRadius = UDim.new(1, 0)
BarFillCorner.Parent = BarFill

pcall(function()
    -- Smoothly animates the loading bar for exactly 5 seconds
    TweenService:Create(BarFill, TweenInfo.new(5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 1, 0)}):Play()
end)
task.wait(5.2)
pcall(function() LoadingScreen:Destroy() end)

-- ========================================================
-- ⚙️ CONFIGURATION & STATES
-- ========================================================
local cfg = {
    silentAim = false,
    useKeybind = true,
    silentAimKey = Enum.KeyCode.V,
    uiToggleKey = Enum.KeyCode.RightControl,
    silentAimHitChance = 100,
    silentAimFOV = 150,
    silentAimFOVShow = false,
    silentAimFOVFilled = false,
    silentAimFOVOpacity = 0.2,
    silentAimFOVColor = Color3.fromRGB(160, 210, 235),
    
    silentAimPart = "Head",
    silentAimClosestPart = false,
    
    silentAimTeamCheck = false,
    silentAimWallCheck = false,
    silentAimMaxDist = 1000,
    
    silentAimPredX = 0,
    silentAimPredY = 0,
    
    bypassRevolver = false,
}

getgenv().AimAssistEnabled = false
getgenv().SpeedEnabled = false
getgenv().FlyEnabled = false
getgenv().TeleportLoopEnabled = false
getgenv().EspEnabled = false

getgenv().CustomSpeed = 50
getgenv().CustomJumpPower = 50
getgenv().FlySpeed = 60
getgenv().AimSmoothness = 2 
getgenv().SelectedAimHitpart = "Head"
getgenv().SelectedTeleportTargetName = nil

getgenv().AimAssistKey = Enum.KeyCode.Q
getgenv().SpeedKey = Enum.KeyCode.Z
getgenv().FlyKey = Enum.KeyCode.C
getgenv().TeleportKey = Enum.KeyCode.X

local bodyPartsList = {
    "Head", "UpperTorso", "HumanoidRootPart", "LowerTorso",
    "LeftUpperArm", "RightUpperArm", "LeftLowerArm", "RightLowerArm",
    "LeftHand", "RightHand", "LeftUpperLeg", "RightUpperLeg",
    "LeftLowerLeg", "RightLowerLeg", "LeftFoot", "RightFoot"
}

local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1.5
fovCircle.NumSides = 64
fovCircle.Radius = cfg.silentAimFOV
fovCircle.Color = cfg.silentAimFOVColor
fovCircle.Filled = cfg.silentAimFOVFilled
fovCircle.Visible = cfg.silentAimFOVShow
fovCircle.Transparency = 1 - cfg.silentAimFOVOpacity

local silentAimCachedPart = nil
local espObjects = {}
local flyConnection = nil
local bodyVel = nil
local bodyGyro = nil
local lockedPlayer = nil

-- ========================================================
-- 🎯 SILENT AIM UTILITIES
-- ========================================================
local function isHoldingRevolver()
    if not cfg.bypassRevolver then return false end
    local char = LocalPlayer.Character
    if not char then return false end
    
    local tool = char:FindFirstChildOfClass("Tool")
    if tool then
        local toolName = string.lower(tool.Name)
        if string.find(toolName, "revolver") or string.find(toolName, "rev") then
            return true
        end
    end
    return false
end

local function getHum(p)
    local c = p and p.Character
    return c and c:FindFirstChildOfClass("Humanoid")
end

local function getHRP(p)
    local c = p and p.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end

local function isAlive(p)
    local h = getHum(p)
    return h and h.Health > 0
end

local function sameTeam(p)
    return LocalPlayer.Team and p.Team and LocalPlayer.Team == p.Team
end

local function wallBetween(pos)
    if not cfg.silentAimWallCheck then return false end
    local ro = Camera.CFrame.Position
    local rd = pos - ro
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {LocalPlayer.Character}
    params.FilterType = Enum.RaycastFilterType.Exclude
    local hit = workspace:Raycast(ro, rd, params)
    if not hit then return false end
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character and hit.Instance:IsDescendantOf(p.Character) then return false end
    end
    return true
end

local function safeWorldToViewportPoint(pos)
    local sp, on = Vector3.new(), false
    pcall(function()
        sp, on = Camera:WorldToViewportPoint(pos)
    end)
    return sp, on
end

local function getClosestBodyPart(char)
    local closestPart, shortestDist = nil, math.huge
    local mousePos = UserInputService:GetMouseLocation()
    for _, pName in ipairs(bodyPartsList) do
        local part = char:FindFirstChild(pName)
        if part then
            local screenPos, onScreen = safeWorldToViewportPoint(part.Position)
            local dist = onScreen and (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude or math.huge
            if dist < shortestDist then
                shortestDist = dist
                closestPart = part
            end
        end
    end
    return closestPart or char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
end

local function getTargetPart(char)
    if not char then return nil end
    if cfg.silentAimClosestPart then
        return getClosestBodyPart(char)
    end
    return char:FindFirstChild(cfg.silentAimPart) or char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
end

local function getClosestPlayerToCursor()
    if isHoldingRevolver() then return nil end
    
    local closestPlayer = nil
    local shortestDist = cfg.silentAimFOV
    local mousePos = UserInputService:GetMouseLocation()

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and isAlive(p) then
            if cfg.silentAimTeamCheck and sameTeam(p) then continue end
            local hrp = getHRP(p)
            if hrp then
                local dist3D = (hrp.Position - Camera.CFrame.Position).Magnitude
                if dist3D <= cfg.silentAimMaxDist then
                    local sp, onScreen = safeWorldToViewportPoint(hrp.Position)
                    if onScreen then
                        local dist2D = (Vector2.new(sp.X, sp.Y) - mousePos).Magnitude
                        if dist2D < shortestDist then
                            local part = getTargetPart(p.Character)
                            if part and not wallBetween(part.Position) then
                                shortestDist = dist2D
                                closestPlayer = part
                            end
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- HOOK METAMETHOD FOR SILENT AIM
local _grm = getrawmetatable(game)
local _oldIndex = _grm.__index
setreadonly(_grm, false)

_grm.__index = function(self, key)
    if not checkcaller() and self == Mouse and cfg.silentAim and not isHoldingRevolver() then
        if (key == "Hit" or key == "Target" or key == "UnitRay") and silentAimCachedPart then
            if math.random(1, 100) <= cfg.silentAimHitChance then
                local origin = Camera.CFrame.Position
                local hitPos = silentAimCachedPart.Position + Vector3.new(
                    silentAimCachedPart.AssemblyLinearVelocity.X * cfg.silentAimPredX,
                    silentAimCachedPart.AssemblyLinearVelocity.Y * cfg.silentAimPredY,
                    silentAimCachedPart.AssemblyLinearVelocity.Z * cfg.silentAimPredX
                )
                if key == "UnitRay" then
                    return Ray.new(origin, (hitPos - origin).Unit)
                elseif key == "Hit" then
                    return CFrame.new(hitPos)
                elseif key == "Target" then
                    return silentAimCachedPart
                end
            end
        end
    end
    return _oldIndex(self, key)
end
setreadonly(_grm, true)

-- ========================================================
-- 🎨 MAIN UI WINDOW DESIGN
-- ========================================================
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 620, 0, 430)
MainFrame.Position = UDim2.new(0.5, -310, 0.5, -215)
MainFrame.BackgroundColor3 = Color3.fromRGB(245, 250, 255)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = RiaHub

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(135, 206, 250)
MainStroke.Thickness = 2.5
MainStroke.Parent = MainFrame

-- Topbar
local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 42)
TopBar.BackgroundColor3 = Color3.fromRGB(230, 242, 255)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local TopBarCorner = Instance.new("UICorner")
TopBarCorner.CornerRadius = UDim.new(0, 12)
TopBarCorner.Parent = TopBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -120, 1, 0)
TitleLabel.Position = UDim2.new(0, 14, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Text = "☁️ Ria"
TitleLabel.TextColor3 = Color3.fromRGB(25, 25, 112)
TitleLabel.TextSize = 14
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TopBar

local StickerBadge = Instance.new("TextLabel")
StickerBadge.Size = UDim2.new(0, 115, 0, 24)
StickerBadge.Position = UDim2.new(1, -170, 0.5, -12)
StickerBadge.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
StickerBadge.Font = Enum.Font.GothamBold
StickerBadge.Text = "✨ Sanrio Edition ✨"
StickerBadge.TextColor3 = Color3.fromRGB(70, 130, 180)
StickerBadge.TextSize = 10
StickerBadge.Parent = TopBar

local StickerCorner = Instance.new("UICorner")
StickerCorner.CornerRadius = UDim.new(1, 0)
StickerCorner.Parent = StickerBadge

local StickerStroke = Instance.new("UIStroke")
StickerStroke.Color = Color3.fromRGB(135, 206, 250)
StickerStroke.Thickness = 1.5
StickerStroke.Parent = StickerBadge

local UnloadButton = Instance.new("TextButton")
UnloadButton.Size = UDim2.new(0, 28, 0, 28)
UnloadButton.Position = UDim2.new(1, -36, 0.5, -14)
UnloadButton.BackgroundColor3 = Color3.fromRGB(255, 182, 193)
UnloadButton.Font = Enum.Font.GothamBold
UnloadButton.Text = "X"
UnloadButton.TextColor3 = Color3.fromRGB(139, 0, 0)
UnloadButton.TextSize = 11
UnloadButton.Parent = TopBar

local UnloadCorner = Instance.new("UICorner")
UnloadCorner.CornerRadius = UDim.new(0, 6)
UnloadCorner.Parent = UnloadButton

-- Dragging Support
local dragging, dragInput, dragStart, startPos
TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)
TopBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Sidebar
local Sidebar = Instance.new("ScrollingFrame")
Sidebar.Size = UDim2.new(0, 150, 1, -52)
Sidebar.Position = UDim2.new(0, 6, 0, 46)
Sidebar.BackgroundTransparency = 1
Sidebar.BorderSizePixel = 0
Sidebar.CanvasSize = UDim2.new(0, 0, 0, 320)
Sidebar.ScrollBarThickness = 2
Sidebar.Parent = MainFrame

local SidebarLayout = Instance.new("UIListLayout")
SidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder
SidebarLayout.Padding = UDim.new(0, 5)
SidebarLayout.Parent = Sidebar

local PagesContainer = Instance.new("Frame")
PagesContainer.Size = UDim2.new(1, -164, 1, -52)
PagesContainer.Position = UDim2.new(0, 160, 0, 46)
PagesContainer.BackgroundTransparency = 1
PagesContainer.Parent = MainFrame

local tabs = {}
local activeTabName = nil

local function createTabPage(name)
    local scrollingFrame = Instance.new("ScrollingFrame")
    scrollingFrame.Name = name .. "Page"
    scrollingFrame.Size = UDim2.new(1, 0, 1, 0)
    scrollingFrame.BackgroundTransparency = 1
    scrollingFrame.BorderSizePixel = 0
    scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 550)
    scrollingFrame.ScrollBarThickness = 3
    scrollingFrame.Visible = false
    scrollingFrame.Parent = PagesContainer

    local listLayout = Instance.new("UIListLayout")
    listLayout.SortOrder = Enum.SortOrder.LayoutOrder
    listLayout.Padding = UDim.new(0, 8)
    listLayout.Parent = scrollingFrame

    local tabButton = Instance.new("TextButton")
    tabButton.Size = UDim2.new(1, 0, 0, 34)
    tabButton.BackgroundColor3 = Color3.fromRGB(235, 245, 255)
    tabButton.Font = Enum.Font.GothamMedium
    tabButton.Text = "  " .. name
    tabButton.TextColor3 = Color3.fromRGB(70, 130, 180)
    tabButton.TextSize = 11
    tabButton.TextXAlignment = Enum.TextXAlignment.Left
    tabButton.AutoButtonColor = false
    tabButton.Parent = Sidebar

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = tabButton

    tabButton.MouseButton1Click:Connect(function()
        for _, t in pairs(tabs) do
            t.Page.Visible = false
            t.Button.BackgroundColor3 = Color3.fromRGB(235, 245, 255)
            t.Button.TextColor3 = Color3.fromRGB(70, 130, 180)
        end
        scrollingFrame.Visible = true
        tabButton.BackgroundColor3 = Color3.fromRGB(135, 206, 250)
        tabButton.TextColor3 = Color3.fromRGB(25, 25, 112)
        activeTabName = name
    end)

    if not activeTabName then
        scrollingFrame.Visible = true
        tabButton.BackgroundColor3 = Color3.fromRGB(135, 206, 250)
        tabButton.TextColor3 = Color3.fromRGB(25, 25, 112)
        activeTabName = name
    end

    tabs[name] = {Page = scrollingFrame, Button = tabButton}
    return scrollingFrame
end

local silentAimTab = createTabPage("☁️ Silent Aim")
local aimAssistTab = createTabPage("✨ Hard Camlock")
local speedTab = createTabPage("🏃 Speed & Jump")
local flyTab = createTabPage("🦋 Stable Fly")
local tpTab = createTabPage("🌌 Teleport")
local espTab = createTabPage("🌸 Cute ESP")

-- ========================================================
-- 🛠️ UI WIDGET BUILDERS
-- ========================================================
local function createCard(parentTab, height)
    local card = Instance.new("Frame")
    card.Size = UDim2.new(1, -5, 0, height or 40)
    card.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    card.BorderSizePixel = 0
    card.Parent = parentTab
    
    local c = Instance.new("UICorner", card)
    c.CornerRadius = UDim.new(0, 8)
    
    local s = Instance.new("UIStroke", card)
    s.Color = Color3.fromRGB(215, 235, 245)
    s.Thickness = 1
    return card
end

local function addToggle(parentTab, text, default, callback)
    local card = createCard(parentTab, 40)
    local lbl = Instance.new("TextLabel", card)
    lbl.Size = UDim2.new(0.6, 0, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(70, 130, 180)
    lbl.Font = Enum.Font.GothamMedium
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", card)
    btn.Size = UDim2.new(0, 42, 0, 24)
    btn.Position = UDim2.new(1, -54, 0.5, -12)
    btn.BackgroundColor3 = default and Color3.fromRGB(135, 206, 250) or Color3.fromRGB(220, 220, 220)
    btn.Text = ""
    btn.AutoButtonColor = false

    local bc = Instance.new("UICorner", btn)
    bc.CornerRadius = UDim.new(1, 0)

    local state = default
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.BackgroundColor3 = state and Color3.fromRGB(135, 206, 250) or Color3.fromRGB(220, 220, 220)
        callback(state)
    end)
    return {
        SetState = function(sState)
            state = sState
            btn.BackgroundColor3 = state and Color3.fromRGB(135, 206, 250) or Color3.fromRGB(220, 220, 220)
            callback(state)
        end
    }
end

local function addSlider(parentTab, text, min, max, default, callback)
    local card = createCard(parentTab, 50)
    
    local lbl = Instance.new("TextLabel", card)
    lbl.Size = UDim2.new(0.6, 0, 0, 20)
    lbl.Position = UDim2.new(0, 10, 0, 4)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(70, 130, 180)
    lbl.Font = Enum.Font.GothamMedium
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local valLbl = Instance.new("TextLabel", card)
    valLbl.Size = UDim2.new(0.3, 0, 0, 20)
    valLbl.Position = UDim2.new(0.7, -10, 0, 4)
    valLbl.BackgroundTransparency = 1
    valLbl.Text = tostring(default)
    valLbl.TextColor3 = Color3.fromRGB(130, 160, 180)
    valLbl.Font = Enum.Font.GothamBold
    valLbl.TextSize = 11
    valLbl.TextXAlignment = Enum.TextXAlignment.Right

    local bg = Instance.new("Frame", card)
    bg.Size = UDim2.new(1, -20, 0, 6)
    bg.Position = UDim2.new(0, 10, 0, 32)
    bg.BackgroundColor3 = Color3.fromRGB(225, 235, 240)
    Instance.new("UICorner", bg).CornerRadius = UDim.new(1, 0)

    local fill = Instance.new("Frame", bg)
    fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(135, 206, 250)
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

    local sDragging = false
    local function update(input)
        local pos = math.clamp((input.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
        local val = math.floor(min + (max - min) * pos)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        valLbl.Text = tostring(val)
        callback(val)
    end

    bg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            sDragging = true
            update(input)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then sDragging = false end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if sDragging and input.UserInputType == Enum.UserInputType.MouseMovement then update(input) end
    end)
end

local function addDropdown(parentTab, text, list, default, callback)
    local card = createCard(parentTab, 40)
    
    local lbl = Instance.new("TextLabel", card)
    lbl.Size = UDim2.new(0.4, 0, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(70, 130, 180)
    lbl.Font = Enum.Font.GothamMedium
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", card)
    btn.Size = UDim2.new(0, 115, 0, 24)
    btn.Position = UDim2.new(1, -125, 0.5, -12)
    btn.BackgroundColor3 = Color3.fromRGB(235, 245, 255)
    btn.Text = default
    btn.TextColor3 = Color3.fromRGB(25, 25, 112)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 10
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    local currentIndex = 1
    for i, opt in ipairs(list) do
        if opt == default then currentIndex = i end
    end

    btn.MouseButton1Click:Connect(function()
        currentIndex = currentIndex + 1
        if currentIndex > #list then currentIndex = 1 end
        local chosen = list[currentIndex]
        btn.Text = chosen
        callback(chosen)
    end)
end

local function addKeybind(parentTab, text, defaultKey, callback)
    local card = createCard(parentTab, 40)
    local lbl = Instance.new("TextLabel", card)
    lbl.Size = UDim2.new(0.5, 0, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(70, 130, 180)
    lbl.Font = Enum.Font.GothamMedium
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", card)
    btn.Size = UDim2.new(0, 65, 0, 24)
    btn.Position = UDim2.new(1, -75, 0.5, -12)
    btn.BackgroundColor3 = Color3.fromRGB(235, 245, 255)
    btn.Text = defaultKey.Name
    btn.TextColor3 = Color3.fromRGB(25, 25, 112)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 10
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    local listening = false
    btn.MouseButton1Click:Connect(function()
        if listening then return end
        listening = true
        btn.Text = "..."
        local connection
        connection = UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Keyboard then
                callback(input.KeyCode)
                btn.Text = input.KeyCode.Name
                listening = false
                if connection then connection:Disconnect() end
            end
        end)
    end)
end

local function createNumericInput(parentTab, name, defaultVal, minVal, maxVal, callback)
    local Frame = createCard(parentTab, 40)
    local TextLabel = Instance.new("TextLabel", Frame)
    TextLabel.Size = UDim2.new(0.6, 0, 1, 0)
    TextLabel.Position = UDim2.new(0, 10, 0, 0)
    TextLabel.BackgroundTransparency = 1
    TextLabel.Font = Enum.Font.GothamMedium
    TextLabel.Text = name
    TextLabel.TextColor3 = Color3.fromRGB(70, 130, 180)
    TextLabel.TextSize = 11
    TextLabel.TextXAlignment = Enum.TextXAlignment.Left

    local TextBox = Instance.new("TextBox", Frame)
    TextBox.Size = UDim2.new(0, 65, 0, 24)
    TextBox.Position = UDim2.new(1, -75, 0.5, -12)
    TextBox.BackgroundColor3 = Color3.fromRGB(235, 245, 255)
    TextBox.BorderSizePixel = 0
    TextBox.Font = Enum.Font.GothamBold
    TextBox.Text = tostring(defaultVal)
    TextBox.TextColor3 = Color3.fromRGB(25, 25, 112)
    TextBox.TextSize = 11
    TextBox.ClearTextOnFocus = false
    Instance.new("UICorner", TextBox).CornerRadius = UDim.new(0, 6)

    TextBox.FocusLost:Connect(function()
        local num = tonumber(TextBox.Text)
        if num then
            num = math.clamp(num, minVal, maxVal)
            TextBox.Text = tostring(num)
            callback(num)
        else
            TextBox.Text = tostring(defaultVal)
        end
    end)
end

-- ========================================================
-- ⚡ POPULATE TABS & CONTROLS
-- ========================================================

-- 1. Silent Aim Tab
local silentAimToggleRef = nil
silentAimToggleRef = addToggle(silentAimTab, "Silent Aim Enabled", cfg.silentAim, function(v) cfg.silentAim = v end)
addToggle(silentAimTab, "Show FOV Circle", cfg.silentAimFOVShow, function(v) cfg.silentAimFOVShow = v end)
addSlider(silentAimTab, "FOV Size", 10, 500, cfg.silentAimFOV, function(v) cfg.silentAimFOV = v end)
addToggle(silentAimTab, "Bypass Revolver", cfg.bypassRevolver, function(v) cfg.bypassRevolver = v end)
addToggle(silentAimTab, "Wall Check", cfg.silentAimWallCheck, function(v) cfg.silentAimWallCheck = v end)
addToggle(silentAimTab, "Target Closest Part", cfg.silentAimClosestPart, function(v) cfg.silentAimClosestPart = v end)
addDropdown(silentAimTab, "HitPart", bodyPartsList, cfg.silentAimPart, function(v) cfg.silentAimPart = v end)
addToggle(silentAimTab, "Enable Keybind Toggle", cfg.useKeybind, function(v) cfg.useKeybind = v end)
addKeybind(silentAimTab, "Silent Aim Key", cfg.silentAimKey, function(v) cfg.silentAimKey = v end)
addKeybind(silentAimTab, "UI Toggle Key", cfg.uiToggleKey, function(v) cfg.uiToggleKey = v end)

-- 2. Hard Camlock Tab
addDropdown(aimAssistTab, "Target Hitpart", {"Head", "HumanoidRootPart", "UpperTorso"}, "Head", function(part)
    getgenv().SelectedAimHitpart = part
end)
local camlockToggleRef = nil
camlockToggleRef = addToggle(aimAssistTab, "Hard Camlock", false, function(state)
    getgenv().AimAssistEnabled = state
    if not state then lockedPlayer = nil end
end)
addKeybind(aimAssistTab, "Camlock Keybind", Enum.KeyCode.Q, function(v) getgenv().AimAssistKey = v end)
createNumericInput(aimAssistTab, "Lock Speed (1-20)", 2, 1, 20, function(val)
    getgenv().AimSmoothness = val
end)

-- 3. Speed & Jump Tab
addToggle(speedTab, "Speed Hack", false, function(state)
    getgenv().SpeedEnabled = state
end)
addKeybind(speedTab, "Speed Keybind", Enum.KeyCode.Z, function(v) getgenv().SpeedKey = v end)
createNumericInput(speedTab, "Speed Value", 50, 1, 300, function(val)
    getgenv().CustomSpeed = val
end)
createNumericInput(speedTab, "Jump Power", 50, 1, 300, function(val)
    getgenv().CustomJumpPower = val
end)

-- 4. Fly Tab
local function toggleFly(state)
    getgenv().FlyEnabled = state
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    
    if state then
        if hrp then
            if bodyVel then bodyVel:Destroy() end
            if bodyGyro then bodyGyro:Destroy() end
            
            bodyVel = Instance.new("BodyVelocity")
            bodyVel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bodyVel.Velocity = Vector3.new(0, 0, 0)
            bodyVel.Parent = hrp
            
            bodyGyro = Instance.new("BodyGyro")
            bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            bodyGyro.CFrame = hrp.CFrame
            bodyGyro.Parent = hrp
        end
        if hum then hum.PlatformStand = true end
        
        if flyConnection then flyConnection:Disconnect() end
        flyConnection = RunService.RenderStepped:Connect(function()
            local current_char = LocalPlayer.Character
            local current_hrp = current_char and current_char:FindFirstChild("HumanoidRootPart")
            if not getgenv().FlyEnabled or not current_hrp or not bodyVel or not bodyGyro then return end
            
            local camCF = Camera.CFrame
            local moveDir = Vector3.new(0, 0, 0)
            
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + camCF.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - camCF.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - camCF.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + camCF.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0, 1, 0) end

            if moveDir.Magnitude > 0 then
                bodyVel.Velocity = moveDir.Unit * getgenv().FlySpeed
            else
                bodyVel.Velocity = Vector3.new(0, 0, 0)
            end
            bodyGyro.CFrame = camCF
        end)
    else
        if flyConnection then flyConnection:Disconnect(); flyConnection = nil end
        if bodyVel then bodyVel:Destroy(); bodyVel = nil end
        if bodyGyro then bodyGyro:Destroy(); bodyGyro = nil end
        if hum then hum.PlatformStand = false end
    end
end

addToggle(flyTab, "Stable Fly Mode", false, toggleFly)
addKeybind(flyTab, "Fly Keybind", Enum.KeyCode.C, function(v) getgenv().FlyKey = v end)
createNumericInput(flyTab, "Fly Speed", 60, 1, 500, function(val)
    getgenv().FlySpeed = val
end)

-- 5. Teleport Tab
local function getPlayerNames()
    local names = {"Select Player"}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then table.insert(names, p.Name) end
    end
    return names
end

addDropdown(tpTab, "Target Player", getPlayerNames(), "Select Player", function(name)
    getgenv().SelectedTeleportTargetName = name
end)
addKeybind(tpTab, "Teleport Keybind", Enum.KeyCode.X, function(v) getgenv().TeleportKey = v end)
addToggle(tpTab, "Teleport Loop", false, function(state)
    getgenv().TeleportLoopEnabled = state
end)

local tpButtonFrame = createCard(tpTab, 40)
local tpBText = Instance.new("TextLabel", tpButtonFrame)
tpBText.Size = UDim2.new(0.5, 0, 1, 0)
tpBText.Position = UDim2.new(0, 10, 0, 0)
tpBText.BackgroundTransparency = 1
tpBText.Font = Enum.Font.GothamMedium
tpBText.Text = "Instant Teleport"
tpBText.TextColor3 = Color3.fromRGB(70, 130, 180)
tpBText.TextSize = 11
tpBText.TextXAlignment = Enum.TextXAlignment.Left

local ExecTpButton = Instance.new("TextButton", tpButtonFrame)
ExecTpButton.Size = UDim2.new(0, 75, 0, 24)
ExecTpButton.Position = UDim2.new(1, -85, 0.5, -12)
ExecTpButton.BackgroundColor3 = Color3.fromRGB(135, 206, 250)
ExecTpButton.Font = Enum.Font.GothamBold
ExecTpButton.Text = "Go!"
ExecTpButton.TextColor3 = Color3.fromRGB(25, 25, 112)
ExecTpButton.TextSize = 11
Instance.new("UICorner", ExecTpButton).CornerRadius = UDim.new(0, 6)

local function executeTeleport()
    local targetName = getgenv().SelectedTeleportTargetName
    if not targetName or targetName == "Select Player" then return end
    local targetPlayer = Players:FindFirstChild(targetName)
    if targetPlayer and targetPlayer.Character then
        local targetPart = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if targetPart and myRoot then
            myRoot.CFrame = targetPart.CFrame + Vector3.new(0, 3, 0)
        end
    end
end
ExecTpButton.MouseButton1Click:Connect(executeTeleport)

-- 6. ESP Tab
addToggle(espTab, "Cinnamoroll Blue ESP", false, function(state)
    getgenv().EspEnabled = state
    if not state then
        for _, obj in pairs(espObjects) do
            if obj then obj:Remove() end
        end
        espObjects = {}
    end
end)

-- ========================================================
-- ⚙️ GLOBAL RUNTIME & KEYBIND LISTENER
-- ========================================================
local function unloadScript()
    cfg.silentAim = false
    getgenv().AimAssistEnabled = false
    getgenv().SpeedEnabled = false
    getgenv().FlyEnabled = false
    getgenv().TeleportLoopEnabled = false
    getgenv().EspEnabled = false
    
    if flyConnection then flyConnection:Disconnect() end
    if bodyVel then bodyVel:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
    if fovCircle then fovCircle:Remove() end
    if RiaHub then RiaHub:Destroy() end
end

UnloadButton.MouseButton1Click:Connect(unloadScript)

UserInputService.InputBegan:Connect(function(input, gpe)
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if input.KeyCode == cfg.uiToggleKey then
            MainFrame.Visible = not MainFrame.Visible
        end
        
        if not gpe then
            if cfg.useKeybind and input.KeyCode == cfg.silentAimKey then
                cfg.silentAim = not cfg.silentAim
                if silentAimToggleRef then silentAimToggleRef.SetState(cfg.silentAim) end
            elseif input.KeyCode == getgenv().AimAssistKey then
                getgenv().AimAssistEnabled = not getgenv().AimAssistEnabled
                if camlockToggleRef then camlockToggleRef.SetState(getgenv().AimAssistEnabled) end
                if not getgenv().AimAssistEnabled then lockedPlayer = nil end
            elseif input.KeyCode == getgenv().SpeedKey then
                getgenv().SpeedEnabled = not getgenv().SpeedEnabled
            elseif input.KeyCode == getgenv().FlyKey then
                toggleFly(not getgenv().FlyEnabled)
            elseif input.KeyCode == getgenv().TeleportKey then
                executeTeleport()
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    local mousePos = UserInputService:GetMouseLocation()
    fovCircle.Position = mousePos
    fovCircle.Radius = cfg.silentAimFOV
    fovCircle.Color = cfg.silentAimFOVColor
    fovCircle.Visible = cfg.silentAim and cfg.silentAimFOVShow and not isHoldingRevolver()

    if cfg.silentAim then
        silentAimCachedPart = getClosestPlayerToCursor()
    else
        silentAimCachedPart = nil
    end

    if getgenv().SpeedEnabled and LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.WalkSpeed = getgenv().CustomSpeed
            hum.JumpPower = getgenv().CustomJumpPower
        end
    end

    if getgenv().TeleportLoopEnabled then
        executeTeleport()
    end

    -- Camlock Logic
    if getgenv().AimAssistEnabled then
        local hitpartName = getgenv().SelectedAimHitpart
        if not lockedPlayer or not (lockedPlayer.Character and lockedPlayer.Character:FindFirstChildOfClass("Humanoid") and lockedPlayer.Character.Humanoid.Health > 0) then
            local closestDist = math.huge
            local bestPlayer = nil
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    local hum = player.Character:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        local pPart = player.Character:FindFirstChild(hitpartName) or player.Character:FindFirstChild("HumanoidRootPart")
                        if pPart then
                            local screenPos, onScreen = Camera:WorldToViewportPoint(pPart.Position)
                            local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                            if screenDist < closestDist then
                                closestDist = screenDist
                                bestPlayer = player
                            end
                        end
                    end
                end
            end
            lockedPlayer = bestPlayer
        end
        
        if lockedPlayer and lockedPlayer.Character then
            local targetPart = lockedPlayer.Character:FindFirstChild(hitpartName) or lockedPlayer.Character:FindFirstChild("HumanoidRootPart")
            if targetPart then
                local currentCF = Camera.CFrame
                local targetCF = CFrame.new(currentCF.Position, targetPart.Position)
                local smoothness = math.clamp(getgenv().AimSmoothness, 1, 20)
                Camera.CFrame = currentCF:Lerp(targetCF, 1 / smoothness)
            end
        end
    else
        lockedPlayer = nil
    end
end)

print("✨ Ria Executed Successfully! Press Right Control to toggle UI.")
