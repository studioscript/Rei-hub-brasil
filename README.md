--[[
    REI HUB ⚡ DELTA EDITION
    Blox Fruits Auto Farm & Combat Suite
    Desenvolvido para máxima performance no Delta Executor
    Versão: 1.0.0
--]]

-- // Services Declaration
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- // Local Player & Character References
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Backpack = LocalPlayer:WaitForChild("Backpack")

-- // Remote Events & Functions
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local CommF = Remotes:WaitForChild("CommF_")
local ValidatorEvent = Remotes:WaitForChild("Validator")

-- // Variáveis Globais de Configuração
_G.Weapon = "Melee"
_G.FarmSpeed = 300
_G.AttackRange = 15
_G.AutoFarm = false
_G.AutoRaid = false
_G.BringMobs = false
_G.FastAttack = false
_G.KillAura = false
_G.Aimbot = false
_G.SilentAim = false
_G.ESPEnabled = false
_G.FPSBoost = false

-- // Tabelas de Configuração de Farm por Level
local FarmConfig = {
    {Level = 0, QuestNPC = "Bandit Quest Giver", Monster = "Bandit", NPC_CFrame = CFrame.new(1060, 16, 1550), MonsterArea = CFrame.new(1200, 16, 1550)},
    {Level = 10, QuestNPC = "Monkey Quest Giver", Monster = "Monkey", NPC_CFrame = CFrame.new(-1592, 16, 142), MonsterArea = CFrame.new(-1500, 16, 142)},
    {Level = 30, QuestNPC = "Pirate Quest Giver", Monster = "Pirate", NPC_CFrame = CFrame.new(-1500, 24, 3900), MonsterArea = CFrame.new(-1450, 24, 3900)},
    {Level = 50, QuestNPC = "Brute Quest Giver", Monster = "Brute", NPC_CFrame = CFrame.new(-1750, 24, 4200), MonsterArea = CFrame.new(-1700, 24, 4200)},
    {Level = 80, QuestNPC = "Desert Bandit Quest Giver", Monster = "Desert Bandit", NPC_CFrame = CFrame.new(900, 6, 4400), MonsterArea = CFrame.new(950, 6, 4400)},
    {Level = 100, QuestNPC = "Desert Officer Quest Giver", Monster = "Desert Officer", NPC_CFrame = CFrame.new(1550, 6, 4400), MonsterArea = CFrame.new(1600, 6, 4400)},
    {Level = 130, QuestNPC = "Snow Bandit Quest Giver", Monster = "Snow Bandit", NPC_CFrame = CFrame.new(-4950, 90, -2200), MonsterArea = CFrame.new(-4900, 90, -2200)},
    {Level = 160, QuestNPC = "Snowman Quest Giver", Monster = "Snowman", NPC_CFrame = CFrame.new(-5600, 90, -2200), MonsterArea = CFrame.new(-5550, 90, -2200)},
    {Level = 200, QuestNPC = "Fishman Warrior Quest Giver", Monster = "Fishman Warrior", NPC_CFrame = CFrame.new(-2850, 15, 5100), MonsterArea = CFrame.new(-2800, 15, 5100)},
}

-- // Função de Segurança: Obter PlayerData
local function GetPlayerData()
    local success, data = pcall(function()
        return LocalPlayer:WaitForChild("Data"):WaitForChild("Level")
    end)
    
    if success and data then
        return data
    else
        return nil
    end
end

-- // Função de Segurança: Obter Character com Retry
local function GetCharacter()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return LocalPlayer.Character
    else
        LocalPlayer.CharacterAdded:Wait()
        return LocalPlayer.Character
    end
end

-- // Função Anti-Kick: Teletransporte Seguro com TweenService
local function SafeTeleport(targetCFrame, speed)
    local character = GetCharacter()
    local hrp = character:WaitForChild("HumanoidRootPart")
    
    -- Instanciar BodyVelocity para evitar kick de velocidade
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(1, 1, 1) * math.huge
    bodyVelocity.Parent = hrp
    
    local tweenInfo = TweenInfo.new(
        (hrp.Position - targetCFrame.Position).Magnitude / (speed or _G.FarmSpeed),
        Enum.EasingStyle.Linear,
        Enum.EasingDirection.Out
    )
    
    local tween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
    
    local success, err = pcall(function()
        tween:Play()
        tween.Completed:Wait()
    end)
    
    -- Remover BodyVelocity após o trajeto
    if bodyVelocity then
        bodyVelocity:Destroy()
    end
    
    return success
