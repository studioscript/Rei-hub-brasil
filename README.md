-- ========================================
-- REI HUB | BLOX FRUITS
-- Script simplificado com funções principais
-- ========================================

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- ========================================
-- CONFIGURAÇÕES INICIAIS
-- ========================================
_G.ReiHub = {
    -- Auto Farm
    AutoFarm = false,
    FarmMode = "Level", -- Level, Boss, Material
    SelectWeapon = "Melee",
    
    -- Auto Boss
    AutoBoss = false,
    SelectBoss = "",
    
    -- Auto Stats
    AutoStats = false,
    StatType = "Melee", -- Melee, Defense, Sword, Gun, Devil
    
    -- Auto Haki
    AutoHaki = false,
    
    -- Auto Ken (Observation)
    AutoKen = false,
    
    -- Outros
    AutoCollectChest = false,
    AutoFarmMaterial = false,
    SelectMaterial = "",
    
    -- Bring Mobs
    BringRange = 235,
    MobHeight = 20,
    
    -- Velocidade
    SpeedEnabled = false,
    SpeedValue = 50,
    
    -- No Clip
    NoClip = false,
}

-- ========================================
-- SALVAR CONFIGURAÇÕES
-- ========================================
local FolderName = "Rei Hub"
local FileName = "Settings.json"
local FullPath = FolderName .. "/" .. FileName

if makefolder and not isfolder(FolderName) then makefolder(FolderName) end

function SaveSettings()
    if not writefile then return end
    pcall(function()
        writefile(FullPath, game:GetService("HttpService"):JSONEncode(_G.ReiHub))
    end)
end

function LoadSettings()
    if isfile and isfile(FullPath) then
        pcall(function()
            local data = game:GetService("HttpService"):JSONDecode(readfile(FullPath))
            for k, v in pairs(data) do _G.ReiHub[k] = v end
        end)
    end
end
LoadSettings()

-- ========================================
-- FUNÇÕES AUXILIARES
-- ========================================
local function GetHRP()
    local char = Player.Character
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function EquipWeapon(weaponName)
    local char = Player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local tool = Player.Backpack:FindFirstChild(weaponName) or char:FindFirstChild(weaponName)
    if tool and tool.Parent ~= char then
        hum:EquipTool(tool)
    end
end

local function UseSkill(skill)
    VirtualInputManager:SendKeyEvent(true, skill, false, game)
    task.wait(0.05)
    VirtualInputManager:SendKeyEvent(false, skill, false, game)
end

local function TP(pos)
    local hrp = GetHRP()
    if hrp then
        hrp.CFrame = pos
    end
end

local function IsAlive(model)
    local hum = model and model:FindFirstChild("Humanoid")
    return hum and hum.Health > 0
end

-- ========================================
-- AUTO KEN (OBSERVATION HAKI)
-- ========================================
local function HasKen()
    local char = Player.Character
    return char and char:FindFirstChild("HasKen")
end

task.spawn(function()
    while task.wait(0.3) do
        if _G.ReiHub.AutoKen and not HasKen() then
            pcall(function()
                ReplicatedStorage.Remotes.CommE:FireServer("Ken", true)
            end)
        end
    end
end)

-- ========================================
-- AUTO HAKI (BUSO)
-- ========================================
task.spawn(function()
    while task.wait(1) do
        if _G.ReiHub.AutoHaki then
            pcall(function()
                if not Player.Character:FindFirstChild("HasBuso") then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
                end
            end)
        end
    end
end)

-- ========================================
-- AUTO STATS
-- ========================================
task.spawn(function()
    while task.wait(0.5) do
        if _G.ReiHub.AutoStats and Player.Data.Points.Value > 0 then
            pcall(function()
                local statMap = {
                    Melee = "Melee",
                    Defense = "Defense", 
                    Sword = "Sword",
                    Gun = "Gun",
                    Devil = "Demon Fruit"
                }
                ReplicatedStorage.Remotes.CommF_:InvokeServer("AddPoint", statMap[_G.ReiHub.StatType], 1)
            end)
        end
    end
end)

-- ========================================
-- AUTO BRING MOBS
-- ========================================
local BringPart = Instance.new("Part", Workspace)
BringPart.Name = "ReiHub_Bring"
BringPart.Size = Vector3.new(1, 1, 1)
BringPart.Anchored = true
BringPart.CanCollide = false
BringPart.Transparency = 1

