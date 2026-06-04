-- =============================================
-- 🌿 REI HUB V3 | BLOX FRUITS
-- OTIMIZADO PARA DELTA EXECUTOR
-- INTERFACE: RAYFIELD (100% COMPATÍVEL)
-- =============================================

-- =============================================
-- CARREGAR RAYFIELD (MELHOR UI)
-- =============================================
local Rayfield = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Rayfield/main/source"))()

-- =============================================
-- DETECTAR MUNDO
-- =============================================
local placeId = game.PlaceId
local World1 = placeId == 2753915549 or placeId == 85211729168715
local World2 = placeId == 4442272183 or placeId == 79091703265657
local World3 = placeId == 7449423635 or placeId == 100117331123089
local CurrentWorld = World1 and "Sea 1" or (World2 and "Sea 2" or "Sea 3")

-- =============================================
-- SERVIÇOS
-- =============================================
local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")
local VirtualInput = game:GetService("VirtualInputManager")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- =============================================
-- CONFIGURAÇÕES
-- =============================================
_G.ReiHub = {
    -- Auto Farm
    AutoFarm = false,
    AutoAcceptQuest = true,
    SelectedWeapon = "Melee",
    FarmMethod = "Teleport",
    
    -- Auto Boss
    AutoBoss = false,
    SelectedBoss = "",
    
    -- Auto Stats
    AutoStats = false,
    StatType = "Melee",
    
    -- Haki
    AutoKen = false,
    AutoBuso = false,
    
    -- Movement
    SpeedEnabled = false,
    SpeedValue = 50,
    JumpEnabled = false,
    JumpValue = 50,
    NoClip = false,
    
    -- Coleta
    AutoChest = false,
    AutoCollectFruit = false,
    
    -- Bring
    AutoBring = false,
    BringRange = 250,
    
    -- Outros
    FullBright = false,
    AntiAFK = false,
}

-- =============================================
-- SALVAR CONFIG
-- =============================================
local SettingsFolder = "ReiHubV3"
local SettingsFile = "Config.json"

if makefolder and not isfolder(SettingsFolder) then makefolder(SettingsFolder) end

local function SaveConfig()
    if not writefile then return end
    pcall(function()
        writefile(SettingsFolder .. "/" .. SettingsFile, game:GetService("HttpService"):JSONEncode(_G.ReiHub))
    end)
end

local function LoadConfig()
    if isfile and isfile(SettingsFolder .. "/" .. SettingsFile) then
        pcall(function()
            local data = game:GetService("HttpService"):JSONDecode(readfile(SettingsFolder .. "/" .. SettingsFile))
            for k, v in pairs(data) do _G.ReiHub[k] = v end
        end)
    end
end
LoadConfig()

-- =============================================
-- FUNÇÕES AUXILIARES
-- =============================================
local function GetHRP()
    local char = Player.Character
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function Teleport(cf)
    local hrp = GetHRP()
    if hrp then hrp.CFrame = cf end
end

local function IsAlive(mob)
    local hum = mob and mob:FindFirstChild("Humanoid")
    return hum and hum.Health > 0
end

local function EquipWeapon(weapon)
    local char = Player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local tool = Player.Backpack:FindFirstChild(weapon) or char:FindFirstChild(weapon)
    if tool and tool.Parent ~= char then
        hum:EquipTool(tool)
    end
end

local function UseSkill(skill)
    VirtualInput:SendKeyEvent(true, skill, false, game)
    task.wait(0.05)
    VirtualInput:SendKeyEvent(false, skill, false, game)
end

-- =============================================
-- AUTO KEN (OBSERVATION HAKI)
-- =============================================
task.spawn(function()
    while task.wait(0.3) do
        if _G.ReiHub.AutoKen then
            pcall(function()
                ReplicatedStorage.Remotes.CommE:FireServer("Ken", true)
            end)
        end
    end
end)

-- =============================================
-- AUTO BUSO HAKI
-- =============================================
task.spawn(function()
    while task.wait(1) do
        if _G.ReiHub.AutoBuso then
            pcall(function()
                if not Player.Character:FindFirstChild("HasBuso") then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
                end
            end)
        end
    end
end)

-- =============================================
-- AUTO STATS
-- =============================================
local StatMap = { Melee = "Melee", Defense = "Defense", Sword = "Sword", Gun = "Gun", Devil = "Demon Fruit" }

task.spawn(function()
    while task.wait(0.3) do
        if _G.ReiHub.AutoStats and Player.Data.Points.Value > 0 then
            pcall(function()
                ReplicatedStorage.Remotes.CommF_:InvokeServer("AddPoint", StatMap[_G.ReiHub.StatType], 1)
            end)
        end
    end
end)

-- =============================================
-- SPEED HACK
-- =============================================
task.spawn(function()
    while task.wait(0.1) do
        if _G.ReiHub.SpeedEnabled then
            local char = Player.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum and hum.WalkSpeed ~= _G.ReiHub.SpeedValue then
                hum.WalkSpeed = _G.ReiHub.SpeedValue
            end
        end
    end
end)

-- =============================================
-- JUMP POWER
-- =============================================
task.spawn(function()
    while task.wait(0.1) do
        if _G.ReiHub.JumpEnabled then
            local char = Player.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum and hum.JumpPower ~= _G.ReiHub.JumpValue then
                hum.JumpPower = _G.ReiHub.JumpValue
            end
        end
    end
end)

-- =============================================
-- NO CLIP
-- =============================================
RunService.Stepped:Connect(function()
    if _G.ReiHub.NoClip then
        local char = Player.Character
        if char then
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end
end)

-- =============================================
-- ANTI AFK
-- =============================================
task.spawn(function()
    while task.wait(30) do
        if _G.ReiHub.AntiAFK then
            VirtualInput:SendKeyEvent(true, "W", false, game)
            task.wait(0.1)
            VirtualInput:SendKeyEvent(false, "W", false, game)
        end
    end
end)

