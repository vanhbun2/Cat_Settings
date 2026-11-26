local plrs=game:GetService("Players")
local lt=game:GetService("Lighting")
local tw=game:GetService("TweenService")
local uis=game:GetService("UserInputService")
local rs=game:GetService("RunService")
local stats=game:GetService("Stats")
local http=game:GetService("HttpService")
local tsv=game:GetService("TextService")
local lp=plrs.LocalPlayer
local pg=lp:WaitForChild("PlayerGui", 10) or lp:FindFirstChildOfClass("PlayerGui")
local env=(getgenv and getgenv()) or _G
if env.CatSettingsUnload then
    pcall(env.CatSettingsUnload)
    env.CatSettingsUnload=nil
end
local old=pg and pg:FindFirstChild("CatSettings")
if old then
    old:Destroy()
end
local file="CatSettings.json"
local cs=type(writefile) == "function" and type(readfile) == "function" and type(isfile) == "function"
local dtheme=Color3.fromRGB(120, 105, 255)
local cf={
    avatar="122586817481447",
    theme=dtheme,
    size=1,
    tr=0.12,
    key=Enum.KeyCode.RightShift,
    notif=true,
    btn=true,
    rgb=false,
    sl={},
    dc=""
}
local fs={
    spec=nil,
    key=nil,
    game=true,
    menu=true
}
local sv={
    skies={},
    fonts={},
    active=nil
}
local function trim(v)
    return tostring(v):match("^%s*(.-)%s*$") or ""
end
local function nid(v)
    local t=tostring(v)
    return t:match("^%s*(%d+)%s*$") or t:match("rbxassetid://(%d+)") or t:match("[?&]id=(%d+)") or t:match("/library/(%d+)") or t:match("/asset/(%d+)") or t:match("/catalog/(%d+)")
end
local function hex(v)
    local h=tostring(v):gsub("[#%s]", "")
    if #h ~= 6 or not h:match("^%x+$") then
        return nil
    end
    return Color3.fromRGB(tonumber(h:sub(1, 2), 16), tonumber(h:sub(3, 4), 16), tonumber(h:sub(5, 6), 16))
end
local function tohex(c)
    return string.format("#%02X%02X%02X", math.round(math.clamp(c.R, 0, 1) * 255), math.round(math.clamp(c.G, 0, 1) * 255), math.round(math.clamp(c.B, 0, 1) * 255))
end
local function loadcfg()
    if not cs then
        return
    end
    local ok, d=pcall(function()
        if isfile(file) then
            return http:JSONDecode(readfile(file))
        end
    end)
    if not ok or type(d) ~= "table" then
        return
    end
    if type(d.Theme) == "table" then
        local r=tonumber(d.Theme[1])
        local g=tonumber(d.Theme[2])
        local b=tonumber(d.Theme[3])
        if r and g and b then
            cf.theme=Color3.fromRGB(math.clamp(r, 0, 255), math.clamp(g, 0, 255), math.clamp(b, 0, 255))
        end
    end
    if type(d.MenuSize) == "number" then
        cf.size=math.clamp(d.MenuSize, 0.7, 1.2)
    end
    if type(d.Transparency) == "number" then
        cf.tr=math.clamp(d.Transparency, 0, 0.5)
    end
    if type(d.Notifications) == "boolean" then
        cf.notif=d.Notifications
    end
    if type(d.ShowButton) == "boolean" then
        cf.btn=d.ShowButton
    end
    if type(d.RgbBorder) == "boolean" then
        cf.rgb=d.RgbBorder
    end
    if type(d.Crosshair) == "table" then
        cf.sl=d.Crosshair
    end
    if type(d.DiscordInvite) == "string" then
        cf.dc=d.DiscordInvite
    end
    if type(d.Key) == "string" then
        local f, k=pcall(function()
            return Enum.KeyCode[d.Key]
        end)
        if f and k then
            cf.key=k
        end
    end
    if type(d.GameText) == "boolean" then
        fs.game=d.GameText
    end
    if type(d.MenuText) == "boolean" then
        fs.menu=d.MenuText
    end
    if type(d.ActiveFont) == "string" then
        sv.active=d.ActiveFont
        fs.key=d.ActiveFont
    end
    if type(d.Skies) == "table" then
        for _, e in ipairs(d.Skies) do
            if type(e) == "table" and type(e.Name) == "string" and type(e.Ids) == "table" then
                local ids={}
                for _, r in ipairs(e.Ids) do
                    local n=nid(r)
                    if n then
                        table.insert(ids, n)
                    end
                end
                if #ids == 1 or #ids == 6 then
                    table.insert(sv.skies, {Name=e.Name, Ids=ids})
                end
            end
        end
    end
    if type(d.Fonts) == "table" then
        for _, e in ipairs(d.Fonts) do
            if type(e) == "table" and type(e.Name) == "string" and type(e.Value) == "string" then
                table.insert(sv.fonts, {Name=e.Name, Value=e.Value})
            end
        end
    end
end
loadcfg()
local col={
    bg=Color3.fromRGB(15, 17, 25),
    panel=Color3.fromRGB(23, 26, 37),
    card=Color3.fromRGB(31, 35, 49),
    text=Color3.fromRGB(245, 246, 255),
    sub=Color3.fromRGB(153, 160, 180),
    acc=cf.theme,
    green=Color3.fromRGB(90, 220, 155),
    yellow=Color3.fromRGB(255, 205, 90),
    red=Color3.fromRGB(255, 100, 110),
    off=Color3.fromRGB(65, 68, 82)
}
local function weak()
    return setmetatable({}, {__mode="k"})
end
local olt={}
local owt={}
local oobj=weak()
local gtx=weak()
local mtx=weak()
local hsky={}
local hatm={}
local conns={}
local life={}
local astk={}
local scanned={}
local stok={}
local navs={}
local pages={}
local fb={}
local pnames={"Performance", "Sky", "Lighting", "Effects", "Shift Lock", "Servers", "Fonts", "Interface", "Discord", "General"}
local cam0=workspace.CurrentCamera
local s={
    lag=false,
    shd=false,
    ptc=false,
    tex=false,
    pst=false,
    mat=false,
    wtr=false,
    low=false,
    fps=false,
    png=false,
    mem=false,
    fog=true,
    lck=false,
    lckt=lt.ClockTime,
    bloom=false,
    blur=false,
    rays=false,
    dof=false,
    cfov=false,
    fov=cam0 and cam0.FieldOfView or 70,
    gl=1.5,
    spin=0,
    flow=0,
    cyc=0,
    hue=0
}
local kf={
    e="ptc",
    p="pst",
    d="tex",
    m="mat"
}
local bm={
    Gotham="GothamBold",
    GothamSemibold="GothamBold",
    GothamBlack="GothamBlack",
    SourceSans="SourceSansBold",
    SourceSansLight="SourceSansBold",
    SourceSansSemibold="SourceSansBold",
    Arial="ArialBold"
}
local gui
local main
local scale
local side
local nav
local body
local head
local holder
local title
local obtn
local toast
local minb
local closeb
local ind
local fade
local cur
local openpage
local refresh
local shutdown
local resetpos
local dead=false
local mini=false
local anim=0
local order=0
local sweeptok=0
local toasttok=0
local savetok=0
local lockconn
local camconn
local mscale=cf.size
local cseq=0
local runtok=0
local sztween
local scaletw
local function mk(cls, p, par)
    local o=Instance.new(cls)
    for k, v in pairs(p) do
        o[k]=v
    end
    o.Parent=par
    return o
end
local function rc(o, r)
    local i=Instance.new("UICorner")
    i.CornerRadius=UDim.new(0, r or 12)
    i.Parent=o
end
local function st(o, cr, t)
    local i=Instance.new("UIStroke")
    i.Color=cr or Color3.new(1, 1, 1)
    i.Transparency=t or 0.85
    i.Thickness=1
    i.Parent=o
    return i
end
local function ast(o, t)
    table.insert(astk, st(o, col.acc, t))
end
local function pd(o, l)
    local i=Instance.new("UIPadding")
    i.PaddingLeft=UDim.new(0, l or 10)
    i.PaddingRight=UDim.new(0, 10)
    i.Parent=o
end
local function tn(o, p, d)
    if o and o.Parent then
        local t=tw:Create(o, TweenInfo.new(d or 0.2, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), p)
        t:Play()
        return t
    end
end
local function apply(o, sp)
    if not o or not o.Parent or not sp then
        return
    end
    if sp.e then
        o.Font=sp.e
    elseif sp.f then
        o.FontFace=sp.f
    end
end
local function mspec(b)
    local sp=fs.menu and fs.spec
    if not sp then
        return {e=b and Enum.Font.GothamBold or Enum.Font.Gotham}
    end
    if sp.e then
        local n=bm[sp.e.Name]
        if b and n then
            return {e=Enum.Font[n]}
        end
        return sp
    end
    if b and sp.f then
        return {f=Font.new(sp.f.Family, Enum.FontWeight.Bold, sp.f.Style)}
    end
    return sp
end
local function tx(cls, p, par, b)
    local o=mk(cls, p, par)
    mtx[o]=b == true
    pcall(apply, o, mspec(b == true))
    return o
end
local function menufont()
    for o, b in pairs(mtx) do
        if o.Parent then
            pcall(apply, o, mspec(b))
        else
            mtx[o]=nil
        end
    end
end
local function notify(m)
    if dead or not cf.notif or not toast or not toast.Parent then
        return
    end
    toasttok=toasttok+1
    local id=toasttok
    toast.Text=m
    toast.Visible=true
    toast.Position=UDim2.new(0.5, 0, 0, -50)
    tn(toast, {Position=UDim2.new(0.5, 0, 0, 14)}, 0.25)
    task.delay(2.2, function()
        if dead or id ~= toasttok or not toast.Parent then
            return
        end
        tn(toast, {Position=UDim2.new(0.5, 0, 0, -60)}, 0.25)
        task.delay(0.3, function()
            if id == toasttok and toast.Parent then
                toast.Visible=false
            end
        end)
    end)
end
local fitems={}
for _, i in ipairs(Enum.Font:GetEnumItems()) do
    if i.Name ~= "Legacy" and i.Name ~= "Unknown" then
        table.insert(fitems, i)
    end
end
table.sort(fitems, function(a, b)
    return a.Name < b.Name
end)
local function loads(face)
    local done=false
    local bad=false
    local res=nil
    task.spawn(function()
        local ok, r=pcall(function()
            local p=Instance.new("GetTextBoundsParams")
            p.Text="Test"
            p.Font=face
            p.Size=14
            p.Width=1000
            return tsv:GetTextBoundsAsync(p)
        end)
        bad=not ok
        res=r
        done=true
    end)
    local t=0
    while not done and t<1 do
        task.wait(0.1)
        t=t+0.1
    end
    if not done then
        return true
    end
    return not bad and typeof(res)=="Vector2"
end
local function pfont(v, check)
    v=trim(v)
    if v == "" then
        return nil
    end
    for _, i in ipairs(fitems) do
        if i.Name:lower() == v:lower() then
            return {e=i, k="Enum:" .. i.Name}
        end
    end
    local id=nid(v)
    local fam=id and ("rbxassetid://" .. id) or (v:find("^rbxasset") and v)
    if not fam then
        return nil
    end
    local ok, face=pcall(Font.new, fam, Enum.FontWeight.Regular, Enum.FontStyle.Normal)
    if not ok or (check and not loads(face)) then
        return nil
    end
    return {f=face, k="Face:" .. fam}
end
local function skey(k)
    return pfont(k:match("^%a+:(.+)$") or "")
end
local function save()
    if not cs then
        return
    end
    local d={
        Theme={math.round(cf.theme.R * 255), math.round(cf.theme.G * 255), math.round(cf.theme.B * 255)},
        MenuSize=cf.size,
        Transparency=cf.tr,
        Key=cf.key.Name,
        Notifications=cf.notif,
        ShowButton=cf.btn,
        RgbBorder=cf.rgb,
        Crosshair=cf.sl,
        DiscordInvite=cf.dc,
        Skies=sv.skies,
        Fonts=sv.fonts,
        ActiveFont=fs.key,
        GameText=fs.game,
        MenuText=fs.menu
    }
    pcall(function()
        writefile(file, http:JSONEncode(d))
    end)
end
local function later()
    if not cs then
        return
    end
    savetok=savetok+1
    local id=savetok
    task.delay(0.6, function()
        if id == savetok and not dead then
            save()
        end
    end)
end
local function drag(o, h, scaled)
    local on=false
    local di=nil
    local ds=nil
    local sp=nil
    local mv=false
    table.insert(life, h.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on=true
            mv=false
            di=i
            ds=i.Position
            sp=o.Position
        end
    end))
    table.insert(life, uis.InputChanged:Connect(function(i)
        if not on then
            return
        end
        if i.UserInputType == Enum.UserInputType.MouseMovement or i == di then
            local d=i.Position - ds
            if not mv and d.Magnitude < 6 then
                return
            end
            mv=true
            local f=(scaled and scale and scale.Scale) or 1
            if f <= 0 then
                f=1
            end
            o.Position=UDim2.new(sp.X.Scale, sp.X.Offset + d.X / f, sp.Y.Scale, sp.Y.Offset + d.Y / f)
        end
    end))
    table.insert(life, uis.InputEnded:Connect(function(i)
        if i == di or i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on=false
            di=nil
        end
    end))
    return function()
        return mv
    end
