local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local pgui = player:WaitForChild("PlayerGui")

-- === REMOVER VERSÕES ANTIGAS PARA NÃO BUGAR ===
if pgui:FindFirstChild("CyberNoDry_Final_Hub") then
    pgui:FindFirstChild("CyberNoDry_Final_Hub"):Destroy()
end

-- === INTERFACE PRINCIPAL ===
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CyberNoDry_Final_Hub"
screenGui.Parent = pgui
screenGui.ResetOnSpawn = false

-- Frame Principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "Main"
mainFrame.Size = UDim2.new(0, 280, 0, 420)
mainFrame.Position = UDim2.new(0.5, -140, 0.5, -210)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.ZIndex = 5
mainFrame.Parent = screenGui

-- Borda LED RGB
local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 3
uiStroke.Parent = mainFrame
coroutine.wrap(function()
    while task.wait() do
        uiStroke.Color = Color3.fromHSV(tick() % 5 / 5, 1, 1)
    end
end)()

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.Text = "CyberNoDry - HUB"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.BackgroundTransparency = 1
title.ZIndex = 6
title.Parent = mainFrame

-- Container para Botões (Para organizar automático)
local container = Instance.new("Frame")
container.Size = UDim2.new(0, 240, 0, 300)
container.Position = UDim2.new(0.5, -120, 0, 50)
container.BackgroundTransparency = 1
container.Parent = mainFrame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 8)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.Parent = container

-- === BOTÕES DE CONTROLE (X e -) ===
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -35, 0, 8)
closeBtn.Text = "X"
closeBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.ZIndex = 10
closeBtn.Parent = mainFrame

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 30, 0, 30)
minBtn.Position = UDim2.new(1, -70, 0, 8)
minBtn.Text = "-"
minBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
minBtn.TextColor3 = Color3.new(1, 1, 1)
minBtn.Font = Enum.Font.GothamBold
minBtn.ZIndex = 10
minBtn.Parent = mainFrame

-- Botão de Reabrir (fica no canto se minimizar)
local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0, 100, 0, 40)
openBtn.Position = UDim2.new(0, 20, 0, 20)
openBtn.Text = "ABRIR MENU"
openBtn.Visible = false
openBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
openBtn.TextColor3 = Color3.new(1, 1, 1)
openBtn.Parent = screenGui

-- Direitos Autorais
local copyright = Instance.new("TextLabel")
copyright.Size = UDim2.new(1, 0, 0, 30)
copyright.Position = UDim2.new(0, 0, 1, -30)
copyright.Text = "By: CyberNoDry"
copyright.TextColor3 = Color3.fromRGB(200, 200, 200)
copyright.Font = Enum.Font.GothamBold
copyright.BackgroundTransparency = 1
copyright.ZIndex = 6
copyright.Parent = mainFrame

-- === LÓGICA DE TELEPORTE ===
local activePlot = nil
local currentPadIndex = 1

local function getPads(num)
    local pads = {}
    local p = workspace:FindFirstChild("Plots")
    local f = p and p:FindFirstChild(tostring(num))
    if f then
        for _, o in pairs(f:GetDescendants()) do
            if o.Name == "CollectPad" then table.insert(pads, o) end
        end
    end
    return pads
end

local function tpNext()
    if not activePlot then return end
    local pads = getPads(activePlot)
    if #pads > 0 then
        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            if currentPadIndex > #pads then currentPadIndex = 1 end
            hrp.CFrame = pads[currentPadIndex].CFrame
            currentPadIndex = currentPadIndex + 1
        end
    end
end

-- === CRIAR BOTÕES DE PLOTS ===
for i = 1, 5 do
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 45)
    b.Text = "ATIVAR PLOT " .. i
    b.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamBold
    b.ZIndex = 7
    b.Parent = container

    b.MouseButton1Click:Connect(function()
        if activePlot == i then
            activePlot = nil
            b.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
            b.Text = "ATIVAR PLOT " .. i
        else
            activePlot = i
            currentPadIndex = 1
            b.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
            b.Text = "PLOT " .. i .. " ATIVO"
            tpNext()
        end
    end)
end

-- === EVENTOS ===
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)
minBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false; openBtn.Visible = true end)
openBtn.MouseButton1Click:Connect(function() mainFrame.Visible = true; openBtn.Visible = false end)

RunService.Stepped:Connect(function()
    if activePlot and player.Character then
        for _, v in pairs(player.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
        local hum = player.Character:FindFirstChild("Humanoid")
        if hum then hum.Jump = true end
    end
end)

local function onTouch(char)
    char:WaitForChild("HumanoidRootPart").Touched:Connect(function(h)
        if activePlot and h.Name == "CollectPad" then
            task.wait(0.01)
            tpNext()
        end
    end)
end

player.CharacterAdded:Connect(onTouch)
if player.Character then onTouch(player.Character) end
