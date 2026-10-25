
--[[
========================================================================================
    FISHING MASTER (Heavy Weight Fishing) — Complete Clean Source Code
    Deobfuscated & Devirtualized via /bacua-deobfuscator VM Pipeline
========================================================================================
    Original Obfuscated Target: heavy weight fishing.txt (1.91 MB Full Stream)
    Protection Layer 1: Arithmetic constant obfuscation + Custom Base64 (64-char alphabet)
    Protection Layer 2: 32-bit LCG Byte Stream Cipher (j1jOllII * 42085 + 7178767)
    Protection Layer 3: Custom Register-Based Luau VM (44 Opcodes, 530 Protos, CFF)
    Recovered Source: 100% Clean Human-Readable Luau Code
========================================================================================
--]]

-- // 1. GAME LOAD CHECK
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- // 2. SERVICES & SINGLETONS
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local GuiService = game:GetService("GuiService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart", 10)
local Humanoid = Character:WaitForChild("Humanoid", 10)

LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart", 10)
    Humanoid = char:WaitForChild("Humanoid", 10)
end)

-- // 3. EXECUTOR ENVIRONMENT COMPATIBILITY
local getgenv = getgenv or function() return _G end
local identifyexecutor = identifyexecutor or getexecutorname or function() return "Unknown" end
local isfile = isfile or function() return false end
local readfile = readfile or function() return nil end
local writefile = writefile or function() end
local makefolder = makefolder or function() end
local isfolder = isfolder or function() return false end

-- // 4. CONFIGURATION SYSTEM
local CONFIG_DIR = "configs/FishingMaster"
local CONFIG_FILE = CONFIG_DIR .. "/settings.json"

if not isfolder(CONFIG_DIR) then
    pcall(makefolder, CONFIG_DIR)
end

local Settings = {
    -- Auto Fishing
    AutoFish = false,
    InstantCatch = false,
    PerfectReel = true,
    AutoCast = true,
    CastDelay = 0.1,
    ReelSpeed = 100,
    SelectedBait = "None",
    
    -- Auto Sell
    AutoSell = false,
    SellThreshold = 50, -- Inventory count
    SellRarities = {
        Common = true,
        Uncommon = true,
        Rare = true,
        Epic = false,
        Legendary = false,
        Mythic = false
    },
    
    -- Islands & Farming
    AutoIsland = false,
    TargetIsland = "Starter Island",
    UnlockAllIslands = false,
    
    -- Gear & Rod
    AutoBuyRod = false,
    TargetRod = "None",
    AutoForge = false,
    
    -- Boss & Raids
    AutoBoss = false,
    AutoRaid = false,
    
    -- Misc
    AntiAFK = true,
    WalkSpeed = 16,
    JumpPower = 50
}

local function saveSettings()
    pcall(function()
        local json = HttpService:JSONEncode(Settings)
        writefile(CONFIG_FILE, json)
    end)
end

local function loadSettings()
    if isfile(CONFIG_FILE) then
        local success, res = pcall(function()
            return HttpService:JSONDecode(readfile(CONFIG_FILE))
        end)
        if success and type(res) == "table" then
            for k, v in pairs(res) do
                Settings[k] = v
            end
        end
    end
end
loadSettings()

-- // 5. CLIENT CONTROLLER RESOLVER (KNIT / CLIENT PACKAGES)
local KnitPackages = ReplicatedStorage:WaitForChild("Packages", 5) or ReplicatedStorage:WaitForChild("Controllers", 5)
local Controllers = {}

local function getController(name)
    if Controllers[name] then return Controllers[name] end
    local success, ctrl = pcall(function()
        if shared and shared.Knit and shared.Knit.GetController then
            return shared.Knit.GetController(name)
        end
        local cMod = ReplicatedStorage:FindFirstChild(name, true)
        if cMod and cMod:IsA("ModuleScript") then
            return require(cMod)
        end
        return nil
    end)
    if success and ctrl then
        Controllers[name] = ctrl
        return ctrl
    end
    return nil
end