-- =============================================
-- FULL BRIGHT
-- =============================================
task.spawn(function()
    while task.wait(1) do
        if _G.ReiHub.FullBright then
            Lighting.Ambient = Color3.new(1, 1, 1)
            Lighting.Brightness = 2
            Lighting.FogEnd = 9e9
        else
            Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
            Lighting.Brightness = 1
        end
    end
end)

-- =============================================
-- AUTO COLLECT FRUIT
-- =============================================
task.spawn(function()
    while task.wait(0.3) do
        if _G.ReiHub.AutoCollectFruit then
            pcall(function()
                for _, obj in pairs(Workspace:GetChildren()) do
                    if string.find(obj.Name, "Fruit") and obj:FindFirstChild("Handle") then
                        Teleport(obj.Handle.CFrame)
                        task.wait(0.1)
                        VirtualUser:CaptureController()
                        VirtualUser:Button1Down(Vector2.new(1280, 672))
                        task.wait(0.1)
                        VirtualUser:Button1Up(Vector2.new(1280, 672))
                    end
                end
            end)
        end
    end
end)

-- =============================================
-- AUTO COLLECT CHEST
-- =============================================
task.spawn(function()
    while task.wait(0.5) do
        if not _G.ReiHub.AutoChest then task.wait(1) continue end
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            local chests = game:GetService("CollectionService"):GetTagged("_ChestTagged")
            local closest, closestDist = nil, math.huge
            for _, chest in pairs(chests) do
                if not chest:GetAttribute("IsDisabled") then
                    local dist = (hrp.Position - chest:GetPivot().Position).Magnitude
                    if dist < closestDist then
                        closestDist = dist
                        closest = chest
                    end
                end
            end
            if closest then Teleport(closest:GetPivot()) end
        end)
    end
end)

-- =============================================
-- AUTO BRING MOBS
-- =============================================
task.spawn(function()
    while task.wait(0.5) do
        if not _G.ReiHub.AutoBring then task.wait(1) continue end
        if not _G.ReiHub.AutoFarm then continue end
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            for _, mob in pairs(Workspace.Enemies:GetChildren()) do
                local hum = mob:FindFirstChild("Humanoid")
                local root = mob:FindFirstChild("HumanoidRootPart")
                if hum and root and hum.Health > 0 then
                    if (root.Position - hrp.Position).Magnitude <= _G.ReiHub.BringRange then
                        local tween = TweenService:Create(root, TweenInfo.new(0.35), {CFrame = hrp.CFrame * CFrame.new(0, 0, 5)})
                        tween:Play()
                    end
                end
            end
        end)
    end
end)

