local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
    Name = "🐭 Plexzinho Hub | Julio Edition",
    LoadingTitle = "Carregando Plexzinho Roxo",
    LoadingSubtitle = "SOMENTE PVP - Delta",
    ConfigurationSaving = { Enabled = true, FolderName = "PlexzinhoHubJulio", FileName = "config" },
    KeySystem = false
})

Rayfield:Notify({
    Title = "ATENÇÃO IMPORTANTE",
    Content = "Esse script é SOMENTE PARA PVP!!!\nNão use em farm, raid, auto-farm ou qualquer coisa que não seja PvP direto. Pode causar ban. Use por sua conta e risco.",
    Duration = 10,
    Image = 4483362458
})

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local player = Players.LocalPlayer
local cam = Workspace.CurrentCamera

-- Variáveis
local AmboitEnabled = false
local loopSpeedAtivo = false
local isAttacking = false

-- Visual Amboite
local Line = Drawing.new("Line")
Line.Color = Color3.fromRGB(0, 255, 0)
Line.Thickness = 1
local Square = Drawing.new("Square")
Square.Color = Color3.fromRGB(0, 255, 0)
Square.Thickness = 2
Square.Filled = true
Square.Transparency = 0.7

-- Detector de ataque
local function setupAttackDetector(char)
    local hum = char:WaitForChild("Humanoid", 10)
    if not hum then return end
    local animator = hum:WaitForChild("Animator", 10)
    if not animator then return end
    
    animator.AnimationPlayed:Connect(function(track)
        if track and track.Priority == Enum.AnimationPriority.Action then
            local nameLower = track.Name:lower()
            if track.Length < 0.45 or nameLower:find("dash") or nameLower:find("flame") or nameLower:find("run") or nameLower:find("speed") or nameLower:find("boost") then
                return
            end
            task.delay(0.1, function()
                if track.IsPlaying and hum.MoveDirection.Magnitude < 0.3 then
                    isAttacking = true
                end
            end)
            local conn = track.Stopped:Connect(function()
                isAttacking = false
                conn:Disconnect()
            end)
        end
    end)
end

player.CharacterAdded:Connect(setupAttackDetector)
if player.Character then task.spawn(setupAttackDetector, player.Character) end

local function GetClosest()
    local target, dist = nil, math.huge
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local hum = v.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local d = (v.Character.HumanoidRootPart.Position - root.Position).Magnitude
                if d < dist then dist = d target = v end
            end
        end
    end
    return target
end

-- ESP (quadrado minúsculo como pedido)
local ESP_ENABLED = false
local ESP_OBJECTS = {}
local function CreateESP(plr)
    if plr == player then return end
    local Box = Drawing.new("Square")
    Box.Thickness = 2
    Box.Color = Color3.fromRGB(0, 255, 0)
    Box.Filled = true
    Box.Transparency = 0.7
    Box.Visible = false
    
    local NameTag = Drawing.new("Text")
    NameTag.Size = 14
    NameTag.Center = true
    NameTag.Outline = true
    NameTag.Color = Color3.fromRGB(0, 255, 0)
    NameTag.Visible = false
    
    local LevelTag = Drawing.new("Text")
    LevelTag.Size = 13
    LevelTag.Center = true
    LevelTag.Outline = true
    LevelTag.Color = Color3.fromRGB(255, 255, 0)
    LevelTag.Visible = false
    
    local DistanceTag = Drawing.new("Text")
    DistanceTag.Size = 13
    DistanceTag.Center = true
    DistanceTag.Outline = true
    DistanceTag.Color = Color3.fromRGB(255, 0, 0)
    DistanceTag.Visible = false
    
    ESP_OBJECTS[plr] = {Box = Box, Name = NameTag, Level = LevelTag, Distance = DistanceTag}
end

