local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Smiley_hacker gui😈",
   Icon = 0,
   LoadingTitle = "SMILEY_HACKER😁",
   LoadingSubtitle = "by smiley_hacker",
   ShowText = "SMILEY_HACKER😈",
   Theme = "Default",
   ToggleUIKeybind = Enum.KeyCode.K,
   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false,
   ConfigurationSaving = {
      Enabled = false,
      FolderName = nil,
      FileName = "Big Hub"
   },
   Discord = {
      Enabled = false,
      Invite = "noinvitelink",
      RememberJoins = true
   },
   KeySystem = true,
   KeySettings = {
      Title = "smiley_hackers key system",
      Subtitle = ":)",
      Note = "https://pastebin.com/y1eeMJqU",
      FileName = "smiley_hacker",
      SaveKey = false,
      GrabKeyFromSite = false,
      Key = {"smile:)"}
   }
})

-- TABS & SECTIONS
local AimTab = Window:CreateTab("AimBot Guis 🔫", nil)
local AimSection = AimTab:CreateSection("AimBot 🔫")

local VisualTab = Window:CreateTab("Visual Guis👀", nil)
local VisualSection = VisualTab:CreateSection("ESP Guis")

local MovementTab = Window:CreateTab("Movement", nil)
local MovementSection = MovementTab:CreateSection("WalkSpeed")

local GuisTab = Window:CreateTab("Other Guis", nil)
local GuisSection = GuisTab:CreateSection("OP Guis")
-- ==============================================
-- FIXED ESP
-- ==============================================
local hue = 0
local rainbowSpeed = 2
local espRunning = false
local espConnections = {}
local espObjects = {}
local currentRainbowColor = Color3.new(1, 0, 0)

local function getRainbowColor(deltaTime)
    hue = (hue + deltaTime * rainbowSpeed) % 1
    return Color3.fromHSV(hue, 1, 1)
end