-- =============================================
-- QUESTS DATABASE (SEA 1, 2 e 3)
-- =============================================
local Quests = {
    -- ========== SEA 1 ==========
    {Name = "Bandit", Lv = 1, Quest = "BanditQuest1", Qty = 1, Npc = CFrame.new(1045.96, 27.00, 1560.82), MobPos = CFrame.new(1120, 27, 1590), Mob = "Bandit"},
    {Name = "Monkey", Lv = 10, Quest = "JungleQuest", Qty = 1, Npc = CFrame.new(-1598.08, 35.55, 153.37), MobPos = CFrame.new(-1448.51, 67.85, 11.46), Mob = "Monkey"},
    {Name = "Gorilla", Lv = 15, Quest = "JungleQuest", Qty = 2, Npc = CFrame.new(-1598.08, 35.55, 153.37), MobPos = CFrame.new(-1129.88, 40.46, -525.42), Mob = "Gorilla"},
    {Name = "Pirate", Lv = 30, Quest = "BuggyQuest1", Qty = 1, Npc = CFrame.new(-1141.07, 4.10, 3831.54), MobPos = CFrame.new(-1103.51, 13.75, 3896.09), Mob = "Pirate"},
    {Name = "Brute", Lv = 40, Quest = "BuggyQuest1", Qty = 2, Npc = CFrame.new(-1141.07, 4.10, 3831.54), MobPos = CFrame.new(-1140.08, 14.80, 4322.92), Mob = "Brute"},
    {Name = "Desert Bandit", Lv = 60, Quest = "DesertQuest", Qty = 1, Npc = CFrame.new(894.48, 5.14, 4392.43), MobPos = CFrame.new(924.79, 6.44, 4481.58), Mob = "Desert Bandit"},
    {Name = "Desert Officer", Lv = 75, Quest = "DesertQuest", Qty = 2, Npc = CFrame.new(894.48, 5.14, 4392.43), MobPos = CFrame.new(1608.28, 8.61, 4371.00), Mob = "Desert Officer"},
    {Name = "Snow Bandit", Lv = 90, Quest = "SnowQuest", Qty = 1, Npc = CFrame.new(1389.74, 88.15, -1298.90), MobPos = CFrame.new(1354.34, 87.27, -1393.94), Mob = "Snow Bandit"},
    {Name = "Snowman", Lv = 100, Quest = "SnowQuest", Qty = 2, Npc = CFrame.new(1389.74, 88.15, -1298.90), MobPos = CFrame.new(1201.64, 144.57, -1550.06), Mob = "Snowman"},
    {Name = "Chief Petty Officer", Lv = 120, Quest = "MarineQuest2", Qty = 1, Npc = CFrame.new(-5039.58, 27.35, 4324.68), MobPos = CFrame.new(-4881.23, 22.65, 4273.75), Mob = "Chief Petty Officer"},
    {Name = "Sky Bandit", Lv = 150, Quest = "SkyQuest", Qty = 1, Npc = CFrame.new(-4839.53, 716.36, -2619.44), MobPos = CFrame.new(-4953.20, 295.74, -2899.22), Mob = "Sky Bandit"},
    {Name = "Dark Master", Lv = 175, Quest = "SkyQuest", Qty = 2, Npc = CFrame.new(-4839.53, 716.36, -2619.44), MobPos = CFrame.new(-5259.84, 391.39, -2229.03), Mob = "Dark Master"},
    {Name = "Prisoner", Lv = 190, Quest = "PrisonerQuest", Qty = 1, Npc = CFrame.new(5308.93, 1.65, 475.12), MobPos = CFrame.new(5098.97, -0.32, 474.23), Mob = "Prisoner"},
    {Name = "Dangerous Prisoner", Lv = 210, Quest = "PrisonerQuest", Qty = 2, Npc = CFrame.new(5308.93, 1.65, 475.12), MobPos = CFrame.new(5654.56, 15.63, 866.29), Mob = "Dangerous Prisoner"},
    {Name = "Toga Warrior", Lv = 250, Quest = "ColosseumQuest", Qty = 1, Npc = CFrame.new(-1580.04, 6.35, -2986.47), MobPos = CFrame.new(-1820.21, 51.68, -2740.66), Mob = "Toga Warrior"},
    {Name = "Gladiator", Lv = 275, Quest = "ColosseumQuest", Qty = 2, Npc = CFrame.new(-1580.04, 6.35, -2986.47), MobPos = CFrame.new(-1292.83, 56.38, -3339.03), Mob = "Gladiator"},
    {Name = "Military Soldier", Lv = 300, Quest = "MagmaQuest", Qty = 1, Npc = CFrame.new(-5313.37, 10.95, 8515.29), MobPos = CFrame.new(-5411.16, 11.08, 8454.29), Mob = "Military Soldier"},
    {Name = "Military Spy", Lv = 325, Quest = "MagmaQuest", Qty = 2, Npc = CFrame.new(-5313.37, 10.95, 8515.29), MobPos = CFrame.new(-5802.86, 86.26, 8828.85), Mob = "Military Spy"},
    {Name = "Fishman Warrior", Lv = 375, Quest = "FishmanQuest", Qty = 1, Npc = CFrame.new(61122.65, 18.49, 1569.39), MobPos = CFrame.new(60878.30, 18.48, 1543.75), Mob = "Fishman Warrior"},
    {Name = "Fishman Commando", Lv = 400, Quest = "FishmanQuest", Qty = 2, Npc = CFrame.new(61122.65, 18.49, 1569.39), MobPos = CFrame.new(61922.63, 18.48, 1493.93), Mob = "Fishman Commando"},
    {Name = "God's Guard", Lv = 450, Quest = "SkyExp1Quest", Qty = 1, Npc = CFrame.new(-4721.88, 843.87, -1949.96), MobPos = CFrame.new(-4710.04, 845.27, -1927.30), Mob = "God's Guard"},
    {Name = "Shanda", Lv = 475, Quest = "SkyExp1Quest", Qty = 2, Npc = CFrame.new(-7859.09, 5544.19, -381.47), MobPos = CFrame.new(-7678.48, 5566.40, -497.21), Mob = "Shanda"},
    {Name = "Royal Squad", Lv = 525, Quest = "SkyExp2Quest", Qty = 1, Npc = CFrame.new(-7906.81, 5634.66, -1411.99), MobPos = CFrame.new(-7624.25, 5658.13, -1467.35), Mob = "Royal Squad"},
    {Name = "Royal Soldier", Lv = 550, Quest = "SkyExp2Quest", Qty = 2, Npc = CFrame.new(-7906.81, 5634.66, -1411.99), MobPos = CFrame.new(-7836.75, 5645.66, -1790.62), Mob = "Royal Soldier"},
    {Name = "Galley Pirate", Lv = 625, Quest = "FountainQuest", Qty = 1, Npc = CFrame.new(5259.81, 37.35, 4050.02), MobPos = CFrame.new(5551.02, 78.90, 3930.41), Mob = "Galley Pirate"},
    {Name = "Galley Captain", Lv = 650, Quest = "FountainQuest", Qty = 2, Npc = CFrame.new(5259.81, 37.35, 4050.02), MobPos = CFrame.new(5441.95, 42.50, 4950.09), Mob = "Galley Captain"},
    -- ========== SEA 2 ==========
    {Name = "Raider", Lv = 700, Quest = "Area1Quest", Qty = 1, Npc = CFrame.new(-429.54, 71.77, 1836.18), MobPos = CFrame.new(-728.32, 52.77, 2345.77), Mob = "Raider"},
    {Name = "Mercenary", Lv = 725, Quest = "Area1Quest", Qty = 2, Npc = CFrame.new(-429.54, 71.77, 1836.18), MobPos = CFrame.new(-1004.32, 80.15, 1424.61), Mob = "Mercenary"},
    {Name = "Swan Pirate", Lv = 775, Quest = "Area2Quest", Qty = 1, Npc = CFrame.new(638.43, 71.76, 918.28), MobPos = CFrame.new(1068.66, 137.61, 1322.10), Mob = "Swan Pirate"},
    {Name = "Factory Staff", Lv = 800, Quest = "Area2Quest", Qty = 2, Npc = CFrame.new(632.69, 73.10, 918.66), MobPos = CFrame.new(73.07, 81.86, -27.47), Mob = "Factory Staff"},
    {Name = "Marine Lieutenant", Lv = 875, Quest = "MarineQuest3", Qty = 1, Npc = CFrame.new(-2440.79, 71.71, -3216.06), MobPos = CFrame.new(-2821.37, 75.89, -3070.08), Mob = "Marine Lieutenant"},
    {Name = "Marine Captain", Lv = 900, Quest = "MarineQuest3", Qty = 2, Npc = CFrame.new(-2440.79, 71.71, -3216.06), MobPos = CFrame.new(-1861.23, 80.17, -3254.69), Mob = "Marine Captain"},
    {Name = "Zombie", Lv = 950, Quest = "ZombieQuest", Qty = 1, Npc = CFrame.new(-5497.06, 47.59, -795.23), MobPos = CFrame.new(-5657.77, 78.96, -928.68), Mob = "Zombie"},
    {Name = "Vampire", Lv = 975, Quest = "ZombieQuest", Qty = 2, Npc = CFrame.new(-5497.06, 47.59, -795.23), MobPos = CFrame.new(-6037.66, 32.18, -1340.65), Mob = "Vampire"},
    {Name = "Snow Trooper", Lv = 1000, Quest = "SnowMountainQuest", Qty = 1, Npc = CFrame.new(609.85, 400.11, -5372.25), MobPos = CFrame.new(549.14, 427.38, -5563.69), Mob = "Snow Trooper"},
    {Name = "Winter Warrior", Lv = 1050, Quest = "SnowMountainQuest", Qty = 2, Npc = CFrame.new(609.85, 400.11, -5372.25), MobPos = CFrame.new(1142.74, 475.63, -5199.41), Mob = "Winter Warrior"},
    {Name = "Lab Subordinate", Lv = 1100, Quest = "IceSideQuest", Qty = 1, Npc = CFrame.new(-6064.06, 15.24, -4902.97), MobPos = CFrame.new(-5707.47, 15.95, -4513.39), Mob = "Lab Subordinate"},
    {Name = "Horned Warrior", Lv = 1125, Quest = "IceSideQuest", Qty = 2, Npc = CFrame.new(-6064.06, 15.24, -4902.97), MobPos = CFrame.new(-6341.36, 15.95, -5723.16), Mob = "Horned Warrior"},
    {Name = "Magma Ninja", Lv = 1175, Quest = "FireSideQuest", Qty = 1, Npc = CFrame.new(-5428.03, 15.06, -5299.43), MobPos = CFrame.new(-5449.67, 76.65, -5808.20), Mob = "Magma Ninja"},
    {Name = "Lava Pirate", Lv = 1200, Quest = "FireSideQuest", Qty = 2, Npc = CFrame.new(-5428.03, 15.06, -5299.43), MobPos = CFrame.new(-5213.33, 49.73, -4701.45), Mob = "Lava Pirate"},
    {Name = "Ship Deckhand", Lv = 1250, Quest = "ShipQuest1", Qty = 1, Npc = CFrame.new(1037.80, 125.09, 32911.60), MobPos = CFrame.new(1212.01, 150.79, 33059.24), Mob = "Ship Deckhand"},
    {Name = "Ship Engineer", Lv = 1275, Quest = "ShipQuest1", Qty = 2, Npc = CFrame.new(1037.80, 125.09, 32911.60), MobPos = CFrame.new(919.47, 43.54, 32779.96), Mob = "Ship Engineer"},
    {Name = "Ship Steward", Lv = 1300, Quest = "ShipQuest2", Qty = 1, Npc = CFrame.new(968.80, 125.09, 33244.12), MobPos = CFrame.new(919.43, 129.55, 33436.03), Mob = "Ship Steward"},
    {Name = "Ship Officer", Lv = 1325, Quest = "ShipQuest2", Qty = 2, Npc = CFrame.new(968.80, 125.09, 33244.12), MobPos = CFrame.new(1036.01, 181.43, 33315.72), Mob = "Ship Officer"},
    {Name = "Arctic Warrior", Lv = 1350, Quest = "FrostQuest", Qty = 1, Npc = CFrame.new(5667.65, 26.79, -6486.08), MobPos = CFrame.new(5966.24, 62.97, -6179.38), Mob = "Arctic Warrior"},
    {Name = "Snow Lurker", Lv = 1375, Quest = "FrostQuest", Qty = 2, Npc = CFrame.new(5667.65, 26.79, -6486.08), MobPos = CFrame.new(5407.07, 69.19, -6880.88), Mob = "Snow Lurker"},
    {Name = "Sea Soldier", Lv = 1425, Quest = "ForgottenQuest", Qty = 1, Npc = CFrame.new(-3054.44, 235.54, -10142.81), MobPos = CFrame.new(-3028.22, 64.67, -9775.42), Mob = "Sea Soldier"},
    {Name = "Water Fighter", Lv = 1450, Quest = "ForgottenQuest", Qty = 2, Npc = CFrame.new(-3054.44, 235.54, -10142.81), MobPos = CFrame.new(-3352.90, 285.01, -10534.84), Mob = "Water Fighter"},
    -- ========== SEA 3 ==========
    {Name = "Pirate Millionaire", Lv = 1500, Quest = "PiratePortQuest", Qty = 1, Npc = CFrame.new(-290.07, 42.90, 5581.59), MobPos = CFrame.new(-246.00, 47.31, 5584.10), Mob = "Pirate Millionaire"},
    {Name = "Pistol Billionaire", Lv = 1525, Quest = "PiratePortQuest", Qty = 2, Npc = CFrame.new(-290.07, 42.90, 5581.59), MobPos = CFrame.new(-187.33, 86.24, 6013.51), Mob = "Pistol Billionaire"},
    {Name = "Dragon Crew Warrior", Lv = 1575, Quest = "DragonCrewQuest", Qty = 1, Npc = CFrame.new(6737.06, 127.41, -712.30), MobPos = CFrame.new(6709.76, 52.34, -1139.02), Mob = "Dragon Crew Warrior"},
    {Name = "Dragon Crew Archer", Lv = 1600, Quest = "DragonCrewQuest", Qty = 2, Npc = CFrame.new(6737.06, 127.41, -712.30), MobPos = CFrame.new(6668.76, 481.37, 329.12), Mob = "Dragon Crew Archer"},
    {Name = "Hydra Enforcer", Lv = 1625, Quest = "VenomCrewQuest", Qty = 1, Npc = CFrame.new(5206.40, 1004.10, 748.35), MobPos = CFrame.new(4547.11, 1003.10, 334.19), Mob = "Hydra Enforcer"},
    {Name = "Venomous Assailant", Lv = 1650, Quest = "VenomCrewQuest", Qty = 2, Npc = CFrame.new(5206.40, 1004.10, 748.35), MobPos = CFrame.new(4674.92, 1134.82, 996.30), Mob = "Venomous Assailant"},
    {Name = "Marine Commodore", Lv = 1700, Quest = "MarineTreeIsland", Qty = 1, Npc = CFrame.new(2180.54, 27.81, -6741.54), MobPos = CFrame.new(2286.00, 73.13, -7159.80), Mob = "Marine Commodore"},
    {Name = "Marine Rear Admiral", Lv = 1725, Quest = "MarineTreeIsland", Qty = 2, Npc = CFrame.new(2179.98, 28.73, -6740.05), MobPos = CFrame.new(3656.77, 160.52, -7001.59), Mob = "Marine Rear Admiral"},
    {Name = "Fishman Raider", Lv = 1775, Quest = "DeepForestIsland3", Qty = 1, Npc = CFrame.new(-10581.65, 330.87, -8761.18), MobPos = CFrame.new(-10407.52, 331.76, -8368.51), Mob = "Fishman Raider"},
    {Name = "Fishman Captain", Lv = 1800, Quest = "DeepForestIsland3", Qty = 2, Npc = CFrame.new(-10581.65, 330.87, -8761.18), MobPos = CFrame.new(-10994.70, 352.38, -9002.11), Mob = "Fishman Captain"},
    {Name = "Forest Pirate", Lv = 1825, Quest = "DeepForestIsland", Qty = 1, Npc = CFrame.new(-13234.04, 331.48, -7625.40), MobPos = CFrame.new(-13274.47, 332.37, -7769.58), Mob = "Forest Pirate"},
    {Name = "Mythological Pirate", Lv = 1850, Quest = "DeepForestIsland", Qty = 2, Npc = CFrame.new(-13234.04, 331.48, -7625.40), MobPos = CFrame.new(-13680.60, 501.08, -6991.18), Mob = "Mythological Pirate"},
    {Name = "Jungle Pirate", Lv = 1900, Quest = "DeepForestIsland2", Qty = 1, Npc = CFrame.new(-12680.38, 389.97, -9902.01), MobPos = CFrame.new(-12256.16, 331.73, -10485.83), Mob = "Jungle Pirate"},
    {Name = "Musketeer Pirate", Lv = 1925, Quest = "DeepForestIsland2", Qty = 2, Npc = CFrame.new(-12680.38, 389.97, -9902.01), MobPos = CFrame.new(-13457.90, 391.54, -9859.17), Mob = "Musketeer Pirate"},
    {Name = "Reborn Skeleton", Lv = 1975, Quest = "HauntedQuest1", Qty = 1, Npc = CFrame.new(-9479.21, 141.21, 5566.09), MobPos = CFrame.new(-8763.72, 165.72, 6159.86), Mob = "Reborn Skeleton"},
    {Name = "Living Zombie", Lv = 2000, Quest = "HauntedQuest1", Qty = 2, Npc = CFrame.new(-9479.21, 141.21, 5566.09), MobPos = CFrame.new(-10144.13, 138.62, 5838.08), Mob = "Living Zombie"},
    {Name = "Demonic Soul", Lv = 2025, Quest = "HauntedQuest2", Qty = 1, Npc = CFrame.new(-9516.99, 172.01, 6078.46), MobPos = CFrame.new(-9505.87, 172.10, 6158.99), Mob = "Demonic Soul"},
    {Name = "Posessed Mummy", Lv = 2050, Quest = "HauntedQuest2", Qty = 2, Npc = CFrame.new(-9516.99, 172.01, 6078.46), MobPos = CFrame.new(-9582.02, 6.25, 6205.47), Mob = "Posessed Mummy"},
    {Name = "Peanut Scout", Lv = 2075, Quest = "NutsIslandQuest", Qty = 1, Npc = CFrame.new(-2104.39, 38.10, -10194.21), MobPos = CFrame.new(-2143.24, 47.72, -10029.99), Mob = "Peanut Scout"},
    {Name = "Peanut President", Lv = 2100, Quest = "NutsIslandQuest", Qty = 2, Npc = CFrame.new(-2104.39, 38.10, -10194.21), MobPos = CFrame.new(-1859.35, 38.10, -10422.42), Mob = "Peanut President"},
    {Name = "Ice Cream Chef", Lv = 2125, Quest = "IceCreamIslandQuest", Qty = 1, Npc = CFrame.new(-820.64, 65.81, -10965.79), MobPos = CFrame.new(-872.24, 65.81, -10919.95), Mob = "Ice Cream Chef"},
    {Name = "Ice Cream Commander", Lv = 2150, Quest = "IceCreamIslandQuest", Qty = 2, Npc = CFrame.new(-820.64, 65.81, -10965.79), MobPos = CFrame.new(-558.06, 112.04, -11290.77), Mob = "Ice Cream Commander"},
    {Name = "Cookie Crafter", Lv = 2200, Quest = "CakeQuest1", Qty = 1, Npc = CFrame.new(-2021.32, 37.79, -12028.72), MobPos = CFrame.new(-2374.13, 37.79, -12125.30), Mob = "Cookie Crafter"},
    {Name = "Cake Guard", Lv = 2225, Quest = "CakeQuest1", Qty = 2, Npc = CFrame.new(-2021.32, 37.79, -12028.72), MobPos = CFrame.new(-1598.30, 43.77, -12244.58), Mob = "Cake Guard"},
    {Name = "Baking Staff", Lv = 2250, Quest = "CakeQuest2", Qty = 1, Npc = CFrame.new(-1927.91, 37.79, -12842.53), MobPos = CFrame.new(-1887.80, 77.61, -12998.35), Mob = "Baking Staff"},
    {Name = "Head Baker", Lv = 2275, Quest = "CakeQuest2", Qty = 2, Npc = CFrame.new(-1927.91, 37.79, -12842.53), MobPos = CFrame.new(-2216.18, 82.88, -12869.29), Mob = "Head Baker"},
    {Name = "Cocoa Warrior", Lv = 2300, Quest = "ChocQuest1", Qty = 1, Npc = CFrame.new(233.22, 29.87, -12201.23), MobPos = CFrame.new(-21.55, 80.57, -12352.38), Mob = "Cocoa Warrior"},
    {Name = "Chocolate Bar Battler", Lv = 2325, Quest = "ChocQuest1", Qty = 2, Npc = CFrame.new(233.22, 29.87, -12201.23), MobPos = CFrame.new(582.59, 77.18, -12463.16), Mob = "Chocolate Bar Battler"},
    {Name = "Sweet Thief", Lv = 2350, Quest = "ChocQuest2", Qty = 1, Npc = CFrame.new(150.50, 30.69, -12774.50), MobPos = CFrame.new(165.18, 76.05, -12600.83), Mob = "Sweet Thief"},
    {Name = "Candy Rebel", Lv = 2375, Quest = "ChocQuest2", Qty = 2, Npc = CFrame.new(150.50, 30.69, -12774.50), MobPos = CFrame.new(134.86, 77.24, -12876.54), Mob = "Candy Rebel"},
    {Name = "Candy Pirate", Lv = 2400, Quest = "CandyQuest1", Qty = 1, Npc = CFrame.new(-1150.04, 20.37, -14446.33), MobPos = CFrame.new(-1310.50, 26.01, -14562.40), Mob = "Candy Pirate"},
    {Name = "Isle Outlaw", Lv = 2450, Quest = "TikiQuest1", Qty = 1, Npc = CFrame.new(-16548.81, 55.60, -172.81), MobPos = CFrame.new(-16479.90, 226.61, -300.31), Mob = "Isle Outlaw"},
    {Name = "Island Boy", Lv = 2475, Quest = "TikiQuest1", Qty = 2, Npc = CFrame.new(-16548.81, 55.60, -172.81), MobPos = CFrame.new(-16849.39, 192.86, -150.78), Mob = "Island Boy"},
    {Name = "Sun-kissed Warrior", Lv = 2500, Quest = "TikiQuest2", Qty = 1, Npc = CFrame.new(-16538, 55, 1049), MobPos = CFrame.new(-16347, 64, 984), Mob = "Sun-kissed Warrior"},
    {Name = "Isle Champion", Lv = 2525, Quest = "TikiQuest2", Qty = 2, Npc = CFrame.new(-16541.02, 57.30, 1051.46), MobPos = CFrame.new(-16602.10, 130.38, 1087.24), Mob = "Isle Champion"},
    {Name = "Serpent Hunter", Lv = 2551, Quest = "TikiQuest3", Qty = 1, Npc = CFrame.new(-16668.03, 105.32, 1568.60), MobPos = CFrame.new(-16645.64, 163.09, 1352.87), Mob = "Serpent Hunter"},
    {Name = "Skull Slayer", Lv = 2575, Quest = "TikiQuest3", Qty = 2, Npc = CFrame.new(-16668.03, 105.32, 1568.60), MobPos = CFrame.new(-16709.49, 419.68, 1751.09), Mob = "Skull Slayer"},
}

