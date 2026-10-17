
-- MM2 Visual Pack | ESP + Black GUI + 4:3 Stretch
-- Вставить в Xeno -> Execute

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ========== GUI ==========
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MM2VisualPack"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 240, 0, 200)
MainFrame.Position = UDim2.new(0, 20, 0.5, -100)
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
MainFrame.BorderSizePixel = 2
MainFrame.BorderColor3 = Color3.fromRGB(255, 255, 255)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Text = "MM2 VISUAL PACK"
Title.Font = Enum.Font.Code
Title.TextSize = 14
Title.Parent = MainFrame

local function makeButton(text, yPos)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 32)
    btn.Position = UDim2.new(0.05, 0, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    btn.BorderColor3 = Color3.fromRGB(255, 255, 255)
    btn.BorderSizePixel = 1
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Text = text
    btn.Font = Enum.Font.Code
    btn.TextSize = 12
    btn.Parent = MainFrame
    return btn
end

local ESPBtn = makeButton("ESP: OFF", 40)
local StretchBtn = makeButton("4:3 STRETCH: OFF", 80)
local Status = makeButton("STATUS: READY", 120)
Status.BackgroundTransparency = 1
Status.BorderSizePixel = 0
Status.TextColor3 = Color3.fromRGB(150, 150, 150)

-- ========== ESP ==========
local espEnabled = false
local highlights = {}

local function getRoleColor(player)
    local hasKnife = player.Backpack:FindFirstChild("Knife") or (player.Character and player.Character:FindFirstChild("Knife"))
    local hasGun = player.Backpack:FindFirstChild("Gun") or (player.Character and player.Character:FindFirstChild("Gun"))
    if hasKnife then return Color3.fromRGB(255, 0, 0) end
    if hasGun then return Color3.fromRGB(0, 100, 255) end
    return Color3.fromRGB(0, 255, 0)
end

local function applyESP(player)
    if player == LocalPlayer or not player.Character then return end
    local existing = player.Character:FindFirstChild("MM2ESP")
    local color = getRoleColor(player)
    if existing then
        existing.FillColor = color
        existing.OutlineColor = color
        return
    end
    local h = Instance.new("Highlight")
    h.Name = "MM2ESP"
    h.Parent = player.Character
    h.Adornee = player.Character
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.FillTransparency = 0.4
    h.OutlineTransparency = 0
    h.FillColor = color
    h.OutlineColor = color
    highlights[player] = h
end

local function clearESP()
    for _, h in pairs(highlights) do
        if h and h.Parent then h:Destroy() end
    end
    highlights = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character then
            local h = p.Character:FindFirstChild("MM2ESP")
            if h then h:Destroy() end
        end
    end
end

ESPBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    ESPBtn.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
    ESPBtn.TextColor3 = espEnabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 255)
    if espEnabled then
        for _, p in pairs(Players:GetPlayers()) do applyESP(p) end
    else
        clearESP()
    end
end)

task.spawn(function()
    while true do
        task.wait(0.5)
        if espEnabled then
            for _, p in pairs(Players:GetPlayers()) do applyESP(p) end
        end
    end
end)

-- ========== 4:3 STRETCH ==========
local stretchEnabled = false
local stretchConn = nil

local function toggleStretch(on)
    if stretchConn then stretchConn:Disconnect() stretchConn = nil end
    stretchEnabled = on
    if not on then return end
    local ratio = 4 / 3
    stretchConn = RunService.RenderStepped:Connect(function()
        local vp = Camera.ViewportSize
        local cur = vp.X / vp.Y
        local sx = cur / ratio
        Camera.CFrame = Camera.CFrame * CFrame.new(0, 0, 0, sx, 0, 0, 0, 1, 0, 0, 0, 1)
    end)
end

StretchBtn.MouseButton1Click:Connect(function()
    toggleStretch(not stretchEnabled)
    StretchBtn.Text = "4:3 STRETCH: " .. (stretchEnabled and "ON" or "OFF")
    StretchBtn.TextColor3 = stretchEnabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 255)
end)

-- ========== RESPAWN ==========
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if espEnabled then
        for _, p in pairs(Players:GetPlayers()) do applyESP(p) end
    end
end)

Status.Text = "STATUS: LOADED"
print("[MM2 Visual Pack] готов. ESP + GUI + 4:3 stretch.")
