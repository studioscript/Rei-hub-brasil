--[[
    ⚡ REI HUB - DELTA EDITION v2.0 ⚡
    Blox Fruits Ultimate Script - Paleta Verde Neon Premium
    Desenvolvido por: Engenharia Reversa Avançada
    Compatibilidade: Delta Mobile/PC - Roblox Engine Luau
    Proteção: Anti-Crash, Anti-Kick, Memory Management, Garbage Collection
--]]

-- // ============================================
-- // INICIALIZAÇÃO DE SERVIÇOS E CONSTANTES
-- // ============================================
local Services = {
    Players = game:GetService("Players"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    TweenService = game:GetService("TweenService"),
    VirtualUser = game:GetService("VirtualUser"),
    VirtualInputManager = game:GetService("VirtualInputManager"),
    TeleportService = game:GetService("TeleportService"),
    UserInputService = game:GetService("UserInputService"),
    RunService = game:GetService("RunService"),
    Lighting = game:GetService("Lighting"),
    Debris = game:GetService("Debris"),
    CollectionService = game:GetService("CollectionService"),
    Workspace = workspace,
}

-- // Referências Locais
local LocalPlayer = Services.Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Backpack = LocalPlayer:WaitForChild("Backpack")

-- // Remotes do Blox Fruits (Engenharia Reversa)
local Remotes = Services.ReplicatedStorage:WaitForChild("Remotes")
local CommF_ = Remotes:WaitForChild("CommF_")
local ValidatorEvent = Remotes:WaitForChild("Validator")
local SkillEvent = Remotes:FindFirstChild("Skill") or Remotes:FindFirstChild("ActivateSkill")
local DamageEvent = Remotes:FindFirstChild("Damage") or Remotes:FindFirstChild("ApplyDamage")

-- // Sistema de Gerenciamento de Memória (Garbage Collector Automatizado)
local ConnectionManager = {}
ConnectionManager.__index = ConnectionManager

function ConnectionManager.new()
    local self = setmetatable({
        connections = {},
        cleanupQueue = {},
    }, ConnectionManager)
    return self
end

function ConnectionManager:add(connection)
    if typeof(connection) == "RBXScriptConnection" then
        table.insert(self.connections, connection)
        return connection
    end
    return nil
end

function ConnectionManager:disconnectAll()
    for _, conn in ipairs(self.connections) do
        pcall(function()
            if conn and conn.Connected then
                conn:Disconnect()
            end
        end)
    end
    self.connections = {}
    self.cleanupQueue = {}
end

-- // Instância Global do ConnectionManager
local GlobalConnections = ConnectionManager.new()

-- // ============================================
-- // CONFIGURAÇÕES GLOBAIS (PALETA VERDE NEON)
-- // ============================================
local CONFIG = {
    -- Cores Verde Neon Premium
    PrimaryColor = Color3.fromRGB(0, 255, 0),      -- #00FF00 Verde Neon Puro
    SecondaryColor = Color3.fromRGB(0, 200, 0),    -- Verde Escuro
    AccentColor = Color3.fromRGB(50, 255, 50),     -- Verde Claro
    DarkColor = Color3.fromRGB(0, 30, 0),          -- Fundo Escuro
    DarkerColor = Color3.fromRGB(0, 15, 0),        -- Fundo Ultra Escuro
    TextColor = Color3.fromRGB(255, 255, 255),     -- Texto Branco
    GlowColor = Color3.fromRGB(0, 255, 100),       -- Verde Brilhante
    WarningColor = Color3.fromRGB(255, 255, 0),    -- Amarelo Alerta
    
    -- Variáveis de Estado
    Weapon = "Melee",
    FarmSpeed = 300,
    AttackRange = 50,
    KillAuraRange = 50,
    BringMobsRadius = 300,
    CriticalHealthPercent = 30,
    
    -- Toggles Principais
    AutoFarm = false,
    BringMobs = false,
    FastAttack = false,
    KillAura = false,
    SilentAim = false,
    Aimbot = false,
    AutoRaid = false,
    AutoNextIsland = false,
    ESPEnabled = false,
    TracersEnabled = false,
    TelemetryEnabled = false,
    FPSBoostEnabled = false,
    AutoHeal = false,
    VolcanoEvent = false,
    AutoLootBones = false,
    SwordMastery = false,
    NoClipEnabled = false,
}

-- // ============================================
-- // FUNÇÕES DE SEGURANÇA E BYPASS AVANÇADOS
-- // ============================================

-- // Função de Raycast Ground Check (Anti-Queda/Void)
local function RaycastGroundCheck(character)
    local success, result = pcall(function()
        local hrp = character and character:FindFirstChild("HumanoidRootPart")
        if not hrp then return nil end
        
        local rayOrigin = hrp.Position
        local rayDirection = Vector3.new(0, -50, 0)
        
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {character}
        raycastParams.IgnoreWater = false
        
        return Services.Workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    end)
    
    if success and result then
        return result.Position
    end
    return nil
end

-- // Função de Teletransporte Seguro com Anti-Kick e Ground Detection
local function SafeTeleport(targetCFrame, speed, useNoClip)
    local success, errorMsg = pcall(function()
        local character = LocalPlayer.Character
        if not character then return false end
        
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if not hrp then return false end
        
        -- // Armazenar estado original das partes
        local originalCollisions = {}
        if useNoClip or CONFIG.NoClipEnabled then
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    originalCollisions[part] = part.CanCollide
                    part.CanCollide = false
                end
            end
        end
        
        -- // BodyVelocity Anti-Kick (Força Infinita, Velocidade Zero)
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bodyVelocity.Parent = hrp
        
        -- // Calcular distância e duração do Tween
        local distance = (hrp.Position - targetCFrame.Position).Magnitude
        local speedValue = speed or CONFIG.FarmSpeed
        local duration = math.clamp(distance / speedValue, 0.1, 10)
        
        -- // Criar e executar Tween
        local tweenInfo = TweenInfo.new(
            duration,
            Enum.EasingStyle.Linear,
            Enum.EasingDirection.Out
        )
        
        local tween = Services.TweenService:Create(hrp, tweenInfo, {
            CFrame = targetCFrame
        })
        
        tween:Play()
        tween.Completed:Wait()
        
        -- // Ground Check (Verificar se não caiu no limbo)
        local groundPos = RaycastGroundCheck(character)
        if groundPos then
            hrp.CFrame = CFrame.new(targetCFrame.X, groundPos.Y + 5, targetCFrame.Z)
        end
        
        -- // Restaurar colisões
        if useNoClip or CONFIG.NoClipEnabled then
            for part, originalState in pairs(originalCollisions) do
                if part and part.Parent then
                    pcall(function()
                        part.CanCollide = originalState
                    end)
                end
            end
        end
        
        -- // Limpar BodyVelocity
        if bodyVelocity then
            bodyVelocity:Destroy()
        end
        
        return true
    end)
    
    if not success then
        warn("Erro no SafeTeleport: " .. tostring(errorMsg))
        return false
    end
    return true
end

-- // Função para obter Character com Retry
local function GetCharacter()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return LocalPlayer.Character
    else
        LocalPlayer.CharacterAdded:Wait()
        return LocalPlayer.Character
    end
end

-- // Função para obter Player Data com segurança
local function GetPlayerData()
    local success, data = pcall(function()
        local dataFolder = LocalPlayer:FindFirstChild("Data")
        if dataFolder then
            local level = dataFolder:FindFirstChild("Level")
            if level then
                return {
                    Level = level.Value,
                    Beli = dataFolder:FindFirstChild("Beli") and dataFolder.Beli.Value or 0,
                    Fragments = dataFolder:FindFirstChild("Fragments") and dataFolder.Fragments.Value or 0,
                }
            end
        end
        return nil
    end)
    return success and data or nil
end

-- // ============================================
-- // TABELA DE FARM COMPLETA (SEA 1, 2 E 3)
-- // ============================================
local FarmDatabase = {
    -- Sea 1 (Level 0-700)
    {
        MinLevel = 0, MaxLevel = 10, Sea = 1,
        QuestNPC = "Bandit Quest Giver", QuestName = "Bandit Quest 1",
        Monster = "Bandit", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(1059.93, 16.35, 1549.98),
        MonsterCFrame = CFrame.new(1199.31, 16.38, 1526.22),
    },
    {
        MinLevel = 10, MaxLevel = 30, Sea = 1,
        QuestNPC = "Monkey Quest Giver", QuestName = "Monkey Quest 1",
        Monster = "Monkey", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-1599.66, 36.84, 152.6),
        MonsterCFrame = CFrame.new(-1455.54, 24.25, 109.39),
    },
    {
        MinLevel = 30, MaxLevel = 50, Sea = 1,
        QuestNPC = "Pirate Quest Giver", QuestName = "Pirate Quest 1",
        Monster = "Pirate", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-1495.32, 23.75, 3889.02),
        MonsterCFrame = CFrame.new(-1447.98, 23.75, 3921.65),
    },
    {
        MinLevel = 50, MaxLevel = 80, Sea = 1,
        QuestNPC = "Brute Quest Giver", QuestName = "Brute Quest 1",
        Monster = "Brute", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-1746.47, 23.74, 4261.35),
        MonsterCFrame = CFrame.new(-1695.8, 23.76, 4221.65),
    },
    {
        MinLevel = 80, MaxLevel = 100, Sea = 1,
        QuestNPC = "Desert Bandit Quest Giver", QuestName = "Desert Bandit Quest 1",
        Monster = "Desert Bandit", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(896.49, 6.43, 4385.75),
        MonsterCFrame = CFrame.new(950.29, 6.42, 4369.47),
    },
    {
        MinLevel = 100, MaxLevel = 130, Sea = 1,
        QuestNPC = "Desert Officer Quest Giver", QuestName = "Desert Officer Quest 1",
        Monster = "Desert Officer", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(1575.17, 6.45, 4376.07),
        MonsterCFrame = CFrame.new(1613.71, 6.44, 4353.99),
    },
    {
        MinLevel = 130, MaxLevel = 160, Sea = 1,
        QuestNPC = "Snow Bandit Quest Giver", QuestName = "Snow Bandit Quest 1",
        Monster = "Snow Bandit", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-4976.89, 90.34, -2319.01),
        MonsterCFrame = CFrame.new(-4932.83, 90.39, -2259.6),
    },
    {
        MinLevel = 160, MaxLevel = 200, Sea = 1,
        QuestNPC = "Snowman Quest Giver", QuestName = "Snowman Quest 1",
        Monster = "Snowman", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-5610.88, 90.35, -2257.53),
        MonsterCFrame = CFrame.new(-5530.28, 90.46, -2202.16),
    },
    -- Sea 2 (Level 700-1500)
    {
        MinLevel = 200, MaxLevel = 250, Sea = 1,
        QuestNPC = "Fishman Warrior Quest Giver", QuestName = "Fishman Warrior Quest 1",
        Monster = "Fishman Warrior", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-2872.66, 14.53, 5151.71),
        MonsterCFrame = CFrame.new(-2826.61, 14.5, 5091.91),
    },
    {
        MinLevel = 700, MaxLevel = 850, Sea = 2,
        QuestNPC = "Raider Quest Giver", QuestName = "Raider Quest 1",
        Monster = "Raider", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-497.11, 30.4, 3590.27),
        MonsterCFrame = CFrame.new(-425.97, 30.4, 3625.79),
    },
    {
        MinLevel = 850, MaxLevel = 950, Sea = 2,
        QuestNPC = "Mercenary Quest Giver", QuestName = "Mercenary Quest 1",
        Monster = "Mercenary", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-1202.33, 46.01, 1385.48),
        MonsterCFrame = CFrame.new(-1165.17, 46.06, 1409.41),
    },
    {
        MinLevel = 950, MaxLevel = 1100, Sea = 2,
        QuestNPC = "Swan Pirate Quest Giver", QuestName = "Swan Pirate Quest 1",
        Monster = "Swan Pirate", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-2337.62, 8.14, 1124.65),
        MonsterCFrame = CFrame.new(-2281.74, 8.13, 1091.73),
    },
    {
        MinLevel = 1100, MaxLevel = 1250, Sea = 2,
        QuestNPC = "Factory Staff Quest Giver", QuestName = "Factory Staff Quest 1",
        Monster = "Factory Staff", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-271.27, 73.85, -380.11),
        MonsterCFrame = CFrame.new(-215.09, 73.92, -345.31),
    },
    {
        MinLevel = 1250, MaxLevel = 1500, Sea = 2,
        QuestNPC = "Marine Lieutenant Quest Giver", QuestName = "Marine Lieutenant Quest 1",
        Monster = "Marine Lieutenant", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-2998.88, 33.14, 3874.89),
        MonsterCFrame = CFrame.new(-2943.1, 33.13, 3824.64),
    },
    -- Sea 3 (Level 1500+)
    {
        MinLevel = 1500, MaxLevel = 1700, Sea = 3,
        QuestNPC = "Pirate Millionaire Quest Giver", QuestName = "Pirate Millionaire Quest 1",
        Monster = "Pirate Millionaire", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-658.32, 148.24, 1514.56),
        MonsterCFrame = CFrame.new(-593.57, 148.37, 1585.82),
    },
    {
        MinLevel = 1700, MaxLevel = 1900, Sea = 3,
        QuestNPC = "Pistol Billionaire Quest Giver", QuestName = "Pistol Billionaire Quest 1",
        Monster = "Pistol Billionaire", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-964.85, 148.23, 1517.57),
        MonsterCFrame = CFrame.new(-906.9, 148.32, 1559.05),
    },
    {
        MinLevel = 1900, MaxLevel = 2100, Sea = 3,
        QuestNPC = "Jungle Pirate Quest Giver", QuestName = "Jungle Pirate Quest 1",
        Monster = "Jungle Pirate", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-1641.69, 36.49, 102.52),
        MonsterCFrame = CFrame.new(-1540.92, 36.49, 63.86),
    },
    {
        MinLevel = 2100, MaxLevel = 2300, Sea = 3,
        QuestNPC = "Tomb Rider Quest Giver", QuestName = "Tomb Rider Quest 1",
        Monster = "Tomb Rider", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(-1092.55, 40.41, 2176.4),
        MonsterCFrame = CFrame.new(-1028.3, 40.4, 2215.84),
    },
    {
        MinLevel = 2300, MaxLevel = 2550, Sea = 3,
        QuestNPC = "Elite Pirate Quest Giver", QuestName = "Elite Pirate Quest 1",
        Monster = "Elite Pirate", MonsterLevel = "Any",
        QuestCFrame = CFrame.new(5364.55, 28.34, 3336.9),
        MonsterCFrame = CFrame.new(5412.51, 28.34, 3289.45),
    },
}

