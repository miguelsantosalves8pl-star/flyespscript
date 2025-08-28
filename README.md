-- Script de Voo e ESP para Roblox Studio
-- Coloque este script em StarterPlayerScripts como um LocalScript

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Variáveis de controle
local flying = false
local espEnabled = false
local flyConnection
local espBoxes = {}

-- Criação da GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyESPGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 200, 0, 120)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(0, 255, 0)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

-- Botão de Voo
local flyButton = Instance.new("TextButton")
flyButton.Name = "FlyButton"
flyButton.Size = UDim2.new(0, 180, 0, 40)
flyButton.Position = UDim2.new(0, 10, 0, 10)
flyButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
flyButton.BorderSizePixel = 2
flyButton.BorderColor3 = Color3.fromRGB(255, 255, 255)
flyButton.Text = "FLY"
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.TextScaled = true
flyButton.Font = Enum.Font.GothamBold
flyButton.Parent = mainFrame

-- Botão de ESP
local espButton = Instance.new("TextButton")
espButton.Name = "ESPButton"
espButton.Size = UDim2.new(0, 180, 0, 40)
espButton.Position = UDim2.new(0, 10, 0, 60)
espButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
espButton.BorderSizePixel = 2
espButton.BorderColor3 = Color3.fromRGB(255, 255, 255)
espButton.Text = "ESP"
espButton.TextColor3 = Color3.fromRGB(255, 255, 255)
espButton.TextScaled = true
espButton.Font = Enum.Font.GothamBold
espButton.Parent = mainFrame

-- Função de Voo
local function startFlying()
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not rootPart then return end
    
    -- Criar BodyVelocity para controlar o movimento
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.Parent = rootPart
    
    -- Criar BodyAngularVelocity para estabilidade
    local bodyAngularVelocity = Instance.new("BodyAngularVelocity")
    bodyAngularVelocity.MaxTorque = Vector3.new(4000, 4000, 4000)
    bodyAngularVelocity.AngularVelocity = Vector3.new(0, 0, 0)
    bodyAngularVelocity.Parent = rootPart
    
    humanoid.PlatformStand = true
    
    local camera = workspace.CurrentCamera
    local speed = 50
    
    flyConnection = RunService.Heartbeat:Connect(function()
        if not flying then return end
        
        local moveVector = humanoid.MoveDirection
        local cameraDirection = camera.CFrame.LookVector
        local rightVector = camera.CFrame.RightVector
        
        local velocity = Vector3.new(0, 0, 0)
        
        -- Movimento baseado na câmera
        if moveVector.Magnitude > 0 then
            velocity = velocity + (cameraDirection * moveVector.Z * -speed)
            velocity = velocity + (rightVector * moveVector.X * speed)
        end
        
        -- Subir e descer
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            velocity = velocity + Vector3.new(0, speed, 0)
        elseif UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            velocity = velocity + Vector3.new(0, -speed, 0)
        end
        
        bodyVelocity.Velocity = velocity
    end)
end

local function stopFlying()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    
    local character = player.Character
    if character then
        local humanoid = character:FindFirstChild("Humanoid")
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        
        if humanoid then
            humanoid.PlatformStand = false
        end
        
        if rootPart then
            -- Remover objetos de voo
            local bodyVelocity = rootPart:FindFirstChild("BodyVelocity")
            local bodyAngularVelocity = rootPart:FindFirstChild("BodyAngularVelocity")
            
            if bodyVelocity then bodyVelocity:Destroy() end
            if bodyAngularVelocity then bodyAngularVelocity:Destroy() end
        end
    end
end

-- Função ESP
local function createESP(character)
    if not character then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- Apenas o Highlight do corpo (sem nomes)
    

    
    -- Highlight do corpo
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.Adornee = character
    highlight.FillColor = Color3.fromRGB(0, 255, 0)
    highlight.FillTransparency = 0.8
    highlight.OutlineColor = Color3.fromRGB(0, 255, 0)
    highlight.OutlineTransparency = 0
    highlight.Parent = character
    
    espBoxes[character] = highlight
end

local function removeESP(character)
    if espBoxes[character] then
        if espBoxes[character] then espBoxes[character]:Destroy() end
        espBoxes[character] = nil
    end
end

local function updateESP()
    if espEnabled then
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                if not espBoxes[otherPlayer.Character] then
                    createESP(otherPlayer.Character)
                end
            end
        end
    else
        for character, _ in pairs(espBoxes) do
            removeESP(character)
        end
    end
end

-- Eventos dos botões
flyButton.MouseButton1Click:Connect(function()
    flying = not flying
    
    if flying then
        flyButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        flyButton.BorderColor3 = Color3.fromRGB(0, 255, 0)
        startFlying()
    else
        flyButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        flyButton.BorderColor3 = Color3.fromRGB(255, 255, 255)
        stopFlying()
    end
end)

espButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    
    if espEnabled then
        espButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        espButton.BorderColor3 = Color3.fromRGB(0, 255, 0)
    else
        espButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        espButton.BorderColor3 = Color3.fromRGB(255, 255, 255)
    end
    
    updateESP()
end)

-- Eventos para novos jogadores
Players.PlayerAdded:Connect(function(newPlayer)
    newPlayer.CharacterAdded:Connect(function(character)
        if espEnabled then
            wait(1) -- Esperar o personagem carregar completamente
            createESP(character)
        end
    end)
end)

-- Eventos para jogadores existentes
for _, existingPlayer in pairs(Players:GetPlayers()) do
    if existingPlayer ~= player then
        existingPlayer.CharacterAdded:Connect(function(character)
            if espEnabled then
                wait(1)
                createESP(character)
            end
        end)
    end
end

-- Limpeza quando jogador sai
Players.PlayerRemoving:Connect(function(leavingPlayer)
    if leavingPlayer.Character and espBoxes[leavingPlayer.Character] then
        removeESP(leavingPlayer.Character)
    end
end)

-- Parar voo se o personagem morrer ou for removido
player.CharacterRemoving:Connect(function()
    if flying then
        flying = false
        flyButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        flyButton.BorderColor3 = Color3.fromRGB(255, 255, 255)
        stopFlying()
    end
end)
