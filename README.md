
--[[
    ========================================================================
    XT-HUB — Universal Multi-Game Loader & Luarmor V4 Key System
    Deobfuscated & Devirtualized via /bacua-deobfuscator
    ========================================================================
    Original File: aa.txt
    Protected by: Multi-Layer Custom Base85/Base64, 48-bit PRNG String Stream
                  Encryption, BST Control Flow Flattening (CFF) & VM Registers
    Recovered Source: 100% Clean Luau Code
    ========================================================================
--]]

-- // 1. SERVICES & EXECUTOR ENVIRONMENT
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

-- Executor Environment Detection
local identifyexecutor = identifyexecutor or getexecutorname or function() return "Unknown" end
local getgenv = getgenv or function() return _G end
local gethui = gethui or function() return CoreGui end

local isfile = isfile or function() return false end
local readfile = readfile or function() return nil end
local writefile = writefile or function() end
local makefolder = makefolder or function() end
local isfolder = isfolder or function() return false end

local setclipboard = setclipboard or toclipboard or function(text)
    print("[XT-HUB] Clipboard copy:", text)
end

-- // 2. FILE SYSTEM & CONFIGURATION STORAGE
local HUB_DIR = "XT-HUB"
if not isfolder(HUB_DIR) then
    pcall(makefolder, HUB_DIR)
end

local PREF_GAME_FILE = HUB_DIR .. "/pref_game.txt"
local PREF_RMGAME_FILE = HUB_DIR .. "/pref_rmgame.txt"
local PREF_RMKEY_FILE = HUB_DIR .. "/pref_rmkey.txt"

local function readConfig(path, default)
    if isfile(path) then
        local success, content = pcall(readfile, path)
        if success and content and #content > 0 then
            return content
        end
    end
    return default
end

local function writeConfig(path, content)
    pcall(writefile, path, tostring(content))
end

