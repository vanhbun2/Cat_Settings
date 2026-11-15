local plrs = game:GetService("Players")
local lt = game:GetService("Lighting")
local tw = game:GetService("TweenService")
local uis = game:GetService("UserInputService")
local rs = game:GetService("RunService")
local stats = game:GetService("Stats")
local http = game:GetService("HttpService")
local tsv = game:GetService("TextService")
local lp = plrs.LocalPlayer
local pg = lp:WaitForChild("PlayerGui", 10) or lp:FindFirstChildOfClass("PlayerGui")
local env = (getgenv and getgenv()) or _G
if env.CatSettingsUnload then
    pcall(env.CatSettingsUnload)
    env.CatSettingsUnload = nil
end
local old = pg and pg:FindFirstChild("CatSettings")
if old then
    old:Destroy()
end
local file = "CatSettings.json"
local cs = type(writefile) == "function" and type(readfile) == "function" and type(isfile) == "function"
local dtheme = Color3.fromRGB(120, 105, 255)
local cf = {avatar = "74894092061904", theme = dtheme, size = 1, tr = 0.12, key = Enum.KeyCode.RightShift, notif = true, btn = true}
local fs = {spec = nil, key = nil, game = true, menu = true}
local sv = {skies = {}, fonts = {}, active = nil}

local function trim(v)
    return tostring(v):match("^%s*(.-)%s*$") or ""
end

local function nid(v)
    local t = tostring(v)
    return t:match("^%s*(%d+)%s*$") or t:match("rbxassetid://(%d+)") or t:match("[?&]id=(%d+)")
end

local function hex(v)
    local h = tostring(v):gsub("[#%s]", "")
    if #h ~= 6 or not h:match("^%x+$") then
        return nil
    end
    return Color3.fromRGB(tonumber(h:sub(1, 2), 16), tonumber(h:sub(3, 4), 16), tonumber(h:sub(5, 6), 16))
end

local function tohex(c)
    return string.format("#%02X%02X%02X", math.round(math.clamp(c.R, 0, 1) * 255), math.round(math.clamp(c.G, 0, 1) * 255), math.round(math.clamp(c.B, 0, 1) * 255))
end

local function load()
    if not cs then
        return
    end
    local ok, d = pcall(function()
        if isfile(file) then
            return http:JSONDecode(readfile(file))
        end
    end)
    if not ok or type(d) ~= "table" then
        return
    end
    if type(d.Theme) == "table" then
        local r, g, b = tonumber(d.Theme[1]), tonumber(d.Theme[2]), tonumber(d.Theme[3])
        if r and g and b then
            cf.theme = Color3.fromRGB(math.clamp(r, 0, 255), math.clamp(g, 0, 255), math.clamp(b, 0, 255))
        end
    end
    if type(d.MenuSize) == "number" then
        cf.size = math.clamp(d.MenuSize, 0.7, 1.2)
    end
    if type(d.Transparency) == "number" then
        cf.tr = math.clamp(d.Transparency, 0, 0.5)
    end
    if type(d.Notifications) == "boolean" then
        cf.notif = d.Notifications
    end
    if type(d.ShowButton) == "boolean" then
        cf.btn = d.ShowButton
    end
    if type(d.Key) == "string" then
        local f, k = pcall(function()
            return Enum.KeyCode[d.Key]
        end)
        if f and k then
            cf.key = k
        end
    end
    if type(d.GameText) == "boolean" then
        fs.game = d.GameText
    end
    if type(d.MenuText) == "boolean" then
        fs.menu = d.MenuText
    end
    if type(d.ActiveFont) == "string" then
        sv.active = d.ActiveFont
    end
    if type(d.Skies) == "table" then
        for _, e in ipairs(d.Skies) do
            if type(e) == "table" and type(e.Name) == "string" and type(e.Ids) == "table" then
                local ids = {}
                for _, r in ipairs(e.Ids) do
                    local n = nid(r)
                    if n then
                        table.insert(ids, n)
                    end
                end
                if #ids == 1 or #ids == 6 then
                    table.insert(sv.skies, {Name = e.Name, Ids = ids})
                end
            end
        end
    end
    if type(d.Fonts) == "table" then
        for _, e in ipairs(d.Fonts) do
            if type(e) == "table" and type(e.Name) == "string" and type(e.Value) == "string" then
                table.insert(sv.fonts, {Name = e.Name, Value = e.Value})
            end
        end
    end
end
load()

local col = {
    bg = Color3.fromRGB(15, 17, 25),
    panel = Color3.fromRGB(23, 26, 37),
    card = Color3.fromRGB(31, 35, 49),
    text = Color3.fromRGB(245, 246, 255),
    sub = Color3.fromRGB(153, 160, 180),
    acc = cf.theme,
    green = Color3.fromRGB(90, 220, 155),
    yellow = Color3.fromRGB(255, 205, 90),
    red = Color3.fromRGB(255, 100, 110),
    off = Color3.fromRGB(65, 68, 82)
}

local function weak()
    return setmetatable({}, {__mode = "k"})
end

local olt, owt = {}, {}
local oobj, gtx, mtx = weak(), weak(), weak()
local hsky, hatm, conns, life, astk, scanned, stok, navs, pages, fb = {}, {}, {}, {}, {}, {}, {}, {}, {}, {}
local pnames = {"Performance", "Sky", "Lighting", "Effects", "Fonts", "Interface", "General"}
local cam0 = workspace.CurrentCamera

local s = {
    lag = false, shd = false, ptc = false, tex = false, pst = false, mat = false, wtr = false, low = false,
    fps = false, png = false, mem = false, fog = true, lck = false, lckt = lt.ClockTime,
    bloom = false, blur = false, rays = false, dof = false, cfov = false,
    fov = cam0 and cam0.FieldOfView or 70,
    gl = 1.5, spin = 0, flow = 0
}
local kf = {e = "ptc", p = "pst", d = "tex", m = "mat"}
local bm = {
    Gotham = "GothamBold", GothamSemibold = "GothamBold", GothamBlack = "GothamBlack",
    SourceSans = "SourceSansBold", SourceSansLight = "SourceSansBold", SourceSansSemibold = "SourceSansBold", Arial = "ArialBold"
}

local gui, main, scale, side, nav, body, head, holder, title, obtn, toast, minb, closeb, ind, fade
local cur, openpage, refresh, shutdown, resetpos
local dead, mini, anim, order, sweeptok, toasttok, savetok = false, false, 0, 0, 0, 0, 0
local lockconn, camconn

local function mk(cls, p, par)
    local o = Instance.new(cls)
    for k, v in pairs(p) do
        o[k] = v
    end
    o.Parent = par
    return o
end

local function rc(o, r)
    local i = Instance.new("UICorner")
    i.CornerRadius = UDim.new(0, r or 12)
    i.Parent = o
end

local function st(o, cr, t)
    local i = Instance.new("UIStroke")
    i.Color = cr or Color3.new(1, 1, 1)
    i.Transparency = t or 0.85
    i.Thickness = 1
    i.Parent = o
    return i
end

local function ast(o, t)
    table.insert(astk, st(o, col.acc, t))
end

local function pd(o, l)
    local i = Instance.new("UIPadding")
    i.PaddingLeft = UDim.new(0, l or 10)
    i.PaddingRight = UDim.new(0, 10)
    i.Parent = o
end

local function tn(o, p, d)
    if o and o.Parent then
        tw:Create(o, TweenInfo.new(d or 0.2, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), p):Play()
    end
end

local function apply(o, sp)
    if not o or not o.Parent or not sp then
        return
    end
    if sp.e then
        o.Font = sp.e
    elseif sp.f then
        o.FontFace = sp.f
    end
end

local function mspec(b)
    local sp = fs.menu and fs.spec
    if not sp then
        return {e = b and Enum.Font.GothamBold or Enum.Font.Gotham}
    end
    if sp.e then
        local n = bm[sp.e.Name]
        if b and n then
            return {e = Enum.Font[n]}
        end
        return sp
    end
    if b and sp.f then
        return {f = Font.new(sp.f.Family, Enum.FontWeight.Bold, sp.f.Style)}
    end
    return sp
end

local function tx(cls, p, par, b)
    local o = mk(cls, p, par)
    mtx[o] = b == true
    pcall(apply, o, mspec(b == true))
    return o
end

local function menufont()
    for o, b in pairs(mtx) do
        if o.Parent then
            pcall(apply, o, mspec(b))
        else
            mtx[o] = nil
        end
    end
end

local function notify(m)
    if dead or not cf.notif or not toast or not toast.Parent then
        return
    end
    toasttok += 1
    local id = toasttok
    toast.Text = m
    toast.Visible = true
    toast.Position = UDim2.new(0.5, 0, 0, -50)
    tn(toast, {Position = UDim2.new(0.5, 0, 0, 14)}, 0.25)
    task.delay(2.2, function()
        if dead or id ~= toasttok or not toast.Parent then
            return
        end
        tn(toast, {Position = UDim2.new(0.5, 0, 0, -60)}, 0.25)
        task.delay(0.3, function()
            if id == toasttok and toast.Parent then
                toast.Visible = false
            end
        end)
    end)