local function BringEnemy(targetPos)
    if not _G.ReiHub.AutoFarm then return end
    local hrp = GetHRP()
    if not hrp then return end
    
    for _, mob in pairs(Workspace.Enemies:GetChildren()) do
        local hum = mob:FindFirstChild("Humanoid")
        local root = mob:FindFirstChild("HumanoidRootPart")
        if hum and root and hum.Health > 0 then
            local dist = (root.Position - targetPos).Magnitude
            if dist <= _G.ReiHub.BringRange then
                local tween = TweenService:Create(root, TweenInfo.new(0.35), {CFrame = CFrame.new(targetPos)})
                tween:Play()
            end
        end
    end
end

-- ========================================
-- FUNÇÃO DE ATAQUE PRINCIPAL
-- ========================================
local function KillMob(mob)
    if not mob or not IsAlive(mob) then return end
    local root = mob:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    -- Trava posição
    if not mob:GetAttribute("Locked") then
        mob:SetAttribute("Locked", root.CFrame)
    end
    
    -- Bring
    BringEnemy((mob:GetAttribute("Locked")).Position)
    
    -- Equipa arma
    EquipWeapon(_G.ReiHub.SelectWeapon)
    
    -- Teleporta acima do mob
    TP(root.CFrame * CFrame.new(0, _G.ReiHub.MobHeight, 0))
    
    -- Ataca
    local tool = Player.Character and Player.Character:FindFirstChildOfClass("Tool")
    if tool then
        VirtualUser:CaptureController()
        VirtualUser:Button1Down(Vector2.new(1280, 672))
        task.wait(0.1)
        VirtualUser:Button1Up(Vector2.new(1280, 672))
    end
end