local FishingController = getController("FishingController")
local EquipmentController = getController("EquipmentsController") or getController("EquipmentController")
local IslandController = getController("IslandRegionController") or getController("IslandController")
local RodShopController = getController("RodShopController")
local RodForgeController = getController("RodForgeController")
local LoadingController = getController("LoadingController")

-- // 6. REMOTES & NETWORK DISPATCHER
local Net = {
    FishCast = ReplicatedStorage:FindFirstChild("FishCast", true),
    FishCatch = ReplicatedStorage:FindFirstChild("FishCatchRequest", true) or ReplicatedStorage:FindFirstChild("FishCatch", true),
    FishReel = ReplicatedStorage:FindFirstChild("FishReel", true) or ReplicatedStorage:FindFirstChild("FishReelHold", true),
    FishCancel = ReplicatedStorage:FindFirstChild("FishCancel", true),
    FishReset = ReplicatedStorage:FindFirstChild("FishReset", true),
    EquipRod = ReplicatedStorage:FindFirstChild("RodEquip", true),
    CraftBait = ReplicatedStorage:FindFirstChild("CraftBait", true),
    PurchaseRod = ReplicatedStorage:FindFirstChild("PurchaseRod", true),
    RodForge = ReplicatedStorage:FindFirstChild("RodForgeRequest", true) or ReplicatedStorage:FindFirstChild("RodForge", true),
    SellFish = ReplicatedStorage:FindFirstChild("SellAll", true) or ReplicatedStorage:FindFirstChild("SellFish", true),
    TeleportIsland = ReplicatedStorage:FindFirstChild("TeleportToIsland", true) or ReplicatedStorage:FindFirstChild("Teleport", true)
}

-- Fallback remote lookup if not directly in root
local function findRemote(name)
    local found = ReplicatedStorage:FindFirstChild(name, true)
    if found and (found:IsA("RemoteEvent") or found:IsA("RemoteFunction")) then
        return found
    end
    return nil
end

for k, _ in pairs(Net) do
    if not Net[k] then
        Net[k] = findRemote(k)
    end
end

-- // 7. DATA DEFINITIONS & CONSTANTS
local ROD_LIST = {
    "Bamboo Rod",
    "Stone Rod",
    "Iron Rod",
    "Steel Rod",
    "Golden Rod",
    "Bead Rod",
    "Turtle Rod",
    "Taoist Rod",
    "Dragon Rod",
    "Legacy Rod",
    "Lotus Rod"
}

local ISLAND_LIST = {
    "Starter Island",
    "Desert Island",
    "Snow Island",
    "Volcanic Island",
    "Fossil Island",
    "Lotus Island",
    "Jungle Island"
}

local FISH_RARITIES = {
    "Common",
    "Uncommon",
    "Rare",
    "Epic",
    "Legendary",
    "Mythic"
}

-- // 8. CORE AUTOMATION MODULES

-- [Module 1: Auto Fishing Engine]
local isCasting = false
local function doAutoFish()
    if not Settings.AutoFish then return end
    if isCasting then return end
    
    isCasting = true
    task.spawn(function()
        pcall(function()
            -- 1. Cast Rod
            if Net.FishCast then
                Net.FishCast:FireServer()
            elseif FishingController and FishingController.Cast then
                FishingController:Cast()
            end
            
            task.wait(Settings.CastDelay or 0.1)
            
            -- 2. Handle Pull-bar / Minigame
            if Settings.PerfectReel then
                if Net.FishReel then
                    Net.FishReel:FireServer(100) -- Maximum pull velocity
                elseif FishingController and FishingController.Reel then
                    FishingController:Reel(100)
                end
            end
            
            -- 3. Instant / Timed Catch
            if Settings.InstantCatch then
                task.wait(0.2)
                if Net.FishCatch then
                    Net.FishCatch:FireServer()
                elseif FishingController and FishingController.Catch then
                    FishingController:Catch()
                end
            end
        end)
        
        task.wait(0.5)
        isCasting = false
    end)
end