end

local fitems = {}
for _, i in ipairs(Enum.Font:GetEnumItems()) do
    if i.Name ~= "Legacy" and i.Name ~= "Unknown" then
        table.insert(fitems, i)
    end
end
table.sort(fitems, function(a, b)
    return a.Name < b.Name
end)

local function loads(face)
    return (pcall(function()
        local p = Instance.new("GetTextBoundsParams")
        p.Text = "Test"
        p.Font = face
        p.Size = 14
        p.Width = 1000
        tsv:GetTextBoundsAsync(p)
    end))
end

local function pfont(v, check)
    v = trim(v)
    if v == "" then
        return nil
    end
    for _, i in ipairs(fitems) do
        if i.Name:lower() == v:lower() then
            return {e = i, k = "Enum:" .. i.Name}
        end
    end
    local id = nid(v)
    local fam = id and ("rbxassetid://" .. id) or (v:find("^rbxasset") and v)
    if not fam then
        return nil
    end
    local ok, face = pcall(Font.new, fam, Enum.FontWeight.Regular, Enum.FontStyle.Normal)
    if not ok or (check and not loads(face)) then
        return nil
    end
    return {f = face, k = "Face:" .. fam}
end

local function skey(k)
    return pfont(k:match("^%a+:(.+)$") or "")
end

local function save()
    if not cs then
        return
    end
    local d = {
        Theme = {math.round(cf.theme.R * 255), math.round(cf.theme.G * 255), math.round(cf.theme.B * 255)},
        MenuSize = cf.size,
        Transparency = cf.tr,
        Key = cf.key.Name,
        Notifications = cf.notif,
        ShowButton = cf.btn,
        Skies = sv.skies,
        Fonts = sv.fonts,
        ActiveFont = fs.key,
        GameText = fs.game,
        MenuText = fs.menu
    }
    pcall(function()
        writefile(file, http:JSONEncode(d))
    end)
end

local function later()
    if not cs then
        return
    end
    savetok += 1
    local id = savetok
    task.delay(0.6, function()
        if id == savetok and not dead then
            save()
        end
    end)
end

local function drag(o, h, scaled)
    local on, di, ds, sp, mv = false, nil, nil, nil, false
    table.insert(life, h.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on, mv, di, ds, sp = true, false, i, i.Position, o.Position
        end
    end))
    table.insert(life, uis.InputChanged:Connect(function(i)
        if not on then
            return
        end
        if i.UserInputType == Enum.UserInputType.MouseMovement or i == di then
            local d = i.Position - ds
            if not mv and d.Magnitude < 6 then
                return
            end
            mv = true
            local f = (scaled and scale and scale.Scale) or 1
            if f <= 0 then
                f = 1
            end
            o.Position = UDim2.new(sp.X.Scale, sp.X.Offset + d.X / f, sp.Y.Scale, sp.Y.Offset + d.Y / f)
        end
    end))
    table.insert(life, uis.InputEnded:Connect(function(i)
        if i == di or i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on, di = false, nil
        end
    end))
    return function()
        return mv
    end
end

for _, p in ipairs({"Brightness", "ClockTime", "Ambient", "OutdoorAmbient", "ColorShift_Top", "ColorShift_Bottom", "FogColor", "FogStart", "FogEnd", "ExposureCompensation", "GlobalShadows", "EnvironmentDiffuseScale", "EnvironmentSpecularScale"}) do
    local ok, v = pcall(function()
        return lt[p]
    end)
    if ok then
        olt[p] = v
    end
end

local ter = workspace:FindFirstChildOfClass("Terrain")
if ter then
    for _, p in ipairs({"WaterWaveSize", "WaterWaveSpeed", "WaterReflectance"}) do
        pcall(function()
            owt[p] = ter[p]
        end)
    end
end

local oq
pcall(function()
    oq = settings().Rendering.QualityLevel
end)

gui = mk("ScreenGui", {Name = "CatSettings", ResetOnSpawn = false, DisplayOrder = 50, ZIndexBehavior = Enum.ZIndexBehavior.Sibling}, pg)
fade = mk("Frame", {Size = UDim2.new(1, 0, 1, 120), Position = UDim2.fromOffset(0, -60), BackgroundColor3 = Color3.new(0, 0, 0), BackgroundTransparency = 1, BorderSizePixel = 0, ZIndex = 0}, gui)
main = mk("Frame", {Name = "SettingsPanel", Size = UDim2.fromOffset(700, 460), AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5), BackgroundColor3 = col.bg, BackgroundTransparency = cf.tr, BorderSizePixel = 0, ClipsDescendants = true, Visible = false}, gui)
rc(main, 18)
ast(main, 0.5)
scale = mk("UIScale", {Scale = cf.size}, main)

local function fit()
    local cam = workspace.CurrentCamera
    if not cam then
        scale.Scale = cf.size
        return
    end
    local v = cam.ViewportSize
    scale.Scale = math.min(cf.size, math.max(0.4, (v.X - 24) / 700), math.max(0.4, (v.Y - 24) / 460))
end

local function watch()
    if camconn then
        camconn:Disconnect()
        camconn = nil
    end
    local cam = workspace.CurrentCamera
    if cam then
        camconn = cam:GetPropertyChangedSignal("ViewportSize"):Connect(fit)
    end
    fit()
end

side = mk("Frame", {Size = UDim2.new(0, 175, 1, 0), BackgroundColor3 = col.panel, BorderSizePixel = 0}, main)
rc(side, 18)
mk("Frame", {Size = UDim2.new(0, 18, 1, 0), Position = UDim2.new(1, -18, 0, 0), BackgroundColor3 = col.panel, BorderSizePixel = 0}, side)

local function fallback(img, size)
    local l = tx("TextLabel", {Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "CAT", TextColor3 = col.text, TextSize = size, ZIndex = 2, Visible = not img.IsLoaded}, img, true)
    table.insert(life, img:GetPropertyChangedSignal("IsLoaded"):Connect(function()
        l.Visible = not img.IsLoaded
    end))
end

local av = mk("ImageButton", {Size = UDim2.fromOffset(54, 54), Position = UDim2.fromOffset(18, 20), BackgroundColor3 = col.card, Image = "rbxassetid://" .. cf.avatar, ScaleType = Enum.ScaleType.Crop, AutoButtonColor = false}, side)
rc(av, 16)
fallback(av, 14)
tx("TextLabel", {Size = UDim2.new(1, -25, 0, 24), Position = UDim2.fromOffset(18, 82), BackgroundTransparency = 1, Text = "CAT SETTINGS", TextColor3 = col.text, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left}, side, true)
nav = mk("Frame", {Size = UDim2.new(1, -20, 1, -126), Position = UDim2.fromOffset(10, 116), BackgroundTransparency = 1}, side)
mk("UIListLayout", {Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder}, nav)
body = mk("Frame", {Size = UDim2.new(1, -195, 1, -28), Position = UDim2.fromOffset(185, 14), BackgroundTransparency = 1}, main)
head = mk("Frame", {Size = UDim2.new(1, 0, 0, 48), BackgroundTransparency = 1}, body)
title = tx("TextLabel", {Size = UDim2.new(1, -100, 0, 32), Position = UDim2.fromOffset(4, 0), BackgroundTransparency = 1, Text = "Performance", TextColor3 = col.text, TextSize = 20, TextXAlignment = Enum.TextXAlignment.Left}, head, true)
minb = tx("TextButton", {Size = UDim2.fromOffset(32, 32), Position = UDim2.new(1, -76, 0, 0), BackgroundColor3 = col.card, Text = "−", TextColor3 = col.text, TextSize = 20, AutoButtonColor = false}, head, true)
rc(minb, 9)
closeb = tx("TextButton", {Size = UDim2.fromOffset(32, 32), Position = UDim2.new(1, -36, 0, 0), BackgroundColor3 = col.card, Text = "×", TextColor3 = col.text, TextSize = 20, AutoButtonColor = false}, head, true)
rc(closeb, 9)
holder = mk("ScrollingFrame", {Size = UDim2.new(1, 0, 1, -55), Position = UDim2.fromOffset(0, 55), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3, ScrollBarImageColor3 = col.acc, ScrollingDirection = Enum.ScrollingDirection.Y, AutomaticCanvasSize = Enum.AutomaticSize.Y, CanvasSize = UDim2.new()}, body)
mk("UIListLayout", {Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder}, holder)
mk("UIPadding", {PaddingBottom = UDim.new(0, 12)}, holder)

local sl, spos = {}, {}

