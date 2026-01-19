-- Velocity Executor UI
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Variáveis de estado
local noclipEnabled = false
local flyEnabled = false
local speedEnabled = false
local wallhackEnabled = false
local hitboxEnabled = false
local godmodeEnabled = false
local savedCheckpoint = nil
local defaultSpeed = 16
local customSpeed = 100
local flySpeed = 100

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local NoclipButton = Instance.new("TextButton")
local FlyButton = Instance.new("TextButton")
local SpeedButton = Instance.new("TextButton")
local WallhackButton = Instance.new("TextButton")
local HitboxButton = Instance.new("TextButton")
local GodmodeButton = Instance.new("TextButton")
local SaveButton = Instance.new("TextButton")
local TpButton = Instance.new("TextButton")
local CloseButton = Instance.new("TextButton")

-- Propriedades da GUI
ScreenGui.Name = "PUIG"
ScreenGui.Parent = game.CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
MainFrame.BorderSizePixel = 0
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -220)
MainFrame.Size = UDim2.new(0, 300, 0, 440)
MainFrame.Active = true
MainFrame.Draggable = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

Title.Name = "Title"
Title.Parent = MainFrame
Title.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Font = Enum.Font.GothamBold
Title.Text = "PUIG SCRIPT"
Title.TextColor3 = Color3.fromRGB(0, 255, 255)
Title.TextSize = 20

-- Função para criar botões
local function createButton(name, text, position, parent)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Parent = parent
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    button.BorderSizePixel = 0
    button.Position = position
    button.Size = UDim2.new(0, 260, 0, 35)
    button.Font = Enum.Font.GothamSemibold
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 14
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = button
    
    return button
end

-- Criar botões
NoclipButton = createButton("NoclipButton", "Noclip: OFF", UDim2.new(0, 20, 0, 60), MainFrame)
FlyButton = createButton("FlyButton", "Fly: OFF", UDim2.new(0, 20, 0, 105), MainFrame)
SpeedButton = createButton("SpeedButton", "Speed: OFF", UDim2.new(0, 20, 0, 150), MainFrame)
WallhackButton = createButton("WallhackButton", "Wallhack: OFF", UDim2.new(0, 20, 0, 195), MainFrame)
HitboxButton = createButton("HitboxButton", "Hitbox Expander: OFF", UDim2.new(0, 20, 0, 240), MainFrame)
GodmodeButton = createButton("GodmodeButton", "Godmode: OFF", UDim2.new(0, 20, 0, 285), MainFrame)
SaveButton = createButton("SaveButton", "SAVE Checkpoint", UDim2.new(0, 20, 0, 330), MainFrame)
TpButton = createButton("TpButton", "TP to Checkpoint", UDim2.new(0, 20, 0, 375), MainFrame)

SaveButton.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
TpButton.BackgroundColor3 = Color3.fromRGB(0, 100, 200)

CloseButton = Instance.new("TextButton")
CloseButton.Parent = MainFrame
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
CloseButton.BorderSizePixel = 0
CloseButton.Position = UDim2.new(1, -30, 0, 5)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 14

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 4)
closeCorner.Parent = CloseButton

-- Variáveis para Fly
local flyBodyVelocity = nil
local flyBodyGyro = nil

-- Funções
local function toggleNoclip()
    noclipEnabled = not noclipEnabled
    NoclipButton.Text = "Noclip: " .. (noclipEnabled and "ON" or "OFF")
    NoclipButton.BackgroundColor3 = noclipEnabled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 60)
end

local function toggleFly()
    flyEnabled = not flyEnabled
    FlyButton.Text = "Fly: " .. (flyEnabled and "ON" or "OFF")
    FlyButton.BackgroundColor3 = flyEnabled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 60)
    
    if flyEnabled then
        -- Criar componentes de voo
        flyBodyVelocity = Instance.new("BodyVelocity")
        flyBodyVelocity.Parent = rootPart
        flyBodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        
        flyBodyGyro = Instance.new("BodyGyro")
        flyBodyGyro.Parent = rootPart
        flyBodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        flyBodyGyro.P = 9e4
    else
        -- Remover componentes de voo
        if flyBodyVelocity then flyBodyVelocity:Destroy() flyBodyVelocity = nil end
        if flyBodyGyro then flyBodyGyro:Destroy() flyBodyGyro = nil end
        humanoid.PlatformStand = false
    end
end

local function toggleSpeed()
    speedEnabled = not speedEnabled
    SpeedButton.Text = "Speed: " .. (speedEnabled and "ON" or "OFF")
    SpeedButton.BackgroundColor3 = speedEnabled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 60)
    
    if speedEnabled then
        humanoid.WalkSpeed = customSpeed
    else
        humanoid.WalkSpeed = defaultSpeed
    end
end

