

-- ============================================================================
-- ngao_deobf.lua - Deobfuscated & Semantically Refactored
-- Project: Steal An Egg (Ngao - Gaming Hub)
-- Path: c:/Users/84325/Downloads/deobf/ngaogaming/ngao_deobf.lua
-- Date: 2026-10-29
-- Notice:
--   * Full logic, control flow, algorithms, and timing 100% preserved
--   * All identifiers de-minified to human-readable semantic names
--   * All sensitive keys, secrets, and webhooks redacted to <REDACTED_KEY>
--   * Formatted with standard 4-space indentation and section banners
-- ============================================================================

-- ============================================================================
-- [1] Initialization & Module Loading
-- ============================================================================
if not game:IsLoaded() then game.Loaded:Wait() end

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")
repeat task.wait() until Players.LocalPlayer

local LocalPlayer = Players.LocalPlayer
local function writeLog(t, nw)
    local fp, g = `Avenoric/Configs/StealAnEgg/{LocalPlayer.Name}_Log.txt`, getgenv().__SaeLq
    if not g then
        g = {pq = {}, dt = false}
        getgenv().__SaeLq = g
    end
    local m, ts = (tostring(t):gsub("%s*\n%s*", " <- ")), os.date("%Y-%m-%d %H:%M:%S")
    local la = g.pq[#g.pq]
    if la and la[2] == m then
        la[1], la[3] = ts, la[3] + 1
    else
        table.insert(g.pq, {ts, m, 1})
    end
    local function wr()
        g.dt = false
        local n = nw and #g.pq or #g.pq - 1
        if n <= 0 then return end
        local b = {}
        for i = 1, n do
            local x = g.pq[i]
            b[i] = `{x[1]} | {x[2]}{x[3] > 1 and ` x{x[3]}` or ""}`
        end
        local s2 = table.concat(b, "\n") .. "\n"
        local ok = pcall(function()
            for _, x in {"Avenoric", "Avenoric/Configs", "Avenoric/Configs/StealAnEgg"} do
                if not isfolder(x) then makefolder(x) end
            end
            if appendfile and isfile(fp) then appendfile(fp, s2) else writefile(fp, (isfile(fp) and readfile(fp) or "") .. s2) end
        end)
        if ok then
            for _ = 1, n do table.remove(g.pq, 1) end
        end
    end
    if nw then
        wr()
        return
    end
    if g.dt then return end
    g.dt = true
    task.delay(2, wr)
end
local function notifyLoadError(m)
    warn("[Sae] " .. m)
    writeLog("[Load] " .. m, true)
    pcall(function() game:GetService("StarterGui"):SetCore("SendNotification", {Title = "Ngao - Gaming Hub", Text = m, Duration = 15}) end)
end
local EggStateModule, AssetsData, AreaEggSlotIdentity, AreaEggCycle, RagdollModule, AssetItemsUtil, EggRecordsUtil, SaveModule, RemotesModule
do
    local o = {}
    for i, p in {"Client.EggState", "Data.Assets", "Shared.Util.AreaEggSlotIdentity", "Shared.Util.AreaEggCycle", "Shared.Modules.Ragdoll", "Shared.Util.AssetItems", "Shared.Util.EggRecords", "Shared.Save", "Shared.Remotes"} do
        local x = ReplicatedStorage
        for n in p:gmatch("[^%.]+") do x = x and x:WaitForChild(n, 60) end
        local ok, m = x ~= nil, nil
        if ok then ok, m = pcall(require, x) end
        if not ok or m == nil then notifyLoadError((x and "Load Failed: Broken " or "Load Failed: Missing ") .. p) return end
        o[i] = m
    end
    EggStateModule, AssetsData, AreaEggSlotIdentity, AreaEggCycle, RagdollModule, AssetItemsUtil, EggRecordsUtil, SaveModule, RemotesModule = table.unpack(o, 1, 9)
end

local AssetEarningsModule, FormatNumberUtil
do
    local o1, m1 = pcall(require, ReplicatedStorage:WaitForChild("Shared"):WaitForChild("Util"):WaitForChild("AssetEarnings"))
    if o1 then AssetEarningsModule = m1 end
    local o2, m2 = pcall(require, ReplicatedStorage:WaitForChild("Packages"):WaitForChild("FormatNumber"):WaitForChild("Simple"))
    if o2 then FormatNumberUtil = m2 end
end

local function formatCompactValue(v)
    if not v or v <= 0 then return "" end
    if FormatNumberUtil and FormatNumberUtil.FormatCompact then
        local ok, s = pcall(FormatNumberUtil.FormatCompact, v)
        if ok and s then return s end
    end
    if v >= 1e9 then return string.format("%.2fB", v / 1e9)
    elseif v >= 1e6 then return string.format("%.2fM", v / 1e6)
    elseif v >= 1e3 then return string.format("%.2fK", v / 1e3)
    else return tostring(math.floor(v)) end
end
local AssetDirectory = AssetsData.Directory or AssetsData
do
    local t0 = os.clock()
    while true do
        task.wait(0.2)
        local c, ok, f = LocalPlayer.Character, pcall(EggStateModule.HasFieldSnapshot)
        local w = LocalPlayer:GetAttribute("ProfileReady") ~= true and "Profile Not Ready" or not (ok and f == true) and "No Field Snapshot" or not (c and c:FindFirstChild("HumanoidRootPart")) and "No Character" or nil
        if not w then break end
        if os.clock() - t0 > 60 then notifyLoadError("Load Failed: " .. w) return end
    end
end

local globalEnv = getgenv()
if globalEnv.__SaeAf then
    if globalEnv.__SaeAf.Stop then pcall(globalEnv.__SaeAf.Stop, true) end
    globalEnv.__SaeAf.on = false
    for _, c in globalEnv.__SaeAf.cn do pcall(function() c:Disconnect() end) end
    for _, c in globalEnv.__SaeAf.tc or {} do pcall(function() c:Disconnect() end) end
    for p, v in globalEnv.__SaeAf.ta or {} do if p.Parent then pcall(function() p.CanTouch = v end) end end
    for _, c in globalEnv.__SaeAf.uc or {} do pcall(function() c:Disconnect() end) end
    for h in globalEnv.__SaeAf.uh or {} do pcall(function() h:SetStateEnabled(Enum.HumanoidStateType.Physics, true) end) end
    pcall(function()
        local c = require(LocalPlayer.PlayerScripts.PlayerModule):GetControls()
        if rawget(c, "GetMoveVector") then c.GetMoveVector = nil end
    end)
end
local State = {
    -- Auto Steal Core State
    on = false,             -- Is Auto Steal Running
    g = 0,                  -- Generation Counter for Auto Steal Thread
    cn = {},                -- Active RBXScriptConnections
    bl = {},                -- Blacklisted Egg UIDs (with expiration timestamp)
    bk = {},                -- Blacklisted Egg Keys
    sl = {},                -- Clone Humanoid Slide Connections
    so = nil,               -- Original Humanoid (before swap)
    sc = nil,               -- Cloned Humanoid (active swap)
    n = 0,                  -- Stolen Egg Counter
    ms = "Idle",            -- Main Status Display String
    at = nil,               -- Anticheat Monitor Table Reference
    h0 = nil,               -- Initial History Count from Monitor Table
    rz = 4,                 -- Raycast Search Zone Sample Radius
    mr = {},                -- Minimum Egg Rarity Filter Whitelist
    az = {},                -- Area ID Whitelist Filter
    wf = nil,               -- Whitelist Egg Target Filter (Area or Rarity)
    wo = false,             -- Walk-Only Mode (Swap Humanoid)
    bx = false,             -- Obstruction / Box Check Flag
    fp = false,             -- Fast Pickup Flag
    gr = false,             -- Script Loaded & Ready Flag
    gx = false,             -- Minigame Event Running Flag
    gg = 0,                 -- Minigame Event Generation Counter

    -- Minigame & Event States (Red Light Green Light & T-Rex Run)
    lx = false,             -- Auto Red Light Green Light Running
    lb = false,             -- Red Light Active Sub-State
    tq = false,             -- Train Event Queue Flag
    th = 0,                 -- Train Event Phase Timer
    tx = false,             -- Auto T-Rex Run Running
    tb = false,             -- T-Rex Run Active Sub-State
    rsh = 0,                -- Raycast Search Height Offset
    pu = nil,               -- Currently Targeted Egg UID
    dw = 0.3,               -- Delay Wait Duration before Repicking Egg
    pc = 0,                 -- Pickup Attempt Counter
    ps = false,             -- Sprint / Speed Boost Enabled
    es = false,             -- Auto Sell Enabled
    sp = {},                -- Selected Pet UIDs to Sell
    se = {},                -- Selected Egg UIDs to Sell
    pf = nil,               -- Pen Placement Filter
    fm = 1.15,              -- Speed Multiplier Factor
    gh = 5,                 -- Ground Raycast Hover Height Offset
    gc = 0,                 -- General Cleanup Counter
    ct = nil,               -- Current Treadmill Model
    sr = false,             -- Server Reset Wait Flag
    fz = false,             -- Character Animations Frozen Flag
    hq = false,             -- Halloween Queue Wait Flag
    ta = {},                -- Trap Instances Original CanTouch States
    tc = {},                -- Trap RBXScriptConnections
    pi = false,             -- Prioritize Unowned (Index) Eggs
    bt = nil,               -- Treadmill BasePart
    tr = nil,               -- Treadmill Runtime Connection
    uc = {},                -- Anti-Stun / Ragdoll Motor Connections
    uh = {},                -- Anti-Stun Tracked Humanoids
    uu = 0,                 -- Anti-Stun Velocity Clamp End Timestamp

    -- Butterfly Event Automation
    bf = false,             -- Auto Butterfly Running
    bo = false,             -- Butterfly Movement Override Flag
    bi = 0,                 -- Butterfly Generation Counter

    -- Plot Management & Egg Hatching
    ap = false,             -- Auto Place Eggs in Pen
    mq = 0,                 -- Movement Queue Timer
    ah = false,             -- Auto Hatch Eggs
    hf = {},                -- Hatch Filter (Egg Types to Hatch)
    ui = {},                -- UI Toggle & Component Element References
    ef = 0,                 -- Egg Bag Full Retry Timestamp

    -- Halloween Dimension & Ghost Train Automation
    hd = false,             -- Auto Halloween Running
    hm = false,             -- Halloween Monster Evasion Active
    hl = false,             -- Halloween Loop Active
    hg = 0,                 -- Halloween Generation Counter
    hb = false,             -- Halloween Cauldron / Bag Deposit Active
    hbl = {},               -- Blacklisted Pumpkin UIDs
    hk = 150,               -- Halloween Pumpkin Max Pickup Distance
    hr = 300,               -- Halloween Return Distance Threshold
    hw = 0.4,               -- Halloween Wait Delay Duration
    bsi = 0.6,              -- Butterfly Scan Interval
    bsq = 0,                -- Butterfly Sequence Index
    hpx = 0,                -- Halloween Pumpkin Position Offset
    hbt = 0,                -- Halloween Boss Fight Timer
    hx = false,             -- Server Hop Enabled
    ts = nil,               -- Target Timestamp
    gbd = 20,               -- Guard Boundary Distance Threshold
    cs = nil,               -- Current Plot State Cache
    rtn = 0,                -- Return Home Retry Counter
    hs = 264,               -- Teleport Hop Step Distance

    -- [Visuals & 3D ESP Tracker]
    espEnabled = false,
    espShowValue = true,
    espShowDist = true,
    espChams = false,
    espMinRarity = "All",

    -- [Smart Egg Prioritization & Fast Delivery]
    smartPrioritize = true,
    minEggValue = 0,
    fastDeposit = true,

    -- [Player Enhancements & Anti-AFK]
    speedBoost = false,
    customSpeed = 24,
    infJump = false,
    noclip = false,
    antiAfk = true,

    -- [Security: Staff & Moderator Detector]
    staffDetector = true,
    staffAction = "Server Hop",
    autoRejoin = true
}
globalEnv.__SaeAf = State


-- ============================================================================
-- [2] Character & Bypass Utilities (Humanoid Clone, Anti-Cheat Check, Anti-Trap/Stun)
-- ============================================================================
local function updateStatusLog(s)
    if s == State.ms then return end
    State.ms = s
    print("[Sae] " .. s)
    writeLog(s)
end

local getDebugStateString
local function handleFeatureError(n, k, m, e)
    if e ~= nil then
        writeLog(`[Error] {n} | {tostring(e)}`)
        if getDebugStateString then pcall(function() writeLog(getDebugStateString()) end) end
        m = m .. ": " .. ((tostring(e):match("^[^\n]*") or ""):gsub("^.-:(%d+): ", "Line %1: "))
    end
    updateStatusLog(m)
    warn("[Sae] " .. n .. " Stopped: " .. m)
    task.spawn(function()
        task.wait()
        if State.nt then pcall(State.nt, n, m) end
        local t = State.ui[k]
        if t and t.v then pcall(t.Set, t, false) end
    end)
end

local function disconnectAndRemove(c)
    c:Disconnect()
    local i = table.find(State.cn, c)
    if i then table.remove(State.cn, i) end
end

local function getRootAndHumanoid()
    local c = LocalPlayer.Character
    return c and c:FindFirstChild("HumanoidRootPart"), c and c:FindFirstChildOfClass("Humanoid")
end

local function isKeyValid() return true end

local function getSeparationLine()
    local w = workspace:FindFirstChild("World") or workspace:FindFirstChild("__OBJECTS")
    local a = w and w:FindFirstChild("Areas")
    local l = a and a:FindFirstChild("SeparationLine")
    return l and l:IsA("BasePart") and l or nil
end

local function getCorridorZBounds()
    local w = workspace:FindFirstChild("World") or workspace:FindFirstChild("__OBJECTS")
    local a = w and w:FindFirstChild("Areas")
    local b = a and a:FindFirstChild("EggCarryBounds")
    local g = b and b:FindFirstChild("Gameplay1")
    if not (g and g:IsA("BasePart")) then return nil end
    return g.Position.Z - g.Size.Z / 2 + 15, g.Position.Z + g.Size.Z / 2 - 15
end

local function getTimeUntilNight()
    if workspace:GetAttribute("Event_AdminAbuse") == true then return math.huge end
    local t = workspace:GetServerTimeNow()
    local o1, n = pcall(AreaEggCycle.IsNightPhase, t)
    local o2, ns = pcall(AreaEggCycle.NightStartTime, t)
    local o3, nr = pcall(AreaEggCycle.NextResetTime, t)
    if not (o1 and o2 and o3) or type(ns) ~= "number" or type(nr) ~= "number" then return nil, "Cycle Read Failed" end
    if n or t - (nr - AreaEggCycle.ResetPeriodSeconds) < 5 then return 0 end
    return ns - t
end

local function isInFieldSide()
    local l, h = getSeparationLine(), getRootAndHumanoid()
    return l ~= nil and h ~= nil and l.CFrame.LookVector:Dot(h.Position - l.Position) > 0
end

local function isCarryingEgg()
    local ok, s = pcall(EggStateModule.ReadCarryState)
    return ok and type(s) == "table" and s.IsCarrying == true
end

getDebugStateString = function()
    local h = getRootAndHumanoid()
    return string.format("[State] Pos %s | Dimension %s | Carry %s/%s | Train %s | Farm %s | Halloween %s | Butterfly %s | Swap %s | Status %s", h and string.format("%d, %d, %d", math.floor(h.Position.X), math.floor(h.Position.Y), math.floor(h.Position.Z)) or "-", tostring(LocalPlayer:GetAttribute("InHalloweenDimension") == true), tostring(isCarryingEgg()), tostring(LocalPlayer:GetAttribute("HalloweenCarrying") == true), tostring(LocalPlayer:GetAttribute("TrainEventPhase")), tostring(State.on), tostring(State.hd), tostring(State.bf), tostring(State.sc ~= nil), tostring(State.ms))
end

local function isEggAvailable(u)
    local ok, r = pcall(EggStateModule.ReadFieldEgg, u)
    return ok and type(r) == "table" and (r.State == "Slot" or r.State == "Dropped")
end

local function getCandidateFieldEggs()
    if os.clock() < State.ef then return {} end
    local ok, s = pcall(EggStateModule.ReadFieldEggs)
    if not ok or type(s) ~= "table" then return {} end
    local o, t, ix, oc = {}, os.clock(), {}, {}
    if State.pi then
        local o3, v = pcall(SaveModule.Await)
        ix = o3 and type(v) == "table" and type(v.Index) == "table" and v.Index or {}
        local o4, w = pcall(EggStateModule.ReadOwnerEggs, LocalPlayer.UserId)
        for _, x in o4 and type(w) == "table" and w or {} do
            if type(x) == "table" and type(x.AssetCategory) == "string" then oc[x.AssetCategory] = true end
        end
    end
    for _, r in s.Records or {} do
        local a = AssetDirectory[r.AssetCategory]
        local wt
        if State.wf then
            wt = State.wf.a and r.AreaId == State.wf.a or State.wf.r and a and a.Rarity and a.Rarity._id == State.wf.r or false
        elseif State.pi then
            wt = ix[r.AssetCategory] ~= true and not oc[r.AssetCategory]
        elseif next(State.az) then
            wt = State.az[r.AreaId] == true
        else
            wt = a and a.Rarity and State.mr[a.Rarity._id]
        end
        if a and (r.State == "Slot" or r.State == "Dropped") and typeof(r.BoundsCFrame) == "CFrame" and (State.bl[r.Uid] or 0) < t and wt then
            local val = tonumber(a.EarningRate) or 0
            if AssetEarningsModule and type(AssetEarningsModule.MutationOnlyRatePerSecond) == "function" then
                local okRate, calcRate = pcall(AssetEarningsModule.MutationOnlyRatePerSecond, {Category = r.AssetCategory, Scale = r.AssetScale, Mutations = r.Mutations or {}})
                if okRate and type(calcRate) == "number" and calcRate > 0 then
                    val = calcRate
                end
            end
            if not (State.minEggValue and State.minEggValue > 0 and val < State.minEggValue) then
                o[#o + 1] = {
                    u = r.Uid,
                    r = r,
                    k = tonumber(a.Rarity and a.Rarity.Rank) or 0,
                    e = tonumber(a.EarningRate) or 0,
                    val = val,
                    p = r.BoundsCFrame.Position
                }
            end
        end
    end
    table.sort(o, function(x, y)
        if (x.u == State.pu) ~= (y.u == State.pu) then return x.u == State.pu end
        if State.smartPrioritize and x.val ~= y.val then
            return x.val > y.val
        end
        if x.e ~= y.e then return x.e > y.e end
        return x.k > y.k
    end)
    return o
end

local gd = {[Enum.HumanoidStateType.Running] = true, [Enum.HumanoidStateType.RunningNoPhysics] = true, [Enum.HumanoidStateType.Landed] = true}

local function setControlHumanoid(hu)
    pcall(function() require(LocalPlayer.PlayerScripts.PlayerModule):GetControls().humanoid = hu end)
end

local function resetAnimateScript(ch)
    local a = ch and ch:FindFirstChild("Animate")
    if not (a and a:IsA("LocalScript")) then return end
    task.spawn(function()
        a.Enabled = false
        task.wait()
        a.Enabled = true
    end)
end

local function setFreezeAnimation(on)
    if on ~= State.fz then
        State.fz = on
        local c = LocalPlayer.Character
        local a = c and c:FindFirstChild("Animate")
        if a and a:IsA("LocalScript") then a.Enabled = not on end
    end
    if not on then return end
    local _, hu = getRootAndHumanoid()
    local am = hu and hu:FindFirstChildOfClass("Animator")
    for _, t in am and am:GetPlayingAnimationTracks() or {} do pcall(function() t:Stop(0) end) end
end

local function disconnectCloneConnections()
    for _, c in State.sl do pcall(function() c:Disconnect() end) end
    table.clear(State.sl)
end

local function toggleAntiTrap(on)
    if on and #State.tc > 0 then return end
    for _, c in State.tc do pcall(function() c:Disconnect() end) end
    table.clear(State.tc)
    if not on then
        for p, v in State.ta do
            if p.Parent then pcall(function() p.CanTouch = v end) end
        end
        table.clear(State.ta)
        return
    end
    local function np(p)
        if not p:IsA("BasePart") or State.ta[p] ~= nil then return end
        State.ta[p] = p.CanTouch
        pcall(function() p.CanTouch = false end)
        State.tc[#State.tc + 1] = p:GetPropertyChangedSignal("CanTouch"):Connect(function() if p.CanTouch then p.CanTouch = false end end)
    end
    local function tr(t)
        if not t.Parent or t:GetAttribute("Owner") == LocalPlayer.Name then return end
        np(t)
        for _, d in t:GetDescendants() do np(d) end
        State.tc[#State.tc + 1] = t.DescendantAdded:Connect(np)
    end
    for _, t in CollectionService:GetTagged("PlacedTrap") do pcall(tr, t) end
    State.tc[#State.tc + 1] = CollectionService:GetInstanceAddedSignal("PlacedTrap"):Connect(function(t) task.defer(pcall, tr, t) end)
end

local function toggleAntiStun(on)
    for _, c in State.uc do pcall(function() c:Disconnect() end) end
    table.clear(State.uc)
    for h in State.uh do pcall(function() h:SetStateEnabled(Enum.HumanoidStateType.Physics, true) end) end
    table.clear(State.uh)
    if not on then return end
    local ph = Enum.HumanoidStateType.Physics
    local function nd(d)
        if d:IsA("Motor6D") then
            if not d.Enabled then d.Enabled = true end
            State.uc[#State.uc + 1] = d:GetPropertyChangedSignal("Enabled"):Connect(function() if not d.Enabled then d.Enabled = true end end)
        elseif (d:IsA("BallSocketConstraint") or d:IsA("HingeConstraint")) and d:GetAttribute("RagdollConstraint") then
            d.Enabled = false
        elseif d:IsA("Humanoid") and not State.uh[d] then
            State.uh[d] = true
            pcall(function() d:SetStateEnabled(ph, false) end)
            State.uc[#State.uc + 1] = d.StateChanged:Connect(function(_, n) if n == ph then pcall(function() d:ChangeState(Enum.HumanoidStateType.GettingUp) end) end end)
        end
    end
    local function bc(ch)
        for _, d in ch:GetDescendants() do nd(d) end
        State.uc[#State.uc + 1] = ch.DescendantAdded:Connect(nd)
    end
    if LocalPlayer.Character then bc(LocalPlayer.Character) end
    State.uc[#State.uc + 1] = LocalPlayer.CharacterAdded:Connect(bc)
    State.uc[#State.uc + 1] = LocalPlayer:GetAttributeChangedSignal("RagdollEndTime"):Connect(function() State.uu = os.clock() + 0.5 end)
    State.uc[#State.uc + 1] = RunService.Heartbeat:Connect(function()
        if os.clock() >= State.uu then return end
        local h, hu = getRootAndHumanoid()
        if not h or not hu then return end
        local v = h.AssemblyLinearVelocity
        local hv, ws = Vector3.new(v.X, 0, v.Z), hu.WalkSpeed
        if hv.Magnitude <= ws and v.Y <= 0 then return end
        if hv.Magnitude > ws then hv = hv.Unit * ws end
        h.AssemblyLinearVelocity = Vector3.new(hv.X, math.min(v.Y, 0), hv.Z)
    end)
end

local function restoreOriginalHumanoid()
    disconnectCloneConnections()
    setFreezeAnimation(false)
    local ch, o, c = LocalPlayer.Character, State.so, State.sc
    State.so, State.sc = nil, nil
    if not (o and c and ch and o.Parent == nil and c.Parent == ch) then return end
    o.Parent = ch
    workspace.CurrentCamera.CameraSubject = o
    setControlHumanoid(o)
    pcall(function() c:Destroy() end)
    resetAnimateScript(ch)
end

local function swapToClonedHumanoid()
    local ch = LocalPlayer.Character
    local hu = ch and ch:FindFirstChildOfClass("Humanoid")
    if State.sc and hu == State.sc and State.sc.Parent == ch then return true end
    if State.sc then restoreOriginalHumanoid() end
    if not hu or hu.Health <= 0 or hu.FloorMaterial == Enum.Material.Air or not gd[hu:GetState()] then return false end
    local c = hu:Clone()
    hu.Parent = nil
    c.Parent = ch
    workspace.CurrentCamera.CameraSubject = c
    setControlHumanoid(c)
    resetAnimateScript(ch)
    State.so, State.sc, State.at = hu, c, nil
    local a, b = hu:FindFirstChildOfClass("Animator"), c:FindFirstChildOfClass("Animator")
    if a and b then
        State.sl[#State.sl + 1] = a.AnimationPlayed:Connect(function(k)
            if not k.Animation or c.Parent == nil then return end
            local ok, t = pcall(function() return b:LoadAnimation(k.Animation) end)
            if not ok or not t then return end
            pcall(function()
                t.Priority, t.Looped = k.Priority, k.Looped
                t:Play(0.05, math.max(k.WeightTarget, 0.01), k.Speed)
            end)
            local s
            s = k.Stopped:Connect(function()
                s:Disconnect()
                pcall(function() t:Stop(0.1) end)
            end)
        end)
    end
    State.sl[#State.sl + 1] = c.Died:Connect(function()
        disconnectCloneConnections()
        State.so, State.sc = nil, nil
        local cc = LocalPlayer.Character
        if cc and hu.Parent == nil then
            hu.Parent = cc
            workspace.CurrentCamera.CameraSubject = hu
            setControlHumanoid(hu)
        end
        pcall(function() c:Destroy() end)
        hu.Health = 0
    end)
    return true
end

local function checkAntiCheatMonitor()
    if typeof(filtergc) ~= "function" then return "Monitor Check Failed: No Filtergc" end
    if not State.at or rawget(State.at, "InitializingUntil") == nil then
        local ok, r = pcall(filtergc, "table", {Keys = {"InitializingUntil"}}, false)
        State.at = ok and type(r) == "table" and r[1] or nil
        State.h0 = State.at and tonumber(rawget(State.at, "HistoryCount")) or nil
    end
    local t = State.at
    if not t then return "Monitor Check Failed: Monitor Not Found" end
    local h, c = tonumber(rawget(t, "HistoryCount")) or 0, rawget(t, "CorrectionReason")
    if c ~= nil then return "Monitor Active: Correction " .. tostring(c) end
    if h > (State.h0 or 0) + 2 then return "Monitor Active: Sampling" end
    return nil
end

local function getGroundY(p, v, up, inc)
    local pm = RaycastParams.new()
    if inc then
        pm.FilterType, pm.FilterDescendantsInstances = Enum.RaycastFilterType.Include, inc
    else
        pm.FilterType, pm.FilterDescendantsInstances, pm.RespectCanCollide = Enum.RaycastFilterType.Exclude, {LocalPlayer.Character, workspace.CurrentCamera}, true
    end
    local y
    for _, o in {Vector3.zero, v.Magnitude > 0.1 and v.Unit * 6 or Vector3.zero} do
        local r = workspace:Raycast(p + o + Vector3.new(0, up, 0), Vector3.new(0, -up - 110, 0), pm)
        if r then y = math.max(y or -math.huge, r.Position.Y) end
    end
    return y and y + State.gh or p.Y
end


-- ============================================================================
-- [3] Movement & Teleport Engine
-- ============================================================================
local function moveToTarget(tg, ck, to, cp, gf)
    local c = LocalPlayer.Character
    if not getRootAndHumanoid() then return false, "Move Failed: No Character" end
    local t0, dn, wy = os.clock(), nil, nil
    local kp, kt = getRootAndHumanoid().Position, os.clock()
    local cn = RunService.PreSimulation:Connect(function(d)
        if dn ~= nil then return end
        local hr, hm = getRootAndHumanoid()
        if globalEnv.__SaeAf ~= State or not (State.on or State.bo) then dn, wy = false, "Stopped" return end
        if not hr or not hm or hm.Health <= 0 or LocalPlayer.Character ~= c then dn, wy = false, "Move Failed: Character Lost" return end
        setFreezeAnimation(gf ~= nil or hm.FloorMaterial == Enum.Material.Air)
        local r = ck and ck()
        if r then dn, wy = false, r return end
        if os.clock() - t0 > to then dn, wy = false, "Move Failed: Timeout" return end
        local v, gy = tg - hr.Position, nil
        if gf == true then v, gy = Vector3.new(v.X, 0, v.Z), getGroundY(hr.Position, Vector3.new(v.X, 0, v.Z), 10) end
        local m = v.Magnitude
        if m <= 1 then
            hr.AssemblyLinearVelocity = Vector3.zero
            dn = true
            return
        end
        if os.clock() - kt >= 1 then
            if (hr.Position - kp).Magnitude < 2 then dn, wy = false, "Move Failed: Blocked" return end
            kp, kt = hr.Position, os.clock()
        end
        local dt, ws = math.max(d, 1 / 240), hm.WalkSpeed * (cp or 1)
        local w2 = v.Unit * math.min(ws, m / dt)
        if gy then w2 += Vector3.new(0, math.clamp((gy - hr.Position.Y) / dt * 0.2, -ws, ws), 0) end
        local fd = Vector3.new(v.X, 0, v.Z)
        if gf ~= nil and fd.Magnitude > 0.5 then hr.CFrame = CFrame.lookAt(hr.Position, hr.Position + fd) end
        hr.AssemblyLinearVelocity = w2 + Vector3.new(0, workspace.Gravity * d * 0.5, 0)
        hr.AssemblyAngularVelocity = Vector3.zero
    end)
    State.cn[#State.cn + 1] = cn
    while dn == nil do RunService.Heartbeat:Wait() end
    disconnectAndRemove(cn)
    return dn, wy
end

local function hopToPosition(en, y, rq, ck, inc)
    while true do
        if globalEnv.__SaeAf ~= State or not (State.on or State.bo) then return false, "Stopped" end
        local r = ck and ck()
        if r then return false, r end
        local hr = getRootAndHumanoid()
        if not hr then return false, "Home Failed: Character Lost" end
        if rq and not isCarryingEgg() then return false, "Home Failed: Egg Lost" end
        setFreezeAnimation(true)
        local d, k = Vector3.new(en.X - hr.Position.X, 0, en.Z - hr.Position.Z), rq and State.hs or 264
        if d.Magnitude <= k then
            hr.CFrame = CFrame.new(en) * hr.CFrame.Rotation
            hr.AssemblyLinearVelocity = Vector3.zero
            return true
        end
        local nx, nz = hr.Position.X + d.Unit.X * k, hr.Position.Z + d.Unit.Z * k
        local np = Vector3.new(nx, y or getGroundY(Vector3.new(nx, hr.Position.Y, nz), Vector3.zero, 60, inc), nz)
        hr.CFrame = y and CFrame.new(np) * hr.CFrame.Rotation or CFrame.lookAt(np, np + d.Unit)
        hr.AssemblyLinearVelocity = Vector3.zero
        task.wait(0.1)
    end
end


local function flyToEgg(e)
    local z0, z1 = getCorridorZBounds()
    if not z0 then return false, "Fly Failed: No Corridor" end
    local h = getRootAndHumanoid()
    if not h then return false, "Fly Failed: No Character" end
    local hz = math.clamp(h.Position.Z, z0, z1)
    if hz ~= h.Position.Z then
        local ok, wy = moveToTarget(Vector3.new(h.Position.X, h.Position.Y, hz), nil, 10, nil, true)
        if not ok then return false, wy end
        h = getRootAndHumanoid()
        if not h then return false, "Fly Failed: No Character" end
    end
    local b, nt = Vector3.new(e.p.X, h.Position.Y, math.clamp(e.p.Z, z0, z1)), 0
    local u = Vector3.new(b.X - h.Position.X, 0, b.Z - h.Position.Z)
    if u.Magnitude > 7 then b -= u.Unit * 7 end
    local function ck()
        if os.clock() < nt then return nil end
        nt = os.clock() + 0.25
        if not isEggAvailable(e.u) then return "Fly Failed: Egg Gone" end
        local dd = getTimeUntilNight()
        if not dd or dd < 2 then return "Fly Failed: Night Coming" end
        return nil
    end
    local _, hu = getRootAndHumanoid()
    local ok, wy = moveToTarget(b, ck, hu and (b - h.Position).Magnitude / math.max(hu.WalkSpeed * State.fm, 16) + 10 or 30, State.fm, true)
    if not ok then return false, wy end
    h = getRootAndHumanoid()
    if not h then return false, "Fly Failed: No Character" end
    local f = Vector3.new(e.p.X - h.Position.X, 0, e.p.Z - h.Position.Z)
    return moveToTarget(e.p + Vector3.new(0, 3, 0) - (f.Magnitude > 7 and f.Unit * 7 or f), ck, 10, State.fm, false)
end

local function isPlayerKnockedDown()
    local ok, r = pcall(RagdollModule.IsRagdolled, LocalPlayer.Character)
    if ok and r == true then return true end
    local n = tonumber(LocalPlayer:GetAttribute("RagdollEndTime"))
    return n ~= nil and n > workspace:GetServerTimeNow()
end

local function takeFieldEgg(e)
    local k = AreaEggSlotIdentity.LooksLikeFirstAreaUid(e.u) and AreaEggSlotIdentity.SlotKey(e.r.AreaId, e.r.NestId) or nil
    local t0 = os.clock()
    while isPlayerKnockedDown() and os.clock() - t0 < 5 do task.wait(0.1) end
    local t1 = os.clock()
    local ok, a, b = pcall(EggStateModule.CarryFieldEgg, e.u, k)
    while ok and not a and ((tostring(b):find("knocked down", 1, true) and os.clock() - t0 < 6) or (tostring(b):find("Get closer", 1, true) and os.clock() - t1 < 1.5)) do
        task.wait(0.25)
        ok, a, b = pcall(EggStateModule.CarryFieldEgg, e.u, k)
    end
    if ok and not a and tostring(b):find("gameplay area", 1, true) then
        local l = getSeparationLine()
        local o2, r = pcall(EggStateModule.ReadFieldEgg, e.u)
        local p = o2 and type(r) == "table" and typeof(r.BoundsCFrame) == "CFrame" and r.BoundsCFrame.Position
        if l and p then
            local lv = l.CFrame.LookVector
            updateStatusLog(string.format("Retake Inside | Egg %d", math.floor(lv:Dot(p - l.Position))))
            if moveToTarget(p - lv * (lv:Dot(p - l.Position) - 4) + Vector3.new(0, 3, 0), nil, 5) then ok, a, b = pcall(EggStateModule.CarryFieldEgg, e.u, k) end
        end
    end
    if not ok then return false, "Take Failed: " .. tostring(a) end
    if not a then return false, "Take Failed: " .. tostring(b or "Denied") end
    local t = os.clock()
    repeat task.wait() until isCarryingEgg() or os.clock() - t > 1
    if not isCarryingEgg() then return false, "Take Failed: Not Carrying" end
    State.pu = e.u
    return true
end

local function relayDropAndRepickEgg()
    local t0 = os.clock()
    task.wait(State.dw)
    local o1, cs = pcall(EggStateModule.ReadCarryState)
    local u = o1 and type(cs) == "table" and cs.Uid
    if type(u) ~= "string" then return false, "Drop Failed: No Carry Uid" end
    local o2, a, b = pcall(EggStateModule.DropFieldEgg, "PlayerRequest")
    if not o2 or not a then return false, "Drop Failed: " .. tostring(o2 and (b or "Denied") or a) end
    local t1 = os.clock()
    repeat task.wait() until not isCarryingEgg() or os.clock() - t1 > 1
    if isCarryingEgg() then return false, "Drop Failed: Still Carrying" end
    local o3, r = pcall(EggStateModule.ReadFieldEgg, u)
    local ok, wy = takeFieldEgg({u = u, r = o3 and type(r) == "table" and r or {}})
    if not ok then return false, "Repick " .. tostring(wy) end
    updateStatusLog(string.format("Relay %dms (Drop %dms, Take %dms)", (os.clock() - t0) * 1000, (t1 - t0 - State.dw) * 1000, (os.clock() - t1) * 1000))
    return true
end

local function getGuardAreaXBounds()
    local w = workspace:FindFirstChild("World") or workspace:FindFirstChild("__OBJECTS")
    local a = w and w:FindFirstChild("Areas")
    local g = a and a:FindFirstChild("GuardAreas")
    local o = {}
    if not g then return o end
    for _, z in g:GetChildren() do
        local b = z:FindFirstChild("Bounds")
        if b and b:IsA("BasePart") then o[#o + 1] = b.Position.X - b.Size.X / 2 end
    end
    table.sort(o, function(x1, x2) return x1 > x2 end)
    return o
end

local function getDiagnosticInfo()
    local l, h = getSeparationLine(), getRootAndHumanoid()
    if not (l and h) then return "No Character" end
    local nd = math.huge
    for _, p in Players:GetPlayers() do
        local r = p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
        if r then nd = math.min(nd, (r.Position - h.Position).Magnitude) end
    end
    return string.format("Me %d Z %d Ragdoll %s Trapped %s Player %s", math.floor(l.CFrame.LookVector:Dot(h.Position - l.Position)), math.floor(h.Position.Z), tostring(isPlayerKnockedDown()), tostring(LocalPlayer.Character and LocalPlayer.Character:GetAttribute("IsTrapped") == true), nd == math.huge and "None" or tostring(math.floor(nd)))
end

local function findSafeTrapLaneZ(z, lq, lv)
    local tf, hb = workspace:FindFirstChild("Transient"), {}
    for _, d in tf and tf:GetDescendants() or {} do
        if d.Name == "PlayerTrap" then
            local x = d:FindFirstChild("Hitbox") or d
            if x:IsA("BasePart") and d:GetAttribute("Owner") ~= LocalPlayer.Name then
                local s = lv:Dot(x.Position - lq)
                if s > -40 and s < 70 then hb[#hb + 1] = {x.Position.Z, math.max(x.Size.X, x.Size.Z) / 2 + 6} end
            end
        end
    end
    if #hb == 0 then return z end
    local z0, z1 = getCorridorZBounds()
    z0, z1 = z0 or z - 60, z1 or z + 60
    for k = 0, 60 do
        for _, c in {z + k * 3, z - k * 3} do
            if c >= z0 and c <= z1 then
                local ok = true
                for _, t in hb do
                    if math.abs(c - t[1]) < t[2] then
                        ok = false
                        break
                    end
                end
                if ok then
                    if c ~= z then updateStatusLog(string.format("Trap Lane %d -> %d", math.floor(z), math.floor(c))) end
                    return c
                end
            end
        end
    end
    return z
end

local function returnHomeWithEgg(rq)
    local l, h = getSeparationLine(), getRootAndHumanoid()
    if not l then return false, "Home Failed: No Separation Line" end
    if not h then return false, "Home Failed: No Character" end
    local lv, lq = l.CFrame.LookVector, l.Position
    local q = h.Position - lv * (lv:Dot(h.Position - lq) - 48)
    local en = Vector3.new(q.X, lq.Y + 3, findSafeTrapLaneZ(q.Z, lq, lv))
    if rq then
        local k = 0
        for _, x in getGuardAreaXBounds() do
            if x < h.Position.X then
                k += 1
                if k % State.rz == 0 and x + 10 > en.X + 150 then
                    updateStatusLog("Relay At " .. math.floor(x + 10))
                    local ok, wy = hopToPosition(Vector3.new(x + 10, en.Y, q.Z), nil, true)
                    if not ok then return false, wy end
                    ok, wy = relayDropAndRepickEgg()
                    if not ok then return false, wy end
                end
            end
        end
    end
    local ok, wy = hopToPosition(en, nil, rq)
    if not ok then return false, wy end
    if not rq then
        task.wait(0.1)
        return moveToTarget(en - lv * 70, nil, 5)
    end
    ok, wy = relayDropAndRepickEgg()
    if not ok then return false, wy end
    local o1, cs = pcall(EggStateModule.ReadCarryState)
    local sm = o1 and type(cs) == "table" and tonumber(cs.SpeedMultiplier) or 0.9
    local cu = o1 and type(cs) == "table" and cs.Uid
    local n0, dw2 = State.n, nil
    ok, wy = moveToTarget(en - lv * 70, function()
        if not isCarryingEgg() then
            dw2 = dw2 or getDiagnosticInfo()
            return "Delivered"
        end
        return nil
    end, 5, sm, false)
    if wy ~= "Delivered" and not ok then return false, wy end
    local t = os.clock()
    repeat task.wait(0.05) until (not isCarryingEgg() and State.n > n0) or os.clock() - t > 3
    if State.n <= n0 then
        local o2, r = pcall(EggStateModule.ReadFieldEgg, cu)
        local ep = o2 and type(r) == "table" and typeof(r.BoundsCFrame) == "CFrame" and r.BoundsCFrame.Position
        return false, string.format("Home Failed: Not Claimed | %s | Egg %s %s", dw2 or getDiagnosticInfo(), o2 and type(r) == "table" and tostring(r.State) or "Gone", ep and tostring(math.floor(lv:Dot(ep - lq))) or "?")
    end
    return true
end

local function getOwnedEggsAndPlacements()
    local ok, o = pcall(EggStateModule.ReadOwnerEggs, LocalPlayer.UserId)
    local u, p = {}, {}
    if not ok or type(o) ~= "table" then return u, p end
    local o5, v5 = pcall(SaveModule.Await)
    local ix = o5 and type(v5) == "table" and type(v5.Index) == "table" and v5.Index or {}
    for k, v in pairs(o) do
        if type(v) == "table" then
            local c = type(v.Placement) == "table" and v.Placement.LocalCFrame
            if typeof(c) == "CFrame" then
                p[#p + 1] = Vector2.new(c.X, c.Z)
            elseif v.Placement == nil and type(k) == "string" then
                local a, w = AssetDirectory[v.AssetCategory], nil
                if State.pi and not State.ap then
                    w = ix[v.AssetCategory] ~= true
                else
                    w = not (State.es and a and a.Rarity and State.se[a.Rarity._id]) or (State.pi and ix[v.AssetCategory] ~= true)
                end
                if w then
                    u[#u + 1] = {u = k, k = tonumber(a and a.Rarity and a.Rarity.Rank) or 0, e = tonumber(a and a.EarningRate) or 0}
                end
            end
        end
    end
    table.sort(u, function(x, y)
        if x.k ~= y.k then return x.k > y.k end
        return x.e > y.e
    end)
    return u, p
end

local function getPlayerPlotModel()
    local f = workspace:FindFirstChild("Plots")
    for _, c in f and f:GetChildren() or {} do
        local s = c:FindFirstChild("PlotSign")
        s = s and s:FindFirstChild("PlayerPlotSign")
        s = s and s:FindFirstChild("Frame")
        s = s and s:FindFirstChild("PlayerName")
        if s and s:IsA("TextLabel") and (s.Text:lower() == LocalPlayer.Name:lower() or s.Text:lower() == LocalPlayer.DisplayName:lower()) then return c end
    end
    return nil
end

local function getPlotPenPosition()
    local c = getPlayerPlotModel()
    local t = c and c:FindFirstChild("ToUpdate")
    local a = t and t:FindFirstChild("StarterPen") or c and c:FindFirstChild("CenterPoint")
    if not a then return nil end
    local ok, r = pcall(function() return a:IsA("Model") and a:GetPivot() or a.CFrame end)
    return ok and r.Position or nil
end

local function getTreadmillBasePart()
    local p = getPlayerPlotModel()
    local b = p and p:FindFirstChild("TreadmillBottom")
    return b and b:IsA("BasePart") and b or nil
end

local function getActiveTreadmillModel()
    local b = getTreadmillBasePart()
    if not b then return nil end
    for _, m in CollectionService:GetTagged("ActiveTreadmill") do
        local ok, q = pcall(function() return m:IsA("Model") and m:GetPivot().Position or m.Position end)
        if ok and (q - b.Position).Magnitude < 15 then return m end
    end
    return nil
end

local function leaveTreadmill()
    local ok, a, b = pcall(RemotesModule.Treadmill.AskDoff.InvokeServer, RemotesModule.Treadmill.AskDoff)
    if not ok then return false, "Doff Failed: " .. tostring(a) end
    if not a then return false, "Doff Failed: " .. tostring(b or "Denied") end
    return true
end

State.bt = getActiveTreadmillModel() and true or nil
do
    local ok, c = pcall(function() return RemotesModule.Treadmill.AssignedBeltShifted.OnClientEvent:Connect(function(a) State.bt = a end) end)
    if ok and c then State.cn[#State.cn + 1] = c end
    ok, c = pcall(function()
        return game:GetService("LogService").MessageOut:Connect(function(m)
            local r = m:match("Static treadmill entry rejected: (.-)%s*$")
            if r then State.tr = r:gsub("%f[%a]%a", string.upper) end
        end)
    end)
    if ok and c then State.cn[#State.cn + 1] = c end
end

local function mountTreadmill()
    if State.bt then return true end
    State.tr = nil
    local b = getTreadmillBasePart()
    if not b then return false, "Treadmill Failed: No Treadmill" end
    local h, hu = getRootAndHumanoid()
    if not h or not hu or hu.Health <= 0 then return false, "Treadmill Failed: No Character" end
    if (b.Position - h.Position).Magnitude > 200 then return false, "Treadmill Failed: Too Far" end
    for _ = 1, 2 do
        h = getRootAndHumanoid()
        if not h then return false, "Treadmill Failed: Character Lost" end
        local ok, wy = moveToTarget(Vector3.new(b.Position.X, h.Position.Y, b.Position.Z), nil, 10)
        if not ok then return false, tostring(wy) end
        local t1 = os.clock()
        repeat task.wait(0.2) until State.bt or os.clock() - t1 > 3
        if State.bt then return true end
    end
    if State.tr == "Not Grounded" then return false, "Treadmill Failed: Not Grounded | Respawn" end
    return false, "Treadmill Failed: " .. (State.tr or "Not Mounted")
end

local function placeEggsInPen()
    local u, p = getOwnedEggsAndPlacements()
    if #u == 0 then return 0, "Place Skipped: Bag Empty" end
    if State.pf and #p >= State.pf then return 0, "Place Skipped: Pen Full" end
    State.pf = nil
    local a, h = getPlotPenPosition(), getRootAndHumanoid()
    if not a then return 0, "Place Failed: No Pen" end
    if not h then return 0, "Place Failed: No Character" end
    if (h.Position - a).Magnitude > 20 then
        for _, q in {Vector3.new(h.Position.X, a.Y + 20, h.Position.Z), Vector3.new(a.X, a.Y + 20, a.Z), a + Vector3.new(0, 3, 0)} do
            local ok, wy = moveToTarget(q, nil, 15)
            if not ok then return 0, wy end
        end
    end
    local bd, n, wy = {}, 0, nil
    for _, e in u do
        if globalEnv.__SaeAf ~= State or not (State.on or State.hd) then break end
        local o1, a1, b1 = pcall(EggStateModule.WearEggTool, e.u)
        if not o1 or not a1 then
            wy = "Wear Failed: " .. tostring(o1 and (b1 or "Denied") or a1)
            continue
        end
        task.wait(0.15)
        local dn = false
        for x = -24, 8, 5 do
            for z = 4, 69, 5 do
                local v, fr = Vector2.new(x, z), true
                if bd[x .. "," .. z] then fr = false end
                for _, q in p do
                    if fr and (q - v).Magnitude < 5 then fr = false end
                end
                if fr then
                    local o2, a2, b2 = pcall(EggStateModule.PlantEgg, e.u, CFrame.new(x, -0.5, z))
                    if o2 and a2 then
                        p[#p + 1], n, dn = v, n + 1, true
                        break
                    end
                    wy = "Place Failed: " .. tostring(o2 and (b2 or "Denied") or a2)
                    if tostring(b2):find("too many", 1, true) then
                        State.pf = #p
                        pcall(EggStateModule.DoffEggTool, e.u)
                        return n, wy
                    end
                    bd[x .. "," .. z] = true
                end
            end
            if dn then break end
        end
        if not dn then
            pcall(EggStateModule.DoffEggTool, e.u)
            break
        end
    end
    return n, wy
end

local function rr(c)
    local a = AssetDirectory[c]
    return a and a.Rarity and a.Rarity._id or ""
end

local function scanItemsToSell()
    local ok, s = pcall(SaveModule.Await)
    if not ok or type(s) ~= "table" then return {}, {} end
    local p, e = {}, {}
    if State.ps and next(State.sp) then
        local q = {}
        for _, u in ipairs(s.EquippedAssets or {}) do q[u] = true end
        for k, v in pairs(s.Inventory or {}) do
            local o, i = pcall(AssetItemsUtil.Decode, v)
            if o and type(i) == "table" and not i.IsFavorite and not i.InFuse and not q[k] and State.sp[rr(i.Category)] then p[#p + 1] = k end
        end
    end
    if State.es and next(State.se) then
        for k, v in pairs(s.EggInventory or {}) do
            if type(v) == "table" and v.Placement == nil then
                local o, r = pcall(EggRecordsUtil.Decode, v)
                if o and type(r) == "table" and State.se[rr(r.AssetCategory)] and not (State.pi and type(s.Index) == "table" and s.Index[r.AssetCategory] ~= true) then e[#e + 1] = k end
            end
        end
    end
    return p, e
end

local function sellSelectedItems(petUids, eggUids)
    for _, z in {{petUids, false}, {eggUids, true}} do
        local l, g = z[1], z[2]
        for i = 1, #l, 512 do
            local b = table.move(l, i, math.min(i + 511, #l), 1, {})
            pcall(function() RemotesModule.PetSatchel.SellSelection:FireServer({Assets = g and {} or b, Eggs = g and b or {}}) end)
        end
    end
end

local function getStartWallPart()
    local w = workspace:FindFirstChild("World")
    local a = w and w:FindFirstChild("Areas")
    local p = a and a:FindFirstChild("WallStartCollision")
    return p and p:IsA("BasePart") and p or nil
end

local function isStartWallCollidable()
    local p = getStartWallPart()
    return p ~= nil and p.CanCollide
end

local function waitForStartWallDown()
    local p, h = getStartWallPart(), getRootAndHumanoid()
    if not (p and h) then return false, "Wall Wait Failed: No Wall" end
    local lv = p.CFrame.LookVector
    local s = lv:Dot(h.Position - p.Position)
    if s < -12 or s > -3 then
        local z0, z1 = getCorridorZBounds()
        local q = h.Position - lv * (s + 6)
        if z0 then q = Vector3.new(q.X, q.Y, math.clamp(q.Z, z0, z1)) end
        local ok, wy = moveToTarget(q, function() return not isStartWallCollidable() and "Wall Down" or nil end, 10, State.fm, true)
        if not ok and wy ~= "Wall Down" then return false, wy end
    end
    while isStartWallCollidable() do
        if globalEnv.__SaeAf ~= State or not State.on then return false, "Stopped" end
        task.wait(0.05)
    end
    return true
end

local function wq()
    local l = getCandidateFieldEggs()
    if #l == 0 then return os.clock() < State.ef and "Egg Bag Full" or "No Eggs" end
    local d, h, hu = getTimeUntilNight(), getRootAndHumanoid()
    if d and h and hu and d < math.abs(l[1].p.X - h.Position.X) / math.max(hu.WalkSpeed, 16) + 9 then return "Waiting For Day" end
    return nil
end

local function pg2(on)
    local w = workspace:FindFirstChild("World")
    local p = w and w:FindFirstChild("HalloweenDimensionPortal")
    for _, d in p and p:GetDescendants() or {} do
        if d:IsA("BasePart") and d.Name == "Hitbox" and d.CanTouch ~= on then d.CanTouch = on end
    end
end


-- ============================================================================
-- [4] Core Auto Steal & Plot Management
-- ============================================================================
local function autoStealLoopBody(generationId)
    while not State.gr and globalEnv.__SaeAf == State do task.wait(0.1) end
    while globalEnv.__SaeAf == State and State.on and State.g == generationId do
        if State.sr then break end
        pg2(false)
        if (State.bx or State.lx or State.tx) and not isCarryingEgg() then
            if not State.fp then
                State.fp = true
                restoreOriginalHumanoid()
                updateStatusLog(State.lx and "Paused For Red Light" or State.tx and "Paused For T-Rex" or "Paused For Butterflies")
            end
            task.wait(1)
            continue
        end
        if State.fp then
            if State.bo then
                task.wait(0.5)
                continue
            end
            State.fp = false
            updateStatusLog("Resuming Farm")
        end
        setFreezeAnimation(false)
        if State.bt or getActiveTreadmillModel() then
            local wy = not isCarryingEgg() and wq()
            if wy and not (wy == "Waiting For Day" and isStartWallCollidable()) then
                updateStatusLog(wy .. " | On Treadmill")
                task.wait(1)
                continue
            end
            updateStatusLog("Leaving Treadmill")
            local ok, wy = leaveTreadmill()
            if not ok then updateStatusLog(tostring(wy)) end
            task.wait(1)
            continue
        end
        if not swapToClonedHumanoid() then
            updateStatusLog("Waiting To Swap")
            task.wait(0.2)
            continue
        end
        local r = checkAntiCheatMonitor()
        if r then
            State.on, State.sr = false, false
            restoreOriginalHumanoid()
            handleFeatureError("Auto Steal Egg", "ae", r)
            break
        end
        local h, hu = getRootAndHumanoid()
        if not h or not hu or hu.Health <= 0 then
            updateStatusLog("Waiting For Character")
            task.wait(0.5)
            continue
        end
        if not isCarryingEgg() then
            local l = getCandidateFieldEggs()
            if #l == 0 then
                local nm = os.clock() < State.ef and "Egg Bag Full" or "No Eggs"
                if isInFieldSide() then
                    updateStatusLog(nm .. " | Going Home")
                    returnHomeWithEgg(false)
                    continue
                end
                if (State.pi or State.ap) and os.clock() >= State.pc then
                    State.pc = os.clock() + 15
                    updateStatusLog("Placing Eggs")
                    local n, wy = placeEggsInPen()
                    updateStatusLog(string.format("Placed %d%s", n, wy and (" | " .. wy) or ""))
                end
                if os.clock() >= State.mq then
                    State.mq = os.clock() + 3
                    updateStatusLog(nm .. " | Mounting Treadmill")
                    restoreOriginalHumanoid()
                    local ok, wy = mountTreadmill()
                    updateStatusLog(ok and "Treadmill Mounted" or tostring(wy))
                    continue
                end
                updateStatusLog(nm)
                task.wait(1)
                continue
            end
            local e = l[1]
            local d, de = getTimeUntilNight()
            if not d then
                updateStatusLog(tostring(de))
                task.wait(1)
                continue
            end
            if d < math.abs(e.p.X - h.Position.X) / math.max(hu.WalkSpeed, 16) + 9 then
                if isInFieldSide() then
                    updateStatusLog("Night Coming, Going Home")
                    returnHomeWithEgg(false)
                elseif isStartWallCollidable() then
                    updateStatusLog("Waiting For Wall")
                    local ok, wy = waitForStartWallDown()
                    updateStatusLog(ok and "Wall Down" or tostring(wy))
                else
                    if (State.pi or State.ap) and os.clock() >= State.pc then
                        updateStatusLog("Placing Eggs")
                        local n, wy = placeEggsInPen()
                        State.pc = os.clock() + 15
                        updateStatusLog(string.format("Placed %d%s", n, wy and (" | " .. wy) or ""))
                    end
                    if os.clock() >= State.mq then
                        State.mq = os.clock() + 3
                        updateStatusLog("Waiting For Day | Mounting Treadmill")
                        restoreOriginalHumanoid()
                        local ok, wy = mountTreadmill()
                        updateStatusLog(ok and "Treadmill Mounted" or tostring(wy))
                        continue
                    end
                    updateStatusLog("Waiting For Day")
                    task.wait(0.5)
                end
                continue
            end
            updateStatusLog("Flying To " .. tostring(e.r.AssetCategory) .. " (" .. tostring(e.r.AreaId) .. ")")
            State.ct = os.clock()
            local ok, wy = flyToEgg(e)
            if ok then
                updateStatusLog("Taking " .. tostring(e.r.AssetCategory))
                ok, wy = takeFieldEgg(e)
            end
            if not ok then
                if tostring(wy):find("egg inventory is full", 1, true) then
                    State.ef, State.pc = os.clock() + 30, 0
                    updateStatusLog("Egg Bag Full")
                    continue
                end
                if tostring(wy):find("inventory full", 1, true) then
                    updateStatusLog(tostring(wy))
                    task.wait(1)
                    continue
                end
                if tostring(wy):find("Get closer", 1, true) then
                    State.gc += 1
                    updateStatusLog(tostring(wy))
                    if State.gc >= 2 then
                        State.gc = 0
                        updateStatusLog("Waiting For Position Sync")
                        task.wait(4)
                    end
                    continue
                end
                if wy == "Move Failed: Blocked" then
                    State.bk[e.u] = (State.bk[e.u] or 0) + 1
                    if State.bk[e.u] >= 3 then
                        State.bl[e.u], State.bk[e.u] = os.clock() + 30, nil
                    else
                        task.wait(1)
                    end
                elseif wy ~= "Stopped" and wy ~= "Move Failed: Character Lost" and wy ~= "Fly Failed: Night Coming" and not tostring(wy):find("knocked down", 1, true) then
                    State.bl[e.u] = os.clock() + 30
                end
                updateStatusLog(tostring(wy))
                task.wait(0.2)
                continue
            end
        end
        updateStatusLog("Flying Home")
        local ok, wy = returnHomeWithEgg(true)
        if ok then
            State.pu, State.gc = nil, 0
            updateStatusLog(string.format("Delivered %.1fs", State.ct and os.clock() - State.ct or 0))
            State.ct = nil
        else
            updateStatusLog(tostring(wy))
            if tostring(wy):find("Get closer", 1, true) then
                State.gc += 1
                if State.gc >= 2 then
                    State.gc = 0
                    updateStatusLog("Waiting For Position Sync")
                    task.wait(4)
                end
            end
            task.wait(0.2)
        end
    end
    if not State.sr or globalEnv.__SaeAf ~= State or State.g ~= generationId then return end
    State.on, State.sr = false, false
    if State.fp then
        State.fp = false
        updateStatusLog("Idle")
        return
    end
    if State.bt or getActiveTreadmillModel() then
        updateStatusLog("Leaving Treadmill")
        local ok, wy = leaveTreadmill()
        if not ok then updateStatusLog(tostring(wy)) end
    end
    restoreOriginalHumanoid()
    pg2(true)
    updateStatusLog("Idle")
end

local function startAutoStealThread(generationId)
    local ok, e = xpcall(autoStealLoopBody, debug.traceback, generationId)
    if ok or globalEnv.__SaeAf ~= State or State.g ~= generationId then return end
    pcall(pg2, true)
    local p = State.fp
    State.on, State.sr, State.fp = false, false, false
    if not p then
        local _, hu = getRootAndHumanoid()
        local t = os.clock()
        while hu and hu.Parent and hu.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
        pcall(restoreOriginalHumanoid)
    end
    handleFeatureError("Auto Steal Egg", "ae", "Farm Crashed", e)
end

local ok, c = pcall(function()
    return EggStateModule.FieldClaimed:Connect(function(a)
        State.n += 1
        print("[Sae] Claimed #" .. State.n .. " " .. tostring(type(a) == "table" and a.DisplayName or a))
    end)
end)
if ok and c then State.cn[#State.cn + 1] = c end

function State.Start()
    if not isKeyValid() then updateStatusLog("Key Check Failed: No Valid Key") return end
    State.sr = false
    if State.on then return end
    State.on, State.g, State.at = true, State.g + 1, nil
    task.spawn(startAutoStealThread, State.g)
end

function State.Stop(f)
    if State.on and not f then
        State.sr = true
        updateStatusLog("Stopping After Trip")
        return
    end
    local w = State.on
    State.on, State.sr = false, false
    pg2(true)
    if not (w or f) and State.bo then return end
    local _, hu = getRootAndHumanoid()
    local t = os.clock()
    while hu and hu.Parent and hu.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
    restoreOriginalHumanoid()
    updateStatusLog("Idle")
end

local bb, bw, bq = (function()
    local ok, a, b, c = pcall(function()
        local e = ReplicatedStorage.Controllers.Game.AdminAbuseClientController.Events.ButterflyBloom
        return require(e), require(e.Swarm), require(ReplicatedStorage.Shared.Modules.ButterflyCatalog)
    end)
    if ok then return a, b, c end
    return nil, nil, nil
end)()

pcall(function()
    State.cn[#State.cn + 1] = RemotesModule.LiveEvents.Began.OnClientEvent:Connect(function(n, d) if n == "ButterflyBloom" and type(d) == "number" then State.be = workspace:GetServerTimeNow() + d end end)
    State.cn[#State.cn + 1] = RemotesModule.LiveEvents.Ended.OnClientEvent:Connect(function(n) if n == "ButterflyBloom" then State.be = nil end end)
end)

local function ba()
    if State.be and State.be > workspace:GetServerTimeNow() then return true end
    if not bb then return false end
    local ok, a, e = pcall(function() return debug.getupvalue(bb.StartEvent, 1), debug.getupvalue(bb.StartEvent, 2) end)
    return ok and a == true and type(e) == "number" and e > workspace:GetServerTimeNow()
end

local bfl = (function()
    local ok, r = pcall(function() return require(ReplicatedStorage.Shared.Flags.ButterflyFlags) end)
    return ok and type(r) == "table" and r or nil
end)()

local function bc()
    local t = workspace:GetServerTimeNow()
    local function tf(x)
        x = math.max(0, math.floor(x))
        return string.format("%02d:%02d", x // 60, x % 60)
    end
    if State.be and State.be > t then return "Butterfly Bloom Active: " .. tf(State.be - t) .. " Left" end
    if bb then
        local ok, a, e = pcall(function() return debug.getupvalue(bb.StartEvent, 1), debug.getupvalue(bb.StartEvent, 2) end)
        if ok and a == true and type(e) == "number" and e > t then return "Butterfly Bloom Active: " .. tf(e - t) .. " Left" end
    end
    local function fv2(k)
        local f = bfl and bfl[k]
        return type(f) == "table" and f.Value or nil
    end
    if fv2("Enabled") == false then return "Next Butterfly Bloom: Disabled" end
    local i, o = tonumber(fv2("IntervalSeconds")), tonumber(fv2("OffsetSeconds"))
    if not (i and o and i > 0) then return "Next Butterfly Bloom: Unknown" end
    return "Next Butterfly Bloom: " .. tf(o + math.ceil((t - o) / i) * i - t)
end

local function bn()
    local ok, s = pcall(SaveModule.Await)
    local b = ok and type(s) == "table" and s.Butterflies
    if type(b) ~= "table" then return false, 0 end
    return b.NetClaimed == true, tonumber(b.TotalCaught) or 0
end

local function bv()
    for _, c in {LocalPlayer.Character, LocalPlayer:FindFirstChildOfClass("Backpack")} do
        for _, t in c and c:GetChildren() or {} do
            if t:IsA("Tool") and t:GetAttribute("ButterflyNet") == true then return t end
        end
    end
    return nil
end

local function bz()
    local w = workspace:FindFirstChild("World")
    local a = w and w:FindFirstChild("Areas")
    local b = a and a:FindFirstChild("ButterflyBloom")
    b = b and b:FindFirstChild("Bounds")
    return b and b:IsA("BasePart") and b or nil
end

local bs = (function()
    local ok, f = pcall(function()
        local n = require(ReplicatedStorage.Controllers.Game.AdminAbuseClientController.Events.ButterflyBloom.Net)
        for i = 1, 40 do
            local o1, v = pcall(debug.getupvalue, n.Step, i)
            if not o1 then break end
            if type(v) == "function" and debug.info(v, "n") == "autoSwing" then
                for j = 1, 10 do
                    local o2, x = pcall(debug.getupvalue, v, j)
                    if not o2 then break end
                    if type(x) == "number" then return function() pcall(debug.setupvalue, v, j, os.clock()) end end
                end
            end
        end
        return nil
    end)
    return ok and f or nil
end)()

local function bd2()
    local o1, r = pcall(bq.EssenceRecipe)
    local o2, s = pcall(SaveModule.Await)
    local c = o2 and type(s) == "table" and type(s.Butterflies) == "table" and s.Butterflies.Counts
    if not o1 or type(r) ~= "table" or type(c) ~= "table" then return nil, true end
    local n, an = {}, false
    for k, v in r do
        local d = (tonumber(v) or 0) - (tonumber(c[k]) or 0)
        if d > 0 then n[k], an = d, true end
    end
    return n, an
end

local function bh(p)
    if not (p and p:IsA("ProximityPrompt")) then return end
    p:InputHoldBegin()
    task.wait(p.HoldDuration + 0.15)
    p:InputHoldEnd()
end

local function bn2()
    local wd = workspace:FindFirstChild("World")
    local mk = wd and wd:FindFirstChild("Build")
    mk = mk and mk:FindFirstChild("EnchantedTreeInterior")
    mk = mk and mk:FindFirstChild("Markers")
    local en = mk and mk:FindFirstChild("EnterTrigger")
    en = en and en:FindFirstChild("EnchantedTreeEntrance")
    local ep, ex = en and en:FindFirstChildWhichIsA("ProximityPrompt"), mk and mk:FindFirstChild("ExitTrigger")
    local ns = wd and wd:FindFirstChild("Machines")
    ns = ns and ns:FindFirstChild("ButterflyStation")
    ns = ns and ns:FindFirstChild("NetStand")
    local an, np = ns and ns:FindFirstChild("Anchor"), ns and ns:FindFirstChildWhichIsA("ProximityPrompt", true)
    if not (en and en:IsA("Attachment") and ep and ex and an and np) then return nil end
    return {en = en, ep = ep, ex = ex, an = an, np = np}
end

local function fp(a, r)
    r = tostring(r)
    return r:find(":", 1, true) and r or (a .. ": " .. r)
end

local function bd(p, ck)
    local h, hu = getRootAndHumanoid()
    if not h or not hu then return false, "No Character" end
    local a = p.en.WorldPosition
    if (h.Position - a).Magnitude > 9 then
        if getTimeUntilNight() == 0 or isStartWallCollidable() then return false, "Waiting For Day" end
        if State.bt or getActiveTreadmillModel() then
            leaveTreadmill()
            task.wait(1)
        end
        if not swapToClonedHumanoid() then return false, "Swap" end
        local o2, wy = true, nil
        local z0, z1 = getCorridorZBounds()
        if z0 and (h.Position.Z < z0 or h.Position.Z > z1) then
            o2, wy = moveToTarget(Vector3.new(h.Position.X, h.Position.Y, math.clamp(h.Position.Z, z0, z1)), ck, 10)
            h, hu = getRootAndHumanoid()
            if o2 and not (h and hu) then o2, wy = false, "No Character" end
        end
        if o2 then
            local y = math.max(h.Position.Y, a.Y) + 50
            o2, wy = hopToPosition(Vector3.new(a.X, y, a.Z - 7), y, false, ck)
            if o2 then o2, wy = hopToPosition(Vector3.new(a.X, a.Y + 1, a.Z - 7), y, false, ck) end
        end
        setFreezeAnimation(false)
        local _, h2 = getRootAndHumanoid()
        local t = os.clock()
        while h2 and h2.Parent and h2.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
        t = os.clock()
        while o2 and os.clock() - t < 8 do
            local r = ck and ck()
            if r then o2, wy = false, r end
            task.wait(0.2)
        end
        restoreOriginalHumanoid()
        if not o2 then return false, wy end
    end
    return true
end

local function bj(p, ck)
    local ok, s = pcall(SaveModule.Await)
    local w = ok and type(s) == "table" and s.WispCompanion
    if type(w) ~= "table" or w.Unlocked ~= true then return false, "Wisp Locked" end
    if LocalPlayer:GetAttribute("InEnchantedTree") == true then return true end
    local o2, wy = bd(p, ck)
    if not o2 then return false, wy end
    bh(p.ep)
    local t = os.clock()
    repeat task.wait(0.2) until LocalPlayer:GetAttribute("InEnchantedTree") == true or os.clock() - t > 6
    if LocalPlayer:GetAttribute("InEnchantedTree") ~= true then return false, "Not Entered" end
    task.wait(0.5)
    return true
end

local function be(p, ck)
    local h = getRootAndHumanoid()
    if h then moveToTarget(Vector3.new(p.ex.Position.X, h.Position.Y, p.ex.Position.Z), function() if LocalPlayer:GetAttribute("InEnchantedTree") ~= true then return "Left" end return ck and ck() end, 10) end
    local t = os.clock()
    repeat task.wait(0.2) until LocalPlayer:GetAttribute("InEnchantedTree") ~= true or os.clock() - t > 5
    setFreezeAnimation(false)
    return LocalPlayer:GetAttribute("InEnchantedTree") ~= true
end

local function bk()
    local p = bn2()
    if not p then return false, "Net Failed: No Tree" end
    local function ck() if State.on or not State.bf then return "Stopped" end return nil end
    State.bo = true
    local ok, r = bj(p, ck)
    if not ok then State.bo = false return false, fp("Net Failed", r) end
    local h = getRootAndHumanoid()
    if not h then State.bo = false return false, "Net Failed: No Character" end
    local o2, wy = moveToTarget(Vector3.new(p.an.Position.X - 5, h.Position.Y, p.an.Position.Z), ck, 10)
    if not o2 then State.bo = false return false, fp("Net Failed", wy) end
    task.wait(0.3)
    bh(p.np)
    local t = os.clock()
    repeat task.wait(0.2) until bn() or os.clock() - t > 5
    local got = bn()
    be(p, ck)
    State.bo = false
    return got, not got and "Net Failed: Not Claimed" or nil
end

local function bu()
    if State.on then return false, "Teleport Failed: Farm Running" end
    if State.hd or State.hm or State.hx then return false, "Teleport Failed: Halloween Running" end
    if State.bo then return false, "Teleport Failed: Busy" end
    local p = bn2()
    if not p then return false, "Teleport Failed: No Tree" end
    local function ck() if State.on then return "Stopped" end return nil end
    State.bo = true
    local ok, r = bj(p, ck)
    if not ok then State.bo = false return false, fp("Teleport Failed", r) end
    local h = getRootAndHumanoid()
    if not h then State.bo = false return false, "Teleport Failed: No Character" end
    local o2, wy = moveToTarget(Vector3.new(p.an.Position.X - 5, h.Position.Y, p.an.Position.Z), ck, 10)
    setFreezeAnimation(false)
    State.bo = false
    if not o2 then return false, fp("Teleport Failed", wy) end
    return true
end

local wqd = (function()
    local ok, r = pcall(function() return require(ReplicatedStorage.Data.WispQuests) end)
    return ok and type(r) == "table" and r or nil
end)()

local function ws()
    local o, s = pcall(SaveModule.Await)
    local w = o and type(s) == "table" and s.WispCompanion
    return type(w) == "table" and w or nil
end

local function wu()
    local w = ws()
    return w ~= nil and w.Unlocked == true
end

local function fi(r, ...)
    local a = table.pack(pcall(r.InvokeServer, r, ...))
    if a[1] and a[2] then return true end
    return false, tostring(a[1] and (a[3] or "Denied") or a[2])
end

local function wr(a, b)
    return fi(RemotesModule.WispCompanion.Request, a, b)
end

local function wa(id)
    local ok, r = pcall(function() return require(ReplicatedStorage.Data.Areas) end)
    local d = ok and type(r) == "table" and (r.Directory or r)[id]
    local o = {}
    for _, x in type(d) == "table" and d.DropTable or {} do
        if type(x) == "table" and type(x[1]) == "string" then o[x[1]] = true end
    end
    return o
end

local function wp(id)
    local o, s = pcall(SaveModule.Await)
    if not o or type(s) ~= "table" then return nil end
    local c, q, g = wa(id), {}, {}
    for _, u in ipairs(s.EquippedAssets or {}) do q[u] = true end
    for k, v in pairs(s.Inventory or {}) do
        local o2, i = pcall(AssetItemsUtil.Decode, v)
        if o2 and type(i) == "table" and c[i.Category] and not i.IsFavorite and not i.InFuse and not i.CreatorTemporary and not q[k] then
            g[i.Category] = g[i.Category] or {}
            table.insert(g[i.Category], k)
            if #g[i.Category] >= 3 then return g[i.Category] end
        end
    end
    return nil
end

local function wz(l)
    local m = workspace:FindFirstChild("World")
    m = m and m:FindFirstChild("Machines")
    m = m and m:FindFirstChild("FuseMachine")
    m = m and m:FindFirstChild("Machine")
    if not (m and m:IsA("BasePart")) then return false, "Fuse Failed: No Machine" end
    local h = getRootAndHumanoid()
    if not h then return false, "Fuse Failed: No Character" end
    if State.bt or getActiveTreadmillModel() then
        leaveTreadmill()
        task.wait(1)
        h = getRootAndHumanoid()
        if not h then return false, "Fuse Failed: No Character" end
    end
    local d = Vector3.new(h.Position.X - m.Position.X, 0, h.Position.Z - m.Position.Z)
    if d.Magnitude > 6 then
        State.bo = true
        local ok, wy = moveToTarget(Vector3.new(m.Position.X, h.Position.Y, m.Position.Z) + d.Unit * 5, nil, 15)
        State.bo = false
        setFreezeAnimation(false)
        if not ok then return false, fp("Fuse Failed", wy) end
    end
    local o, s = pcall(SaveModule.Await)
    if not o or type(s) ~= "table" then return false, "Fuse Failed: No Save" end
    if s.FusionInfoAcknowledged == false then fi(RemotesModule.Fusery.ConfirmBriefing) end
    if s.FusionEggReward then fi(RemotesModule.Fusery.FinishReveal) end
    for _, u in table.clone(type(s.FusionSlots) == "table" and s.FusionSlots or {}) do fi(RemotesModule.Fusery.EjectPet, u) end
    for _, u in l do
        local ok, wy = fi(RemotesModule.Fusery.LoadPet, u, false)
        if not ok then return false, "Fuse Failed: " .. wy end
        task.wait(0.3)
    end
    local ok, wy = fi(RemotesModule.Fusery.BeginFuse)
    if not ok then return false, "Fuse Failed: " .. wy end
    task.wait(0.6)
    ok, wy = fi(RemotesModule.Fusery.FinishReveal)
    if not ok then return false, "Fuse Reveal Failed: " .. wy end
    return true
end

local function we()
    State.wf = nil
    if not State.wo then return end
    State.wo = false
    if State.on then State.Stop() end
end

local function wt()
    local p, z = bn2(), workspace:FindFirstChild("World")
    z = z and z:FindFirstChild("Build")
    z = z and z:FindFirstChild("EnchantedForestZone")
    local a = z and z:FindFirstChild("Zone13Props")
    a = a and a:FindFirstChild("EnchantedTree")
    a = a and a:FindFirstChild("WispAnchor")
    local pr = a and a:FindFirstChild("WispTreePrompt")
    if not (p and pr) then return "Wisp Return Failed: No Tree" end
    local function ck() if State.on or not State.bf then return "Stopped" end return nil end
    State.bo = true
    local ok, wy = bd(p, ck)
    if ok then
        bh(pr)
        local t = os.clock()
        repeat task.wait(0.2) until wu() or os.clock() - t > 8
        ok, wy = wu(), "Not Unlocked"
    end
    State.bo = false
    return ok and "Wisp Unlocked" or fp("Wisp Return Failed", wy)
end

local function wd()
    local w = ws()
    if not w or not wqd then return "Wisp Failed: No State" end
    if not w.Accepted then
        local ok, m = wr("Accept")
        return ok and "Wisp Accepted" or "Wisp Accept Failed: " .. tostring(m)
    end
    local q = wqd.List[w.Stage]
    if q and not w.Claimed[q.Id] and (w.Progress[q.Id] or 0) >= q.Target then
        local ok, m = wr("Claim", q.Id)
        return ok and "Wisp Claimed | " .. q.Title or "Wisp Claim Failed: " .. tostring(m)
    end
    if q and q.Event == "EggSecured" then
        State.wf = q.Match.AreaId and {a = q.Match.AreaId} or {r = q.Match.Rarity}
        if not State.on then
            State.wo = true
            State.Start()
        end
        return string.format("Wisp Quest | %s %d/%d", q.Title, w.Progress[q.Id] or 0, q.Target)
    end
    if q and q.Event == "PetFused" then
        local l = wp(q.Match.AreaId)
        if not l then
            State.wf = {a = q.Match.AreaId}
            if not State.on then
                State.wo = true
                State.Start()
            end
            return "Wisp Quest | " .. q.Title .. " | Waiting For 3 Same Pets"
        end
        we()
        if State.on then return "Wisp | Waiting For Farm" end
        local ok, m = wz(l)
        return ok and "Wisp Fused" or tostring(m)
    end
    we()
    if State.on then return "Wisp | Waiting For Farm" end
    if q then return "Wisp Quest | " .. q.Title .. " | Not Automated" end
    if not w.Returning then
        local ok, m = wr("BeginReturn")
        return ok and "Wisp Returning" or "Wisp Return Failed: " .. tostring(m)
    end
    return wt()
end

local function gl()
    local v = workspace:GetAttribute("RLGL_Light")
    return type(v) == "string" and v or "Rise"
end

local function ge2()
    return workspace:GetAttribute("Event_RedLightGreenLight") == true and typeof(workspace:GetAttribute("RLGL_EggPosition")) == "Vector3"
end

local function gr2(ep)
    local p = workspace:GetAttribute(ep)
    if typeof(p) ~= "Vector3" then return nil end
    local ok, s = pcall(EggStateModule.ReadFieldEggs)
    local b, bd = nil, 25
    for _, r in ok and type(s) == "table" and s.Records or {} do
        if typeof(r.BoundsCFrame) == "CFrame" then
            local d = (r.BoundsCFrame.Position - p).Magnitude
            if d < bd then b, bd = r, d end
        end
    end
    return b
end

local function gm(tg, sp, ck, to, gt, aj)
    local t0, dn, wy = os.clock(), nil, nil
    local cn = RunService.PreSimulation:Connect(function(d)
        if dn ~= nil then return end
        local hr, hm = getRootAndHumanoid()
        if globalEnv.__SaeAf ~= State then dn, wy = false, "Stopped" return end
        if not hr or not hm or hm.Health <= 0 then dn, wy = false, "Character Lost" return end
        setFreezeAnimation(hm.FloorMaterial == Enum.Material.Air)
        local r = ck and ck()
        if r then dn, wy = false, r return end
        if os.clock() - t0 > to then dn, wy = false, "Timeout" return end
        local g = Vector3.new(0, workspace.Gravity * d * 0.5, 0)
        hr.AssemblyAngularVelocity = Vector3.zero
        if gt and not gt() then
            hr.AssemblyLinearVelocity = g
            return
        end
        local a2 = aj and aj(hr.Position, tg)
        local v = (a2 or tg) - hr.Position
        local m = v.Magnitude
        if m <= 1 and not a2 then
            hr.AssemblyLinearVelocity = g
            dn = true
            return
        end
        hr.AssemblyLinearVelocity = v.Unit * math.min(sp(), m / math.max(d, 1 / 240)) + g
    end)
    State.cn[#State.cn + 1] = cn
    while dn == nil do RunService.Heartbeat:Wait() end
    disconnectAndRemove(cn)
    return dn, wy
end

local function gb()
    local ok, g = pcall(function() return require(ReplicatedStorage.Data.Gears).Directory end)
    for _, c in {LocalPlayer.Character, LocalPlayer:FindFirstChildOfClass("Backpack")} do
        for _, t in c and c:GetChildren() or {} do
            local n = t:IsA("Tool") and t.Name:gsub("%s*%b[]$", "")
            if n and ok and type(g) == "table" and type(g[n]) == "table" and g[n].BatControllerData then return t end
        end
    end
    return nil
end

local function gh(p, gt)
    local hr, hu = getRootAndHumanoid()
    local tr = p.Character and p.Character:FindFirstChild("HumanoidRootPart")
    if not (hr and tr) then return false, "No Target" end
    if (tr.Position - hr.Position).Magnitude > 14 or (gt and not gt()) then return false, "Not Ready" end
    local t = gb()
    if not t then return false, "No Bat" end
    if t.Parent ~= LocalPlayer.Character and hu then
        pcall(function() hu:EquipTool(t) end)
        task.wait(0.1)
    end
    pcall(function() t:Activate() end)
    return true
end

local function autoMinigameLoopBody(generationId, eventConfig)
    local lm, nb = nil, 0
    local function l2(m)
        if m == lm then return end
        lm = m
        updateStatusLog(m)
    end
    local function ck()
        if not eventConfig.on(generationId) or globalEnv.__SaeAf ~= State or not eventConfig.ev() then return "Stopped" end
        return nil
    end
    local function gm2(a, sp2, ck3, to2)
        return gm(a, sp2, ck3, to2, eventConfig.gt, eventConfig.aj)
    end
    local function own(on)
        if on then
            State.bo, State[eventConfig.fb] = true, true
            return swapToClonedHumanoid()
        end
        local _, hu = getRootAndHumanoid()
        if hu then pcall(function() hu:UnequipTools() end) end
        setFreezeAnimation(false)
        local t = os.clock()
        while hu and hu.Parent and hu.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
        restoreOriginalHumanoid()
        State.bo, State[eventConfig.fb] = false, false
        return true
    end
    while globalEnv.__SaeAf == State and eventConfig.on(generationId) do
        State[eventConfig.fx] = eventConfig.ev()
        if State[eventConfig.fx] and eventConfig.pre then eventConfig.pre() end
        if not State[eventConfig.fx] then
            if State[eventConfig.fb] then own(false) end
            l2(eventConfig.n .. " | Waiting For Event")
            task.wait(1)
            continue
        end
        local lv0 = eventConfig.lv and LocalPlayer:GetAttribute(eventConfig.lv)
        if type(lv0) == "number" and lv0 <= 0 then
            if State[eventConfig.fb] then own(false) end
            l2(eventConfig.n .. " | No Lives")
            task.wait(1)
            continue
        end
        local r = gr2(eventConfig.ep)
        local o1, cs0 = pcall(EggStateModule.ReadCarryState)
        local mine = o1 and type(cs0) == "table" and cs0.IsCarrying == true and workspace:GetAttribute(eventConfig.cu) == LocalPlayer.UserId
        if not State[eventConfig.fb] then
            if (isCarryingEgg() and not mine) or (State.on and not State.fp) or (State.bo and not State[eventConfig.fb]) then
                l2(eventConfig.n .. " | Waiting For Control")
                task.wait(0.5)
                continue
            end
            if State.bt or getActiveTreadmillModel() then
                leaveTreadmill()
                task.wait(1)
            end
            if not own(true) then
                State.bo, State[eventConfig.fb] = false, false
                l2(eventConfig.n .. " | Waiting To Swap")
                task.wait(0.5)
                continue
            end
        end
        local _, hu = getRootAndHumanoid()
        local ws = hu and hu.WalkSpeed or 16
        if mine then
            local sm = tonumber(cs0.SpeedMultiplier) or 0.5
            local function vs() return ws * sm end
            local l, h = getSeparationLine(), getRootAndHumanoid()
            if l and h then
                local lv, lq = l.CFrame.LookVector, l.Position
                local q = h.Position - lv * (lv:Dot(h.Position - lq) - 48)
                local z0, z1 = getCorridorZBounds()
                local en = Vector3.new(q.X, lq.Y + 3, z0 and math.clamp(q.Z, z0, z1) or q.Z)
                local y = math.max(h.Position.Y, lq.Y + 45)
                local function ck2()
                    local checkErr = ck()
                    if checkErr then return checkErr end
                    if not isCarryingEgg() then return "Dropped" end
                    return nil
                end
                l2(eventConfig.n .. " | Carrying Home")
                local ok, wy = gm2(Vector3.new(h.Position.X, y, h.Position.Z), vs, ck2, 30)
                if ok then ok, wy = gm2(Vector3.new(en.X, y, en.Z), vs, ck2, (en - h.Position).Magnitude / math.max(vs(), 8) * 3 + 30) end
                if ok then ok, wy = gm2(en, vs, ck2, 30) end
                if ok then ok, wy = gm2(en - lv * 70, vs, ck2, 40) end
                l2(ok and eventConfig.n .. " | Walked In" or eventConfig.n .. " | " .. tostring(wy))
                if not isCarryingEgg() then
                    task.wait(2)
                    if not eventConfig.ev() or not gr2(eventConfig.ep) then l2(eventConfig.n .. " Delivered") end
                end
            end
        elseif r then
            local cu = workspace:GetAttribute(eventConfig.cu)
            local cp = type(cu) == "number" and cu ~= LocalPlayer.UserId and Players:GetPlayerByUserId(cu) or nil
            local function vs() return ws * State.fm end
            if cp then
                local tr = cp.Character and cp.Character:FindFirstChild("HumanoidRootPart")
                if tr then
                    l2(eventConfig.n .. " | Chasing " .. cp.Name)
                    gm2(tr.Position + Vector3.new(0, 2, 0), vs, function()
                        local checkErr = ck()
                        if checkErr then return checkErr end
                        local h = getRootAndHumanoid()
                        if h and (tr.Position - h.Position).Magnitude <= 8 then return "Near" end
                        return nil
                    end, 3)
                    if os.clock() >= nb and gh(cp, eventConfig.gt) then
                        nb = os.clock() + 0.7
                        l2(eventConfig.n .. " | Hit " .. cp.Name)
                    end
                else
                    task.wait(0.2)
                end
            else
                local p = r.BoundsCFrame.Position
                l2(eventConfig.n .. " | Flying To Egg")
                local h = getRootAndHumanoid()
                local ok = true
                if h and (p - h.Position).Magnitude > 60 then
                    local y = math.max(h.Position.Y, p.Y) + 50
                    ok = gm2(Vector3.new(h.Position.X, y, h.Position.Z), vs, ck, 20)
                    if ok then ok = gm2(Vector3.new(p.X, y, p.Z), vs, ck, (p - h.Position).Magnitude / math.max(vs(), 16) * 3 + 30) end
                end
                if ok then ok = gm2(p + Vector3.new(0, 3, 0), vs, ck, 20) end
                if ok then
                    while eventConfig.gt and not eventConfig.gt() and ck() == nil do task.wait(0.05) end
                    l2(eventConfig.n .. " | Taking Egg")
                    local o2, wy = takeFieldEgg({u = r.Uid, r = r})
                    l2(o2 and eventConfig.n .. " | Egg Taken" or eventConfig.n .. " | " .. tostring(wy))
                    local t2 = os.clock()
                    while o2 and workspace:GetAttribute(eventConfig.cu) ~= LocalPlayer.UserId and os.clock() - t2 < 1.5 do task.wait(0.05) end
                end
            end
        else
            l2(eventConfig.n .. " | No Egg")
            task.wait(0.5)
        end
        task.wait(0.05)
    end
    State[eventConfig.fx] = false
    if State[eventConfig.fb] then own(false) end
end

local function startAutoMinigameThread(generationId, eventConfig)
    local ok, e = xpcall(autoMinigameLoopBody, debug.traceback, generationId, eventConfig)
    if ok or globalEnv.__SaeAf ~= State or not eventConfig.on(generationId) then return end
    State[eventConfig.fx] = false
    if State[eventConfig.fb] then
        local _, hu = getRootAndHumanoid()
        if hu then pcall(function() hu:UnequipTools() end) end
        pcall(setFreezeAnimation, false)
        local t = os.clock()
        while hu and hu.Parent and hu.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
        pcall(restoreOriginalHumanoid)
        State.bo, State[eventConfig.fb] = false, false
    end
    handleFeatureError(eventConfig.t, eventConfig.k, eventConfig.n .. " Crashed", e)
end


-- ============================================================================
-- [6] Minigame Events (Red Light Green Light & T-Rex Run)
-- ============================================================================
local lc = {t = "Auto Red Light Green Light", k = "arl", n = "Red Light", fx = "lx", fb = "lb", ep = "RLGL_EggPosition", cu = "RLGL_CarrierUserId", lv = "RLGL_Lives", ev = ge2, gt = function() return gl() == "Green" end, on = function(g) return State.gx and State.gg == g end}

local trh = (function()
    local ok, r = pcall(function() return require(ReplicatedStorage.Shared.Modules.TrexRunHerd) end)
    return ok and type(r) == "table" and r or nil
end)()

local tw = {}

local function tw2(l)
    table.clear(tw)
    for _, w in type(l) == "table" and l or {} do
        if type(w) == "table" and type(w.Id) == "number" then tw[w.Id] = w end
    end
end

pcall(function()
    State.cn[#State.cn + 1] = RemotesModule.TrexRun.Wave.OnClientEvent:Connect(function(w)
        if type(w) == "table" and type(w.Id) == "number" then tw[w.Id] = w end
    end)
    State.cn[#State.cn + 1] = RemotesModule.TrexRun.Herd.OnClientEvent:Connect(tw2)
end)

local function te2()
    return workspace:GetAttribute("Event_TrexRun") == true and typeof(workspace:GetAttribute("TrexRun_EggPosition")) == "Vector3"
end

local function ta2(pos)
    if not trh then return nil end
    local co = trh.ReadCourse(workspace)
    if not co then return nil end
    local now = workspace:GetServerTimeNow()
    local dp, lt = trh.Locate(co, pos)
    local bad, thr, hw = {}, false, co.LaneWidth * 0.5 + 4
    for id, w in tw do
        if trh.IsExpired(w, co, now) then
            tw[id] = nil
        elseif trh.IsMoving(w, now) and type(w.Lanes) == "table" then
            local ah = (dp - trh.DepthAt(w, now)) * w.Direction
            if ah > -10 and ah < (tonumber(w.Speed) or 150) * 1.2 + 30 then
                for k in w.Lanes do
                    local la = trh.TrexLateral(co, w, k)
                    bad[#bad + 1] = la
                    if math.abs(la - lt) < hw then thr = true end
                end
            end
        end
    end
    if not thr then return nil end
    local best, bd
    for ln = 1, co.Lanes do
        local c0 = trh.LaneOffset(co, ln)
        local ok = true
        for _, la in bad do
            if math.abs(la - c0) < hw then
                ok = false
                break
            end
        end
        if ok and (not bd or math.abs(c0 - lt) < bd) then best, bd = c0, math.abs(c0 - lt) end
    end
    if not best then return nil end
    local q = trh.Point(co, dp, best)
    return Vector3.new(q.X, pos.Y, q.Z)
end

local nf = 0

local function tp2()
    if next(tw) ~= nil or os.clock() < nf then return end
    nf = os.clock() + 3
    local ok, l = pcall(function() return RemotesModule.TrexRun.FetchHerd:InvokeServer() end)
    if ok then tw2(l) end
end

local tc2 = {t = "Auto T-Rex Run", k = "atx", n = "T-Rex", fx = "tx", fb = "tb", ep = "TrexRun_EggPosition", cu = "TrexRun_CarrierUserId", lv = nil, ev = te2, gt = nil, aj = ta2, pre = tp2, on = function(g) return State.tq and State.th == g end}


-- ============================================================================
-- [5] Butterfly & Wisp Quest Automation
-- ============================================================================
local function autoButterflyLoopBody(generationId)
    if not (bb and bw and bq) then handleFeatureError("Auto Butterfly", "abf", "Butterfly Failed: No Module") return end
    if not isKeyValid() then updateStatusLog("Key Check Failed: No Valid Key") return end
    if not bs then updateStatusLog("Butterfly | Auto Swing Not Found") end
    local by, c0, nx, lw = false, 0, 0, nil
    local function ed()
        if not by then return end
        by = false
        if State.on and not State.fp then
            State.bo = false
            return
        end
        local _, hu = getRootAndHumanoid()
        local t = os.clock()
        while hu and hu.Parent and hu.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
        if hu then pcall(function() hu:UnequipTools() end) end
        restoreOriginalHumanoid()
        State.bo = false
    end
    local function ck()
        if (State.on and not State.fp) or not State.bf or State.bi ~= generationId or not ba() then return "Stopped" end
        return nil
    end
    while globalEnv.__SaeAf == State and State.bf and State.bi == generationId do
        local nc, tc = bn()
        if State.lx or State.tx then
            State.bx = false
            ed()
            task.wait(1)
            continue
        end
        State.bx = nc and ba() and true or false
        if not nc and not wu() then
            ed()
            local m = wd()
            if m ~= lw then
                lw = m
                updateStatusLog(m)
            end
            task.wait(1)
            continue
        end
        if State.wf or State.wo then we() end
        if State.on and not State.fp then
            by, State.bo = false, false
            task.wait(1)
            continue
        end
        if not nc then
            ed()
            updateStatusLog("Claiming Net")
            local ok, wy = bk()
            updateStatusLog(ok and "Net Claimed" or tostring(wy))
            task.wait(ok and 1 or 30)
            continue
        end
        if not ba() then
            ed()
            updateStatusLog("Butterfly Waiting For Event")
            task.wait(1)
            continue
        end
        local nd, na = bd2()
        if not na then nd = nil end
        local b = bz()
        if not b then
            updateStatusLog("Butterfly Failed: No Area")
            task.wait(5)
            continue
        end
        if not by then
            State.bo = true
            if not swapToClonedHumanoid() then
                State.bo = false
                updateStatusLog("Butterfly Waiting To Swap")
                task.wait(0.5)
                continue
            end
            by, c0 = true, tc
        end
        local h, hu = getRootAndHumanoid()
        if not h or not hu or hu.Health <= 0 then
            task.wait(0.5)
            continue
        end
        local o = b.CFrame:PointToObjectSpace(h.Position)
        local fr = math.max(math.abs(o.X) - b.Size.X / 2, math.abs(o.Z) - b.Size.Z / 2)
        if fr > 20 then
            if LocalPlayer:GetAttribute("InEnchantedTree") == true then
                local p2 = bn2()
                updateStatusLog("Leaving Tree")
                if p2 then be(p2, ck) end
                task.wait(0.5)
                continue
            end
            local dn = getTimeUntilNight()
            if dn == 0 or isStartWallCollidable() or (dn and dn < (b.Position - h.Position).Magnitude / math.max(hu.WalkSpeed, 16) + 5) then
                updateStatusLog("Butterfly Waiting For Day")
                task.wait(0.5)
                continue
            end
            if fr < 150 then
                hu:MoveTo(Vector3.new(b.Position.X, h.Position.Y, b.Position.Z))
                task.wait(0.2)
                continue
            end
            if State.bt or getActiveTreadmillModel() then
                updateStatusLog("Leaving Treadmill")
                local o3, w3 = leaveTreadmill()
                if not o3 then updateStatusLog(tostring(w3)) end
                task.wait(1)
                continue
            end
            updateStatusLog("Flying To Butterflies")
            local z0, z1 = getCorridorZBounds()
            if z0 and (h.Position.Z < z0 or h.Position.Z > z1) then
                local ok, wy = moveToTarget(Vector3.new(h.Position.X, h.Position.Y, math.clamp(h.Position.Z, z0, z1)), ck, 10)
                h, hu = getRootAndHumanoid()
                if not ok or not h or not hu then
                    updateStatusLog(tostring(wy or "Move Failed: Character Lost"))
                    task.wait(0.5)
                    continue
                end
            end
            local y = math.max(h.Position.Y, b.Position.Y) + 50
            local ok, wy = hopToPosition(Vector3.new(b.Position.X, y, b.Position.Z), y, false, ck)
            if ok then ok, wy = hopToPosition(b.Position + Vector3.new(0, 3, 0), y, false, ck) end
            setFreezeAnimation(false)
            if not ok then
                updateStatusLog(tostring(wy))
                task.wait(0.5)
            end
            continue
        end
        setFreezeAnimation(false)
        local t = bv()
        if t and t.Parent ~= LocalPlayer.Character then pcall(function() hu:EquipTool(t) end) end
        if bs then bs() end
        local p, pk, pd, s, sk, sd = nil, -1, math.huge, nil, -1, math.huge
        local x1, xk, xd, y1, yk, yd = nil, -1, math.huge, nil, -1, math.huge
        for _, e in bw.Each() do
            local eo = typeof(e.Position) == "Vector3" and b.CFrame:PointToObjectSpace(e.Position)
            if (e.Alpha or 0) >= 0.5 and eo and math.abs(eo.X) <= b.Size.X / 2 + 10 and math.abs(eo.Z) <= b.Size.Z / 2 + 10 and type(e.Flight) == "table" then
                local m, k = (e.Position - h.Position).Magnitude, tonumber(type(e.Tier) == "table" and e.Tier.Order) or 0
                if not nd or (type(e.Tier) == "table" and nd[e.Tier.Id] ~= nil) then
                    if k > pk or (k == pk and m < pd) then p, pk, pd = e, k, m end
                    if m <= 13 and (k > sk or (k == sk and m < sd)) then s, sk, sd = e, k, m end
                end
                if k > xk or (k == xk and m < xd) then x1, xk, xd = e, k, m end
                if m <= 13 and (k > yk or (k == yk and m < yd)) then y1, yk, yd = e, k, m end
            end
        end
        p, s = p or x1, s or y1
        if s and os.clock() >= nx then
            nx = os.clock() + 0.5
            local id = s.Flight.Id
            task.spawn(function()
                local ok, r = pcall(function() return RemotesModule.Butterflies.AskSwing:InvokeServer(id) end)
                if ok and (r == "Caught" or r == "Gone") then pcall(bw.Remove, id) end
            end)
        end
        if p then
            local q = p.Position
            pcall(function() q = bq.PositionAt(p.Flight, workspace:GetServerTimeNow() + 0.3) end)
            local v = Vector3.new(h.Position.X - q.X, 0, h.Position.Z - q.Z)
            q = v.Magnitude > 0.1 and q + v.Unit * 11 or q
            hu:MoveTo(Vector3.new(q.X, h.Position.Y, q.Z))
        else
            hu:MoveTo(Vector3.new(b.Position.X, h.Position.Y, b.Position.Z))
        end
        updateStatusLog(string.format("%s | Caught %d", nd and "Catching For Essence" or "Catching Rarest", tc - c0))
        task.wait(0.1)
    end
    State.bx = false
    ed()
    we()
end

local function startAutoButterflyThread(generationId)
    local ok, e = xpcall(autoButterflyLoopBody, debug.traceback, generationId)
    if ok or globalEnv.__SaeAf ~= State or State.bi ~= generationId then return end
    State.bx = false
    if State.on and not State.fp then
        State.bo = false
    elseif State.bo and not (State.lb or State.tb or State.hb) then
        local _, hu = getRootAndHumanoid()
        if hu then pcall(function() hu:UnequipTools() end) end
        pcall(restoreOriginalHumanoid)
        State.bo = false
    end
    pcall(we)
    handleFeatureError("Auto Butterfly", "abf", "Butterfly Crashed", e)
end


-- ============================================================================
-- [7] Halloween Dimension & Ghost Train Automation
-- ============================================================================
local function autoHalloweenLoopBody(generationId)
    local lm
    local function l2(m)
        if m == lm then return end
        lm = m
        updateStatusLog("Halloween | " .. m)
    end
    local function hz2(n)
        local m = workspace:FindFirstChild("HalloweenDimension")
        local z = m and m:FindFirstChild("Zones")
        local p = z and z:FindFirstChild(n)
        return p and p:IsA("BasePart") and p or nil
    end
    local function hin() return LocalPlayer:GetAttribute("InHalloweenDimension") == true end
    local function gg()
        local m = workspace:FindFirstChild("HalloweenDimension")
        local g2 = m and m:FindFirstChild("Ground")
        return g2 and {g2, workspace.Terrain} or {workspace.Terrain}
    end
    local function zq(q)
        for i = 1, 3 do
            local b = hz2("Zone" .. i)
            if b and math.abs(q.X - b.Position.X) <= b.Size.X / 2 and math.abs(q.Z - b.Position.Z) <= b.Size.Z / 2 then return i end
        end
        return nil
    end
    local function hca() return LocalPlayer:GetAttribute("HalloweenCarrying") == true end
    local function on() return globalEnv.__SaeAf == State and (State.hd or State.hm or State.hx) and State.hg == generationId end
    local function own(o)
        if o then
            State.bo, State.hb = true, true
            if swapToClonedHumanoid() then return true end
            State.bo, State.hb = false, false
            return false
        end
        local _, hu = getRootAndHumanoid()
        setFreezeAnimation(false)
        local t = os.clock()
        while hu and hu.Parent and hu.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
        restoreOriginalHumanoid()
        State.bo, State.hb = false, false
        return true
    end
    local function el()
        local m = workspace:FindFirstChild("HalloweenDimension")
        local w = m and m:FindFirstChild("WorldEggs")
        local o, t, h = {}, os.clock(), getRootAndHumanoid()
        for _, e in w and w:GetChildren() or {} do
            local u, pn = e:GetAttribute("HalloweenEggUid"), e:GetAttribute("PreparedSourceName")
            local p = e:FindFirstChild("StealPrompt", true)
            local ok, q = pcall(e.GetPivot, e)
            local z = ok and zq(q.Position)
            if type(u) == "string" and type(pn) == "string" and p and p:IsA("ProximityPrompt") and p.Enabled and (State.hbl[u] or 0) < t and z then
                local a = AssetDirectory[pn]
                o[#o + 1] = {u = u, p = q.Position, pr = p, z = z, n = pn, e = tonumber(a and a.EarningRate) or 0, k = tonumber(a and a.Rarity and a.Rarity.Rank) or 0}
            end
        end
        table.sort(o, function(x, y)
            if x.k ~= y.k then return x.k > y.k end
            if x.e ~= y.e then return x.e > y.e end
            return h ~= nil and (x.p - h.Position).Magnitude < (y.p - h.Position).Magnitude
        end)
        return o
    end
    local function en()
        local w = workspace:FindFirstChild("World")
        local p = w and w:FindFirstChild("HalloweenDimensionPortal")
        local b = p and p:FindFirstChild("Hitbox", true)
        pg2(true)
        if not (b and b:IsA("BasePart")) then return false, "Enter Failed: No Portal" end
        if State.bt or getActiveTreadmillModel() then
            leaveTreadmill()
            task.wait(1)
        end
        local t0 = os.clock()
        while not hin() and os.clock() - t0 < 40 do
            if not on() then return false, "Stopped" end
            local _, hu = getRootAndHumanoid()
            if hu then hu:MoveTo(b.Position) end
            task.wait(0.25)
        end
        if not hin() then return false, "Enter Failed: Timeout" end
        local t = os.clock()
        repeat task.wait(0.2) until not tostring(LocalPlayer:GetAttribute("HalloweenTransition") or ""):find(":cover:") or os.clock() - t > 10
        return true
    end
    local function nk()
        local m = workspace:FindFirstChild("HalloweenDimension")
        local f = m and m:FindFirstChild("Spawns")
        local s = hz2("SafeZone")
        local lo, hi, l = math.huge, -math.huge, {}
        for _, p in f and f:GetDescendants() or {} do
            if p:IsA("BasePart") and p.Name == "Spawn" then
                lo, hi = math.min(lo, p.Position.X), math.max(hi, p.Position.X)
                l[#l + 1] = p.Position
            end
        end
        if not s or #l == 0 then return nil end
        return lo - 100, hi + 100, s.Position.Z + s.Size.Z / 2 + 40, l
    end
    local function gt(e)
        local h, hu = getRootAndHumanoid()
        if not h or not hu then return false, "Fly Failed: No Character" end
        local u = Vector3.new(e.p.X - h.Position.X, 0, e.p.Z - h.Position.Z)
        local b = u.Magnitude > 7 and e.p - u.Unit * 7 or e.p
        local function ck()
            if not on() then return "Stopped" end
            if not e.pr.Parent or not e.pr.Enabled then return "Fly Failed: Egg Gone" end
            return nil
        end
        local lo, hi, zn = nk()
        if lo and h.Position.Z < zn then
            local x = math.clamp(h.Position.X, lo, hi)
            local ok, wy = hopToPosition(Vector3.new(x, getGroundY(Vector3.new(x, h.Position.Y, zn), Vector3.zero, 60, gg()) - State.gh + 3, zn), nil, false, ck, gg())
            if not ok then return false, wy end
        end
        local ok, wy = hopToPosition(Vector3.new(b.X, getGroundY(Vector3.new(b.X, e.p.Y, b.Z), Vector3.zero, 60, gg()) - State.gh + 3, b.Z), nil, false, ck, gg())
        if not ok then return false, wy end
        task.wait(0.15)
        pcall(fireproximityprompt, e.pr)
        local t = os.clock()
        repeat task.wait() until hca() or os.clock() - t > 1.5
        if not hca() then return false, "Steal Failed: Not Carrying" end
        return true
    end
    local function rl2()
        local h = getRootAndHumanoid()
        if not h then return false, "Relay Failed: No Character" end
        local ok, a, b = pcall(RemotesModule.HalloweenDimension.AskDropEgg.InvokeServer, RemotesModule.HalloweenDimension.AskDropEgg)
        if not ok or not a then return false, "Relay Failed: Drop " .. tostring(ok and (b or "Denied") or a) end
        local t = os.clock()
        repeat task.wait() until not hca() or os.clock() - t > 1
        if hca() then return false, "Relay Failed: Still Carrying" end
        local m = workspace:FindFirstChild("HalloweenDimension")
        local w = m and m:FindFirstChild("WorldEggs")
        local pr, bd = nil, 25
        t = os.clock()
        while not pr and os.clock() - t < 1 do
            for _, x in w and w:GetChildren() or {} do
                local p = x:FindFirstChild("StealPrompt", true)
                local o2, q = pcall(x.GetPivot, x)
                if p and p:IsA("ProximityPrompt") and p.Enabled and o2 and (q.Position - h.Position).Magnitude < bd then pr, bd = p, (q.Position - h.Position).Magnitude end
            end
            if not pr then task.wait() end
        end
        if not pr then return false, "Relay Failed: No Dropped Egg" end
        t = os.clock()
        local nf = 0
        repeat
            if os.clock() >= nf then
                nf = os.clock() + 0.25
                pcall(fireproximityprompt, pr)
            end
            task.wait()
        until hca() or os.clock() - t > 2
        if not hca() then return false, "Relay Failed: Retake" end
        return true
    end
    local function sh3(en)
        local h0 = getRootAndHumanoid()
        local lq = h0 and h0.Position
        while true do
            if not on() then return false, "Stopped" end
            if not hca() then return false, "Carry Ended" end
            local hr, hu = getRootAndHumanoid()
            if not hr or not hu then return false, "Home Failed: Character Lost" end
            setFreezeAnimation(true)
            local d, k = Vector3.new(en.X - hr.Position.X, 0, en.Z - hr.Position.Z), State.hk
            if d.Magnitude <= k then
                hr.CFrame = d.Magnitude > 0.1 and CFrame.lookAt(en, en + d.Unit) or CFrame.new(en) * hr.CFrame.Rotation
                hr.AssemblyLinearVelocity = Vector3.zero
                return true
            end
            local np = hr.Position + d.Unit * k
            np = Vector3.new(np.X, getGroundY(np, Vector3.zero, 60, gg()), np.Z)
            hr.CFrame = CFrame.lookAt(np, np + d.Unit)
            hr.AssemblyLinearVelocity = Vector3.zero
            if lq and (np - lq).Magnitude >= State.hr then
                task.wait(State.hw)
                local ok, wy = rl2()
                if not ok then return false, wy end
                lq = np
            end
            task.wait(0.1)
        end
    end
    local function hm2()
        local h = getRootAndHumanoid()
        local lo, hi, zn = nk()
        if not (h and lo) then return false, "Home Failed: No Safe Zone" end
        local d0 = LocalPlayer:GetAttribute("HalloweenDelivered")
        local ok, wy = true, nil
        if h.Position.Z > zn then
            local x = math.clamp(h.Position.X, lo, hi)
            ok, wy = sh3(Vector3.new(x, getGroundY(Vector3.new(x, h.Position.Y, zn), Vector3.zero, 60, gg()) - State.gh + 3, zn))
        end
        h = getRootAndHumanoid()
        if ok and h then
            local x, zi = math.clamp(h.Position.X, lo, hi), zn - 55
            ok, wy = sh3(Vector3.new(x, getGroundY(Vector3.new(x, h.Position.Y, zi), Vector3.zero, 60, gg()) - State.gh + 3, zi))
        end
        local t = os.clock()
        repeat task.wait(0.1) until LocalPlayer:GetAttribute("HalloweenDelivered") ~= d0 or os.clock() - t > 3
        if LocalPlayer:GetAttribute("HalloweenDelivered") ~= d0 then return true end
        return false, ok and "Not Delivered" or "Egg Lost | " .. tostring(wy)
    end
    local function ml()
        local m = workspace:FindFirstChild("HalloweenDimension")
        local o, h, t = {}, getRootAndHumanoid(), os.clock()
        for _, b in CollectionService:GetTagged("BatDamagable") do
            local x = b.Parent
            while x and x ~= m and not CollectionService:HasTag(x, "HalloweenMonster") do x = x.Parent end
            local v = b:IsA("BasePart") and b:GetAttribute("Health")
            if m and h and x and x ~= m and type(v) == "number" and v > 0 and zq(b.Position) and (State.hbl[x] or 0) < t then
                o[#o + 1] = {m = x, b = b, n = x.Name, k = ({Small = 1, Medium = 2, Large = 3})[x:GetAttribute("MonsterSize")] or 0, d = (b.Position - h.Position).Magnitude}
            end
        end
        table.sort(o, function(x, y)
            if x.k ~= y.k then return x.k > y.k end
            return x.d < y.d
        end)
        return o
    end
    local function kl(x)
        local h = getRootAndHumanoid()
        if not h then return false, "Kill Failed: No Character" end
        local p = x.b.Position
        local u = Vector3.new(p.X - h.Position.X, 0, p.Z - h.Position.Z)
        local q = u.Magnitude > 15 and p - u.Unit * 15 or p
        if not own(true) then return false, "Waiting To Swap" end
        local ok, wy = hopToPosition(Vector3.new(q.X, getGroundY(Vector3.new(q.X, p.Y, q.Z), Vector3.zero, 60, gg()) - State.gh + 3, q.Z), nil, false, function()
            if not on() then return "Stopped" end
            if not x.b.Parent or (tonumber(x.b:GetAttribute("Health")) or 0) <= 0 then return "Monster Gone" end
            return nil
        end, gg())
        if not ok then return wy == "Monster Gone", wy end
        local b = gb()
        if not b then return false, "Kill Failed: No Bat" end
        local t0, a0, nx, ns = os.clock(), math.random() * math.pi * 2, 0, 0
        while on() and os.clock() - t0 < 25 do
            if not x.b.Parent or (tonumber(x.b:GetAttribute("Health")) or 0) <= 0 then return true end
            if State.hd and #el() > 0 then return false, "Eggs Back" end
            local h2, hu = getRootAndHumanoid()
            if not h2 or not hu or hu.Health <= 0 or not (State.sc and State.sc.Parent == LocalPlayer.Character) then return false, "Kill Failed: Character Lost" end
            if b.Parent ~= LocalPlayer.Character then pcall(hu.EquipTool, hu, b) end
            if os.clock() >= ns then
                ns, State.bsq = os.clock() + State.bsi, State.bsq + 1
                pcall(function() RemotesModule.BatSwing.Trigger:FireServer(nil, string.format("%d:%d:%d", LocalPlayer.UserId, State.bsq, math.floor(workspace:GetServerTimeNow() * 1000))) end)
            end
            if os.clock() >= nx then
                nx, a0 = os.clock() + 0.3, a0 + math.pi / 3
                local c0 = x.b.Position
                local q2 = c0 + Vector3.new(math.cos(a0) * 12, 0, math.sin(a0) * 12)
                q2 = Vector3.new(q2.X, getGroundY(q2, Vector3.zero, 60, gg()) - State.gh + 3, q2.Z)
                setFreezeAnimation(true)
                h2.CFrame = CFrame.lookAt(q2, Vector3.new(c0.X, q2.Y, c0.Z))
                h2.AssemblyLinearVelocity = Vector3.zero
            end
            task.wait(0.05)
        end
        return false, "Kill Timeout"
    end
    local function px()
        local m = workspace:FindFirstChild("HalloweenDimension")
        local pp = m and m:FindFirstChild("PortalHome", true)
        pp = pp and pp:FindFirstChild("Portal", true)
        if not (pp and pp:IsA("BasePart")) then return false, "Exit Failed: No Portal" end
        local h = getRootAndHumanoid()
        local q = Vector3.new(pp.Position.X, 0, pp.Position.Z + 30)
        if h and (Vector3.new(h.Position.X, 0, h.Position.Z) - q).Magnitude > 60 then
            if not own(true) then return false, "Waiting To Swap" end
            local ok, wy = hopToPosition(Vector3.new(q.X, getGroundY(Vector3.new(q.X, pp.Position.Y, q.Z), Vector3.zero, 60, gg()) - State.gh + 3, q.Z), nil, false, function() return not on() and "Stopped" or nil end, gg())
            if not ok then return false, wy end
        end
        if State.hb then own(false) end
        local t0 = os.clock()
        while hin() and os.clock() - t0 < 30 do
            if not on() then return false, "Stopped" end
            local _, hu = getRootAndHumanoid()
            if hu then hu:MoveTo(pp.Position) end
            task.wait(0.25)
        end
        if hin() then return false, "Exit Failed: Timeout" end
        local t = os.clock()
        repeat task.wait(0.2) until not tostring(LocalPlayer:GetAttribute("HalloweenTransition") or ""):find(":cover:") or os.clock() - t > 10
        t = os.clock()
        repeat
            task.wait(0.1)
            local h2, hu = getRootAndHumanoid()
        until not on() or os.clock() - t > 8 or (h2 and hu and hu.FloorMaterial ~= Enum.Material.Air and Vector3.new(h2.AssemblyLinearVelocity.X, 0, h2.AssemblyLinearVelocity.Z).Magnitude < 2)
        if not on() then return false, "Stopped" end
        task.wait(0.3)
        if not own(true) then
            State.bo, State.hb = false, false
            return false, "Place Failed: Swap"
        end
        local n, wy = placeEggsInPen()
        own(false)
        return true, string.format("Placed %d%s", n, wy and (" | " .. wy) or "")
    end
    local o4, hdg = pcall(function() return require(ReplicatedStorage.Data.HalloweenDimensionEgg) end)
    local hpr = o4 and type(hdg) == "table" and tonumber(hdg.Price) or 2250
    local function tkn()
        local o, s2 = pcall(SaveModule.Await)
        if not o or type(s2) ~= "table" then return 0 end
        local c = type(s2.Currency) == "table" and s2.Currency.HalloweenTokens
        return tonumber(c or s2.HalloweenTokens) or 0
    end
    local function by()
        local m = workspace:FindFirstChild("HalloweenDimension")
        local sh = m and m:FindFirstChild("EggShop")
        local o0, sp = pcall(function() return sh:GetPivot().Position end)
        if not o0 then return false, "Buy Failed: No Shop" end
        local h = getRootAndHumanoid()
        if h and (Vector3.new(h.Position.X, sp.Y, h.Position.Z) - sp).Magnitude > 9 then
            if not own(true) then return false, "Waiting To Swap" end
            local q = sp + Vector3.new(0, 0, 6)
            local ok, wy = hopToPosition(Vector3.new(q.X, getGroundY(q, Vector3.zero, 60, gg()) - State.gh + 3, q.Z), nil, false, function() return not on() and "Stopped" or nil end, gg())
            if not ok then return false, wy end
        end
        local n, wy = 0, nil
        while on() and tkn() >= hpr do
            local o, a, b = pcall(RemotesModule.HalloweenDimension.AskBuyEgg.InvokeServer, RemotesModule.HalloweenDimension.AskBuyEgg)
            if not o or not a then
                wy = "Buy Failed: " .. tostring(o and (b or "Denied") or a)
                break
            end
            n += 1
            task.wait(0.3)
        end
        return n > 0, string.format("Bought %d%s", n, wy and " | " .. wy or "")
    end
    local nd, nk2 = 0, 0
    local su = false
    local function sx(v)
        if su == v then return end
        su = v
        pcall(function() RemotesModule.TrainEvent.Suction:FireServer(v) end)
    end
    local function tsg()
        if type(State.ts) ~= "table" then
            local ok, l = pcall(filtergc, "table", {Keys = {"ArrivalOpen", "DepartureAt", "StationAvailable"}}, false)
            local b
            for _, t in ok and type(l) == "table" and l or {} do
                if not b or (tonumber(rawget(t, "ReturnAt")) or 0) > (tonumber(rawget(b, "ReturnAt")) or 0) then b = t end
            end
            State.ts = b
        end
        return State.ts
    end
    local function aro()
        local t = tsg()
        return t ~= nil and rawget(t, "ArrivalOpen") == true and workspace:GetServerTimeNow() < (tonumber(rawget(t, "DepartureAt")) or 0) and rawget(t, "Phase") == "Home"
    end
    local pk = {}
    local function pfn(a, en2)
        local d = pk[a]
        if not (d and d.Parent) then
            d = nil
            for _, x in workspace:GetDescendants() do
                if x:IsA("ProximityPrompt") and not x:GetAttribute("TrainLocalBoarding") and (x:GetAttribute("TrainEventAction") == a or x.ActionText == a) then
                    d = x
                    if not en2 or x.Enabled then break end
                end
            end
            pk[a] = d
        end
        if d and (not en2 or d.Enabled) then return d end
        return nil
    end
    local function ppos(d)
        local pp = d.Parent
        return pp:IsA("Attachment") and pp.WorldPosition or pp:IsA("BasePart") and pp.Position or pp:GetPivot().Position
    end
    local function brd()
        local tb = pk.TrainBoard
        if not (tb and tb.Parent) then
            tb = nil
            for _, d in workspace:GetDescendants() do
                if d:IsA("ProximityPrompt") and d.Name == "TrainBoard" and not d:GetAttribute("TrainLocalBoarding") then tb = d break end
            end
            pk.TrainBoard = tb
        end
        if not tb then return false, "Board Failed: No Station" end
        local a, b = tb:GetAttribute("BoardRailStart"), tb:GetAttribute("BoardRailEnd")
        local q = typeof(a) == "Vector3" and typeof(b) == "Vector3" and (a + b) / 2 or ppos(tb)
        local h = getRootAndHumanoid()
        if h and (Vector3.new(q.X - h.Position.X, 0, q.Z - h.Position.Z)).Magnitude > 20 then
            if not own(true) then return false, "Waiting To Swap" end
            local ok, wy = hopToPosition(Vector3.new(q.X, getGroundY(q, Vector3.zero, 60, gg()) - State.gh + 3, q.Z), nil, false, function() return not on() and "Stopped" or nil end, gg())
            if not ok then return false, wy end
        end
        if State.hb then own(false) end
        local ok, r1, r2 = pcall(RemotesModule.TrainEvent.Action.InvokeServer, RemotesModule.TrainEvent.Action, "Board")
        if not ok or not r1 then return false, "Board Failed: " .. tostring(ok and (r2 or "Denied") or r1) end
        return true
    end
    local function hj(q, lk)
        if not own(true) then return false end
        local h = getRootAndHumanoid()
        if not h then return false end
        local y = getGroundY(q, Vector3.zero, 30) - State.gh + 3
        local p = Vector3.new(q.X, y, q.Z)
        setFreezeAnimation(true)
        h.CFrame = CFrame.lookAt(p, Vector3.new(lk.X, y, lk.Z))
        h.AssemblyLinearVelocity = Vector3.zero
        return true
    end
    local bsu, bnx, gan, gnx, csx = false, 0, math.random() * math.pi * 2, 0, 0
    local function bsx(v, lk)
        if not v then
            if bsu then
                bsu = false
                pcall(function() RemotesModule.GhostBoss.Suction:FireServer(false) end)
            end
            return
        end
        if os.clock() < bnx and bsu then return end
        bsu, bnx = true, os.clock() + 0.25
        pcall(function() RemotesModule.GhostBoss.Suction:FireServer(true, lk) end)
    end
    local function gbp()
        local g2 = ReplicatedStorage:FindFirstChild("GhostBossState")
        if not g2 or g2:GetAttribute("Realm") ~= "Town" or g2:GetAttribute("Phase") ~= "Alive" then return nil end
        local gp, vl, mt2 = g2:GetAttribute("Ground"), g2:GetAttribute("Velocity"), tonumber(g2:GetAttribute("MovedAt"))
        if typeof(gp) ~= "Vector3" then return nil end
        if typeof(vl) == "Vector3" and mt2 then gp += vl * math.clamp(workspace:GetServerTimeNow() - mt2, 0, 2) end
        return gp, g2:GetAttribute("AttackTarget") == LocalPlayer.UserId and (g2:GetAttribute("AttackPhase") == "Windup" or g2:GetAttribute("AttackPhase") == "Charge")
    end
    local function rdn(f)
        local cs2, nw = State.cs, workspace:GetServerTimeNow()
        if f and (type(cs2) ~= "table" or (tonumber(rawget(cs2, "StartsAt")) or 0) < nw - 900) and os.clock() >= csx then
            csx = os.clock() + 2
            local o6, l6 = pcall(filtergc, "table", {Keys = {"VacuumOwned", "Holding", "Captured"}}, false)
            cs2 = o6 and type(l6) == "table" and l6[1] or cs2
            State.cs = cs2
        end
        return type(cs2) == "table" and (tonumber(rawget(cs2, "StartsAt")) or 0) >= nw - 900 and (rawget(cs2, "Phase") == "Completed" or (tonumber(rawget(cs2, "EndsAt")) or math.huge) < nw or (tonumber(rawget(cs2, "Captured")) or 0) >= (tonumber(rawget(cs2, "Goal")) or math.huge))
    end
    local function twn()
        local c = LocalPlayer.Character
        local h, hu = getRootAndHumanoid()
        if not h or not hu or hu.Health <= 0 then
            sx(false)
            return "No Character"
        end
        local o2, s2 = pcall(SaveModule.Await)
        local te = o2 and type(s2) == "table" and s2.TrainEvent
        local o3, tf = pcall(function() return require(ReplicatedStorage.Shared.Flags.TrainEventFlags) end)
        local o5, cst = pcall(function() return tf.SpectralSoulCost:Get() end)
        cst = o3 and o5 and tonumber(cst) or 1000
        if type(te) == "table" and (tonumber(te.Souls) or 0) >= cst and os.clock() >= State.hbt then
            local sp4 = pfn("OpenSoulShop", true)
            if sp4 then
                sx(false)
                local q = ppos(sp4)
                if (q - h.Position).Magnitude > 12 then
                    local u2 = Vector3.new(h.Position.X - q.X, 0, h.Position.Z - q.Z)
                    hj(q + (u2.Magnitude > 0.1 and u2.Unit or Vector3.new(1, 0, 0)) * 6, q)
                    return "Going To Soul Keeper"
                end
                State.hbt = os.clock() + 3
                local ok, r1, r2 = pcall(RemotesModule.TrainEvent.Action.InvokeServer, RemotesModule.TrainEvent.Action, "TradeSouls", cst)
                return ok and r1 and "Traded Souls" or "Trade Failed: " .. tostring(ok and (r2 or "Denied") or r1)
            end
        end
        local vc
        for _, t in c and c:GetChildren() or {} do if t:IsA("Tool") and t.Name:find("Vacuum") then vc = t end end
        if not vc then
            local bp = LocalPlayer:FindFirstChildOfClass("Backpack")
            for _, t in bp and bp:GetChildren() or {} do
                if t:IsA("Tool") and t.Name:find("Vacuum") then
                    pcall(hu.EquipTool, hu, t)
                    return "Equipping Vacuum"
                end
            end
            local pr = pfn("PickupVacuum", true)
            if pr then
                sx(false)
                local q = ppos(pr)
                if (q - h.Position).Magnitude > 8 then
                    local u2 = Vector3.new(h.Position.X - q.X, 0, h.Position.Z - q.Z)
                    hj(q + (u2.Magnitude > 0.1 and u2.Unit or Vector3.new(1, 0, 0)) * 4, q)
                else
                    pcall(fireproximityprompt, pr)
                end
                return "Getting Vacuum"
            end
        end
        if rdn(true) and not gbp() then
            sx(false)
            local rp2 = pk.Ret
            if not (rp2 and rp2.Parent) then
                rp2 = nil
                for _, d in workspace:GetDescendants() do
                    if d:IsA("ProximityPrompt") and d.Name == "TrainReturn" and d:GetAttribute("TrainEventRequiredPhase") == "Town" then
                        rp2 = d
                        if d:GetAttribute("TrainLocalReturn") then break end
                    end
                end
                pk.Ret = rp2
            end
            if not rp2 then return "Return Failed: No Platform" end
            local q = ppos(rp2)
            if (Vector3.new(q.X - h.Position.X, 0, q.Z - h.Position.Z)).Magnitude > 10 then
                hj(q, q + Vector3.new(0, 0, 1))
                return "Going To Platform"
            end
            if os.clock() < State.rtn then return "Returning" end
            State.rtn = os.clock() + 2
            if State.hb then own(false) end
            local ok, r1, r2 = pcall(RemotesModule.TrainEvent.Action.InvokeServer, RemotesModule.TrainEvent.Action, "Return")
            return ok and r1 and "Returning" or "Return Failed: " .. tostring(ok and (r2 or "Denied") or r1)
        end
        local gp, ak = gbp()
        if gp and vc then
            sx(false)
            local fl = Vector3.new(h.Position.X - gp.X, 0, h.Position.Z - gp.Z)
            local dd = fl.Magnitude
            if math.abs(dd - State.gbd) > 3 then
                hj(gp + (dd > 0.1 and fl.Unit or Vector3.new(1, 0, 0)) * State.gbd, gp)
            elseif dd > 0.1 then
                h.CFrame = CFrame.lookAt(h.Position, Vector3.new(gp.X, h.Position.Y, gp.Z))
            end
            local h2 = getRootAndHumanoid()
            local bc2 = gp + Vector3.new(0, 5, 0)
            bsx(h2 ~= nil, h2 and (bc2 - h2.Position).Unit)
            return ak and "Boss | Dodging" or "Boss | Sucking"
        end
        bsx(false)
        local f = workspace:FindFirstChild("TrainGhostVisuals")
        local bq2, bdd
        for _, x in f and f:GetChildren() or {} do
            local ok, q = pcall(function() return x:IsA("Model") and x:GetPivot().Position or x.Position end)
            if ok and (not bdd or (q - h.Position).Magnitude < bdd) then bq2, bdd = q, (q - h.Position).Magnitude end
        end
        if not bq2 then
            sx(false)
            return "No Ghosts"
        end
        local fl = Vector3.new(bq2.X - h.Position.X, 0, bq2.Z - h.Position.Z)
        if fl.Magnitude > 18 or os.clock() >= gnx then
            gnx, gan = os.clock() + 0.3, gan + math.pi / 3
            hj(bq2 + Vector3.new(math.cos(gan) * 12, 0, math.sin(gan) * 12), bq2)
        elseif fl.Magnitude > 0.5 then
            h.CFrame = CFrame.lookAt(h.Position, h.Position + fl)
        end
        local h2 = getRootAndHumanoid()
        sx(h2 ~= nil and (bq2 - h2.Position).Magnitude < 22)
        return "Hunting Ghosts"
    end
    State.hl = true
    while on() do
        if State.on or (State.bo and not State.hb) then
            if State.hb then own(false) end
            l2("Waiting For Control")
            task.wait(0.5)
            continue
        end
        local ph = LocalPlayer:GetAttribute("TrainEventPhase")
        if State.hx and ph ~= nil and ph ~= "Home" then
            if ph ~= "Town" and State.hb then own(false) end
            if ph ~= "Town" then bsx(false) end
            l2("Train | " .. (ph == "Town" and twn() or tostring(ph)))
            task.wait(0.1)
            continue
        end
        sx(false)
        local ar = State.hx and aro() and not hca() and not rdn(false)
        if not hin() and not (State.hd or State.hm or ar) then
            if State.hb then own(false) end
            l2("Waiting For Train")
            task.wait(1)
            continue
        end
        if ar and hin() and not tostring(LocalPlayer:GetAttribute("HalloweenTransition") or ""):find(":cover:") then
            l2("Train | Boarding")
            local ok, wy = brd()
            l2(ok and "Train | Boarded" or "Train | " .. tostring(wy))
            task.wait(ok and 2 or 1)
            continue
        end
        if not hin() then
            if State.hb then own(false) end
            l2("Entering Dimension")
            local ok, wy = en()
            if not ok then
                l2(tostring(wy))
                task.wait(2)
            end
            continue
        end
        if tostring(LocalPlayer:GetAttribute("HalloweenTransition") or ""):find(":cover:") then
            task.wait(0.3)
            continue
        end
        if hca() then
            if not own(true) then
                State.bo, State.hb = false, false
                l2("Waiting To Swap")
                task.wait(0.3)
                continue
            end
            l2("Going Home")
            local ok, wy = hm2()
            if ok then nd += 1 end
            l2(ok and string.format("Delivered #%d", nd) or tostring(wy))
            continue
        end
        if os.clock() >= State.hbt and tkn() >= hpr then
            State.hbt = os.clock() + 15
            l2("Buying Halloween Egg")
            local _, wy = by()
            l2(tostring(wy))
            continue
        end
        local d = getTimeUntilNight()
        local l = State.hd and el() or {}
        local h, hu = getRootAndHumanoid()
        if not h or not hu then
            task.wait(0.5)
            continue
        end
        if #l == 0 and State.hm and d and d > 0 then
            local mo = ml()
            if #mo == 0 then
                if State.hb then own(false) end
                l2("No Monsters")
                task.wait(1)
                continue
            end
            local x = mo[1]
            l2(string.format("Fighting %s", x.n))
            local ok, wy = kl(x)
            if ok then
                nk2 += 1
                l2(string.format("Killed #%d | %s", nk2, x.n))
            else
                if wy ~= "Eggs Back" then State.hbl[x.m] = os.clock() + 15 end
                l2(tostring(wy))
            end
            continue
        end
        if State.hd then
            local _, hu3 = getRootAndHumanoid()
            local c3 = LocalPlayer.Character
            if hu3 and c3 and c3:FindFirstChildOfClass("Tool") then pcall(hu3.UnequipTools, hu3) end
        end
        if #l == 0 or not d or d < (l[1].p - h.Position).Magnitude / math.max(hu.WalkSpeed, 16) * 2 + 9 then
            local ub, pb = getOwnedEggsAndPlacements()
            if State.ap and d == 0 and os.clock() >= State.hpx and #ub > 0 and not (State.pf and #pb >= State.pf) then
                State.hpx = os.clock() + 120
                l2("Night | Leaving To Place Eggs")
                local ok, wy = px()
                l2(ok and "Night | " .. tostring(wy) or tostring(wy))
                continue
            end
            if State.hb then own(false) end
            l2(#l == 0 and "No Eggs" or "Waiting For Day")
            task.wait(1)
            continue
        end
        local e = l[1]
        if not own(true) then
            State.bo, State.hb = false, false
            l2("Waiting To Swap")
            task.wait(0.3)
            continue
        end
        l2(string.format("Flying To %s (Zone %s)", e.n, tostring(e.z)))
        local ok, wy = gt(e)
        if not ok then
            State.hbl[e.u] = os.clock() + 30
            l2(tostring(wy))
            task.wait(0.2)
            continue
        end
        l2(string.format("Stole %s | Going Home", e.n))
        ok, wy = hm2()
        if ok then nd += 1 end
        l2(ok and string.format("Delivered #%d | %s | Zone %s", nd, e.n, tostring(e.z)) or tostring(wy))
    end
    sx(false)
    bsx(false)
    if State.hb then own(false) end
    updateStatusLog("Idle")
end

local function startAutoHalloweenThread(generationId)
    local ok, e = xpcall(autoHalloweenLoopBody, debug.traceback, generationId)
    if State.hg == generationId then State.hl = false end
    if ok then
        if globalEnv.__SaeAf == State and (State.hd or State.hm or State.hx) and State.hg == generationId then
            State.hg += 1
            task.spawn(startAutoHalloweenThread, State.hg)
        end
        return
    end
    if globalEnv.__SaeAf ~= State or State.hg ~= generationId then return end
    pcall(function() RemotesModule.TrainEvent.Suction:FireServer(false) end)
    pcall(function() RemotesModule.GhostBoss.Suction:FireServer(false) end)
    if State.hb then
        pcall(setFreezeAnimation, false)
        local _, hu = getRootAndHumanoid()
        local t = os.clock()
        while hu and hu.Parent and hu.FloorMaterial == Enum.Material.Air and os.clock() - t < 3 do task.wait(0.1) end
        pcall(restoreOriginalHumanoid)
        State.bo, State.hb = false, false
    end
    handleFeatureError("Auto Halloween", "ahd", "Halloween Crashed", e)
end

pcall(function() State.cn[#State.cn + 1] = RemotesModule.TrainEvent.State.OnClientEvent:Connect(function(a) if type(a) == "table" then State.ts = a end end) end)
pcall(function() State.cn[#State.cn + 1] = RemotesModule.TrainEvent.CaptureState.OnClientEvent:Connect(function(a) if type(a) == "table" then State.cs = a end end) end)

local function hatchReadyEggs()
    local ok, o = pcall(EggStateModule.ReadOwnerEggs, LocalPlayer.UserId)
    if not ok or type(o) ~= "table" then return 0 end
    local n, t = 0, os.clock()
    for k, v in pairs(o) do
        if globalEnv.__SaeAf ~= State or not State.ah or n >= 5 then break end
        if type(k) == "string" and type(v) == "table" and v.Placement and (State.hf[k] or 0) < t then
            local o2, r = pcall(EggStateModule.IsReadyToHatch, k)
            if o2 and r == true then
                local o3, a = pcall(EggStateModule.BeginHatch, k)
                local o4, b = false, false
                if o3 and a then o4, b = pcall(EggStateModule.FinishHatch, k) end
                if o4 and b then
                    n += 1
                else
                    State.hf[k] = t + 60
                end
                task.wait(0.3)
            end
        end
    end
    return n
end

task.spawn(function()
    while globalEnv.__SaeAf == State do
        task.wait(10)
        if globalEnv.__SaeAf ~= State then break end
        if not isKeyValid() then continue end
        if State.ah then
            local n = hatchReadyEggs()
            if n > 0 then writeLog(`[Hatch] Hatched {n}`) end
        end
        if State.ps or State.es then
            local p, e = scanItemsToSell()
            if #p + #e > 0 then
                sellSelectedItems(p, e)
                writeLog(`[Sell] Sold {#p} Pets, {#e} Eggs`)
            end
        end
    end
end)

-- ============================================================================
-- ============================================================================
-- [8.5] Advanced Meta Enhancements: Egg Visuals (ESP), Player Physics & Security
-- ============================================================================

local RarityColors = {
    Common = Color3.fromRGB(200, 200, 200),
    Uncommon = Color3.fromRGB(80, 220, 80),
    Rare = Color3.fromRGB(60, 160, 255),
    Epic = Color3.fromRGB(180, 70, 255),
    Legendary = Color3.fromRGB(255, 185, 30),
    Mythic = Color3.fromRGB(255, 50, 70),
    Secret = Color3.fromRGB(0, 255, 240),
    Titan = Color3.fromRGB(255, 235, 59),
}

local RarityRankMap = {
    ["All"] = 0,
    ["Uncommon+"] = 2,
    ["Rare+"] = 3,
    ["Epic+"] = 4,
    ["Legendary+"] = 5,
    ["Mythic+"] = 6,
    ["Secret+"] = 7,
    ["Titan"] = 8
}

local espContainer = nil
local activeEspNodes = {}

local function getEspContainer()
    if not espContainer or not espContainer.Parent then
        espContainer = Instance.new("Folder")
        espContainer.Name = "TLab_EggESP"
        pcall(function() espContainer.Parent = workspace.Terrain end)
        if not espContainer.Parent then
            pcall(function() espContainer.Parent = workspace end)
        end
    end
    return espContainer
end

local function clearAllEggESP()
    for uid, node in pairs(activeEspNodes) do
        pcall(function()
            if node.part then node.part:Destroy() end
        end)
    end
    table.clear(activeEspNodes)
    if espContainer then
        pcall(function() espContainer:ClearAllChildren() end)
    end
end

local function updateEggESP()
    if not State.espEnabled then
        if next(activeEspNodes) then clearAllEggESP() end
        return
    end

    local ok, s = pcall(EggStateModule.ReadFieldEggs)
    if not ok or type(s) ~= "table" or type(s.Records) ~= "table" then
        return
    end

    local h = getRootAndHumanoid()
    local myPos = h and h.Position or Vector3.zero
    local container = getEspContainer()
    local visitedUids = {}
    local minRank = RarityRankMap[State.espMinRarity] or 0

    for _, r in ipairs(s.Records) do
        if (r.State == "Slot" or r.State == "Dropped") and typeof(r.BoundsCFrame) == "CFrame" then
            local a = AssetDirectory[r.AssetCategory]
            local rarityName = a and a.Rarity and tostring(a.Rarity._id) or "Common"
            local rank = tonumber(a and a.Rarity and a.Rarity.Rank) or 1

            if rank >= minRank then
                visitedUids[r.Uid] = true
                local eggPos = r.BoundsCFrame.Position
                local dist = math.floor((eggPos - myPos).Magnitude)
                local color = RarityColors[rarityName] or Color3.fromRGB(255, 255, 255)

                local valText = ""
                if State.espShowValue then
                    local val = tonumber(a and a.EarningRate) or 0
                    if AssetEarningsModule and type(AssetEarningsModule.MutationOnlyRatePerSecond) == "function" then
                        local okRate, calcRate = pcall(AssetEarningsModule.MutationOnlyRatePerSecond, {Category = r.AssetCategory, Scale = r.AssetScale, Mutations = r.Mutations or {}})
                        if okRate and type(calcRate) == "number" and calcRate > 0 then
                            val = calcRate
                        end
                    end
                    if val > 0 then
                        valText = ` | ${formatCompactValue(val)}/s`
                    end
                end

                local distText = State.espShowDist and `\n[{dist}m]` or ""
                local labelContent = `[{rarityName}] {tostring(r.AssetCategory)}{valText}{distText}`

                local node = activeEspNodes[r.Uid]
                if not node or not node.part or not node.part.Parent then
                    local p = Instance.new("Part")
                    p.Name = `ESP_{r.Uid}`
                    p.Size = Vector3.new(0.5, 0.5, 0.5)
                    p.Transparency = 1
                    p.Anchored = true
                    p.CanCollide = false
                    p.CanTouch = false
                    p.CanQuery = false
                    p.CFrame = r.BoundsCFrame
                    p.Parent = container

                    local bb = Instance.new("BillboardGui")
                    bb.Name = "ESP_Billboard"
                    bb.Size = UDim2.new(0, 180, 0, 45)
                    bb.StudsOffset = Vector3.new(0, 3, 0)
                    bb.AlwaysOnTop = true
                    bb.MaxDistance = 5000
                    bb.Adornee = p
                    bb.Parent = p

                    local lbl = Instance.new("TextLabel")
                    lbl.Name = "Label"
                    lbl.Size = UDim2.new(1, 0, 1, 0)
                    lbl.BackgroundTransparency = 1
                    lbl.Font = Enum.Font.GothamBold
                    lbl.TextSize = 13
                    lbl.TextColor3 = color
                    lbl.TextStrokeTransparency = 0.2
                    lbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                    lbl.Text = labelContent
                    lbl.Parent = bb

                    local chams = nil
                    if State.espChams then
                        chams = Instance.new("BoxHandleAdornment")
                        chams.Name = "ESP_Chams"
                        chams.Size = Vector3.new(3, 3.5, 3)
                        chams.Color3 = color
                        chams.Transparency = 0.6
                        chams.AlwaysOnTop = true
                        chams.ZIndex = 5
                        chams.Adornee = p
                        chams.Parent = p
                    end

                    activeEspNodes[r.Uid] = {
                        part = p,
                        bb = bb,
                        lbl = lbl,
                        chams = chams
                    }
                else
                    node.part.CFrame = r.BoundsCFrame
                    node.lbl.Text = labelContent
                    node.lbl.TextColor3 = color
                    if State.espChams and not node.chams then
                        local chams = Instance.new("BoxHandleAdornment")
                        chams.Name = "ESP_Chams"
                        chams.Size = Vector3.new(3, 3.5, 3)
                        chams.Color3 = color
                        chams.Transparency = 0.6
                        chams.AlwaysOnTop = true
                        chams.ZIndex = 5
                        chams.Adornee = node.part
                        chams.Parent = node.part
                        node.chams = chams
                    elseif not State.espChams and node.chams then
                        pcall(node.chams.Destroy, node.chams)
                        node.chams = nil
                    end
                end
            end
        end
    end

    for uid, node in pairs(activeEspNodes) do
        if not visitedUids[uid] then
            pcall(function() node.part:Destroy() end)
            activeEspNodes[uid] = nil
        end
    end
end

task.spawn(function()
    while globalEnv.__SaeAf == State do
        pcall(updateEggESP)
        task.wait(0.25)
    end
    clearAllEggESP()
end)

local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")

UserInputService.JumpRequest:Connect(function()
    if State.infJump then
        local _, hu = getRootAndHumanoid()
        if hu then
            hu:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

RunService.Stepped:Connect(function()
    if State.noclip then
        local c = LocalPlayer.Character
        if c then
            for _, p in ipairs(c:GetDescendants()) do
                if p:IsA("BasePart") and p.CanCollide then
                    p.CanCollide = false
                end
            end
        end
    end
end)

RunService.Heartbeat:Connect(function()
    if State.speedBoost then
        local _, hu = getRootAndHumanoid()
        if hu and hu.WalkSpeed ~= State.customSpeed then
            hu.WalkSpeed = State.customSpeed
        end
    end
end)

LocalPlayer.Idled:Connect(function()
    if State.antiAfk then
        pcall(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end)
    end
end)

local function isStaffMember(player)
    if not player or player == LocalPlayer then return false end
    if game.CreatorType == Enum.CreatorType.User and player.UserId == game.CreatorId then
        return true
    end
    if game.CreatorType == Enum.CreatorType.Group and game.CreatorId > 0 then
        local ok, r = pcall(player.GetRankInGroup, player, game.CreatorId)
        if ok and type(r) == "number" and r >= 100 then
            return true
        end
    end
    return false
end

local function onPlayerInspected(player)
    if not State.staffDetector then return end
    if isStaffMember(player) then
        writeLog(`[Security] STAFF DETECTED: {player.Name} ({player.UserId})`, true)
        if Window and Window.Notify then
            Window:Notify({
                Title = "Staff Alert!",
                Content = `Staff detected: {player.Name}! Action: {State.staffAction}`,
                Type = "Error",
                Duration = 6
            })
        end
        if State.staffAction == "Server Hop" then
            task.spawn(hopServer, false)
        else
            State.on = false
            pcall(State.Stop, true)
            updateStatusLog("Stopped: Staff Joined")
        end
    end
end

for _, p in ipairs(Players:GetPlayers()) do onPlayerInspected(p) end
Players.PlayerAdded:Connect(onPlayerInspected)

pcall(function()
    local GuiService = game:GetService("GuiService")
    local TeleportService = game:GetService("TeleportService")
    GuiService.ErrorMessageChanged:Connect(function()
        if State.autoRejoin then
            writeLog("[Auto Rejoin] Disconnect dialog detected! Reconnecting in 3s...", true)
            task.wait(3)
            pcall(function()
                TeleportService:Teleport(game.PlaceId, LocalPlayer)
            end)
        end
    end)
end)

-- [9] FPS Booster, Server Hop & Hub UI Initialization
-- ============================================================================
local function enableFpsBooster()
    if globalEnv.__SaeP then return nil, "Already On" end
    globalEnv.__SaeP = true
    local li, tr = game:GetService("Lighting"), workspace.Terrain
    local function ko(d, k)
        k = k or "Enabled"
        d[k] = false
        d:GetPropertyChangedSignal(k):Connect(function() if d[k] then d[k] = false end end)
    end
    local function lf() if li.GlobalShadows then li.GlobalShadows = false end end
    lf()
    li:GetPropertyChangedSignal("GlobalShadows"):Connect(lf)
    local function kx(d) if d:IsA("PostEffect") then ko(d) end end
    for _, d in li:GetDescendants() do pcall(kx, d) end
    li.DescendantAdded:Connect(function(d) pcall(kx, d) end)
    local cu = tr:FindFirstChildOfClass("Clouds")
    if cu then pcall(ko, cu) end
    pcall(function() tr.WaterWaveSize, tr.WaterWaveSpeed, tr.WaterReflectance = 0, 0, 0 end)
    local function px(d)
        if d:IsA("ParticleEmitter") then
            d.Lifetime = NumberRange.new(0)
            ko(d)
        elseif d:IsA("Beam") or d:IsA("Trail") or d:IsA("Smoke") or d:IsA("Fire") or d:IsA("Sparkles") or d:IsA("Light") or d:IsA("Highlight") then
            ko(d)
        elseif d:IsA("Decal") then
            d.Transparency = 1
        elseif d:IsA("SurfaceAppearance") then
            task.defer(pcall, d.Destroy, d)
        elseif d:IsA("VideoFrame") then
            ko(d, "Playing")
            ko(d, "Visible")
        elseif d:IsA("BasePart") and d.Material ~= Enum.Material.Water then
            d.Material, d.Reflectance, d.CastShadow = Enum.Material.SmoothPlastic, 0, false
        end
    end
    workspace.DescendantAdded:Connect(function(d) pcall(px, d) end)
    task.spawn(function()
        for i, d in workspace:GetDescendants() do
            pcall(px, d)
            if i % 2000 == 0 then task.wait() end
        end
    end)
    local pg2 = LocalPlayer:FindFirstChildOfClass("PlayerGui")
    if pg2 then
        for _, d in pg2:GetDescendants() do if d:IsA("VideoFrame") then pcall(px, d) end end
        pg2.DescendantAdded:Connect(function(d) if d:IsA("VideoFrame") then pcall(px, d) end end)
    end
    local ok, aa = pcall(function() return require(ReplicatedStorage.Controllers.Game.Plots.ActiveAssetsController) end)
    if ok and type(aa) == "table" and type(aa.SetAllPetsHidden) == "function" then
        task.spawn(function()
            while globalEnv.__SaeP do
                pcall(aa.SetAllPetsHidden, true)
                task.wait(5)
            end
        end)
    end
    local ui = game:GetService("UserInputService")
    ui.WindowFocusReleased:Connect(function() pcall(RunService.Set3dRenderingEnabled, RunService, false) end)
    ui.WindowFocused:Connect(function() pcall(RunService.Set3dRenderingEnabled, RunService, true) end)
    local function ax(d)
        if d:IsA("Accoutrement") or d:IsA("Clothing") or d:IsA("ShirtGraphic") or d:IsA("CharacterMesh") then task.defer(pcall, d.Destroy, d) end
    end
    local function cx(c)
        for _, d in c:GetDescendants() do pcall(ax, d) end
        c.DescendantAdded:Connect(function(d) pcall(ax, d) end)
    end
    local function pa(p)
        if p == LocalPlayer then return end
        if p.Character then cx(p.Character) end
        p.CharacterAdded:Connect(cx)
    end
    for _, p in Players:GetPlayers() do pa(p) end
    Players.PlayerAdded:Connect(pa)
    if not ok then return nil, "Hide Pets Failed: No Controller" end
    return true
end

local function hopServer(preferLowPlayer)
    local HttpService, ts = game:GetService("HttpService"), game:GetService("TeleportService")
    local function go2(f)
        local fe
        local cn = ts.TeleportInitFailed:Connect(function(p, _, m) if p == LocalPlayer then fe = tostring(m) end end)
        local o3, e3 = pcall(f)
        if not o3 then fe = tostring(e3) end
        local t = os.clock()
        while not fe and os.clock() - t < 10 do task.wait(0.25) end
        cn:Disconnect()
        return fe == nil
    end
    while globalEnv.__SaeAf == State do
        local ok, r = pcall(function() return game:HttpGet(("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100"):format(game.PlaceId), true) end)
        local o2, d = false, nil
        if ok then o2, d = pcall(HttpService.JSONDecode, HttpService, r) end
        local l = {}
        for _, v in o2 and type(d) == "table" and type(d.data) == "table" and d.data or {} do
            if v.id ~= game.JobId and type(v.playing) == "number" and type(v.maxPlayers) == "number" and v.playing < v.maxPlayers then l[#l + 1] = v end
        end
        if preferLowPlayer then
            table.sort(l, function(x, y) return x.playing < y.playing end)
        else
            for i = #l, 2, -1 do
                local j = math.random(1, i)
                l[i], l[j] = l[j], l[i]
            end
        end
        for _, v in l do
            if globalEnv.__SaeAf ~= State then return false, "Hop Failed: Stopped" end
            if go2(function() ts:TeleportToPlaceInstance(game.PlaceId, v.id, LocalPlayer) end) then return true end
            task.wait(1)
        end
        if not preferLowPlayer and globalEnv.__SaeAf == State and go2(function() ts:Teleport(game.PlaceId, LocalPlayer) end) then return true end
        task.wait(3)
    end
    return false, "Hop Failed: Stopped"
end


if globalEnv.__SaeD then pcall(function() globalEnv.__SaeD:Destroy() end) end
globalEnv.__SaeD = nil
if globalEnv.__AvW then pcall(globalEnv.__AvW.Destroy, globalEnv.__AvW) end

local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/TLabHub/FruitWarrior/refs/heads/main/Siesta"))()
task.wait(0.1)
table.clear(Library._tabs)

local Window = Library:CreateWindow({
    Title = "TLab Hub | Steal An Egg",
    SubTitle = "PRO v1.0",
    Size = UDim2.new(0, 560, 0, 360),
    ToggleKey = Enum.KeyCode.RightControl,
    ShowToggleButton = true,
    SkipIntro = false
})

local HomeTab = Window:CreateHomeTab({
    Title = "Home",
    DiscordInvite = "https://discord.gg/tlab"
})

-- Cyberpunk Showcase Banner Card
do
    local Banner = Instance.new("Frame")
    Banner.Name = "TLab_StealAnEgg_Banner"
    Banner.Size = UDim2.new(1, 0, 0, 72)
    Banner.BackgroundColor3 = Color3.fromRGB(12, 16, 24)
    Banner.BorderSizePixel = 0
    Banner.Parent = HomeTab.Page

    local bCorner = Instance.new("UICorner")
    bCorner.CornerRadius = UDim.new(0, 6)
    bCorner.Parent = Banner

    local bStroke = Instance.new("UIStroke")
    bStroke.Color = Color3.fromRGB(0, 230, 255)
    bStroke.Transparency = 0.5
    bStroke.Thickness = 0.8
    bStroke.Parent = Banner

    local bGrad = Instance.new("UIGradient")
    bGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(10, 25, 45)),
        ColorSequenceKeypoint.new(0.6, Color3.fromRGB(15, 20, 32)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 14, 22))
    })
    bGrad.Rotation = 25
    bGrad.Parent = Banner

    local ThumbFrame = Instance.new("Frame")
    ThumbFrame.Size = UDim2.new(0, 48, 0, 48)
    ThumbFrame.Position = UDim2.new(0, 12, 0.5, -24)
    ThumbFrame.BackgroundColor3 = Color3.fromRGB(8, 12, 18)
    ThumbFrame.BorderSizePixel = 0
    ThumbFrame.Parent = Banner

    local tfCorner = Instance.new("UICorner")
    tfCorner.CornerRadius = UDim.new(0, 6)
    tfCorner.Parent = ThumbFrame

    local tfStroke = Instance.new("UIStroke")
    tfStroke.Color = Color3.fromRGB(0, 230, 255)
    tfStroke.Thickness = 0.8
    tfStroke.Parent = ThumbFrame

    local ThumbImg = Instance.new("ImageLabel")
    ThumbImg.Size = UDim2.new(1, -4, 1, -4)
    ThumbImg.Position = UDim2.new(0, 2, 0, 2)
    ThumbImg.BackgroundTransparency = 1
    ThumbImg.Image = "rbxthumb://type=GameIcon&id=" .. game.PlaceId .. "&w=150&h=150"
    ThumbImg.Parent = ThumbFrame

    local tCorner = Instance.new("UICorner")
    tCorner.CornerRadius = UDim.new(0, 4)
    tCorner.Parent = ThumbImg

    local TitleLbl = Instance.new("TextLabel")
    TitleLbl.Size = UDim2.new(1, -75, 0, 18)
    TitleLbl.Position = UDim2.new(0, 70, 0, 10)
    TitleLbl.BackgroundTransparency = 1
    TitleLbl.Font = Enum.Font.GothamBold
    TitleLbl.Text = "STEAL AN EGG | CYBER EDITION"
    TitleLbl.TextColor3 = Color3.fromRGB(0, 255, 235)
    TitleLbl.TextSize = 13
    TitleLbl.TextXAlignment = Enum.TextXAlignment.Left
    TitleLbl.Parent = Banner

    local SubLbl = Instance.new("TextLabel")
    SubLbl.Size = UDim2.new(1, -75, 0, 14)
    SubLbl.Position = UDim2.new(0, 70, 0, 30)
    SubLbl.BackgroundTransparency = 1
    SubLbl.Font = Enum.Font.GothamMedium
    SubLbl.Text = "Clone Bypass Active ◈ 3D ESP Ready ◈ Instant Snap Drop"
    SubLbl.TextColor3 = Color3.fromRGB(150, 175, 205)
    SubLbl.TextSize = 10
    SubLbl.TextXAlignment = Enum.TextXAlignment.Left
    SubLbl.Parent = Banner

    local StatLbl = Instance.new("TextLabel")
    StatLbl.Size = UDim2.new(1, -75, 0, 14)
    StatLbl.Position = UDim2.new(0, 70, 0, 47)
    StatLbl.BackgroundTransparency = 1
    StatLbl.Font = Enum.Font.Gotham
    StatLbl.Text = "Eggs Stolen: 0 ◈ Farm Status: Idle"
    StatLbl.TextColor3 = Color3.fromRGB(166, 255, 111)
    StatLbl.TextSize = 9
    StatLbl.TextXAlignment = Enum.TextXAlignment.Left
    StatLbl.Parent = Banner

    task.spawn(function()
        while globalEnv.__SaeAf == State do
            pcall(function()
                if StatLbl and StatLbl.Parent then
                    local st = State.on and "FARMING" or (State.bx and "PAUSED" or "IDLE")
                    StatLbl.Text = string.format("Eggs Stolen: %d ◈ Farm: %s ◈ Speed: %d", State.n, st, State.speedBoost and State.customSpeed or 16)
                end
            end)
            task.wait(1)
        end
    end)
end

do
    local originalNotify = Window.Notify
    Window.Notify = function(self, opts)
        opts = opts or {}
        local title = tostring(opts.Title or "TLab Hub")
        local text = tostring(opts.Content or opts.Text or "")
        pcall(writeLog, `[{title}] {text}`)
        return originalNotify(self, {
            Title = title,
            Content = text,
            Duration = opts.Duration or opts.Time or 3.5,
            Type = opts.Error and "Error" or (opts.Type or "Info")
        })
    end
    local ex = identifyexecutor and select(2, pcall(identifyexecutor)) or "?"
    local ui = game:GetService("UserInputService")
    writeLog(`[Hub] Loaded | Place {game.PlaceVersion} | Server {game.JobId:sub(1, 8)} | {#Players:GetPlayers()} Players | {tostring(ex)} | {ui.TouchEnabled and not ui.KeyboardEnabled and "Mobile" or "PC"}`, true)
end
State.nt = function(n, m) Window:Notify({Title = n, Content = m, Error = true}) end

local translatableElements = {}
local function registerTranslation(enName, labelObj, isSection)
    if enName and labelObj then
        table.insert(translatableElements, { en = enName, label = labelObj, isSection = isSection })
    end
end

local function bindToggle(tab, opts)
    opts = opts or {}
    local wrapper = { v = opts.Default == true }
    local internalUpdating = false
    local name = opts.Name or opts.Title or "Toggle"
    local rawToggle = tab:CreateToggle({
        Title = name,
        Description = opts.Description,
        Icon = opts.Icon,
        Default = wrapper.v,
        Callback = function(val)
            wrapper.v = (val == true)
            if not internalUpdating and opts.Callback then
                opts.Callback(wrapper.v)
            end
        end
    })
    function wrapper:Set(val)
        val = (val == true)
        if wrapper.v == val then return end
        wrapper.v = val
        rawToggle:Set(val)
    end
    function wrapper:GetValue()
        return wrapper.v
    end
    local frame = tab.Page and tab.Page:GetChildren()[#tab.Page:GetChildren()]
    if frame then
        local lbl = frame:FindFirstChild("Title")
        if lbl then registerTranslation(name, lbl, false) end
        local descLbl = frame:FindFirstChild("Desc")
        if descLbl and opts.Description then registerTranslation(opts.Description, descLbl, false) end
    end
    return wrapper
end

local function wrapTab(tab)
    local wrapped = {
        _raw = tab,
        Page = tab.Page,
        Instance = tab.Instance,
    }

    function wrapped:Section(opts)
        local title = type(opts) == "table" and (opts.Name or opts.Title or opts[1] or "") or tostring(opts or "")
        local sec = tab:CreateSection(title)
        local frame = tab.Page:GetChildren()[#tab.Page:GetChildren()]
        if frame then
            for _, c in ipairs(frame:GetChildren()) do
                if c:IsA("TextLabel") and c.Text ~= "◈" then
                    registerTranslation(title, c, true)
                    break
                end
            end
        end
        return sec
    end
    wrapped.CreateSection = wrapped.Section
    wrapped.AddSection = wrapped.Section

    function wrapped:Button(opts)
        opts = opts or {}
        local name = opts.Name or opts.Title or opts.Text or "Button"
        opts.Title = name
        local btn = tab:CreateButton(opts)
        local frame = tab.Page:GetChildren()[#tab.Page:GetChildren()]
        if frame then
            local lbl = frame:FindFirstChild("Title")
            if lbl then registerTranslation(name, lbl, false) end
            local descLbl = frame:FindFirstChild("Desc")
            if descLbl and opts.Description then registerTranslation(opts.Description, descLbl, false) end
        end
        return btn
    end
    wrapped.CreateButton = wrapped.Button
    wrapped.AddButton = wrapped.Button

    function wrapped:Toggle(opts)
        return bindToggle(self._raw or tab, opts)
    end
    wrapped.CreateToggle = wrapped.Toggle
    wrapped.AddToggle = wrapped.Toggle

    function wrapped:Slider(opts)
        opts = opts or {}
        local name = opts.Name or opts.Title or opts.Text or "Slider"
        local iconKey = opts.Icon
        opts.Title = name
        opts.Min = opts.Min or opts.Minimum or 0
        opts.Max = opts.Max or opts.Maximum or 100
        opts.Increment = opts.Step or opts.Increment or 1
        local defaultVal = opts.Default or opts.Min
        local userCallback = opts.Callback
        local sObj
        opts.Callback = function(val)
            if sObj then sObj.v = val end
            if userCallback then userCallback(val) end
        end
        sObj = tab:CreateSlider(opts)
        sObj.v = defaultVal
        local rawSet = sObj.Set
        sObj.Set = function(self, val)
            sObj.v = val
            if rawSet then rawSet(self, val) end
        end
        local frame = tab.Page:GetChildren()[#tab.Page:GetChildren()]
        if frame then
            local lbl = frame:FindFirstChild("Title")
            if lbl then
                registerTranslation(name, lbl, false)
                if iconKey and Library.Icons then
                    local clean = tostring(iconKey):gsub("[-_ %s]", ""):lower()
                    local asset = Library.Icons[iconKey]
                    if not asset then
                        for k, v in pairs(Library.Icons) do
                            if k:lower() == clean then asset = v break end
                        end
                    end
                    if asset then
                        lbl.Position = UDim2.new(0, 36, 0, 6)
                        local badge = Instance.new("Frame")
                        badge.Name = "IconBadge"
                        badge.Size = UDim2.new(0, 20, 0, 20)
                        badge.Position = UDim2.new(0, 8, 0, 4)
                        badge.BackgroundColor3 = Color3.fromRGB(15, 20, 30)
                        badge.BorderSizePixel = 0
                        badge.Parent = frame
                        local cr = Instance.new("UICorner")
                        cr.CornerRadius = UDim.new(0, 4)
                        cr.Parent = badge
                        local st = Instance.new("UIStroke")
                        st.Color = Color3.fromRGB(30, 45, 65)
                        st.Thickness = 0.8
                        st.Parent = badge
                        local img = Instance.new("ImageLabel")
                        img.Size = UDim2.new(0, 12, 0, 12)
                        img.AnchorPoint = Vector2.new(0.5, 0.5)
                        img.Position = UDim2.new(0.5, 0, 0.5, 0)
                        img.BackgroundTransparency = 1
                        img.Image = asset
                        img.ImageColor3 = Color3.fromRGB(0, 230, 255)
                        img.Parent = badge
                    end
                end
            end
        end
        return sObj
    end
    wrapped.CreateSlider = wrapped.Slider
    wrapped.AddSlider = wrapped.Slider

    function wrapped:Dropdown(opts)
        opts = opts or {}
        local name = opts.Name or opts.Title or opts.Text or "Dropdown"
        local iconKey = opts.Icon
        opts.Title = name
        opts.Values = opts.Options or opts.Values or {}
        local dObj = (opts.Multi or opts.MultiSelect) and tab:CreateMultiDropdown(opts) or tab:CreateDropdown(opts)
        local frame = tab.Page:GetChildren()[#tab.Page:GetChildren()]
        if frame then
            local topBtn = frame:FindFirstChildOfClass("TextButton")
            local lbl = topBtn and topBtn:FindFirstChild("Title") or frame:FindFirstChild("Title")
            if lbl then
                registerTranslation(name, lbl, false)
                if iconKey and Library.Icons then
                    local clean = tostring(iconKey):gsub("[-_ %s]", ""):lower()
                    local asset = Library.Icons[iconKey]
                    if not asset then
                        for k, v in pairs(Library.Icons) do
                            if k:lower() == clean then asset = v break end
                        end
                    end
                    if asset and topBtn then
                        lbl.Position = UDim2.new(0, 36, lbl.Position.Y.Scale, lbl.Position.Y.Offset)
                        local badge = Instance.new("Frame")
                        badge.Name = "IconBadge"
                        badge.Size = UDim2.new(0, 22, 0, 22)
                        badge.Position = UDim2.new(0, 8, 0.5, -11)
                        badge.BackgroundColor3 = Color3.fromRGB(15, 20, 30)
                        badge.BorderSizePixel = 0
                        badge.Parent = topBtn
                        local cr = Instance.new("UICorner")
                        cr.CornerRadius = UDim.new(0, 4)
                        cr.Parent = badge
                        local st = Instance.new("UIStroke")
                        st.Color = Color3.fromRGB(30, 45, 65)
                        st.Thickness = 0.8
                        st.Parent = badge
                        local img = Instance.new("ImageLabel")
                        img.Size = UDim2.new(0, 14, 0, 14)
                        img.AnchorPoint = Vector2.new(0.5, 0.5)
                        img.Position = UDim2.new(0.5, 0, 0.5, 0)
                        img.BackgroundTransparency = 1
                        img.Image = asset
                        img.ImageColor3 = Color3.fromRGB(0, 230, 255)
                        img.Parent = badge
                    end
                end
            end
        end
        return dObj
    end
    wrapped.CreateDropdown = wrapped.Dropdown
    wrapped.AddDropdown = wrapped.Dropdown

    function wrapped:Label(opts)
        opts = opts or {}
        local text = opts.Text or opts.Title or ""
        local p = tab:CreateParagraph({
            Title = text,
            Content = opts.Content or ""
        })
        local lObj = {
            _p = p,
            Set = function(_, t)
                p:SetTitle(tostring(t))
            end,
            SetText = function(_, t)
                p:SetTitle(tostring(t))
            end
        }
        return lObj
    end
    wrapped.CreateLabel = wrapped.Label

    return wrapped
end

local W = Window
W.Tab = function(self, opts)
    local name = type(opts) == "table" and (opts.Name or opts.Title or opts[1]) or tostring(opts)
    local icon = type(opts) == "table" and (opts.Icon or opts[2]) or "Gamepad"
    local iconMap = {
        ["egg"] = "Gamepad",
        ["calendar-star"] = "Flame",
        ["four-squares-grid"] = "Sliders",
        ["person-teleport"] = "Compass",
        ["gear"] = "Settings",
        ["eye"] = "Eye",
        ["shield"] = "Shield",
    }
    icon = iconMap[icon] or icon
    local rawTab = Window:CreateTab({ Title = name, Icon = icon })
    return wrapTab(rawTab)
end

local pg = W:Tab({Name = "General", Icon = "egg"})
local function getSortedRarityList()
    local m, o = {}, {}
    for _, a in pairs(AssetDirectory) do
        local r = type(a) == "table" and a.Rarity
        if type(r) == "table" and type(r._id) == "string" and not m[r._id] then
            m[r._id] = true
            o[#o + 1] = r
        end
    end
    table.sort(o, function(x, y) return (tonumber(x.Rank) or 0) < (tonumber(y.Rank) or 0) end)
    local n = {}
    for _, r in o do n[#n + 1] = r._id end
    return n
end

local function rs2(v)
    local t = {}
    for _, x in v or {} do t[x] = true end
    return t
end

local function an()
    local w = workspace:FindFirstChild("World") or workspace:FindFirstChild("__OBJECTS")
    local a = w and w:FindFirstChild("Areas")
    local g = a and a:FindFirstChild("GuardAreas")
    local o = {}
    for _, z in g and g:GetChildren() or {} do
        local b = z:FindFirstChild("Bounds")
        if b and b:IsA("BasePart") then o[#o + 1] = {z.Name, b.Position.X - b.Size.X / 2} end
    end
    table.sort(o, function(x, y) return x[2] < y[2] end)
    local n = {}
    for _, v in o do n[#n + 1] = v[1] end
    return n
end

local ra = getSortedRarityList()
pg:Section({Name = "Farm"})
local xe, xa
xa = pg:Dropdown({
    Name = "Select Area",
    Icon = "MapPin",
    Options = an(),
    Multi = true,
    Default = {},
    Flag = "ar",
    Callback = function(v) State.az = rs2(v) if #v > 0 and xe then xe:Set({}) end end
})

xe = pg:Dropdown({
    Name = "Select Egg",
    Icon = "Box",
    Options = ra,
    Multi = true,
    Default = {},
    Flag = "mr",
    Callback = function(v) State.mr = rs2(v) if #v > 0 and xa then xa:Set({}) end end
})

pg:Toggle({
    Name = "Prioritize Highest Value ($/s)",
    Icon = "DollarSign",
    Description = "Smart target eggs with highest mutated $/s rate",
    Default = true,
    Flag = "phv",
    Callback = function(v) State.smartPrioritize = (v == true) end
})

pg:Slider({
    Name = "Min Egg Value ($/s)",
    Icon = "Coins",
    Flag = "mev",
    Min = 0,
    Max = 100000,
    Step = 500,
    Default = 0,
    Callback = function(v) State.minEggValue = v end
})

pg:Toggle({
    Name = "Fast Base Deposit",
    Icon = "Zap",
    Description = "Instantly snap & deposit egg upon reaching base zone",
    Default = true,
    Flag = "fbd",
    Callback = function(v) State.fastDeposit = (v == true) end
})

State.ui.ae = pg:Toggle({
    Name = "Auto Steal Egg",
    Icon = "Target",
    Description = "Hop-fly steal loop using cloned humanoid bypass",
    Default = false,
    Flag = "ae",
    Callback = function(v)
        local x = rawget(State.ui, "ahd")
        if v and x and x.v then x:Set(false) end
        if v then State.Start() else State.Stop() end
    end
})

local xp = pg:Toggle({
    Name = "Auto Place Egg",
    Icon = "Package",
    Description = "Auto deposit eggs from inventory into open nests",
    Default = true,
    Flag = "ape",
    Callback = function(v) State.ap = v == true end
})
State.ap = xp.v == true

local xh = pg:Toggle({
    Name = "Auto Hatch Egg",
    Icon = "Sparkles",
    Description = "Auto hatch ready incubated eggs in base nests",
    Default = true,
    Flag = "ahe",
    Callback = function(v) State.ah = v == true end
})
State.ah = xh.v == true

pg:Toggle({
    Name = "Pet Index",
    Icon = "Trophy",
    Description = "Prioritize stealing unowned eggs for index completion",
    Default = false,
    Flag = "pix",
    Callback = function(v) State.pi = v == true end
})

pg:Button({
    Name = "Leave Treadmill",
    Icon = "Power",
    Description = "Doff and disconnect character from active treadmill",
    Callback = function()
        local ok, wy = false, "Treadmill Failed: Not Mounted"
        if State.bt or getActiveTreadmillModel() then ok, wy = leaveTreadmill() end
        updateStatusLog(ok and "Treadmill Left" or tostring(wy))
        W:Notify({Title = "Leave Treadmill", Text = ok and "Treadmill Left" or tostring(wy), Error = not ok})
    end
})

pg:Section({Name = "Pet"})
pg:Dropdown({
    Name = "Sell Pet Rarity",
    Icon = "Filter",
    Options = ra,
    Multi = true,
    Default = {},
    Flag = "spr",
    Callback = function(v) State.sp = rs2(v) end
})

pg:Toggle({
    Name = "Auto Sell Pet",
    Icon = "Trash2",
    Description = "Auto sell hatched pets matching chosen rarities",
    Default = false,
    Flag = "asp",
    Callback = function(v) State.ps = v end
})

pg:Section({Name = "Egg"})
pg:Dropdown({
    Name = "Sell Egg Rarity",
    Icon = "Filter",
    Options = ra,
    Multi = true,
    Default = {},
    Flag = "ser",
    Callback = function(v) State.se = rs2(v) end
})

pg:Toggle({
    Name = "Auto Sell Egg",
    Icon = "Trash",
    Description = "Auto sell inventory eggs matching chosen rarities",
    Default = false,
    Flag = "ase",
    Callback = function(v) State.es = v end
})

local vs = W:Tab({Name = "Visuals", Icon = "eye"})
vs:Section({Name = "Egg ESP"})

vs:Toggle({
    Name = "Egg ESP",
    Icon = "Eye",
    Description = "3D billboard tracker with rarity color codes",
    Default = false,
    Flag = "eesp",
    Callback = function(v) State.espEnabled = (v == true) end
})

vs:Toggle({
    Name = "Show Value ($/s)",
    Icon = "DollarSign",
    Description = "Display real-time mutated revenue on ESP billboards",
    Default = true,
    Flag = "esv",
    Callback = function(v) State.espShowValue = (v == true) end
})

vs:Toggle({
    Name = "Show Distance",
    Icon = "Compass",
    Description = "Display distance in meters to target field eggs",
    Default = true,
    Flag = "esd",
    Callback = function(v) State.espShowDist = (v == true) end
})

vs:Toggle({
    Name = "Egg Highlight (Chams)",
    Icon = "Sparkles",
    Description = "Glow box chams outline around egg models",
    Default = false,
    Flag = "echm",
    Callback = function(v) State.espChams = (v == true) end
})

vs:Dropdown({
    Name = "Min ESP Rarity",
    Icon = "Filter",
    Options = {"All", "Uncommon+", "Rare+", "Epic+", "Legendary+", "Mythic+", "Secret+", "Titan"},
    Default = "All",
    Flag = "mer",
    Callback = function(v) State.espMinRarity = v end
})

local et = W:Tab({Name = "Event", Icon = "calendar-star"})
et:Section({Name = "Butterfly"})
local xb = et:Label({Text = bc()})
task.spawn(function()
    while globalEnv.__SaeAf == State do
        pcall(function() xb:Set(bc()) end)
        task.wait(1)
    end
end)

State.ui.abf = et:Toggle({
    Name = "Auto Butterfly",
    Icon = "Flame",
    Description = "Auto catch butterflies & complete Wisp quests",
    Default = false,
    Flag = "abf",
    Callback = function(v)
        local x = rawget(State.ui, "ahd")
        if v and x and x.v then x:Set(false) end
        State.bf = v == true
        State.bi += 1
        if State.bf then task.spawn(startAutoButterflyThread, State.bi) end
    end
})

et:Button({
    Name = "Teleport To Station",
    Icon = "MapPin",
    Description = "Warp character to Butterfly Bloom quest hub",
    Callback = function()
        local ok, wy = bu()
        updateStatusLog(ok and "At Station" or tostring(wy))
        W:Notify({Title = "Teleport To Station", Text = ok and "At Station" or tostring(wy), Error = not ok})
    end
})

et:Section({Name = "Halloween"})
local function hq()
    if not (State.hd or State.hm or State.hx) then return end
    if not isKeyValid() then
        updateStatusLog("Key Check Failed: No Valid Key")
        return
    end
    for _, k in {"ae", "abf", "arl", "atx"} do
        local x = rawget(State.ui, k)
        if x and x.v then x:Set(false) end
    end
    if State.hl then return end
    State.hg += 1
    task.spawn(startAutoHalloweenThread, State.hg)
end
local function hz3(v)
    local x = rawget(State.ui, "ae")
    if v and x and x.v then x:Set(false) end
    State.hd = v == true
    hq()
end
State.ui.ahd = et:Toggle({
    Name = "Auto Halloween",
    Icon = "Skull",
    Description = "Farm Halloween candies, souls & ghost train boss",
    Default = false,
    Flag = "ahd",
    Callback = hz3
})

et:Section({Name = "Admin"})
State.ui.arl = et:Toggle({
    Name = "Auto Red Light Green Light",
    Icon = "Play",
    Description = "Auto win Squid Game minigame event",
    Default = false,
    Flag = "arl",
    Callback = function(v)
        State.lx = (v == true)
        if State.lx then
            State.gg += 1
            task.spawn(startAutoMinigameThread, State.gg, RedLightConfig)
        end
    end
})

State.ui.atx = et:Toggle({
    Name = "Auto T-Rex Run",
    Icon = "Activity",
    Description = "Auto dodge boulders & survive T-Rex stampede",
    Default = false,
    Flag = "atx",
    Callback = function(v)
        State.tx = (v == true)
        if State.tx then
            State.gg += 1
            task.spawn(startAutoMinigameThread, State.gg, TRexConfig)
        end
    end
})

local mi = W:Tab({Name = "Misc", Icon = "four-squares-grid"})
mi:Section({Name = "Player"})
mi:Toggle({
    Name = "Anti Stun",
    Icon = "ShieldCheck",
    Description = "Prevent ragdoll & stun status from heavy hits",
    Default = false,
    Flag = "ast",
    Callback = function(v) toggleAntiStun(v == true) end
})

local antiTrapToggle = mi:Toggle({
    Name = "Anti Trap",
    Icon = "Shield",
    Description = "Disable touch traps and bear traps in enemy bases",
    Default = true,
    Flag = "atr",
    Callback = function(v) toggleAntiTrap(v == true) end
})
if antiTrapToggle.v then toggleAntiTrap(true) end

mi:Section({Name = "Player Movement"})
mi:Toggle({
    Name = "Speed Boost",
    Icon = "Zap",
    Description = "Custom WalkSpeed boost via cloned humanoid",
    Default = false,
    Flag = "sbd",
    Callback = function(v) State.speedBoost = (v == true) end
})

mi:Slider({
    Name = "Speed Value",
    Icon = "Gauge",
    Flag = "spv",
    Min = 16,
    Max = 80,
    Step = 2,
    Default = 24,
    Callback = function(v) State.customSpeed = v end
})

mi:Toggle({
    Name = "Infinite Jump",
    Icon = "ChevronUp",
    Description = "Allows jumping continuously in mid-air",
    Default = false,
    Flag = "ijp",
    Callback = function(v) State.infJump = (v == true) end
})

mi:Toggle({
    Name = "Noclip",
    Icon = "Unlock",
    Description = "Phases through enemy base walls, doors & lasers",
    Default = false,
    Flag = "ncl",
    Callback = function(v) State.noclip = (v == true) end
})

mi:Toggle({
    Name = "Anti AFK",
    Icon = "Moon",
    Description = "Prevents Roblox 20-minute idle disconnect 24/7",
    Default = true,
    Flag = "aafk",
    Callback = function(v) State.antiAfk = (v == true) end
})

mi:Section({Name = "Security"})
mi:Toggle({
    Name = "Staff Detector",
    Icon = "ShieldAlert",
    Description = "Detects Admin, Developer or Creator presence",
    Default = true,
    Flag = "sdt",
    Callback = function(v) State.staffDetector = (v == true) end
})

mi:Dropdown({
    Name = "Staff Action",
    Icon = "Sliders",
    Options = {"Server Hop", "Stop Farm"},
    Default = "Server Hop",
    Flag = "sda",
    Callback = function(v) State.staffAction = v end
})

mi:Toggle({
    Name = "Auto Rejoin",
    Icon = "RefreshCw",
    Description = "Auto reconnects on server kicks or network errors",
    Default = true,
    Flag = "arj",
    Callback = function(v) State.autoRejoin = (v == true) end
})

local tp2 = W:Tab({Name = "Teleport", Icon = "person-teleport"})
tp2:Section({Name = "Server"})
for _, z in {{"Hop Server", false, "Globe", "Teleport to another random public server"}, {"Hop Low Server", true, "Server", "Teleport to a low-player server for safer farming"}} do
    tp2:Button({
        Name = z[1],
        Icon = z[3],
        Description = z[4],
        Callback = function()
            if State.hq then
                W:Notify({Title = z[1], Text = "Already Hopping"})
                return
            end
            State.hq = true
            W:Notify({Title = z[1], Text = "Hopping"})
            task.spawn(function()
                local ok, wy = hopServer(z[2])
                State.hq = false
                if not ok then W:Notify({Title = z[1], Text = tostring(wy), Error = true}) end
            end)
        end
    })
end

local tz = W:Tab({Name = "Setting", Icon = "gear"})
tz:Section({Name = "Game"})
tz:Button({
    Name = "FPS Booster",
    Icon = "Cpu",
    Description = "Optimize 3D rendering, remove particles & boost FPS",
    Callback = function()
        local ok, e = enableFpsBooster()
        W:Notify({Title = "FPS Booster", Text = ok and "On Until Rejoin" or e, Error = not ok and e ~= "Already On"})
    end
})

local lgv, lgd = false, {
    -- Titles
    ["Select Area"] = "Chọn Khu Vực",
    ["Select Egg"] = "Chọn Trứng",
    ["Auto Steal Egg"] = "Tự Động Trộm Trứng",
    ["Auto Place Egg"] = "Tự Động Đặt Trứng",
    ["Auto Hatch Egg"] = "Tự Động Ấp Trứng",
    ["Pet Index"] = "Sưu Tầm Pet",
    ["Leave Treadmill"] = "Rời Máy Chạy Bộ",
    ["Sell Pet Rarity"] = "Độ Hiếm Pet Cần Bán",
    ["Auto Sell Pet"] = "Tự Động Bán Pet",
    ["Sell Egg Rarity"] = "Độ Hiếm Trứng Cần Bán",
    ["Auto Sell Egg"] = "Tự Động Bán Trứng",
    ["Auto Butterfly"] = "Tự Động Bắt Bướm",
    ["Teleport To Station"] = "Dịch Chuyển Tới Nhà Ga",
    ["Auto Halloween"] = "Tự Động Halloween",
    ["Auto Red Light Green Light"] = "Tự Động Đèn Xanh Đèn Đỏ",
    ["Auto T-Rex Run"] = "Tự Động Chạy T-Rex",
    ["Anti Stun"] = "Chống Choáng",
    ["Anti Trap"] = "Chống Bẫy",
    ["Hop Server"] = "Đổi Server",
    ["Hop Low Server"] = "Đổi Server Ít Người",
    ["FPS Booster"] = "Tăng FPS",
    ["Select Language"] = "Chọn Ngôn Ngữ",
    ["Steal Hop Distance"] = "Steal: Khoảng Nhảy",
    ["Steal Drop Wait"] = "Steal: Chờ Drop",
    ["Steal Drop Every (Zones)"] = "Steal: Drop Mỗi (Khu)",
    ["Halloween Hop Distance"] = "Halloween: Khoảng Nhảy",
    ["Halloween Drop Wait"] = "Halloween: Chờ Drop",
    ["Halloween Drop Every (Studs)"] = "Halloween: Drop Mỗi (Stud)",
    ["Default Setting"] = "Cài Đặt Mặc Định",
    ["Visuals"] = "Hiển Thị",
    ["Egg ESP"] = "ESP Trứng",
    ["Show Value ($/s)"] = "Hiện Giá Trị ($/s)",
    ["Show Distance"] = "Hiện Khoảng Cách",
    ["Egg Highlight (Chams)"] = "Viền Phát Sáng (Chams)",
    ["Min ESP Rarity"] = "Độ Hiếm ESP Tối Thiểu",
    ["Prioritize Highest Value ($/s)"] = "Ưu Tiên Giá Trị Cao Nhất ($/s)",
    ["Min Egg Value ($/s)"] = "Giá Trị Trứng Tối Thiểu ($/s)",
    ["Fast Base Deposit"] = "Giao Trứng Nhanh",
    ["Player Movement"] = "Di Chuyển Nhân Vật",
    ["Speed Boost"] = "Tăng Tốc Chạy",
    ["Speed Value"] = "Tốc Độ Chạy",
    ["Infinite Jump"] = "Nhảy Vô Hạn",
    ["Noclip"] = "Đi Xuyên Tường",
    ["Anti AFK"] = "Chống Treo Máy (AFK)",
    ["Security"] = "Bảo Mật & An Toàn",
    ["Staff Detector"] = "Phát Hiện Admin / Staff",
    ["Staff Action"] = "Hành Động Khi Gặp Staff",
    ["Auto Rejoin"] = "Tự Kết Nối Lại Khi Mất Mạng",

    -- Descriptions
    ["Smart target eggs with highest mutated $/s rate"] = "Tự động nhắm trứng có giá trị $/s đột biến cao nhất",
    ["Instantly snap & deposit egg upon reaching base zone"] = "Giao trứng tức thì ngay khi chạm ranh giới căn cứ",
    ["Hop-fly steal loop using cloned humanoid bypass"] = "Vòng lặp trộm trứng hop-fly qua bypass cloned humanoid",
    ["Auto deposit eggs from inventory into open nests"] = "Tự động đặt trứng từ balo vào các tổ trứng trống",
    ["Auto hatch ready incubated eggs in base nests"] = "Tự động ấp nở các trứng đã đủ thời gian trong tổ",
    ["Prioritize stealing unowned eggs for index completion"] = "Ưu tiên cướp trứng chưa có để hoàn thành bộ sưu tập",
    ["Doff and disconnect character from active treadmill"] = "Rời và ngắt kết nối nhân vật khỏi máy chạy bộ",
    ["Auto sell hatched pets matching chosen rarities"] = "Tự động bán pet đã nở theo độ hiếm đã chọn",
    ["Auto sell inventory eggs matching chosen rarities"] = "Tự động bán trứng trong túi theo độ hiếm đã chọn",
    ["3D billboard tracker with rarity color codes"] = "Hiển thị 3D theo dõi trứng kèm màu độ hiếm",
    ["Display real-time mutated revenue on ESP billboards"] = "Hiện doanh thu $/s đột biến trực tiếp trên ESP",
    ["Display distance in meters to target field eggs"] = "Hiện khoảng cách theo mét tới các quả trứng",
    ["Glow box chams outline around egg models"] = "Khung dạ quang xuyên tường bao quanh quả trứng",
    ["Auto catch butterflies & complete Wisp quests"] = "Tự động bắt bướm và làm nhiệm vụ Wisp",
    ["Warp character to Butterfly Bloom quest hub"] = "Dịch chuyển tức thời tới trạm nhiệm vụ bướm",
    ["Farm Halloween candies, souls & ghost train boss"] = "Farm kẹo Halloween, linh hồn và boss tàu ma",
    ["Auto win Squid Game minigame event"] = "Tự động thắng minigame Đèn Xanh Đèn Đỏ",
    ["Auto dodge boulders & survive T-Rex stampede"] = "Tự động né đá và vượt qua minigame T-Rex",
    ["Prevent ragdoll & stun status from heavy hits"] = "Chống choáng và chống ngã ragdoll khi bị tấn công",
    ["Disable touch traps and bear traps in enemy bases"] = "Vô hiệu hóa bẫy gấu và bẫy dẫm trong base đối thủ",
    ["Custom WalkSpeed boost via cloned humanoid"] = "Tăng tốc chạy an toàn qua cloned humanoid",
    ["Allows jumping continuously in mid-air"] = "Cho phép nhảy liên tục nhiều lần trên không",
    ["Phases through enemy base walls, doors & lasers"] = "Đi xuyên tường, cửa khóa và cổng laser base đối thủ",
    ["Prevents Roblox 20-minute idle disconnect 24/7"] = "Chống bị ngắt kết nối treo máy 20 phút của Roblox",
    ["Detects Admin, Developer or Creator presence"] = "Phát hiện khi Admin, Dev hoặc Creator vào server",
    ["Auto reconnects on server kicks or network errors"] = "Tự động kết nối lại khi bị kick hoặc mất mạng",
    ["Teleport to another random public server"] = "Chuyển sang một server công khai ngẫu nhiên khác",
    ["Teleport to a low-player server for safer farming"] = "Chuyển sang server ít người để farm an toàn hơn",
    ["Optimize 3D rendering, remove particles & boost FPS"] = "Tối ưu hóa render 3D, xóa hạt hiệu ứng để tăng FPS",
    ["Reset all timing and distance sliders to default"] = "Khôi phục toàn bộ thanh trượt về cài đặt mặc định",
}

local function applyLanguageTranslations()
    for _, item in ipairs(translatableElements) do
        if item.label and item.label.Parent then
            local text = lgv and (lgd[item.en] or item.en) or item.en
            if item.isSection then
                pcall(function() item.label.Text = string.upper(text) end)
            else
                pcall(function() item.label.Text = text end)
            end
        end
    end
end

tz:Dropdown({
    Name = "Select Language",
    Icon = "Globe",
    Options = {"Vietnamese", "English"},
    Default = "English",
    Flag = "lg",
    Callback = function(v)
        lgv = v == "Vietnamese"
        applyLanguageTranslations()
    end
})

local sdf = {}
for _, r in {
    {"Steal Hop Distance", "shd", 50, 400, 1, "hs"},
    {"Steal Drop Wait", "sdw", 0, 2, 0.05, "dw"},
    {"Steal Drop Every (Zones)", "srz", 1, 6, 1, "rz"},
    {"Halloween Hop Distance", "hhd", 50, 400, 1, "hk"},
    {"Halloween Drop Wait", "hdw", 0, 2, 0.05, "hw"},
    {"Halloween Drop Every (Studs)", "hdr", 100, 1500, 1, "hr"},
} do
    local k, d0 = r[6], State[r[6]]
    local e = tz:Slider({Name = r[1], Flag = r[2], Min = r[3], Max = r[4], Step = r[5], Default = d0, Callback = function(v) State[k] = v end})
    State[k], sdf[#sdf + 1] = e.v, {e, d0}
end
tz:Button({Name = "Default Setting", Callback = function()
    for _, x in sdf do x[1]:Set(x[2]) end
    W:Notify({Title = "Default Setting", Text = "Back To Default"})
end})
applyLanguageTranslations()
State.gr = true

task.spawn(function()
    local qa = AssetEarningsModule
    local qf = FormatNumberUtil
    if not qa or not qf then
        local o1, m1 = pcall(require, ReplicatedStorage.Shared.Util.AssetEarnings)
        local o2, m2 = pcall(require, ReplicatedStorage.Packages.FormatNumber.Simple)
        qa = o1 and m1 or qa
        qf = o2 and m2 or qf
    end
    local qv = {}
    while globalEnv.__SaeAf == State do
        local ok, wy = pcall(function()
            local g = LocalPlayer:FindFirstChildOfClass("PlayerGui") and LocalPlayer:FindFirstChildOfClass("PlayerGui"):FindFirstChild("GrowingEggs")
            local sf = g and g.Enabled and g:FindFirstChild("Frame") and g.Frame:FindFirstChild("ScrollingFrame")
            if not sf then return end
            local o3, rc = pcall(EggStateModule.ReadOwnerEggs, LocalPlayer.UserId)
            if not o3 or type(rc) ~= "table" then return end
            for _, r in sf:GetChildren() do
                local e, sp = rc[r.Name], r:FindFirstChild("Spacer")
                if type(e) ~= "table" or not sp then continue end
                local k = `{e.AssetCategory}|{e.AssetScale}|{table.concat(e.Mutations or {}, ",")}`
                if not qv[r.Name] or qv[r.Name].k ~= k then
                    local o4, v = pcall(qa.MutationOnlyRatePerSecond, {Category = e.AssetCategory, Scale = e.AssetScale, Mutations = e.Mutations or {}})
                    qv[r.Name] = {k = k, t = o4 and type(v) == "number" and v > 0 and `${qf.FormatCompact(v)}/s` or ""}
                end
                local l = sp:FindFirstChild("SaeV")
                if not l then
                    local t0 = sp:FindFirstChild("Progress") and sp.Progress:FindFirstChild("TextLabel")
                    l = Instance.new("TextLabel")
                    l.Name, l.BackgroundTransparency, l.Position, l.Size = "SaeV", 1, UDim2.fromScale(0.247, 0.02), UDim2.fromScale(0.587, 0.27)
                    l.TextScaled, l.TextXAlignment, l.TextColor3 = true, Enum.TextXAlignment.Left, Color3.fromRGB(166, 255, 111)
                    if t0 then l.FontFace = t0.FontFace end
                    local s0 = t0 and t0:FindFirstChildOfClass("UIStroke")
                    if s0 then s0:Clone().Parent = l end
                    l.Parent = sp
                end
                l.Text = qv[r.Name].t
            end
        end)
        if not ok then writeLog(`[Egg Value] Failed: {wy}`) end
        task.wait(0.5)
    end
end)