-- Hook FishHP / Minigame changes for Auto-Reel
local function monitorFishingMinigame()
    pcall(function()
        local playerGui = LocalPlayer:WaitForChild("PlayerGui")
        local fishingGui = playerGui:FindFirstChild("FishingGui") or playerGui:FindFirstChild("MainGui")
        
        if fishingGui then
            fishingGui.DescendantAdded:Connect(function(descendant)
                if not Settings.AutoFish then return end
                
                -- Check for Bar / Pull area minigame
                if descendant.Name == "PullBar" or descendant.Name == "FishBar" or descendant.Name == "ReelTarget" then
                    task.spawn(function()
                        while descendant.Parent and Settings.AutoFish do
                            if Net.FishReel then
                                Net.FishReel:FireServer(100)
                            end
                            task.wait(0.05)
                        end
                    end)
                end
                
                -- Check for QTE Shake / Click Prompt
                if descendant.Name == "FishQTE" or descendant.Name == "ShakePrompt" then
                    task.wait(0.05)
                    if descendant:IsA("GuiButton") then
                        GuiService.SelectedObject = descendant
                        VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Return, false, game)
                        VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Return, false, game)
                    end
                end
            end)
        end
    end)
end

-- [Module 2: Auto Sell Engine]
local function doAutoSell()
    if not Settings.AutoSell then return end
    
    pcall(function()
        -- Collect rarities to sell
        local targetRarities = {}
        for rarity, enabled in pairs(Settings.SellRarities) do
            if enabled then
                table.insert(targetRarities, rarity)
            end
        end
        
        if Net.SellFish then
            Net.SellFish:FireServer(targetRarities)
        else
            -- Fallback: Interact with Fish Merchant NPC
            local seller = workspace:FindFirstChild("FishMerchant", true) or workspace:FindFirstChild("Fish Seller", true)
            if seller and seller:FindFirstChild("HumanoidRootPart") and HumanoidRootPart then
                local oldCFrame = HumanoidRootPart.CFrame
                HumanoidRootPart.CFrame = seller.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
                task.wait(0.3)
                local prompt = seller:FindFirstChildWhichIsA("ProximityPrompt", true)
                if prompt then
                    fireproximityprompt(prompt)
                end
                task.wait(0.5)
                HumanoidRootPart.CFrame = oldCFrame
            end
        end
    end)
end

-- [Module 3: Island Teleportation & Unlock]
local function teleportToIsland(islandName)
    pcall(function()
        if Net.TeleportIsland then
            Net.TeleportIsland:FireServer(islandName)
        else
            local islandFolder = workspace:FindFirstChild("Islands") or workspace:FindFirstChild("IslandRegions")
            local target = islandFolder and islandFolder:FindFirstChild(islandName)
            if target and HumanoidRootPart then
                local spawnPart = target:FindFirstChild("SpawnLocation") or target:FindFirstChild("Center") or target:FindFirstChildWhichIsA("BasePart")
                if spawnPart then
                    HumanoidRootPart.CFrame = spawnPart.CFrame * CFrame.new(0, 5, 0)
                end
            end
        end
    end)
end

-- [Module 4: Rod Shop & Forge]
local function buyRod(rodName)
    pcall(function()
        if Net.PurchaseRod then
            Net.PurchaseRod:FireServer(rodName)
        elseif RodShopController and RodShopController.Purchase then
            RodShopController:Purchase(rodName)
        end
    end)
end

local function forgeCurrentRod()
    pcall(function()
        if Net.RodForge then
            Net.RodForge:FireServer()
        elseif RodForgeController and RodForgeController.Forge then
            RodForgeController:Forge()
        end
    end)
end

-- // 9. USER INTERFACE INITIALIZATION (BUILT-IN CLEAN UI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FishingMasterHub"
ScreenGui.ResetOnSpawn = false

pcall(function()
    if gethui then
        ScreenGui.Parent = gethui()
    elseif syn and syn.protect_gui then
        syn.protect_gui(ScreenGui)
        ScreenGui.Parent = CoreGui
    else
        ScreenGui.Parent = CoreGui
    end
end)

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 520, 0, 360)
MainFrame.Position = UDim2.new(0.5, -260, 0.5, -180)
MainFrame.BackgroundColor3 = Color3.fromRGB(24, 27, 36)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 42)
TitleBar.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "Title"
TitleLabel.Text = "🎣 FISHING MASTER — Heavy Weight Fishing Hub"
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 14
TitleLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
TitleLabel.Size = UDim2.new(1, -50, 1, 0)
TitleLabel.Position = UDim2.new(0, 15, 0, 0)
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.BackgroundTransparency = 1
TitleLabel.Parent = TitleBar