local function startESP()
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LocalPlayer = Players.LocalPlayer
    local Camera = workspace.CurrentCamera

    local ESP = {
        Enabled = true,
        ShowTeamColor = false,
        ShowHealthBar = true,
        TextSize = 14,
        BoxThickness = 2,
        BoxTransparency = 0.5,
        TextTransparency = 0,
        HealthBarThickness = 2,
        HealthBarWidth = 50,
        HealthBarHeight = 5,
        MaxDistance = 5000,
    }

    local function CreateESPForPlayer(player)
        if player == LocalPlayer then return end

        local espObject = {}
        espObject.Box = Drawing.new("Square")
        espObject.Box.Visible = false
        espObject.Box.Thickness = ESP.BoxThickness
        espObject.Box.Transparency = ESP.BoxTransparency
        espObject.Box.Filled = false

        espObject.Text = Drawing.new("Text")
        espObject.Text.Visible = false
        espObject.Text.Size = ESP.TextSize
        espObject.Text.Center = true
        espObject.Text.Outline = true
        espObject.Text.Transparency = ESP.TextTransparency

        espObject.HealthBG = Drawing.new("Square")
        espObject.HealthBG.Visible = false
        espObject.HealthBG.Thickness = ESP.HealthBarThickness
        espObject.HealthBG.Filled = true
        espObject.HealthBG.Color = Color3.fromRGB(100, 100, 100)
        espObject.HealthBG.Transparency = 0.7

        espObject.HealthBar = Drawing.new("Square")
        espObject.HealthBar.Visible = false
        espObject.HealthBar.Thickness = ESP.HealthBarThickness
        espObject.HealthBar.Filled = true
        espObject.HealthBar.Transparency = 0.7

        espObjects[player] = espObject

        table.insert(espConnections, player.AncestryChanged:Connect(function()
            if not player:IsDescendantOf(game) then
                if espObjects[player] then
                    for _, object in pairs(espObjects[player]) do
                        object:Remove()
                    end
                    espObjects[player] = nil
                end
            end
        end))
    end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then CreateESPForPlayer(player) end
    end

    table.insert(espConnections, Players.PlayerAdded:Connect(CreateESPForPlayer))
    table.insert(espConnections, Players.PlayerRemoving:Connect(function(player)
        if espObjects[player] then
            for _, object in pairs(espObjects[player]) do object:Remove() end
            espObjects[player] = nil
        end
    end))

    local function GetHealthColor(health, maxHealth)
        local percent = health / maxHealth
        return Color3.new(math.clamp(2*(1-percent),0,1), math.clamp(2*percent,0,1), 0)
    end

    local renderConn = RunService.RenderStepped:Connect(function(deltaTime)
        currentRainbowColor = getRainbowColor(deltaTime)
        if not ESP.Enabled then
            for _, obj in pairs(espObjects) do for _, d in pairs(obj) do d.Visible = false end end
            return
        end

        for player, obj in pairs(espObjects) do
            local char = player.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local humanoid = char and char:FindFirstChild("Humanoid")
            if not root or not humanoid then
                for _, d in pairs(obj) do d.Visible = false end
                continue
            end

            local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
            local dist = (root.Position - Camera.CFrame.Position).Magnitude
            if not onScreen or dist > ESP.MaxDistance then
                for _, d in pairs(obj) do d.Visible = false end
                continue
            end

            local topVec = Camera:WorldToViewportPoint(root.Position + Vector3.new(0,3,0))
            local bottomVec = Camera:WorldToViewportPoint(root.Position - Vector3.new(0,3,0))
            local height = math.abs(topVec.Y - bottomVec.Y)
            local width = height * 0.6

            obj.Box.Position = Vector2.new(pos.X - width/2, pos.Y - height/2)
            obj.Box.Size = Vector2.new(width, height)
            obj.Box.Color = currentRainbowColor
            obj.Box.Visible = true

            obj.Text.Position = Vector2.new(pos.X, pos.Y - height/2 - 20)
            obj.Text.Text = string.format("%s [%.0f%%]", player.Name, (humanoid.Health/humanoid.MaxHealth)*100)
            obj.Text.Color = currentRainbowColor
            obj.Text.Visible = true

            obj.HealthBG.Position = Vector2.new(pos.X - ESP.HealthBarWidth/2, pos.Y - height/2 - 10)
            obj.HealthBG.Size = Vector2.new(ESP.HealthBarWidth, ESP.HealthBarHeight)
            obj.HealthBG.Visible = ESP.ShowHealthBar

            local hpPercent = humanoid.Health/humanoid.MaxHealth
            obj.HealthBar.Position = Vector2.new(pos.X - ESP.HealthBarWidth/2, pos.Y - height/2 - 10)
            obj.HealthBar.Size = Vector2.new(ESP.HealthBarWidth * hpPercent, ESP.HealthBarHeight)
            obj.HealthBar.Color = GetHealthColor(humanoid.Health, humanoid.MaxHealth)
            obj.HealthBar.Visible = ESP.ShowHealthBar
        end
    end)
    table.insert(espConnections, renderConn)
    print("Rainbow ESP loaded!")
end

local function stopESP()
    for _, obj in pairs(espObjects) do for _, d in pairs(obj) do d:Remove() end end
    espObjects = {}
    for _, conn in pairs(espConnections) do conn:Disconnect() end
    espConnections = {}
    espRunning = false
    print("Rainbow ESP unloaded!")
end

local VisualToggle = VisualTab:CreateToggle({
   Name = "Smiley_hackers ESP",
   CurrentValue = false,
   Flag = "ESP_Toggle", -- UNIQUE FLAG
   Callback = function(Value)
        if Value and not espRunning then
            espRunning = true
            startESP()
        elseif not Value and espRunning then
            stopESP()
        end
   end,
})

-- ==============================================
-- AIMBOT BUTTONS
-- ==============================================
local AimButton1 = AimTab:CreateButton({
   Name = "Equinox AimBot Gui",
   Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Xxtan31/Equinox-Hub/main/Aimbots/directions.lua", true))()
   end,
})