end
for _, p in ipairs({"Brightness", "ClockTime", "Ambient", "OutdoorAmbient", "ColorShift_Top", "ColorShift_Bottom", "FogColor", "FogStart", "FogEnd", "ExposureCompensation", "GlobalShadows", "EnvironmentDiffuseScale", "EnvironmentSpecularScale"}) do
    local ok, v=pcall(function()
        return lt[p]
    end)
    if ok then
        olt[p]=v
    end
end
local ter=workspace:FindFirstChildOfClass("Terrain")
if ter then
    for _, p in ipairs({"WaterWaveSize", "WaterWaveSpeed", "WaterReflectance"}) do
        pcall(function()
            owt[p]=ter[p]
        end)
    end
end
local oq
pcall(function()
    oq=settings().Rendering.QualityLevel
end)
gui=mk("ScreenGui", {Name="CatSettings", ResetOnSpawn=false, DisplayOrder=50, ZIndexBehavior=Enum.ZIndexBehavior.Sibling}, pg or (gethui and gethui()) or game:GetService("CoreGui"))
fade=mk("Frame", {Name="CatFade", Size=UDim2.fromScale(1, 1), Position=UDim2.fromOffset(0, 0), BackgroundColor3=Color3.new(0, 0, 0), BackgroundTransparency=1, BorderSizePixel=0, ZIndex=30, Active=false}, gui)
main=mk("Frame", {Name="SettingsPanel", Size=UDim2.fromOffset(700, 460), AnchorPoint=Vector2.new(0.5, 0.5), Position=UDim2.fromScale(0.5, 0.5), BackgroundColor3=col.bg, BackgroundTransparency=cf.tr, BorderSizePixel=0, ClipsDescendants=true, Visible=false, ZIndex=10}, gui)
rc(main, 18)
ast(main, 0.5)
scale=mk("UIScale", {Scale=cf.size}, main)
local function fit()
    if scaletw then
        scaletw:Cancel()
        scaletw=nil
    end
    local cam=workspace.CurrentCamera
    if not cam then
        mscale=cf.size
        scale.Scale=mscale
        return
    end
    local v=cam.ViewportSize
    mscale=math.min(cf.size, math.max(0.4, (v.X - 24) / 700), math.max(0.4, (v.Y - 24) / 460))
    scale.Scale=mscale
end
local function watch()
    if camconn then
        camconn:Disconnect()
        camconn=nil
    end
    local cam=workspace.CurrentCamera
    if cam then
        camconn=cam:GetPropertyChangedSignal("ViewportSize"):Connect(fit)
    end
    fit()
end
side=mk("Frame", {Size=UDim2.new(0, 175, 1, 0), BackgroundColor3=col.panel, BorderSizePixel=0}, main)
rc(side, 18)
mk("Frame", {Size=UDim2.new(0, 18, 1, 0), Position=UDim2.new(1, -18, 0, 0), BackgroundColor3=col.panel, BorderSizePixel=0}, side)
local function fallback(img, size)
    local l=tx("TextLabel", {Size=UDim2.fromScale(1, 1), BackgroundTransparency=1, Text="CAT", TextColor3=col.text, TextSize=size, ZIndex=2, Visible=not img.IsLoaded}, img, true)
    table.insert(life, img:GetPropertyChangedSignal("IsLoaded"):Connect(function()
        l.Visible=not img.IsLoaded
    end))
end
local av=mk("ImageButton", {Size=UDim2.fromOffset(54, 54), Position=UDim2.fromOffset(18, 20), BackgroundColor3=col.card, Image="rbxassetid://" .. cf.avatar, ScaleType=Enum.ScaleType.Crop, AutoButtonColor=false}, side)
rc(av, 16)
fallback(av, 14)
tx("TextLabel", {Size=UDim2.new(1, -25, 0, 24), Position=UDim2.fromOffset(18, 82), BackgroundTransparency=1, Text="CAT SETTINGS", TextColor3=col.text, TextSize=15, TextXAlignment=Enum.TextXAlignment.Left}, side, true)
nav=mk("Frame", {Size=UDim2.new(1, -20, 1, -126), Position=UDim2.fromOffset(10, 116), BackgroundTransparency=1}, side)
mk("UIListLayout", {Padding=UDim.new(0, 4), SortOrder=Enum.SortOrder.LayoutOrder}, nav)
body=mk("Frame", {Size=UDim2.new(1, -195, 1, -28), Position=UDim2.fromOffset(185, 14), BackgroundTransparency=1}, main)
head=mk("Frame", {Size=UDim2.new(1, 0, 0, 48), BackgroundTransparency=1, Active=true}, body)
title=tx("TextLabel", {Size=UDim2.new(1, -100, 0, 32), Position=UDim2.fromOffset(4, 0), BackgroundTransparency=1, Text="Performance", TextColor3=col.text, TextSize=20, TextXAlignment=Enum.TextXAlignment.Left}, head, true)
minb=tx("TextButton", {Size=UDim2.fromOffset(32, 32), Position=UDim2.new(1, -76, 0, 0), BackgroundColor3=col.card, Text="−", TextColor3=col.text, TextSize=20, AutoButtonColor=false}, head, true)
rc(minb, 9)
closeb=tx("TextButton", {Size=UDim2.fromOffset(32, 32), Position=UDim2.new(1, -36, 0, 0), BackgroundColor3=col.card, Text="×", TextColor3=col.text, TextSize=20, AutoButtonColor=false}, head, true)
rc(closeb, 9)
holder=mk("ScrollingFrame", {Size=UDim2.new(1, 0, 1, -55), Position=UDim2.fromOffset(0, 55), BackgroundTransparency=1, BorderSizePixel=0, ScrollBarThickness=3, ScrollBarImageColor3=col.acc, ScrollingDirection=Enum.ScrollingDirection.Y, AutomaticCanvasSize=Enum.AutomaticSize.Y, CanvasSize=UDim2.new()}, body)
mk("UIListLayout", {Padding=UDim.new(0, 8), SortOrder=Enum.SortOrder.LayoutOrder}, holder)
mk("UIPadding", {PaddingBottom=UDim.new(0, 12)}, holder)
local sl={}
local spos={}
local function stat(k, y, init)
    local p=UDim2.new(1, -125, 0, y)
    local l=tx("TextLabel", {Size=UDim2.fromOffset(110, 28), Position=p, BackgroundColor3=col.card, BackgroundTransparency=0.1, Text=init, TextColor3=col.green, TextSize=12, Visible=false, ZIndex=20, Active=true}, gui, true)
    rc(l, 9)
    drag(l, l, false)
    sl[k]=l
    spos[k]=p
end
stat("fps", 12, "FPS: --")
stat("png", 46, "PING: --")
stat("mem", 80, "MEM: --")
toast=tx("TextLabel", {Size=UDim2.fromOffset(280, 40), AnchorPoint=Vector2.new(0.5, 0), Position=UDim2.new(0.5, 0, 0, -60), BackgroundColor3=col.card, BackgroundTransparency=0.05, TextColor3=col.text, Text="", TextSize=13, ZIndex=40, Visible=false}, gui, false)
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
        local ok, v=pcall(function()
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
            o.Parent=nil
        end
    end
end
local skyprops={"SkyboxBk", "SkyboxDn", "SkyboxFt", "SkyboxLf", "SkyboxRt", "SkyboxUp", "StarCount", "CelestialBodiesShown", "SunAngularSize", "MoonAngularSize", "SunTextureId", "MoonTextureId", "SkyboxOrientation"}
local atmprops={"Density", "Offset", "Color", "Decay", "Glare", "Haze"}
local function catsky(fresh)
    local e=lt:FindFirstChild("CatSky")
    if e and not fresh then
        return e
    end
    if e then
        e:Destroy()
    end
    local sk=Instance.new("Sky")
    sk.Name="CatSky"
    if not fresh then
        local src=lt:FindFirstChildOfClass("Sky")
        if src then
            for _, p in ipairs(skyprops) do
                pcall(function()
                    sk[p]=src[p]
                end)
            end
        end
    end
    hide("Sky", hsky, "CatSky")
    sk.Parent=lt
    return sk
end
local function catatm()
    local e=lt:FindFirstChild("CatAtmosphere")
    if e then
        return e
    end
    local a=Instance.new("Atmosphere")
    a.Name="CatAtmosphere"
    local src=lt:FindFirstChildOfClass("Atmosphere")
    if src then
        for _, p in ipairs(atmprops) do
            pcall(function()
                a[p]=src[p]
            end)
        end
    end
    hide("Atmosphere", hatm, "CatAtmosphere")
    a.Parent=lt
    return a
end
local function setatm(p, v)
    pcall(function()
        catatm()[p]=v
    end)
end
local function lockclock()
    if lockconn then
        lockconn:Disconnect()
        lockconn=nil
    end
    if s.lck and not dead then
        lockconn=lt:GetPropertyChangedSignal("ClockTime"):Connect(function()
            if s.lck and math.abs(lt.ClockTime - s.lckt) > 0.001 then
                lt.ClockTime=s.lckt
            end
        end)
    end
end
local function setclock(v)
    s.lckt=v
    lt.ClockTime=v
end
local function shadows()
    local b=olt.GlobalShadows
    if b == nil then
        b=true
    end
    pcall(function()
        lt.GlobalShadows=b and not s.shd
    end)
end
local function fog(on)
    s.fog=on
    if on then
        lt.FogStart=olt.FogStart or lt.FogStart
        lt.FogEnd=olt.FogEnd or lt.FogEnd
    else
        lt.FogStart=100000
        lt.FogEnd=100000
    end
end
local bskies={
    {
        Name="SKY 01 - CLASSIC",
        Ids={"15983968922", "15983966825", "15983965025", "15983967420", "15983966246", "15983964246"}
    },
    {
        Name="SKY 02 - NOVA",
        Ids={"11362875469", "11362888971", "11362882324", "11362894347", "11362878718", "11363654446"}
    },
    {
        Name="SKY 03 - HALO",
        Ids={"13107325341", "13107329809", "13107334845", "13107337703", "13107340396", "13107344387"}
    },
    {
        Name="SKY 04 - ECLIPSE",
        Ids={"5260808177", "5260653793", "5260817288", "5260800833", "5260811073", "5260824661"}
    }
}
local ed={
    bloom={
        c="BloomEffect",
        n="CatBloom",
        d={
            Intensity=0.35,
            Size=32,
            Threshold=1
        }
    },
    blur={
        c="BlurEffect",
        n="CatBlur",
        d={
            Size=8
        }
    },
    rays={
        c="SunRaysEffect",
        n="CatSunRays",
        d={
            Intensity=0.25,
            Spread=1
        }
    },
    dof={
        c="DepthOfFieldEffect",
        n="CatDoF",
        d={
            FarIntensity=0.3,
            FocusDistance=50,
            InFocusRadius=30,
            NearIntensity=0
        }
    }
}
local ev={}
for k, d in pairs(ed) do
    ev[k]=table.clone(d.d)
end
local cv={
    Brightness=0,
    Contrast=0,
    Saturation=0,
    TintColor=Color3.new(1, 1, 1)
}
local function seteffect(k, on)
    local d=ed[k]
    local e=lt:FindFirstChild(d.n)
    s[k]=on
    if on then
        if not e then
            e=Instance.new(d.c)
            e.Name=d.n
            e.Parent=lt
        end
        for p, v in pairs(ev[k]) do
            e[p]=v
        end
    elseif e then
        e:Destroy()
    end
end
local function setev(k, p, v)
    ev[k][p]=v
    local e=lt:FindFirstChild(ed[k].n)
    if e then
        e[p]=v
    end
end
local function setcc(p, v)
    cv[p]=v
    local e=lt:FindFirstChild("CatColorCorrection")
    if not e then
        e=Instance.new("ColorCorrectionEffect")
        e.Name="CatColorCorrection"
        e.Parent=lt
    end
    for n, x in pairs(cv) do
        e[n]=x
    end
end
local ofov
local function fovbind()
    pcall(function()
        rs:UnbindFromRenderStep("CatSettingsFOV")
    end)
    if s.cfov and not dead then
        rs:BindToRenderStep("CatSettingsFOV", Enum.RenderPriority.Camera.Value + 1, function()
            local cam=workspace.CurrentCamera
            if cam and cam.FieldOfView ~= s.fov then
                cam.FieldOfView=s.fov
            end
        end)
    end
end
local function setfov(on)
    local cam=workspace.CurrentCamera
    s.cfov=on
    if on and cam and not ofov then
        ofov=cam.FieldOfView
    end
    fovbind()
    if not on and cam and ofov then
        cam.FieldOfView=ofov
        ofov=nil
    end
end
local function run(fn, msg)
    runtok=runtok+1
    local id=runtok
    if dead then
        return
    end
    local function exec()
        fn()
        if msg then
            notify(msg)
        end
        task.delay(s.gl + 0.1, function()
            if not dead and id == runtok then
                refresh()
            end
        end)
    end
    if s.gl <= 0 then
        exec()
        return
    end
    tn(fade, {BackgroundTransparency=0.45}, 0.18)
    task.delay(0.2, function()
        if dead or id ~= runtok then
            return
        end
        exec()
        tn(fade, {BackgroundTransparency=1}, 0.5)
    end)
end
local faces={"SkyboxBk", "SkyboxDn", "SkyboxFt", "SkyboxLf", "SkyboxRt", "SkyboxUp"}
local function applycustom(e)
    run(function()
        local sk=catsky(true)
        for i, f in ipairs(faces) do
            sk[f]="rbxassetid://" .. (e.Ids[i] or e.Ids[1])
        end
    end, e.Name .. " applied")
    refresh()
end
local lroot="CatSkies"
local lcache="CatSkiesCache"
local fk={
    bk="SkyboxBk",
    dn="SkyboxDn",
    ft="SkyboxFt",
    lf="SkyboxLf",
    rt="SkyboxRt",
    up="SkyboxUp"
}
local lfs=cs and type(isfolder) == "function" and type(listfiles) == "function" and type(makefolder) == "function" and type(getcustomasset) == "function"
local lsk
local function scanlocal()
    lsk={}
    if not lfs then
        return
    end
    local function walk(dir, depth)
        local fl={}
        local n=0
        local subs={}
        for _, fp in ipairs(listfiles(dir)) do
            if isfolder(fp) then
                table.insert(subs, fp)
            else
                local k=fp:lower():match("[_%-%s](%a%a)%.%w+$")
                if k and fk[k] then
                    if not fl[k] then
                        n=n+1
                    end
                    fl[k]=fp
                end
            end
        end
        if n == 6 then
            table.insert(lsk, {n=dir:match("[^/\\]+$") or dir, f=fl})
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
        local urls={}
        local ok=pcall(function()
            local base=e.n:gsub("[^%w]", "_")
            if not isfolder(lcache) then
                makefolder(lcache)
            end
            for k, prop in pairs(fk) do
                local dst=lcache .. "/" .. base .. "_" .. k .. ".png"
                if not isfile(dst) then
                    writefile(dst, readfile(e.f[k]))
                end
                urls[prop]=getcustomasset(dst)
            end
        end)
        if not ok then
            notify("Local sky failed")
            return
        end
        run(function()
            local sk=catsky(true)
            for prop, u in pairs(urls) do
                sk[prop]=u
            end
        end, e.n .. " applied")
    end)
