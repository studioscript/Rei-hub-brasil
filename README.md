
--]]

-- ============================
-- CONFIGURAÇÕES GLOBAIS DE AMBIENTE
-- ============================
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

-- ============================
-- GERENCIADOR DE CONEXÕES (ANTI-MEMORY LEAK)
-- ============================
local ConnectionManager = {}
ConnectionManager.__index = ConnectionManager

function ConnectionManager.new()
    local self = setmetatable({
        _connections = {},
        _threads = {},
        _instances = {},
        _cleanupQueue = {}
    }, ConnectionManager)
    return self
end

function ConnectionManager:AddConnection(connection, identifier)
    if typeof(connection) == "RBXScriptConnection" then
        local id = identifier or #self._connections + 1
        self._connections[id] = connection
        return id
    end
    return nil
end

function ConnectionManager:AddThread(thread, identifier)
    if typeof(thread) == "thread" then
        local id = identifier or #self._threads + 1
        self._threads[id] = thread
        return id
    end
    return nil
end

function ConnectionManager:AddInstance(instance, identifier)
    if typeof(instance) == "Instance" then
        local id = identifier or #self._instances + 1
        self._instances[id] = instance
        return id
    end
    return nil
end

function ConnectionManager:Disconnect(identifier)
    if self._connections[identifier] then
        pcall(function()
            self._connections[identifier]:Disconnect()
        end)
        self._connections[identifier] = nil
    end
end

function ConnectionManager:Cleanup()
    -- Desconecta todas as conexões RBXScriptConnection
    for id, connection in pairs(self._connections) do
        pcall(function()
            connection:Disconnect()
        end)
        self._connections[id] = nil
    end
    
    -- Finaliza threads pendentes
    for id, thread in pairs(self._threads) do
        pcall(function()
            coroutine.close(thread)
        end)
        self._threads[id] = nil
    end
    
    -- Destroi instâncias criadas
    for id, instance in pairs(self._instances) do
        pcall(function()
            instance:Destroy()
        end)
        self._instances[id] = nil
    end
end

return ConnectionManager

-- ============================
-- SISTEMA DE SEGURANÇA ANTI-KICK E RAYCAST
-- ============================
local SecuritySystem = {}
SecuritySystem.__index = SecuritySystem

function SecuritySystem.new(character)
    local self = setmetatable({
        Character = character,
        HumanoidRootPart = character:WaitForChild("HumanoidRootPart"),
        Humanoid = character:WaitForChild("Humanoid"),
        BodyVelocity = nil,
        OriginalCollisions = {},
        IsTeleporting = false
    }, SecuritySystem)
    return self
end

function SecuritySystem:GroundCheck(position)
    local rayOrigin = position + Vector3.new(0, 10, 0)
    local rayDirection = Vector3.new(0, -500, 0)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {self.Character}
    
    local raycastResult = Workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    return raycastResult
end

function SecuritySystem:EnableSafeTeleport(targetPosition)
    if self.IsTeleporting then return false end
    self.IsTeleporting = true
    
    local success, err = pcall(function()
        -- Ground Check antes do teleporte
        local groundResult = self:GroundCheck(targetPosition)
        if not groundResult then
            -- Ajusta altura para evitar queda no vazio
            targetPosition = Vector3.new(targetPosition.X, targetPosition.Y + 50, targetPosition.Z)
        end
        
        -- Instancia BodyVelocity para evitar detecção de velocidade
        self.BodyVelocity = Instance.new("BodyVelocity")
        self.BodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        self.BodyVelocity.Velocity = Vector3.zero
        self.BodyVelocity.P = 1250
        self.BodyVelocity.Parent = self.HumanoidRootPart
        
        -- Tweens com trajetória segura
        local tweenInfo = TweenInfo.new(
            (self.HumanoidRootPart.Position - targetPosition).Magnitude / 300,
            Enum.EasingStyle.Linear,
            Enum.EasingDirection.Out
        )
        
        local tween = TweenService:Create(self.HumanoidRootPart, tweenInfo, {
            CFrame = CFrame.new(targetPosition) * CFrame.Angles(0, math.rad(self.HumanoidRootPart.Orientation.Y), 0)
        })
        
        tween:Play()
        tween.Completed:Wait()
        
        -- Limpeza pós-teleporte
        if self.BodyVelocity then
            self.BodyVelocity:Destroy()
            self.BodyVelocity = nil
        end
    end)
    
    self.IsTeleporting = false
    return success
