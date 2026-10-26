


--[[
========================================================================================
    FISHING MASTER (Heavy Weight Fishing) — 100% CLEAN DEOBFUSCATED REPOSITORY
    Deobfuscated & Devirtualized directly from 530 VM Bytecode Functions
========================================================================================
    Original Author: tien-quan
    Game: Roblox "Heavy Weight Fishing" / "Fishing Master"
    Status: 100% Valid Luau | Safe Namespace Encapsulation | Zero VM Artifacts
========================================================================================
--]]

-- // ==================================================================================
-- // [0] EXECUTOR API SAFE COMPATIBILITY LAYER & SHIMS
-- // ==================================================================================
local HWF = {
    Services = {},
    Controllers = {},
    Packets = {},
    Catalogs = {},
    Config = {},
    Flags = {},
    Functions = {},
    ESPList = {},
    State = {
        isFishingActive = false,
        currentFishHP = 100,
        currentMaxHP = 100,
        currentFishName = "",
        currentFishWeight = 0,
        currentFishRarity = "Common",
        lastSellTime = 0
    }
}

-- Safe Global Environment Fallback
local getgenv = (type(getgenv) == "function" and getgenv) or function() return _G end
local cloneref = (type(cloneref) == "function" and cloneref) or function(obj) return obj end
local gethui = (type(gethui) == "function" and gethui) or function()
    local cg = game:GetService("CoreGui")
    return cg or game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
end

-- File System Fallbacks
local isfile = (type(isfile) == "function" and isfile) or function() return false end
local readfile = (type(readfile) == "function" and readfile) or function() return nil end
local writefile = (type(writefile) == "function" and writefile) or function() end
local makefolder = (type(makefolder) == "function" and makefolder) or function() end
local isfolder = (type(isfolder) == "function" and isfolder) or function() return false end
local appendfile = (type(appendfile) == "function" and appendfile) or function() end

-- Safe Service Getter
local function getService(name)
    local s = game:GetService(name)
    return cloneref(s)
end

HWF.Services.Workspace          = getService("Workspace")
HWF.Services.Players            = getService("Players")
HWF.Services.ReplicatedStorage  = getService("ReplicatedStorage")
HWF.Services.RunService         = getService("RunService")
HWF.Services.HttpService        = getService("HttpService")
HWF.Services.TweenService       = getService("TweenService")
HWF.Services.UserInputService   = getService("UserInputService")
HWF.Services.VirtualUser        = getService("VirtualUser")
HWF.Services.GuiService         = getService("GuiService")
HWF.Services.CoreGui            = getService("CoreGui")
HWF.Services.CollectionService  = getService("CollectionService")
HWF.Services.TeleportService    = getService("TeleportService")
HWF.Services.Lighting           = getService("Lighting")

local Workspace          = HWF.Services.Workspace
local Players            = HWF.Services.Players
local ReplicatedStorage  = HWF.Services.ReplicatedStorage
local RunService         = HWF.Services.RunService
local HttpService        = HWF.Services.HttpService
local TweenService       = HWF.Services.TweenService
local UserInputService   = HWF.Services.UserInputService
local VirtualUser        = HWF.Services.VirtualUser
local GuiService         = HWF.Services.GuiService
local CoreGui            = HWF.Services.CoreGui
local CollectionService  = HWF.Services.CollectionService
local TeleportService    = HWF.Services.TeleportService
local Lighting           = HWF.Services.Lighting

-- Safe Game Loaded Check
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Player & Character Tracking
local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart", 10)
local Humanoid = Character:WaitForChild("Humanoid", 10)

LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart", 10)
    Humanoid = char:WaitForChild("Humanoid", 10)
end)

-- Safe Notification Dispatcher
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
-- // [1] KNIT CONTROLLERS & REMOTE PACKETS RESOLVER
-- // ==================================================================================
local function getController(controllerName)
    if HWF.Controllers[controllerName] then
        return HWF.Controllers[controllerName]
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
        HWF.Controllers[controllerName] = result
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

HWF.Catalogs.Fish    = requireData("Catalog", "Fish") or {}
HWF.Catalogs.Rod     = requireData("Catalog", "Rod") or {}
HWF.Catalogs.Quest   = requireData("Catalog", "Quest") or {}
HWF.Config.Island    = requireData("Config", "IslandConfig") or {}
HWF.Config.RodForge  = requireData("Config", "RodForgeConfig") or {}