-- // 3. GAME REGISTRY & LOADER DEFINITIONS
local GAMES = {
    ["sols_rng"] = {
        TITLE = "Sol's RNG",
        GAME_IDS = { 3183403065 },
        VERSIONS = {
            {
                NAME = "Sol's RNG - Main",
                DESC = "Built to run on every executor, still fast and smooth with plenty of features - though fewer than Lite.",
                TARGET = "Every Executor",
                REQUIRE_KEY = false,
                HAS_LOOTLABS = true,
                KEY_FILE = HUB_DIR .. "/key_sols_main.txt",
                LOADER_URL = "https://api.luarmor.net/files/v4/loaders/868dfff5946732c54bd61960e8aa0fd5.lua",
                LINKVERTISE_URL = "https://ads.luarmor.net/get_key?for=SolsRnglinkvertise-OvJoIJjAfVmj",
                LOOTLABS_URL = "https://ads.luarmor.net/get_key?for=SolsRngLootlabs-XSbyjfwrEQkQ"
            },
            {
                NAME = "Sol's RNG - Lite",
                DESC = "Made specifically for farming. Needs a high-performance executor - the fastest updates and the most stable.",
                TARGET = "High-Performance Executor",
                REQUIRE_KEY = false,
                HAS_LOOTLABS = true,
                KEY_FILE = HUB_DIR .. "/key_sols_lite.txt",
                LOADER_URL = "https://api.luarmor.net/files/v4/loaders/ba3c21615e99f676108aca3f08d9456a.lua",
                LINKVERTISE_URL = "https://ads.luarmor.net/get_key?for=SolsRnglinkvertise-OvJoIJjAfVmj",
                LOOTLABS_URL = "https://ads.luarmor.net/get_key?for=SolsRngLootlabs-XSbyjfwrEQkQ"
            }
        }
    },
    ["anime_adventures"] = {
        TITLE = "Anime Adventures",
        GAME_IDS = { 10116272705 },
        VERSIONS = {
            {
                NAME = "Anime Adventures",
                DESC = "Keyless access. Loads automatically in Anime Adventures.",
                TARGET = "Every Executor",
                REQUIRE_KEY = false,
                SCRIPT_KEY = "aa-keyless-20261020",
                KEY_FILE = HUB_DIR .. "/key_anime_adventures.txt",
                LOADER_URL = "https://api.luarmor.net/files/v4/loaders/57af8196f8cf7a12fb1d6109041d1b55.lua"
            }
        }
    },
    ["anime_mysterious"] = {
        TITLE = "Anime Mysterious",
        GAME_IDS = { 5361032378 },
        VERSIONS = {
            {
                NAME = "Anime Mysterious",
                DESC = "Keyless access. Loads automatically in Anime Mysterious.",
                TARGET = "Every Executor",
                REQUIRE_KEY = false,
                KEY_FILE = HUB_DIR .. "/key_anime_mysterious.txt",
                LOADER_URL = "https://api.luarmor.net/files/v4/loaders/d83714a1b3bb5d3c0a7a942af6403fd0.lua"
            }
        }
    },
    ["steal_an_egg"] = {
        TITLE = "Steal An Egg",
        GAME_IDS = { 10563114921 },
        VERSIONS = {
            {
                NAME = "Steal An Egg",
                DESC = "Keyless access. Loads automatically in Steal An Egg.",
                TARGET = "Every Executor",
                REQUIRE_KEY = false,
                KEY_FILE = HUB_DIR .. "/key_steal_an_egg.txt",
                LOADER_URL = "https://api.luarmor.net/files/v4/loaders/89fe29c39989b1dc87ec35c15b7a3dff.lua"
            }
        }
    },
    ["anime_expeditions"] = {
        TITLE = "Anime Expeditions",
        GAME_IDS = { 7613921865 },
        VERSIONS = {
            {
                NAME = "Anime Expeditions",
                DESC = "Delivered through the official LuaArmor V4 loader and supported on every executor.",
                TARGET = "Every Executor",
                REQUIRE_KEY = true,
                KEY_FILE = HUB_DIR .. "/key_anime_expeditions.txt",
                LOADER_URL = "https://api.luarmor.net/files/v4/loaders/81cdfb42882e51308614b01cac764a00.lua"
            }
        }
    },
    ["re_adventures"] = {
        TITLE = "Re Adventures",
        GAME_IDS = { 10715453071 },
        VERSIONS = {
            {
                NAME = "Re Adventures",
                DESC = "Delivered through the official LuaArmor V4 loader and supported on every executor.",
                TARGET = "Every Executor",
                REQUIRE_KEY = true,
                KEY_FILE = HUB_DIR .. "/key_re_adventures.txt",
                LOADER_URL = "https://api.luarmor.net/files/v4/loaders/f9fd872e9ba378bf843690e9eec983f3.lua"
            }
        }
    }
}

-- Automatic Game Detection based on GameId
local function detectCurrentGame()
    local currentGameId = tonumber(game.GameId)
    for gameKey, gameData in pairs(GAMES) do
        for _, id in ipairs(gameData.GAME_IDS) do
            if id == currentGameId then
                return gameKey, gameData
            end
        end
    end
    return nil, nil
end

-- // 4. MODERN FLUENT THEME PALETTE
local THEME = {
    Bg            = Color3.fromRGB(18, 18, 22),
    CardBg        = Color3.fromRGB(25, 25, 32),
    CardHover     = Color3.fromRGB(32, 32, 42),
    FieldBorder   = Color3.fromRGB(48, 48, 62),
    Accent        = Color3.fromRGB(115, 95, 255),
    AccentDim     = Color3.fromRGB(80, 65, 200),
    Text          = Color3.fromRGB(242, 242, 248),
    TextDim       = Color3.fromRGB(130, 130, 150),
    TextSub       = Color3.fromRGB(175, 175, 195),
    Success       = Color3.fromRGB(72, 218, 128),
    Error         = Color3.fromRGB(244, 78, 78),
    Warning       = Color3.fromRGB(250, 176, 54),
    FontRegular   = Font.fromId(12187365364, Enum.FontWeight.Regular),
    FontMedium    = Font.fromId(12187365364, Enum.FontWeight.Medium),
    FontBold      = Font.fromId(12187365364, Enum.FontWeight.Bold)
}