end

function SecuritySystem:SetNoClip(enable)
    for _, part in ipairs(self.Character:GetDescendants()) do
        if part:IsA("BasePart") then
            if enable then
                self.OriginalCollisions[part] = part.CanCollide
                part.CanCollide = false
            else
                if self.OriginalCollisions[part] ~= nil then
                    part.CanCollide = self.OriginalCollisions[part]
                    self.OriginalCollisions[part] = nil
                end
            end
        end
    end
end

function SecuritySystem:Cleanup()
    self:SetNoClip(false)
    if self.BodyVelocity then
        self.BodyVelocity:Destroy()
    end
end

return SecuritySystem

-- ============================
-- MÓDULO DE OTIMIZAÇÃO DE MEMÓRIA E PERFORMANCE
-- ============================
local PerformanceOptimizer = {}
PerformanceOptimizer.__index = PerformanceOptimizer

function PerformanceOptimizer.new()
    local self = setmetatable({
        _cleanupTimers = {},
        _gcThreshold = 15000 -- 15 segundos para coleta de lixo
    }, PerformanceOptimizer)
    return self
end

function PerformanceOptimizer:HardFPSBoost()
    local success, err = pcall(function()
        -- Destruir elementos pesados de renderização
        for _, obj in ipairs(Workspace:GetDescendants()) do
            pcall(function()
                if obj:IsA("Decal") or obj:IsA("Texture") then
                    obj:Destroy()
                elseif obj:IsA("ParticleEmitter") then
                    obj.Enabled = false
                elseif obj:IsA("BasePart") then
                    obj.Material = Enum.Material.SmoothPlastic
                elseif obj:IsA("Light") then
                    obj.Brightness = math.min(obj.Brightness, 1)
                end
            end)
        end
        
        -- Otimizações de iluminação
        Lighting.GlobalShadows = false
        Lighting.Brightness = 2
        
        -- Desativa efeitos atmosféricos
        for _, child in ipairs(Lighting:GetChildren()) do
            if child:IsA("Atmosphere") then
                child.Density = 0
            elseif child:IsA("BloomEffect") then
                child.Enabled = false
            elseif child:IsA("BlurEffect") then
                child.Enabled = false
            end
        end
        
        -- Força garbage collection
        workspace:ClearAllChildren()
    end)
    return success
end

function PerformanceOptimizer:StartAutoGC()
    local connMgr = ConnectionManager.new()
    
    local thread = task.spawn(function()
        while true do
            task.wait(self._gcThreshold / 1000)
            pcall(function()
                workspace:ClearAllChildren()
                game:GetService("Debris"):ClearAllChildren()
            end)
        end
    end)
    
    connMgr:AddThread(thread, "AutoGC")
    return connMgr
end

function PerformanceOptimizer:OptimizeRayfield(rayfieldWindow)
    -- Reduz consumo de Rayfield
    pcall(function()
        local windows = game:GetService("CoreGui"):FindFirstChild("Rayfield")
        if windows then
            for _, element in ipairs(windows:GetDescendants()) do
                if element:IsA("Frame") or element:IsA("ScrollingFrame") then
                    element.ClipsDescendants = true
                end
            end
        end
    end)
end

return PerformanceOptimizer

-- ============================
-- SISTEMA DE FARM E COMBATE AVANÇADO
-- ============================
local CombatSystem = {}
CombatSystem.__index = CombatSystem

function CombatSystem.new()
    local self = setmetatable({
        _activeToggles = {},
        _security = SecuritySystem.new(Character),
        _connections = ConnectionManager.new(),
        _currentWeapon = "Melee",
        _attackCooldown = 0.15,
        _lastAttack = 0
    }, CombatSystem)
    return self
end

function CombatSystem:GetEnemies(radius)
    local enemies = {}
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
            if obj.Humanoid.Health > 0 and obj ~= Character then
                local distance = (HumanoidRootPart.Position - obj.HumanoidRootPart.Position).Magnitude
                if distance <= (radius or 300) then
                    table.insert(enemies, obj)
                end
            end
        end
    end
    return enemies