-- // Função para encontrar configuração de farm baseada no level
local function GetFarmConfig(level)
    for i = #FarmDatabase, 1, -1 do
        if level >= FarmDatabase[i].MinLevel then
            return FarmDatabase[i]
        end
    end
    return FarmDatabase[1]
end

-- // ============================================
-- // SISTEMA DE EQUIPAMENTO DE ARMAS
-- // ============================================
local function EquipWeapon(weaponType)
    local success, errorMsg = pcall(function()
        local character = GetCharacter()
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then return end
        
        local function findToolInContainer(container)
            local tools = {}
            for _, tool in ipairs(container:GetChildren()) do
                if tool:IsA("Tool") then
                    local toolTip = tool:FindFirstChild("ToolTip")
                    if toolTip then
                        local tipLower = toolTip.Value:lower()
                        if weaponType == "Melee" and (tipLower:find("combat") or tipLower:find("fighting style") or tipLower:find("blox fruit")) then
                            table.insert(tools, tool)
                        elseif weaponType == "Sword" and tipLower:find("sword") then
                            table.insert(tools, tool)
                        end
                    end
                end
            end
            return tools
        end
        
        local backpackTools = findToolInContainer(Backpack)
        local characterTools = findToolInContainer(character)
        
        local allTools = {}
        for _, tool in ipairs(backpackTools) do
            table.insert(allTools, tool)
        end
        for _, tool in ipairs(characterTools) do
            table.insert(allTools, tool)
        end
        
        if #allTools > 0 then
            local tool = allTools[1]
            if tool.Parent ~= character then
                humanoid:EquipTool(tool)
            end
        end
    end)
    
    if not success then
        warn("Erro ao equipar arma: " .. tostring(errorMsg))
    end