end
local function randomsky()
    if lfs and not lsk then
        scanlocal()
    end
    local pool={}
    for _, e in ipairs(sv.skies) do
        local e=e
        table.insert(pool, function()
            applycustom(e)
        end)
    end
    for _, e in ipairs(bskies) do
        local e=e
        table.insert(pool, function()
            applycustom(e)
        end)
    end
    for _, e in ipairs(lsk or {}) do
        local e=e
        table.insert(pool, function()
            applylocal(e)
        end)
    end
    pool[math.random(#pool)]()
end
local function pids(v)
    local ids={}
    for t in tostring(v):gmatch("[^,%s;]+") do
        local i=nid(t)
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
    s.spin=0
    s.flow=0
    s.cyc=0
    s.fb=false
    local a=lt:FindFirstChild("CatSky")
    if a then
        a:Destroy()
    end
    local b=lt:FindFirstChild("CatAtmosphere")
    if b then
        b:Destroy()
    end
    for _, o in ipairs(hsky) do
        pcall(function()
            o.Parent=lt
        end)
    end
    for _, o in ipairs(hatm) do
        pcall(function()
            o.Parent=lt
        end)
    end
    table.clear(hsky)
    table.clear(hatm)
    for p, v in pairs(olt) do
        pcall(function()
            lt[p]=v
        end)
    end
    s.fog=true
    s.lck=false
    s.lckt=lt.ClockTime
    lockclock()
    shadows()
    if not q then
        notify("Lighting restored")
    end
end
local function reffects(q)
    for k in pairs(ed) do
        seteffect(k, false)
        ev[k]=table.clone(ed[k].d)
    end
    local cc=lt:FindFirstChild("CatColorCorrection")
    if cc then
        cc:Destroy()
    end
    cv.Brightness=0
    cv.Contrast=0
    cv.Saturation=0
    cv.TintColor=Color3.new(1, 1, 1)
    setfov(false)
    if s.xreset then
        s.xreset()
    end
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
    local e=oobj[o]
    if e then
        return e
    end
    local d
    if k == "e" or k == "p" then
        d={
            k=k,
            en=o.Enabled
        }
    elseif k == "d" then
        d={
            k=k,
            t=o.Transparency
        }
    elseif k == "m" then
        d={
            k=k,
            m=o.Material,
            r=o.Reflectance
        }
    end
    oobj[o]=d
    return d
end
local function applyd(o, d)
    local k=d.k
    if k == "e" then
        o.Enabled=d.en and not s.ptc
    elseif k == "p" then
        o.Enabled=d.en and not s.pst
    elseif k == "d" then
        o.Transparency=s.tex and 1 or d.t
    elseif k == "m" then
        if s.mat then
            o.Material=Enum.Material.SmoothPlastic
            o.Reflectance=0
        else
            o.Material=d.m
            o.Reflectance=d.r
        end
    end
end
local function scan(k)
    stok[k]=(stok[k] or 0) + 1
    local tok=stok[k]
    task.spawn(function()
        local roots=k == "p" and {lt, workspace.CurrentCamera} or {workspace}
        local t=os.clock()
        for _, r in ipairs(roots) do
            for _, o in ipairs(r:GetDescendants()) do
                if dead or not scanned[k] or tok ~= stok[k] then
                    return
                end
                if kind(o) == k then
                    pcall(function()
                        local d=capture(o, k)
                        if d then
                            applyd(o, d)
                        end
                    end)
                end
                if os.clock() - t > 0.003 then
                    task.wait()
                    t=os.clock()
                end
            end
        end
    end)
end
local function setkind(k)
    if s[kf[k]] then
        if not scanned[k] then
            scanned[k]=true
            scan(k)
        end
    else
        scanned[k]=nil
        stok[k]=(stok[k] or 0) + 1
        for o, d in pairs(oobj) do
            if d.k == k then
                if o.Parent then
                    pcall(applyd, o, d)
                end
                oobj[o]=nil
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
            ter.WaterWaveSize=0
            ter.WaterWaveSpeed=0
            ter.WaterReflectance=0
        else
            for p, v in pairs(owt) do
                ter[p]=v
            end
        end
    end)
end
local function quality()
    pcall(function()
        local r=settings().Rendering
        if s.low then
            r.QualityLevel=Enum.QualityLevel.Level01
        elseif oq then
            r.QualityLevel=oq
        end
    end)
end
local function setlag(v)
    s.lag=v
    s.shd=v
    s.ptc=v
    s.pst=v
    s.mat=v
    s.wtr=v
    s.low=v
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
    s.lag=false
    s.shd=false
    s.ptc=false
    s.tex=false
    s.pst=false
    s.mat=false
    s.wtr=false
    s.low=false
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
    local sp=fs.spec
    if not sp or not fs.game then
        return
    end
    if gtx[o] == nil then
        local ok, f=pcall(function()
            return o.FontFace
        end)
        if not ok then
            return
        end
        gtx[o]=f
    end
    pcall(apply, o, sp)
end
local function sweep()
    sweeptok=sweeptok+1
    local tok=sweeptok
    task.spawn(function()
        local t=os.clock()
        for _, r in ipairs(pg and {pg, workspace} or {workspace}) do
            for _, o in ipairs(r:GetDescendants()) do
                if tok ~= sweeptok or dead then
                    return
                end
                if isgt(o) then
                    stylegt(o)
                end
                if os.clock() - t > 0.003 then
                    task.wait()
                    t=os.clock()
                end
            end
        end
    end)
end
local function rgt()
    sweeptok=sweeptok+1
    for o, f in pairs(gtx) do
        if o.Parent then
            pcall(function()
                o.FontFace=f
            end)
        end
    end
    table.clear(gtx)
end
local function same(o, sp)
    if sp.e then
        return o.Font == sp.e
    end
    local f=o.FontFace
    return f.Family == sp.f.Family and f.Weight == sp.f.Weight and f.Style == sp.f.Style
end
local function fbrefresh()
    for k, b in pairs(fb) do
        if b.Parent then
            local on=fs.key == k
            b.Text=on and "Active" or "Apply"
            b.BackgroundColor3=on and col.green or col.acc
        else
            fb[k]=nil
        end
    end
end
local function setfont(sp, q)
    fs.spec=sp
    fs.key=sp and sp.k or nil
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
    fs.game=on
    if on and fs.spec then
        sweep()
    else
        rgt()
    end
    later()
end
local function setmenu(on)
    fs.menu=on
    menufont()
    later()
end
local function theme(q)
    col.acc=cf.theme
    main.BackgroundTransparency=cf.tr
    holder.ScrollBarImageColor3=col.acc
    ind.BackgroundColor3=col.acc
    for i=#astk, 1, -1 do
        local x=astk[i]
        if x.Parent then
            x.Color=col.acc
        else
            table.remove(astk, i)
        end
    end
    fit()
    if not q then
        later()
        notify("Theme updated")
    end
end
local function clear()
    order=0
    cseq=0
    for _, o in ipairs(holder:GetChildren()) do
        if o:IsA("GuiObject") then
            o:Destroy()
        end
    end
end
local function nxt()
    order=order+1
    return order
end
local function card(h)
    local f=mk("Frame", {Size=UDim2.new(1, -5, 0, h), BackgroundColor3=col.card, BackgroundTransparency=1, BorderSizePixel=0, LayoutOrder=nxt()}, holder)
    rc(f, 12)
    st(f, Color3.new(1, 1, 1), 0.94)
    if s.still then
        f.BackgroundTransparency=0.08
        return f
    end
    cseq=cseq+1
    local id=cseq
    local pop=mk("UIScale", {Scale=0.94}, f)
    task.delay(0.015 * math.min(id, 18), function()
        if f.Parent then
            tn(f, {BackgroundTransparency=0.08}, 0.22)
            tw:Create(pop, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale=1}):Play()
        end
    end)
    return f
end
local function section(n)
    local l=tx("TextLabel", {Size=UDim2.new(1, -5, 0, 24), BackgroundTransparency=1, Text=n, TextColor3=col.acc, TextSize=12, TextXAlignment=Enum.TextXAlignment.Left, TextTransparency=1, LayoutOrder=nxt()}, holder, true)
    pd(l, 4)
    if s.still then
        l.TextTransparency=0
        return l
    end
    cseq=cseq+1
    local id=cseq
    task.delay(0.015 * math.min(id, 18), function()
        if l.Parent then
            tn(l, {TextTransparency=0}, 0.2)
        end
    end)
    return l
end
local function hover(b, list)
    list=list or conns
    local sc=mk("UIScale", {Scale=1}, b)
    table.insert(list, b.MouseEnter:Connect(function()
        tn(b, {BackgroundTransparency=0.2}, 0.12)
    end))
    table.insert(list, b.MouseLeave:Connect(function()
        tn(b, {BackgroundTransparency=0}, 0.12)
        tw:Create(sc, TweenInfo.new(0.16, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale=1}):Play()
    end))
    table.insert(list, b.MouseButton1Down:Connect(function()
        tw:Create(sc, TweenInfo.new(0.07, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Scale=0.92}):Play()
    end))
    table.insert(list, b.MouseButton1Up:Connect(function()
        tw:Create(sc, TweenInfo.new(0.16, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale=1}):Play()
    end))
end
local function row(n, btns, pf)
    local cd=card(48)
    local off=12
    local made={}
    for i=#btns, 1, -1 do
        local b=btns[i]
        local w=b.w or 78
        local bt=tx("TextButton", {Size=UDim2.fromOffset(w, 30), Position=UDim2.new(1, -(off + w), 0.5, -15), BackgroundColor3=b.c or col.acc, Text=b.t, TextColor3=col.text, TextSize=12, AutoButtonColor=false}, cd, true)
        rc(bt, 9)
        hover(bt)
        table.insert(conns, bt.MouseButton1Click:Connect(b.f))
        made[i]=bt
        off=off+w + 8
    end
    local p={
        Size=UDim2.new(1, -(off + 14), 1, 0),
        Position=UDim2.fromOffset(14, 0),
        BackgroundTransparency=1,
        Text=n,
        TextColor3=col.text,
        TextSize=13,
        TextXAlignment=Enum.TextXAlignment.Left,
        TextTruncate=Enum.TextTruncate.AtEnd
    }
    if pf then
        p.Font=pf
        mk("TextLabel", p, cd)
    else
        tx("TextLabel", p, cd)
    end
    return cd, made