local function stat(k, y, init)
    local p = UDim2.new(1, -125, 0, y)
    local l = tx("TextLabel", {Size = UDim2.fromOffset(110, 28), Position = p, BackgroundColor3 = col.card, BackgroundTransparency = 0.1, Text = init, TextColor3 = col.green, TextSize = 12, Visible = false}, gui, true)
    rc(l, 9)
    drag(l, l, false)
    sl[k], spos[k] = l, p
end
stat("fps", 12, "FPS: --")
stat("png", 46, "PING: --")
stat("mem", 80, "MEM: --")

toast = tx("TextLabel", {Size = UDim2.fromOffset(280, 40), AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, -60), BackgroundColor3 = col.card, BackgroundTransparency = 0.05, TextColor3 = col.text, Text = "", TextSize = 13, ZIndex = 20, Visible = false}, gui, false)
rc(toast, 12)
ast(toast, 0.6)

local function curatm()
    return lt:FindFirstChild("CatAtmosphere") or lt:FindFirstChildOfClass("Atmosphere")
end

local function cursky()
    return lt:FindFirstChild("CatSky") or lt:FindFirstChildOfClass("Sky")
end

local function rp(o, p, d)
    if o then
        local ok, v = pcall(function()
            return o[p]
        end)
        if ok and v ~= nil then
            return v
        end
    end
    return d
end

local function hide(cls, store, keep)
    for _, o in ipairs(lt:GetChildren()) do
        if o:IsA(cls) and o.Name ~= keep then
            table.insert(store, o)
            o.Parent = nil
        end
    end
end

local skyprops = {"SkyboxBk", "SkyboxDn", "SkyboxFt", "SkyboxLf", "SkyboxRt", "SkyboxUp", "StarCount", "CelestialBodiesShown", "SunAngularSize", "MoonAngularSize", "SunTextureId", "MoonTextureId", "SkyboxOrientation"}
local atmprops = {"Density", "Offset", "Color", "Decay", "Glare", "Haze"}

local function catsky(fresh)
    local e = lt:FindFirstChild("CatSky")
    if e and not fresh then
        return e
    end
    if e then
        e:Destroy()
    end
    local sk = Instance.new("Sky")
    sk.Name = "CatSky"
    if not fresh then
        local src = lt:FindFirstChildOfClass("Sky")
        if src then
            for _, p in ipairs(skyprops) do
                pcall(function()
                    sk[p] = src[p]
                end)
            end
        end
    end
    hide("Sky", hsky, "CatSky")
    sk.Parent = lt
    return sk
end

local function catatm()
    local e = lt:FindFirstChild("CatAtmosphere")
    if e then
        return e
    end
    local a = Instance.new("Atmosphere")
    a.Name = "CatAtmosphere"
    local src = lt:FindFirstChildOfClass("Atmosphere")
    if src then
        for _, p in ipairs(atmprops) do
            pcall(function()
                a[p] = src[p]
            end)
        end
    end
    hide("Atmosphere", hatm, "CatAtmosphere")
    a.Parent = lt
    return a
end

local function setatm(p, v)
    pcall(function()
        catatm()[p] = v
    end)
end

local function lockclock()
    if lockconn then
        lockconn:Disconnect()
        lockconn = nil
    end
    if s.lck and not dead then
        lockconn = lt:GetPropertyChangedSignal("ClockTime"):Connect(function()
            if s.lck and math.abs(lt.ClockTime - s.lckt) > 0.001 then
                lt.ClockTime = s.lckt
            end
        end)
    end
end

local function setclock(v)
    s.lckt = v
    lt.ClockTime = v
end

local function shadows()
    local b = olt.GlobalShadows
    if b == nil then
        b = true
    end
    pcall(function()
        lt.GlobalShadows = b and not s.shd
    end)
end

local function fog(on)
    s.fog = on
    if on then
        lt.FogStart = olt.FogStart or lt.FogStart
        lt.FogEnd = olt.FogEnd or lt.FogEnd
    else
        lt.FogStart = 100000
        lt.FogEnd = 100000
    end
end

local pskies = {
    {n = "REALISTIC DAY", t = 13, b = 3, a = Color3.fromRGB(170, 180, 200), o = Color3.fromRGB(190, 195, 205), f = Color3.fromRGB(180, 215, 245)},
    {n = "SUNRISE", t = 6.2, b = 2.4, a = Color3.fromRGB(150, 130, 140), o = Color3.fromRGB(200, 160, 150), f = Color3.fromRGB(255, 190, 150)},
    {n = "GOLDEN HOUR", t = 17, b = 2.8, a = Color3.fromRGB(180, 140, 115), o = Color3.fromRGB(210, 155, 120), f = Color3.fromRGB(255, 180, 125)},
    {n = "SUNSET", t = 18.3, b = 2.5, a = Color3.fromRGB(145, 105, 130), o = Color3.fromRGB(190, 130, 140), f = Color3.fromRGB(245, 145, 135)},
    {n = "EVENING", t = 19.5, b = 2, a = Color3.fromRGB(100, 105, 145), o = Color3.fromRGB(125, 135, 180), f = Color3.fromRGB(100, 120, 175)},
    {n = "NIGHT", t = 0, b = 1.5, a = Color3.fromRGB(65, 75, 115), o = Color3.fromRGB(90, 105, 150), f = Color3.fromRGB(35, 45, 90)},
    {n = "MOONLIGHT", t = 0, b = 1.2, a = Color3.fromRGB(50, 60, 100), o = Color3.fromRGB(70, 85, 130), f = Color3.fromRGB(25, 35, 75)},
    {n = "CLOUDY", t = 12, b = 2, a = Color3.fromRGB(130, 140, 150), o = Color3.fromRGB(160, 170, 180), f = Color3.fromRGB(145, 160, 180)}
}

local pspace = {
    {n = "MILKY WAY", c = Color3.fromRGB(55, 65, 110)},
    {n = "DEEP SPACE", c = Color3.fromRGB(35, 40, 85)},
    {n = "NEBULA", c = Color3.fromRGB(100, 55, 140)},
    {n = "AURORA", c = Color3.fromRGB(40, 115, 110)},
    {n = "GALAXY", c = Color3.fromRGB(80, 45, 125)}
}

local ed = {
    bloom = {c = "BloomEffect", n = "CatBloom", d = {Intensity = 0.35, Size = 32, Threshold = 1}},
    blur = {c = "BlurEffect", n = "CatBlur", d = {Size = 8}},
    rays = {c = "SunRaysEffect", n = "CatSunRays", d = {Intensity = 0.25, Spread = 1}},
    dof = {c = "DepthOfFieldEffect", n = "CatDoF", d = {FarIntensity = 0.3, FocusDistance = 50, InFocusRadius = 30, NearIntensity = 0}}
}

local ev = {}
for k, d in pairs(ed) do
    ev[k] = table.clone(d.d)
end

local cv = {Brightness = 0, Contrast = 0, Saturation = 0, TintColor = Color3.new(1, 1, 1)}

local function seteffect(k, on)
    local d = ed[k]
    local e = lt:FindFirstChild(d.n)
    s[k] = on
    if on then
        if not e then
            e = Instance.new(d.c)
            e.Name = d.n
            e.Parent = lt
        end
        for p, v in pairs(ev[k]) do
            e[p] = v
        end
    elseif e then
        e:Destroy()
    end
end

local function setev(k, p, v)
    ev[k][p] = v
    local e = lt:FindFirstChild(ed[k].n)
    if e then
        e[p] = v
    end
end

local function setcc(p, v)
    cv[p] = v
    local e = lt:FindFirstChild("CatColorCorrection")
    if not e then
        e = Instance.new("ColorCorrectionEffect")
        e.Name = "CatColorCorrection"
        e.Parent = lt
    end
    for n, x in pairs(cv) do
        e[n] = x
    end
end

local ofov

local function fovbind()
    pcall(function()
        rs:UnbindFromRenderStep("CatSettingsFOV")
    end)
    if s.cfov and not dead then
        rs:BindToRenderStep("CatSettingsFOV", Enum.RenderPriority.Camera.Value + 1, function()
            local cam = workspace.CurrentCamera
            if cam and cam.FieldOfView ~= s.fov then
                cam.FieldOfView = s.fov
            end
        end)
    end
end

local function setfov(on)
    local cam = workspace.CurrentCamera
    s.cfov = on
    if on and cam and not ofov then
        ofov = cam.FieldOfView
    end
    fovbind()
    if not on and cam and ofov then
        cam.FieldOfView = ofov
        ofov = nil
    end
end

local gtok = 0

local function glide(list, dur)
    gtok += 1
    local id = gtok
    dur = dur or s.gl
    if dur <= 0 then
        for _, d in ipairs(list) do
            if d[2] == "ClockTime" then
                s.lckt = d[3]
            end
            d[1][d[2]] = d[3]
        end
        return
    end
    local from = {}
    for i, d in ipairs(list) do
        from[i] = d[1][d[2]]
    end
    local t = 0
    local con
    con = rs.Heartbeat:Connect(function(dt)
        if dead or id ~= gtok then
            con:Disconnect()
            return
        end
        t += dt
        local a = math.min(t / dur, 1)
        a = a * a * (3 - 2 * a)
        for i, d in ipairs(list) do
            local f, g, v = from[i], d[3], nil
            if typeof(g) == "Color3" then
                v = f:Lerp(g, a)
            elseif d[2] == "ClockTime" then
                v = (f + ((g - f + 12) % 24 - 12) * a) % 24
                s.lckt = v
            else
                v = f + (g - f) * a
            end
            d[1][d[2]] = v
        end
        if a >= 1 then
            con:Disconnect()
        end
    end)