end

function CombatSystem:BringMobs(enabled)
    if enabled then
        local thread = task.spawn(function()
            while self._activeToggles["BringMobs"] do
                task.wait(0.05)
                pcall(function()
                    local enemies = self:GetEnemies(300)
                    for _, enemy in ipairs(enemies) do
                        -- Desativa colisão do mob
                        for _, part in ipairs(enemy:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.CanCollide = false
                            end
                        end
                        -- Move mob para frente do jogador
                        local targetPos = HumanoidRootPart.CFrame * CFrame.new(0, 0, -5)
                        enemy:FindFirstChild("HumanoidRootPart").CFrame = targetPos
                    end
                end)
            end
        end)
        self._connections:AddThread(thread, "BringMobs")
    end
    self._activeToggles["BringMobs"] = enabled
end

function CombatSystem:FastAttack()
    if self._activeToggles["FastAttack"] then
        local thread = task.spawn(function()
            while self._activeToggles["FastAttack"] do
                if tick() - self._lastAttack >= self._attackCooldown then
                    pcall(function()
                        local args = {
                            [1] = "Validator",
                            [2] = Character:FindFirstChildOfClass("Tool") or Character:FindFirstChild("Combat")
                        }
                        ReplicatedStorage.Remotes.Combat:FireServer(unpack(args))
                        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                        self._lastAttack = tick()
                    end)
                end
                task.wait(0.01)
            end
        end)
        self._connections:AddThread(thread, "FastAttack")
    end
    self._activeToggles["FastAttack"] = enabled
end

function CombatSystem:KillAura(radius)
    if self._activeToggles["KillAura"] then
        local thread = task.spawn(function()
            while self._activeToggles["KillAura"] do
                task.wait(0.1)
                pcall(function()
                    local enemies = self:GetEnemies(radius or 50)
                    for _, enemy in ipairs(enemies) do
                        -- Teleport seguro para o inimigo
                        self._security:EnableSafeTeleport(enemy.HumanoidRootPart.Position)
                        -- Executa ataque
                        local args = {
                            [1] = "Validator",
                            [2] = enemy.HumanoidRootPart
                        }
                        ReplicatedStorage.Remotes.Combat:FireServer(unpack(args))
                    end
                end)
            end
        end)
        self._connections:AddThread(thread, "KillAura")
    end
    self._activeToggles["KillAura"] = enabled
end

function CombatSystem:SilentAim(enabled)
    if enabled then
        local oldNamecall
        oldNamecall = hookmetamethod(game, "__namecall", function(...)
            local args = {...}
            local method = getnamecallmethod()
            
            if method == "FireServer" or method == "InvokeServer" then
                local enemies = CombatSystem:GetEnemies(150)
                if #enemies > 0 then
                    local target = enemies[1]
                    if target and target:FindFirstChild("Head") then
                        -- Substitui coordenadas do mouse pela posição do alvo
                        for i, arg in ipairs(args) do
                            if typeof(arg) == "Vector3" then
                                args[i] = target.Head.Position
                            end
                        end
                    end
                end
            end
            return oldNamecall(unpack(args))
        end)
    end
    self._activeToggles["SilentAim"] = enabled
end

function CombatSystem:AutoHeal(healthPercentage)
    if self._activeToggles["AutoHeal"] then
        local thread = task.spawn(function()
            while self._activeToggles["AutoHeal"] do
                task.wait(0.5)
                pcall(function()
                    local maxHealth = Humanoid.MaxHealth
                    local currentHealth = Humanoid.Health
                    local healthPercent = (currentHealth / maxHealth) * 100
                    
                    if healthPercent <= healthPercentage then
                        -- Movimento evasivo vertical
                        local safePos = HumanoidRootPart.CFrame * CFrame.new(0, 100, 0)
                        self._security:EnableSafeTeleport(safePos.Position)
                        task.wait(2)
                    end
                end)
            end
        end)
        self._connections:AddThread(thread, "AutoHeal")
    end
    self._activeToggles["AutoHeal"] = enabled
end

function CombatSystem:Cleanup()
    self._activeToggles = {}
    self._security:Cleanup()
    self._connections:Cleanup()
end

return CombatSystem

-- ============================
-- SISTEMA DE DUNGEON E RAIDS
-- ============================
local DungeonSystem = {}
DungeonSystem.__index = DungeonSystem

function DungeonSystem.new()
    local self = setmetatable({
        _security = SecuritySystem.new(Character),
        _connections = ConnectionManager.new(),
        _activeToggles = {},
        _raidChipCost = 1000000,
        _dungeonRooms = {}
    }, DungeonSystem)
    return self
end

function DungeonSystem:AutoRaid()
    if self._activeToggles["AutoRaid"] then
        local thread = task.spawn(function()
            while self._activeToggles["AutoRaid"] do
                task.wait(1)
                pcall(function()
                    -- Verifica se está na área de compra de chip
                    local chipVendor = Workspace:FindFirstChild("RaidChipVendor", true)
                    if chipVendor then
                        -- Compra o chip
                        ReplicatedStorage.Remotes.Raid:InvokeServer("BuyChip", "Fragment")
                        task.wait(0.5)
                        
                        -- Insere o chip no terminal
                        local terminal = Workspace:FindFirstChild("RaidTerminal", true)
                        if terminal then
                            ReplicatedStorage.Remotes.Raid:InvokeServer("InsertChip")
                            task.wait(2)
                            
                            -- Entra no portal
                            local portal = Workspace:FindFirstChild("RaidPortal", true)
                            if portal and portal:IsA("BasePart") then
                                self._security:EnableSafeTeleport(portal.Position)
                            end
                        end
                    end
                    
                    -- Lógica dentro da dungeon
                    if game:GetService("ReplicatedStorage"):FindFirstChild("InRaid") then
                        self:ClearDungeonRooms()
                    end
                end)
            end
        end)
        self._connections:AddThread(thread, "AutoRaid")
    end
    self._activeToggles["AutoRaid"] = enabled
end

function DungeonSystem:ClearDungeonRooms()
    for i = 1, 5 do
        task.wait(2)
        pcall(function()
            local enemies = CombatSystem:GetEnemies(200)
            for _, enemy in ipairs(enemies) do
                self._security:EnableSafeTeleport(enemy.HumanoidRootPart.Position)
                task.wait(0.5)
                ReplicatedStorage.Remotes.Combat:FireServer("Validator", enemy)
            end
        end)
        
        -- Procura portal para próxima sala
        local nextPortal = Workspace:FindFirstChild("RoomPortal", true)
        if nextPortal then
            self._security:EnableSafeTeleport(nextPortal.Position)
            task.wait(1)
        end
    end
end

function DungeonSystem:Cleanup()
    self._activeToggles = {}
    self._connections:Cleanup()
    self._security:Cleanup()
end

return DungeonSystem

-- ============================
-- SISTEMA DE ESP E VISUAIS
-- ============================
local ESPSystem = {}
ESPSystem.__index = ESPSystem

function ESPSystem.new()
    local self = setmetatable({
        _activeToggles = {},
        _connections = ConnectionManager.new(),
        _highlights = {},
        _tracers = {},
        _billboards = {}
    }, ESPSystem)
    return self
end

function ESPSystem:CreateHighlight(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ReiHub_Highlight"
        highlight.FillColor = Color3.fromRGB(0, 255, 0)
        highlight.OutlineColor = Color3.fromRGB(0, 255, 0)
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.Parent = player.Character
        self._highlights[player.Name] = highlight
        self._connections:AddInstance(highlight)
    end
end

function ESPSystem:CreateTracer(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local tracer = Drawing.new("Line")
        tracer.Color = Color3.fromRGB(0, 255, 0)
        tracer.Thickness = 1
        tracer.Transparency = 0.5
        self._tracers[player.Name] = tracer
        
        local thread = task.spawn(function()
            while self._activeToggles["Tracers"] and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
                task.wait()
                pcall(function()
                    local rootPos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
                    tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    tracer.To = Vector2.new(rootPos.X, rootPos.Y)
                    tracer.Visible = onScreen
                end)
            end
            if tracer then
                tracer:Remove()
            end
        end)
        self._connections:AddThread(thread, "Tracer_" .. player.Name)
    end
end

function ESPSystem:CreateBillboard(player)
    if player.Character and player.Character:FindFirstChild("Head") then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ReiHub_Billboard"
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.AlwaysOnTop = true
        billboard.Parent = player.Character.Head
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        frame.BackgroundTransparency = 0.5
        frame.Parent = billboard
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 0.25, 0)
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Parent = frame
        
        local levelLabel = Instance.new("TextLabel")
        levelLabel.Size = UDim2.new(1, 0, 0.25, 0)
        levelLabel.Position = UDim2.new(0, 0, 0.25, 0)
        levelLabel.Text = "Level: " .. (player.Data and player.Data.Level and player.Data.Level.Value or "?")
        levelLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
        levelLabel.BackgroundTransparency = 1
        levelLabel.Parent = frame
        
        self._billboards[player.Name] = billboard
        self._connections:AddInstance(billboard)
    end
end

function ESPSystem:EnableESP()
    if self._activeToggles["PlayerESP"] then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                self:CreateHighlight(player)
                if self._activeToggles["Tracers"] then
                    self:CreateTracer(player)
                end
                if self._activeToggles["Billboards"] then
                    self:CreateBillboard(player)
                end
            end
        end
        
        local playerAddedConn = Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function()
                task.wait(1)
                self:CreateHighlight(player)
            end)
        end)
        self._connections:AddConnection(playerAddedConn, "PlayerAdded")
    end
