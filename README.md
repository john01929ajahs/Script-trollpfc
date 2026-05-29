--[[
    TROLL SLAP TOWER - HACK
    Criado por: MadaraMods
--]]

local Player = game:GetService("Players").LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

-- ============ CRÉDITOS ============
local creator = "MadaraMods"

-- ============ ESTADOS (TUDO COMEÇA DESLIGADO) ============
local infiniteJumpEnabled = false
local noclipEnabled = false
local espEnabled = false

-- ============ PULO INFINITO ============
UserInputService.JumpRequest:Connect(function()
    if infiniteJumpEnabled and Player.Character then
        local humanoid = Player.Character:FindFirstChild("Humanoid")
        if humanoid and humanoid:GetState() ~= Enum.HumanoidStateType.Jumping then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- ============ NO CLIP ============
local function applyNoclip()
    if not noclipEnabled then
        if Player.Character then
            for _, part in pairs(Player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
        return
    end
    
    if Player.Character then
        for _, part in pairs(Player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end

RunService.RenderStepped:Connect(applyNoclip)

Player.CharacterAdded:Connect(function()
    task.wait(0.5)
    if noclipEnabled then
        applyNoclip()
    end
end)

-- ============ ESP PARA PLAYERS E NPCS ============
local espObjects = {}

-- Pega todos os jogadores
local function getAllPlayers()
    local players = {}
    for _, v in pairs(game:GetService("Players"):GetPlayers()) do
        if v ~= Player and v.Character then
            local humanoid = v.Character:FindFirstChild("Humanoid")
            local rootPart = v.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and humanoid.Health > 0 and rootPart then
                table.insert(players, {
                    target = v,
                    type = "player",
                    name = v.Name,
                    character = v.Character,
                    rootPart = rootPart,
                    humanoid = humanoid
                })
            end
        end
    end
    return players
end

-- Pega todos os NPCs
local function getAllNPCs()
    local npcs = {}
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= Player.Character then
            local humanoid = obj:FindFirstChild("Humanoid")
            local rootPart = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Torso") or obj:FindFirstChild("UpperTorso")
            
            if humanoid and humanoid.Health > 0 and rootPart then
                -- Verifica se não é um jogador
                local isPlayer = false
                for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                    if player.Character == obj then
                        isPlayer = true
                        break
                    end
                end
                
                if not isPlayer then
                    table.insert(npcs, {
                        target = obj,
                        type = "npc",
                        name = obj.Name,
                        character = obj,
                        rootPart = rootPart,
                        humanoid = humanoid
                    })
                end
            end
        end
    end
    
    return npcs
end

-- Atualiza ESP
local function updateESP()
    if not espEnabled then
        for _, obj in pairs(espObjects) do
            if obj.text then obj.text:Destroy() end
            if obj.box then obj.box:Destroy() end
            if obj.headDot then obj.headDot:Destroy() end
        end
        espObjects = {}
        return
    end
    
    local playerPos = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
    if not playerPos then return end
    
    -- Pega players e NPCs
    local players = getAllPlayers()
    local npcs = getAllNPCs()
    
    -- Desenha ESP para players
    for _, data in pairs(players) do
        local rootPart = data.rootPart
        local humanoid = data.humanoid
        local dist = (playerPos.Position - rootPart.Position).Magnitude
        
        if dist <= 300 then
            local esp = nil
            for _, e in pairs(espObjects) do
                if e.target == data.target then esp = e break end
            end
            
            local head = data.character:FindFirstChild("Head")
            local headPos = head and head.Position or rootPart.Position + Vector3.new(0, 2.5, 0)
            local vector, onScreen = Camera:WorldToViewportPoint(headPos)
            
            if onScreen then
                if not esp then
                    local text = Drawing.new("Text")
                    text.Size = 13
                    text.Center = true
                    text.Outline = true
                    text.Color = 0xff3333
                    
                    local box = Drawing.new("Square")
                    box.Thickness = 2
                    box.Color = 0xff3333
                    box.Filled = false
                    
                    local headDot = Drawing.new("Circle")
                    headDot.Radius = 5
                    headDot.Thickness = 2
                    headDot.Color = 0xff3333
                    headDot.Filled = true
                    
                    esp = {text = text, box = box, headDot = headDot, target = data.target}
                    table.insert(espObjects, esp)
                end
                
                local size = rootPart.Size.Y * 3
                local health = math.floor(humanoid.Health)
                
                esp.text.Text = string.format("👤 %s | ❤️ %d | 📍%.0fm", data.name, health, dist)
                esp.text.Position = Vector2.new(vector.X, vector.Y - size - 15)
                esp.text.Visible = true
                esp.text.Color = 0xff3333
                
                esp.box.Size = Vector2.new(50, size + 20)
                esp.box.Position = Vector2.new(vector.X - 25, vector.Y - size)
                esp.box.Visible = true
                esp.box.Color = 0xff3333
                
                esp.headDot.Position = Vector2.new(vector.X, vector.Y)
                esp.headDot.Visible = true
                esp.headDot.Color = 0xff3333
            else
                if esp then
                    esp.text.Visible = false
                    esp.box.Visible = false
                    esp.headDot.Visible = false
                end
            end
        end
    end
    
    -- Desenha ESP para NPCs
    for _, data in pairs(npcs) do
        local rootPart = data.rootPart
        local humanoid = data.humanoid
        local dist = (playerPos.Position - rootPart.Position).Magnitude
        
        if dist <= 300 then
            local esp = nil
            for _, e in pairs(espObjects) do
                if e.target == data.target then esp = e break end
            end
            
            local head = data.character:FindFirstChild("Head")
            local headPos = head and head.Position or rootPart.Position + Vector3.new(0, 2, 0)
            local vector, onScreen = Camera:WorldToViewportPoint(headPos)
            
            if onScreen then
                if not esp then
                    local text = Drawing.new("Text")
                    text.Size = 13
                    text.Center = true
                    text.Outline = true
                    text.Color = 0xffaa33
                    
                    local box = Drawing.new("Square")
                    box.Thickness = 2
                    box.Color = 0xffaa33
                    box.Filled = false
                    
                    local headDot = Drawing.new("Circle")
                    headDot.Radius = 5
                    headDot.Thickness = 2
                    headDot.Color = 0xffaa33
                    headDot.Filled = true
                    
                    esp = {text = text, box = box, headDot = headDot, target = data.target}
                    table.insert(espObjects, esp)
                end
                
                local size = rootPart.Size.Y * 3
                local health = math.floor(humanoid.Health)
                local npcName = string.sub(data.name, 1, 15)
                
                esp.text.Text = string.format("🤖 %s | ❤️ %d | 📍%.0fm", npcName, health, dist)
                esp.text.Position = Vector2.new(vector.X, vector.Y - size - 15)
                esp.text.Visible = true
                esp.text.Color = 0xffaa33
                
                esp.box.Size = Vector2.new(50, size + 20)
                esp.box.Position = Vector2.new(vector.X - 25, vector.Y - size)
                esp.box.Visible = true
                esp.box.Color = 0xffaa33
                
                esp.headDot.Position = Vector2.new(vector.X, vector.Y)
                esp.headDot.Visible = true
                esp.headDot.Color = 0xffaa33
            else
                if esp then
                    esp.text.Visible = false
                    esp.box.Visible = false
                    esp.headDot.Visible = false
                end
            end
        end
    end
end

-- ============ FUNÇÃO PARA DESATIVAR TUDO ============
local function disableAllHacks()
    -- Desativa Pulo Infinito
    infiniteJumpEnabled = false
    
    -- Desativa No Clip
    noclipEnabled = false
    applyNoclip()
    
    -- Desativa ESP
    espEnabled = false
    updateESP()
end

-- ============ MENU CORRIGIDO ============

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MadaraModsHack"
screenGui.Parent = game:GetService("CoreGui")
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 320, 0, 320)
mainFrame.Position = UDim2.new(0.5, -160, 0.5, -160)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(255, 70, 70)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.BackgroundColor3 = Color3.fromRGB(255, 70, 70)
titleBar.BackgroundTransparency = 0.2
titleBar.Parent = mainFrame

-- Título com créditos
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -90, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🔥 MADARA MODS 🔥"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = titleBar

-- Botão Minimizar
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 35, 1, -8)
minimizeBtn.Position = UDim2.new(1, -75, 0, 4)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
minimizeBtn.Text = "▼"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 14
minimizeBtn.Parent = titleBar

-- Botão Fechar (X) - DESATIVA TUDO
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 35, 1, -8)
closeBtn.Position = UDim2.new(1, -40, 0, 4)
closeBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 14
closeBtn.Parent = titleBar