end

local function run(fn)
    if s.gl <= 0 or dead then
        fn()
        return
    end
    tn(fade, {BackgroundTransparency = 0.45}, 0.25)
    task.delay(0.28, function()
        if dead then
            return
        end
        fn()
        tn(fade, {BackgroundTransparency = 1}, 0.6)
    end)
end

local function applysky(p)
    run(function()
        catsky(true)
        local a = catatm()
        glide({
            {lt, "ClockTime", p.t}, {lt, "Brightness", p.b}, {lt, "Ambient", p.a}, {lt, "OutdoorAmbient", p.o}, {lt, "FogColor", p.f},
            {a, "Density", 0.25}, {a, "Haze", 1}, {a, "Glare", 0.2}, {a, "Offset", 0.1}, {a, "Color", p.f}, {a, "Decay", Color3.fromRGB(90, 100, 130)}
        })
    end)
    notify(p.n .. " applied")
    refresh()
end

local function applyspace(p)
    run(function()
        local sk = catsky(true)
        sk.StarCount = 5000
        sk.SunAngularSize = 0
        sk.MoonAngularSize = 11
        sk.CelestialBodiesShown = true
        local a = catatm()
        glide({
            {lt, "ClockTime", 0}, {lt, "Brightness", 1.5}, {lt, "Ambient", p.c}, {lt, "OutdoorAmbient", p.c}, {lt, "FogColor", p.c},
            {a, "Density", 0.05}, {a, "Haze", 0}, {a, "Color", p.c}
        })
    end)
    notify(p.n .. " applied")
    refresh()
end

local faces = {"SkyboxBk", "SkyboxDn", "SkyboxFt", "SkyboxLf", "SkyboxRt", "SkyboxUp"}

local function applycustom(e)
    run(function()
        local sk = catsky(true)
        for i, f in ipairs(faces) do
            sk[f] = "rbxassetid://" .. (e.Ids[i] or e.Ids[1])
        end
    end)
    notify(e.Name .. " applied")
    refresh()
end

local lroot, lcache = "CatSkies", "CatSkiesCache"
local fk = {bk = "SkyboxBk", dn = "SkyboxDn", ft = "SkyboxFt", lf = "SkyboxLf", rt = "SkyboxRt", up = "SkyboxUp"}
local lfs = cs and type(isfolder) == "function" and type(listfiles) == "function" and type(makefolder) == "function" and type(getcustomasset) == "function"
local lsk

local function scanlocal()
    lsk = {}
    if not lfs then
        return
    end
    local function walk(dir, depth)
        local fl, n, subs = {}, 0, {}
        for _, fp in ipairs(listfiles(dir)) do
            if isfolder(fp) then
                table.insert(subs, fp)
            else
                local k = fp:lower():match("[_%-%s](%a%a)%.%w+$")
                if k and fk[k] then
                    if not fl[k] then
                        n += 1
                    end
                    fl[k] = fp
                end
            end
        end
        if n == 6 then
            table.insert(lsk, {n = dir:match("[^/\\]+$") or dir, f = fl})
        end
        if depth < 3 then
            for _, sd in ipairs(subs) do
                walk(sd, depth + 1)
            end
        end
    end
    pcall(function()
        if not isfolder(lroot) then
            makefolder(lroot)
        end
        walk(lroot, 0)
    end)
    table.sort(lsk, function(a, b)
        return a.n < b.n
    end)
end

local function applylocal(e)
    task.spawn(function()
        local ok = pcall(function()
            local base = e.n:gsub("[^%w]", "_")
            if not isfolder(lcache) then
                makefolder(lcache)
            end
            local urls = {}
            for k, prop in pairs(fk) do
                local dst = lcache .. "/" .. base .. "_" .. k .. ".png"
                if not isfile(dst) then
                    writefile(dst, readfile(e.f[k]))
                end
                urls[prop] = getcustomasset(dst)
            end
            run(function()
                local sk = catsky(true)
                for prop, u in pairs(urls) do
                    sk[prop] = u
                end
            end)
        end)
        notify(ok and (e.n .. " applied") or "Local sky failed")
        if ok then
            refresh()
        end
    end)
end