local AimButton2 = AimTab:CreateButton({
   Name = "LunaHub🔥",
   Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/UnoXLuna/Luna-Hub/refs/heads/main/Script-Loader"))()
   end,
})

-- ==============================================
-- WALKspeed SLIDER & INFINITE JUMP
-- ==============================================
local MovementSlider = MovementTab:CreateSlider({
   Name = "WalkSpeed",
   Range = {0, 600},
   Increment = 1,
   Suffix = "WalkSpeed",
   CurrentValue = 16,
   Flag = "WalkSpeed_Slider", -- UNIQUE FLAG
   Callback = function(Value)
        local localPlayer = game.Players.LocalPlayer
        local char = localPlayer.Character or localPlayer.CharacterAdded:Wait()
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then humanoid.WalkSpeed = Value end
   end,
})

local MovementToggle_InfJump = MovementTab:CreateToggle({ -- UNIQUE NAME
   Name = "inf jump",
   CurrentValue = false,
   Flag = "InfJump_Toggle", -- UNIQUE FLAG
   Callback = function(Value)
        local UserInputService = game:GetService("UserInputService")
        local localPlayer = game.Players.LocalPlayer

        -- Disconnect existing connection first to avoid duplicates
        if _G.InfJumpConn then _G.InfJumpConn:Disconnect() end

        if Value then
            _G.InfJumpConn = UserInputService.JumpRequest:Connect(function()
                local char = localPlayer.Character
                local humanoid = char and char:FindFirstChildOfClass("Humanoid")
                if humanoid then humanoid:ChangeState("Jumping") end
            end)
        end
   end,
})

-- ==============================================
-- FIXED NOCLIP (GUI TOGGLE ONLY - NO EXTRA BUTTON/KEYBIND)
-- ==============================================
local noclipEnabled = false
local noclipRenderConn = nil -- Track render connection to avoid duplicates

local MovementToggle_NoClip = MovementTab:CreateToggle({ -- UNIQUE NAME
   Name = "NoClip",
   CurrentValue = false,
   Flag = "NoClip_Toggle", -- UNIQUE FLAG
   Callback = function(Value)
        noclipEnabled = Value
        local localPlayer = game.Players.LocalPlayer

        -- Disconnect existing render connection if it exists
        if noclipRenderConn then
            noclipRenderConn:Disconnect()
            noclipRenderConn = nil
        end

        -- If NoClip is on, start render loop; if off, reset collision
        if Value then
            local RunService = game:GetService("RunService")
            noclipRenderConn = RunService.RenderStepped:Connect(function()
                local char = localPlayer.Character
                if not char then return end
                -- Efficiently set CanCollide off for all BaseParts (no loop every wait())
                for _, part in pairs(char:GetChildren()) do
                    if part:IsA("BasePart") then part.CanCollide = false end
                end
            end)
        else
            local char = localPlayer.Character
            if char then
                for _, part in pairs(char:GetChildren()) do
                    if part:IsA("BasePart") then part.CanCollide = true end
                end
                end
        end
   end,
})