end

-- // Função: Equipar Ferramenta Baseada no Tipo
local function EquipWeapon(weaponType)
    local character = GetCharacter()
    local humanoid = character:WaitForChild("Humanoid")
    
    local function findTool()
        local tools = {}
        
        -- Buscar na Backpack
        for _, tool in ipairs(Backpack:GetChildren()) do
            if tool:IsA("Tool") then
                if weaponType == "Melee" and tool.ToolTip and (string.find(tool.ToolTip:lower(), "combat") or string.find(tool.ToolTip:lower(), "fighting")) then
                    table.insert(tools, tool)
                elseif weaponType == "Sword" and tool.ToolTip and string.find(tool.ToolTip:lower(), "sword") then
                    table.insert(tools, tool)
                end
            end
        end
        
        -- Buscar no Character
        for _, tool in ipairs(character:GetChildren()) do
            if tool:IsA("Tool") then
                if weaponType == "Melee" and tool.ToolTip and (string.find(tool.ToolTip:lower(), "combat") or string.find(tool.ToolTip:lower(), "fighting")) then
                    table.insert(tools, tool)
                elseif weaponType == "Sword" and tool.ToolTip and string.find(tool.ToolTip:lower(), "sword") then
                    table.insert(tools, tool)
                end
            end
        end
        
        return tools[1]
    end
    
    local tool = findTool()
    if tool and tool.Parent ~= character then
        humanoid:EquipTool(tool)
    elseif tool and tool.Parent == character then
        -- Já está equipada
    end
end

-- // Função: Atacar Inimigo
local function AttackEnemy(target)
    if not target then return end
    
    local character = GetCharacter()
    local hrp = character:WaitForChild("HumanoidRootPart")
    
    -- Equipar arma
    EquipWeapon(_G.Weapon)
    
    -- Teleportar para o inimigo
    local targetHRP = target:FindFirstChild("HumanoidRootPart")
    if not targetHRP then return end
    
    local attackPosition = targetHRP.CFrame * CFrame.new(0, 0, 3)
    SafeTeleport(attackPosition, _G.FarmSpeed * 2)
    
    -- Atacar usando VirtualUser
    wait(0.1)
    VirtualUser:Button1Down(Vector2.new(850, 520))
    wait(0.05)
    VirtualUser:Button1Up(Vector2.new(850, 520))
end