-- Container do conteúdo
local content = Instance.new("Frame")
content.Size = UDim2.new(1, 0, 1, -45)
content.Position = UDim2.new(0, 0, 0, 45)
content.BackgroundTransparency = 1
content.Parent = mainFrame

-- Linha de créditos
local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(0.9, 0, 0, 25)
creditLabel.Position = UDim2.new(0.05, 0, 0.02, 0)
creditLabel.BackgroundColor3 = Color3.fromRGB(255, 70, 70)
creditLabel.BackgroundTransparency = 0.3
creditLabel.Text = "⚡ CRIADO POR: MADARA MODS ⚡"
creditLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
creditLabel.Font = Enum.Font.GothamBold
creditLabel.TextSize = 12
creditLabel.Parent = content

-- Botão Pulo Infinito
local jumpBtn = Instance.new("TextButton")
jumpBtn.Size = UDim2.new(0.9, 0, 0, 45)
jumpBtn.Position = UDim2.new(0.05, 0, 0.12, 0)
jumpBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
jumpBtn.Text = "🌀 PULO INFINITO : OFF"
jumpBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpBtn.Font = Enum.Font.GothamBold
jumpBtn.TextSize = 14
jumpBtn.BorderSizePixel = 0
jumpBtn.Parent = content

-- Botão No Clip
local noclipBtn = Instance.new("TextButton")
noclipBtn.Size = UDim2.new(0.9, 0, 0, 45)
noclipBtn.Position = UDim2.new(0.05, 0, 0.27, 0)
noclipBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
noclipBtn.Text = "🧱 NO CLIP : OFF"
noclipBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
noclipBtn.Font = Enum.Font.GothamBold
noclipBtn.TextSize = 14
noclipBtn.BorderSizePixel = 0
noclipBtn.Parent = content