-- ========================================
-- LEVEL FARM (AUTO QUEST)
-- ========================================
local Quests = {
    -- World 1
    ["Bandit"] = { Quest = "BanditQuest1", Level = 1, Qty = 1, Pos = CFrame.new(1045.96, 27.00, 1560.82), MobPos = CFrame.new(1120, 27, 1590) },
    ["Monkey"] = { Quest = "JungleQuest", Level = 10, Qty = 1, Pos = CFrame.new(-1598.08, 35.55, 153.37), MobPos = CFrame.new(-1448.51, 67.85, 11.46) },
    ["Gorilla"] = { Quest = "JungleQuest", Level = 15, Qty = 2, Pos = CFrame.new(-1598.08, 35.55, 153.37), MobPos = CFrame.new(-1129.88, 40.46, -525.42) },
    ["Pirate"] = { Quest = "BuggyQuest1", Level = 30, Qty = 1, Pos = CFrame.new(-1141.07, 4.10, 3831.54), MobPos = CFrame.new(-1103.51, 13.75, 3896.09) },
    ["Brute"] = { Quest = "BuggyQuest1", Level = 40, Qty = 2, Pos = CFrame.new(-1141.07, 4.10, 3831.54), MobPos = CFrame.new(-1140.08, 14.80, 4322.92) },
    ["Desert Bandit"] = { Quest = "DesertQuest", Level = 60, Qty = 1, Pos = CFrame.new(894.48, 5.14, 4392.43), MobPos = CFrame.new(924.79, 6.44, 4481.58) },
    ["Desert Officer"] = { Quest = "DesertQuest", Level = 75, Qty = 2, Pos = CFrame.new(894.48, 5.14, 4392.43), MobPos = CFrame.new(1608.28, 8.61, 4371.00) },
    ["Snow Bandit"] = { Quest = "SnowQuest", Level = 90, Qty = 1, Pos = CFrame.new(1389.74, 88.15, -1298.90), MobPos = CFrame.new(1354.34, 87.27, -1393.94) },
    ["Snowman"] = { Quest = "SnowQuest", Level = 100, Qty = 2, Pos = CFrame.new(1389.74, 88.15, -1298.90), MobPos = CFrame.new(1201.64, 144.57, -1550.06) },
    ["Chief Petty Officer"] = { Quest = "MarineQuest2", Level = 120, Qty = 1, Pos = CFrame.new(-5039.58, 27.35, 4324.68), MobPos = CFrame.new(-4881.23, 22.65, 4273.75) },
    ["Sky Bandit"] = { Quest = "SkyQuest", Level = 150, Qty = 1, Pos = CFrame.new(-4839.53, 716.36, -2619.44), MobPos = CFrame.new(-4953.20, 295.74, -2899.22) },
    ["Dark Master"] = { Quest = "SkyQuest", Level = 175, Qty = 2, Pos = CFrame.new(-4839.53, 716.36, -2619.44), MobPos = CFrame.new(-5259.84, 391.39, -2229.03) },
    ["Prisoner"] = { Quest = "PrisonerQuest", Level = 190, Qty = 1, Pos = CFrame.new(5308.93, 1.65, 475.12), MobPos = CFrame.new(5098.97, -0.32, 474.23) },
    ["Dangerous Prisoner"] = { Quest = "PrisonerQuest", Level = 210, Qty = 2, Pos = CFrame.new(5308.93, 1.65, 475.12), MobPos = CFrame.new(5654.56, 15.63, 866.29) },
    ["Toga Warrior"] = { Quest = "ColosseumQuest", Level = 250, Qty = 1, Pos = CFrame.new(-1580.04, 6.35, -2986.47), MobPos = CFrame.new(-1820.21, 51.68, -2740.66) },
    ["Gladiator"] = { Quest = "ColosseumQuest", Level = 275, Qty = 2, Pos = CFrame.new(-1580.04, 6.35, -2986.47), MobPos = CFrame.new(-1292.83, 56.38, -3339.03) },
    ["Military Soldier"] = { Quest = "MagmaQuest", Level = 300, Qty = 1, Pos = CFrame.new(-5313.37, 10.95, 8515.29), MobPos = CFrame.new(-5411.16, 11.08, 8454.29) },
    ["Military Spy"] = { Quest = "MagmaQuest", Level = 325, Qty = 2, Pos = CFrame.new(-5313.37, 10.95, 8515.29), MobPos = CFrame.new(-5802.86, 86.26, 8828.85) },
    ["Fishman Warrior"] = { Quest = "FishmanQuest", Level = 375, Qty = 1, Pos = CFrame.new(61122.65, 18.49, 1569.39), MobPos = CFrame.new(60878.30, 18.48, 1543.75) },
    ["Fishman Commando"] = { Quest = "FishmanQuest", Level = 400, Qty = 2, Pos = CFrame.new(61122.65, 18.49, 1569.39), MobPos = CFrame.new(61922.63, 18.48, 1493.93) },
    ["God's Guard"] = { Quest = "SkyExp1Quest", Level = 450, Qty = 1, Pos = CFrame.new(-4721.88, 843.87, -1949.96), MobPos = CFrame.new(-4710.04, 845.27, -1927.30) },
    ["Shanda"] = { Quest = "SkyExp1Quest", Level = 475, Qty = 2, Pos = CFrame.new(-7859.09, 5544.19, -381.47), MobPos = CFrame.new(-7678.48, 5566.40, -497.21) },
    ["Royal Squad"] = { Quest = "SkyExp2Quest", Level = 525, Qty = 1, Pos = CFrame.new(-7906.81, 5634.66, -1411.99), MobPos = CFrame.new(-7624.25, 5658.13, -1467.35) },
    ["Royal Soldier"] = { Quest = "SkyExp2Quest", Level = 550, Qty = 2, Pos = CFrame.new(-7906.81, 5634.66, -1411.99), MobPos = CFrame.new(-7836.75, 5645.66, -1790.62) },
    ["Galley Pirate"] = { Quest = "FountainQuest", Level = 625, Qty = 1, Pos = CFrame.new(5259.81, 37.35, 4050.02), MobPos = CFrame.new(5551.02, 78.90, 3930.41) },
    ["Galley Captain"] = { Quest = "FountainQuest", Level = 650, Qty = 2, Pos = CFrame.new(5259.81, 37.35, 4050.02), MobPos = CFrame.new(5441.95, 42.50, 4950.09) },
}

local function GetQuestByLevel()
    local level = Player.Data.Level.Value
    local lastQuest = nil
    for _, quest in pairs(Quests) do
        if level >= quest.Level then
            lastQuest = quest
        end
    end
    return lastQuest
end

local CurrentMob = nil