-- // Função: Bring Mobs (Agrupar Monstros)
local function BringAllMobs(monsterName)
    local enemiesFolder = workspace:FindFirstChild("Enemies")
    if not enemiesFolder then return end
    
    local character = GetCharacter()
    local playerHRP = character:WaitForChild("HumanoidRootPart")
    local targetPosition = playerHRP.CFrame * CFrame.new(0, 0, 5)
    
    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
        if enemy.Name == monsterName and enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
            local humanoid = enemy.Humanoid
            if humanoid.Health > 0 then
                -- Desativar colisão
                for _, part in ipairs(enemy:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
                
                -- Teleportar monstro
                local enemyHRP = enemy.HumanoidRootPart
                enemyHRP.CFrame = targetPosition
            end
        end
    end
end

-- // Função: Auto Farm com Base no Level
local function StartAutoFarm()
    local levelData = GetPlayerData()
    if not levelData then return end
    
    local currentLevel = levelData.Value
    local config = nil
    
    -- Selecionar configuração baseada no level
    for i = #FarmConfig, 1, -1 do
        if currentLevel >= FarmConfig[i].Level then
            config = FarmConfig[i]
            break
        end
    end
    
    if not config then return end
    
    _G.AutoFarm = true
    
    task.spawn(function()
        while _G.AutoFarm do
            local success, err = pcall(function()
                -- Passo 1: Teleportar para o NPC da Quest
                SafeTeleport(config.NPC_CFrame, _G.FarmSpeed)
                wait(0.5)
                
                -- Passo 2: Iniciar Quest
                CommF:InvokeServer("StartQuest", config.Monster, 1)
                wait(0.3)
                
                -- Passo 3: Teleportar para área de monstros
                SafeTeleport(config.MonsterArea, _G.FarmSpeed)
                wait(0.3)
                
                -- Passo 4: Farm Loop
                local farmLoopStart = tick()
                while _G.AutoFarm and tick() - farmLoopStart < 30 do
                    local enemiesFolder = workspace:FindFirstChild("Enemies")
                    if not enemiesFolder then break end
                    
                    -- Bring Mobs se ativado
                    if _G.BringMobs then
                        BringAllMobs(config.Monster)
                        wait(0.5)
                    end
                    
                    -- Buscar inimigo mais próximo
                    local nearestEnemy = nil
                    local nearestDistance = math.huge
                    
                    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                        if enemy.Name == config.Monster and enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
                            local humanoid = enemy.Humanoid
                            if humanoid.Health > 0 then
                                local character = GetCharacter()
                                local playerPos = character:WaitForChild("HumanoidRootPart").Position
                                local distance = (enemy.HumanoidRootPart.Position - playerPos).Magnitude
                                
                                if distance < nearestDistance then
                                    nearestDistance = distance
                                    nearestEnemy = enemy
                                end
                            end
                        end
                    end
                    
                    if nearestEnemy then
                        AttackEnemy(nearestEnemy)
                        wait(0.2)
                    end
                    
                    wait()
                end
            end)
            
            if not success then
                warn("Erro no AutoFarm: " .. tostring(err))
            end
            
            wait(1)
        end
    end)
end

-- // Função: Auto Farm Vulcão (Magma Village)
local function StartMagmaFarm()
    _G.AutoFarm = true
    
    task.spawn(function()
        local militarySoldierCFrame = CFrame.new(-5408, 11, 8447)
        local militaryOfficerCFrame = CFrame.new(-5245, 11, 8475)
        
        while _G.AutoFarm do
            local success, err = pcall(function()
                -- Farm Military Soldier
                SafeTeleport(militarySoldierCFrame, _G.FarmSpeed)
                wait(0.3)
                
                for i = 1, 10 do
                    local enemiesFolder = workspace:FindFirstChild("Enemies")
                    if enemiesFolder then
                        for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                            if enemy.Name == "Military Soldier" and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                                if _G.BringMobs then
                                    BringAllMobs("Military Soldier")
                                end
                                AttackEnemy(enemy)
                                wait(0.3)
                            end
                        end
                    end
                    wait(0.5)
                end
                
                -- Farm Military Officer
                SafeTeleport(militaryOfficerCFrame, _G.FarmSpeed)
                wait(0.3)
                
                for i = 1, 10 do
                    local enemiesFolder = workspace:FindFirstChild("Enemies")
                    if enemiesFolder then
                        for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                            if enemy.Name == "Military Officer" and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                                if _G.BringMobs then
                                    BringAllMobs("Military Officer")
                                end
                                AttackEnemy(enemy)
                                wait(0.3)
                            end
                        end
                    end
                    wait(0.5)
                end
            end)
            
            if not success then
                warn("Erro no Magma Farm: " .. tostring(err))
            end
            
            wait(2)
        end
    end)
end

-- // Função: Fast Attack / Kill Aura
local function StartFastAttack()
    _G.FastAttack = true
    
    task.spawn(function()
        while _G.FastAttack do
            local success, err = pcall(function()
                if _G.KillAura then
                    -- Detectar inimigos no raio
                    local character = GetCharacter()
                    local playerPos = character:WaitForChild("HumanoidRootPart").Position
                    
                    local enemiesFolder = workspace:FindFirstChild("Enemies")
                    if enemiesFolder then
                        for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                            if enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
                                local humanoid = enemy.Humanoid
                                if humanoid.Health > 0 then
                                    local distance = (enemy.HumanoidRootPart.Position - playerPos).Magnitude
                                    
                                    if distance <= _G.AttackRange then
                                        -- Disparar ataque via Remote
                                        EquipWeapon(_G.Weapon)
                                        
                                        -- Método 1: Validator
                                        ValidatorEvent:FireServer()
                                        
                                        -- Método 2: VirtualUser Click
                                        wait(0.05)
                                        VirtualUser:Button1Down(Vector2.new(850, 520))
                                        wait(0.03)
                                        VirtualUser:Button1Up(Vector2.new(850, 520))
                                    end
                                end
                            end
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro no FastAttack: " .. tostring(err))
            end
            
            wait(0.001) -- Delay mínimo para ataques rápidos
        end
    end)
end

-- // Função: Aimbot / Silent Aim
local function StartAimbot()
    _G.Aimbot = true
    
    task.spawn(function()
        while _G.Aimbot do
            local success, err = pcall(function()
                local nearestPlayer = nil
                local nearestDistance = math.huge
                local character = GetCharacter()
                local myPosition = character:WaitForChild("HumanoidRootPart").Position
                
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local distance = (player.Character.HumanoidRootPart.Position - myPosition).Magnitude
                        
                        if distance < nearestDistance and distance <= _G.AttackRange then
                            nearestDistance = distance
                            nearestPlayer = player
                        end
                    end
                end
                
                if nearestPlayer and nearestPlayer.Character then
                    local targetHRP = nearestPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if targetHRP then
                        if _G.SilentAim then
                            -- Silent Aim: Redirecionar para linha de visão
                            local myHRP = character:WaitForChild("HumanoidRootPart")
                            local lookVector = (targetHRP.Position - myHRP.Position).Unit
                            local newCFrame = CFrame.new(myHRP.Position, myHRP.Position + lookVector)
                            
                            -- Aplicar rotação sem mover posição
                            myHRP.CFrame = CFrame.new(myHRP.Position) * CFrame.Angles(0, math.atan2(lookVector.X, lookVector.Z), 0)
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro no Aimbot: " .. tostring(err))
            end
            
            wait()
        end
    end)
end

-- // Função: Auto-Raid Solo
local function StartAutoRaid()
    _G.AutoRaid = true
    
    task.spawn(function()
        while _G.AutoRaid do
            local success, err = pcall(function()
                local character = GetCharacter()
                local position = character:WaitForChild("HumanoidRootPart").Position
                
                -- Verificar se está no laboratório
                local raidAreas = {
                    ["Flame"] = true,
                    ["Ice"] = true,
                    ["Quake"] = true,
                    ["Dark"] = true,
                    ["Light"] = true,
                    ["String"] = true,
                    ["Rumble"] = true,
                    ["Magma"] = true,
                    ["Human: Buddha"] = true,
                }
                
                -- Comprar chip e iniciar raid
                local currentArea = workspace:FindFirstChild("__THINGS"):FindFirstChild("FruitRaids")
                
                if not currentArea then
                    -- Comprar chip com Beli
                    CommF:InvokeServer("RaidsNpc", "Select")
                else
                    -- Dentro da dungeon: farmar salas
                    for wave = 1, 5 do
                        local enemiesFolder = workspace:FindFirstChild("Enemies")
                        if enemiesFolder then
                            local waveCleared = false
                            
                            while not waveCleared do
                                local allDead = true
                                
                                for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                                    if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                                        allDead = false
                                        AttackEnemy(enemy)
                                        wait(0.3)
                                    end
                                end
                                
                                if allDead then
                                    waveCleared = true
                                end
                                
                                wait(0.5)
                            end
                            
                            -- Aguardar portal abrir
                            wait(2)
                            
                            -- Mover para próxima ilha
                            local portals = workspace:FindFirstChild("Portals")
                            if portals and #portals:GetChildren() > 0 then
                                local portal = portals:GetChildren()[1]
                                if portal:FindFirstChild("CFrame") then
                                    SafeTeleport(portal.CFrame, _G.FarmSpeed)
                                    wait(1)
                                end
                            end
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro no AutoRaid: " .. tostring(err))
            end
            
            wait(5)
        end
    end)
end

-- // Função: Player ESP
local function ToggleESP(state)
    _G.ESPEnabled = state
    
    if not state then
        -- Remover ESP de todos
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = player.Character:FindFirstChild("ESPHighlight")
                if highlight then highlight:Destroy() end
                
                local billboard = player.Character:FindFirstChild("ESPBillboard")
                if billboard then billboard:Destroy() end
            end
        end
        return
    end
    
    task.spawn(function()
        while _G.ESPEnabled do
            local success, err = pcall(function()
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        -- Highlight
                        if not player.Character:FindFirstChild("ESPHighlight") then
                            local highlight = Instance.new("Highlight")
                            highlight.Name = "ESPHighlight"
                            highlight.FillColor = Color3.fromRGB(255, 0, 0)
                            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                            highlight.FillTransparency = 0.5
                            highlight.Parent = player.Character
                        end
                        
                        -- BillboardGui
                        if not player.Character:FindFirstChild("ESPBillboard") then
                            local billboard = Instance.new("BillboardGui")
                            billboard.Name = "ESPBillboard"
                            billboard.Size = UDim2.new(0, 200, 0, 50)
                            billboard.StudsOffset = Vector3.new(0, 3, 0)
                            billboard.AlwaysOnTop = true
                            billboard.Parent = player.Character
                            
                            local textLabel = Instance.new("TextLabel")
                            textLabel.Size = UDim2.new(1, 0, 1, 0)
                            textLabel.BackgroundTransparency = 1
                            textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                            textLabel.TextStrokeTransparency = 0
                            textLabel.Font = Enum.Font.GothamBold
                            textLabel.TextScaled = true
                            textLabel.Parent = billboard
                            
                            -- Atualizar informações
                            local function updateInfo()
                                local level = "?"
                                local health = "?"
                                
                                if player:FindFirstChild("Data") and player.Data:FindFirstChild("Level") then
                                    level = player.Data.Level.Value
                                end
                                
                                if player.Character:FindFirstChild("Humanoid") then
                                    health = math.floor(player.Character.Humanoid.Health)
                                end
                                
                                textLabel.Text = string.format("%s\nLv. %s | HP: %s", player.Name, level, health)
                            end
                            
                            updateInfo()
                            billboard:GetPropertyChangedSignal("Parent"):Connect(function()
                                if not billboard.Parent then
                                    return
                                end
                            end)
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro no ESP: " .. tostring(err))
            end
            
            wait(1)
        end
    end)
end

-- // Função: FPS Boost
local function ToggleFPSBoost(state)
    _G.FPSBoost = state
    
    if not state then return end
    
    local success, err = pcall(function()
        -- Limpar Decals e Textures
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("Decal") or obj:IsA("Texture") then
                obj:Destroy()
            elseif obj:IsA("ParticleEmitter") then
                obj.Enabled = false
            elseif obj:IsA("BasePart") then
                obj.Material = Enum.Material.SmoothPlastic
            end
        end
    end)
    
    if not success then
        warn("Erro no FPS Boost: " .. tostring(err))
    end
end

-- // ===== INTERFACE GRÁFICA (UI) =====
-- // Carregar Rayfield Library
local Rayfield = loadstring(game:HttpGet("https://raw.githubusercontent.com/SiriusSoftwareLtd/Rayfield/main/source.lua"))()

-- // Criar Janela Principal
local Window = Rayfield:CreateWindow({
    Name = "Rei Hub ⚡ Delta Edition",
    LoadingTitle = "Rei Hub - Carregando...",
    LoadingSubtitle = "by Rei Development",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "ReiHubConfig",
        FileName = "ReiHubSettings"
    },
    Discord = {
        Enabled = false
    },
    KeySystem = false
})

-- // Aba: Auto Farm
local AutoFarmTab = Window:CreateTab("Auto Farm", 4483362458)

AutoFarmTab:CreateSection("Farm Principal")

local FarmToggle = AutoFarmTab:CreateToggle({
    Name = "Auto Farm Level",
    CurrentValue = false,
    Flag = "AutoFarmToggle",
    Callback = function(Value)
        if Value then
            StartAutoFarm()
        else
            _G.AutoFarm = false
        end
    end,
})

AutoFarmTab:CreateToggle({
    Name = "Auto Farm Vulcão (Magma)",
    CurrentValue = false,
    Flag = "MagmaFarmToggle",
    Callback = function(Value)
        if Value then
            StartMagmaFarm()
        else
            _G.AutoFarm = false
        end
    end,
})

AutoFarmTab:CreateToggle({
    Name = "Bring Mobs (Agrupar)",
    CurrentValue = false,
    Flag = "BringMobsToggle",
    Callback = function(Value)
        _G.BringMobs = Value
    end,
})

-- // Aba: Combat
local CombatTab = Window:CreateTab("Combat", 4483362458)

CombatTab:CreateSection("Ataques")

CombatTab:CreateToggle({
    Name = "Fast Attack",
    CurrentValue = false,
    Flag = "FastAttackToggle",
    Callback = function(Value)
        if Value then
            StartFastAttack()
        else
            _G.FastAttack = false
        end
    end,
})

CombatTab:CreateToggle({
    Name = "Kill Aura",
    CurrentValue = false,
    Flag = "KillAuraToggle",
    Callback = function(Value)
        _G.KillAura = Value
        if not Value and not _G.SilentAim and not _G.Aimbot then
            _G.FastAttack = false
        end
    end,
})

CombatTab:CreateSlider({
    Name = "Alcance de Ataque",
    Range = {5, 100},
    Increment = 1,
    Suffix = "Studs",
    CurrentValue = 15,
    Flag = "AttackRange",
    Callback = function(Value)
        _G.AttackRange = Value
    end,
})

CombatTab:CreateSection("Aimbot")

CombatTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = false,
    Flag = "AimbotToggle",
    Callback = function(Value)
        if Value then
            _G.Aimbot = true
            StartAimbot()
        else
            _G.Aimbot = false
        end
    end,
})

CombatTab:CreateToggle({
    Name = "Silent Aim",
    CurrentValue = false,
    Flag = "SilentAimToggle",
    Callback = function(Value)
        _G.SilentAim = Value
        if Value and not _G.Aimbot then
            _G.Aimbot = true
            StartAimbot()
        elseif not Value and not _G.Aimbot then
            _G.Aimbot = false
        end
    end,
})

-- // Aba: Dungeon / Raid
local DungeonTab = Window:CreateTab("Dungeon / Raid", 4483362458)

DungeonTab:CreateSection("Auto Raid")

DungeonTab:CreateToggle({
    Name = "Auto-Raid Solo",
    CurrentValue = false,
    Flag = "AutoRaidToggle",
    Callback = function(Value)
        if Value then
            StartAutoRaid()
        else
            _G.AutoRaid = false
        end
    end,
})

DungeonTab:CreateParagraph({
    Title = "Instruções",
    Content = "Vá até o laboratório de frutas para iniciar a raid automaticamente."
})

-- // Aba: ESP / Players
local ESPTab = Window:CreateTab("ESP / Players", 4483362458)

ESPTab:CreateSection("Visual")

ESPTab:CreateToggle({
    Name = "Player ESP",
    CurrentValue = false,
    Flag = "ESPToggle",
    Callback = function(Value)
        ToggleESP(Value)
    end,
})

ESPTab:CreateParagraph({
    Title = "Informações",
    Content = "Highlight + Billboard com nome, level e vida dos jogadores."
})

-- // Aba: Configuração
local ConfigTab = Window:CreateTab("Configuração", 4483362458)

ConfigTab:CreateSection("Arma")

ConfigTab:CreateDropdown({
    Name = "Seleção de Arma",
    Options = {"Melee", "Sword"},
    CurrentOption = "Melee",
    Flag = "WeaponDropdown",
    Callback = function(Option)
        _G.Weapon = Option
    end,
})

ConfigTab:CreateSection("Velocidade")

ConfigTab:CreateSlider({
    Name = "Velocidade de Movimento",
    Range = {50, 500},
    Increment = 10,
    Suffix = "Studs/s",
    CurrentValue = 300,
    Flag = "FarmSpeed",
    Callback = function(Value)
        _G.FarmSpeed = Value
    end,
})

ConfigTab:CreateSection("Performance")

ConfigTab:CreateToggle({
    Name = "FPS Boost",
    CurrentValue = false,
    Flag = "FPSBoostToggle",
    Callback = function(Value)
        ToggleFPSBoost(Value)
    end,
})

-- // Inicialização
Rayfield:LoadConfiguration()

-- // Anti-AFK
LocalPlayer.Idled:Connect(function()
    VirtualUser:Button2Down(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
    wait(1)
    VirtualUser:Button2Up(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
end)

-- // Mensagem de Carregamento
print("✅ Rei Hub ⚡ Delta Edition carregado com sucesso!")
print("📱 Compatível com Delta Mobile/PC")
print("🛡️ Proteções ativas: Anti-Kick, pcall, verificação de integridade")
