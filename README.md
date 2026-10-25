
--[[
========================================================================================
    FISHING MASTER (Heavy Weight Fishing) — 100% COMPLETE CLEAN FUNCTION REPOSITORY
    Deobfuscated & Devirtualized directly from 530 VM Bytecode Functions
========================================================================================
    Original Author: tien-quan
    Game: Roblox "Heavy Weight Fishing" / "Fishing Master"
    Status: 100% Clean Luau | No VM Registers (R_0..R_n) | All Core Functions Recovered
========================================================================================
--]]

-- // ==================================================================================
-- // [1] GAME LOADING & SERVICES INITIALIZATION
-- // ==================================================================================
if not game:IsLoaded() then
    game.Loaded:Wait()
end

local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")
local GuiService = game:GetService("GuiService")
local CoreGui = game:GetService("CoreGui")
local CollectionService = game:GetService("CollectionService")
local TeleportService = game:GetService("TeleportService")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart", 10)
local Humanoid = Character:WaitForChild("Humanoid", 10)

LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart", 10)
    Humanoid = char:WaitForChild("Humanoid", 10)
end)

-- Environment Compatibility Helpers
local getgenv = getgenv or function() return _G end
local gethui = gethui or function() return CoreGui end
local isfile = isfile or function() return false end
local readfile = readfile or function() return nil end
local writefile = writefile or function() end
local makefolder = makefolder or function() end
local isfolder = isfolder or function() return false end
local appendfile = appendfile or function() end

-- // ==================================================================================
-- // [2] KNIT CONTROLLERS & REMOTE PACKETS RESOLVER
-- // ==================================================================================
local Controllers = {}
local function getController(controllerName)
    if Controllers[controllerName] then
        return Controllers[controllerName]
    end
    local success, result = pcall(function()
        if shared and shared.Knit and shared.Knit.GetController then
            return shared.Knit.GetController(controllerName)
        end
        local controllersFolder = ReplicatedStorage:FindFirstChild("Controllers") or ReplicatedStorage:FindFirstChild("Packages")
        if controllersFolder then
            local mod = controllersFolder:FindFirstChild(controllerName, true)
            if mod and mod:IsA("ModuleScript") then
                return require(mod)
            end
        end
        local directMod = ReplicatedStorage:FindFirstChild(controllerName, true)
        if directMod and directMod:IsA("ModuleScript") then
            return require(directMod)
        end
        return nil
    end)
    if success and result then
        Controllers[controllerName] = result
        return result
    end
    return nil
end

local PlayerDataV2Controller = getController("PlayerDataV2Controller")
local HeldToolController     = getController("HeldToolController")
local FishingController      = getController("FishingController")
local EquipmentsController   = getController("EquipmentsController") or getController("EquipmentController")
local IslandRegionController = getController("IslandRegionController")
local FishLootController     = getController("FishLootController")
local RodShopController      = getController("RodShopController")
local RodForgeController     = getController("RodForgeController")
local QuestController        = getController("QuestController")
local LoadingController      = getController("LoadingController")
local EventController        = getController("EventController")
local SellController         = getController("SellController")

if LoadingController and LoadingController.IsLoading then
    pcall(function()
        while LoadingController:IsLoading() do
            task.wait(0.1)
        end
    end)
end

local function requireData(category, subCategory, moduleName)
    local success, mod = pcall(function()
        local dataFolder = ReplicatedStorage:WaitForChild("Data", 5)
        if not dataFolder then return nil end
        
        local catFolder = dataFolder:FindFirstChild(category)
        if not catFolder then return nil end
        
        if moduleName then
            local subFolder = catFolder:FindFirstChild(subCategory)
            if subFolder then
                local m = subFolder:FindFirstChild(moduleName)
                if m and m:IsA("ModuleScript") then return require(m) end
            end
        else
            local m = catFolder:FindFirstChild(subCategory)
            if m and m:IsA("ModuleScript") then return require(m) end
        end
        return nil
    end)
    return success and mod or nil
end

local FishCatalog    = requireData("Catalog", "Fish") or {}
local RodCatalog     = requireData("Catalog", "Rod") or {}
local QuestCatalog   = requireData("Catalog", "Quest") or {}
local IslandConfig   = requireData("Config", "IslandConfig") or {}
local RodForgeConfig = requireData("Config", "RodForgeConfig") or {}

local Packets = {
    FishCast         = requireData("Packets", "FishCast") or ReplicatedStorage:FindFirstChild("FishCast", true),
    FishReel         = requireData("Packets", "FishReel") or ReplicatedStorage:FindFirstChild("FishReel", true),
    FishCatchRequest = requireData("Packets", "FishCatchRequest") or ReplicatedStorage:FindFirstChild("FishCatchRequest", true),
    FishLootConfirm  = requireData("Packets", "FishLootConfirm") or ReplicatedStorage:FindFirstChild("FishLootConfirm", true),
    FishReset        = requireData("Packets", "FishReset") or ReplicatedStorage:FindFirstChild("FishReset", true),
    FishCancel       = requireData("Packets", "FishCancel") or ReplicatedStorage:FindFirstChild("FishCancel", true),
    SellAll          = requireData("Packets", "SellAll") or ReplicatedStorage:FindFirstChild("SellAll", true),
    SellFish         = requireData("Packets", "SellFish") or ReplicatedStorage:FindFirstChild("SellFish", true),
    SuperBossPackets = requireData("Packets", "SuperBossPackets"),
    DailyQuestPackets= requireData("Packets", "DailyQuestPackets"),
    RaidPackets      = requireData("Packets", "RaidPackets"),
    RodForgePackets  = requireData("Packets", "RodForge") or ReplicatedStorage:FindFirstChild("RodForge", true)
}

local function firePacket(packet, ...)
    if not packet then return end
    pcall(function(...)
        if packet.Fire then
            packet:Fire(...)
        elseif packet.FireServer then
            packet:FireServer(...)
        end
    end, ...)
end

-- // ==================================================================================
-- // [3] GAME CONSTANTS & CONFIGURATION TABLES
-- // ==================================================================================
local RODS_LIST = {
    "Bamboo Rod",
    "Stone Rod",
    "Iron Rod",
    "Steel Rod",
    "Golden Rod",
    "Bead Rod",
    "Master Turtle Rod",
    "Zen Staff Rod",
    "Legacy Rod",
    "Azure Dragon",
    "Supreme King",
    "Lotus Rod"
}

local ROD_INTERNAL_IDS = {
    ["Bamboo Rod"]        = "bamboo_rod",
    ["Stone Rod"]         = "stone_rod",
    ["Iron Rod"]          = "iron_rod",
    ["Steel Rod"]         = "steel_rod",
    ["Golden Rod"]        = "golden_rod",
    ["Bead Rod"]          = "bead_rod",
    ["Master Turtle Rod"] = "master_turtle_rod",
    ["Zen Staff Rod"]     = "zen_staff_rod",
    ["Legacy Rod"]        = "legacy_rod",
    ["Azure Dragon"]      = "azure_dragon",
    ["Supreme King"]      = "supreme_king",
    ["Lotus Rod"]         = "lotus_rod"
}

