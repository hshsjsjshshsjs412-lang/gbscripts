-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

-- VARIÁVEIS
local LocalPlayer = Players.LocalPlayer
local espActive = false

-- CONFIGURAÇÕES VISUAIS
local SETTINGS = {
    FillColor = Color3.fromRGB(0, 255, 127),
    OutlineColor = Color3.fromRGB(255, 255, 255),
    FillTransparency = 0.5,
    OutlineTransparency = 0,
    TracerColor = Color3.fromRGB(255, 255, 255) -- Cor das linhas
}

-- INTERFACE (UI)
local ScreenGui = Instance.new("ScreenGui")
local ToggleButton = Instance.new("TextButton")

ScreenGui.Name = "DebugMenu"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

ToggleButton.Size = UDim2.new(0, 150, 0, 40)
ToggleButton.Position = UDim2.new(0.05, 0, 0.05, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ToggleButton.Text = "ESP: OFF"
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.Font = Enum.Font.RobotoMono
ToggleButton.TextSize = 18
ToggleButton.Parent = ScreenGui

--- FUNÇÃO PARA CRIAR LINHAS (TRACERS) ---
-- Usaremos uma Attachment na câmera e outra no alvo
local CameraAttachment = Instance.new("Attachment")
CameraAttachment.Name = "TracerAttachment"
CameraAttachment.Parent = workspace.Terrain -- Fica em um lugar fixo no mundo

local function createTracer(character)
    local root = character:WaitForChild("HumanoidRootPart", 5)
    if not root then return end

    local targetAttachment = Instance.new("Attachment")
    targetAttachment.Name = "TargetAttachment"
    targetAttachment.Parent = root

    local beam = Instance.new("Beam")
    beam.Name = "TracerBeam"
    beam.Attachment0 = CameraAttachment
    beam.Attachment1 = targetAttachment
    beam.Color = ColorSequence.new(SETTINGS.TracerColor)
    beam.Width0 = 0.1
    beam.Width1 = 0.1
    beam.FaceCamera = true
    beam.Enabled = espActive
    beam.AlwaysOnTop = true -- Isso faz ver através das paredes
    beam.Parent = root
end

-- ATUALIZAÇÃO DA POSIÇÃO DA LINHA
RunService.RenderStepped:Connect(function()
    -- Posiciona a origem da linha um pouco abaixo da visão da câmera
    CameraAttachment.WorldCFrame = Camera.CFrame * CFrame.new(0, -2, -1)
end)

-- FUNÇÃO PARA CRIAR O HIGHLIGHT
local function createHighlight(character: Model)
    if not character or character == LocalPlayer.Character then return end
    
    local existingH = character:FindFirstChild("DebugHighlight")
    if existingH then existingH:Destroy() end

    local highlight = Instance.new("Highlight")
    highlight.Name = "DebugHighlight"
    highlight.Adornee = character
    highlight.FillColor = SETTINGS.FillColor
    highlight.OutlineColor = SETTINGS.OutlineColor
    highlight.FillTransparency = SETTINGS.FillTransparency
    highlight.Enabled = espActive
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- Garante ver pelas paredes
    highlight.Parent = character
    
    -- Cria a linha junto com o highlight
    createTracer(character)
end

-- FUNÇÃO PRINCIPAL DE TOGGLE
local function toggleVisuals()
    espActive = not espActive
    ToggleButton.Text = "ESP: " .. (espActive and "ON" or "OFF")
    ToggleButton.BackgroundColor3 = espActive and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)

    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            local h = player.Character:FindFirstChild("DebugHighlight")
            local b = player.Character:FindFirstChild("HumanoidRootPart") and player.Character.HumanoidRootPart:FindFirstChild("TracerBeam")
            
            if h then h.Enabled = espActive end
            if b then b.Enabled = espActive end
            
            if not h and player ~= LocalPlayer then
                createHighlight(player.Character)
            end
        end
    end
end

-- CONEXÕES
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        task.wait(1)
        createHighlight(char)
    end)
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.T then toggleVisuals() end
end)

ToggleButton.MouseButton1Click:Connect(toggleVisuals)

-- Inicialização
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer and player.Character then
        createHighlight(player.Character)
    end
end