local function randomsky()
    if lfs and not lsk then
        scanlocal()
    end
    local pool = {}
    for _, p in ipairs(pskies) do
        table.insert(pool, function()
            applysky(p)
        end)
    end
    for _, p in ipairs(pspace) do
        table.insert(pool, function()
            applyspace(p)
        end)
    end
    for _, e in ipairs(sv.skies) do
        table.insert(pool, function()
            applycustom(e)
        end)
    end
    for _, e in ipairs(lsk or {}) do
        table.insert(pool, function()
            applylocal(e)
        end)
    end
    pool[math.random(#pool)]()
end

local function pids(v)
    local ids = {}
    for t in tostring(v):gmatch("[^,%s;]+") do
        local i = nid(t)
        if not i then
            return nil
        end
        table.insert(ids, i)
    end
    if #ids == 1 or #ids == 6 then
        return ids
    end
    return nil
end

local function rlight(q)
    gtok += 1
    s.spin = 0
    s.flow = 0
    local a = lt:FindFirstChild("CatSky")
    if a then
        a:Destroy()
    end
    local b = lt:FindFirstChild("CatAtmosphere")
    if b then
        b:Destroy()
    end
    for _, o in ipairs(hsky) do
        pcall(function()
            o.Parent = lt
        end)
    end
    for _, o in ipairs(hatm) do
        pcall(function()
            o.Parent = lt
        end)
    end
    table.clear(hsky)
    table.clear(hatm)
    for p, v in pairs(olt) do
        pcall(function()
            lt[p] = v
        end)
    end
    s.fog = true
    s.lck = false
    s.lckt = lt.ClockTime
    lockclock()
    shadows()
    if not q then
        notify("Lighting restored")
    end
end

local function reffects(q)
    for k in pairs(ed) do
        seteffect(k, false)
        ev[k] = table.clone(ed[k].d)
    end
    local cc = lt:FindFirstChild("CatColorCorrection")
    if cc then
        cc:Destroy()
    end
    cv.Brightness, cv.Contrast, cv.Saturation, cv.TintColor = 0, 0, 0, Color3.new(1, 1, 1)
    setfov(false)
    if not q then
        notify("Effects reset")
    end
end

local function kind(o)
    if not o or not o.Parent then
        return nil
    end
    if o:IsA("ParticleEmitter") or o:IsA("Trail") or o:IsA("Beam") then
        return "e"
    end
    if o:IsA("PostEffect") then
        if o.Name:sub(1, 3) == "Cat" then
            return nil
        end
        return "p"
    end
    if o:IsA("Decal") then
        if o.Name:lower() == "face" then
            return nil
        end
        return "d"
    end
    if s.mat and o:IsA("BasePart") and not o:IsA("Terrain") then
        return "m"
    end
    return nil
end

local function capture(o, k)
    local e = oobj[o]
    if e then
        return e
    end
    local d
    if k == "e" or k == "p" then
        d = {k = k, en = o.Enabled}
    elseif k == "d" then
        d = {k = k, t = o.Transparency}
    elseif k == "m" then
        d = {k = k, m = o.Material, r = o.Reflectance}
    end
    oobj[o] = d
    return d
end

local function applyd(o, d)
    local k = d.k
    if k == "e" then
        o.Enabled = d.en and not s.ptc
    elseif k == "p" then
        o.Enabled = d.en and not s.pst
    elseif k == "d" then
        o.Transparency = s.tex and 1 or d.t
    elseif k == "m" then
        if s.mat then
            o.Material = Enum.Material.SmoothPlastic
            o.Reflectance = 0
        else
            o.Material = d.m
            o.Reflectance = d.r
        end
    end
end

local function scan(k)
    stok[k] = (stok[k] or 0) + 1
    local tok = stok[k]
    task.spawn(function()
        local roots = k == "p" and {lt, workspace.CurrentCamera} or {workspace}
        local t = os.clock()
        for _, r in ipairs(roots) do
            for _, o in ipairs(r:GetDescendants()) do
                if dead or not scanned[k] or tok ~= stok[k] then
                    return
                end
                if kind(o) == k then
                    pcall(function()
                        local d = capture(o, k)
                        if d then
                            applyd(o, d)
                        end
                    end)
                end
                if os.clock() - t > 0.003 then
                    task.wait()
                    t = os.clock()
                end
            end
        end
    end)
end

local function setkind(k)
    if s[kf[k]] then
        if not scanned[k] then
            scanned[k] = true
            scan(k)
        end
    else
        scanned[k] = nil
        stok[k] = (stok[k] or 0) + 1
        for o, d in pairs(oobj) do
            if d.k == k then
                if o.Parent then
                    pcall(applyd, o, d)
                end
                oobj[o] = nil
            end
        end
    end
end

local function water()
    if not ter then
        return
    end
    pcall(function()
        if s.wtr then
            ter.WaterWaveSize = 0
            ter.WaterWaveSpeed = 0
            ter.WaterReflectance = 0
        else
            for p, v in pairs(owt) do
                ter[p] = v
            end
        end
    end)
end

local function quality()
    pcall(function()
        local r = settings().Rendering
        if s.low then
            r.QualityLevel = Enum.QualityLevel.Level01
        elseif oq then
            r.QualityLevel = oq
        end
    end)
end

local function setlag(v)
    s.lag, s.shd, s.ptc, s.pst, s.mat, s.wtr, s.low = v, v, v, v, v, v, v
    shadows()
    for k in pairs(kf) do
        setkind(k)
    end
    water()
    quality()
    refresh()
    notify(v and "Fix Lag enabled" or "Fix Lag disabled")
end

local function rgraphics(q)
    s.lag, s.shd, s.ptc, s.tex, s.pst, s.mat, s.wtr, s.low = false, false, false, false, false, false, false, false
    for k in pairs(kf) do
        setkind(k)
    end
    shadows()
    water()
    quality()
    if not q then
        notify("Graphics restored")
    end
end

local function isgt(o)
    return (o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox")) and not o:IsDescendantOf(gui)
end

local function stylegt(o)
    local sp = fs.spec
    if not sp or not fs.game then
        return
    end
    if gtx[o] == nil then
        local ok, f = pcall(function()
            return o.FontFace
        end)
        if not ok then
            return
        end
        gtx[o] = f
    end
    pcall(apply, o, sp)
end

local function sweep()
    sweeptok += 1
    local tok = sweeptok
    task.spawn(function()
        local t = os.clock()
        for _, r in ipairs({pg, workspace}) do
            for _, o in ipairs(r:GetDescendants()) do
                if tok ~= sweeptok or dead then
                    return
                end
                if isgt(o) then
                    stylegt(o)
                end
                if os.clock() - t > 0.003 then
                    task.wait()
                    t = os.clock()
                end
            end
        end
    end)
end

local function rgt()
    sweeptok += 1
    for o, f in pairs(gtx) do
        if o.Parent then
            pcall(function()
                o.FontFace = f
            end)
        end
    end
    table.clear(gtx)
end

local function same(o, sp)
    if sp.e then
        return o.Font == sp.e
    end
    local f = o.FontFace
    return f.Family == sp.f.Family and f.Weight == sp.f.Weight and f.Style == sp.f.Style
end

local function fbrefresh()
    for k, b in pairs(fb) do
        if b.Parent then
            local on = fs.key == k
            b.Text = on and "Active" or "Apply"
            b.BackgroundColor3 = on and col.green or col.acc
        else
            fb[k] = nil
        end
    end
end

local function setfont(sp, q)
    fs.spec = sp
    fs.key = sp and sp.k or nil
    menufont()
    if sp then
        if fs.game then
            sweep()
        end
    else
        rgt()
    end
    fbrefresh()
    later()
    if not q then
        notify(sp and "Font applied" or "Font restored")
    end
end

local function setgame(on)
    fs.game = on
    if on and fs.spec then
        sweep()
    else
        rgt()
    end
    later()
end

local function setmenu(on)
    fs.menu = on
    menufont()
    later()
end

local function theme(q)
    col.acc = cf.theme
    main.BackgroundTransparency = cf.tr
    holder.ScrollBarImageColor3 = col.acc
    ind.BackgroundColor3 = col.acc
    for i = #astk, 1, -1 do
        local x = astk[i]
        if x.Parent then
            x.Color = col.acc
        else
            table.remove(astk, i)
        end
    end
    fit()
    later()
    if not q then
        notify("Theme updated")
    end
end

local function clear()
    order = 0
    for _, o in ipairs(holder:GetChildren()) do
        if o:IsA("GuiObject") then
            o:Destroy()
        end
    end
end

local function nxt()
    order += 1
    return order
end

local function card(h)
    local f = mk("Frame", {Size = UDim2.new(1, -5, 0, h), BackgroundColor3 = col.card, BackgroundTransparency = 0.08, BorderSizePixel = 0, LayoutOrder = nxt()}, holder)
    rc(f, 12)
    st(f, Color3.new(1, 1, 1), 0.94)
    return f
end

local function section(n)
    tx("TextLabel", {Size = UDim2.new(1, -5, 0, 24), BackgroundTransparency = 1, Text = n, TextColor3 = col.acc, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left, LayoutOrder = nxt()}, holder, true)
end

local function hover(b, list)
    list = list or conns
    table.insert(list, b.MouseEnter:Connect(function()
        tn(b, {BackgroundTransparency = 0.2}, 0.12)
    end))
    table.insert(list, b.MouseLeave:Connect(function()
        tn(b, {BackgroundTransparency = 0}, 0.12)
    end))
end

local function row(n, btns, pf)
    local cd = card(48)
    local off = 12
    local made = {}
    for i = #btns, 1, -1 do
        local b = btns[i]
        local w = b.w or 78
        local bt = tx("TextButton", {Size = UDim2.fromOffset(w, 30), Position = UDim2.new(1, -(off + w), 0.5, -15), BackgroundColor3 = b.c or col.acc, Text = b.t, TextColor3 = col.text, TextSize = 12, AutoButtonColor = false}, cd, true)
        rc(bt, 9)
        hover(bt)
        table.insert(conns, bt.MouseButton1Click:Connect(b.f))
        made[i] = bt
        off += w + 8
    end
    local p = {Size = UDim2.new(1, -(off + 14), 1, 0), Position = UDim2.fromOffset(14, 0), BackgroundTransparency = 1, Text = n, TextColor3 = col.text, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd}
    if pf then
        p.Font = pf
        mk("TextLabel", p, cd)
    else
        tx("TextLabel", p, cd)
    end
    return cd, made
end

local function btn(n, t, f, c)
    local _, m = row(n, {{t = t or "Apply", f = f, c = c}})
    return m[1]
end

local function toggle(n, init, cb)
    local cd = card(48)
    local on = init == true
    tx("TextLabel", {Size = UDim2.new(1, -90, 1, 0), Position = UDim2.fromOffset(14, 0), BackgroundTransparency = 1, Text = n, TextColor3 = col.text, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left}, cd)
    local tg = mk("TextButton", {Size = UDim2.fromOffset(46, 25), Position = UDim2.new(1, -60, 0.5, -12), BackgroundColor3 = on and col.acc or col.off, Text = "", AutoButtonColor = false}, cd)
    rc(tg, 20)
    local kn = mk("Frame", {Size = UDim2.fromOffset(19, 19), Position = on and UDim2.new(1, -22, 0, 3) or UDim2.fromOffset(3, 3), BackgroundColor3 = Color3.new(1, 1, 1), BorderSizePixel = 0}, tg)
    rc(kn, 20)
    table.insert(conns, tg.MouseButton1Click:Connect(function()
        on = not on
        tn(tg, {BackgroundColor3 = on and col.acc or col.off})
        tn(kn, {Position = on and UDim2.new(1, -22, 0, 3) or UDim2.fromOffset(3, 3)})
        cb(on)
    end))
end

local function fmt(v)
    return tostring(math.floor(v * 1000 + 0.5) / 1000)
end

local function slider(n, mn, mx, init, cb, step)
    step = step or 0.1
    local cd = card(64)
    local v = math.clamp(init, mn, mx)
    local rg = mx - mn
    tx("TextLabel", {Size = UDim2.new(1, -90, 0, 22), Position = UDim2.fromOffset(14, 6), BackgroundTransparency = 1, Text = n, TextColor3 = col.text, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left}, cd)
    local vl = tx("TextLabel", {Size = UDim2.fromOffset(70, 22), Position = UDim2.new(1, -84, 0, 6), BackgroundTransparency = 1, Text = fmt(v), TextColor3 = col.acc, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Right}, cd)
    local hit = mk("Frame", {Size = UDim2.new(1, -28, 0, 26), Position = UDim2.fromOffset(14, 32), BackgroundTransparency = 1, Active = true}, cd)
    local bar = mk("Frame", {Size = UDim2.new(1, 0, 0, 6), Position = UDim2.new(0, 0, 0.5, -3), BackgroundColor3 = Color3.fromRGB(60, 64, 80), BorderSizePixel = 0}, hit)
    rc(bar, 10)
    local pc = rg == 0 and 0 or (v - mn) / rg
    local fl = mk("Frame", {Size = UDim2.new(pc, 0, 1, 0), BackgroundColor3 = col.acc, BorderSizePixel = 0}, bar)
    rc(fl, 10)
    local kn = mk("Frame", {Size = UDim2.fromOffset(15, 15), Position = UDim2.new(pc, -7, 0.5, -7), BackgroundColor3 = Color3.new(1, 1, 1), BorderSizePixel = 0}, bar)
    rc(kn, 20)
    local on, di = false, nil
    local function set(x)
        local r = math.clamp((x - hit.AbsolutePosition.X) / math.max(hit.AbsoluteSize.X, 1), 0, 1)
        local q = math.clamp(math.floor((mn + rg * r) / step + 0.5) * step, mn, mx)
        q = math.floor(q * 1000 + 0.5) / 1000
        local p = rg == 0 and 0 or (q - mn) / rg
        fl.Size = UDim2.new(p, 0, 1, 0)
        kn.Position = UDim2.new(p, -7, 0.5, -7)
        vl.Text = fmt(q)
        if q ~= v then
            v = q
            cb(v)
        end
    end
    table.insert(conns, hit.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on, di = true, i
            set(i.Position.X)
        end
    end))
    table.insert(conns, uis.InputChanged:Connect(function(i)
        if on and (i.UserInputType == Enum.UserInputType.MouseMovement or i == di) then
            set(i.Position.X)
        end
    end))
    table.insert(conns, uis.InputEnded:Connect(function(i)
        if i == di or i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on, di = false, nil
        end
    end))
end

local function form(phs, bt, cb)
    local cnt = #phs
    local cd = card(10 + cnt * 40)
    local bx = {}
    for i, ph in ipairs(phs) do
        local b = tx("TextBox", {Size = UDim2.new(1, i == cnt and -112 or -24, 0, 30), Position = UDim2.fromOffset(12, 10 + (i - 1) * 40), BackgroundColor3 = col.panel, PlaceholderText = ph, PlaceholderColor3 = col.sub, Text = "", TextColor3 = col.text, TextSize = 12, ClearTextOnFocus = false, TextXAlignment = Enum.TextXAlignment.Left}, cd)
        rc(b, 9)
        pd(b, 10)
        bx[i] = b
    end
    local function submit()
        local vals = {}
        for i, b in ipairs(bx) do
            vals[i] = b.Text
        end
        cb(vals)
    end
    local sb = tx("TextButton", {Size = UDim2.fromOffset(80, 30), Position = UDim2.new(1, -92, 0, 10 + (cnt - 1) * 40), BackgroundColor3 = col.acc, Text = bt, TextColor3 = col.text, TextSize = 12, AutoButtonColor = false}, cd, true)
    rc(sb, 9)
    hover(sb)
    table.insert(conns, sb.MouseButton1Click:Connect(submit))
    table.insert(conns, bx[cnt].FocusLost:Connect(function(e)
        if e then
            submit()
        end
    end))
end

local function textin(ph, cb)
    local cd = card(48)
    local b = tx("TextBox", {Size = UDim2.new(1, -24, 0, 30), Position = UDim2.fromOffset(12, 9), BackgroundColor3 = col.panel, PlaceholderText = ph, PlaceholderColor3 = col.sub, Text = "", TextColor3 = col.text, TextSize = 12, ClearTextOnFocus = false, TextXAlignment = Enum.TextXAlignment.Left}, cd)
    rc(b, 9)
    pd(b, 10)
    local tok = 0
    table.insert(conns, b:GetPropertyChangedSignal("Text"):Connect(function()
        tok += 1
        local id = tok
        task.delay(0.05, function()
            if id == tok and not dead then
                cb(b.Text)
            end
        end)
    end))
end

local function colorin(n, init, cb)
    local cd = card(48)
    tx("TextLabel", {Size = UDim2.new(1, -212, 1, 0), Position = UDim2.fromOffset(14, 0), BackgroundTransparency = 1, Text = n, TextColor3 = col.text, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd}, cd)
    local pv = mk("Frame", {Size = UDim2.fromOffset(22, 22), Position = UDim2.new(1, -190, 0.5, -11), BackgroundColor3 = init, BorderSizePixel = 0}, cd)
    rc(pv, 6)
    st(pv, Color3.new(1, 1, 1), 0.7)
    local bx = tx("TextBox", {Size = UDim2.fromOffset(90, 30), Position = UDim2.new(1, -160, 0.5, -15), BackgroundColor3 = col.panel, Text = tohex(init), TextColor3 = col.text, TextSize = 12, ClearTextOnFocus = false}, cd)
    rc(bx, 9)
    local sb = tx("TextButton", {Size = UDim2.fromOffset(58, 30), Position = UDim2.new(1, -64, 0.5, -15), BackgroundColor3 = col.acc, Text = "Set", TextColor3 = col.text, TextSize = 12, AutoButtonColor = false}, cd, true)
    rc(sb, 9)
    hover(sb)
    local function submit()
        local cr = hex(bx.Text)
        if not cr then
            notify("Invalid color")
            return
        end
        pv.BackgroundColor3 = cr
        bx.Text = tohex(cr)
        cb(cr)
    end
    table.insert(conns, sb.MouseButton1Click:Connect(submit))
    table.insert(conns, bx.FocusLost:Connect(function(e)
        if e then
            submit()
        end
    end))
end

local function resetall()
    rgraphics(true)
    rlight(true)
    reffects(true)
    rgt()
    fs.spec, fs.key, fs.game, fs.menu = nil, nil, true, true
    menufont()
    cf.theme, cf.size, cf.tr, cf.key, cf.notif, cf.btn = dtheme, 1, 0.12, Enum.KeyCode.RightShift, true, true
    s.fps, s.png, s.mem = false, false, false
    for _, l in pairs(sl) do
        l.Visible = false
    end
    obtn.Visible = true
    theme(true)
    resetpos()
    refresh()
    later()
    notify("Defaults restored")
end

pages.performance = function()
    section("PERFORMANCE")
    toggle("Fix Lag", s.lag, setlag)
    for _, d in ipairs({{"Show FPS", "fps"}, {"Show Ping", "png"}, {"Show Memory", "mem"}}) do
        toggle(d[1], s[d[2]], function(v)
            s[d[2]] = v
            sl[d[2]].Visible = v
        end)
    end
    section("GRAPHICS")
    toggle("Remove Shadows", s.shd, function(v)
        s.shd = v
        shadows()
    end)
    for _, d in ipairs({{"Remove Particles", "ptc", "e"}, {"Remove Textures", "tex", "d"}, {"Remove Post Effects", "pst", "p"}, {"Smooth Materials", "mat", "m"}}) do
        toggle(d[1], s[d[2]], function(v)
            s[d[2]] = v
            setkind(d[3])
        end)
    end
    toggle("Simple Water", s.wtr, function(v)
        s.wtr = v
        water()
    end)
    toggle("Low Quality Level", s.low, function(v)
        s.low = v
        quality()
    end)
    btn("Restore Graphics", "Restore", function()
        rgraphics()
        refresh()
    end)
end

pages.sky = function()
    section("PRESETS")
    btn("Random Sky", "Random", randomsky)
    for _, p in ipairs(pskies) do
        btn(p.n, "Apply", function()
            applysky(p)
        end)
    end
    section("SPACE")
    for _, p in ipairs(pspace) do
        btn(p.n, "Apply", function()
            applyspace(p)
        end)
    end
    section("CUSTOM SKY")
    form({"Name", "Asset ID (1 or 6, separated by comma)"}, "Add", function(v)
        local ids = pids(v[2])
        if not ids then
            notify("Need 1 or 6 valid IDs")
            return
        end
        local n = trim(v[1])
        if n == "" then
            n = "Sky " .. (#sv.skies + 1)
        end
        local e = {Name = n, Ids = ids}
        table.insert(sv.skies, e)
        later()
        applycustom(e)
    end)
    for i, e in ipairs(sv.skies) do
        row(e.Name, {
            {t = "Apply", f = function()
                applycustom(e)
            end},
            {t = "X", w = 34, c = col.red, f = function()
                table.remove(sv.skies, i)
                later()
                refresh()
            end}
        })
    end
    section("LOCAL SKIES")
    if not lfs then
        row("Executor has no local file support", {})
    else
        if not lsk then
            scanlocal()
        end
        btn("Rescan CatSkies folder", "Rescan", function()
            scanlocal()
            refresh()
        end)
        if #lsk == 0 then
            row("Put sky folders in the CatSkies folder", {})
        end
        for _, e in ipairs(lsk) do
            btn(e.n, "Apply", function()
                applylocal(e)
            end)
        end
    end
    section("SKY SETTINGS")
    local sk = cursky()
    slider("Star Count", 0, 5000, rp(sk, "StarCount", 3000), function(v)
        catsky().StarCount = v
    end, 100)
    slider("Sun Size", 0, 60, rp(sk, "SunAngularSize", 21), function(v)
        catsky().SunAngularSize = v
    end, 1)
    slider("Moon Size", 0, 60, rp(sk, "MoonAngularSize", 11), function(v)
        catsky().MoonAngularSize = v
    end, 1)
    toggle("Sun and Moon", rp(sk, "CelestialBodiesShown", true), function(v)
        catsky().CelestialBodiesShown = v
    end)
    section("ANIMATION")
    slider("Transition Time", 0, 5, s.gl, function(v)
        s.gl = v
    end, 0.1)
    slider("Sky Rotation", -20, 20, s.spin, function(v)
        s.spin = v
        if v ~= 0 then
            catsky()
        end
    end, 0.5)
    slider("Time Flow", -1, 1, s.flow, function(v)
        s.flow = v
    end, 0.05)
    btn("Default Sky", "Restore", function()
        rlight()
        refresh()
    end)
end

pages.lighting = function()
    section("TIME")
    slider("Time of Day", 0, 24, lt.ClockTime, setclock, 0.1)
    toggle("Lock Time", s.lck, function(v)
        s.lck = v
        s.lckt = lt.ClockTime
        lockclock()
    end)
    section("LIGHT")
    slider("Brightness", 0, 6, lt.Brightness, function(v)
        lt.Brightness = v
    end, 0.1)
    slider("Exposure", -3, 3, lt.ExposureCompensation, function(v)
        lt.ExposureCompensation = v
    end, 0.1)
    for _, d in ipairs({{"Diffuse Scale", "EnvironmentDiffuseScale"}, {"Specular Scale", "EnvironmentSpecularScale"}}) do
        slider(d[1], 0, 1, rp(lt, d[2], 1), function(v)
            pcall(function()
                lt[d[2]] = v
            end)
        end, 0.05)
    end
    toggle("Fog", s.fog, fog)
    section("COLORS")
    for _, d in ipairs({{"Ambient", "Ambient"}, {"Outdoor Ambient", "OutdoorAmbient"}, {"Fog Color", "FogColor"}, {"Shift Top", "ColorShift_Top"}, {"Shift Bottom", "ColorShift_Bottom"}}) do
        colorin(d[1], lt[d[2]], function(cr)
            lt[d[2]] = cr
        end)
    end
    section("ATMOSPHERE")
    local a = curatm()
    for _, d in ipairs({{"Density", 0, 1, 0.395, 0.01}, {"Offset", 0, 1, 0, 0.01}, {"Haze", 0, 10, 0, 0.1}, {"Glare", 0, 10, 0, 0.1}}) do
        slider(d[1], d[2], d[3], rp(a, d[1], d[4]), function(v)
            setatm(d[1], v)
        end, d[5])
    end
    colorin("Atmosphere Color", rp(a, "Color", Color3.fromRGB(199, 170, 107)), function(cr)
        setatm("Color", cr)
    end)
    colorin("Atmosphere Decay", rp(a, "Decay", Color3.fromRGB(92, 60, 13)), function(cr)
        setatm("Decay", cr)
    end)
    btn("Restore Lighting", "Restore", function()
        rlight()
        refresh()
    end)
end

local function fx(sec, k, lbl, list)
    section(sec)
    toggle(lbl, s[k], function(v)
        seteffect(k, v)
    end)
    for _, d in ipairs(list) do
        slider(d[1], d[2], d[3], ev[k][d[4]], function(v)
            setev(k, d[4], v)
        end, d[5])
    end
end

pages.effects = function()
    fx("BLOOM", "bloom", "Bloom", {{"Bloom Intensity", 0, 3, "Intensity", 0.05}, {"Bloom Size", 0, 56, "Size", 1}, {"Bloom Threshold", 0, 4, "Threshold", 0.05}})
    fx("BLUR", "blur", "Blur", {{"Blur Size", 0, 56, "Size", 1}})
    fx("SUN RAYS", "rays", "Sun Rays", {{"Rays Intensity", 0, 1, "Intensity", 0.01}, {"Rays Spread", 0, 1, "Spread", 0.01}})
    fx("DEPTH OF FIELD", "dof", "Depth of Field", {{"Far Intensity", 0, 1, "FarIntensity", 0.01}, {"Focus Distance", 0, 200, "FocusDistance", 1}, {"Focus Radius", 0, 100, "InFocusRadius", 1}})
    section("COLOR")
    for _, d in ipairs({{"Color Brightness", -0.5, 0.5, "Brightness", 0.01}, {"Contrast", -1, 1, "Contrast", 0.05}, {"Saturation", -1, 1, "Saturation", 0.05}}) do
        slider(d[1], d[2], d[3], cv[d[4]], function(v)
            setcc(d[4], v)
        end, d[5])
    end
    colorin("Tint", cv.TintColor, function(cr)
        setcc("TintColor", cr)
    end)
    section("CAMERA")
    toggle("Custom FOV", s.cfov, setfov)
    slider("Field of View", 30, 120, s.fov, function(v)
        s.fov = v
    end, 1)
    btn("Reset Effects", "Reset", function()
        reffects()
        refresh()
    end)
end

pages.fonts = function()
    table.clear(fb)
    section("TARGET")
    toggle("Game Text", fs.game, setgame)
    toggle("Menu Text", fs.menu, setmenu)
    section("CUSTOM FONT")
    form({"Font name or Font ID"}, "Add", function(v)
        local val = trim(v[1])
        local sp = pfont(val, true)
        if not sp then
            notify("Font not found")
            return
        end
        if sp.f then
            local ex = false
            for _, e in ipairs(sv.fonts) do
                if e.Value == val then
                    ex = true
                    break
                end
            end
            if not ex then
                table.insert(sv.fonts, {Name = val, Value = val})
            end
        end
        setfont(sp)
        refresh()
    end)
    for i, e in ipairs(sv.fonts) do
        local sp = pfont(e.Value)
        if sp then
            local _, m = row(e.Name, {
                {t = "Apply", f = function()
                    setfont(sp)
                end},
                {t = "X", w = 34, c = col.red, f = function()
                    table.remove(sv.fonts, i)
                    later()
                    refresh()
                end}
            })
            fb[sp.k] = m[1]
        end
    end
    section("FONTS")
    local rws = {}
    textin("Search font", function(q)
        q = q:lower()
        for _, r in ipairs(rws) do
            r.c.Visible = q == "" or r.n:find(q, 1, true) ~= nil
        end
    end)
    btn("Default", "Reset", function()
        setfont(nil)
    end, col.red)
    for _, it in ipairs(fitems) do
        local sp = {e = it, k = "Enum:" .. it.Name}
        local cd, m = row(it.Name, {{t = "Apply", f = function()
            setfont(sp)
        end}}, it)
        fb[sp.k] = m[1]
        table.insert(rws, {c = cd, n = it.Name:lower()})
    end
    fbrefresh()
end

local themes = {
    {"Purple", Color3.fromRGB(120, 105, 255)},
    {"Blue", Color3.fromRGB(70, 150, 255)},
    {"Cyan", Color3.fromRGB(55, 210, 220)},
    {"Pink", Color3.fromRGB(255, 105, 175)},
    {"Red", Color3.fromRGB(245, 85, 100)},
    {"Green", Color3.fromRGB(70, 200, 140)},
    {"Orange", Color3.fromRGB(255, 150, 75)},
    {"White", Color3.fromRGB(230, 230, 245)}
}

pages.interface = function()
    section("WINDOW")
    slider("Menu Size", 0.7, 1.2, cf.size, function(v)
        cf.size = v
        fit()
        later()
    end, 0.05)
    slider("Transparency", 0, 0.5, cf.tr, function(v)
        cf.tr = v
        main.BackgroundTransparency = v
        later()
    end, 0.01)
    toggle("Notifications", cf.notif, function(v)
        cf.notif = v
        later()
    end)
    toggle("Show Button", cf.btn, function(v)
        cf.btn = v
        obtn.Visible = v
        later()
        if not v then
            notify("Press " .. cf.key.Name .. " to open")
        end
    end)
    form({"Toggle key (" .. cf.key.Name .. ")"}, "Set", function(v)
        local ok, k = pcall(function()
            return Enum.KeyCode[trim(v[1])]
        end)
        if not ok or not k then
            notify("Invalid key")
            return
        end
        cf.key = k
        later()
        notify("Key set to " .. k.Name)
        refresh()
    end)
    btn("Reset Position", "Reset", function()
        resetpos()
    end)
    section("THEME")
    for _, t in ipairs(themes) do
        btn(t[1], "Apply", function()
            cf.theme = t[2]
            theme()
            refresh()
        end, t[2])
    end
    colorin("Custom Color", cf.theme, function(cr)
        cf.theme = cr
        theme()
        refresh()
    end)
end

pages.general = function()
    section("RESTORE")
    btn("Restore Graphics", "Restore", function()
        rgraphics()
        refresh()
    end)
    btn("Restore Lighting", "Restore", function()
        rlight()
        refresh()
    end)
    btn("Restore Effects", "Restore", function()
        reffects()
        refresh()
    end)
    btn("Restore Fonts", "Restore", function()
        setfont(nil)
    end)
    btn("Restore Defaults", "Reset", resetall)
    section("SCRIPT")
    btn("Unload Script", "Unload", function()
        shutdown()
    end, col.red)
end

openpage = function(name, keep)
    local build = pages[name:lower()]
    if not build or dead then
        return
    end
    local pos = holder.CanvasPosition
    cur = name
    title.Text = name
    for _, x in ipairs(conns) do
        x:Disconnect()
    end
    table.clear(conns)
    clear()
    build()
    for i, n in ipairs(pnames) do
        local on = n == name
        tn(navs[n], {BackgroundColor3 = on and col.acc or col.panel})
        if on then
            tn(ind, {Position = UDim2.fromOffset(4, 124 + (i - 1) * 42)}, 0.25)
        end
    end
    holder.Position = keep and UDim2.fromOffset(0, 55) or UDim2.fromOffset(18, 55)
    tn(holder, {Position = UDim2.fromOffset(0, 55)}, 0.25)
    if keep then
        task.spawn(function()
            rs.Heartbeat:Wait()
            rs.Heartbeat:Wait()
            if not dead and holder.Parent then
                holder.CanvasPosition = pos
            end
        end)
    else
        holder.CanvasPosition = Vector2.new(0, 0)
    end
end

refresh = function()
    if cur then
        openpage(cur, true)
    end
end

hover(minb, life)
hover(closeb, life)
ind = mk("Frame", {Size = UDim2.fromOffset(4, 20), Position = UDim2.fromOffset(4, 124), BackgroundColor3 = col.acc, BorderSizePixel = 0}, side)
rc(ind, 2)

for i, n in ipairs(pnames) do
    local b = tx("TextButton", {Name = n, Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = col.panel, Text = n, TextColor3 = col.text, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left, AutoButtonColor = false, LayoutOrder = i}, nav)
    rc(b, 9)
    pd(b, 12)
    hover(b, life)
    b.MouseButton1Click:Connect(function()
        openpage(n)
    end)
    navs[n] = b
end

obtn = mk("ImageButton", {Name = "CatButton", Size = UDim2.fromOffset(58, 58), Position = UDim2.new(0, 25, 0.5, -29), BackgroundColor3 = col.panel, Image = "rbxassetid://" .. cf.avatar, ScaleType = Enum.ScaleType.Crop, AutoButtonColor = false, Visible = cf.btn}, gui)
rc(obtn, 18)
ast(obtn, 0.45)
fallback(obtn, 13)

local moved = drag(obtn, obtn, false)
drag(main, head, true)

resetpos = function()
    main.Position = UDim2.fromScale(0.5, 0.5)
    obtn.Position = UDim2.new(0, 25, 0.5, -29)
    for k, l in pairs(sl) do
        l.Position = spos[k]
    end
    notify("Position reset")
end

local function layout(m)
    mini = m
    side.Visible = not m
    holder.Visible = not m
    body.Position = m and UDim2.fromOffset(12, 10) or UDim2.fromOffset(185, 14)
    body.Size = m and UDim2.new(1, -24, 0, 40) or UDim2.new(1, -195, 1, -28)
    minb.Text = m and "+" or "−"
end

local function setmin(m)
    layout(m)
    if m then
        title.Text = "CAT SETTINGS"
        main.Size = UDim2.fromOffset(250, 60)
    else
        main.Size = UDim2.fromOffset(700, 460)
        openpage(cur or "Performance", true)
    end
end

local function openmenu()
    if dead then
        return
    end
    anim += 1
    layout(false)
    main.Visible = true
    main.Size = UDim2.fromOffset(620, 400)
    tn(main, {Size = UDim2.fromOffset(700, 460)}, 0.25)
    openpage(cur or "Performance", true)
end

local function closemenu()
    if dead then
        return
    end
    anim += 1
    local id = anim
    tn(main, {Size = UDim2.fromOffset(620, 400)}, 0.18)
    task.delay(0.2, function()
        if anim == id and main.Parent then
            main.Visible = false
        end
    end)
end

local function togglemenu()
    if main.Visible then
        closemenu()
    else
        openmenu()
    end
end

shutdown = function()
    if dead then
        return
    end
    save()
    dead = true
    anim += 1
    sweeptok += 1
    if lockconn then
        lockconn:Disconnect()
        lockconn = nil
    end
    if camconn then
        camconn:Disconnect()
        camconn = nil
    end
    for _, x in ipairs(conns) do
        x:Disconnect()
    end
    for _, x in ipairs(life) do
        x:Disconnect()
    end
    table.clear(conns)
    table.clear(life)
    pcall(function()
        rs:UnbindFromRenderStep("CatSettingsFOV")
    end)
    rgraphics(true)
    rlight(true)
    reffects(true)
    rgt()
    if env.CatSettingsUnload == shutdown then
        env.CatSettingsUnload = nil
    end
    if gui then
        gui:Destroy()
    end
end

minb.MouseButton1Click:Connect(function()
    setmin(not mini)
end)
closeb.MouseButton1Click:Connect(closemenu)
obtn.MouseButton1Click:Connect(function()
    if moved() then
        return
    end
    togglemenu()
end)

table.insert(life, uis.InputBegan:Connect(function(i, processed)
    if processed or dead then
        return
    end
    if i.KeyCode == cf.key then
        togglemenu()
    end
end))

local ff, ft, t2, t3 = 0, 0, 0, 0

table.insert(life, rs.RenderStepped:Connect(function(dt)
    if dead then
        return
    end
    ff += 1
    ft += dt
    if s.spin ~= 0 then
        local sk = lt:FindFirstChild("CatSky")
        if sk then
            local o = sk.SkyboxOrientation
            sk.SkyboxOrientation = Vector3.new(o.X, (o.Y + s.spin * dt) % 360, o.Z)
        end
    end
    if ft >= 0.5 then
        if s.fps and sl.fps.Visible then
            local n = math.floor(ff / ft + 0.5)
            sl.fps.Text = "FPS: " .. n
            sl.fps.TextColor3 = n >= 50 and col.green or (n >= 30 and col.yellow or col.red)
        end
        ff, ft = 0, 0
    end
end))

table.insert(life, rs.Heartbeat:Connect(function(dt)
    if dead then
        return
    end
    t2 += dt
    t3 += dt
    if s.flow ~= 0 and not s.lck then
        lt.ClockTime = (lt.ClockTime + s.flow * dt) % 24
    end
    if t2 >= 1 then
        t2 = 0
        if s.png and sl.png.Visible then
            local ms
            local ok, p = pcall(function()
                return lp:GetNetworkPing() * 1000
            end)
            if ok and p then
                ms = p
            else
                local ok2, v = pcall(function()
                    return stats.Network.ServerStatsItem["Data Ping"]:GetValue()
                end)
                if ok2 and v then
                    ms = v
                end
            end
            if ms then
                sl.png.Text = "PING: " .. math.floor(ms) .. " ms"
            end
        end
        if s.mem and sl.mem.Visible then
            local ok, v = pcall(function()
                return stats:GetTotalMemoryUsageMb()
            end)
            if ok and v then
                sl.mem.Text = "MEM: " .. math.floor(v) .. " MB"
            end
        end
    end
    if t3 >= 0.5 then
        t3 = 0
        local sp = fs.spec
        if sp and fs.game then
            for o in pairs(gtx) do
                if o.Parent then
                    local ok, m = pcall(same, o, sp)
                    if ok and not m then
                        pcall(apply, o, sp)
                    end
                else
                    gtx[o] = nil
                end
            end
        end
    end
end))

local function handle(o)
    if dead or not o.Parent then
        return
    end
    local k = kind(o)
    if k and s[kf[k]] then
        pcall(function()
            local d = capture(o, k)
            if d then
                applyd(o, d)
            end
        end)
    end
    if fs.spec and fs.game and isgt(o) then
        stylegt(o)
    end
end

local function added(o)
    if dead then
        return
    end
    local fa = fs.spec ~= nil and fs.game
    if not (fa or s.ptc or s.tex or s.pst or s.mat) then
        return
    end
    local k = kind(o)
    if (k ~= nil and s[kf[k]]) or (fa and isgt(o)) then
        task.defer(handle, o)
    end
end

table.insert(life, workspace.DescendantAdded:Connect(added))
table.insert(life, lt.DescendantAdded:Connect(added))
if pg then
    table.insert(life, pg.DescendantAdded:Connect(function(o)
        if not dead and fs.spec ~= nil and fs.game and isgt(o) then
            task.defer(handle, o)
        end
    end))
end

watch()
table.insert(life, workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(watch))

if sv.active then
    local sp = skey(sv.active)
    if sp then
        setfont(sp, true)
    end
end

theme(true)
env.CatSettingsUnload = shutdown
openpage("Performance")
notify("CAT SETTINGS loaded")