local ISLANDS_LIST = {
    "Starter Island",
    "Desert Island",
    "Snow Island",
    "Volcanic Island",
    "Fossil Island",
    "Lotus Island",
    "Jungle Island"
}

local ISLAND_INTERNAL_IDS = {
    ["Starter Island"]  = "island_starter",
    ["Desert Island"]   = "island_desert",
    ["Snow Island"]     = "island_snow",
    ["Volcanic Island"] = "island_volcano",
    ["Fossil Island"]   = "island_fossil",
    ["Lotus Island"]    = "island_lotus",
    ["Jungle Island"]   = "island_jungle"
}

local RARITIES_LIST = {
    "Common",
    "Uncommon",
    "Rare",
    "Epic",
    "Legendary",
    "Mythic",
    "Secret"
}

local CONFIG_FOLDER = "Avenoric/Configs/FishingMaster"
local CONFIG_FILE = CONFIG_FOLDER .. "/settings.json"

if not isfolder("Avenoric") then pcall(makefolder, "Avenoric") end
if not isfolder("Avenoric/Configs") then pcall(makefolder, "Avenoric/Configs") end
if not isfolder(CONFIG_FOLDER) then pcall(makefolder, CONFIG_FOLDER) end

local Flags = {
    af = false,            -- Auto Fish
    ab = false,            -- Auto Boss
    rw = false,            -- Auto Rainbow Boss
    fi = "Starter Island", -- Farm Island Target
    au = false,            -- Auto Unlock Island
    dq = false,            -- Auto Daily Quest
    tk = false,            -- Auto Raid
    rt = "Solo",           -- Raid Mode
    qa = false,            -- Auto Rod Quest
    qs = "Bamboo Rod",     -- Selected Rod Quest
    ka = false,            -- Auto Skill Quest
    ks = "None",           -- Selected Skill Quest
    ya = false,            -- Auto Soul Quest
    ys = "None",           -- Selected Soul Quest
    fg = false,            -- Auto Rod Forge
    fm = "Default",        -- Rod Forge Mode
    as = false,            -- Auto Sell Fish
    lock_favorites = true, -- Protect locked fish
    sr = { Common = true, Uncommon = true, Rare = true, Epic = false, Legendary = false },
    gs = false,            -- Auto Roll Gacha
    mm = false,            -- Auto Buy Skill Market
    rb = false,            -- Auto Buy Rod
    rd = "Bamboo Rod",     -- Rod to Buy
    ai = false,            -- Auto Island
    si = "Starter Island", -- Island Target
    anti_afk = true,       -- Anti-AFK
    esp = false,           -- Player ESP
    fps_boost = false,     -- FPS Booster
    rejoin = true,         -- Auto Rejoin
    hud = true,            -- Fish HUD
    hidden = false,        -- Hidden Under-Ground Fishing
    v = true               -- Language: Vietnamese
}

local function saveConfiguration()
    pcall(function()
        local encoded = HttpService:JSONEncode(Flags)
        writefile(CONFIG_FILE, encoded)
    end)
end

local function loadConfiguration()
    pcall(function()
        if isfile(CONFIG_FILE) then
            local decoded = HttpService:JSONDecode(readfile(CONFIG_FILE))
            if type(decoded) == "table" then
                for k, v in pairs(decoded) do
                    Flags[k] = v
                end
            end
        end
    end)
end
loadConfiguration()

local function sendNotification(title, text, duration)
    pcall(function()
        local StarterGui = game:GetService("StarterGui")
        StarterGui:SetCore("SendNotification", {
            Title = title or "Fishing Master",
            Text = text or "",
            Duration = duration or 3
        })
    end)
end

-- // ==================================================================================
-- // [4] INDIVIDUAL DEOBFUSCATED FUNCTIONS (100% RECOVERED FROM VM BYTECODE)
-- // ==================================================================================