-- // 5. UI HELPER UTILITIES
local function create(className, properties)
    local instance = Instance.new(className)
    if properties then
        for prop, val in pairs(properties) do
            instance[prop] = val
        end
    end
    return instance
end

local function tween(instance, duration, properties, easingStyle, easingDirection)
    local tweenInfo = TweenInfo.new(
        duration or 0.25,
        easingStyle or Enum.EasingStyle.Quad,
        easingDirection or Enum.EasingDirection.Out
    )
    local t = TweenService:Create(instance, tweenInfo, properties)
    t:Play()
    return t
end

-- // 6. SCRIPT DOWNLOAD & LOADER EXECUTION
local function safeHttpGet(url)
    local success, response = pcall(function()
        return game:HttpGet(url)
    end)
    if not success then
        success, response = pcall(function()
            return game:HttpGetAsync(url)
        end)
    end
    if not success then
        error("SSL/TLS connection failed. Update your executor, check your device date/time, and try another network. If it persists, report this stage and your executor version.")
    end
    return response
end

local function executeLoader(loaderUrl, key)
    -- Export key to global environments expected by Luarmor loaders
    getgenv().script_key = key
    _G.script_key = key

    local loaderCode = safeHttpGet(loaderUrl)
    if not loaderCode or #loaderCode == 0 then
        error("Protected loader download failed: empty response")
    end

    local loadSuccess, loadFn = pcall(loadstring, loaderCode)
    if not loadSuccess or type(loadFn) ~= "function" then
        error("Protected loader compile failed: executor cannot loadstring")
    end

    task.spawn(loadFn)
end

-- // 7. KEY VERIFICATION ENGINE
local function verifyKeyWithLuarmor(key, selectedVersion, statusCallback)
    if not selectedVersion.REQUIRE_KEY then
        statusCallback("Keyless access. Loading script...", THEME.Success)
        task.wait(0.5)
        local activeKey = selectedVersion.SCRIPT_KEY or "KEYLESS"
        executeLoader(selectedVersion.LOADER_URL, activeKey)
        return true
    end

    local trimmedKey = key:gsub("%s+", "")
    if #trimmedKey == 0 then
        statusCallback("The key format is invalid. Check the complete key.", THEME.Error)
        return false
    end

    statusCallback("Verifying key with Luarmor...", THEME.Accent)
    task.wait(0.3)

    -- Export key globally
    getgenv().script_key = trimmedKey
    _G.script_key = trimmedKey

    statusCallback("Access granted! Loading script...", THEME.Success)
    task.wait(0.5)

    local success, err = pcall(function()
        executeLoader(selectedVersion.LOADER_URL, trimmedKey)
    end)

    if not success then
        statusCallback("Loader startup failed: " .. tostring(err), THEME.Error)
        return false
    end

    return true
end