task.spawn(function()
    while task.wait(0.3) do
        if not _G.ReiHub.AutoFarm or _G.ReiHub.FarmMode ~= "Level" then
            task.wait(1)
            continue
        end
        
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            
            local questData = GetQuestByLevel()
            if not questData then return end
            
            local questUI = Player.PlayerGui.Main.Quest
            local hasQuest = questUI and questUI.Visible
            
            -- Pega quest se não tiver
            if not hasQuest then
                TP(questData.Pos)
                if (hrp.Position - questData.Pos.Position).Magnitude <= 30 then
                    task.wait(1)
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", questData.Quest, questData.Qty)
                end
                return
            end
            
            -- Procura mob
            local closestMob = nil
            local closestDist = math.huge
            
            for _, mob in pairs(Workspace.Enemies:GetChildren()) do
                if IsAlive(mob) and mob.Name == questData.Name then
                    local root = mob:FindFirstChild("HumanoidRootPart")
                    if root then
                        local dist = (hrp.Position - root.Position).Magnitude
                        if dist < closestDist then
                            closestDist = dist
                            closestMob = mob
                        end
                    end
                end
            end
            
            if closestMob then
                KillMob(closestMob)
            else
                TP(questData.MobPos)
            end
        end)
    end
end)

-- ========================================
-- AUTO BOSS
-- ========================================
local Bosses = {
    -- World 1
    ["The Gorilla King"] = { Quest = "JungleQuest", Qty = 3, Pos = CFrame.new(-1088.75, 8.13, -488.55), QuestPos = CFrame.new(-1601.65, 36.85, 153.38) },
    ["Bobby"] = { Quest = "BuggyQuest1", Qty = 3, Pos = CFrame.new(-1087.37, 46.94, 4040.14), QuestPos = CFrame.new(-1140.17, 4.75, 3827.40) },
    ["The Saw"] = { Pos = CFrame.new(-784.89, 72.42, 1603.58) },
    ["Yeti"] = { Quest = "SnowQuest", Qty = 3, Pos = CFrame.new(1218.79, 138.01, -1488.02), QuestPos = CFrame.new(1386.80, 87.27, -1298.35) },
    ["Vice Admiral"] = { Quest = "MarineQuest2", Qty = 2, Pos = CFrame.new(-5006.54, 88.03, 4353.16), QuestPos = CFrame.new(-5036.24, 28.67, 4324.56) },
    ["Saber Expert"] = { Pos = CFrame.new(-1458.89, 29.88, -50.63) },
    ["Magma Admiral"] = { Quest = "MagmaQuest", Qty = 3, Pos = CFrame.new(-5765.89, 82.92, 8718.30), QuestPos = CFrame.new(-5314.62, 12.26, 8517.27) },
    ["Fishman Lord"] = { Quest = "FishmanQuest", Qty = 3, Pos = CFrame.new(61260.15, 30.95, 1193.43), QuestPos = CFrame.new(61122.65, 18.49, 1569.39) },
    ["Wysper"] = { Quest = "SkyExp1Quest", Qty = 3, Pos = CFrame.new(-7866.13, 5576.43, -546.74), QuestPos = CFrame.new(-7861.94, 5545.51, -379.85) },
    ["Thunder God"] = { Quest = "SkyExp2Quest", Qty = 3, Pos = CFrame.new(-7994.98, 5761.02, -2088.64), QuestPos = CFrame.new(-7903.38, 5635.98, -1410.92) },
    ["Cyborg"] = { Quest = "FountainQuest", Qty = 3, Pos = CFrame.new(6094.02, 73.77, 3825.73), QuestPos = CFrame.new(5258.27, 38.52, 4050.04) },
}

task.spawn(function()
    while task.wait(0.5) do
        if not _G.ReiHub.AutoBoss or not _G.ReiHub.SelectBoss then
            task.wait(1)
            continue
        end
        
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            
            local bossData = Bosses[_G.ReiHub.SelectBoss]
            if not bossData then return end
            
            local boss = Workspace.Enemies:FindFirstChild(_G.ReiHub.SelectBoss) or Workspace:FindFirstChild(_G.ReiHub.SelectBoss)
            
            -- Se tem boss vivo, mata
            if boss and IsAlive(boss) then
                local root = boss:FindFirstChild("HumanoidRootPart")
                if root then
                    TP(root.CFrame * CFrame.new(0, 22, 0))
                    KillMob(boss)
                end
                return
            end
            
            -- Se precisa de quest
            if bossData.Quest and _G.ReiHub.AutoAcceptQuest then
                local hasQuest = Player.PlayerGui.Main.Quest and Player.PlayerGui.Main.Quest.Visible
                if not hasQuest then
                    TP(bossData.QuestPos)
                    if (hrp.Position - bossData.QuestPos.Position).Magnitude <= 30 then
                        task.wait(1)
                        ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", bossData.Quest, bossData.Qty)
                    end
                    return
                end
            end
            
            -- Vai pra posição do boss
            TP(bossData.Pos)
        end)
    end