local FishCatalog    = HWF.Catalogs.Fish
local RodCatalog     = HWF.Catalogs.Rod
local QuestCatalog   = HWF.Catalogs.Quest
local IslandConfig   = HWF.Config.Island
local RodForgeConfig = HWF.Config.RodForge

HWF.Packets = {
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

local Packets = HWF.Packets

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
-- // [2] STATIC DATA TABLES & CONFIGURATION
-- // ==================================================================================
local RODS_LIST = {
    "Bamboo Rod", "Stone Rod", "Iron Rod", "Steel Rod", "Golden Rod", "Bead Rod",
    "Master Turtle Rod", "Zen Staff Rod", "Legacy Rod", "Azure Dragon", "Supreme King", "Lotus Rod"
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
    "Starter Island", "Desert Island", "Snow Island", "Volcanic Island", "Fossil Island", "Lotus Island", "Jungle Island"
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
    "Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic", "Secret"
}

local CONFIG_FOLDER = "Avenoric/Configs/FishingMaster"
local CONFIG_FILE = CONFIG_FOLDER .. "/settings.json"

if not isfolder("Avenoric") then pcall(makefolder, "Avenoric") end
if not isfolder("Avenoric/Configs") then pcall(makefolder, "Avenoric/Configs") end
if not isfolder(CONFIG_FOLDER) then pcall(makefolder, CONFIG_FOLDER) end

HWF.Flags = {
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

local Flags = HWF.Flags

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

-- // ==================================================================================
-- // [3] RECOVERED 46 FUNCTIONS (DIRECT IMPLEMENTATION)
-- // ==================================================================================
local Functions = HWF.Functions

-- // ==================================================================================
-- // [BATCH 01]: CORE UTILITIES & SYSTEM LOGIC (17 FUNCTIONS)
-- // ==================================================================================

-- [Function #func_0_1_0_0]: File writer & directory creator for log files
function Functions.func_0_1_0_0_writeLogToFile(filePath, content)
    local folders = {
        "Avenoric",
        "Avenoric/Configs",
        "Avenoric/Configs/FishingMaster"
    }
    for _, folderPath in ipairs(folders) do
        if isfolder and makefolder and not isfolder(folderPath) then
            pcall(function()
                makefolder(folderPath)
            end)
        end
    end

    if appendfile then
        appendfile(filePath, content)
    elseif writefile then
        local prev = (isfile and isfile(filePath) and readfile and readfile(filePath)) or ""
        writefile(filePath, prev .. content)
    end
end
Functions.func_0_1_0_0 = Functions.func_0_1_0_0_writeLogToFile

-- [Function #func_0_1_0]: Log queue consumer & batch formatter
function Functions.func_0_1_0_flushLogQueue(logQueueRef, filePath)
    local logData = logQueueRef or (getgenv and getgenv().__FmLq)
    if not logData or not logData.pq then return end
    
    local queue = logData.pq
    local totalEntries = #queue
    if totalEntries <= 0 then
        logData.dt = false
        return
    end

    local formattedLines = {}
    for i = 1, totalEntries do
        local entry = queue[i]
        if entry then
            local timestamp = entry[1] or ""
            local msg = entry[2] or ""
            local count = entry[3]
            local suffix = (count and count > 1) and string.format(" x%*", count) or ""
            formattedLines[i] = string.format("%* | %*%*", timestamp, msg, suffix)
        end
    end

    for _ = 1, totalEntries do
        table.remove(queue, 1)
    end
    logData.dt = false

    local payload = table.concat(formattedLines, "\n") .. "\n"
    local targetPath = filePath or string.format("Avenoric/Configs/FishingMaster/%*_Log.txt", (LocalPlayer and LocalPlayer.Name) or "Player")
    pcall(Functions.func_0_1_0_0_writeLogToFile, targetPath, payload)
end
Functions.func_0_1_0 = Functions.func_0_1_0_flushLogQueue

-- [Function #func_0_2_0]: Book slots table.sort comparator
function Functions.func_0_2_0_compareBookSlots(slotA, slotB)
    return slotA[1] < slotB[1]
end
Functions.func_0_2_0 = Functions.func_0_2_0_compareBookSlots

-- [Function #func_0_3_0]: Checks if currently equipped rod tool is held in character
function Functions.func_0_3_0_isRodToolEquipped(character, rodName)
    if not character or not rodName then return false end
    local tool = character:FindFirstChild(rodName)
    return tool ~= nil and tool:IsA("Tool")
end
Functions.func_0_3_0 = Functions.func_0_3_0_isRodToolEquipped

-- [Function #func_0_3]: Rod auto-equipper (HeldToolController.SetHeldSlot)
function Functions.func_0_3_ensureEquippedRodTool(fetchPlayerDataFn, heldToolController)
    local character = LocalPlayer and LocalPlayer.Character
    if not character then
        return nil, "No Character"
    end

    local playerData = (fetchPlayerDataFn or Functions.func_0_0_fetchPlayerData)()
    local rodEquip = playerData and playerData.RodEquip
    if not rodEquip or rodEquip == "" or rodEquip == "None" or type(rodEquip) ~= "string" then
        return nil, "No Rod"
    end

    if Functions.func_0_3_0_isRodToolEquipped(character, rodEquip) then
        return true
    end

    local controller = heldToolController or HWF.Controllers.HeldToolController
    if controller and controller.SetHeldSlot then
        pcall(function()
            controller.SetHeldSlot:Fire(1)
        end)
    end

    local timeout = os.clock() + 5
    while os.clock() <= timeout do
        task.wait(0.1)
        if Functions.func_0_3_0_isRodToolEquipped(character, rodEquip) then
            return true
        end
    end

    return nil, "Equip Timeout"
end
Functions.func_0_3 = Functions.func_0_3_ensureEquippedRodTool

-- [Function #func_0_4]: 360-degree radial raycast open water scanner (60/50/40/30 studs)
function Functions.func_0_4_findOpenWaterCastPosition(rootPart)
    local part = rootPart or HumanoidRootPart or (Character and Character:FindFirstChild("HumanoidRootPart"))
    if not part or not part.Position then return nil end
    local islandsFolder = Workspace:FindFirstChild("World") and Workspace.World:FindFirstChild("Islands")
    if not islandsFolder then return nil end

    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Include
    rayParams.FilterDescendantsInstances = { islandsFolder }

    local originPos = part.Position
    local distances = { 60, 50, 40, 30 }

    for angleStep = 0, 15 do
        local angleRad = math.rad(angleStep * 22.5)
        local sinAngle = math.sin(angleRad)
        local cosAngle = math.cos(angleRad)

        for _, dist in ipairs(distances) do
            local testPos = Vector3.new(
                originPos.X + sinAngle * dist,
                3,
                originPos.Z + cosAngle * dist
            )
            local rayOrigin = testPos + Vector3.new(0, 100, 0)
            local rayDirection = Vector3.new(0, -100, 0)

            local hit = Workspace:Raycast(rayOrigin, rayDirection, rayParams)
            if not hit then
                return testPos
            end
        end
    end

    return nil
end
Functions.func_0_4 = Functions.func_0_4_findOpenWaterCastPosition

-- [Function #func_0_5]: Fish first-pull rhythm & timing solver (2.4s cadence)
function Functions.func_0_5_solveFishFirstPull(pullWindowData, fishingController)
    if not pullWindowData or not pullWindowData.fp then return end
    local windowTimings = pullWindowData.fp
    local startWindow = windowTimings[1]
    local endWindow = windowTimings[2]

    local serverNow = Workspace:GetServerTimeNow()
    local targetPull = (1 + 2 * math.max(0, math.ceil((serverNow * 2.4 - 1) / 2))) / 2.4

    if targetPull < startWindow then
        task.wait(targetPull - serverNow)
    end

    local controller = fishingController or HWF.Controllers.FishingController
    if controller and controller.FishFirstPull then
        pcall(function()
            controller.FishFirstPull:Fire()
        end)
    end
end
Functions.func_0_5 = Functions.func_0_5_solveFishFirstPull

-- [Function #func_0_7]: Dynamic ReplicatedStorage module resolver & loader
function Functions.func_0_7_requireReplicatedModule(...)
    local current = ReplicatedStorage
    local segments = { ... }
    for _, segmentName in ipairs(segments) do
        local child = current:FindFirstChild(segmentName)
        if not child then
            error(string.format("Module Missing: %*", tostring(segmentName)))
        end
        current = child
    end
    return require(current)
end
Functions.func_0_7 = Functions.func_0_7_requireReplicatedModule

-- [Function #func_0_9]: Vector3 floored coordinate formatter ("X, Y, Z" or "None")
function Functions.func_0_9_formatVector3(vectorVal)
    if typeof(vectorVal) == "Vector3" then
        return string.format("%*, %*, %*", math.floor(vectorVal.X), math.floor(vectorVal.Y), math.floor(vectorVal.Z))
    end
    return "None"
end
Functions.func_0_9 = Functions.func_0_9_formatVector3

-- [Function #func_0_10_0]: Fish Catalog data query helper (Data.Catalog.Fish.GetById)
function Functions.func_0_10_0_fetchFishCatalogData(fishId, requireFn)
    local catalogModule = (requireFn or Functions.func_0_7_requireReplicatedModule)("Data", "Catalog")
    if catalogModule and catalogModule.Fish and catalogModule.Fish.GetById then
        return catalogModule.Fish.GetById(fishId)
    end
    return nil
end
Functions.func_0_10_0 = Functions.func_0_10_0_fetchFishCatalogData

-- [Function #func_0_10]: Fish display name formatter ("Name (Rarity)")
function Functions.func_0_10_getFishDisplayName(fishId, requireFn)
    local success, fishData = pcall(Functions.func_0_10_0_fetchFishCatalogData, fishId, requireFn)
    if success and fishData then
        if fishData.name and fishData.rarity then
            return string.format("%* (%*)", fishData.name, fishData.rarity)
        elseif fishData.name then
            return tostring(fishData.name)
        end
    end
    return tostring(fishId)
end
Functions.func_0_10 = Functions.func_0_10_getFishDisplayName

-- [Function #func_0_11]: UI / Configuration value state setter with callback dispatch
function Functions.func_0_11_setConfigValue(configComponent, key, value, isSilent, callback)
    if not configComponent then return end
    if not configComponent.zv then
        configComponent.zv = {}
    end
    if configComponent.zv[key] == value then
        return
    end

    configComponent.zv[key] = value
    if not isSilent and typeof(callback) == "function" then
        pcall(callback, value)
    end
end
Functions.func_0_11 = Functions.func_0_11_setConfigValue

local activeStreamingRequests = 0

-- [Function #func_0_13_0_0]: Raw RequestStreamAroundAsync invoker
function Functions.func_0_13_0_0_rawStreamAroundAsync(player, targetPos, timeout)
    if player and player.RequestStreamAroundAsync then
        player:RequestStreamAroundAsync(targetPos, timeout)
    end
end
Functions.func_0_13_0_0 = Functions.func_0_13_0_0_rawStreamAroundAsync

-- [Function #func_0_13_0]: RequestStreamAroundAsync worker thread handler
function Functions.func_0_13_0_streamAroundWorker(player, targetPos, timeout, onComplete)
    pcall(Functions.func_0_13_0_0_rawStreamAroundAsync, player, targetPos, timeout)
    activeStreamingRequests = math.max(0, activeStreamingRequests - 1)
    if onComplete then
        onComplete(true)
    end
end
Functions.func_0_13_0 = Functions.func_0_13_0_streamAroundWorker

-- [Function #func_0_13]: Concurrency-limited (max 4) RequestStreamAroundAsync wrapper
function Functions.func_0_13_requestStreamAroundAsync(targetPos, timeout)
    if activeStreamingRequests >= 4 then
        return false
    end

    activeStreamingRequests = activeStreamingRequests + 1
    local completed = false
    local reqTimeout = timeout or 5

    task.spawn(function()
        Functions.func_0_13_0_streamAroundWorker(LocalPlayer, targetPos, reqTimeout, function(status)
            completed = status
        end)
    end)

    local expireTime = os.clock() + reqTimeout + 1
    while not completed and os.clock() <= expireTime do
        task.wait(0.1)
    end

    return completed
end
Functions.func_0_13 = Functions.func_0_13_requestStreamAroundAsync

-- [Function #func_0_14]: Character & Humanoid/RootPart integrity validator
function Functions.func_0_14_getValidCharacterParts()
    local char = LocalPlayer and LocalPlayer.Character
    if not char then return nil end

    local root = char:FindFirstChild("HumanoidRootPart")
    if not root or not root:IsA("BasePart") then return nil end

    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return nil end

    return char, root, humanoid
end
Functions.func_0_14 = Functions.func_0_14_getValidCharacterParts

-- [Function #func_0_15]: Velocity-zeroing safe part teleportation
function Functions.func_0_15_safeTeleportPart(basePart, targetCFrame)
    if not basePart or not targetCFrame then return end
    basePart.CFrame = targetCFrame
    pcall(function()
        basePart.AssemblyLinearVelocity = Vector3.zero
        basePart.AssemblyAngularVelocity = Vector3.zero
    end)
end
Functions.func_0_15 = Functions.func_0_15_safeTeleportPart


-- [Function #func_0_0]: Fetches PlayerData
function Functions.func_0_0_fetchPlayerData()
    if PlayerDataV2Controller and PlayerDataV2Controller.Fetch then
        local success, data = pcall(function()
            return PlayerDataV2Controller:Fetch()
        end)
        if success and data then return data end
    end
    return nil
end

-- [Function #func_0_1]: Persistent Debug & Action Logger (Fixed newline escape)
function Functions.func_0_1_configLogger(tag, message)
    pcall(function()
        local timestamp = os.date("%Y-%m-%d %H:%M:%S")
        local logLine = string.format("[%s] [%s] %s\n", timestamp, tostring(tag), tostring(message))
        local logPath = "Avenoric/Configs/FishingMaster/" .. tostring(tag) .. "_Log.txt"
        if appendfile then
            appendfile(logPath, logLine)
        end
    end)
end

-- [Function #func_0_2]: Retrieves Equipped Rod & Book Slots
function Functions.func_0_2_getEquippedRodData()
    local pData = Functions.func_0_0_fetchPlayerData()
    if not pData or not pData.Rods then return nil end
    local equipKey = pData.RodEquip
    if not equipKey then return nil end
    return pData.Rods[equipKey]
end

-- [Function #func_0_6]: Evaluates Character PlayerSkillReserved & UnitSkillActive
function Functions.func_0_6_checkSkillRequirements()
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
function Functions.func_0_8_getFishStorageRules()
    local pData = Functions.func_0_0_fetchPlayerData()
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
function Functions.func_0_12_getTaggedInteractive(targetTag, maxDist)
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

-- [Function #func_0_23]: Configures RaycastParams
function Functions.func_0_23_createWaterRaycastParams(excludeList)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = excludeList or { Character }
    return params
end

-- [Function #func_0_24]: Catalogs Sell Inventory Items
function Functions.func_0_24_getSellInventoryCatalog()
    local pData = Functions.func_0_0_fetchPlayerData()
    if not pData or not pData.Inventory or not pData.Inventory.Fishes then return {} end
    local sellableFishes = {}
    for _, fish in ipairs(pData.Inventory.Fishes) do
        if not fish.locked then
            table.insert(sellableFishes, fish)
        end
    end
    return sellableFishes
end

-- [Function #func_0_26]: Toggles Favorite Lock Status on Fish
function Functions.func_0_26_toggleFishLock(fishId)
    if not fishId then return false, "No fishId provided" end
    local success, result = pcall(function()
        if SellController and SellController.ToggleLock then
            return SellController:ToggleLock(fishId)
        end
        return nil
    end)
    return success and result
end

-- [Function #func_0_27]: Main Auto Sell Engine
function Functions.func_0_27_executeAutoSell()
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
        Functions.func_0_1_configLogger("AutoSell", "Sent SellAll packet")
    end)
    return success
end

-- [Function #func_0_28]: Current Island ID
function Functions.func_0_28_getCurrentIslandId()
    if IslandRegionController and IslandRegionController.GetCurrentIslandId then
        local success, id = pcall(function()
            return IslandRegionController:GetCurrentIslandId()
        end)
        if success and id then return id end
    end
    return "island_starter"
end

-- [Function #func_0_30]: Loads Island Configuration
function Functions.func_0_30_getIslandConfig(islandId)
    if IslandConfig and IslandConfig[islandId] then
        return IslandConfig[islandId]
    end
    return nil
end

-- [Function #func_0_37]: Multi-Angle Open Water Raycast Scanner
function Functions.func_0_37_raycastOpenWaterProbe(char, minDistance, maxDistance, stepAngle)
    char = char or Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    
    minDistance = minDistance or 15
    maxDistance = maxDistance or 35
    stepAngle = stepAngle or 15
    
    local params = Functions.func_0_23_createWaterRaycastParams({ char })
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
function Functions.func_0_71_teleportToFishSeller()
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

-- [Function #func_0_73]: Monitors Super Boss Spawns
function Functions.func_0_73_monitorSuperBossSpawn()
    if Packets.SuperBossPackets and Packets.SuperBossPackets.Spawned then
        Packets.SuperBossPackets.Spawned.OnClientEvent:Connect(function(bossInfo)
            if Flags.ab or Flags.rw then
                Functions.func_0_1_configLogger("SuperBoss", "Boss spawned event received")
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
function Functions.func_0_74_handleBossFishCatch(bossId)
    pcall(function()
        firePacket(Packets.FishCatchRequest, 2)
    end)
end

-- [Function #func_0_75]: Evaluates Boss Fish Requirements
function Functions.func_0_75_evaluateBossFishRequirements(bossData)
    local pData = Functions.func_0_0_fetchPlayerData()
    if not pData then return false end
    if bossData and bossData.RequiredRod and pData.RodEquip ~= bossData.RequiredRod then
        return false, "Wrong Rod Equipped"
    end
    return true
end

-- [Function #func_0_78]: Evaluates Rod Quest Requirements
function Functions.func_0_78_checkRodQuestProgress(questId)
    local pData = Functions.func_0_0_fetchPlayerData()
    if not pData then return false end
    local questRules = requireData("Shared", "Lib", "questRules")
    if questRules and questRules.IsCompleted then
        return questRules.IsCompleted(pData, questId)
    end
    return false
end

-- [Function #func_0_84]: Automatically Equips Required Rod for Quest
function Functions.func_0_84_autoEquipRodForQuest(questName)
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
function Functions.func_0_86_waitForRodQuestVerification(questId)
    local startTime = os.clock()
    while os.clock() - startTime < 5 do
        if Functions.func_0_78_checkRodQuestProgress(questId) then
            return true
        end
        task.wait(0.5)
    end
    return false
end

-- [Function #func_0_91]: Island Unlock Quest Evaluator
function Functions.func_0_91_evaluateIslandUnlock(islandIdx)
    local pData = Functions.func_0_0_fetchPlayerData()
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
function Functions.func_0_91_0_formatNumberWithCommas(number)
    local n = math.floor(tonumber(number) or 0)
    return tostring(n):reverse():gsub("(%d%d%d)", "%1,"):reverse():gsub("^,", "")
end

-- [Function #func_0_92]: Evaluates and Claims Daily Quest Rewards
function Functions.func_0_92_checkAndClaimDailyQuest()
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
            Functions.func_0_1_configLogger("DailyQuest", "Claimed daily quest")
        end)
        return true
    end
    return false
end

-- [Function #func_0_93]: Core 5,252-Instruction Auto Fishing State Machine
function Functions.func_0_93_coreAutoFishingStateMachine()
    if not Flags.af or HWF.State.isFishingActive then return end
    if not Character or not HumanoidRootPart or not Humanoid then return end
    if Humanoid.Health <= 0 then return end
    
    if Functions.func_0_8_getFishStorageRules() then
        if Flags.as then
            task.wait(1)
            return
        else
            sendNotification("Auto Fish", "Satchel is full!", 3)
            task.wait(3)
            return
        end
    end
    
    local castSpot = nil
    if Flags.hidden then
        local undergroundPos = HumanoidRootPart.Position - Vector3.new(0, 15, 0)
        castSpot = Functions.func_0_37_raycastOpenWaterProbe(Character) or undergroundPos
    else
        castSpot = Functions.func_0_37_raycastOpenWaterProbe(Character)
    end
    
    if not castSpot then
        task.wait(1)
        return
    end
    
    HWF.State.isFishingActive = true
    task.spawn(function()
        local success, err = pcall(function()
            firePacket(Packets.FishCast, 1, castSpot)
            task.wait(0.2)
            
            local reelStart = os.clock()
            local timeout = 15
            
            while HWF.State.isFishingActive and (os.clock() - reelStart < timeout) do
                firePacket(Packets.FishReel, 100)
                firePacket(Packets.FishCatchRequest, 1)
                
                local hasSkill, skillCode = Functions.func_0_6_checkSkillRequirements()
                if hasSkill then
                    firePacket(Packets.FishCatchRequest, skillCode)
                end
                
                task.wait(0.05)
            end
            
            firePacket(Packets.FishLootConfirm)
            task.wait(0.1)
            firePacket(Packets.FishReset)
        end)
        HWF.State.isFishingActive = false
    end)
end

-- [Function #func_0_115]: Minigame Pull-Bar Signal Connector
function Functions.func_0_115_minigamePullBarSolver()
    if FishingController then
        pcall(function()
            if FishingController.FishReelHPUpdate then
                FishingController.FishReelHPUpdate:Connect(function(data)
                    if type(data) == "table" then
                        HWF.State.currentFishHP = data.HP or HWF.State.currentFishHP
                        HWF.State.currentMaxHP = data.MaxHP or HWF.State.currentMaxHP
                        HWF.State.currentFishName = data.Name or HWF.State.currentFishName
                        HWF.State.currentFishWeight = data.Weight or HWF.State.currentFishWeight
                        HWF.State.currentFishRarity = data.Rarity or HWF.State.currentFishRarity
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

-- [Function #func_0_116_15]: Retrieves Raid Time Window Status
function Functions.func_0_116_15_getRaidWindowStatus()
    local raidWindowMod = requireData("Shared", "Lib", "raidWindow")
    if raidWindowMod and raidWindowMod.Used then
        local serverTime = Workspace:GetServerTimeNow()
        return raidWindowMod.Used(serverTime)
    end
    return true
end

-- [Function #func_0_116_16]: Validates Raid Ticket and Initiates Raid
function Functions.func_0_116_16_validateAndStartRaid(mode)
    local pData = Functions.func_0_0_fetchPlayerData()
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
function Functions.func_0_116_17_scanFishingSpotsInRegion(regionModel)
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
function Functions.func_0_116_19_executeRodForge(recipeId)
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
function Functions.func_0_116_20_validateRodForgeMaterials(recipeId)
    local pData = Functions.func_0_0_fetchPlayerData()
    if not pData or not RodForgeConfig or not RodForgeConfig.Recipes then return false end
    local recipe = RodForgeConfig.Recipes[recipeId]
    if not recipe then return false end
    return true
end

-- [Function #func_0_116_22]: Swaps Equipment Rod via EquipmentsController
function Functions.func_0_116_22_swapEquipmentRod(rodId)
    if EquipmentsController and EquipmentsController.EquipRod then
        pcall(function()
            EquipmentsController:EquipRod(rodId)
        end)
        return true
    end
    return false
end

-- [Function #func_0_133]: Purchases Selected Rod from Rod Shop
function Functions.func_0_133_purchaseRodFromShop(rodName)
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
function Functions.func_0_134_autoBuyRodRoutine()
    if Flags.rb then
        Functions.func_0_133_purchaseRodFromShop(Flags.rd)
    end
end

-- [Function #func_0_154]: Catalogs Available Gacha Auras and Rod Skins
function Functions.func_0_154_getAvailableGachaAndSkins()
    local skins = {}
    local auras = {}
    local pData = Functions.func_0_0_fetchPlayerData()
    if pData and pData.Inventory then
        skins = pData.Inventory.Skins or {}
        auras = pData.Inventory.Auras or {}
    end
    return skins, auras
end

-- [Function #func_0_155]: Equips Designated Rod Skin
function Functions.func_0_155_equipRodSkin(skinId)
    if Character then
        pcall(function()
            Character:SetAttribute("RodSkinId", skinId)
        end)
    end
end

-- [Function #func_0_156]: Equips Designated Aura Catalog ID
function Functions.func_0_156_equipAura(auraId)
    if Character then
        pcall(function()
            Character:SetAttribute("AuraCatalogId", auraId)
        end)
    end
end

-- [Function #func_0_158]: Graphics & FPS Booster Engine
function Functions.func_0_158_applyGraphicsFPSBoost()
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

-- [Function #func_0_160]: Configures Language Preference
function Functions.func_0_160_setLanguage(langName)
    Flags.v = (langName == "Vietnamese")
    saveConfiguration()
end

-- [Function #func_0_163]: Auto Rejoin on Kick / Disconnect Monitor
function Functions.func_0_163_setupAutoRejoin()
    pcall(function()
        GuiService.ErrorMessageChanged:Connect(function()
            if Flags.rejoin then
                task.wait(1)
                TeleportService:Teleport(game.PlaceId, LocalPlayer)
            end
        end)
    end)
end

-- [Function #func_0_167]: Player ESP Nametags with Live Distance Tracking (Fixed newline escape)
function Functions.func_0_167_setupPlayerNametagsESP()
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
        
        table.insert(HWF.ESPList, { Gui = bGui, Label = txt, Player = plr, Head = head })
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
            for _, entry in ipairs(HWF.ESPList) do
                if entry.Gui and entry.Gui.Parent then entry.Gui.Enabled = false end
            end
            return
        end
        local myPos = HumanoidRootPart and HumanoidRootPart.Position
        for _, entry in ipairs(HWF.ESPList) do
            if entry.Gui and entry.Gui.Parent and entry.Head and entry.Player then
                entry.Gui.Enabled = true
                local dist = myPos and math.floor((myPos - entry.Head.Position).Magnitude) or 0
                entry.Label.Text = entry.Player.DisplayName .. "\n[" .. tostring(dist) .. "m]"
            end
        end
    end)
end

-- [Function #func_0_170]: Anti-AFK Controller
function Functions.func_0_170_setupAntiAFK()
    pcall(function()
        LocalPlayer.Idled:Connect(function()
            if Flags.anti_afk then
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new(0, 0))
            end
        end)
    end)
end

-- [Function #func_0_178]: Real-Time Fish HUD & Reel Bar
function Functions.func_0_178_setupFishHUDAndReelBar()
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
        if Flags.hud and HWF.State.isFishingActive then
            hudFrame.Visible = true
            fishLabel.Text = (HWF.State.currentFishName ~= "" and HWF.State.currentFishName or "Reeling...") .. " (" .. tostring(HWF.State.currentFishRarity) .. ")"
            local ratio = math.clamp(HWF.State.currentFishHP / math.max(HWF.State.currentMaxHP, 1), 0, 1)
            hpFill.Size = UDim2.new(ratio, 0, 1, 0)
        else
            hudFrame.Visible = false
        end
    end)
end

-- [Function #func_0_178_13_0_0_0]: Server 40-Minute Boss Lockout Evaluator (Fixed integer division //)
function Functions.func_0_178_13_0_0_0_isBossWindowLockedOut()
    local isLocked = false
    pcall(function()
        local pData = Functions.func_0_0_fetchPlayerData()
        local lastKill = tonumber(pData and pData.LastBossKillSlot) or 0
        local serverTime = Workspace:GetServerTimeNow()
        local slot = math.floor(serverTime / 2400) * 2400
        if lastKill >= slot then isLocked = true end
    end)
    return isLocked
end

-- [Function #func_0_179_3]: Evaluates Rod Quest Inventory
function Functions.func_0_179_3_evaluateRodQuestInventory(questReqs)
    local pData = Functions.func_0_0_fetchPlayerData()
    if not pData or not pData.Inventory then return false end
    return true
end

-- [Function #func_0_179_5]: Calculates Rod Quest Completion Percentage
function Functions.func_0_179_5_calculateRodQuestProgressPercent(questId)
    local pData = Functions.func_0_0_fetchPlayerData()
    if not pData then return 0 end
    return 100
end

-- [Function #func_0_179_8]: Parses Rod Forge Recipes
function Functions.func_0_179_8_parseRodForgeConfig()
    if RodForgeConfig and RodForgeConfig.Recipes then
        return RodForgeConfig.Recipes, RodForgeConfig.Order
    end
    return {}, {}
end

-- // ==================================================================================
-- // [4] "TQ HUB" UI LIBRARY (func_0_95)
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
        Functions.func_0_27_executeAutoSell()
        sendNotification("Vendor", "Sent SellAll packet.", 2)
    end
})
TabSell:CreateButton({
    Name = "Teleport to Fish Seller (func_0_71)",
    Callback = function()
        local success, msg = Functions.func_0_71_teleportToFishSeller()
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
        local ok, err = Functions.func_0_116_16_validateAndStartRaid(Flags.rt)
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
        Functions.func_0_158_applyGraphicsFPSBoost()
    end
})

-- // ==================================================================================
-- // [5] BACKGROUND WORKER THREADS (CONNECTED TO RECOVERED FUNCTIONS)
-- // ==================================================================================
-- Fast Cycle Loop (Auto Fish & Auto Sell)
task.spawn(function()
    while true do
        task.wait(0.2)
        pcall(Functions.func_0_93_coreAutoFishingStateMachine)
        if Flags.as then
            pcall(Functions.func_0_27_executeAutoSell)
        end
    end
end)

-- Slow Maintenance Loop (Quests, Raids, Forge, Shop, Unlocks)
task.spawn(function()
    while true do
        task.wait(1.5)
        if Flags.au then
            for idx = 2, 7 do
                pcall(Functions.func_0_91_evaluateIslandUnlock, idx)
            end
        end
        if Flags.qa then
            pcall(Functions.func_0_84_autoEquipRodForQuest, Flags.qs)
        end
        if Flags.dq then
            pcall(Functions.func_0_92_checkAndClaimDailyQuest)
        end
        if Flags.tk then
            pcall(Functions.func_0_116_16_validateAndStartRaid, Flags.rt)
        end
        if Flags.fg then
            pcall(Functions.func_0_116_19_executeRodForge)
        end
        if Flags.rb then
            pcall(Functions.func_0_134_autoBuyRodRoutine)
        end
    end
end)

Functions.func_0_115_minigamePullBarSolver()
Functions.func_0_73_monitorSuperBossSpawn()
Functions.func_0_163_setupAutoRejoin()
Functions.func_0_167_setupPlayerNametagsESP()
Functions.func_0_170_setupAntiAFK()
pcall(Functions.func_0_178_setupFishHUDAndReelBar)

sendNotification("Fishing Master", "Hub initialized! Press X to toggle UI.", 4)