local function UpdateESP()
    for plr, obj in pairs(ESP_OBJECTS) do
        if ESP_ENABLED and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = plr.Character.HumanoidRootPart
            local pos, onScreen = cam:WorldToViewportPoint(hrp.Position)
            if onScreen then
                local dist = math.floor((hrp.Position - player.Character.HumanoidRootPart.Position).Magnitude)
                local level = plr:FindFirstChild("Data") and plr.Data:FindFirstChild("Level") and plr.Data.Level.Value or "N/A"
                -- Quadrado minúsculo
                local sizeX = 40 / pos.Z  -- Pequeno
                local sizeY = 60 / pos.Z  -- Retângulo vertical pequeno (tipo highlight no torso)
                obj.Box.Size = Vector2.new(sizeX, sizeY)
                obj.Box.Position = Vector2.new(pos.X - sizeX/2, pos.Y - sizeY/2)
                obj.Box.Visible = true
                
                obj.Distance.Text = "[" .. dist .. "m]"
                obj.Distance.Position = Vector2.new(pos.X, pos.Y - sizeY/2 - 45)
                obj.Distance.Visible = true
                
                obj.Level.Text = "Level: " .. level
                obj.Level.Position = Vector2.new(pos.X, pos.Y - sizeY/2 - 30)
                obj.Level.Visible = true
                
                obj.Name.Text = plr.DisplayName or plr.Name
                obj.Name.Position = Vector2.new(pos.X, pos.Y - sizeY/2 - 15)
                obj.Name.Visible = true
            else
                obj.Box.Visible = false
                obj.Name.Visible = false
                obj.Level.Visible = false
                obj.Distance.Visible = false
            end
        else
            obj.Box.Visible = false
            obj.Name.Visible = false
            obj.Level.Visible = false
            obj.Distance.Visible = false
        end
    end
end

-- Bypass velocidade 200
task.spawn(function()
    while true do
        task.wait()
        if loopSpeedAtivo and player.Character and player.Character:FindFirstChild("Humanoid") then
            local hum = player.Character.Humanoid
            hum.WalkSpeed = 200
            if hum.MoveDirection.Magnitude > 0 then
                player.Character:TranslateBy(hum.MoveDirection * 1.5)
            end
        end
    end
end)

-- ABA PvP
local PvPTab = Window:CreateTab("PvP")

PvPTab:CreateToggle({
    Name = "Amboite (AUTO Ataques Reais)",
    CurrentValue = false,
    Callback = function(Value)
        AmboitEnabled = Value
        if not Value then Line.Visible = false Square.Visible = false end
    end
})

PvPTab:CreateToggle({
    Name = "ESP Julio Edition",
    CurrentValue = false,
    Callback = function(Value) ESP_ENABLED = Value end
})

PvPTab:CreateToggle({
    Name = "Velocidade 200 (Bypass Andando)",
    CurrentValue = false,
    Callback = function(Value)
        loopSpeedAtivo = Value
        if not Value and player.Character and player.Character:FindFirstChild("Humanoid") then
            player.Character.Humanoid.WalkSpeed = 16
        end
    end
})

PvPTab:CreateButton({
    Name = "🛡️ ATIVAR BYPASS",
    Callback = function()
        Rayfield:Notify({Title="Bypass", Content="Anti-Ban Ativado!", Duration=5})
    end
})

-- Loop principal
RunService.RenderStepped:Connect(function()
    UpdateESP()
    if AmboitEnabled then
        local target = GetClosest()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local part = target.Character.HumanoidRootPart
            local myPart = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if part and myPart then
                local pos, onScreen = cam:WorldToViewportPoint(part.Position)
                local myPos = cam:WorldToViewportPoint(myPart.Position)
                Square.Visible = true
                Square.Size = Vector2.new(12, 12)
                Square.Position = Vector2.new(pos.X - 6, pos.Y - 6)
                Line.Visible = true
                Line.From = Vector2.new(myPos.X, myPos.Y)
                Line.To = Vector2.new(pos.X, pos.Y)
               
                if isAttacking then
                    local offset = Vector3.new(0, 2.2 + math.random(-5,5)/10, 0)
                    local targetCFrame = CFrame.new(cam.CFrame.Position, part.Position + offset)
                    cam.CFrame = cam.CFrame:Lerp(targetCFrame, 0.4)
                end
            else
                Square.Visible = false
                Line.Visible = false
            end
        else
            Square.Visible = false
            Line.Visible = false
        end
    end
end)

for _, p in pairs(Players:GetPlayers()) do CreateESP(p) end
Players.PlayerAdded:Connect(CreateESP)

Rayfield:Notify({
    Title = "Script Carregado!",
    Content = "Esse script é SOMENTE PARA PVP!!!\nNão use em farm, raid ou auto-farm. Use apenas em PvP direto.\nESP agora com quadrado minúsculo preenchido verde como pedido.",
    Duration = 12,
    Image = 4483362458
})