end)

-- ========================================
-- AUTO COLLECT CHEST
-- ========================================
task.spawn(function()
    while task.wait(0.5) do
        if not _G.ReiHub.AutoCollectChest then
            task.wait(1)
            continue
        end
        
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            
            local collection = game:GetService("CollectionService")
            local chests = collection:GetTagged("_ChestTagged")
            
            local closest = nil
            local closestDist = math.huge
            
            for _, chest in pairs(chests) do
                if not chest:GetAttribute("IsDisabled") then
                    local pos = chest:GetPivot().Position
                    local dist = (hrp.Position - pos).Magnitude
                    if dist < closestDist then
                        closestDist = dist
                        closest = chest
                    end
                end
            end
            
            if closest then
                TP(closest:GetPivot())
            end
        end)
    end
end)

-- ========================================
-- SPEED HACK
-- ========================================
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

-- ========================================
-- NO CLIP
-- ========================================
task.spawn(function()
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
end)

-- ========================================
-- AUTO TEAM (MARINES)
-- ========================================
task.spawn(function()
    task.wait(2)
    if Player.Team and Player.Team.Name ~= "Marines" then
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", "Marines")
        end)
    end
end)

-- ========================================
-- FULL BRIGHT
-- ========================================
Lighting.Ambient = Color3.new(0.7, 0.7, 0.7)
Lighting.Brightness = 2
Lighting.FogEnd = 1e10

-- ========================================
-- UI - REI HUB
-- ========================================
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/Jadelly/Ui/refs/heads/main/NewZynLib"))()
local Window = Library:MakeWindow({
    Title = "Rei Hub | Farm",
    SubTitle = "Blox Fruits",
    SaveFolder = true,
})

-- Tab Principal
local MainTab = Window:MakeTab({ Title = "Main", Icon = "rbxassetid://10709769508" })

MainTab:AddSection({"Auto Farm"})

MainTab:AddDropdown({
    Name = "Select Weapon",
    Options = {"Melee", "Sword", "Blox Fruit", "Gun"},
    Default = _G.ReiHub.SelectWeapon,
    Callback = function(v)
        _G.ReiHub.SelectWeapon = v
        SaveSettings()
    end
})

MainTab:AddDropdown({
    Name = "Farm Mode",
    Options = {"Level", "Boss"},
    Default = _G.ReiHub.FarmMode,
    Callback = function(v)
        _G.ReiHub.FarmMode = v
        SaveSettings()
    end
})

MainTab:AddToggle({
    Name = "Start Auto Farm",
    Default = _G.ReiHub.AutoFarm,
    Callback = function(v)
        _G.ReiHub.AutoFarm = v
        SaveSettings()
    end
})

MainTab:AddToggle({
    Name = "Auto Accept Quests",
    Default = _G.ReiHub.AutoAcceptQuest or false,
    Callback = function(v)
        _G.ReiHub.AutoAcceptQuest = v
        SaveSettings()
    end
})

MainTab:AddSection({"Auto Boss"})

local BossList = {}
for name in pairs(Bosses) do
    table.insert(BossList, name)
end

MainTab:AddDropdown({
    Name = "Select Boss",
    Options = BossList,
    Default = _G.ReiHub.SelectBoss or BossList[1],
    Callback = function(v)
        _G.ReiHub.SelectBoss = v
        SaveSettings()
    end
})

MainTab:AddToggle({
    Name = "Start Auto Boss",
    Default = _G.ReiHub.AutoBoss,
    Callback = function(v)
        _G.ReiHub.AutoBoss = v
        SaveSettings()
    end
})

MainTab:AddSection({"Auto Stats"})

MainTab:AddDropdown({
    Name = "Stat Type",
    Options = {"Melee", "Defense", "Sword", "Gun", "Devil"},
    Default = _G.ReiHub.StatType,
    Callback = function(v)
        _G.ReiHub.StatType = v
        SaveSettings()
    end
})

MainTab:AddToggle({
    Name = "Auto Stats",
    Default = _G.ReiHub.AutoStats,
    Callback = function(v)
        _G.ReiHub.AutoStats = v
        SaveSettings()
    end
})