-- =============================================
-- PEGAR QUEST POR LEVEL
-- =============================================
local function GetCurrentQuest()
    local level = Player.Data.Level.Value
    local selected = nil
    for i = #Quests, 1, -1 do
        if level >= Quests[i].Lv then
            selected = Quests[i]
            break
        end
    end
    return selected
end

-- =============================================
-- AUTO FARM
-- =============================================
local function HasQuest()
    local main = Player.PlayerGui:FindFirstChild("Main")
    local quest = main and main:FindFirstChild("Quest")
    return quest and quest.Visible or false
end

local function AbandonQuest()
    pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("AbandonQuest") end)
end

local function GetQuest(data)
    if not data then return end
    Teleport(data.Npc)
    task.wait(0.5)
    if GetHRP() and (GetHRP().Position - data.Npc.Position).Magnitude <= 40 then
        task.wait(0.3)
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", data.Quest, data.Qty) end)
        task.wait(0.5)
    end
end

local function GetNearestMob(mobName)
    local hrp = GetHRP()
    if not hrp then return nil end
    local closest, closestDist = nil, math.huge
    for _, mob in pairs(Workspace.Enemies:GetChildren()) do
        if IsAlive(mob) and mob.Name == mobName then
            local root = mob:FindFirstChild("HumanoidRootPart") or mob:FindFirstChild("Torso")
            if root then
                local dist = (hrp.Position - root.Position).Magnitude
                if dist < closestDist then
                    closestDist = dist
                    closest = mob
                end
            end
        end
    end
    return closest