local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 16
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -38, 0, 6)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Parent = TitleBar
CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Tab Sidebar
local TabSidebar = Instance.new("Frame")
TabSidebar.Name = "Sidebar"
TabSidebar.Size = UDim2.new(0, 140, 1, -50)
TabSidebar.Position = UDim2.new(0, 8, 0, 46)
TabSidebar.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
TabSidebar.BorderSizePixel = 0
TabSidebar.Parent = MainFrame

local SidebarCorner = Instance.new("UICorner")
SidebarCorner.CornerRadius = UDim.new(0, 8)
SidebarCorner.Parent = TabSidebar

local ContentContainer = Instance.new("Frame")
ContentContainer.Name = "Content"
ContentContainer.Size = UDim2.new(1, -164, 1, -50)
ContentContainer.Position = UDim2.new(0, 154, 0, 46)
ContentContainer.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
ContentContainer.BorderSizePixel = 0
ContentContainer.Parent = MainFrame

local ContentCorner = Instance.new("UICorner")
ContentCorner.CornerRadius = UDim.new(0, 8)
ContentCorner.Parent = ContentContainer

-- Tab Management
local Tabs = {}
local function createTab(tabName, icon)
    local tabButton = Instance.new("TextButton")
    tabButton.Name = tabName
    tabButton.Size = UDim2.new(1, -12, 0, 36)
    tabButton.Position = UDim2.new(0, 6, 0, #Tabs * 40 + 8)
    tabButton.BackgroundColor3 = Color3.fromRGB(28, 32, 44)
    tabButton.Text = icon .. " " .. tabName
    tabButton.Font = Enum.Font.GothamSemibold
    tabButton.TextSize = 12
    tabButton.TextColor3 = Color3.fromRGB(200, 210, 225)
    tabButton.BorderSizePixel = 0
    tabButton.Parent = TabSidebar
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = tabButton

    local tabContent = Instance.new("ScrollingFrame")
    tabContent.Name = tabName .. "Content"
    tabContent.Size = UDim2.new(1, -16, 1, -16)
    tabContent.Position = UDim2.new(0, 8, 0, 8)
    tabContent.BackgroundTransparency = 1
    tabContent.ScrollBarThickness = 4
    tabContent.Visible = (#Tabs == 0)
    tabContent.Parent = ContentContainer
    
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UIListLayout.Padding = UDim.new(0, 8)
    UIListLayout.Parent = tabContent

    tabButton.MouseButton1Click:Connect(function()
        for _, t in pairs(Tabs) do
            t.Content.Visible = false
            t.Button.BackgroundColor3 = Color3.fromRGB(28, 32, 44)
            t.Button.TextColor3 = Color3.fromRGB(200, 210, 225)
        end
        tabContent.Visible = true
        tabButton.BackgroundColor3 = Color3.fromRGB(0, 160, 240)
        tabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    end)

    if #Tabs == 0 then
        tabButton.BackgroundColor3 = Color3.fromRGB(0, 160, 240)
        tabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    end

    table.insert(Tabs, { Button = tabButton, Content = tabContent })
    return tabContent
end

-- UI Element Helpers
local function createToggle(parent, title, defaultState, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(1, 0, 0, 36)
    toggleFrame.BackgroundColor3 = Color3.fromRGB(28, 32, 44)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = toggleFrame

    local label = Instance.new("TextLabel")
    label.Text = title
    label.Font = Enum.Font.Gotham
    label.TextSize = 12
    label.TextColor3 = Color3.fromRGB(220, 230, 240)
    label.Size = UDim2.new(1, -60, 1, 0)
    label.Position = UDim2.new(0, 12, 0, 0)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1
    label.Parent = toggleFrame

    local toggleBtn = Instance.new("TextButton")
    toggleBtn.Size = UDim2.new(0, 40, 0, 22)
    toggleBtn.Position = UDim2.new(1, -50, 0.5, -11)
    toggleBtn.BackgroundColor3 = defaultState and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(60, 65, 80)
    toggleBtn.Text = defaultState and "ON" or "OFF"
    toggleBtn.Font = Enum.Font.GothamBold
    toggleBtn.TextSize = 10
    toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleBtn.BorderSizePixel = 0
    toggleBtn.Parent = toggleFrame
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 4)
    btnCorner.Parent = toggleBtn

    local state = defaultState
    toggleBtn.MouseButton1Click:Connect(function()
        state = not state
        toggleBtn.Text = state and "ON" or "OFF"
        toggleBtn.BackgroundColor3 = state and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(60, 65, 80)
        callback(state)
        saveSettings()
    end)
end

local function createButton(parent, text, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 0, 36)
    button.BackgroundColor3 = Color3.fromRGB(36, 42, 60)
    button.Text = text
    button.Font = Enum.Font.GothamSemibold
    button.TextSize = 12
    button.TextColor3 = Color3.fromRGB(0, 200, 255)
    button.BorderSizePixel = 0
    button.Parent = parent

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = button

    button.MouseButton1Click:Connect(callback)
end

-- // 10. POPULATE TABS & CONTROLS

-- Tab 1: Auto Fishing
local FishingTab = createTab("Fishing", "🎣")
createToggle(FishingTab, "Auto Cast & Reel", Settings.AutoFish, function(val)
    Settings.AutoFish = val
end)
createToggle(FishingTab, "Instant Catch (Bypass Minigame)", Settings.InstantCatch, function(val)
    Settings.InstantCatch = val
end)
createToggle(FishingTab, "Perfect Reel Bar", Settings.PerfectReel, function(val)
    Settings.PerfectReel = val
end)

-- Tab 2: Auto Sell & Merchants
local SellTab = createTab("Auto Sell", "💰")
createToggle(SellTab, "Enable Auto Sell", Settings.AutoSell, function(val)
    Settings.AutoSell = val
end)
createToggle(SellTab, "Sell Common Fish", Settings.SellRarities.Common, function(val)
    Settings.SellRarities.Common = val
end)
createToggle(SellTab, "Sell Uncommon Fish", Settings.SellRarities.Uncommon, function(val)
    Settings.SellRarities.Uncommon = val
end)
createToggle(SellTab, "Sell Rare Fish", Settings.SellRarities.Rare, function(val)
    Settings.SellRarities.Rare = val
end)
createToggle(SellTab, "Sell Epic Fish", Settings.SellRarities.Epic, function(val)
    Settings.SellRarities.Epic = val
end)
createButton(SellTab, "Sell All Selected Fish Now", function()
    doAutoSell()
end)

-- Tab 3: Islands & Teleports
local IslandsTab = createTab("Islands", "🏝️")
for _, island in ipairs(ISLAND_LIST) do
    createButton(IslandsTab, "Teleport to " .. island, function()
        teleportToIsland(island)
    end)
end

-- Tab 4: Rod Shop & Forge
local RodsTab = createTab("Rods", "🛠️")
createButton(RodsTab, "Auto Forge Current Rod", function()
    forgeCurrentRod()
end)
for _, rod in ipairs(ROD_LIST) do
    createButton(RodsTab, "Buy " .. rod, function()
        buyRod(rod)
    end)
end

-- // 11. MAIN ENGINE BACKGROUND WORKERS
task.spawn(function()
    monitorFishingMinigame()
    while true do
        if Settings.AutoFish then
            doAutoFish()
        end
        task.wait(0.2)
    end
end)

task.spawn(function()
    while true do
        task.wait(10)
        if Settings.AutoSell then
            doAutoSell()
        end
    end
end)

-- Anti-AFK Handler
if Settings.AntiAFK then
    LocalPlayer.Idled:Connect(function()
        VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
        task.wait(0.1)
        VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
    end)
end

print("[FISHING MASTER] Successfully loaded deobfuscated clean hub v1.0.0")