end
local function btn(n, t, f, c)
    local _, m=row(n, {{t=t or "Apply", f=f, c=c}})
    return m[1]
end
local function toggle(n, init, cb)
    local cd=card(48)
    local on=init == true
    tx("TextLabel", {Size=UDim2.new(1, -90, 1, 0), Position=UDim2.fromOffset(14, 0), BackgroundTransparency=1, Text=n, TextColor3=col.text, TextSize=13, TextXAlignment=Enum.TextXAlignment.Left}, cd)
    local tg=mk("TextButton", {Size=UDim2.fromOffset(46, 25), Position=UDim2.new(1, -60, 0.5, -12), BackgroundColor3=on and col.acc or col.off, Text="", AutoButtonColor=false}, cd)
    rc(tg, 20)
    local kn=mk("Frame", {Size=UDim2.fromOffset(19, 19), Position=on and UDim2.new(1, -22, 0, 3) or UDim2.fromOffset(3, 3), BackgroundColor3=Color3.new(1, 1, 1), BorderSizePixel=0}, tg)
    rc(kn, 20)
    table.insert(conns, tg.MouseButton1Click:Connect(function()
        on=not on
        tn(tg, {BackgroundColor3=on and col.acc or col.off})
        tw:Create(kn, TweenInfo.new(0.28, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position=on and UDim2.new(1, -22, 0, 3) or UDim2.fromOffset(3, 3)}):Play()
        cb(on)
    end))
    return function(v)
        on=v == true
        tn(tg, {BackgroundColor3=on and col.acc or col.off})
        tw:Create(kn, TweenInfo.new(0.28, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position=on and UDim2.new(1, -22, 0, 3) or UDim2.fromOffset(3, 3)}):Play()
    end
end
local function fmt(v)
    return tostring(math.floor(v * 1000 + 0.5) / 1000)
end
local function slider(n, mn, mx, init, cb, step)
    step=step or 0.1
    local cd=card(64)
    local v=math.clamp(init, mn, mx)
    local rg=mx - mn
    tx("TextLabel", {Size=UDim2.new(1, -90, 0, 22), Position=UDim2.fromOffset(14, 6), BackgroundTransparency=1, Text=n, TextColor3=col.text, TextSize=13, TextXAlignment=Enum.TextXAlignment.Left}, cd)
    local vl=tx("TextLabel", {Size=UDim2.fromOffset(70, 22), Position=UDim2.new(1, -84, 0, 6), BackgroundTransparency=1, Text=fmt(v), TextColor3=col.acc, TextSize=12, TextXAlignment=Enum.TextXAlignment.Right}, cd)
    local hit=mk("Frame", {Size=UDim2.new(1, -28, 0, 26), Position=UDim2.fromOffset(14, 32), BackgroundTransparency=1, Active=true}, cd)
    local bar=mk("Frame", {Size=UDim2.new(1, 0, 0, 6), Position=UDim2.new(0, 0, 0.5, -3), BackgroundColor3=Color3.fromRGB(60, 64, 80), BorderSizePixel=0}, hit)
    rc(bar, 10)
    local pc=rg == 0 and 0 or (v - mn) / rg
    local fl=mk("Frame", {Size=UDim2.new(pc, 0, 1, 0), BackgroundColor3=col.acc, BorderSizePixel=0}, bar)
    rc(fl, 10)
    local kn=mk("Frame", {Size=UDim2.fromOffset(15, 15), AnchorPoint=Vector2.new(0.5, 0.5), Position=UDim2.new(pc, 0, 0.5, 0), BackgroundColor3=Color3.new(1, 1, 1), BorderSizePixel=0}, bar)
    rc(kn, 20)
    local on=false
    local di=nil
    local function set(x)
        local r=math.clamp((x - hit.AbsolutePosition.X) / math.max(hit.AbsoluteSize.X, 1), 0, 1)
        local q=math.clamp(math.floor((mn + rg * r) / step + 0.5) * step, mn, mx)
        q=math.floor(q * 1000 + 0.5) / 1000
        local p=rg == 0 and 0 or (q - mn) / rg
        fl.Size=UDim2.new(p, 0, 1, 0)
        kn.Position=UDim2.new(p, 0, 0.5, 0)
        vl.Text=fmt(q)
        if q ~= v then
            v=q
            cb(v)
        end
    end
    table.insert(conns, hit.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on=true
            di=i
            set(i.Position.X)
            tn(kn, {Size=UDim2.fromOffset(20, 20)}, 0.12)
        end
    end))
    table.insert(conns, uis.InputChanged:Connect(function(i)
        if on and (i.UserInputType == Enum.UserInputType.MouseMovement or i == di) then
            set(i.Position.X)
        end
    end))
    table.insert(conns, uis.InputEnded:Connect(function(i)
        if i == di or i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            on=false
            di=nil
            tn(kn, {Size=UDim2.fromOffset(15, 15)}, 0.15)
        end
    end))
end
local function form(phs, bt, cb)
    local cnt=#phs
    local cd=card(10 + cnt * 40)
    local bx={}
    for i, ph in ipairs(phs) do
        local b=tx("TextBox", {Size=UDim2.new(1, i == cnt and -112 or -24, 0, 30), Position=UDim2.fromOffset(12, 10 + (i - 1) * 40), BackgroundColor3=col.panel, PlaceholderText=ph, PlaceholderColor3=col.sub, Text="", TextColor3=col.text, TextSize=12, ClearTextOnFocus=false, TextXAlignment=Enum.TextXAlignment.Left}, cd)
        rc(b, 9)
        pd(b, 10)
        bx[i]=b
    end
    local function submit()
        local vals={}
        for i, b in ipairs(bx) do
            vals[i]=b.Text
        end
        cb(vals)
    end
    local sb=tx("TextButton", {Size=UDim2.fromOffset(80, 30), Position=UDim2.new(1, -92, 0, 10 + (cnt - 1) * 40), BackgroundColor3=col.acc, Text=bt, TextColor3=col.text, TextSize=12, AutoButtonColor=false}, cd, true)
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
    local cd=card(48)
    local b=tx("TextBox", {Size=UDim2.new(1, -24, 0, 30), Position=UDim2.fromOffset(12, 9), BackgroundColor3=col.panel, PlaceholderText=ph, PlaceholderColor3=col.sub, Text="", TextColor3=col.text, TextSize=12, ClearTextOnFocus=false, TextXAlignment=Enum.TextXAlignment.Left}, cd)
    rc(b, 9)
    pd(b, 10)
    local tok=0
    table.insert(conns, b:GetPropertyChangedSignal("Text"):Connect(function()
        tok=tok+1
        local id=tok
        task.delay(0.05, function()
            if id == tok and not dead then
                cb(b.Text)
            end
        end)
    end))
end
local function colorin(n, init, cb)
    local cd=card(48)
    tx("TextLabel", {Size=UDim2.new(1, -212, 1, 0), Position=UDim2.fromOffset(14, 0), BackgroundTransparency=1, Text=n, TextColor3=col.text, TextSize=13, TextXAlignment=Enum.TextXAlignment.Left, TextTruncate=Enum.TextTruncate.AtEnd}, cd)
    local pv=mk("Frame", {Size=UDim2.fromOffset(22, 22), Position=UDim2.new(1, -190, 0.5, -11), BackgroundColor3=init, BorderSizePixel=0}, cd)
    rc(pv, 6)
    st(pv, Color3.new(1, 1, 1), 0.7)
    local bx=tx("TextBox", {Size=UDim2.fromOffset(90, 30), Position=UDim2.new(1, -160, 0.5, -15), BackgroundColor3=col.panel, Text=tohex(init), TextColor3=col.text, TextSize=12, ClearTextOnFocus=false}, cd)
    rc(bx, 9)
    local sb=tx("TextButton", {Size=UDim2.fromOffset(58, 30), Position=UDim2.new(1, -64, 0.5, -15), BackgroundColor3=col.acc, Text="Set", TextColor3=col.text, TextSize=12, AutoButtonColor=false}, cd, true)
    rc(sb, 9)
    hover(sb)
    local function submit()
        local cr=hex(bx.Text)
        if not cr then
            notify("Invalid color")
            return
        end
        pv.BackgroundColor3=cr
        bx.Text=tohex(cr)
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
    fs.spec=nil
    fs.key=nil
    fs.game=true
    fs.menu=true
    menufont()
    cf.theme=dtheme
    cf.size=1
    cf.tr=0.12
    cf.key=Enum.KeyCode.RightShift
    cf.notif=true
    cf.btn=true
    if s.slreset then
        s.slreset()
    end
    cf.rgb=false
    if s.setrgb then
        s.setrgb(false)
    end
    s.fps=false
    s.png=false
    s.mem=false
    for _, l in pairs(sl) do
        l.Visible=false
    end
    obtn.Visible=true
    theme(true)
    resetpos()
    refresh()
    later()
    notify("Defaults restored")
end
pages.performance=function()
    section("PERFORMANCE")
    local lagset=toggle("Fix Lag", s.lag, setlag)
    local function sync()
        s.lag=s.shd and s.ptc and s.pst and s.mat and s.wtr and s.low
        lagset(s.lag)
    end
    for _, d in ipairs({{"Show FPS", "fps"}, {"Show Ping", "png"}, {"Show Memory", "mem"}}) do
        local d=d
        toggle(d[1], s[d[2]], function(v)
            s[d[2]]=v
            sl[d[2]].Visible=v
        end)
    end
    section("GRAPHICS")
    toggle("Remove Shadows", s.shd, function(v)
        s.shd=v
        shadows()
        sync()
    end)
    for _, d in ipairs({{"Remove Particles", "ptc", "e"}, {"Remove Textures", "tex", "d"}, {"Remove Post Effects", "pst", "p"}, {"Smooth Materials", "mat", "m"}}) do
        local d=d
        toggle(d[1], s[d[2]], function(v)
            s[d[2]]=v
            setkind(d[3])
            sync()
        end)
    end
    toggle("Simple Water", s.wtr, function(v)
        s.wtr=v
        water()
        sync()
    end)
    toggle("Low Quality Level", s.low, function(v)
        s.low=v
        quality()
        sync()
    end)
    btn("Restore Graphics", "Restore", function()
        rgraphics()
        refresh()
    end)