end

local function AttackMob(mob)
    if not mob or not IsAlive(mob) then return end
    local root = mob:FindFirstChild("HumanoidRootPart") or mob:FindFirstChild("Torso")
    if not root then return end
    
    EquipWeapon(_G.ReiHub.SelectedWeapon)
    
    if _G.ReiHub.FarmMethod == "Teleport" then
        Teleport(root.CFrame * CFrame.new(0, 15, 0))
    end
    
    task.wait(0.1)
    VirtualUser:CaptureController()
    VirtualUser:Button1Down(Vector2.new(1280, 672))
    task.wait(0.1)
    VirtualUser:Button1Up(Vector2.new(1280, 672))
end

task.spawn(function()
    while task.wait(0.3) do
        if not _G.ReiHub.AutoFarm then task.wait(1) continue end
        
        pcall(function()
            if not GetHRP() then return end
            
            local questData = GetCurrentQuest()
            if not questData then return end
            
            if not HasQuest() and _G.ReiHub.AutoAcceptQuest then
                AbandonQuest()
                task.wait(0.3)
                GetQuest(questData)
                task.wait(0.5)
                return
            end
            
            if HasQuest() then
                local mob = GetNearestMob(questData.Mob)
                if mob then
                    AttackMob(mob)
                else
                    Teleport(questData.MobPos)
                    task.wait(0.3)
                end
            end
        end)
    end
end)