local function toggleWallhack()
    wallhackEnabled = not wallhackEnabled
    WallhackButton.Text = "Wallhack: " .. (wallhackEnabled and "ON" or "OFF")
    WallhackButton.BackgroundColor3 = wallhackEnabled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 60)
    
    for _, v in pairs(game.Workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Parent ~= character then
            if wallhackEnabled then
                v.Transparency = 0.5
            end
        end
    end
end

local function toggleHitbox()
    hitboxEnabled = not hitboxEnabled
    HitboxButton.Text = "Hitbox Expander: " .. (hitboxEnabled and "ON" or "OFF")
    HitboxButton.BackgroundColor3 = hitboxEnabled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 60)
    
    for _, v in pairs(game.Workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Parent:FindFirstChild("Humanoid") and v.Parent ~= character then
            if hitboxEnabled then
                v.Size = Vector3.new(10, 10, 10)
                v.Transparency = 0.7
                v.CanCollide = false
            else
                v.Size = Vector3.new(2, 2, 1)
                v.Transparency = 0
            end
        end
    end
end

local function toggleGodmode()
    godmodeEnabled = not godmodeEnabled
    GodmodeButton.Text = "Godmode: " .. (godmodeEnabled and "ON" or "OFF")
    GodmodeButton.BackgroundColor3 = godmodeEnabled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 60)
    
    if godmodeEnabled then
        -- Remover conexão de dano
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        humanoid.Health = humanoid.MaxHealth
    else
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
    end
end

local function saveCheckpoint()
    savedCheckpoint = rootPart.CFrame
    SaveButton.Text = "Checkpoint SAVED!"
    wait(1)
    SaveButton.Text = "SAVE Checkpoint"
end

local function teleportToCheckpoint()
    if savedCheckpoint then
        rootPart.CFrame = savedCheckpoint
        TpButton.Text = "Teleported!"
        wait(1)
        TpButton.Text = "TP to Checkpoint"
    else
        TpButton.Text = "No checkpoint saved!"
        wait(1)
        TpButton.Text = "TP to Checkpoint"
    end
end

-- Função para alternar visibilidade da UI (tecla K)
local function toggleUI()
    MainFrame.Visible = not MainFrame.Visible
end

-- Noclip Loop
game:GetService("RunService").Stepped:Connect(function()
    if noclipEnabled and character then
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- Fly Loop
game:GetService("RunService").Heartbeat:Connect(function()
    if flyEnabled and flyBodyVelocity and flyBodyGyro and rootPart then
        local cam = workspace.CurrentCamera
        local direction = Vector3.new(0, 0, 0)
        local userInputService = game:GetService("UserInputService")
        
        humanoid.PlatformStand = true
        
        if userInputService:IsKeyDown(Enum.KeyCode.W) then
            direction = direction + (cam.CFrame.LookVector * flySpeed)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.S) then
            direction = direction - (cam.CFrame.LookVector * flySpeed)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.A) then
            direction = direction - (cam.CFrame.RightVector * flySpeed)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.D) then
            direction = direction + (cam.CFrame.RightVector * flySpeed)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.Space) then
            direction = direction + (Vector3.new(0, 1, 0) * flySpeed)
        end
        if userInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            direction = direction - (Vector3.new(0, 1, 0) * flySpeed)
        end
        
        flyBodyVelocity.Velocity = direction
        flyBodyGyro.CFrame = cam.CFrame
    end
end)

-- Speed Loop para manter a velocidade
game:GetService("RunService").RenderStepped:Connect(function()
    if speedEnabled and humanoid then
        humanoid.WalkSpeed = customSpeed
    end
    
    -- Godmode Loop - mantém vida cheia
    if godmodeEnabled and humanoid then
        if humanoid.Health < humanoid.MaxHealth then
            humanoid.Health = humanoid.MaxHealth
        end
    end
end)

-- Conectar botões
NoclipButton.MouseButton1Click:Connect(toggleNoclip)
FlyButton.MouseButton1Click:Connect(toggleFly)
SpeedButton.MouseButton1Click:Connect(toggleSpeed)
WallhackButton.MouseButton1Click:Connect(toggleWallhack)
HitboxButton.MouseButton1Click:Connect(toggleHitbox)
GodmodeButton.MouseButton1Click:Connect(toggleGodmode)
SaveButton.MouseButton1Click:Connect(saveCheckpoint)
TpButton.MouseButton1Click:Connect(teleportToCheckpoint)
CloseButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

-- Detectar tecla K para abrir/fechar UI
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
        toggleUI()
    end
end)

-- Atualizar character ao resetar
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
    
    -- Resetar todas as funções
    if flyEnabled then
        flyEnabled = false
        toggleFly()
    end
    if speedEnabled then
        speedEnabled = false
        toggleSpeed()
    end
end)

print("PUIG Script carregado! Pressione K para abrir/fechar a UI")