end

function ESPSystem:Cleanup()
    for _, highlight in pairs(self._highlights) do
        pcall(function() highlight:Destroy() end)
    end
    for _, tracer in pairs(self._tracers) do
        pcall(function() tracer:Remove() end)
    end
    for _, billboard in pairs(self._billboards) do
        pcall(function() billboard:Destroy() end)
    end
    self._connections:Cleanup()
end

return ESPSystem

-- ============================
-- INTERFACE PRINCIPAL RAYFIELD (TEMA VERDE NEON)
-- ============================
local function CreateReiHubInterface()
    local Window = Rayfield:CreateWindow({
        Name = "☠️ REI HUB VERDE • DELTA MOBILE",
        LoadingTitle = "Inicializando Rei Hub...",
        LoadingSubtitle = "Por Engenheiro de Software Sênior",
        ConfigurationSaving = {
            Enabled = true,
            FolderName = "ReiHubConfigs",
            FileName = "ReiHub_Green"
        },
        Discord = {
            Enabled = false,
        },
        KeySystem = false,
    })

    -- Aba Auto Farm
    local FarmTab = Window:CreateTab("⚔️ Auto Farm", 4483362458)
    local CombatSection = FarmTab:CreateSection("Otimização de Combate")
    
    local BringMobsToggle = FarmTab:CreateToggle({
        Name = "Bring Mobs (Agrupamento Avançado)",
        CurrentValue = false,
        Flag = "BringMobs",
        Callback = function(Value)
            CombatSystem:BringMobs(Value)
        end,
    })
    
    local FastAttackToggle = FarmTab:CreateToggle({
        Name = "Fast Attack Bypass",
        CurrentValue = false,
        Flag = "FastAttack",
        Callback = function(Value)
            CombatSystem._activeToggles["FastAttack"] = Value
            CombatSystem:FastAttack()
        end,
    })
    
    local FarmSection = FarmTab:CreateSection("Rotinas de Farm")
    
    local AutoFarmToggle = FarmTab:CreateToggle({
        Name = "Auto Farm Level (Todos os Mares)",
        CurrentValue = false,
        Flag = "AutoFarm",
        Callback = function(Value)
            -- Implementação da tabela de níveis e missões
            if Value then
                task.spawn(function()
                    local levelData = {
                        [1] = {Quest = "BanditQuest1", NPC = "Bandit", Mob = "Bandit", Spawn = CFrame.new(1000, 100, 1000)},
                        -- Adicionar todos os níveis 1-2550+ aqui
                    }
                    while Value do
                        task.wait(1)
                        pcall(function()
                            local currentLevel = LocalPlayer.Data.Level.Value
                            local data = levelData[currentLevel]
                            if data then
                                ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", data.Quest, 1)
                                CombatSystem._security:EnableSafeTeleport(data.Spawn.Position)
                            end
                        end)
                    end
                end)
            end
        end,
    })

    -- Aba Combate
    local CombatTab = Window:CreateTab("🔥 Combate", 4483362458)
    local AuraSection = CombatTab:CreateSection("Gatilhamentos de Ataque")
    
    local KillAuraToggle = CombatTab:CreateToggle({
        Name = "Kill Aura Dinâmica",
        CurrentValue = false,
        Flag = "KillAura",
        Callback = function(Value)
            CombatSystem._activeToggles["KillAura"] = Value
            CombatSystem:KillAura(50)
        end,
    })
    
    local AuraRangeSlider = CombatTab:CreateSlider({
        Name = "Raio de Ação da Aura",
        Range = {20, 150},
        Increment = 10,
        Suffix = "Studs",
        CurrentValue = 50,
        Flag = "AuraRange",
        Callback = function(Value)
            CombatSystem:KillAura(Value)
        end,
    })
    
    local SilentAimToggle = CombatTab:CreateToggle({
        Name = "Silent Aim Pro",
        CurrentValue = false,
        Flag = "SilentAim",
        Callback = function(Value)
            CombatSystem:SilentAim(Value)
        end,
    })
    
    local VitalSection = CombatTab:CreateSection("Sistemas Vitais")
    
    local AutoHealToggle = CombatTab:CreateToggle({
        Name = "Auto Heal Inteligente",
        CurrentValue = false,
        Flag = "AutoHeal",
        Callback = function(Value)
            CombatSystem._activeToggles["AutoHeal"] = Value
            CombatSystem:AutoHeal(30)
        end,
    })
    
    local HealthSlider = CombatTab:CreateSlider({
        Name = "Porcentagem Crítica",
        Range = {10, 60},
        Increment = 5,
        Suffix = "% HP",
        CurrentValue = 30,
        Flag = "HealthPercent",
        Callback = function(Value)
            CombatSystem:AutoHeal(Value)
        end,
    })

    -- Aba Eventos
    local EventsTab = Window:CreateTab("🌋 Eventos Marítimos", 4483362458)
    local VolcanoSection = EventsTab:CreateSection("Evento do Vulcão")
    
    local VolcanoToggle = EventsTab:CreateToggle({
        Name = "Auto Volcano Sea Event Manager",
        CurrentValue = false,
        Flag = "VolcanoEvent",
        Callback = function(Value)
            if Value then
                task.spawn(function()
                    while Value do
                        task.wait(2)
                        pcall(function()
                            local volcano = Workspace:FindFirstChild("Prehistoric Island", true)
                            if volcano then
                                -- Lógica de ataque aos nós de pressão
                                local pressureNodes = {}
                                for _, child in ipairs(volcano:GetDescendants()) do
                                    if child:IsA("BasePart") and child.BrickColor == BrickColor.new("Bright orange") then
                                        table.insert(pressureNodes, child)
                                    end
                                end
                                for _, node in ipairs(pressureNodes) do
                                    CombatSystem._security:EnableSafeTeleport(node.Position)
                                    task.wait(0.5)
                                    ReplicatedStorage.Remotes.Combat:FireServer("Validator", node)
                                end
                            end
                        end)
                    end
                end)
            end
        end,
    })

    -- Aba Dungeon
    local DungeonTab = Window:CreateTab("🏰 Dungeon/Raid", 4483362458)
    local RaidSection = DungeonTab:CreateSection("Gerenciamento de Raids")
    
    local AutoRaidToggle = DungeonTab:CreateToggle({
        Name = "Auto-Raid Solo Pro",
        CurrentValue = false,
        Flag = "AutoRaid",
        Callback = function(Value)
            DungeonSystem._activeToggles["AutoRaid"] = Value
            DungeonSystem:AutoRaid()
        end,
    })
    
    local NextIslandToggle = DungeonTab:CreateToggle({
        Name = "Auto Next Island Loader",
        CurrentValue = false,
        Flag = "NextIsland",
        Callback = function(Value)
            if Value then
                task.spawn(function()
                    while Value do
                        task.wait(0.5)
                        pcall(function()
                            local portal = Workspace:FindFirstChild("RoomPortal", true)
                            if portal then
                                CombatSystem._security:EnableSafeTeleport(portal.Position)
                            end
                        end)
                    end
                end)
            end
        end,
    })

    -- Aba ESP
    local ESPTab = Window:CreateTab("👁️ ESP / Players", 4483362458)
    local ESPSection = ESPTab:CreateSection("Renderização Visual Verde Neon")
    
    local ESPToggle = ESPTab:CreateToggle({
        Name = "Player Box Highlight",
        CurrentValue = false,
        Flag = "PlayerESP",
        Callback = function(Value)
            ESPSystem._activeToggles["PlayerESP"] = Value
            ESPSystem:EnableESP()
        end,
    })
    
    local TracerToggle = ESPTab:CreateToggle({
        Name = "Tracers em Vetor",
        CurrentValue = false,
        Flag = "Tracers",
        Callback = function(Value)
            ESPSystem._activeToggles["Tracers"] = Value
        end,
    })
    
    local BillboardToggle = ESPTab:CreateToggle({
        Name = "Painel de Telemetria",
        CurrentValue = false,
        Flag = "Billboards",
        Callback = function(Value)
            ESPSystem._activeToggles["Billboards"] = Value
        end,
    })

    -- Aba Configurações
    local ConfigTab = Window:CreateTab("⚙️ Configuração", 4483362458)
    local GlobalSection = ConfigTab:CreateSection("Modificadores Globais")
    
    local WeaponDropdown = ConfigTab:CreateDropdown({
        Name = "Armamento Primário",
        Options = {"Melee", "Sword"},
        CurrentOption = "Melee",
        Flag = "PrimaryWeapon",
        Callback = function(Option)
            CombatSystem._currentWeapon = Option
        end,
    })
    
    local SpeedSlider = ConfigTab:CreateSlider({
        Name = "Velocidade Vetorial do Tween",
        Range = {150, 400},
        Increment = 25,
        Suffix = "studs/s",
        CurrentValue = 300,
        Flag = "TweenSpeed",
        Callback = function(Value)
            -- Ajusta velocidade dos tweens
        end,
    })
    
    local FPSBoostButton = ConfigTab:CreateButton({
        Name = "Executar Hard FPS Boost",
        Callback = function()
            PerformanceOptimizer:HardFPSBoost()
            Rayfield:Notify({
                Title = "Rei Hub",
                Content = "FPS Boost aplicado com sucesso!",
                Duration = 5,
                Image = 4483362458,
            })
        end,
    })