-- =============================================
-- AUTO BOSS
-- =============================================
local Bosses = {}
if World1 then
    Bosses = {"The Gorilla King", "Bobby", "The Saw", "Yeti", "Vice Admiral", "Saber Expert", "Magma Admiral", "Fishman Lord", "Wysper", "Thunder God", "Cyborg", "Ice Admiral", "Greybeard"}
elseif World2 then
    Bosses = {"Diamond", "Jeremy", "Fajita", "Don Swan", "Smoke Admiral", "Awakened Ice Admiral", "Tide Keeper", "Darkbeard", "Cursed Captain", "Order"}
elseif World3 then
    Bosses = {"Stone", "Hydra Leader", "Kilo Admiral", "Captain Elephant", "Beautiful Pirate", "Cake Queen", "Longma", "Soul Reaper"}
end

task.spawn(function()
    while task.wait(0.5) do
        if not _G.ReiHub.AutoBoss or not _G.ReiHub.SelectedBoss then task.wait(1) continue end
        
        pcall(function()
            local boss = Workspace.Enemies:FindFirstChild(_G.ReiHub.SelectedBoss)
            if boss and IsAlive(boss) then
                local root = boss:FindFirstChild("HumanoidRootPart")
                if root then
                    Teleport(root.CFrame * CFrame.new(0, 20, 0))
                    AttackMob(boss)
                end
            end
        end)
    end
end)

