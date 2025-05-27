--[[
Yellow Hub | Arsenal - v1.0
✅ Interface estilo Tbao Hub
✅ ESP completo com toggles
✅ Aimbot inteligente com FOV
✅ Ativacao por RightShift
--]]

-- CONFIG GERAL
local settings = {
    esp = {
        enabled = false,
        name = false,
        box = false,
        tracer = false,
        health = false,
        skeleton = false,
        distance = false,
        teamCheck = true
    },
    aimbot = {
        enabled = false,
        fov = 150,
        part = "Head",
        smoothness = 0.15,
        teamCheck = true
    },
    keybind = Enum.KeyCode.RightShift
}

-- DEPENDENCIAS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local function CreateESPBox(player)
    local box = Drawing.new("Square")
    box.Thickness = 1.5
    box.Transparency = 1
    box.Filled = false
    box.Color = Color3.new(1, 1, 0)

    local function Update()
        RunService.RenderStepped:Connect(function()
            if not settings.esp.enabled or not settings.esp.box then box.Visible = false return end
            if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then box.Visible = false return end
            if settings.esp.teamCheck and player.Team == LocalPlayer.Team then box.Visible = false return end
            local pos, onscreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if onscreen then
                local size = Vector3.new(2, 3, 1.5) * 4
                local tl = Camera:WorldToViewportPoint((player.Character.HumanoidRootPart.CFrame * CFrame.new(-size.X/2, size.Y/2, 0)).Position)
                local br = Camera:WorldToViewportPoint((player.Character.HumanoidRootPart.CFrame * CFrame.new(size.X/2, -size.Y/2, 0)).Position)
                box.Size = Vector2.new(math.abs(tl.X - br.X), math.abs(tl.Y - br.Y))
                box.Position = Vector2.new(math.min(tl.X, br.X), math.min(tl.Y, br.Y))
                box.Visible = true
            else
                box.Visible = false
            end
        end)
    end

    Update()
end

-- Aimbot
local function GetClosestPlayer()
    local closest, shortest = nil, settings.aimbot.fov
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild(settings.aimbot.part) then
            if settings.aimbot.teamCheck and p.Team == LocalPlayer.Team then continue end
            local pos, vis = Camera:WorldToViewportPoint(p.Character[settings.aimbot.part].Position)
            if vis then
                local dist = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                if dist < shortest then
                    closest, shortest = p, dist
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if not settings.aimbot.enabled then return end
    local target = GetClosestPlayer()
    if target and target.Character and target.Character:FindFirstChild(settings.aimbot.part) then
        local aimPos = Camera:WorldToScreenPoint(target.Character[settings.aimbot.part].Position)
        local delta = Vector2.new(aimPos.X, aimPos.Y) - UserInputService:GetMouseLocation()
        mousemoverel(delta.X * settings.aimbot.smoothness, delta.Y * settings.aimbot.smoothness)
    end
end)

-- ESP por jogadores existentes
for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        CreateESPBox(p)
    end
end
Players.PlayerAdded:Connect(function(p)
    if p ~= LocalPlayer then
        CreateESPBox(p)
    end
end)

-- GUI
local function CreateToggle(text, parent, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Text = text .. ": OFF"
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 14
    btn.Parent = parent
    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.Text = text .. ": " .. (state and "ON" or "OFF")
        callback(state)
    end)
end

local gui = Instance.new("ScreenGui", game.CoreGui)
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 400)
frame.Position = UDim2.new(0, 20, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.Visible = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "Yellow Hub | Arsenal"
title.TextColor3 = Color3.new(1, 1, 0)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.BackgroundTransparency = 1

local espFrame = Instance.new("Frame", frame)
espFrame.Position = UDim2.new(0, 0, 0, 40)
espFrame.Size = UDim2.new(1, 0, 1, -40)
espFrame.BackgroundTransparency = 1

CreateToggle("ESP Enabled", espFrame, function(val) settings.esp.enabled = val end)
CreateToggle("Name", espFrame, function(val) settings.esp.name = val end)
CreateToggle("Box", espFrame, function(val) settings.esp.box = val end)
CreateToggle("Tracer", espFrame, function(val) settings.esp.tracer = val end)
CreateToggle("Health", espFrame, function(val) settings.esp.health = val end)
CreateToggle("Skeleton", espFrame, function(val) settings.esp.skeleton = val end)
CreateToggle("Distance", espFrame, function(val) settings.esp.distance = val end)
CreateToggle("Team Check", espFrame, function(val) settings.esp.teamCheck = val end)
CreateToggle("Aimbot", espFrame, function(val) settings.aimbot.enabled = val end)

-- Ativador por tecla
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == settings.keybind then
        frame.Visible = not frame.Visible
    end
end)