end

-- // ============================================
-- // SISTEMA BRING MOBS AVANÇADO (300 STUDS)
-- // ============================================
local function BringAllMobs(monsterName)
    local success, errorMsg = pcall(function()
        local character = GetCharacter()
        local playerHRP = character:FindFirstChild("HumanoidRootPart")
        if not playerHRP then return end
        
        local enemiesFolder = Services.Workspace:FindFirstChild("Enemies")
        if not enemiesFolder then return end
        
        local targetPosition = playerHRP.CFrame * CFrame.new(0, 0, 5)
        
        for _, enemy in ipairs(enemiesFolder:GetChildren()) do
            if enemy.Name == monsterName or monsterName == "All" then
                local enemyHRP = enemy:FindFirstChild("HumanoidRootPart")
                local humanoid = enemy:FindFirstChild("Humanoid")
                
                if enemyHRP and humanoid and humanoid.Health > 0 then
                    local distance = (enemyHRP.Position - playerHRP.Position).Magnitude
                    
                    if distance <= CONFIG.BringMobsRadius then
                        -- Desativar colisão para movimento suave
                        for _, part in ipairs(enemy:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.CanCollide = false
                            end
                        end
                        
                        -- Suavizar movimento do mob
                        local tweenInfo = TweenInfo.new(0.05, Enum.EasingStyle.Linear)
                        local tween = Services.TweenService:Create(enemyHRP, tweenInfo, {
                            CFrame = targetPosition
                        })
                        tween:Play()
                    end
                end
            end
        end
    end)
    
    if not success then
        warn("Erro no BringMobs: " .. tostring(errorMsg))
    end
end

-- // ============================================
-- // SISTEMA DE ATAQUE RÁPIDO COM BYPASS
-- // ============================================
local function PerformFastAttack(target)
    local success, errorMsg = pcall(function()
        if not target then return end
        
        local character = GetCharacter()
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        
        -- Teleportar para posição de ataque
        local targetHRP = target:FindFirstChild("HumanoidRootPart")
        if not targetHRP then return end
        
        local attackPos = targetHRP.CFrame * CFrame.new(0, 0, 2)
        SafeTeleport(attackPos, CONFIG.FarmSpeed * 2, true)
        
        -- Equipar arma
        EquipWeapon(CONFIG.Weapon)
        
        -- Método 1: Disparar Validator (Bypass de dano)
        pcall(function()
            ValidatorEvent:FireServer()
        end)
        
        -- Método 2: VirtualUser Click (Simulação física)
        task.wait(0.01)
        Services.VirtualUser:Button1Down(Vector2.new(850, 520))
        task.wait(0.01)
        Services.VirtualUser:Button1Up(Vector2.new(850, 520))
        
        -- Método 3: Aplicar dano direto se disponível
        if DamageEvent then
            pcall(function()
                DamageEvent:FireServer(target, 100)
            end)
        end
    end)
    
    if not success then
        warn("Erro no FastAttack: " .. tostring(errorMsg))
    end
end

-- // ============================================
-- // SISTEMA KILL AURA DINÂMICA
-- // ============================================
local function KillAuraLoop()
    task.spawn(function()
        while CONFIG.KillAura do
            local success, errorMsg = pcall(function()
                local character = GetCharacter()
                local playerPos = character:FindFirstChild("HumanoidRootPart").Position
                
                -- Verificar inimigos em Enemies
                local enemiesFolder = Services.Workspace:FindFirstChild("Enemies")
                if enemiesFolder then
                    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                        if CONFIG.KillAura and enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
                            local humanoid = enemy.Humanoid
                            if humanoid.Health > 0 then
                                local distance = (enemy.HumanoidRootPart.Position - playerPos).Magnitude
                                if distance <= CONFIG.KillAuraRange then
                                    PerformFastAttack(enemy)
                                end
                            end
                        end
                    end
                end
                
                -- Verificar Players se necessário
                if CONFIG.AttackPlayers then
                    for _, player in ipairs(Services.Players:GetPlayers()) do
                        if player ~= LocalPlayer and player.Character then
                            local targetHRP = player.Character:FindFirstChild("HumanoidRootPart")
                            local humanoid = player.Character:FindFirstChild("Humanoid")
                            if targetHRP and humanoid and humanoid.Health > 0 then
                                local distance = (targetHRP.Position - playerPos).Magnitude
                                if distance <= CONFIG.KillAuraRange then
                                    PerformFastAttack(player.Character)
                                end
                            end
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro KillAura: " .. tostring(errorMsg))
            end
            
            task.wait(0.001)
        end
    end)
end

-- // ============================================
-- // SISTEMA SILENT AIM PRO
-- // ============================================
local function SilentAimSystem()
    task.spawn(function()
        while CONFIG.SilentAim do
            local success, errorMsg = pcall(function()
                local nearestTarget = nil
                local nearestDistance = math.huge
                local character = GetCharacter()
                local myPos = character:FindFirstChild("HumanoidRootPart").Position
                
                -- Encontrar alvo mais próximo
                for _, player in ipairs(Services.Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character then
                        local targetHRP = player.Character:FindFirstChild("HumanoidRootPart")
                        local humanoid = player.Character:FindFirstChild("Humanoid")
                        if targetHRP and humanoid and humanoid.Health > 0 then
                            local distance = (targetHRP.Position - myPos).Magnitude
                            if distance < nearestDistance and distance <= CONFIG.AttackRange then
                                nearestDistance = distance
                                nearestTarget = player
                            end
                        end
                    end
                end
                
                -- Aplicar Silent Aim
                if nearestTarget and nearestTarget.Character then
                    local targetHead = nearestTarget.Character:FindFirstChild("Head") or 
                                      nearestTarget.Character:FindFirstChild("HumanoidRootPart")
                    if targetHead then
                        -- Redirecionar câmera e ataques para o alvo
                        local myHRP = character:FindFirstChild("HumanoidRootPart")
                        if myHRP then
                            local lookVector = (targetHead.Position - myHRP.Position).Unit
                            myHRP.CFrame = CFrame.lookAt(myHRP.Position, Vector3.new(targetHead.Position.X, myHRP.Position.Y, targetHead.Position.Z))
                            
                            -- Interceptar skills (se possível)
                            if SkillEvent then
                                -- Redirecionar projétil para o alvo
                            end
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro SilentAim: " .. tostring(errorMsg))
            end
            
            task.wait()
        end
    end)
end

-- // ============================================
-- // SISTEMA AUTO HEAL INTELIGENTE
-- // ============================================
local function AutoHealSystem()
    task.spawn(function()
        while CONFIG.AutoHeal do
            local success, errorMsg = pcall(function()
                local character = GetCharacter()
                local humanoid = character:FindFirstChild("Humanoid")
                if not humanoid then return end
                
                local maxHealth = humanoid.MaxHealth
                local currentHealth = humanoid.Health
                local healthPercent = (currentHealth / maxHealth) * 100
                
                if healthPercent <= CONFIG.CriticalHealthPercent then
                    -- Movimento evasivo
                    local hrp = character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        -- Subir 100 studs
                        local escapeCFrame = hrp.CFrame * CFrame.new(0, 100, 0)
                        SafeTeleport(escapeCFrame, CONFIG.FarmSpeed * 3, true)
                        
                        -- Aguardar regeneração
                        while humanoid.Health < maxHealth * 0.8 do
                            task.wait(0.5)
                            if not CONFIG.AutoHeal then break end
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro AutoHeal: " .. tostring(errorMsg))
            end
            
            task.wait(0.1)
        end
    end)
end

-- // ============================================
-- // SISTEMA AUTO FARM PRINCIPAL
-- // ============================================
local function AutoFarmSystem()
    task.spawn(function()
        while CONFIG.AutoFarm do
            local success, errorMsg = pcall(function()
                local playerData = GetPlayerData()
                if not playerData then
                    task.wait(1)
                    return
                end
                
                local level = playerData.Level
                local farmConfig = GetFarmConfig(level)
                
                -- Verificar se é Sword Mastery
                if CONFIG.SwordMastery then
                    CONFIG.Weapon = "Sword"
                end
                
                -- Passo 1: Teleportar para NPC da Quest
                SafeTeleport(farmConfig.QuestCFrame, CONFIG.FarmSpeed, false)
                task.wait(0.5)
                
                -- Passo 2: Aceitar Quest
                local questArgs = {farmConfig.QuestName, 1}
                CommF_:InvokeServer("StartQuest", farmConfig.QuestName, 1)
                task.wait(0.3)
                
                -- Passo 3: Ir para área de monstros
                SafeTeleport(farmConfig.MonsterCFrame, CONFIG.FarmSpeed, false)
                task.wait(0.3)
                
                -- Passo 4: Loop de Farm
                local farmStart = tick()
                while CONFIG.AutoFarm and (tick() - farmStart) < 45 do
                    -- Bring Mobs se ativado
                    if CONFIG.BringMobs then
                        BringAllMobs(farmConfig.Monster)
                    end
                    
                    -- Procurar inimigo mais próximo
                    local enemiesFolder = Services.Workspace:FindFirstChild("Enemies")
                    if enemiesFolder then
                        local nearestEnemy = nil
                        local minDistance = math.huge
                        local character = GetCharacter()
                        local myPos = character:FindFirstChild("HumanoidRootPart").Position
                        
                        for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                            if enemy.Name == farmConfig.Monster then
                                local humanoid = enemy:FindFirstChild("Humanoid")
                                local hrp = enemy:FindFirstChild("HumanoidRootPart")
                                if humanoid and hrp and humanoid.Health > 0 then
                                    local dist = (hrp.Position - myPos).Magnitude
                                    if dist < minDistance then
                                        minDistance = dist
                                        nearestEnemy = enemy
                                    end
                                end
                            end
                        end
                        
                        if nearestEnemy then
                            PerformFastAttack(nearestEnemy)
                        end
                    end
                    
                    task.wait(0.15)
                end
            end)
            
            if not success then
                warn("Erro no AutoFarm: " .. tostring(errorMsg))
            end
            
            task.wait(0.5)
        end
    end)
end

-- // ============================================
-- // SISTEMA AUTO VOLCANO SEA EVENT (PREHISTORIC ISLAND)
-- // ============================================
local function VolcanoEventSystem()
    task.spawn(function()
        while CONFIG.VolcanoEvent do
            local success, errorMsg = pcall(function()
                -- Procurar Prehistoric Island no workspace
                local prehistoricIsland = nil
                for _, obj in ipairs(Services.Workspace:GetChildren()) do
                    if obj.Name:find("Prehistoric") or obj.Name:find("Volcano") then
                        prehistoricIsland = obj
                        break
                    end
                end
                
                if prehistoricIsland then
                    -- Subprocesso 1: Atacar nós de pressão do vulcão
                    local pressureParts = {}
                    for _, part in ipairs(prehistoricIsland:GetDescendants()) do
                        if part:IsA("BasePart") and (part.Name:find("Pressure") or part.BrickColor == BrickColor.new("Bright orange")) then
                            table.insert(pressureParts, part)
                        end
                    end
                    
                    for _, part in ipairs(pressureParts) do
                        if CONFIG.VolcanoEvent then
                            local attackCFrame = CFrame.new(part.Position + Vector3.new(0, 5, 0))
                            SafeTeleport(attackCFrame, CONFIG.FarmSpeed * 2, true)
                            PerformFastAttack(part.Parent or part)
                            task.wait(0.1)
                        end
                    end
                    
                    -- Subprocesso 2: Eliminar Lava Golems
                    local enemiesFolder = Services.Workspace:FindFirstChild("Enemies")
                    if enemiesFolder then
                        for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                            if enemy.Name:find("Lava Golem") and CONFIG.VolcanoEvent then
                                local enemyHRP = enemy:FindFirstChild("HumanoidRootPart")
                                if enemyHRP then
                                    local floatCFrame = enemyHRP.CFrame * CFrame.new(0, 10, 0)
                                    SafeTeleport(floatCFrame, CONFIG.FarmSpeed * 3, true)
                                    PerformFastAttack(enemy)
                                end
                            end
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro VolcanoEvent: " .. tostring(errorMsg))
            end
            
            task.wait(0.5)
        end
    end)
end

-- // ============================================
-- // SISTEMA AUTO LOOT BONES & EGGS
-- // ============================================
local function AutoLootSystem()
    task.spawn(function()
        while CONFIG.AutoLootBones do
            local success, errorMsg = pcall(function()
                local character = GetCharacter()
                local playerHRP = character:FindFirstChild("HumanoidRootPart")
                if not playerHRP then return end
                
                -- Procurar itens looteáveis
                for _, obj in ipairs(Services.Workspace:GetDescendants()) do
                    if CONFIG.AutoLootBones and obj:IsA("BasePart") or obj:IsA("Model") then
                        local itemName = obj.Name:lower()
                        if itemName:find("bone") or itemName:find("egg") or itemName:find("dinosaur") or itemName:find("fossil") then
                            local itemPos = obj:IsA("BasePart") and obj.Position or obj:GetPivot().Position
                            local distance = (itemPos - playerHRP.Position).Magnitude
                            
                            if distance < 50 then
                                SafeTeleport(CFrame.new(itemPos), CONFIG.FarmSpeed * 4, true)
                                task.wait(0.1)
                                
                                -- Tocar no item para coletar
                                firetouchinterest(playerHRP, obj, 0)
                                firetouchinterest(playerHRP, obj, 1)
                            end
                        end
                    end
                end
            end)
            
            if not success then
                warn("Erro AutoLoot: " .. tostring(errorMsg))
            end
            
            task.wait(0.2)
        end
    end)
end

-- // ============================================
-- // SISTEMA AUTO RAID SOLO PRO
-- // ============================================
local function AutoRaidSystem()
    task.spawn(function()
        while CONFIG.AutoRaid do
            local success, errorMsg = pcall(function()
                local playerData = GetPlayerData()
                if not playerData then return end
                
                -- Verificar se está no laboratório
                local raidArea = Services.Workspace:FindFirstChild("__THINGS")
                if raidArea then
                    local fruitRaids = raidArea:FindFirstChild("FruitRaids")
                    if fruitRaids then
                        -- Dentro da raid, limpar salas
                        for wave = 1, 5 do
                            if not CONFIG.AutoRaid then break end
                            
                            local enemiesFolder = Services.Workspace:FindFirstChild("Enemies")
                            if enemiesFolder then
                                local waveComplete = false
                                
                                while not waveComplete and CONFIG.AutoRaid do
                                    local allDead = true
                                    
                                    for _, enemy in ipairs(enemiesFolder:GetChildren()) do
                                        local humanoid = enemy:FindFirstChild("Humanoid")
                                        if humanoid and humanoid.Health > 0 then
                                            allDead = false
                                            PerformFastAttack(enemy)
                                        end
                                    end
                                    
                                    if allDead then
                                        waveComplete = true
                                    end
                                    
                                    task.wait(0.3)
                                end
                            end
                            
                            -- Aguardar e ir para próxima ilha
                            if CONFIG.AutoNextIsland then
                                task.wait(2)
                                local portals = Services.Workspace:FindFirstChild("Portals")
                                if portals then
                                    for _, portal in ipairs(portals:GetChildren()) do
                                        if portal:FindFirstChild("CFrame") then
                                            SafeTeleport(portal.CFrame, CONFIG.FarmSpeed * 2, false)
                                            task.wait(1)
                                            break
                                        end
                                    end
                                end
                            end
                        end
                    else
                        -- Comprar chip se necessário
                        if playerData.Fragments >= 1000 then
                            CommF_:InvokeServer("RaidsNpc", "Select", "Flame")
                        elseif playerData.Beli >= 100000 then
                            CommF_:InvokeServer("Beli", "RaidsNpc")
                        end
                    end
                else
                    -- Teleportar para laboratório
                    local labCFrame = CFrame.new(-5550, 250, -4400)
                    SafeTeleport(labCFrame, CONFIG.FarmSpeed, false)
                end
            end)
            
            if not success then
                warn("Erro AutoRaid: " .. tostring(errorMsg))
            end
            
            task.wait(3)
        end
    end)
end

-- // ============================================
-- // SISTEMA ESP (PLAYER HIGHLIGHT + TRACERS + TELEMETRY)
-- // ============================================
local function ESPSystem()
    local espConnections = ConnectionManager.new()
    
    return function(enabled)
        if not enabled then
            espConnections:disconnectAll()
            -- Limpar ESP existente
            for _, player in ipairs(Services.Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    pcall(function()
                        local highlight = player.Character:FindFirstChild("ReiHub_Highlight")
                        if highlight then highlight:Destroy() end
                        
                        local billboard = player.Character:FindFirstChild("ReiHub_Billboard")
                        if billboard then billboard:Destroy() end
                        
                        local tracer = player.Character:FindFirstChild("ReiHub_Tracer")
                        if tracer then tracer:Destroy() end
                    end)
                end
            end
            return
        end
        
        task.spawn(function()
            while CONFIG.ESPEnabled do
                local success, errorMsg = pcall(function()
                    for _, player in ipairs(Services.Players:GetPlayers()) do
                        if player ~= LocalPlayer and player.Character then
                            local character = player.Character
                            local hrp = character:FindFirstChild("HumanoidRootPart")
                            if not hrp then continue end
                            
                            -- Highlight Verde Neon
                            if not character:FindFirstChild("ReiHub_Highlight") then
                                local highlight = Instance.new("Highlight")
                                highlight.Name = "ReiHub_Highlight"
                                highlight.FillColor = CONFIG.PrimaryColor
                                highlight.OutlineColor = CONFIG.GlowColor
                                highlight.FillTransparency = 0.3
                                highlight.OutlineTransparency = 0
                                highlight.Parent = character
                            end
                            
                            -- Tracers (Beam visual)
                            if CONFIG.TracersEnabled and not character:FindFirstChild("ReiHub_Tracer") then
                                local character2 = GetCharacter()
                                local myHRP = character2:FindFirstChild("HumanoidRootPart")
                                if myHRP then
                                    local beam = Instance.new("Beam")
                                    beam.Name = "ReiHub_Tracer"
                                    beam.Attachment0 = Instance.new("Attachment")
                                    beam.Attachment1 = Instance.new("Attachment")
                                    beam.Attachment0.Parent = myHRP
                                    beam.Attachment1.Parent = hrp
                                    beam.Color = ColorSequence.new(CONFIG.PrimaryColor)
                                    beam.Width0 = 0.1
                                    beam.Width1 = 0.1
                                    beam.Parent = character2
                                end
                            end
                            
                            -- Telemetria (BillboardGui)
                            if CONFIG.TelemetryEnabled and not character:FindFirstChild("ReiHub_Billboard") then
                                local billboard = Instance.new("BillboardGui")
                                billboard.Name = "ReiHub_Billboard"
                                billboard.Size = UDim2.new(0, 250, 0, 80)
                                billboard.StudsOffset = Vector3.new(0, 4, 0)
                                billboard.AlwaysOnTop = true
                                billboard.Parent = character
                                
                                local frame = Instance.new("Frame")
                                frame.Size = UDim2.new(1, 0, 1, 0)
                                frame.BackgroundColor3 = CONFIG.DarkColor
                                frame.BackgroundTransparency = 0.3
                                frame.BorderSizePixel = 0
                                frame.Parent = billboard
                                
                                local nameLabel = Instance.new("TextLabel")
                                nameLabel.Size = UDim2.new(1, 0, 0.4, 0)
                                nameLabel.BackgroundTransparency = 1
                                nameLabel.TextColor3 = CONFIG.PrimaryColor
                                nameLabel.TextStrokeTransparency = 0
                                nameLabel.Font = Enum.Font.GothamBold
                                nameLabel.TextScaled = true
                                nameLabel.Text = player.Name
                                nameLabel.Parent = frame
                                
                                local infoLabel = Instance.new("TextLabel")
                                infoLabel.Size = UDim2.new(1, 0, 0.6, 0)
                                infoLabel.Position = UDim2.new(0, 0, 0.4, 0)
                                infoLabel.BackgroundTransparency = 1
                                infoLabel.TextColor3 = CONFIG.TextColor
                                infoLabel.TextStrokeTransparency = 0
                                infoLabel.Font = Enum.Font.Gotham
                                infoLabel.TextScaled = true
                                infoLabel.Parent = frame
                                
                                -- Atualizar informações
                                task.spawn(function()
                                    while CONFIG.TelemetryEnabled and infoLabel.Parent do
                                        pcall(function()
                                            local level = "?"
                                            local fruit = "N/A"
                                            local health = "?"
                                            
                                            if player:FindFirstChild("Data") and player.Data:FindFirstChild("Level") then
                                                level = player.Data.Level.Value
                                            end
                                            
                                            if player.Character:FindFirstChild("Humanoid") then
                                                health = math.floor(player.Character.Humanoid.Health)
                                            end
                                            
                                            if player.Backpack then
                                                for _, item in ipairs(player.Backpack:GetChildren()) do
                                                    if item:IsA("Tool") and item:FindFirstChild("ToolTip") then
                                                        fruit = item.ToolTip.Value
                                                        break
                                                    end
                                                end
                                            end
                                            
                                            infoLabel.Text = string.format("Lv.%s | %s | HP:%s", level, fruit, health)
                                        end)
                                        task.wait(0.5)
                                    end
                                end)
                            end
                        end
                    end
                end)
                
                if not success then
                    warn("Erro ESP: " .. tostring(errorMsg))
                end
                
                task.wait(1)
            end
        end)
    end
end

-- // ============================================
-- // SISTEMA FPS BOOST HARD
-- // ============================================
local function HardFPSBoost()
    local success, errorMsg = pcall(function()
        -- Limpar Decals e Textures
        for _, obj in ipairs(Services.Workspace:GetDescendants()) do
            if obj:IsA("Decal") or obj:IsA("Texture") then
                obj:Destroy()
            elseif obj:IsA("ParticleEmitter") then
                obj.Enabled = false
            elseif obj:IsA("BasePart") then
                obj.Material = Enum.Material.SmoothPlastic
                obj.Reflectance = 0
            end
        end
        
        -- Desativar efeitos de iluminação
        Services.Lighting.GlobalShadows = false
        Services.Lighting.FogEnd = 1000
        Services.Lighting.Brightness = 1.5
        
        -- Desativar Atmosphere e Bloom
        for _, effect in ipairs(Services.Lighting:GetChildren()) do
            if effect:IsA("Atmosphere") or effect:IsA("BloomEffect") or effect:IsA("BlurEffect") then
                effect.Enabled = false
            end
        end
        
        -- Otimização de sombras
        sethiddenprop(Services.Lighting, "Technology", "Compatibility")
    end)
    
    if not success then
        warn("Erro FPS Boost: " .. tostring(errorMsg))
    end
end

-- // ============================================
-- // INTERFACE GRÁFICA (RAYFIELD LIBRARY)
-- // ============================================
local function CreateUI()
    -- Carregar Rayfield com paleta verde neon customizada
    local Rayfield = loadstring(game:HttpGet("https://raw.githubusercontent.com/SiriusSoftwareLtd/Rayfield/main/source.lua"))()
    
    local Window = Rayfield:CreateWindow({
        Name = "⚡ REI HUB - DELTA EDITION",
        LoadingTitle = "Rei Hub v2.0 - Verde Neon",
        LoadingSubtitle = "Engenharia Reversa Avançada",
        ConfigurationSaving = {
            Enabled = true,
            FolderName = "ReiHubV2",
            FileName = "Config"
        },
        Discord = {
            Enabled = false,
        },
        KeySystem = false,
    })
    
    -- // Aba: Auto Farm
    local FarmTab = Window:CreateTab("🚜 Auto Farm", 4483362458)
    
    FarmTab:CreateSection("Otimização de Combate")
    
    FarmTab:CreateToggle({
        Name = "Bring Mobs (Agrupamento 300 studs)",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.BringMobs = Value
        end,
    })
    
    FarmTab:CreateToggle({
        Name = "Fast Attack Bypass",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.FastAttack = Value
        end,
    })
    
    FarmTab:CreateSection("Rotinas de Farm")
    
    FarmTab:CreateToggle({
        Name = "Auto Farm Level (Tabela Completa)",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.AutoFarm = Value
            if Value then
                AutoFarmSystem()
            end
        end,
    })
    
    FarmTab:CreateToggle({
        Name = "Auto Farm Sword Mastery",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.SwordMastery = Value
            if Value then
                CONFIG.Weapon = "Sword"
            end
        end,
    })
    
    -- // Aba: Vulcão / Sea Events
    local VolcanoTab = Window:CreateTab("🌋 Vulcão / Events", 4483362458)
    
    VolcanoTab:CreateSection("Evento do Vulcão Marítimo")
    
    VolcanoTab:CreateToggle({
        Name = "Auto Volcano Sea Event Manager",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.VolcanoEvent = Value
            if Value then
                VolcanoEventSystem()
            end
        end,
    })
    
    VolcanoTab:CreateToggle({
        Name = "Auto Loot Dinosaur Bones & Eggs",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.AutoLootBones = Value
            if Value then
                AutoLootSystem()
            end
        end,
    })
    
    VolcanoTab:CreateParagraph({
        Title = "Instruções",
        Content = "Ative em servidor com Perigo 6 para farmar o evento pré-histórico."
    })
    
    -- // Aba: Combat
    local CombatTab = Window:CreateTab("⚔️ Combat", 4483362458)
    
    CombatTab:CreateSection("Gatilhamentos de Ataque")
    
    CombatTab:CreateToggle({
        Name = "Kill Aura Dinâmica",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.KillAura = Value
            if Value then
                KillAuraLoop()
            end
        end,
    })
    
    CombatTab:CreateSlider({
        Name = "Raio da Kill Aura",
        Range = {20, 150},
        Increment = 1,
        Suffix = "studs",
        CurrentValue = 50,
        Callback = function(Value)
            CONFIG.KillAuraRange = Value
        end,
    })
    
    CombatTab:CreateToggle({
        Name = "Silent Aim Pro",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.SilentAim = Value
            if Value then
                SilentAimSystem()
            end
        end,
    })
    
    CombatTab:CreateSection("Sistemas Vitais")
    
    CombatTab:CreateToggle({
        Name = "Auto Heal Inteligente",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.AutoHeal = Value
            if Value then
                AutoHealSystem()
            end
        end,
    })
    
    CombatTab:CreateSlider({
        Name = "Porcentagem Crítica de HP",
        Range = {10, 60},
        Increment = 1,
        Suffix = "%",
        CurrentValue = 30,
        Callback = function(Value)
            CONFIG.CriticalHealthPercent = Value
        end,
    })
    
    -- // Aba: Dungeon / Raid
    local RaidTab = Window:CreateTab("🏰 Dungeon / Raid", 4483362458)
    
    RaidTab:CreateSection("Gerenciamento de Raids")
    
    RaidTab:CreateToggle({
        Name = "Auto-Raid Solo Pro",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.AutoRaid = Value
            if Value then
                AutoRaidSystem()
            end
        end,
    })
    
    RaidTab:CreateToggle({
        Name = "Auto Next Island Loader",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.AutoNextIsland = Value
        end,
    })
    
    -- // Aba: ESP / Players
    local ESPTab = Window:CreateTab("👁️ ESP / Players", 4483362458)
    
    local espHandler = ESPSystem()
    
    ESPTab:CreateSection("Renderização Visual Verde Neon")
    
    ESPTab:CreateToggle({
        Name = "Player Box Highlight",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.ESPEnabled = Value
            espHandler(Value)
        end,
    })
    
    ESPTab:CreateToggle({
        Name = "Tracers em Vetor",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.TracersEnabled = Value
            espHandler(CONFIG.ESPEnabled)
        end,
    })
    
    ESPTab:CreateToggle({
        Name = "Painel de Telemetria",
        CurrentValue = false,
        Callback = function(Value)
            CONFIG.TelemetryEnabled = Value
            espHandler(CONFIG.ESPEnabled)
        end,
    })
    
    -- // Aba: Configuração
    local ConfigTab = Window:CreateTab("⚙️ Configuração", 4483362458)
    
    ConfigTab:CreateSection("Modificadores Globais")
    
    ConfigTab:CreateDropdown({
        Name = "Armamento Primário",
        Options = {"Melee", "Sword"},
        CurrentOption = "Melee",
        Callback = function(Option)
            CONFIG.Weapon = Option
        end,
    })
    
    ConfigTab:CreateSlider({
        Name = "Velocidade do Tween",
        Range = {150, 400},
        Increment = 10,
        Suffix = "studs/s",
        CurrentValue = 300,
        Callback = function(Value)
            CONFIG.FarmSpeed = Value
        end,
    })
    
    ConfigTab:CreateSlider({
        Name = "Alcance de Ataque",
        Range = {20, 200},
        Increment = 1,
        Suffix = "studs",
        CurrentValue = 50,
        Callback = function(Value)
            CONFIG.AttackRange = Value
        end,
    })
    
    ConfigTab:CreateButton({
        Name = "Executar Hard FPS Boost",
        Callback = function()
            HardFPSBoost()
        end,
    })
    
    -- Carregar configuração salva
    Rayfield:LoadConfiguration()
end

-- // ============================================
-- // INICIALIZAÇÃO DO SCRIPT
-- // ============================================
local function Initialize()
    print("=" :rep(50))
    print("REI HUB - v2.0 ")
    print(" Paleta Verde Neon Premium Ativada")
    print(" Anti-Crash e Garbage Collector Inicializados")
    print("=" :rep(50))
    
    -- Criar Interface
    CreateUI()
    
    -- Sistema Anti-AFK
    Services.Players.LocalPlayer.Idled:Connect(function()
        Services.VirtualUser:Button2Down(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        Services.VirtualUser:Button2Up(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
    end)
end

-- // Executar com proteção global
local success, err = pcall(Initialize)
if not success then
    warn("Erro crítico na inicialização: " .. tostring(err))
end

-- // Retornar tabela do script para debugging
return {
    CONFIG = CONFIG,
    ConnectionManager = GlobalConnections,
    Functions = {
        SafeTeleport = SafeTeleport,
        GetFarmConfig = GetFarmConfig,
        EquipWeapon = EquipWeapon,
        BringAllMobs = BringAllMobs,
        PerformFastAttack = PerformFastAttack,
    }
}