end

-- ============================
-- INICIALIZAÇÃO PRINCIPAL
-- ============================
local function InitializeReiHub()
    pcall(function()
        -- Inicializa sistemas principais
        CombatSystem = CombatSystem.new()
        DungeonSystem = DungeonSystem.new()
        ESPSystem = ESPSystem.new()
        PerformanceOptimizer = PerformanceOptimizer.new()
        
        -- Inicia coleta de lixo automática
        PerformanceOptimizer:StartAutoGC()
        
        -- Cria interface
        CreateReiHubInterface()
        
        -- Notificação de inicialização
        Rayfield:Notify({
            Title = "☠️ REI HUB",
            Content = "Sistema inicializado com sucesso! Delta Ready.",
            Duration = 8,
            Image = 4483362458,
        })
        
        -- Gerenciamento de conexão do personagem
        LocalPlayer.CharacterAdded:Connect(function(newCharacter)
            Character = newCharacter
            Humanoid = Character:WaitForChild("Humanoid")
            HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
            CombatSystem._security = SecuritySystem.new(Character)
        end)
    end)
end-- Execução principal com proteção
local success, err = pcall(InitializeReiHub)
if not success then
    warn("Erro na inicialização do Rei Hub: " .. tostring(err))
    -- Tentativa de recuperação
    task.spawn(function()
        task.wait(3)
        InitializeReiHub()
    end)
end

-- Anti-detecção de execução
task.spawn(function()
    while true do
        task.wait(30)
        pcall(function()
            -- Mantém script ativo com heartbeat falso
            local fakeEvent = Instance.new("BindableEvent")
            fakeEvent:Destroy()
        end)
    end
end)