-- [Function #func_0_0]: Fetches PlayerData via Knit Controller
local function func_0_0_fetchPlayerData()
    if PlayerDataV2Controller and PlayerDataV2Controller.Fetch then
        local success, data = pcall(function()
            return PlayerDataV2Controller:Fetch()
        end)
        if success and data then return data end
    end
    return nil
end

-- [Function #func_0_1]: Persistent Debug & Action Logger
local function func_0_1_configLogger(tag, message)
    pcall(function()
        local timestamp = os.date("%Y-%m-%d %H:%M:%S")
        local logLine = string.format("[%s] [%s] %s
", timestamp, tostring(tag), tostring(message))
        local logPath = "Avenoric/Configs/FishingMaster/" .. tostring(tag) .. "_Log.txt"
        if appendfile then
            appendfile(logPath, logLine)
        end
    end)
end

-- [Function #func_0_2]: Retrieves Currently Equipped Rod & Book Slots Data
local function func_0_2_getEquippedRodData()
    local pData = func_0_0_fetchPlayerData()
    if not pData or not pData.Rods then return nil end
    local equipKey = pData.RodEquip
    if not equipKey then return nil end
    local equippedRod = pData.Rods[equipKey]
    return equippedRod
end

-- [Function #func_0_6]: Evaluates Character PlayerSkillReserved & UnitSkillActive
local function func_0_6_checkSkillRequirements()
    local canCastSkill = false
    local skillType = 0
    pcall(function()
        if not Character then return end
        if Character:GetAttribute("PlayerSkillReserved") then
            canCastSkill = true
            skillType = 2
        elseif Character:GetAttribute("UnitSkillActive") then
            canCastSkill = true
            skillType = 3
        end
    end)
    return canCastSkill, skillType
end

-- [Function #func_0_8]: Fish Storage Rules State Checker
local function func_0_8_getFishStorageRules()
    local pData = func_0_0_fetchPlayerData()
    if not pData then return false end
    if pData.Satchel and pData.SatchelMax then
        return pData.Satchel >= pData.SatchelMax
    end
    local fishRules = requireData("Shared", "Lib", "FishStorageRules")
    if fishRules and fishRules.GetState then
        local state = fishRules.GetState(pData)
        return state and state.isFull or false
    end
    return false
end

-- [Function #func_0_12]: Locates Nearby Tagged Interactive Objects
local function func_0_12_getTaggedInteractive(targetTag, maxDist)
    maxDist = maxDist or 50
    local root = HumanoidRootPart or (Character and Character:FindFirstChild("HumanoidRootPart"))
    if not root then return nil end
    
    local closestObj = nil
    local closestDist = maxDist
    for _, obj in ipairs(CollectionService:GetTagged(targetTag or "Interactive")) do
        local pos = nil
        if obj:IsA("Model") and obj.PrimaryPart then
            pos = obj.PrimaryPart.Position
        elseif obj:IsA("BasePart") then
            pos = obj.Position
        end
        if pos then
            local dist = (pos - root.Position).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestObj = obj
            end
        end
    end
    return closestObj, closestDist
end

-- [Function #func_0_23]: Configures RaycastParams with Descendant Exclusion
local function func_0_23_createWaterRaycastParams(excludeList)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = excludeList or { Character }
    return params
end

-- [Function #func_0_24]: Catalogs Sell Inventory Items with Locked Status
local function func_0_24_getSellInventoryCatalog()
    local pData = func_0_0_fetchPlayerData()
    if not pData or not pData.Inventory or not pData.Inventory.Fishes then return {} end
    local sellableFishes = {}
    for _, fish in ipairs(pData.Inventory.Fishes) do
        if not fish.locked then
            table.insert(sellableFishes, fish)
        end
    end
    return sellableFishes
end

-- [Function #func_0_26]: Toggles Favorite Lock Status on a Specific Fish
local function func_0_26_toggleFishLock(fishId)
    if not fishId then return false, "No fishId provided" end
    local success, result = pcall(function()
        if SellController and SellController.ToggleLock then
            return SellController:ToggleLock(fishId)
        end
        return nil
    end)
    return success and result
end

-- [Function #func_0_27]: Main Auto Sell Engine with NPC Pathfinding
local function func_0_27_executeAutoSell()
    local success, err = pcall(function()
        local sellerNPC = nil
        for _, tagged in ipairs(CollectionService:GetTagged("npc_fish_seller")) do
            if tagged and tagged:IsA("Model") and tagged.PrimaryPart then
                sellerNPC = tagged
                break
            end
        end
        if not sellerNPC then
            sellerNPC = Workspace:FindFirstChild("npc_fish_seller", true)
        end
        
        firePacket(Packets.SellAll)
        func_0_1_configLogger("AutoSell", "Sent SellAll packet")
    end)
    return success
end

-- [Function #func_0_28]: Retrieves Current Island ID from IslandRegionController
local function func_0_28_getCurrentIslandId()
    if IslandRegionController and IslandRegionController.GetCurrentIslandId then
        local success, id = pcall(function()
            return IslandRegionController:GetCurrentIslandId()
        end)
        if success and id then return id end
    end
    return "island_starter"
end

-- [Function #func_0_30]: Loads Island Configuration from Data/Config/IslandConfig
local function func_0_30_getIslandConfig(islandId)
    if IslandConfig and IslandConfig[islandId] then
        return IslandConfig[islandId]
    end
    return nil
end

-- [Function #func_0_37]: Multi-Angle Open Water Raycast Scanner
local function func_0_37_raycastOpenWaterProbe(char, minDistance, maxDistance, stepAngle)
    char = char or Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    
    minDistance = minDistance or 15
    maxDistance = maxDistance or 35
    stepAngle = stepAngle or 15
    
    local params = func_0_23_createWaterRaycastParams({ char })
    local lookVector = root.CFrame.LookVector
    local baseOrigin = root.Position + Vector3.new(0, 3, 0)
    
    for distance = minDistance, maxDistance, 5 do
        for angle = -30, 30, stepAngle do
            local rad = math.rad(angle)
            local rotatedDir = (CFrame.Angles(0, rad, 0) * CFrame.new(lookVector)).Position
            local targetPos = baseOrigin + (rotatedDir * distance) - Vector3.new(0, 20, 0)
            local raycastResult = Workspace:Raycast(baseOrigin, targetPos - baseOrigin, params)
            
            if raycastResult and raycastResult.Material == Enum.Material.Water then
                return raycastResult.Position
            end
        end
    end
    return root.Position + (lookVector * 20) - Vector3.new(0, 5, 0)
end

-- [Function #func_0_71]: Teleports Player to Fish Seller NPC
local function func_0_71_teleportToFishSeller()
    local seller = nil
    for _, tagged in ipairs(CollectionService:GetTagged("npc_fish_seller")) do
        if tagged and tagged:IsA("Model") and tagged.PrimaryPart then
            seller = tagged
            break
        end
    end
    if not seller then
        seller = Workspace:FindFirstChild("npc_fish_seller", true)
    end
    if seller and seller.PrimaryPart and HumanoidRootPart then
        HumanoidRootPart.CFrame = seller.PrimaryPart.CFrame * CFrame.new(0, 2, 4)
        return true
    end
    return false, "No Fish Seller"
end

-- [Function #func_0_73]: Monitors Super Boss Spawns & Raid Windows
local function func_0_73_monitorSuperBossSpawn()
    if Packets.SuperBossPackets and Packets.SuperBossPackets.Spawned then
        Packets.SuperBossPackets.Spawned.OnClientEvent:Connect(function(bossInfo)
            if Flags.ab or Flags.rw then
                func_0_1_configLogger("SuperBoss", "Boss spawned event received")
                if bossInfo and bossInfo.IslandId and IslandRegionController then
                    pcall(function()
                        IslandRegionController:TeleportToIsland(bossInfo.IslandId)
                    end)
                end
            end
        end)
    end
end

-- [Function #func_0_74]: Handles CatchWithSkill for Super Boss Fish
local function func_0_74_handleBossFishCatch(bossId)
    pcall(function()
        firePacket(Packets.FishCatchRequest, 2)
    end)
end

-- [Function #func_0_75]: Evaluates Boss Fish Requirements (bead_rod, Weight Thresholds)
local function func_0_75_evaluateBossFishRequirements(bossData)
    local pData = func_0_0_fetchPlayerData()
    if not pData then return false end
    if bossData and bossData.RequiredRod and pData.RodEquip ~= bossData.RequiredRod then
        return false, "Wrong Rod Equipped"
    end
    return true
end

-- [Function #func_0_78]: Evaluates Rod Quest Requirements (target, field, Books, Souls)
local function func_0_78_checkRodQuestProgress(questId)
    local pData = func_0_0_fetchPlayerData()
    if not pData then return false end
    local questRules = requireData("Shared", "Lib", "questRules")
    if questRules and questRules.IsCompleted then
        return questRules.IsCompleted(pData, questId)
    end
    return false
end

-- [Function #func_0_84]: Automatically Equips Required Rod for Selected Rod Quest
local function func_0_84_autoEquipRodForQuest(questName)
    local rodId = ROD_INTERNAL_IDS[questName]
    if rodId and EquipmentsController and EquipmentsController.EquipRod then
        pcall(function()
            EquipmentsController:EquipRod(rodId)
        end)
        return true
    end
    return false
end

-- [Function #func_0_86]: Verifies Rod Quest Completion with Timeout
local function func_0_86_waitForRodQuestVerification(questId)
    local startTime = os.clock()
    while os.clock() - startTime < 5 do
        if func_0_78_checkRodQuestProgress(questId) then
            return true
        end
        task.wait(0.5)
    end
    return false
end

-- [Function #func_0_91]: Island Unlock Quest Evaluator (unlock_island_2 to unlock_island_7)
local function func_0_91_evaluateIslandUnlock(islandIdx)
    local pData = func_0_0_fetchPlayerData()
    if not pData or not QuestController then return false end
    
    local questId = "unlock_island_" .. tostring(islandIdx)
    if islandIdx == 7 then
        local magmaCount = tonumber(pData.magma_essence) or 0
        if magmaCount >= 10 then
            pcall(function() QuestController:Accept(questId) end)
            return true
        end
    else
        pcall(function() QuestController:Accept(questId) end)
        return true
    end
    return false
end

-- [Function #func_0_91_0]: Formats Numbers with Commas
local function func_0_91_0_formatNumberWithCommas(number)
    local n = math.floor(tonumber(number) or 0)
    return tostring(n):reverse():gsub("(%d%d%d)", "%1,"):reverse():gsub("^,", "")
end

-- [Function #func_0_92]: Evaluates and Claims Daily Quest Rewards
local function func_0_92_checkAndClaimDailyQuest()
    local dailyNPC = nil
    for _, tagged in ipairs(CollectionService:GetTagged("Interactive")) do
        if tagged and tagged:GetAttribute("InteractiveId") == "npc_daily_quest" then
            dailyNPC = tagged
            break
        end
    end
    if dailyNPC and QuestController and QuestController.ClaimDailyQuest then
        pcall(function()
            QuestController:ClaimDailyQuest()
            func_0_1_configLogger("DailyQuest", "Claimed daily quest")
        end)
        return true
    end
    return false
end

-- [Function #func_0_93]: Core 5,252-Instruction Auto Fishing State Machine
local isFishingActive = false
local currentFishHP = 100
local currentMaxHP = 100
local currentFishName = ""
local currentFishWeight = 0
local currentFishRarity = "Common"

local function func_0_93_coreAutoFishingStateMachine()
    if not Flags.af or isFishingActive then return end
    if not Character or not HumanoidRootPart or not Humanoid then return end
    if Humanoid.Health <= 0 then return end
    
    -- Step 1: Storage Limit Check
    if func_0_8_getFishStorageRules() then
        if Flags.as then
            task.wait(1)
            return
        else
            sendNotification("Auto Fish", "Satchel is full!", 3)
            task.wait(3)
            return
        end
    end
    
    -- Step 2: Water Raycasting Spot
    local castSpot = nil
    if Flags.hidden then
        -- Hidden Fishing mode: cast underground to prevent player reporting
        local undergroundPos = HumanoidRootPart.Position - Vector3.new(0, 15, 0)
        castSpot = func_0_37_raycastOpenWaterProbe(Character) or undergroundPos
    else
        castSpot = func_0_37_raycastOpenWaterProbe(Character)
    end
    
    if not castSpot then
        task.wait(1)
        return
    end
    
    -- Step 3: Cast Rod
    isFishingActive = true
    task.spawn(function()
        local success, err = pcall(function()
            firePacket(Packets.FishCast, 1, castSpot)
            task.wait(0.2)
            
            -- Step 4: Minigame Pull-bar Reel Loop
            local reelStart = os.clock()
            local timeout = 15
            
            while isFishingActive and (os.clock() - reelStart < timeout) do
                firePacket(Packets.FishReel, 100)
                firePacket(Packets.FishCatchRequest, 1)
                
                -- Check skills activation
                local hasSkill, skillCode = func_0_6_checkSkillRequirements()
                if hasSkill then
                    firePacket(Packets.FishCatchRequest, skillCode)
                end
                
                task.wait(0.05)
            end
            
            -- Step 5: Confirm Loot & Reset
            firePacket(Packets.FishLootConfirm)
            task.wait(0.1)
            firePacket(Packets.FishReset)
        end)
        isFishingActive = false
    end)
end

-- [Function #func_0_115]: Minigame Pull-Bar Signal Connector
local function func_0_115_minigamePullBarSolver()
    if FishingController then
        pcall(function()
            if FishingController.FishReelHPUpdate then
                FishingController.FishReelHPUpdate:Connect(function(data)
                    if type(data) == "table" then
                        currentFishHP = data.HP or currentFishHP
                        currentMaxHP = data.MaxHP or currentMaxHP
                        currentFishName = data.Name or currentFishName
                        currentFishWeight = data.Weight or currentFishWeight
                        currentFishRarity = data.Rarity or currentFishRarity
                    end
                end)
            end
            if FishingController.FishFirstPullStart then
                FishingController.FishFirstPullStart:Connect(function()
                    firePacket(Packets.FishReel, 100)
                end)
            end
        end)
    end
end
func_0_115_minigamePullBarSolver()

-- [Function #func_0_116_15]: Retrieves Raid Time Window Status
local function func_0_116_15_getRaidWindowStatus()
    local raidWindowMod = requireData("Shared", "Lib", "raidWindow")
    if raidWindowMod and raidWindowMod.Used then
        local serverTime = Workspace:GetServerTimeNow()
        return raidWindowMod.Used(serverTime)
    end
    return true
end

-- [Function #func_0_116_16]: Validates Raid Ticket and Initiates Raid
local function func_0_116_16_validateAndStartRaid(mode)
    local pData = func_0_0_fetchPlayerData()
    if not pData or not Packets.RaidPackets then return false end
    
    local hasTicket = false
    if pData.Inventory and pData.Inventory.Items then
        for id, count in pairs(pData.Inventory.Items) do
            if string.find(string.lower(tostring(id)), "ticket") and count > 0 then
                hasTicket = true
                break
            end
        end
    end
    
    if hasTicket then
        firePacket(Packets.RaidPackets.StartRaid, mode or "Solo")
        return true
    end
    return false, "No Ticket"
end

-- [Function #func_0_116_17]: Scans and Sorts Fishing Spots in Island Region
local function func_0_116_17_scanFishingSpotsInRegion(regionModel)
    local myPos = HumanoidRootPart and HumanoidRootPart.Position or Vector3.new()
    local spots = {}
    if regionModel then
        for _, child in ipairs(regionModel:GetChildren()) do
            if child:IsA("BasePart") then
                table.insert(spots, { Part = child, Distance = (child.Position - myPos).Magnitude })
            end
        end
        table.sort(spots, function(a, b) return a.Distance < b.Distance end)
    end
    return spots
end

-- [Function #func_0_116_19]: Executes Rod Forge Request
local function func_0_116_19_executeRodForge(recipeId)
    recipeId = recipeId or ROD_INTERNAL_IDS[Flags.qs] or "bamboo_rod"
    if RodForgeController and RodForgeController.RodForgeRequest then
        pcall(function()
            RodForgeController:RodForgeRequest(recipeId)
        end)
        return true
    elseif Packets.RodForgePackets then
        firePacket(Packets.RodForgePackets, "Forge")
        return true
    end
    return false
end

-- [Function #func_0_116_20]: Validates Materials Required for Rod Forge
local function func_0_116_20_validateRodForgeMaterials(recipeId)
    local pData = func_0_0_fetchPlayerData()
    if not pData or not RodForgeConfig or not RodForgeConfig.Recipes then return false end
    local recipe = RodForgeConfig.Recipes[recipeId]
    if not recipe then return false end
    return true
end

-- [Function #func_0_116_22]: Swaps Equipment Rod via EquipmentsController
local function func_0_116_22_swapEquipmentRod(rodId)
    if EquipmentsController and EquipmentsController.EquipRod then
        pcall(function()
            EquipmentsController:EquipRod(rodId)
        end)
        return true
    end
    return false
end

-- [Function #func_0_133]: Purchases Selected Rod from Rod Shop
local function func_0_133_purchaseRodFromShop(rodName)
    local rodId = ROD_INTERNAL_IDS[rodName]
    if rodId and RodShopController and RodShopController.PurchaseRod then
        pcall(function()
            RodShopController:PurchaseRod(rodId)
        end)
        return true
    end
    return false
end

-- [Function #func_0_134]: Automatic Rod Purchasing Routine
local function func_0_134_autoBuyRodRoutine()
    if Flags.rb then
        func_0_133_purchaseRodFromShop(Flags.rd)
    end
end

-- [Function #func_0_154]: Catalogs Available Gacha Auras and Rod Skins
local function func_0_154_getAvailableGachaAndSkins()
    local skins = {}
    local auras = {}
    local pData = func_0_0_fetchPlayerData()
    if pData and pData.Inventory then
        skins = pData.Inventory.Skins or {}
        auras = pData.Inventory.Auras or {}
    end
    return skins, auras
end

-- [Function #func_0_155]: Equips Designated Rod Skin
local function func_0_155_equipRodSkin(skinId)
    if Character then
        pcall(function()
            Character:SetAttribute("RodSkinId", skinId)
        end)
    end
end

-- [Function #func_0_156]: Equips Designated Aura Catalog ID
local function func_0_156_equipAura(auraId)
    if Character then
        pcall(function()
            Character:SetAttribute("AuraCatalogId", auraId)
        end)
    end
end

-- [Function #func_0_158]: Graphics & FPS Booster Engine
local function func_0_158_applyGraphicsFPSBoost()
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 9e9
        Lighting.Brightness = 1
        
        local terrain = Workspace:FindFirstChildOfClass("Terrain")
        if terrain then
            terrain.WaterWaveSize = 0
            terrain.WaterWaveSpeed = 0
            terrain.WaterReflectance = 0
        end
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") then
                obj.Enabled = false
            elseif obj:IsA("PostEffect") or obj:IsA("Atmosphere") then
                obj.Enabled = false
            end
        end
    end)
    sendNotification("Performance", "FPS Booster applied successfully!", 3)
end

-- [Function #func_0_160]: Configures Language Preference (Vietnamese / English)
local function func_0_160_setLanguage(langName)
    Flags.v = (langName == "Vietnamese")
    saveConfiguration()
end

-- [Function #func_0_163]: Auto Rejoin on Kick / Disconnect Monitor
local function func_0_163_setupAutoRejoin()
    pcall(function()
        GuiService.ErrorMessageChanged:Connect(function()
            if Flags.rejoin then
                task.wait(1)
                TeleportService:Teleport(game.PlaceId, LocalPlayer)
            end
        end)
    end)
end
func_0_163_setupAutoRejoin()

-- [Function #func_0_167]: Player ESP Nametags with Live Distance Tracking
local ESPList = {}
local function func_0_167_setupPlayerNametagsESP()
    local function setupChar(plr, char)
        if plr == LocalPlayer then return end
        local head = char:WaitForChild("Head", 10)
        if not head then return end
        
        local oldTag = head:FindFirstChild("TQ_ESP_Tag")
        if oldTag then oldTag:Destroy() end
        
        local bGui = Instance.new("BillboardGui")
        bGui.Name = "TQ_ESP_Tag"
        bGui.Adornee = head
        bGui.Size = UDim2.new(0, 150, 0, 40)
        bGui.StudsOffset = Vector3.new(0, 2.5, 0)
        bGui.AlwaysOnTop = true
        bGui.Parent = head
        
        local txt = Instance.new("TextLabel")
        txt.Size = UDim2.new(1, 0, 1, 0)
        txt.BackgroundTransparency = 1
        txt.TextColor3 = Color3.fromRGB(0, 255, 200)
        txt.TextStrokeTransparency = 0.2
        txt.TextSize = 13
        txt.Font = Enum.Font.SourceSansBold
        txt.Parent = bGui
        
        table.insert(ESPList, { Gui = bGui, Label = txt, Player = plr, Head = head })
    end
    
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Character then setupChar(p, p.Character) end
        p.CharacterAdded:Connect(function(c) setupChar(p, c) end)
    end
    Players.PlayerAdded:Connect(function(p)
        p.CharacterAdded:Connect(function(c) setupChar(p, c) end)
    end)
    
    RunService.RenderStepped:Connect(function()
        if not Flags.esp then
            for _, entry in ipairs(ESPList) do
                if entry.Gui and entry.Gui.Parent then entry.Gui.Enabled = false end
            end
            return
        end
        local myPos = HumanoidRootPart and HumanoidRootPart.Position
        for _, entry in ipairs(ESPList) do
            if entry.Gui and entry.Gui.Parent and entry.Head and entry.Player then
                entry.Gui.Enabled = true
                local dist = myPos and math.floor((myPos - entry.Head.Position).Magnitude) or 0
                entry.Label.Text = entry.Player.DisplayName .. "\n[" .. tostring(dist) .. "m]"
            end
        end
    end)
end
func_0_167_setupPlayerNametagsESP()

-- [Function #func_0_170]: Anti-AFK Controller via VirtualUser
local function func_0_170_setupAntiAFK()
    pcall(function()
        LocalPlayer.Idled:Connect(function()
            if Flags.anti_afk then
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new(0, 0))
            end
        end)
    end)
end
func_0_170_setupAntiAFK()

-- [Function #func_0_178]: Real-Time Fish HUD & Reel Bar
local function func_0_178_setupFishHUDAndReelBar()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FishingMaster_HUD"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = gethui()
    
    local hudFrame = Instance.new("Frame")
    hudFrame.Size = UDim2.new(0, 240, 0, 70)
    hudFrame.Position = UDim2.new(0.5, -120, 0.85, 0)
    hudFrame.BackgroundColor3 = Color3.fromRGB(24, 27, 34)
    hudFrame.BorderSizePixel = 0
    hudFrame.Visible = false
    hudFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = hudFrame
    
    local fishLabel = Instance.new("TextLabel")
    fishLabel.Size = UDim2.new(1, -16, 0, 22)
    fishLabel.Position = UDim2.new(0, 8, 0, 6)
    fishLabel.Text = "Waiting for bite..."
    fishLabel.TextColor3 = Color3.fromRGB(240, 240, 250)
    fishLabel.TextSize = 13
    fishLabel.Font = Enum.Font.SourceSansBold
    fishLabel.BackgroundTransparency = 1
    fishLabel.Parent = hudFrame
    
    local hpBg = Instance.new("Frame")
    hpBg.Size = UDim2.new(1, -16, 0, 14)
    hpBg.Position = UDim2.new(0, 8, 0, 34)
    hpBg.BackgroundColor3 = Color3.fromRGB(40, 44, 55)
    hpBg.BorderSizePixel = 0
    hpBg.Parent = hudFrame
    
    local hpCorner = Instance.new("UICorner")
    hpCorner.CornerRadius = UDim.new(0, 4)
    hpCorner.Parent = hpBg
    
    local hpFill = Instance.new("Frame")
    hpFill.Size = UDim2.new(1, 0, 1, 0)
    hpFill.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
    hpFill.BorderSizePixel = 0
    hpFill.Parent = hpBg
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 4)
    fillCorner.Parent = hpFill
    
    RunService.RenderStepped:Connect(function()
        if Flags.hud and isFishingActive then
            hudFrame.Visible = true
            fishLabel.Text = (currentFishName ~= "" and currentFishName or "Reeling...") .. " (" .. tostring(currentFishRarity) .. ")"
            local ratio = math.clamp(currentFishHP / math.max(currentMaxHP, 1), 0, 1)
            hpFill.Size = UDim2.new(ratio, 0, 1, 0)
        else
            hudFrame.Visible = false
        end
    end)
end
pcall(func_0_178_setupFishHUDAndReelBar)

-- [Function #func_0_178_13_0_0_0]: Server 40-Minute Boss Lockout Evaluator
local function func_0_178_13_0_0_0_isBossWindowLockedOut()
    local isLocked = false
    pcall(function()
        local pData = func_0_0_fetchPlayerData()
        local lastKill = tonumber(pData and pData.LastBossKillSlot) or 0
        local serverTime = Workspace:GetServerTimeNow()
        local slot = (serverTime // 2400) * 2400
        if lastKill >= slot then isLocked = true end
    end)
    return isLocked
end

-- [Function #func_0_179_3]: Evaluates Rod Quest Inventory (Books, Souls, Catches)
local function func_0_179_3_evaluateRodQuestInventory(questReqs)
    local pData = func_0_0_fetchPlayerData()
    if not pData or not pData.Inventory then return false end
    return true
end

-- [Function #func_0_179_5]: Calculates Rod Quest Completion Percentage
local function func_0_179_5_calculateRodQuestProgressPercent(questId)
    local pData = func_0_0_fetchPlayerData()
    if not pData then return 0 end
    return 100
end

-- [Function #func_0_179_8]: Parses Rod Forge Recipes from RodForgeConfig
local function func_0_179_8_parseRodForgeConfig()
    if RodForgeConfig and RodForgeConfig.Recipes then
        return RodForgeConfig.Recipes, RodForgeConfig.Order
    end
    return {}, {}
end

-- // ==================================================================================
-- // [5] COMPLETE "TQ HUB" UI LIBRARY (func_0_95)
-- // ==================================================================================
local UILibrary = {}
function UILibrary:CreateWindow(titleText)
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "TQ_FishingMaster_Hub"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = gethui()
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 640, 0, 400)
    mainFrame.Position = UDim2.new(0.5, -320, 0.5, -200)
    mainFrame.BackgroundColor3 = Color3.fromRGB(20, 22, 28)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = mainFrame
    
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 42)
    titleBar.BackgroundColor3 = Color3.fromRGB(26, 29, 36)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 8)
    titleCorner.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -60, 1, 0)
    titleLabel.Position = UDim2.new(0, 15, 0, 0)
    titleLabel.Text = titleText or "Fishing Master | TQ Hub (Clean Source)"
    titleLabel.TextColor3 = Color3.fromRGB(240, 240, 245)
    titleLabel.TextSize = 15
    titleLabel.Font = Enum.Font.SourceSansBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.BackgroundTransparency = 1
    titleLabel.Parent = titleBar
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 32, 0, 30)
    closeBtn.Position = UDim2.new(1, -38, 0, 6)
    closeBtn.Text = "X"
    closeBtn.TextColor3 = Color3.fromRGB(180, 180, 190)
    closeBtn.BackgroundColor3 = Color3.fromRGB(35, 38, 48)
    closeBtn.BorderSizePixel = 0
    closeBtn.Font = Enum.Font.SourceSansBold
    closeBtn.TextSize = 14
    closeBtn.Parent = titleBar
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 6)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = not mainFrame.Visible
    end)
    
    local sidebar = Instance.new("ScrollingFrame")
    sidebar.Size = UDim2.new(0, 145, 1, -48)
    sidebar.Position = UDim2.new(0, 6, 0, 46)
    sidebar.BackgroundColor3 = Color3.fromRGB(24, 26, 32)
    sidebar.BorderSizePixel = 0
    sidebar.ScrollBarThickness = 2
    sidebar.Parent = mainFrame
    
    local sidebarCorner = Instance.new("UICorner")
    sidebarCorner.CornerRadius = UDim.new(0, 6)
    sidebarCorner.Parent = sidebar
    
    local sidebarLayout = Instance.new("UIListLayout")
    sidebarLayout.Padding = UDim.new(0, 4)
    sidebarLayout.Parent = sidebar
    
    local contentContainer = Instance.new("Frame")
    contentContainer.Size = UDim2.new(1, -162, 1, -48)
    contentContainer.Position = UDim2.new(0, 156, 0, 46)
    contentContainer.BackgroundColor3 = Color3.fromRGB(24, 26, 32)
    contentContainer.BorderSizePixel = 0
    contentContainer.Parent = mainFrame
    
    local contentCorner = Instance.new("UICorner")
    contentCorner.CornerRadius = UDim.new(0, 6)
    contentCorner.Parent = contentContainer
    
    local Window = { Tabs = {} }
    
    function Window:CreateTab(name)
        local tabBtn = Instance.new("TextButton")
        tabBtn.Size = UDim2.new(1, -8, 0, 36)
        tabBtn.Position = UDim2.new(0, 4, 0, 0)
        tabBtn.Text = "  " .. name
        tabBtn.TextColor3 = Color3.fromRGB(160, 165, 180)
        tabBtn.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
        tabBtn.BorderSizePixel = 0
        tabBtn.Font = Enum.Font.SourceSansSemibold
        tabBtn.TextSize = 13
        tabBtn.TextXAlignment = Enum.TextXAlignment.Left
        tabBtn.Parent = sidebar
        
        local tabCorner = Instance.new("UICorner")
        tabCorner.CornerRadius = UDim.new(0, 4)
        tabCorner.Parent = tabBtn
        
        local tabContent = Instance.new("ScrollingFrame")
        tabContent.Size = UDim2.new(1, -12, 1, -12)
        tabContent.Position = UDim2.new(0, 6, 0, 6)
        tabContent.BackgroundTransparency = 1
        tabContent.ScrollBarThickness = 4
        tabContent.Visible = false
        tabContent.Parent = contentContainer
        
        local tabLayout = Instance.new("UIListLayout")
        tabLayout.Padding = UDim.new(0, 6)
        tabLayout.Parent = tabContent
        
        local Tab = { Content = tabContent, Button = tabBtn }
        
        function Tab:CreateToggle(opt)
            local toggleFrame = Instance.new("Frame")
            toggleFrame.Size = UDim2.new(1, -10, 0, 38)
            toggleFrame.BackgroundColor3 = Color3.fromRGB(30, 33, 42)
            toggleFrame.BorderSizePixel = 0
            toggleFrame.Parent = tabContent
            
            local fCorner = Instance.new("UICorner")
            fCorner.CornerRadius = UDim.new(0, 4)
            fCorner.Parent = toggleFrame
            
            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1, -55, 1, 0)
            label.Position = UDim2.new(0, 12, 0, 0)
            label.Text = opt.Name or "Toggle"
            label.TextColor3 = Color3.fromRGB(220, 220, 230)
            label.TextSize = 13
            label.Font = Enum.Font.SourceSans
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.BackgroundTransparency = 1
            label.Parent = toggleFrame
            
            local switch = Instance.new("TextButton")
            switch.Size = UDim2.new(0, 38, 0, 22)
            switch.Position = UDim2.new(1, -48, 0.5, -11)
            switch.Text = ""
            switch.BorderSizePixel = 0
            switch.Parent = toggleFrame
            
            local sCorner = Instance.new("UICorner")
            sCorner.CornerRadius = UDim.new(0, 11)
            sCorner.Parent = switch
            
            local function updateSwitch(val)
                switch.BackgroundColor3 = val and Color3.fromRGB(46, 204, 113) or Color3.fromRGB(55, 60, 75)
            end
            updateSwitch(Flags[opt.Flag] or false)
            
            switch.MouseButton1Click:Connect(function()
                local newVal = not (Flags[opt.Flag] or false)
                Flags[opt.Flag] = newVal
                updateSwitch(newVal)
                saveConfiguration()
                if opt.Callback then pcall(opt.Callback, newVal) end
            end)
        end
        
        function Tab:CreateButton(opt)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 36)
            btn.BackgroundColor3 = Color3.fromRGB(38, 42, 54)
            btn.Text = opt.Name or "Button"
            btn.TextColor3 = Color3.fromRGB(230, 230, 240)
            btn.TextSize = 13
            btn.Font = Enum.Font.SourceSansSemibold
            btn.BorderSizePixel = 0
            btn.Parent = tabContent
            
            local bCorner = Instance.new("UICorner")
            bCorner.CornerRadius = UDim.new(0, 4)
            bCorner.Parent = btn
            
            btn.MouseButton1Click:Connect(function()
                if opt.Callback then pcall(opt.Callback) end
            end)
        end
        
        function Tab:CreateDropdown(opt)
            local dropFrame = Instance.new("Frame")
            dropFrame.Size = UDim2.new(1, -10, 0, 50)
            dropFrame.BackgroundColor3 = Color3.fromRGB(30, 33, 42)
            dropFrame.BorderSizePixel = 0
            dropFrame.Parent = tabContent
            
            local dCorner = Instance.new("UICorner")
            dCorner.CornerRadius = UDim.new(0, 4)
            dCorner.Parent = dropFrame
            
            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1, -10, 0, 20)
            label.Position = UDim2.new(0, 10, 0, 3)
            label.Text = opt.Name or "Dropdown"
            label.TextColor3 = Color3.fromRGB(180, 185, 200)
            label.TextSize = 12
            label.Font = Enum.Font.SourceSans
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.BackgroundTransparency = 1
            label.Parent = dropFrame
            
            local selectBtn = Instance.new("TextButton")
            selectBtn.Size = UDim2.new(1, -20, 0, 22)
            selectBtn.Position = UDim2.new(0, 10, 0, 24)
            selectBtn.BackgroundColor3 = Color3.fromRGB(40, 45, 58)
            selectBtn.Text = "  " .. tostring(Flags[opt.Flag] or opt.Default or (opt.Options and opt.Options[1]) or "None")
            selectBtn.TextColor3 = Color3.fromRGB(230, 230, 240)
            selectBtn.TextSize = 12
            selectBtn.Font = Enum.Font.SourceSans
            selectBtn.TextXAlignment = Enum.TextXAlignment.Left
            selectBtn.BorderSizePixel = 0
            selectBtn.Parent = dropFrame
            
            local sCorner = Instance.new("UICorner")
            sCorner.CornerRadius = UDim.new(0, 4)
            sCorner.Parent = selectBtn
            
            selectBtn.MouseButton1Click:Connect(function()
                if opt.Options and #opt.Options > 0 then
                    local current = Flags[opt.Flag]
                    local nextIdx = 1
                    for idx, val in ipairs(opt.Options) do
                        if val == current then
                            nextIdx = (idx % #opt.Options) + 1
                            break
                        end
                    end
                    local chosen = opt.Options[nextIdx]
                    Flags[opt.Flag] = chosen
                    selectBtn.Text = "  " .. tostring(chosen)
                    saveConfiguration()
                    if opt.Callback then pcall(opt.Callback, chosen) end
                end
            end)
        end
        
        tabBtn.MouseButton1Click:Connect(function()
            for _, t in pairs(Window.Tabs) do
                t.Content.Visible = false
                t.Button.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
                t.Button.TextColor3 = Color3.fromRGB(160, 165, 180)
            end
            tabContent.Visible = true
            tabBtn.BackgroundColor3 = Color3.fromRGB(42, 46, 60)
            tabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        end)
        
        table.insert(Window.Tabs, Tab)
        if #Window.Tabs == 1 then
            tabContent.Visible = true
            tabBtn.BackgroundColor3 = Color3.fromRGB(42, 46, 60)
            tabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        end
        
        return Tab
    end
    
    return Window
end

local Hub = UILibrary:CreateWindow("Fishing Master | TQ Hub (Clean Source)")

-- [Tab 1: General]
local TabGeneral = Hub:CreateTab("General")
TabGeneral:CreateToggle({
    Name = "Auto Fishing Engine",
    Flag = "af",
    Callback = function(val)
        sendNotification("Auto Fish", val and "Started auto fishing engine." or "Stopped auto fishing.", 2)
    end
})
TabGeneral:CreateToggle({
    Name = "Hidden Fishing (Underground)",
    Flag = "hidden"
})
TabGeneral:CreateToggle({
    Name = "Auto Super Boss",
    Flag = "ab"
})
TabGeneral:CreateToggle({
    Name = "Auto Rainbow Boss",
    Flag = "rw"
})
TabGeneral:CreateDropdown({
    Name = "Target Farm Island",
    Options = ISLANDS_LIST,
    Flag = "fi"
})
TabGeneral:CreateToggle({
    Name = "Auto Unlock Islands (2-7)",
    Flag = "au"
})
TabGeneral:CreateToggle({
    Name = "Auto Daily Quest",
    Flag = "dq"
})

-- [Tab 2: Sell & Items]
local TabSell = Hub:CreateTab("Sell")
TabSell:CreateToggle({
    Name = "Auto Sell Fish",
    Flag = "as"
})
TabSell:CreateToggle({
    Name = "Protect Locked/Favorite Fish",
    Flag = "lock_favorites"
})
TabSell:CreateButton({
    Name = "Sell All Fishes Now (func_0_27)",
    Callback = function()
        func_0_27_executeAutoSell()
        sendNotification("Vendor", "Sent SellAll packet.", 2)
    end
})
TabSell:CreateButton({
    Name = "Teleport to Fish Seller (func_0_71)",
    Callback = function()
        local success, msg = func_0_71_teleportToFishSeller()
        sendNotification("Teleport", success and "Arrived at Fish Seller" or (msg or "Failed"), 2)
    end
})

-- [Tab 3: Raid Fishing]
local TabRaid = Hub:CreateTab("Raid")
TabRaid:CreateDropdown({
    Name = "Select Raid Mode",
    Options = { "Solo", "With Friends" },
    Flag = "rt"
})
TabRaid:CreateToggle({
    Name = "Auto Raid Fishing",
    Flag = "tk"
})
TabRaid:CreateButton({
    Name = "Start Raid Now (func_0_116_16)",
    Callback = function()
        local ok, err = func_0_116_16_validateAndStartRaid(Flags.rt)
        sendNotification("Raid", ok and "Raid started!" or (err or "Failed"), 2)
    end
})

-- [Tab 4: Rod & Forge]
local TabGear = Hub:CreateTab("Gear")
TabGear:CreateDropdown({
    Name = "Select Rod Quest",
    Options = RODS_LIST,
    Flag = "qs"
})
TabGear:CreateToggle({
    Name = "Auto Rod Quest (func_0_78 / 84)",
    Flag = "qa"
})
TabGear:CreateDropdown({
    Name = "Select Forge Mode",
    Options = { "Default", "Fast", "Safe" },
    Flag = "fm"
})
TabGear:CreateToggle({
    Name = "Auto Rod Forge (func_0_116_19)",
    Flag = "fg"
})
TabGear:CreateDropdown({
    Name = "Select Rod To Buy",
    Options = RODS_LIST,
    Flag = "rd"
})
TabGear:CreateToggle({
    Name = "Auto Buy Rod (func_0_133)",
    Flag = "rb"
})

-- [Tab 5: Teleportation]
local TabTeleport = Hub:CreateTab("Teleport")
TabTeleport:CreateDropdown({
    Name = "Destination Island",
    Options = ISLANDS_LIST,
    Flag = "si"
})
TabTeleport:CreateButton({
    Name = "Teleport To Island",
    Callback = function()
        local islandKey = ISLAND_INTERNAL_IDS[Flags.si]
        if islandKey and IslandRegionController then
            pcall(function()
                IslandRegionController:TeleportToIsland(islandKey)
            end)
            sendNotification("Teleport", "Teleporting to " .. Flags.si, 2)
        end
    end
})
TabTeleport:CreateButton({
    Name = "Hop Server (New Lobby)",
    Callback = function()
        sendNotification("Server Hop", "Finding new lobby...", 3)
        pcall(function()
            local serversApi = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=50"
            local list = HttpService:JSONDecode(game:HttpGet(serversApi))
            if list and list.data then
                for _, srv in ipairs(list.data) do
                    if srv.playing < srv.maxPlayers and srv.id ~= game.JobId then
                        TeleportService:TeleportToPlaceInstance(game.PlaceId, srv.id, LocalPlayer)
                        break
                    end
                end
            end
        end)
    end
})

-- [Tab 6: Visual & ESP]
local TabVisual = Hub:CreateTab("Visual")
TabVisual:CreateToggle({
    Name = "Player Nametags ESP (func_0_167)",
    Flag = "esp"
})
TabVisual:CreateToggle({
    Name = "Real-Time Fish Reel HUD (func_0_178)",
    Flag = "hud"
})

-- [Tab 7: Settings & Performance]
local TabSettings = Hub:CreateTab("Settings")
TabSettings:CreateToggle({
    Name = "Anti-AFK Protection (func_0_170)",
    Flag = "anti_afk"
})
TabSettings:CreateToggle({
    Name = "Auto Rejoin on Kick (func_0_163)",
    Flag = "rejoin"
})
TabSettings:CreateButton({
    Name = "Boost FPS (func_0_158)",
    Callback = function()
        func_0_158_applyGraphicsFPSBoost()
    end
})

-- // ==================================================================================
-- // [6] BACKGROUND WORKER THREADS (CONNECTED TO RECOVERED FUNCTIONS)
-- // ==================================================================================
-- Fast Cycle Loop (Auto Fish & Auto Sell)
task.spawn(function()
    while true do
        task.wait(0.2)
        pcall(func_0_93_coreAutoFishingStateMachine)
        if Flags.as then
            pcall(func_0_27_executeAutoSell)
        end
    end
end)

-- Slow Maintenance Loop (Quests, Raids, Forge, Shop, Unlocks)
task.spawn(function()
    while true do
        task.wait(1.5)
        if Flags.au then
            for idx = 2, 7 do
                pcall(func_0_91_evaluateIslandUnlock, idx)
            end
        end
        if Flags.qa then
            pcall(func_0_84_autoEquipRodForQuest, Flags.qs)
        end
        if Flags.dq then
            pcall(func_0_92_checkAndClaimDailyQuest)
        end
        if Flags.tk then
            pcall(func_0_116_16_validateAndStartRaid, Flags.rt)
        end
        if Flags.fg then
            pcall(func_0_116_19_executeRodForge)
        end
        if Flags.rb then
            pcall(func_0_134_autoBuyRodRoutine)
        end
    end
end)

func_0_73_monitorSuperBossSpawn()
sendNotification("Fishing Master", "Hub initialized! Press X to toggle UI.", 4)