-- =============================================
-- TELEPORTS RÁPIDOS
-- =============================================
local function RequestEntrance(pos)
    pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("requestEntrance", pos) end)
end

-- =============================================
-- INTERFACE RAYFIELD (COMPLETO)
-- =============================================
Rayfield:SetConfiguration({
    Theme = "Default",
    Font = "Inter",
    Keybind = Enum.KeyCode.RightShift,
})

local Window = Rayfield:CreateWindow({
    Name = "🌿 REI HUB V3 | " .. CurrentWorld,
    Icon = "🌿",
    LoadingTitle = "Carregando Rei Hub...",
    LoadingSubtitle = "Delta Edition",
})

-- ========== TAB: AUTO FARM ==========
local FarmTab = Window:CreateTab("⚔️ FARM")

FarmTab:CreateSection("⚙️ CONFIGURAÇÕES")

FarmTab:CreateDropdown({
    Name = "WEAPON",
    Options = {"Melee", "Sword", "Blox Fruit", "Gun"},
    CurrentOption = _G.ReiHub.SelectedWeapon,
    Callback = function(v)
        _G.ReiHub.SelectedWeapon = v
        SaveConfig()
    end,
})

FarmTab:CreateDropdown({
    Name = "FARM METHOD",
    Options = {"Teleport", "Walk"},
    CurrentOption = _G.ReiHub.FarmMethod,
    Callback = function(v)
        _G.ReiHub.FarmMethod = v
        SaveConfig()
    end,
})

FarmTab:CreateToggle({
    Name = "🔴 AUTO FARM",
    CurrentValue = _G.ReiHub.AutoFarm,
    Callback = function(v)
        _G.ReiHub.AutoFarm = v
        SaveConfig()
    end,
})

FarmTab:CreateToggle({
    Name = "📋 AUTO ACCEPT QUESTS",
    CurrentValue = _G.ReiHub.AutoAcceptQuest,
    Callback = function(v)
        _G.ReiHub.AutoAcceptQuest = v
        SaveConfig()
    end,
})

FarmTab:CreateToggle({
    Name = "🔄 AUTO BRING MOBS",
    CurrentValue = _G.ReiHub.AutoBring,
    Callback = function(v)
        _G.ReiHub.AutoBring = v
        SaveConfig()
    end,
})

FarmTab:CreateSlider({
    Name = "📏 BRING RANGE",
    Range = {50, 500},
    Increment = 10,
    CurrentValue = _G.ReiHub.BringRange,
    Callback = function(v)
        _G.ReiHub.BringRange = v
        SaveConfig()
    end,
})

-- ========== TAB: AUTO BOSS ==========
local BossTab = Window:CreateTab("👑 BOSS")

BossTab:CreateSection("🎯 BOSS CONFIG")

BossTab:CreateDropdown({
    Name = "SELECT BOSS",
    Options = Bosses,
    CurrentOption = _G.ReiHub.SelectedBoss or Bosses[1],
    Callback = function(v)
        _G.ReiHub.SelectedBoss = v
        SaveConfig()
    end,
})

BossTab:CreateToggle({
    Name = "🔴 AUTO BOSS",
    CurrentValue = _G.ReiHub.AutoBoss,
    Callback = function(v)
        _G.ReiHub.AutoBoss = v
        SaveConfig()
    end,
})

-- ========== TAB: STATS ==========
local StatsTab = Window:CreateTab("📊 STATS")

StatsTab:CreateSection("📈 STATS CONFIG")

StatsTab:CreateDropdown({
    Name = "STAT TYPE",
    Options = {"Melee", "Defense", "Sword", "Gun", "Devil"},
    CurrentOption = _G.ReiHub.StatType,
    Callback = function(v)
        _G.ReiHub.StatType = v
        SaveConfig()
    end,
})

StatsTab:CreateToggle({
    Name = "🔴 AUTO STATS",
    CurrentValue = _G.ReiHub.AutoStats,
    Callback = function(v)
        _G.ReiHub.AutoStats = v
        SaveConfig()
    end,
})

-- ========== TAB: HAKI ==========
local HakiTab = Window:CreateTab("🌀 HAKI")

HakiTab:CreateSection("👁️ OBSERVATION HAKI")

HakiTab:CreateToggle({
    Name = "🔴 AUTO KEN",
    CurrentValue = _G.ReiHub.AutoKen,
    Callback = function(v)
        _G.ReiHub.AutoKen = v
        SaveConfig()
    end,
})

HakiTab:CreateSection("🛡️ ARMAMENT HAKI")

HakiTab:CreateToggle({
    Name = "🔴 AUTO BUSO HAKI",
    CurrentValue = _G.ReiHub.AutoBuso,
    Callback = function(v)
        _G.ReiHub.AutoBuso = v
        SaveConfig()
    end,
})

-- ========== TAB: MOVEMENT ==========
local MoveTab = Window:CreateTab("🏃 MOVEMENT")

MoveTab:CreateSection("💨 SPEED HACK")

MoveTab:CreateToggle({
    Name = "🔴 SPEED HACK",
    CurrentValue = _G.ReiHub.SpeedEnabled,
    Callback = function(v)
        _G.ReiHub.SpeedEnabled = v
        SaveConfig()
    end,
})

MoveTab:CreateSlider({
    Name = "SPEED VALUE",
    Range = {16, 250},
    Increment = 1,
    CurrentValue = _G.ReiHub.SpeedValue,
    Callback = function(v)
        _G.ReiHub.SpeedValue = v
        SaveConfig()
    end,
})

MoveTab:CreateSection("🦘 JUMP POWER")