-- Botão ESP
local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.new(0.9, 0, 0, 45)
espBtn.Position = UDim2.new(0.05, 0, 0.42, 0)
espBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
espBtn.Text = "👁️ ESP : OFF"
espBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
espBtn.Font = Enum.Font.GothamBold
espBtn.TextSize = 14
espBtn.BorderSizePixel = 0
espBtn.Parent = content

-- Label de informação
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(0.9, 0, 0, 60)
infoLabel.Position = UDim2.new(0.05, 0, 0.60, 0)
infoLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
infoLabel.Text = "📌 TODAS FUNÇÕES DESATIVADAS\nClique nos botões para ativar!\n\n❌ Fechar menu desativa TUDO"
infoLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextSize = 11
infoLabel.TextWrapped = true
infoLabel.Parent = content

-- Estados dos botões
local jumpState = false
local noclipState = false
local espState = false

-- Funções dos botões
jumpBtn.MouseButton1Click:Connect(function()
    jumpState = not jumpState
    infiniteJumpEnabled = jumpState
    jumpBtn.Text = jumpState and "🌀 PULO INFINITO : ON" or "🌀 PULO INFINITO : OFF"
    jumpBtn.BackgroundColor3 = jumpState and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(100, 0, 0)
end)

noclipBtn.MouseButton1Click:Connect(function()
    noclipState = not noclipState
    noclipEnabled = noclipState
    noclipBtn.Text = noclipState and "🧱 NO CLIP : ON" or "🧱 NO CLIP : OFF"
    noclipBtn.BackgroundColor3 = noclipState and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(100, 0, 0)
    applyNoclip()
end)

espBtn.MouseButton1Click:Connect(function()
    espState = not espState
    espEnabled = espState
    espBtn.Text = espState and "👁️ ESP : ON" or "👁️ ESP : OFF"
    espBtn.BackgroundColor3 = espState and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(100, 0, 0)
    if not espState then
        updateESP()
    end
end)

-- Fechar menu - DESATIVA TUDO
closeBtn.MouseButton1Click:Connect(function()
    -- Desativa todas as funções
    disableAllHacks()
    
    -- Reseta os botões visualmente
    jumpState = false
    noclipState = false
    espState = false
    
    infiniteJumpEnabled = false
    noclipEnabled = false
    espEnabled = false
    
    jumpBtn.Text = "🌀 PULO INFINITO : OFF"
    jumpBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
    
    noclipBtn.Text = "🧱 NO CLIP : OFF"
    noclipBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
    
    espBtn.Text = "👁️ ESP : OFF"
    espBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
    
    -- Limpa ESP
    updateESP()
    
    -- Fecha o menu
    mainFrame.Visible = false
end)

-- Minimizar
local isMinimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    content.Visible = not isMinimized
    minimizeBtn.Text = isMinimized and "▲" or "▼"
    mainFrame.Size = isMinimized and UDim2.new(0, 320, 0, 45) or UDim2.new(0, 320, 0, 320)
end)

-- ============ LOOP PRINCIPAL ============
RunService.RenderStepped:Connect(function()
    if espEnabled then
        updateESP()
    end
end)

-- ============ MENSAGEM INICIAL ============
print("=" .. string.rep("=", 50))
print("🔥 TROLL SLAP TOWER - HACK 🔥")
print("⚡ CRIADO POR: MADARA MODS ⚡")
print("=" .. string.rep("=", 50))
print("⚠️ TODAS AS FUNÇÕES ESTÃO DESATIVADAS!")
print("")
print("🎮 PARA ATIVAR:")
print("   - Clique nos botões VERMELHOS")
print("   - Botão fica VERDE quando ativado")
print("")
print("📋 FUNÇÕES:")
print("   🌀 Pulo Infinito")
print("   🧱 No Clip")
print("   👁️ ESP (Players em VERMELHO / NPCs em AMARELO)")
print("")
print("❌ Ao clicar no X:")
print("   - Todas funções são DESATIVADAS")
print("   - Menu é FECHADO")
print("=" .. string.rep("=", 50))