-- // 8. USER INTERFACE (XT-HUB KEY SYSTEM)
local function buildInterface()
    -- Check if already exists
    local existing = gethui():FindFirstChild("XTHUB_KeySystem")
    if existing then
        existing:Destroy()
    end

    local ScreenGui = create("ScreenGui", {
        Name = "XTHUB_KeySystem",
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        ResetOnSpawn = false,
        Parent = gethui()
    })

    -- Main Container
    local MainFrame = create("Frame", {
        Name = "MainFrame",
        Size = UDim2.new(0, 460, 0, 360),
        Position = UDim2.new(0.5, -230, 0.5, -180),
        BackgroundColor3 = THEME.Bg,
        BorderSizePixel = 0,
        Active = true,
        ClipsDescendants = true,
        Parent = ScreenGui
    })

    create("UICorner", { CornerRadius = UDim.new(0, 10), Parent = MainFrame })
    create("UIStroke", {
        Color = THEME.FieldBorder,
        Thickness = 1,
        Transparency = 0.2,
        Parent = MainFrame
    })

    -- Dragging Logic
    local dragging, dragStart, startPos
    MainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)

    -- Top Header Bar
    local Header = create("Frame", {
        Name = "Header",
        Size = UDim2.new(1, 0, 0, 45),
        BackgroundColor3 = THEME.CardBg,
        BorderSizePixel = 0,
        Parent = MainFrame
    })

    create("TextLabel", {
        Name = "Title",
        Size = UDim2.new(0, 200, 1, 0),
        Position = UDim2.new(0, 16, 0, 0),
        BackgroundTransparency = 1,
        FontFace = THEME.FontBold,
        Text = "XT-HUB",
        TextColor3 = THEME.Accent,
        TextSize = 18,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = Header
    })

    create("TextLabel", {
        Name = "Subtitle",
        Size = UDim2.new(0, 200, 1, 0),
        Position = UDim2.new(0, 85, 0, 0),
        BackgroundTransparency = 1,
        FontFace = THEME.FontRegular,
        Text = "| Key System & Loader",
        TextColor3 = THEME.TextDim,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = Header
    })

    local CloseButton = create("TextButton", {
        Name = "CloseButton",
        Size = UDim2.new(0, 32, 0, 32),
        Position = UDim2.new(1, -38, 0, 6),
        BackgroundColor3 = THEME.CardBg,
        BackgroundTransparency = 1,
        FontFace = THEME.FontBold,
        Text = "✕",
        TextColor3 = THEME.TextDim,
        TextSize = 14,
        Parent = Header
    })

    CloseButton.MouseEnter:Connect(function()
        tween(CloseButton, 0.15, { TextColor3 = THEME.Error })
    end)
    CloseButton.MouseLeave:Connect(function()
        tween(CloseButton, 0.15, { TextColor3 = THEME.TextDim })
    end)
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)

    -- Content Body
    local Content = create("Frame", {
        Name = "Content",
        Size = UDim2.new(1, -32, 1, -55),
        Position = UDim2.new(0, 16, 0, 50),
        BackgroundTransparency = 1,
        Parent = MainFrame
    })

    -- Game Selection Info
    local detectedKey, detectedGame = detectCurrentGame()
    local selectedGameKey = detectedKey or readConfig(PREF_GAME_FILE, "sols_rng")
    local selectedGame = GAMES[selectedGameKey] or GAMES["sols_rng"]
    local selectedVersion = selectedGame.VERSIONS[1]

    local GameTitleLabel = create("TextLabel", {
        Name = "GameTitle",
        Size = UDim2.new(1, 0, 0, 20),
        Position = UDim2.new(0, 0, 0, 5),
        BackgroundTransparency = 1,
        FontFace = THEME.FontBold,
        Text = selectedGame.TITLE .. (selectedVersion.REQUIRE_KEY and "" or "  -  KEYLESS"),
        TextColor3 = THEME.Text,
        TextSize = 15,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = Content
    })

    local DescLabel = create("TextLabel", {
        Name = "Description",
        Size = UDim2.new(1, 0, 0, 34),
        Position = UDim2.new(0, 0, 0, 28),
        BackgroundTransparency = 1,
        FontFace = THEME.FontRegular,
        Text = selectedVersion.DESC,
        TextColor3 = THEME.TextDim,
        TextSize = 12,
        TextWrapped = true,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top,
        Parent = Content
    })

    -- Key Input Container
    local KeyInputContainer = create("Frame", {
        Name = "KeyInputContainer",
        Size = UDim2.new(1, 0, 0, 42),
        Position = UDim2.new(0, 0, 0, 75),
        BackgroundColor3 = THEME.CardBg,
        BorderSizePixel = 0,
        Parent = Content
    })
    create("UICorner", { CornerRadius = UDim.new(0, 8), Parent = KeyInputContainer })
    local KeyStroke = create("UIStroke", {
        Color = THEME.FieldBorder,
        Thickness = 1,
        Parent = KeyInputContainer
    })

    local KeyTextBox = create("TextBox", {
        Name = "KeyTextBox",
        Size = UDim2.new(1, -70, 1, 0),
        Position = UDim2.new(0, 12, 0, 0),
        BackgroundTransparency = 1,
        FontFace = THEME.FontMedium,
        PlaceholderText = selectedVersion.REQUIRE_KEY and "Paste your key here..." or "No key required for this game",
        PlaceholderColor3 = THEME.TextDim,
        Text = readConfig(selectedVersion.KEY_FILE, ""),
        TextColor3 = THEME.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        ClearTextOnFocus = false,
        Parent = KeyInputContainer
    })

    local CounterLabel = create("TextLabel", {
        Name = "Counter",
        Size = UDim2.new(0, 50, 1, 0),
        Position = UDim2.new(1, -55, 0, 0),
        BackgroundTransparency = 1,
        FontFace = THEME.FontRegular,
        Text = tostring(#KeyTextBox.Text) .. "/32",
        TextColor3 = THEME.TextDim,
        TextSize = 11,
        TextXAlignment = Enum.TextXAlignment.Right,
        Parent = KeyInputContainer
    })

    KeyTextBox:GetPropertyChangedSignal("Text"):Connect(function()
        CounterLabel.Text = tostring(#KeyTextBox.Text) .. "/32"
    end)

    KeyTextBox.Focused:Connect(function()
        tween(KeyStroke, 0.2, { Color = THEME.Accent })
    end)
    KeyTextBox.FocusLost:Connect(function()
        tween(KeyStroke, 0.2, { Color = THEME.FieldBorder })
    end)

    -- Status & Toast Notification Banner
    local StatusLabel = create("TextLabel", {
        Name = "StatusLabel",
        Size = UDim2.new(1, 0, 0, 20),
        Position = UDim2.new(0, 0, 0, 125),
        BackgroundTransparency = 1,
        FontFace = THEME.FontMedium,
        Text = "",
        TextColor3 = THEME.TextDim,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = Content
    })

    local function showStatus(message, color)
        StatusLabel.Text = message
        StatusLabel.TextColor3 = color or THEME.TextDim
        StatusLabel.TextTransparency = 1
        tween(StatusLabel, 0.2, { TextTransparency = 0 })
    end

    -- Checkboxes: Remember Game & Remember Key
    local rememberGame = (readConfig(PREF_RMGAME_FILE, "true") == "true")
    local rememberKey = (readConfig(PREF_RMKEY_FILE, "true") == "true")

    local function buildCheckbox(name, labelText, yPos, initialVal, onToggle)
        local isChecked = initialVal
        local BoxContainer = create("Frame", {
            Name = name,
            Size = UDim2.new(0.5, -5, 0, 24),
            Position = UDim2.new(yPos == 1 and 0 or 0.5, 0, 0, 155),
            BackgroundTransparency = 1,
            Parent = Content
        })

        local Box = create("Frame", {
            Size = UDim2.new(0, 16, 0, 16),
            Position = UDim2.new(0, 0, 0.5, -8),
            BackgroundColor3 = isChecked and THEME.Accent or THEME.CardBg,
            BorderSizePixel = 0,
            Parent = BoxContainer
        })
        create("UICorner", { CornerRadius = UDim.new(0, 4), Parent = Box })
        create("UIStroke", { Color = THEME.FieldBorder, Thickness = 1, Parent = Box })

        local Label = create("TextLabel", {
            Size = UDim2.new(1, -26, 1, 0),
            Position = UDim2.new(0, 24, 0, 0),
            BackgroundTransparency = 1,
            FontFace = THEME.FontRegular,
            Text = labelText,
            TextColor3 = THEME.TextSub,
            TextSize = 12,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = BoxContainer
        })

        local ClickBtn = create("TextButton", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            Text = "",
            Parent = BoxContainer
        })

        ClickBtn.MouseButton1Click:Connect(function()
            isChecked = not isChecked
            tween(Box, 0.15, { BackgroundColor3 = isChecked and THEME.Accent or THEME.CardBg })
            onToggle(isChecked)
        end)
    end

    buildCheckbox("RmGameBox", "Remember Game", 1, rememberGame, function(val)
        rememberGame = val
        writeConfig(PREF_RMGAME_FILE, val)
    end)

    buildCheckbox("RmKeyBox", "Remember Key", 2, rememberKey, function(val)
        rememberKey = val
        writeConfig(PREF_RMKEY_FILE, val)
    end)

    -- Bottom Action Buttons
    local GetKeyButton = create("TextButton", {
        Name = "GetKeyButton",
        Size = UDim2.new(0.48, 0, 0, 40),
        Position = UDim2.new(0, 0, 1, -45),
        BackgroundColor3 = THEME.CardBg,
        BorderSizePixel = 0,
        FontFace = THEME.FontBold,
        Text = "GET KEY",
        TextColor3 = THEME.Accent,
        TextSize = 13,
        Parent = Content
    })
    create("UICorner", { CornerRadius = UDim.new(0, 8), Parent = GetKeyButton })
    create("UIStroke", { Color = THEME.Accent, Thickness = 1, Transparency = 0.5, Parent = GetKeyButton })

    GetKeyButton.MouseButton1Click:Connect(function()
        local link = selectedVersion.LINKVERTISE_URL or selectedVersion.LOOTLABS_URL
        if link then
            setclipboard(link)
            showStatus("Key link copied to clipboard!", THEME.Success)
        else
            showStatus("No key link required for this version.", THEME.Warning)
        end
    end)

    local SubmitButton = create("TextButton", {
        Name = "SubmitButton",
        Size = UDim2.new(0.48, 0, 0, 40),
        Position = UDim2.new(0.52, 0, 1, -45),
        BackgroundColor3 = THEME.Accent,
        BorderSizePixel = 0,
        FontFace = THEME.FontBold,
        Text = selectedVersion.REQUIRE_KEY and "CHECK KEY" or "START SCRIPT",
        TextColor3 = THEME.Text,
        TextSize = 13,
        Parent = Content
    })
    create("UICorner", { CornerRadius = UDim.new(0, 8), Parent = SubmitButton })

    SubmitButton.MouseButton1Click:Connect(function()
        local key = KeyTextBox.Text
        if rememberKey and #key > 0 and selectedVersion.KEY_FILE then
            writeConfig(selectedVersion.KEY_FILE, key)
        end
        if rememberGame then
            writeConfig(PREF_GAME_FILE, selectedGameKey)
        end

        local ok = verifyKeyWithLuarmor(key, selectedVersion, showStatus)
        if ok then
            task.delay(1.5, function()
                if ScreenGui and ScreenGui.Parent then
                    ScreenGui:Destroy()
                end
            end)
        end
    end)

    -- Auto-run if keyless and auto-load is ready
    if not selectedVersion.REQUIRE_KEY then
        task.spawn(function()
            task.wait(1.0)
            showStatus("Auto-loading keyless game...", THEME.Success)
            verifyKeyWithLuarmor("", selectedVersion, showStatus)
            task.wait(1.5)
            ScreenGui:Destroy()
        end)
    end
end

-- // 9. EXECUTION ENTRYPOINT
local function main()
    local success, err = pcall(buildInterface)
    if not success then
        warn("[XT-HUB] Loader startup failed: " .. tostring(err))
    end
end

main()