MoveTab:CreateToggle({
    Name = "🔴 JUMP HACK",
    CurrentValue = _G.ReiHub.JumpEnabled,
    Callback = function(v)
        _G.ReiHub.JumpEnabled = v
        SaveConfig()
    end,
})

MoveTab:CreateSlider({
    Name = "JUMP VALUE",
    Range = {50, 500},
    Increment = 10,
    CurrentValue = _G.ReiHub.JumpValue,
    Callback = function(v)
        _G.ReiHub.JumpValue = v
        SaveConfig()
    end,
})

MoveTab:CreateSection("🌀 NO CLIP")

MoveTab:CreateToggle({
    Name = "🔴 NO CLIP",
    CurrentValue = _G.ReiHub.NoClip,
    Callback = function(v)
        _G.ReiHub.NoClip = v
        SaveConfig()
    end,
})

-- ========== TAB: COLLECT ==========
local CollectTab = Window:CreateTab("💰 COLLECT")

CollectTab:CreateSection("🎁 AUTO COLLECT")

CollectTab:CreateToggle({
    Name = "🔴 AUTO COLLECT CHEST",
    CurrentValue = _G.ReiHub.AutoChest,
    Callback = function(v)
        _G.ReiHub.AutoChest = v
        SaveConfig()
    end,
})

CollectTab:CreateToggle({
    Name = "🔴 AUTO COLLECT FRUIT",
    CurrentValue = _G.ReiHub.AutoCollectFruit,
    Callback = function(v)
        _G.ReiHub.AutoCollectFruit = v
        SaveConfig()
    end,
})

-- ========== TAB: TELEPORT ==========
local TeleportTab = Window:CreateTab("🌍 TELEPORT")

TeleportTab:CreateSection("🌊 MUNDOS")

TeleportTab:CreateButton({
    Name = "🌊 SEA 1",
    Callback = function()
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelMain") end)
    end,
})

TeleportTab:CreateButton({
    Name = "🌊 SEA 2",
    Callback = function()
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelDressrosa") end)
    end,
})

TeleportTab:CreateButton({
    Name = "🌊 SEA 3",
    Callback = function()
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelZou") end)
    end,
})

if World2 then
    TeleportTab:CreateSection("🚢 SEA 2 - ILHAS")
    TeleportTab:CreateButton({ Name = "🚢 CURSED SHIP", Callback = function() Teleport(CFrame.new(916.92, 181.09, 33422)) end })
    TeleportTab:CreateButton({ Name = "🏰 SWAN ROOM", Callback = function() Teleport(CFrame.new(2286.20, 15.17, 863.83)) end })
    TeleportTab:CreateButton({ Name = "🔥 FACTORY", Callback = function() Teleport(CFrame.new(448.46, 199.35, -441.38)) end })
elseif World3 then
    TeleportTab:CreateSection("🐉 SEA 3 - ILHAS")
    TeleportTab:CreateButton({ Name = "🐉 HYDRA ISLAND", Callback = function() Teleport(CFrame.new(5206.40, 1004.10, 748.35)) end })
    TeleportTab:CreateButton({ Name = "🍰 CAKE ISLAND", Callback = function() Teleport(CFrame.new(-2091.91, 70.00, -12142.83)) end })
    TeleportTab:CreateButton({ Name = "👻 HAUNTED CASTLE", Callback = function() Teleport(CFrame.new(-9516.99, 172.01, 6078.46)) end })
    TeleportTab:CreateButton({ Name = "🏝️ TIKI OUTPOST", Callback = function() Teleport(CFrame.new(-16548.81, 55.60, -172.81)) end })
    TeleportTab:CreateButton({ Name = "🌊 SUBMERGED ISLAND", Callback = function()
        pcall(function()
            Teleport(CFrame.new(-16269.70, 25.22, 1373.65))
            task.wait(2)
            ReplicatedStorage.Modules.Net:FindFirstChild("RF/SubmarineWorkerSpeak"):InvokeServer("TravelToSubmergedIsland")
        end)
    end })
end

-- ========== TAB: SERVER ==========
local ServerTab = Window:CreateTab("🖥️ SERVER")

ServerTab:CreateSection("🔄 SERVER CONTROL")

ServerTab:CreateButton({
    Name = "🔄 HOP SERVER",
    Callback = function()
        pcall(function()
            for i = math.random(1, 75), 100 do
                local servers = ReplicatedStorage.__ServerBrowser:InvokeServer(i)
                for id, data in pairs(servers) do
                    if tonumber(data.Count) < 12 then
                        game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, id)
                        return
                    end
                end
            end
        end)
    end,
})

ServerTab:CreateButton({
    Name = "🔄 REJOIN SERVER",
    Callback = function()
        game:GetService("TeleportService"):Teleport(game.PlaceId, Player)
    end,
})

ServerTab:CreateButton({
    Name = "📋 COPY JOB ID",
    Callback = function()
        setclipboard(game.JobId)
    end,
})

-- ========== TAB: SETTINGS ==========
local SettingsTab = Window:CreateTab("⚙️ SETTINGS")

SettingsTab:CreateSection("🎨 VISUAL")

SettingsTab:CreateToggle({
    Name = "💡 FULL BRIGHT",
    CurrentValue = _G.ReiHub.FullBright,
    Callback = function(v)
        _G.ReiHub.FullBright = v
        SaveConfig()
    end,
})

SettingsTab:CreateSection("🛡️ ANTI AFK")

SettingsTab:CreateToggle({
    Name = "🔴 ANTI AFK",
    CurrentValue = _G.ReiHub.AntiAFK,
    Callback = function(v)
        _G.ReiHub.AntiAFK = v
        SaveConfig()
    end,
})

-- =============================================
-- NOTIFICAÇÃO
-- =============================================
task.spawn(function()
    task.wait(2)
    Rayfield:Notify({
        Title = "🌿 REI HUB V3",
        Content = "Carregado! Mundo: " .. CurrentWorld,
        Duration = 3,
    })
end)

print("✅ REI HUB V3 CARREGADO! Mundo: " .. CurrentWorld)
print("🎯 Tecla para abrir: RightShift")