local GuisButton = GuisTab:CreateButton({
   Name = "Infinite Yield",
   Callback = function()
   loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
   end,
})
local MovementButton = MovementTab:CreateButton({
   Name = "Fly Gui v3",
   Callback = function()
   --[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
loadstring(game:HttpGet("https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"))()
   end,
})
local GuisButton = GuisTab:CreateButton({
   Name = "OP Troll is a pinning tower Gui",
   Callback = function()
   --[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
loadstring(game:HttpGet("https://raw.githubusercontent.com/aaaa45451619/SLAPP-all-prayer/refs/heads/145/Troll%20Tower%202%20Free"))()
   end,
})
local GuisButton = GuisTab:CreateButton({
   Name = "AirHub Gui",
   Callback = function()
   loadstring(game:HttpGet("https://raw.githubusercontent.com/Exunys/AirHub/main/AirHub.lua"))()
   end,
})

local FOVSlider = MovementTab:CreateSlider({
   Name = "Field of View",
   Range = {30, 120},
   Increment = 1,
   Suffix = "FOV",
   CurrentValue = 70,
   Flag = "FOV_Slider",
   Callback = function(Value)
        workspace.CurrentCamera.FieldOfView = Value
   end,
})

-- ==============================================
-- SPIN BOT + FOV + SPEED
-- ==============================================
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local spinConnection
local spinningEnabled = false

local TARGET_FOV = 120
local TARGET_SPEED = 60

-- Apply movement boosts
local function applyBoosts()
    local cam = Workspace.CurrentCamera
    if cam then
        cam.FieldOfView = TARGET_FOV
    end

    local char = player.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.WalkSpeed = TARGET_SPEED
        end
    end
end

-- Start spinning
local function startSpin()
    local char = player.Character
    if not char then return end

    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local camera = Workspace.CurrentCamera

    -- Camera zoom
    player.CameraMode = Enum.CameraMode.Classic
    camera.CameraType = Enum.CameraType.Custom
    player.CameraMinZoomDistance = 15
    player.CameraMaxZoomDistance = 15

    applyBoosts()

    if spinConnection then
        spinConnection:Disconnect()
    end

    spinConnection = RunService.RenderStepped:Connect(function(dt)
        if root and spinningEnabled then
            root.CFrame = root.CFrame * CFrame.Angles(0, math.rad(99999)*dt, 0)
        end
    end)
end

-- Reapply after respawn
player.CharacterAdded:Connect(function()
    if spinningEnabled then
        task.wait(0.5)
        startSpin()
    end
end)

-- Rayfield Toggle
local MovementToggle_Spin = MovementTab:CreateToggle({
   Name = "Spin Bot",
   CurrentValue = false,
   Flag = "Spin_Toggle",
   Callback = function(Value)
        spinningEnabled = Value

        if Value then
            startSpin()
        else
            if spinConnection then
                spinConnection:Disconnect()
                spinConnection = nil
            end

            -- Restore defaults
            local cam = Workspace.CurrentCamera
            if cam then
                cam.FieldOfView = 70
            end

            local char = player.Character
            if char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then
                    hum.WalkSpeed = 16
                end
            end

            player.CameraMinZoomDistance = 0.5
            player.CameraMaxZoomDistance = 128
        end
   end,
})

local Button = GuisTab:CreateButton({
Name = "dmon gui",
Callback = function()
loadstring(game:HttpGet("http://dmonmods.xyz/loader.txt"))()
end,
})

local AimButton = AimTab:CreateButton({
   Name = " OP!!!",
   Callback = function()
       --[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
loadstring(game:HttpGet("https://pastebin.com/raw/uBsYYmVP"))()
   end,
})

local AimButton = AimTab:CreateButton({
   Name = "ai aimbot",
   Callback = function()
       --[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
loadstring(game:HttpGet("https://pastebin.com/raw/sgkRrt3c"))("t.me/ByteScripts")
   end,
})

local AimButton = AimTab:CreateButton({
   Name = "nexis aimbot",
   Callback = function()
       --// Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-------------------------------------------------
-- GUI SETUP
-------------------------------------------------
local gui = Instance.new("ScreenGui", Player:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false

local openBtn = Instance.new("TextButton", gui)
openBtn.Size = UDim2.new(0, 60, 0, 28)
openBtn.Position = UDim2.new(0, 10, 0, 120)
openBtn.Text = "Open"
openBtn.BackgroundColor3 = Color3.fromRGB(80, 40, 110)
openBtn.TextColor3 = Color3.new(1, 1, 1)
openBtn.TextScaled = true

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 230, 0, 210)
main.Position = UDim2.new(0.5, -115, 0.5, -105)
main.BackgroundColor3 = Color3.fromRGB(40, 20, 60)
main.Visible = false

local closeBtn = Instance.new("TextButton", main)
closeBtn.Size = UDim2.new(0, 22, 0, 22)
closeBtn.Position = UDim2.new(1, -25, 0, 3)
closeBtn.Text = "X"
closeBtn.BackgroundColor3 = Color3.fromRGB(120, 60, 160)
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.TextScaled = true

openBtn.MouseButton1Click:Connect(function() main.Visible = true end)
closeBtn.MouseButton1Click:Connect(function() main.Visible = false end)

local welcome = Instance.new("TextLabel", main)
welcome.Size = UDim2.new(1, -30, 0, 24)
welcome.Position = UDim2.new(0, 6, 0, 3)
welcome.BackgroundTransparency = 1
welcome.TextScaled = true
welcome.TextColor3 = Color3.fromRGB(210, 170, 255)
welcome.Text = "Welcome " .. Player.Name

-------------------------------------------------
-- Tabs
-------------------------------------------------
local tabs = Instance.new("Frame", main)
tabs.Size = UDim2.new(0, 70, 1, -30)
tabs.Position = UDim2.new(0, 0, 0, 30)
tabs.BackgroundColor3 = Color3.fromRGB(55, 30, 80)

local function tabBtn(name, y)
	local b = Instance.new("TextButton", tabs)
	b.Size = UDim2.new(1, 0, 0, 28)
	b.Position = UDim2.new(0, 0, 0, y)
	b.Text = name
	b.TextScaled = true
	b.BackgroundColor3 = Color3.fromRGB(80, 40, 110)
	b.TextColor3 = Color3.new(1, 1, 1)
	return b
end

local aimBtn = tabBtn("Aim", 0)
local visBtn = tabBtn("Visuals", 30)
local otherBtn = tabBtn("Other", 60)

local function page()
	local f = Instance.new("Frame", main)
	f.Size = UDim2.new(1, -70, 1, -30)
	f.Position = UDim2.new(0, 70, 0, 30)
	f.BackgroundTransparency = 1
	f.Visible = false
	return f
end

local aimPage = page()
local visPage = page()
local otherPage = page()
aimPage.Visible = true

aimBtn.MouseButton1Click:Connect(function()
	aimPage.Visible = true visPage.Visible = false otherPage.Visible = false
end)
visBtn.MouseButton1Click:Connect(function()
	aimPage.Visible = false visPage.Visible = true otherPage.Visible = false
end)
otherBtn.MouseButton1Click:Connect(function()
	aimPage.Visible = false visPage.Visible = false otherPage.Visible = true
end)

-------------------------------------------------
-- AIM TAB
-------------------------------------------------
local fovToggle = Instance.new("TextButton", aimPage)
fovToggle.Size = UDim2.new(0, 90, 0, 24)
fovToggle.Position = UDim2.new(0, 6, 0, 6)
fovToggle.Text = "FOV : OFF"
fovToggle.BackgroundColor3 = Color3.fromRGB(80, 40, 110)
fovToggle.TextColor3 = Color3.new(1, 1, 1)
fovToggle.TextScaled = true

local fovBox = Instance.new("TextBox", aimPage)
fovBox.Size = UDim2.new(0, 70, 0, 22)
fovBox.Position = UDim2.new(0, 6, 0, 34)
fovBox.PlaceholderText = "Size"
fovBox.BackgroundColor3 = Color3.fromRGB(70, 40, 90)
fovBox.TextColor3 = Color3.new(1, 1, 1)
fovBox.TextScaled = true

local setBtn = Instance.new("TextButton", aimPage)
setBtn.Size = UDim2.new(0, 40, 0, 22)
setBtn.Position = UDim2.new(0, 80, 0, 34)
setBtn.Text = "Set"
setBtn.BackgroundColor3 = Color3.fromRGB(90, 50, 120)
setBtn.TextColor3 = Color3.new(1, 1, 1)
setBtn.TextScaled = true

local aimToggle = Instance.new("TextButton", aimPage)
aimToggle.Size = UDim2.new(0, 90, 0, 24)
aimToggle.Position = UDim2.new(0, 6, 0, 70)
aimToggle.Text = "Aimlock : OFF"
aimToggle.BackgroundColor3 = Color3.fromRGB(80, 40, 110)
aimToggle.TextColor3 = Color3.new(1, 1, 1)
aimToggle.TextScaled = true

local sliderBG = Instance.new("Frame", aimPage)
sliderBG.Size = UDim2.new(0, 110, 0, 6)
sliderBG.Position = UDim2.new(0, 6, 0, 100)
sliderBG.BackgroundColor3 = Color3.fromRGB(70, 40, 90)

local slider = Instance.new("TextButton", sliderBG)
slider.Size = UDim2.new(0, 12, 0, 12)
slider.Position = UDim2.new(0, 0, -0.5, 0)
slider.BackgroundColor3 = Color3.fromRGB(200, 150, 255)
slider.Text = ""

-------------------------------------------------
-- VISUALS TAB
-------------------------------------------------
local boxToggle = Instance.new("TextButton", visPage)
boxToggle.Size = UDim2.new(0, 90, 0, 24)
boxToggle.Position = UDim2.new(0, 6, 0, 6)
boxToggle.Text = "Box : OFF"
boxToggle.BackgroundColor3 = Color3.fromRGB(80, 40, 110)
boxToggle.TextColor3 = Color3.new(1, 1, 1)
boxToggle.TextScaled = true

local healthToggle = Instance.new("TextButton", visPage)
healthToggle.Size = UDim2.new(0, 90, 0, 24)
healthToggle.Position = UDim2.new(0, 6, 0, 36)
healthToggle.Text = "Health : OFF"
healthToggle.BackgroundColor3 = Color3.fromRGB(80, 40, 110)
healthToggle.TextColor3 = Color3.new(1, 1, 1)
healthToggle.TextScaled = true

local studsToggle = Instance.new("TextButton", visPage)
studsToggle.Size = UDim2.new(0, 90, 0, 24)
studsToggle.Position = UDim2.new(0, 6, 0, 66)
studsToggle.Text = "Studs : OFF"
studsToggle.BackgroundColor3 = Color3.fromRGB(80, 40, 110)
studsToggle.TextColor3 = Color3.new(1, 1, 1)
studsToggle.TextScaled = true

-------------------------------------------------
-- LOGIC SETUP
-------------------------------------------------
local fovOn, aimOn = false, false
local fovSize, aimStrength = 120, 0.08
local espColor = Color3.fromRGB(255, 255, 255)

local circle = Drawing.new("Circle")
circle.Visible = false
circle.Thickness = 1
circle.Filled = false -- Hollow circle
circle.Radius = fovSize
circle.Color = Color3.new(1, 1, 1)

fovToggle.MouseButton1Click:Connect(function()
	fovOn = not fovOn
	circle.Visible = fovOn
	fovToggle.Text = "FOV : " .. (fovOn and "ON" or "OFF")
end)

aimToggle.MouseButton1Click:Connect(function()
	aimOn = not aimOn
	aimToggle.Text = "Aimlock : " .. (aimOn and "ON" or "OFF")
end)

setBtn.MouseButton1Click:Connect(function()
	local n = tonumber(fovBox.Text)
	if n then fovSize = n circle.Radius = n end
end)

local dragging = false
slider.MouseButton1Down:Connect(function() dragging = true end)
UIS.InputEnded:Connect(function() dragging = false end)
UIS.InputChanged:Connect(function(i)
	if dragging then
		local pos = math.clamp((i.Position.X - sliderBG.AbsolutePosition.X) / sliderBG.AbsoluteSize.X, 0, 1)
		slider.Position = UDim2.new(pos, -6, -0.5, 0)
		aimStrength = pos
	end
end)

local boxes, healthbars, texts = {}, {}, {}
local function makeDrawings(plr)
	if plr == Player then return end
	boxes[plr] = Drawing.new("Square")
	boxes[plr].Thickness, boxes[plr].Filled = 1, false
	healthbars[plr] = Drawing.new("Line")
	texts[plr] = Drawing.new("Text")
	texts[plr].Size, texts[plr].Center, texts[plr].Outline = 13, true, true
end

for _, p in pairs(Players:GetPlayers()) do makeDrawings(p) end
Players.PlayerAdded:Connect(makeDrawings)

Players.PlayerRemoving:Connect(function(p)
	if boxes[p] then boxes[p]:Remove() boxes[p] = nil end
	if healthbars[p] then healthbars[p]:Remove() healthbars[p] = nil end
	if texts[p] then texts[p]:Remove() texts[p] = nil end
end)

local boxOn, hpOn, studsOn = false, false, false
boxToggle.MouseButton1Click:Connect(function() boxOn = not boxOn boxToggle.Text = "Box : " .. (boxOn and "ON" or "OFF") end)
healthToggle.MouseButton1Click:Connect(function() hpOn = not hpOn healthToggle.Text = "Health : " .. (hpOn and "ON" or "OFF") end)
studsToggle.MouseButton1Click:Connect(function() studsOn = not studsOn studsToggle.Text = "Studs : " .. (studsOn and "ON" or "OFF") end)

-------------------------------------------------
-- MAIN LOOP
-------------------------------------------------
RunService.RenderStepped:Connect(function()
	circle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

	for plr, box in pairs(boxes) do
		local char = plr.Character
		local root = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChild("Humanoid")

		if char and root and hum and hum.Health > 0 then
			local rootPos, visible = Camera:WorldToViewportPoint(root.Position)
			
			if visible then
				-- Precise scaling for perfect fit
				local head = char:FindFirstChild("Head")
				if head then
					local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
					local legPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))
					
					local h = math.abs(headPos.Y - legPos.Y)
					local w = h * 0.6 -- Standard character width ratio
					
					box.Visible = boxOn
					box.Size = Vector2.new(w, h)
					box.Position = Vector2.new(rootPos.X - w / 2, rootPos.Y - h / 2)
					box.Color = espColor

					if hpOn then
						local pct = hum.Health / hum.MaxHealth
						healthbars[plr].Visible = true
						healthbars[plr].From = Vector2.new(rootPos.X - w / 2 - 4, rootPos.Y + h / 2)
						healthbars[plr].To = Vector2.new(rootPos.X - w / 2 - 4, rootPos.Y + h / 2 - (h * pct))
						healthbars[plr].Color = Color3.fromHSV(pct * 0.3, 1, 1)
					else healthbars[plr].Visible = false end

					if studsOn then
						texts[plr].Visible = true
						texts[plr].Text = math.floor((Camera.CFrame.Position - root.Position).Magnitude) .. " studs"
						texts[plr].Position = Vector2.new(rootPos.X, rootPos.Y + h / 2 + 2)
                        texts[plr].Color = Color3.new(1, 1, 1)
					else texts[plr].Visible = false end
				end
			else box.Visible, healthbars[plr].Visible, texts[plr].Visible = false, false, false end
		else box.Visible, healthbars[plr].Visible, texts[plr].Visible = false, false, false end
	end

	if aimOn then
		local target, dist = nil, fovSize
		local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

		for _, p in pairs(Players:GetPlayers()) do
			if p ~= Player and p.Character and p.Character:FindFirstChild("Head") and p.Character.Humanoid.Health > 0 then
				local headPos, vis = Camera:WorldToViewportPoint(p.Character.Head.Position)
				if vis then
					local mag = (Vector2.new(headPos.X, headPos.Y) - center).Magnitude
					if mag < dist then
						dist = mag
						target = p.Character.Head
					end
				end
			end
		end

		if target then
			Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Position), aimStrength)
		end
	end
end)
   end,
})