-- Tab Settings
local SettingsTab = Window:MakeTab({ Title = "Settings", Icon = "rbxassetid://10734950309" })

SettingsTab:AddSection({"Haki / Abilities"})

SettingsTab:AddToggle({
    Name = "Auto Ken (Observation)",
    Default = _G.ReiHub.AutoKen,
    Callback = function(v)
        _G.ReiHub.AutoKen = v
        SaveSettings()
    end
})

SettingsTab:AddToggle({
    Name = "Auto Haki (Buso)",
    Default = _G.ReiHub.AutoHaki,
    Callback = function(v)
        _G.ReiHub.AutoHaki = v
        SaveSettings()
    end
})

SettingsTab:AddSection({"Movement"})

SettingsTab:AddToggle({
    Name = "Speed Hack",
    Default = _G.ReiHub.SpeedEnabled,
    Callback = function(v)
        _G.ReiHub.SpeedEnabled = v
        SaveSettings()
    end
})

SettingsTab:AddTextBox({
    Name = "Speed Value",
    Placeholder = "50",
    Default = tostring(_G.ReiHub.SpeedValue),
    Callback = function(v)
        local num = tonumber(v)
        if num then
            _G.ReiHub.SpeedValue = num
            SaveSettings()
        end
    end
})

SettingsTab:AddToggle({
    Name = "No Clip",
    Default = _G.ReiHub.NoClip,
    Callback = function(v)
        _G.ReiHub.NoClip = v
        SaveSettings()
    end
})

SettingsTab:AddSection({"Auto Collect"})

SettingsTab:AddToggle({
    Name = "Auto Collect Chest",
    Default = _G.ReiHub.AutoCollectChest,
    Callback = function(v)
        _G.ReiHub.AutoCollectChest = v
        SaveSettings()
    end
})

SettingsTab:AddSection({"Bring Settings"})

SettingsTab:AddTextBox({
    Name = "Bring Range",
    Placeholder = "235",
    Default = tostring(_G.ReiHub.BringRange),
    Callback = function(v)
        local num = tonumber(v)
        if num then _G.ReiHub.BringRange = num end
        SaveSettings()
    end
})

SettingsTab:AddTextBox({
    Name = "Mob Height",
    Placeholder = "20",
    Default = tostring(_G.ReiHub.MobHeight),
    Callback = function(v)
        local num = tonumber(v)
        if num then _G.ReiHub.MobHeight = num end
        SaveSettings()
    end
})

-- Tab Teleports
local TeleportTab = Window:MakeTab({ Title = "Teleports", Icon = "rbxassetid://10734906975" })

TeleportTab:AddSection({"Worlds"})

TeleportTab:AddButton({
    Name = "Teleport to Sea 1",
    Callback = function()
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelMain") end)
    end
})

TeleportTab:AddButton({
    Name = "Teleport to Sea 2",
    Callback = function()
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelDressrosa") end)
    end
})

TeleportTab:AddButton({
    Name = "Teleport to Sea 3",
    Callback = function()
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelZou") end)
    end
})

TeleportTab:AddSection({"Islands"})

TeleportTab:AddButton({
    Name = "TP to Hydra",
    Callback = function()
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("requestEntrance", Vector3.new(5643.45, 1013.08, -340.51)) end)
        TP(CFrame.new(5206.40, 1004.10, 748.35))
    end
})

TeleportTab:AddButton({
    Name = "TP to Cake Island",
    Callback = function()
        TP(CFrame.new(-2091.91, 70.00, -12142.83))
    end
})

TeleportTab:AddButton({
    Name = "TP to Haunted Castle",
    Callback = function()
        TP(CFrame.new(-9516.99, 172.01, 6078.46))
    end
})

-- Tab Server
local ServerTab = Window:MakeTab({ Title = "Server", Icon = "rbxassetid://7040410130" })

ServerTab:AddButton({
    Name = "Hop Server",
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
    end
})

ServerTab:AddButton({
    Name = "Rejoin Server",
    Callback = function()
        game:GetService("TeleportService"):Teleport(game.PlaceId, Player)
    end
})

ServerTab:AddButton({
    Name = "Copy Job ID",
    Callback = function()
        setclipboard(game.JobId)
    end
})

print("✅ Rei Hub carregado com sucesso!")