end
pages.sky=function()
    local function modelsky(id)
        local ok, res=pcall(function()
            return game:GetObjects("rbxassetid://" .. id)
        end)
        if not ok or type(res) ~= "table" then
            return nil
        end
        for _, o in ipairs(res) do
            local sk=o:IsA("Sky") and o or o:FindFirstChildWhichIsA("Sky", true)
            if sk then
                local ids={}
                for i, f in ipairs(faces) do
                    local n=nid(sk[f])
                    if not n then
                        return nil
                    end
                    ids[i]=n
                end
                return ids
            end
        end
        return nil
    end
    section("SKY PACK")
    btn("Random Sky", "Random", randomsky)
    for _, e in ipairs(bskies) do
        local e=e
        row(e.Name, {
            {t="Apply", f=function()
                applycustom(e)
            end}
        })
    end
    section("SKY FROM LINK")
    form({"Name", "Sky link or ID (1 or 6, separated by comma)"}, "Add", function(v)
        local ids=pids(v[2])
        if not ids then
            notify("Need a valid link or ID")
            return
        end
        local n=trim(v[1])
        task.spawn(function()
            if #ids == 1 then
                notify("Loading sky...")
                ids=modelsky(ids[1]) or ids
            end
            if n == "" then
                n="Sky " .. (#sv.skies + 1)
            end
            local e={
                Name=n,
                Ids=ids
            }
            table.insert(sv.skies, e)
            later()
            applycustom(e)
        end)
    end)
    for _, e in ipairs(sv.skies) do
        local e=e
        row(e.Name, {
            {t="Apply", f=function()
                applycustom(e)
            end},
            {t="X", w=34, c=col.red, f=function()
                for k, x in ipairs(sv.skies) do
                    if x == e then
                        table.remove(sv.skies, k)
                        break
                    end
                end
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
    local sk=cursky()
    slider("Star Count", 0, 5000, rp(sk, "StarCount", 3000), function(v)
        catsky().StarCount=v
    end, 100)
    slider("Sun Size", 0, 60, rp(sk, "SunAngularSize", 21), function(v)
        catsky().SunAngularSize=v
    end, 1)
    slider("Moon Size", 0, 60, rp(sk, "MoonAngularSize", 11), function(v)
        catsky().MoonAngularSize=v
    end, 1)
    toggle("Sun and Moon", rp(sk, "CelestialBodiesShown", true), function(v)
        catsky().CelestialBodiesShown=v
    end)
    section("ANIMATION")
    slider("Transition Time", 0, 5, s.gl, function(v)
        s.gl=v
    end, 0.1)
    slider("Sky Rotation", -20, 20, s.spin, function(v)
        s.spin=v
        if v ~= 0 then
            catsky()
        end
    end, 0.5)
    slider("Time Flow", -1, 1, s.flow, function(v)
        s.flow=v
    end, 0.05)
    slider("Color Cycle", 0, 1, s.cyc, function(v)
        s.cyc=v
    end, 0.05)
    btn("Default Sky", "Restore", function()
        rlight()
        refresh()
    end)
end
pages.lighting=function()
    section("TIME")
    slider("Time of Day", 0, 24, lt.ClockTime, setclock, 0.1)
    toggle("Lock Time", s.lck, function(v)
        s.lck=v
        s.lckt=lt.ClockTime
        lockclock()
    end)
    section("LIGHT")
    slider("Brightness", 0, 6, lt.Brightness, function(v)
        lt.Brightness=v
    end, 0.1)
    slider("Exposure", -3, 3, lt.ExposureCompensation, function(v)
        lt.ExposureCompensation=v
    end, 0.1)
    for _, d in ipairs({{"Diffuse Scale", "EnvironmentDiffuseScale"}, {"Specular Scale", "EnvironmentSpecularScale"}}) do
        local d=d
        slider(d[1], 0, 1, rp(lt, d[2], 1), function(v)
            pcall(function()
                lt[d[2]]=v
            end)
        end, 0.05)
    end
    toggle("Fog", s.fog, fog)
    toggle("Fullbright", s.fb, function(v)
        if s.setfb then
            s.setfb(v)
        end
    end)
    section("COLORS")
    for _, d in ipairs({{"Ambient", "Ambient"}, {"Outdoor Ambient", "OutdoorAmbient"}, {"Fog Color", "FogColor"}, {"Shift Top", "ColorShift_Top"}, {"Shift Bottom", "ColorShift_Bottom"}}) do
        local d=d
        colorin(d[1], lt[d[2]], function(cr)
            lt[d[2]]=cr
        end)
    end
    section("ATMOSPHERE")
    local a=curatm()
    for _, d in ipairs({{"Density", 0, 1, 0.395, 0.01}, {"Offset", 0, 1, 0, 0.01}, {"Haze", 0, 10, 0, 0.1}, {"Glare", 0, 10, 0, 0.1}}) do
        local d=d
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
pages.effects=function()
    local cgp={
        {"Vivid", 0, 0.1, 0.5, Color3.fromRGB(255, 255, 255)},
        {"Cinematic", -0.03, 0.25, -0.1, Color3.fromRGB(255, 235, 215)},
        {"Dreamy", 0.05, -0.1, 0.3, Color3.fromRGB(255, 220, 245)},
        {"Cyber", 0, 0.2, 0.4, Color3.fromRGB(200, 220, 255)},
        {"Noir", 0, 0.3, -1, Color3.fromRGB(255, 255, 255)}
    }
    local function fx(sec, k, lbl, list)
        section(sec)
        toggle(lbl, s[k], function(v)
            seteffect(k, v)
        end)
        for _, d in ipairs(list) do
            local d=d
            slider(d[1], d[2], d[3], ev[k][d[4]], function(v)
                setev(k, d[4], v)
            end, d[5])
        end
    end
    fx("BLOOM", "bloom", "Bloom", {{"Bloom Intensity", 0, 3, "Intensity", 0.05}, {"Bloom Size", 0, 56, "Size", 1}, {"Bloom Threshold", 0, 4, "Threshold", 0.05}})
    fx("BLUR", "blur", "Blur", {{"Blur Size", 0, 56, "Size", 1}})
    fx("SUN RAYS", "rays", "Sun Rays", {{"Rays Intensity", 0, 1, "Intensity", 0.01}, {"Rays Spread", 0, 1, "Spread", 0.01}})
    fx("DEPTH OF FIELD", "dof", "Depth of Field", {{"Far Intensity", 0, 1, "FarIntensity", 0.01}, {"Focus Distance", 0, 200, "FocusDistance", 1}, {"Focus Radius", 0, 100, "InFocusRadius", 1}})
    section("COLOR")
    for _, d in ipairs({{"Color Brightness", -0.5, 0.5, "Brightness", 0.01}, {"Contrast", -1, 1, "Contrast", 0.05}, {"Saturation", -1, 1, "Saturation", 0.05}}) do
        local d=d
        slider(d[1], d[2], d[3], cv[d[4]], function(v)
            setcc(d[4], v)
        end, d[5])
    end
    colorin("Tint", cv.TintColor, function(cr)
        setcc("TintColor", cr)
    end)
    section("COLOR PRESETS")
    for _, d in ipairs(cgp) do
        local d=d
        btn(d[1], "Apply", function()
            setcc("Brightness", d[2])
            setcc("Contrast", d[3])
            setcc("Saturation", d[4])
            setcc("TintColor", d[5])
            notify(d[1] .. " applied")
            refresh()
        end)
    end
    if s.fxpage then
        s.fxpage()
    end
    section("CAMERA")
    toggle("Custom FOV", s.cfov, setfov)
    slider("Field of View", 30, 120, s.fov, function(v)
        s.fov=v
    end, 1)
    btn("Reset Effects", "Reset", function()
        reffects()
        refresh()
    end)
end
pages.fonts=function()
    table.clear(fb)
    section("TARGET")
    toggle("Game Text", fs.game, setgame)
    toggle("Menu Text", fs.menu, setmenu)
    section("CUSTOM FONT")
    form({"Font name or Font ID"}, "Add", function(v)
        local val=trim(v[1])
        local sp=pfont(val, true)
        if not sp then
            notify("Font not found")
            return
        end
        if sp.f then
            local ex=false
            for _, e in ipairs(sv.fonts) do
                if e.Value == val then
                    ex=true
                    break
                end
            end
            if not ex then
                table.insert(sv.fonts, {Name=val, Value=val})
            end
        end
        setfont(sp)
        refresh()
    end)
    for _, e in ipairs(sv.fonts) do
        local e=e
        local sp=pfont(e.Value)
        if sp then
            local _, m=row(e.Name, {
                {t="Apply", f=function()
                    setfont(sp)
                end},
                {t="X", w=34, c=col.red, f=function()
                    for k, x in ipairs(sv.fonts) do
                        if x == e then
                            table.remove(sv.fonts, k)
                            break
                        end
                    end
                    later()
                    refresh()
                end}
            })
            fb[sp.k]=m[1]
        end
    end
    section("FONTS")
    local rws={}
    textin("Search font", function(q)
        q=q:lower()
        for _, r in ipairs(rws) do
            r.c.Visible=q == "" or r.n:find(q, 1, true) ~= nil
        end
    end)
    btn("Default", "Reset", function()
        setfont(nil)
    end, col.red)
    for _, it in ipairs(fitems) do
        local sp={
            e=it,
            k="Enum:" .. it.Name
        }
        local cd, m=row(it.Name, {{t="Apply", f=function()
            setfont(sp)
        end}}, it)
        fb[sp.k]=m[1]
        table.insert(rws, {c=cd, n=it.Name:lower()})
    end
    fbrefresh()
end
pages.interface=function()
    local themes={
        {"Purple", Color3.fromRGB(120, 105, 255)},
        {"Blue", Color3.fromRGB(70, 150, 255)},
        {"Cyan", Color3.fromRGB(55, 210, 220)},
        {"Pink", Color3.fromRGB(255, 105, 175)},
        {"Red", Color3.fromRGB(245, 85, 100)},
        {"Green", Color3.fromRGB(70, 200, 140)},
        {"Orange", Color3.fromRGB(255, 150, 75)},
        {"White", Color3.fromRGB(230, 230, 245)}
    }
    section("WINDOW")
    slider("Menu Size", 0.7, 1.2, cf.size, function(v)
        cf.size=v
        fit()
        later()
    end, 0.05)
    slider("Transparency", 0, 0.5, cf.tr, function(v)
        cf.tr=v
        main.BackgroundTransparency=v
        later()
    end, 0.01)
    toggle("Notifications", cf.notif, function(v)
        cf.notif=v
        later()
    end)
    toggle("Show Button", cf.btn, function(v)
        cf.btn=v
        obtn.Visible=v
        later()
        if not v then
            notify("Press " .. cf.key.Name .. " to open")
        end
    end)
    toggle("RGB Border", cf.rgb, function(v)
        cf.rgb=v
        if s.setrgb then
            s.setrgb(v)
        end
        later()
    end)
    form({"Toggle key (" .. cf.key.Name .. ")"}, "Set", function(v)
        local ok, k=pcall(function()
            return Enum.KeyCode[trim(v[1])]
        end)
        if not ok or not k then
            notify("Invalid key")
            return
        end
        cf.key=k
        later()
        notify("Key set to " .. k.Name)
        refresh()
    end)
    btn("Reset Position", "Reset", function()
        resetpos()
    end)
    section("THEME")
    for _, t in ipairs(themes) do
        local t=t
        btn(t[1], "Apply", function()
            cf.theme=t[2]
            theme()
            refresh()
        end, t[2])
    end
    colorin("Custom Color", cf.theme, function(cr)
        cf.theme=cr
        theme()
        refresh()
    end)
end
pages.general=function()
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
openpage=function(name, keep)
    local build=pages[name:lower()]
    if not build or dead then
        return
    end
    local pos=holder.CanvasPosition
    cur=name
    if not mini then
        title.Text=name
    end
    for _, x in ipairs(conns) do
        x:Disconnect()
    end
    table.clear(conns)
    clear()
    build()
    for i, n in ipairs(pnames) do
        local on=n == name
        tn(navs[n], {BackgroundColor3=on and col.acc or col.panel})
        if on then
            tn(ind, {Position=UDim2.fromOffset(4, 120 + (i - 1) * 32)}, 0.25)
        end
    end
    holder.Position=keep and UDim2.fromOffset(0, 55) or UDim2.fromOffset(18, 55)
    tn(holder, {Position=UDim2.fromOffset(0, 55)}, 0.25)
    if keep then
        task.spawn(function()
            rs.Heartbeat:Wait()
            rs.Heartbeat:Wait()
            if not dead and holder.Parent then
                holder.CanvasPosition=pos
            end
        end)
    else
        holder.CanvasPosition=Vector2.new(0, 0)
    end
end
refresh=function()
    if cur and not mini and main.Visible then
        s.still=true
        openpage(cur, true)
        s.still=false
    end
end
hover(minb, life)
hover(closeb, life)
ind=mk("Frame", {Size=UDim2.fromOffset(4, 20), Position=UDim2.fromOffset(4, 120), BackgroundColor3=col.acc, BorderSizePixel=0}, side)
rc(ind, 2)
for i, n in ipairs(pnames) do
    local n=n
    local b=tx("TextButton", {Name=n, Size=UDim2.new(1, 0, 0, 28), BackgroundColor3=col.panel, Text=n, TextColor3=col.text, TextSize=12, TextXAlignment=Enum.TextXAlignment.Left, AutoButtonColor=false, LayoutOrder=i}, nav)
    rc(b, 9)
    pd(b, 12)
    hover(b, life)
    b.MouseButton1Click:Connect(function()
        openpage(n)
    end)
    navs[n]=b
end
obtn=mk("ImageButton", {Name="CatButton", Size=UDim2.fromOffset(58, 58), Position=UDim2.new(0, 25, 0.5, -29), BackgroundColor3=col.panel, Image="rbxassetid://" .. cf.avatar, ScaleType=Enum.ScaleType.Crop, AutoButtonColor=false, Visible=cf.btn, ZIndex=25}, gui)
rc(obtn, 18)
ast(obtn, 0.45)
fallback(obtn, 13)
local obs=mk("UIScale", {Scale=1}, obtn)
table.insert(life, obtn.MouseButton1Down:Connect(function()
    tw:Create(obs, TweenInfo.new(0.08, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Scale=0.88}):Play()
    tn(obtn, {Rotation=8}, 0.08)
end))
table.insert(life, obtn.MouseButton1Up:Connect(function()
    tw:Create(obs, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale=1}):Play()
    tn(obtn, {Rotation=4}, 0.2)
end))
table.insert(life, obtn.MouseEnter:Connect(function()
    tn(obtn, {Rotation=4}, 0.15)
end))
table.insert(life, obtn.MouseLeave:Connect(function()
    tn(obtn, {Rotation=0}, 0.15)
    tw:Create(obs, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale=1}):Play()
end))
local moved=drag(obtn, obtn, false)
drag(main, head, true)
resetpos=function()
    main.Position=UDim2.fromScale(0.5, 0.5)
    obtn.Position=UDim2.new(0, 25, 0.5, -29)
    for k, l in pairs(sl) do
        l.Position=spos[k]
    end
    notify("Position reset")
end
local function layout(m)
    mini=m
    side.Visible=not m
    holder.Visible=not m
    tn(body, {Position=m and UDim2.fromOffset(12, 10) or UDim2.fromOffset(185, 14), Size=m and UDim2.new(1, -24, 0, 40) or UDim2.new(1, -195, 1, -28)}, 0.22)
    minb.Text=m and "+" or "−"
end
local function setmin(m)
    layout(m)
    if sztween then
        sztween:Cancel()
        sztween=nil
    end
    if m then
        title.Text="CAT SETTINGS"
        sztween=tn(main, {Size=UDim2.fromOffset(250, 60)}, 0.22)
    else
        sztween=tn(main, {Size=UDim2.fromOffset(700, 460)}, 0.25)
        openpage(cur or "Performance", true)
    end
end
local function openmenu()
    if dead then
        return
    end
    anim=anim+1
    if sztween then
        sztween:Cancel()
        sztween=nil
    end
    layout(false)
    main.Visible=true
    fit()
    main.Size=UDim2.fromOffset(700, 460)
    scale.Scale=mscale * 0.82
    main.BackgroundTransparency=1
    tn(main, {BackgroundTransparency=cf.tr}, 0.2)
    scaletw=tw:Create(scale, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale=mscale})
    scaletw:Play()
    openpage(cur or "Performance", true)
    if s.navin then
        s.navin()
    end
end
local function closemenu()
    if dead then
        return
    end
    anim=anim+1
    local id=anim
    if sztween then
        sztween:Cancel()
        sztween=nil
    end
    tn(main, {BackgroundTransparency=math.min(1, cf.tr + 0.3)}, 0.16)
    scaletw=tw:Create(scale, TweenInfo.new(0.18, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Scale=mscale * 0.86})
    scaletw:Play()
    task.delay(0.2, function()
        if anim == id and main.Parent then
            main.Visible=false
            main.BackgroundTransparency=cf.tr
            scale.Scale=mscale
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
shutdown=function()
    if dead then
        return
    end
    save()
    dead=true
    anim=anim+1
    sweeptok=sweeptok+1
    if lockconn then
        lockconn:Disconnect()
        lockconn=nil
    end
    if camconn then
        camconn:Disconnect()
        camconn=nil
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
    if s.kill then
        s.kill()
    end
    if env.CatSettingsUnload == shutdown then
        env.CatSettingsUnload=nil
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
local ff=0
local ft=0
local t2=0
local t3=0
table.insert(life, rs.RenderStepped:Connect(function(dt)
    if dead then
        return
    end
    ff=ff+1
    ft=ft+dt
    if s.spin ~= 0 then
        local sk=lt:FindFirstChild("CatSky")
        if sk then
            local o=sk.SkyboxOrientation
            sk.SkyboxOrientation=Vector3.new(o.X, (o.Y + s.spin * dt) % 360, o.Z)
        end
    end
    if ft >= 0.5 then
        if s.fps and sl.fps.Visible then
            local n=math.floor(ff / ft + 0.5)
            sl.fps.Text="FPS: " .. n
            sl.fps.TextColor3=n >= 50 and col.green or (n >= 30 and col.yellow or col.red)
        end
        ff=0
        ft=0
    end
end))
table.insert(life, rs.Heartbeat:Connect(function(dt)
    if dead then
        return
    end
    t2=t2+dt
    t3=t3+dt
    if s.flow ~= 0 and not s.lck then
        lt.ClockTime=(lt.ClockTime + s.flow * dt) % 24
    end
    if s.cyc ~= 0 then
        s.hue=(s.hue + s.cyc * 0.1 * dt) % 1
        local at=lt:FindFirstChild("CatAtmosphere") or catatm()
        at.Color=Color3.fromHSV(s.hue, 0.55, 1)
        lt.OutdoorAmbient=Color3.fromHSV(s.hue, 0.45, 0.85)
        lt.FogColor=Color3.fromHSV(s.hue, 0.5, 1)
    end
    if t2 >= 1 then
        t2=0
        if s.png and sl.png.Visible then
            local ms
            local ok, p=pcall(function()
                return lp:GetNetworkPing() * 1000
            end)
            if ok and p then
                ms=p
            else
                local ok2, v=pcall(function()
                    return stats.Network.ServerStatsItem["Data Ping"]:GetValue()
                end)
                if ok2 and v then
                    ms=v
                end
            end
            if ms then
                sl.png.Text="PING: " .. math.floor(ms) .. " ms"
            end
        end
        if s.mem and sl.mem.Visible then
            local ok, v=pcall(function()
                return stats:GetTotalMemoryUsageMb()
            end)
            if ok and v then
                sl.mem.Text="MEM: " .. math.floor(v) .. " MB"
            end
        end
    end
    if t3 >= 0.5 then
        t3=0
        local sp=fs.spec
        if sp and fs.game then
            for o in pairs(gtx) do
                if o.Parent then
                    local ok, m=pcall(same, o, sp)
                    if ok and not m then
                        pcall(apply, o, sp)
                    end
                else
                    gtx[o]=nil
                end
            end
        end
    end
end))
local function handle(o)
    if dead or not o.Parent then
        return
    end
    local k=kind(o)
    if k and s[kf[k]] then
        pcall(function()
            local d=capture(o, k)
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
    local fa=fs.spec ~= nil and fs.game
    if not (fa or s.ptc or s.tex or s.pst or s.mat) then
        return
    end
    local k=kind(o)
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
    local sp=skey(sv.active)
    if sp then
        setfont(sp, true)
    end
end
theme(true)
xpcall(function()
    s.mbi=1
    local TS=game:GetService("TeleportService")
    local srv={}
    local busy=false
    local scanpages=2
    local minfree=1
    local maxping=0
    local maxpl=0
    local view=1
    local vnames={"Ping", "Players", "FPS"}
    local vmodes={"ping", "players", "fps"}
    local tplist
    local tpi=0
    local function rq(o)
        local f=request or http_request or (syn and syn.request) or (fluxus and fluxus.request) or (type(env.http) == "table" and env.http.request)
        if type(f) ~= "function" then
            return nil
        end
        local ok, r=pcall(f, o)
        if ok and type(r) == "table" then
            return r
        end
        return nil
    end
    local function clip(t)
        local f=setclipboard or toclipboard or set_clipboard or (syn and syn.write_clipboard)
        if type(f) ~= "function" then
            return false
        end
        return (pcall(f, t))
    end
    local function req(url)
        local r=rq({Url=url, Method="GET"})
        if r then
            if r.StatusCode == 200 then
                return r.Body
            end
            return nil, r.StatusCode
        end
        local ok, b=pcall(function()
            return game:HttpGet(url)
        end)
        if ok and type(b) == "string" then
            return b
        end
        return nil, "nohttp"
    end
    local function fetch()
        local out={}
        local cursor=""
        local code=nil
        for p=1, scanpages do
            local url="https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&excludeFullGames=true&limit=100"
            if cursor ~= "" then
                url=url .. "&cursor=" .. cursor
            end
            local body, c=req(url)
            if not body then
                code=c
                break
            end
            local ok, d=pcall(function()
                return http:JSONDecode(body)
            end)
            if not ok or type(d) ~= "table" or type(d.data) ~= "table" then
                code="bad"
                break
            end
            for _, v in ipairs(d.data) do
                if type(v) == "table" and v.id ~= game.JobId and type(v.playing) == "number" and type(v.maxPlayers) == "number" and v.maxPlayers - v.playing >= minfree then
                    local pn=tonumber(v.ping)
                    if (maxping <= 0 or (pn and pn <= maxping)) and (maxpl <= 0 or v.playing <= maxpl) then
                        table.insert(out, {id=v.id, ping=pn, fps=tonumber(v.fps), playing=v.playing, max=v.maxPlayers})
                    end
                end
            end
            if type(d.nextPageCursor) ~= "string" then
                break
            end
            cursor=d.nextPageCursor
            if p < scanpages then
                task.wait(0.6)
            end
        end
        return out, code
    end
    local function sortlist(l, m)
        local out={}
        for _, v in ipairs(l) do
            if (m ~= "ping" or v.ping) and (m ~= "fps" or v.fps) then
                table.insert(out, v)
            end
        end
        if m == "random" then
            for i=#out, 2, -1 do
                local j=math.random(i)
                out[i], out[j]=out[j], out[i]
            end
        else
            table.sort(out, function(a, b)
                if m == "players" then
                    return a.playing < b.playing
                elseif m == "fps" then
                    return a.fps > b.fps
                end
                return a.ping < b.ping
            end)
        end
        return out
    end
    local function tpgo()
        tpi=tpi + 1
        local v=tplist and tplist[tpi]
        if not v or tpi > 5 then
            tplist=nil
            busy=false
            notify("No server available")
            return
        end
        notify("Joining " .. (v.ping and (math.floor(v.ping + 0.5) .. " ms") or "server") .. "...")
        local ok=pcall(function()
            TS:TeleportToPlaceInstance(game.PlaceId, v.id, lp)
        end)
        if not ok then
            tpgo()
            return
        end
        task.delay(12, function()
            busy=false
        end)
    end
    table.insert(life, TS.TeleportInitFailed:Connect(function(plr)
        if tplist and not dead and plr == lp then
            tpgo()
        end
    end))
    local function hop(mode, listonly)
        if busy then
            return
        end
        busy=true
        notify("Scanning servers...")
        task.spawn(function()
            local list, code=fetch()
            if dead then
                return
            end
            if #list == 0 then
                busy=false
                if code == 429 then
                    notify("Rate limited, try again later")
                elseif code == "nohttp" then
                    notify("Executor has no HTTP support")
                elseif maxping > 0 then
                    notify("No server under " .. maxping .. " ms")
                else
                    notify("No server found")
                end
                return
            end
            if listonly then
                srv=list
                busy=false
                notify(#list .. " servers found")
                refresh()
                return
            end
            local l=sortlist(list, mode)
            if #l == 0 then
                busy=false
                notify("Servers have no " .. mode .. " data")
                return
            end
            tplist=l
            tpi=0
            tpgo()
        end)
    end
    local function pcolor(p)
        if not p then
            return nil
        end
        if p <= 80 then
            return col.green
        elseif p <= 150 then
            return col.yellow
        end
        return col.red
    end
    pages.servers=function()
        section("CURRENT SERVER")
        local ok, p=pcall(function()
            return lp:GetNetworkPing() * 1000
        end)
        row("Ping: " .. ((ok and p) and (math.floor(p) .. " ms") or "--") .. "   Players: " .. #plrs:GetPlayers() .. "/" .. plrs.MaxPlayers, {})
        row("Job ID: " .. string.sub(game.JobId, 1, 8) .. "...", {
            {t="Copy", w=64, f=function()
                notify(clip(game.JobId) and "Job ID copied" or "Executor has no clipboard support")
            end}
        })
        btn("Rejoin this server", "Rejoin", function()
            notify("Rejoining...")
            pcall(function()
                TS:TeleportToPlaceInstance(game.PlaceId, game.JobId, lp)
            end)
        end)
        section("QUICK HOP")
        btn("Lowest ping", "Hop", function()
            hop("ping")
        end, col.green)
        btn("Fewest players", "Hop", function()
            hop("players")
        end)
        btn("Highest FPS", "Hop", function()
            hop("fps")
        end)
        btn("Random server", "Hop", function()
            hop("random")
        end)
        section("JOIN BY ID")
        form({"Job ID"}, "Join", function(v)
            local id=trim(v[1])
            if not id:match("^[%w%-]+$") then
                notify("Invalid Job ID")
                return
            end
            if busy then
                return
            end
            busy=true
            tplist={{id=id}}
            tpi=0
            tpgo()
        end)
        section("FILTERS")
        slider("Pages to Scan", 1, 8, scanpages, function(v)
            scanpages=v
        end, 1)
        slider("Min Free Slots", 1, 10, minfree, function(v)
            minfree=v
        end, 1)
        slider("Max Ping (0 = off)", 0, 400, maxping, function(v)
            maxping=v
        end, 10)
        slider("Max Players (0 = off)", 0, 100, maxpl, function(v)
            maxpl=v
        end, 1)
        section("SERVER LIST")
        btn("Scan servers", "Scan", function()
            hop("ping", true)
        end)
        btn("Sort: " .. vnames[view], "Change", function()
            view=view % #vnames + 1
            refresh()
        end)
        local shown=sortlist(srv, vmodes[view])
        for i=1, math.min(#shown, 12) do
            local v=shown[i]
            row((v.ping and (math.floor(v.ping + 0.5) .. " ms") or "? ms") .. "  |  " .. v.playing .. "/" .. v.max .. "  |  " .. (v.fps and (math.floor(v.fps + 0.5) .. " fps") or "? fps"), {
                {t="Join", w=64, c=pcolor(v.ping), f=function()
                    if busy then
                        return
                    end
                    busy=true
                    tplist={v}
                    tpi=0
                    tpgo()
                end}
            })
        end
    end
    if type(cf.dc) ~= "string" or not cf.dc:match("^[%w%-_]+$") then
        cf.dc="rAnMe9pN2D"
    end
    local function dcode(v)
        v=trim(v)
        return v:match("discord%.gg/([%w%-_]+)") or v:match("discord%.com/invite/([%w%-_]+)") or v:match("discordapp%.com/invite/([%w%-_]+)") or (v:match("^[%w%-_]+$") and v)
    end
    local function djoin(code)
        local r=rq({
            Url="http://127.0.0.1:6463/rpc?v=1",
            Method="POST",
            Headers={["Content-Type"]="application/json", ["Origin"]="https://discord.com"},
            Body=http:JSONEncode({cmd="INVITE_BROWSER", args={code=code}, nonce=http:GenerateGUID(false)})
        })
        if r and (r.StatusCode == 200 or r.Success) then
            notify("Opening Discord...")
            return
        end
        if clip("https://discord.gg/" .. code) then
            notify("Discord app not found, link copied")
        else
            notify("Could not open Discord")
        end
    end
    pages.discord=function()
        section("DISCORD")
        row(cf.dc ~= "" and ("discord.gg/" .. cf.dc) or "No invite set", {})
        if cf.dc ~= "" then
            btn("Join Discord", "Join", function()
                task.spawn(djoin, cf.dc)
            end, col.green)
            btn("Copy invite link", "Copy", function()
                notify(clip("https://discord.gg/" .. cf.dc) and "Link copied" or "Executor has no clipboard support")
            end)
        end
        section("SET INVITE")
        form({"Invite link or code"}, "Save", function(v)
            local c=dcode(v[1])
            if not c then
                notify("Invalid invite")
                return
            end
            cf.dc=c
            later()
            notify("Invite saved")
            refresh()
        end)
    end
    local sc=type(cf.sl) == "table" and cf.sl or {}
    local function nm(v, d, a, b)
        v=tonumber(v)
        if not v then
            return d
        end
        return math.clamp(v, a, b)
    end
    local C={
        on=sc.on == true,
        m=math.floor(nm(sc.m, 1, 1, 2)),
        st=math.floor(nm(sc.st, 1, 1, 7)),
        sz=nm(sc.sz, 10, 4, 40),
        th=nm(sc.th, 2, 1, 6),
        tr=nm(sc.tr, 0, 0, 0.9),
        c={255, 255, 255},
        ol=sc.ol ~= false,
        al=sc.al == true,
        rb=sc.rb == true,
        fl=sc.fl == true,
        off=nm(sc.off, 1.75, 0, 4)
    }
    if type(sc.c) == "table" then
        C.c={math.floor(nm(sc.c[1], 255, 0, 255)), math.floor(nm(sc.c[2], 255, 0, 255)), math.floor(nm(sc.c[3], 255, 0, 255))}
    end
    cf.sl=C
    local snames={"Dot", "Ring", "Dot + Ring", "Cross", "Crosshair", "Plus", "Diamond"}
    local canfile=cs and type(getcustomasset) == "function"
    local curl
    local ours={}
    local flon=false
    local cg
    local box
    local els={}
    local hid=false
    local ocur=true
    local pvt=0
    local function paint(c)
        for _, e in ipairs(els) do
            if e.k == "f" then
                e.o.BackgroundColor3=c
            else
                e.o.Color=c
            end
        end
    end
    local function fr(w, h, x, y, round)
        local f=Instance.new("Frame")
        f.AnchorPoint=Vector2.new(0.5, 0.5)
        f.Size=UDim2.fromOffset(w, h)
        f.Position=UDim2.fromOffset(x, y)
        f.BorderSizePixel=0
        f.BackgroundTransparency=C.tr
        if round then
            local u=Instance.new("UICorner")
            u.CornerRadius=UDim.new(1, 0)
            u.Parent=f
        end
        if C.ol then
            local k=Instance.new("UIStroke")
            k.Color=Color3.new(0, 0, 0)
            k.Thickness=1
            k.Transparency=math.min(1, C.tr + 0.2)
            k.Parent=f
        end
        f.Parent=box
        table.insert(els, {k="f", o=f})
    end
    local function ring(d, round)
        local f=Instance.new("Frame")
        f.AnchorPoint=Vector2.new(0.5, 0.5)
        f.Size=UDim2.fromOffset(d, d)
        f.Position=UDim2.fromOffset(0, 0)
        f.BackgroundTransparency=1
        f.BorderSizePixel=0
        if round then
            local u=Instance.new("UICorner")
            u.CornerRadius=UDim.new(1, 0)
            u.Parent=f
        end
        local k=Instance.new("UIStroke")
        k.Thickness=C.th
        k.Transparency=C.tr
        k.Parent=f
        f.Parent=box
        table.insert(els, {k="s", o=k})
        return f
    end
    local function build()
        if cg then
            cg:Destroy()
        end
        els={}
        cg=Instance.new("ScreenGui")
        cg.Name="CatCrosshair"
        cg.IgnoreGuiInset=true
        cg.ResetOnSpawn=false
        cg.DisplayOrder=4
        cg.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
        box=Instance.new("Frame")
        box.Position=UDim2.fromScale(0.5, 0.5)
        box.Size=UDim2.fromOffset(0, 0)
        box.BackgroundTransparency=1
        box.Parent=cg
        local sz=C.sz
        local th=C.th
        local st=C.st
        if st == 1 then
            local d=math.max(2, sz * 0.5)
            fr(d, d, 0, 0, true)
        elseif st == 2 then
            ring(sz, true)
        elseif st == 3 then
            ring(sz, true)
            fr(th + 2, th + 2, 0, 0, true)
        elseif st == 4 or st == 5 then
            local g=sz * 0.35
            local l=sz * 0.65
            fr(l, th, -(g + l / 2), 0)
            fr(l, th, g + l / 2, 0)
            fr(th, l, 0, -(g + l / 2))
            fr(th, l, 0, g + l / 2)
            if st == 5 then
                fr(th + 1, th + 1, 0, 0, true)
            end
        elseif st == 6 then
            fr(sz * 1.6, th, 0, 0)
            fr(th, sz * 1.6, 0, 0)
        else
            ring(sz * 0.8, false).Rotation=45
        end
        paint(Color3.fromRGB(C.c[1], C.c[2], C.c[3]))
        cg.Enabled=false
        cg.Parent=gui.Parent
    end
    local crct
    local function crc32(str)
        if not crct then
            crct={}
            for i=0, 255 do
                local x=i
                for _=1, 8 do
                    if bit32.band(x, 1) == 1 then
                        x=bit32.bxor(bit32.rshift(x, 1), 0xEDB88320)
                    else
                        x=bit32.rshift(x, 1)
                    end
                end
                crct[i]=x
            end
        end
        local x=0xFFFFFFFF
        for i=1, #str do
            x=bit32.bxor(crct[bit32.band(bit32.bxor(x, string.byte(str, i)), 255)], bit32.rshift(x, 8))
        end
        return bit32.bxor(x, 0xFFFFFFFF)
    end
    local function u32(n)
        return string.char(bit32.band(bit32.rshift(n, 24), 255), bit32.band(bit32.rshift(n, 16), 255), bit32.band(bit32.rshift(n, 8), 255), bit32.band(n, 255))
    end
    local function chunk(t, d)
        return u32(#d) .. t .. d .. u32(crc32(t .. d))
    end
    local function makepng()
        local N=96
        local half=N / 2
        local SS=3
        local sz=C.sz
        local th=C.th
        local st=C.st
        local fns={}
        local function circ(r)
            table.insert(fns, function(x, y)
                return math.sqrt(x * x + y * y) - r
            end)
        end
        local function annulus(rm, w)
            table.insert(fns, function(x, y)
                return math.abs(math.sqrt(x * x + y * y) - rm) - w / 2
            end)
        end
        local function bx(cx, cy, hx, hy)
            table.insert(fns, function(x, y)
                local qx=math.abs(x - cx) - hx
                local qy=math.abs(y - cy) - hy
                local ox=math.max(qx, 0)
                local oy=math.max(qy, 0)
                return math.sqrt(ox * ox + oy * oy) + math.min(math.max(qx, qy), 0)
            end)
        end
        if st == 1 then
            circ(math.max(1, sz * 0.25))
        elseif st == 2 then
            annulus(sz / 2 - th / 2, th)
        elseif st == 3 then
            annulus(sz / 2 - th / 2, th)
            circ((th + 2) / 2)
        elseif st == 4 or st == 5 then
            local g=sz * 0.35
            local l=sz * 0.65
            bx(-(g + l / 2), 0, l / 2, th / 2)
            bx(g + l / 2, 0, l / 2, th / 2)
            bx(0, -(g + l / 2), th / 2, l / 2)
            bx(0, g + l / 2, th / 2, l / 2)
            if st == 5 then
                circ((th + 1) / 2)
            end
        elseif st == 6 then
            bx(0, 0, sz * 0.8, th / 2)
            bx(0, 0, th / 2, sz * 0.8)
        else
            table.insert(fns, function(x, y)
                return math.abs((math.abs(x) + math.abs(y)) * 0.70711 - sz * 0.4) - th / 2
            end)
        end
        local base=1 - C.tr
        local r0=C.c[1]
        local g0=C.c[2]
        local b0=C.c[3]
        local rows={}
        for py=0, N - 1 do
            local row={string.char(0)}
            for px=0, N - 1 do
                local sa=0
                local sr=0
                local sg=0
                local sb=0
                for iy=0, SS - 1 do
                    for ix=0, SS - 1 do
                        local x=px + (ix + 0.5) / SS - half
                        local y=py + (iy + 0.5) / SS - half
                        local d=1e9
                        for _, f in ipairs(fns) do
                            local v=f(x, y)
                            if v < d then
                                d=v
                            end
                        end
                        if d <= 0 then
                            sa=sa + base
                            sr=sr + r0 * base
                            sg=sg + g0 * base
                            sb=sb + b0 * base
                        elseif C.ol and d <= 1 then
                            sa=sa + base * 0.85
                        end
                    end
                end
                if sa > 0 then
                    row[#row + 1]=string.char(math.floor(sr / sa + 0.5), math.floor(sg / sa + 0.5), math.floor(sb / sa + 0.5), math.floor(sa / (SS * SS) * 255 + 0.5))
                else
                    row[#row + 1]=string.char(0, 0, 0, 0)
                end
            end
            rows[#rows + 1]=table.concat(row)
        end
        local raw=table.concat(rows)
        local a=1
        local b=0
        for i=1, #raw do
            a=(a + string.byte(raw, i)) % 65521
            b=(b + a) % 65521
        end
        local ln=#raw
        local nl=65535 - ln
        local z=string.char(120, 1, 1, ln % 256, math.floor(ln / 256), nl % 256, math.floor(nl / 256)) .. raw .. u32(b * 65536 + a)
        return string.char(137, 80, 78, 71, 13, 10, 26, 10) .. chunk("IHDR", u32(N) .. u32(N) .. string.char(8, 6, 0, 0, 0)) .. chunk("IDAT", z) .. chunk("IEND", "")
    end
    local function genurl()
        if not canfile then
            return nil
        end
        local key=C.st .. "_" .. C.sz .. "_" .. C.th .. "_" .. math.floor(C.tr * 100 + 0.5) .. "_" .. C.c[1] .. "_" .. C.c[2] .. "_" .. C.c[3] .. "_" .. (C.ol and 1 or 0)
        local path="CatCursor_" .. key .. ".png"
        local ok, url=pcall(function()
            if not isfile(path) then
                writefile(path, makepng())
            end
            return getcustomasset(path)
        end)
        if ok and type(url) == "string" and url ~= "" then
            return url
        end
        return nil
    end
    local cicreated=false
    local ciorig=nil
    local function setci(url)
        local ps=lp:FindFirstChild("PlayerScripts")
        local pm=ps and ps:FindFirstChild("PlayerModule")
        local cm=pm and pm:FindFirstChild("CameraModule")
        local m=cm and cm:FindFirstChild("MouseLockController")
        if not m then
            return
        end
        local v=m:FindFirstChild("CursorImage")
        if url then
            if not (v and v:IsA("StringValue")) then
                if v then
                    v:Destroy()
                end
                v=Instance.new("StringValue")
                v.Name="CursorImage"
                v.Parent=m
                cicreated=true
            elseif not cicreated and ciorig == nil then
                ciorig=v.Value
            end
            v.Value=url
        elseif v then
            if cicreated then
                v:Destroy()
            elseif ciorig ~= nil then
                v.Value=ciorig
            end
            cicreated=false
            ciorig=nil
        end
    end
    local function sysapply()
        if C.on and C.m == 1 and curl then
            setci(curl)
        else
            setci(nil)
        end
    end
    local gtk=0
    local function regen(now)
        local function go()
            curl=nil
            if C.on and C.m == 1 then
                curl=genurl()
                if curl then
                    ours[curl]=true
                end
            end
            sysapply()
            if cg then
                build()
            end
            later()
        end
        if now then
            go()
            return
        end
        gtk=gtk + 1
        local id=gtk
        task.delay(0.35, function()
            if id == gtk and not dead then
                go()
            end
        end)
    end
    local function showcur()
        if hid then
            hid=false
            uis.MouseIconEnabled=ocur
        end
    end
    local function flset(on)
        local was=flon
        flon=on
        if on or not was then
            return
        end
        local ch=lp.Character
        local hum=ch and ch:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.AutoRotate=true
            hum.CameraOffset=Vector3.new(0, 0, 0)
        end
        if uis.MouseBehavior == Enum.MouseBehavior.LockCenter then
            uis.MouseBehavior=Enum.MouseBehavior.Default
        end
    end
    local function forceupd()
        if not (C.fl and flon) then
            return
        end
        local ch=lp.Character
        local hum=ch and ch:FindFirstChildOfClass("Humanoid")
        if main.Visible then
            uis.MouseBehavior=Enum.MouseBehavior.Default
            if hum then
                hum.AutoRotate=true
            end
            return
        end
        uis.MouseBehavior=Enum.MouseBehavior.LockCenter
        local root=ch and ch:FindFirstChild("HumanoidRootPart")
        local cam=workspace.CurrentCamera
        if hum and root and cam and hum.Health > 0 and not hum.Sit then
            hum.AutoRotate=false
            hum.CameraOffset=hum.CameraOffset:Lerp(Vector3.new(C.off, 0, 0), 0.2)
            local lv=cam.CFrame.LookVector
            root.CFrame=CFrame.new(root.Position) * CFrame.Angles(0, math.atan2(-lv.X, -lv.Z), 0)
        end
    end
    table.insert(life, uis.InputBegan:Connect(function(i, gp)
        if gp or dead or not C.fl then
            return
        end
        if i.KeyCode == Enum.KeyCode.LeftShift then
            flset(not flon)
        end
    end))
    local function crossupd()
        forceupd()
        local locked=uis.MouseBehavior == Enum.MouseBehavior.LockCenter
        local sys=C.on and C.m == 1 and curl ~= nil
        local ov=C.on and not sys
        if sys then
            if locked and (flon or uis.MouseIcon ~= "") then
                if uis.MouseIcon ~= curl then
                    uis.MouseIcon=curl
                end
            end
        end
        if not locked and ours[uis.MouseIcon] and os.clock() > pvt then
            uis.MouseIcon=""
        end
        local prev=not curl and os.clock() < pvt
        local want=prev or (ov and (locked or C.al) and (locked or not main.Visible))
        if want then
            if not cg then
                build()
            end
            cg.Enabled=true
            if C.rb then
                paint(Color3.fromHSV((os.clock() * 0.25) % 1, 0.8, 1))
            end
        elseif cg then
            cg.Enabled=false
        end
        if want and locked then
            if not hid then
                hid=true
                ocur=uis.MouseIconEnabled
            end
            if uis.MouseIconEnabled then
                uis.MouseIconEnabled=false
            end
        else
            showcur()
        end
    end
    pages["shift lock"]=function()
        section("CUSTOM CURSOR")
        toggle("Enable", C.on, function(v)
            C.on=v
            regen(true)
        end)
        btn("Mode: " .. (C.m == 1 and "System cursor" or "Overlay (GUI)"), "Change", function()
            C.m=C.m % 2 + 1
            regen(true)
            refresh()
        end)
        if C.m == 1 and not canfile then
            row("No file support, overlay is used instead", {})
        end
        btn("Preview (3s)", "Show", function()
            pvt=os.clock() + 3
            if curl then
                uis.MouseIcon=curl
            end
        end)
        section("SHIFT LOCK")
        toggle("Force Shift Lock (Shift key)", C.fl, function(v)
            C.fl=v
            if not v then
                flset(false)
            end
            later()
        end)
        slider("Camera Offset", 0, 4, C.off, function(v)
            C.off=v
            later()
        end, 0.25)
        section("STYLE")
        for i, n in ipairs(snames) do
            local i=i
            btn(n, C.st == i and "Active" or "Use", function()
                C.st=i
                regen(true)
                pvt=os.clock() + 2
                if curl then
                    uis.MouseIcon=curl
                end
                refresh()
            end, C.st == i and col.green or nil)
        end
        section("LOOK")
        slider("Size", 4, 40, C.sz, function(v)
            C.sz=v
            regen()
        end, 1)
        slider("Thickness", 1, 6, C.th, function(v)
            C.th=v
            regen()
        end, 1)
        slider("Transparency", 0, 0.9, C.tr, function(v)
            C.tr=v
            regen()
        end, 0.05)
        colorin("Color", Color3.fromRGB(C.c[1], C.c[2], C.c[3]), function(cr)
            C.c={math.round(cr.R * 255), math.round(cr.G * 255), math.round(cr.B * 255)}
            regen()
        end)
        toggle("Outline", C.ol, function(v)
            C.ol=v
            regen()
        end)
        if C.m == 2 then
            toggle("Always Show", C.al, function(v)
                C.al=v
                later()
            end)
            toggle("Rainbow", C.rb, function(v)
                C.rb=v
                if not v and cg then
                    paint(Color3.fromRGB(C.c[1], C.c[2], C.c[3]))
                end
                later()
            end)
        end
    end
    s.kill=function()
        flset(false)
        setci(nil)
        if cg then
            cg:Destroy()
            cg=nil
        end
        showcur()
        if ours[uis.MouseIcon] then
            uis.MouseIcon=""
        end
    end
    s.slreset=function()
        C.on=false
        C.m=1
        C.st=1
        C.sz=10
        C.th=2
        C.tr=0
        C.c={255, 255, 255}
        C.ol=true
        C.al=false
        C.rb=false
        C.fl=false
        C.off=1.75
        s.kill()
    end
    if C.on then
        regen(true)
    end
    local vg
    local vcol=Color3.new(0, 0, 0)
    local vint=0.6
    local function vfix()
        if not (vg and vg.Parent) then
            return
        end
        for _, f in ipairs(vg:GetChildren()) do
            if f:IsA("Frame") then
                f.BackgroundColor3=vcol
                f.UIGradient.Transparency=NumberSequence.new(1 - vint, 1)
            end
        end
    end
    local function vset(on)
        s.vig=on
        if on then
            if not (vg and vg.Parent) then
                vg=Instance.new("ScreenGui")
                vg.Name="CatVignette"
                vg.IgnoreGuiInset=true
                vg.ResetOnSpawn=false
                vg.DisplayOrder=5
                vg.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
                local defs={
                    {UDim2.new(1, 0, 0.3, 0), UDim2.new(0, 0, 0, 0), 90},
                    {UDim2.new(1, 0, 0.3, 0), UDim2.new(0, 0, 0.7, 0), 270},
                    {UDim2.new(0.22, 0, 1, 0), UDim2.new(0, 0, 0, 0), 0},
                    {UDim2.new(0.22, 0, 1, 0), UDim2.new(0.78, 0, 0, 0), 180}
                }
                for _, d in ipairs(defs) do
                    local f=Instance.new("Frame")
                    f.Size=d[1]
                    f.Position=d[2]
                    f.BorderSizePixel=0
                    f.BackgroundColor3=vcol
                    local g=Instance.new("UIGradient")
                    g.Rotation=d[3]
                    g.Parent=f
                    f.Parent=vg
                end
                vg.Parent=gui.Parent
            end
            vfix()
            vg.Enabled=true
        elseif vg then
            vg.Enabled=false
        end
    end
    local mb
    local lv
    local function setmb(on)
        s.mb=on
        lv=nil
        if on then
            if not (mb and mb.Parent) then
                mb=Instance.new("BlurEffect")
                mb.Name="CatMotionBlur"
                mb.Size=0
                mb.Parent=lt
            end
        elseif mb then
            mb:Destroy()
            mb=nil
        end
    end
    local looks={
        {n="Cinematic", fx={bloom={Intensity=0.5, Size=36, Threshold=0.9}, rays={Intensity=0.15, Spread=0.8}}, cc={-0.02, 0.2, -0.05, Color3.fromRGB(255, 235, 215)}},
        {n="Dreamy", fx={bloom={Intensity=0.9, Size=56, Threshold=0.7}, dof={FarIntensity=0.25, FocusDistance=60, InFocusRadius=40, NearIntensity=0}}, cc={0.05, -0.1, 0.3, Color3.fromRGB(255, 220, 245)}},
        {n="Cyber Glow", fx={bloom={Intensity=1.2, Size=40, Threshold=0.6}}, cc={0, 0.25, 0.5, Color3.fromRGB(200, 220, 255)}},
        {n="Clean", fx={}, cc={0, 0.05, 0.15, Color3.new(1, 1, 1)}}
    }
    local function lookapply(d)
        for k in pairs(ed) do
            if d.fx[k] then
                seteffect(k, true)
                for p, v in pairs(d.fx[k]) do
                    setev(k, p, v)
                end
            else
                seteffect(k, false)
            end
        end
        setcc("Brightness", d.cc[1])
        setcc("Contrast", d.cc[2])
        setcc("Saturation", d.cc[3])
        setcc("TintColor", d.cc[4])
        notify(d.n .. " applied")
    end
    s.fxpage=function()
        section("LOOK PRESETS")
        for _, d in ipairs(looks) do
            local d=d
            btn(d.n, "Apply", function()
                lookapply(d)
                refresh()
            end)
        end
        section("VIGNETTE")
        toggle("Vignette", s.vig, vset)
        slider("Vignette Strength", 0, 1, vint, function(v)
            vint=v
            vfix()
        end, 0.05)
        colorin("Vignette Color", vcol, function(cr)
            vcol=cr
            vfix()
        end)
        section("MOTION BLUR")
        toggle("Motion Blur", s.mb, setmb)
        slider("Blur Strength", 0.2, 3, s.mbi, function(v)
            s.mbi=v
        end, 0.1)
    end
    local fbs
    s.setfb=function(on)
        local was=s.fb
        s.fb=on
        if on then
            if not was then
                fbs={}
                for _, p in ipairs({"Brightness", "Ambient", "OutdoorAmbient", "FogStart", "FogEnd"}) do
                    fbs[p]=lt[p]
                end
            end
            lt.Brightness=2.5
            lt.Ambient=Color3.fromRGB(200, 200, 200)
            lt.OutdoorAmbient=Color3.fromRGB(200, 200, 200)
            lt.FogStart=100000
            lt.FogEnd=100000
        elseif was and fbs then
            for p, v in pairs(fbs) do
                lt[p]=v
            end
            fbs=nil
        end
    end
    local rgg
    s.setrgb=function(on)
        local sk=main:FindFirstChildOfClass("UIStroke")
        if not sk then
            return
        end
        if on then
            if not (rgg and rgg.Parent) then
                rgg=Instance.new("UIGradient")
                rgg.Color=ColorSequence.new({
                    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 80, 120)),
                    ColorSequenceKeypoint.new(0.17, Color3.fromRGB(255, 200, 70)),
                    ColorSequenceKeypoint.new(0.33, Color3.fromRGB(90, 255, 140)),
                    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(70, 220, 255)),
                    ColorSequenceKeypoint.new(0.67, Color3.fromRGB(110, 120, 255)),
                    ColorSequenceKeypoint.new(0.83, Color3.fromRGB(230, 90, 255)),
                    ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 80, 120))
                })
                rgg.Parent=sk
            end
            sk.Transparency=0.15
        else
            if rgg then
                rgg:Destroy()
                rgg=nil
            end
            sk.Transparency=0.5
        end
    end
    s.navin=function()
        for i, n in ipairs(pnames) do
            local b=navs[n]
            b.BackgroundTransparency=1
            b.TextTransparency=1
            task.delay(0.03 * i, function()
                if b.Parent then
                    tn(b, {BackgroundTransparency=0, TextTransparency=0}, 0.25)
                end
            end)
        end
    end
    s.xreset=function()
        if vg then
            vg:Destroy()
            vg=nil
        end
        s.vig=false
        setmb(false)
    end
    local ost=obtn:FindFirstChildOfClass("UIStroke")
    table.insert(life, rs.RenderStepped:Connect(function(dt)
        if dead then
            return
        end
        crossupd()
        if s.mb and mb and mb.Parent then
            local cam=workspace.CurrentCamera
            if cam then
                local v=cam.CFrame.LookVector
                if lv and dt > 0 then
                    local target=math.clamp((v - lv).Magnitude / dt * 4 * s.mbi, 0, 28)
                    mb.Size=mb.Size + (target - mb.Size) * math.min(dt * 12, 1)
                end
                lv=v
            end
        end
        if cf.rgb and rgg and rgg.Parent then
            rgg.Rotation=(rgg.Rotation + 80 * dt) % 360
        end
        if ost then
            ost.Transparency=0.45 + 0.2 * math.sin(os.clock() * 2.5)
        end
    end))
    s.setrgb(cf.rgb)
end, function(e)
    warn("CatSettings extras failed: " .. tostring(e))
end)
env.CatSettingsUnload=shutdown
openpage("Performance")
notify("CAT SETTINGS loaded")
