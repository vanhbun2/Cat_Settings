local CatFns={}
local plrs=game:GetService("Players")
local lt=game:GetService("Lighting")
local tw=game:GetService("TweenService")
local uis=game:GetService("UserInputService")
local rs=game:GetService("RunService")
local stats=game:GetService("Stats")
local http=game:GetService("HttpService")
local tsv=game:GetService("TextService")
local TS=game:GetService("TeleportService")
local lp=plrs.LocalPlayer
local env=(type(getgenv)=="function" and getgenv()) or _G
local oldUnload=rawget(env,"CatSettingsUnload")
if type(oldUnload)=="function" then
    pcall(oldUnload)
end
local syn=rawget(env,"syn")
local fluxus=rawget(env,"fluxus")

function CatFns.apifn(n)
    local v=rawget(env,n)
    if type(v)=="function" then
        return v
    end
    return nil
end

function CatFns.apifnAny(a,b,c)
    return CatFns.apifn(a) or CatFns.apifn(b) or CatFns.apifn(c)
end

-- [UPGRADE] nhan dien executor bang identifyexecutor()
local EXECNAME="Unknown executor"
do
    local idf=CatFns.apifn("identifyexecutor") or CatFns.apifn("getexecutorname")
    if idf then
        local ok,a,b=pcall(idf)
        if ok and a then
            EXECNAME=tostring(a)..(b and (" "..tostring(b)) or "")
        end
    elseif syn then
        EXECNAME="Synapse X"
    elseif fluxus then
        EXECNAME="Fluxus"
    elseif rawget(env,"IDENTIFIER") then
        EXECNAME=tostring(rawget(env,"IDENTIFIER"))
    end
end

local FIO={
    write=CatFns.apifnAny("writefile","write_file"),
    read=CatFns.apifnAny("readfile","read_file"),
    isfile=CatFns.apifnAny("isfile","is_file"),
    isfolder=CatFns.apifnAny("isfolder","is_folder"),
    makefolder=CatFns.apifnAny("makefolder","make_folder"),
    listfiles=CatFns.apifnAny("listfiles","list_files"),
    customasset=CatFns.apifnAny("getcustomasset","get_custom_asset")
}

local HASFS=FIO.write~=nil and FIO.read~=nil and FIO.isfile~=nil
local HASLOCALSKY=HASFS and FIO.isfolder~=nil and FIO.listfiles~=nil and FIO.makefolder~=nil and FIO.customasset~=nil

function CatFns.httpreq(o)
    local f=CatFns.apifnAny("request","http_request") or (syn and syn.request) or (fluxus and fluxus.request)
    if not f and type(rawget(env,"http"))=="table" then
        f=rawget(env,"http").request
    end
    if type(f)~="function" then
        return nil
    end
    local ok,r=pcall(f,o)
    if ok and type(r)=="table" then
        return r
    end
    return nil
end

function CatFns.clipcopy(t)
    local f=CatFns.apifnAny("setclipboard","toclipboard","set_clipboard") or (syn and syn.write_clipboard)
    if type(f)~="function" then
        return false
    end
    local ok,result=pcall(f,t)
    return ok and result~=false
end

function CatFns.httpcapable()
    if CatFns.apifnAny("request","http_request") then
        return true
    end
    if syn and type(syn.request)=="function" then
        return true
    end
    if fluxus and type(fluxus.request)=="function" then
        return true
    end
    local h=rawget(env,"http")
    return type(h)=="table" and type(h.request)=="function"
end

function CatFns.clipcapable()
    if CatFns.apifnAny("setclipboard","toclipboard","set_clipboard") then
        return true
    end
    return syn and type(syn.write_clipboard)=="function" or false
end

local CFGFILE="CatSettings.json"
local PROFFILE="CatProfiles.json"
local CFGVER=1
local DEFAVATAR="139350569952709"
local DEFDISCORD="rAnMe9pN2D"

-- [FIX] migr bi khai bao 2 lan, chi giu 1 ban
function CatFns.migr(cfg)
    if type(cfg)~="table" then
        return nil
    end
    local ver=tonumber(cfg.Version)
    if not ver then
        ver=0
    end
    if ver<1 then
        cfg.Version=1
        if cfg.Gradient==nil then
            cfg.Gradient=false
        end
        if cfg.SafeMode==nil then
            cfg.SafeMode=false
        end
    end
    return cfg
end
local dtheme=Color3.fromRGB(120,105,255)

local cf={
    avatar=DEFAVATAR,
    theme=dtheme,
    size=1,
    tr=0.12,
    key=Enum.KeyCode.RightShift,
    notif=true,
    btn=true,
    rgb=false,
    grad=false,
    safe=false,
    sl={},
    dc=DEFDISCORD
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
    active=nil,
    sky=nil -- [UPGRADE] sky dang dung, duoc luu lai
}

local V={
    light={},
    atm={},
    fx={},
    cc={},
    sky={}
}

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
    fov=workspace.CurrentCamera and workspace.CurrentCamera.FieldOfView or 70,
    gl=1.5,
    spin=0,
    flow=0,
    cyc=0,
    hue=0,
    vig=false,
    mb=false,
    mbi=1,
    fb=false,
    still=false
}

local kf={
    e="ptc",
    p="pst",
    d="tex",
    m="mat"
}

local profiles={list={"Default"},active="Default"}
-- [FIX] sl phai duoc khai bao truoc applycfg (truoc day khai bao sau nen applycfg khong thay)
local sl={}

function CatFns.trim(v)
    return tostring(v):match("^%s*(.-)%s*$") or ""
end

function CatFns.nid(v)
    local t=tostring(v)
    return t:match("^%s*(%d+)%s*$") or t:match("rbxassetid://(%d+)") or t:match("[?&]id=(%d+)") or t:match("/library/(%d+)") or t:match("/asset/(%d+)") or t:match("/catalog/(%d+)")
end

function CatFns.hex(v)
    local h=tostring(v):gsub("[#%s]","")
    if #h~=6 or not h:match("^%x+$") then
        return nil
    end
    return Color3.fromRGB(tonumber(h:sub(1,2),16),tonumber(h:sub(3,4),16),tonumber(h:sub(5,6),16))
end

function CatFns.tohex(c)
    return string.format("#%02X%02X%02X",math.round(math.clamp(c.R,0,1)*255),math.round(math.clamp(c.G,0,1)*255),math.round(math.clamp(c.B,0,1)*255))
end

function CatFns.numOr(v,d,lo,hi)
    if type(v)~="number" then
        return d
    end
    if v~=v then
        return d
    end
    if lo and v<lo then
        v=lo
    end
    if hi and v>hi then
        v=hi
    end
    return v
end

function CatFns.boolOr(v,d)
    if type(v)=="boolean" then
        return v
    end
    return d
end

function CatFns.strOr(v,d)
    if type(v)=="string" then
        return v
    end
    return d
end

function CatFns.proffile(p)
    if p=="Default" then
        return CFGFILE
    end
    return "CatSettings_" .. (tostring(p):gsub("[^%w%-_]","_")) .. ".json"
end

function CatFns.cfgpath()
    return CatFns.proffile(profiles.active)
end

function CatFns.readjson(path)
    if not HASFS then
        return nil
    end
    local ok,d=pcall(function()
        if FIO.isfile(path) then
            return http:JSONDecode(FIO.read(path))
        end
    end)
    if ok and type(d)=="table" then
        return d
    end
    return nil
end

-- [UPGRADE] khong ghi file neu noi dung khong doi (giam IO va file .bak)
CatFns.lastjson={}
function CatFns.writejson(path,d)
    if not HASFS then
        return false
    end
    local okE,txt=pcall(function()
        return http:JSONEncode(d)
    end)
    if not okE then
        return false
    end
    if CatFns.lastjson[path]==txt then
        return true
    end
    local ok=pcall(function()
        if FIO.isfile(path) then
            pcall(function()
                FIO.write(path..".bak",FIO.read(path))
            end)
        end
        FIO.write(path,txt)
    end)
    if ok then
        CatFns.lastjson[path]=txt
    end
    return ok
end

function CatFns.loadprofiles()
    local d=CatFns.readjson(PROFFILE)
    profiles.list={"Default"}
    profiles.active="Default"
    local seen={Default=true}
    if type(d)=="table" and type(d.Profiles)=="table" then
        for _,p in ipairs(d.Profiles) do
            if type(p)=="string" and p~="" and not p:find("%W") and not seen[p] then
                seen[p]=true
                table.insert(profiles.list,p)
            end
        end
    end
    if type(d)=="table" and type(d.Active)=="string" then
        for _,p in ipairs(profiles.list) do
            if p==d.Active then
                profiles.active=p
                break
            end
        end
    end
end

function CatFns.saveprofiles()
    CatFns.writejson(PROFFILE,{Profiles=profiles.list,Active=profiles.active})
end

function CatFns.applycfg(d)
    if type(d)~="table" then
        return
    end
    table.clear(sv.skies)
    table.clear(sv.fonts)
    sv.sky=nil
    for _,t in pairs(V) do
        if type(t)=="table" then
            for k in pairs(t) do
                t[k]=nil
            end
        end
    end
    if type(d.Theme)=="table" then
        local r=tonumber(d.Theme[1])
        local g=tonumber(d.Theme[2])
        local b=tonumber(d.Theme[3])
        if r and g and b then
            cf.theme=Color3.fromRGB(math.clamp(math.floor(r),0,255),math.clamp(math.floor(g),0,255),math.clamp(math.floor(b),0,255))
        end
    end
    cf.size=CatFns.numOr(d.MenuSize,cf.size,0.7,1.2)
    cf.tr=CatFns.numOr(d.Transparency,cf.tr,0,0.5)
    cf.notif=CatFns.boolOr(d.Notifications,cf.notif)
    cf.btn=CatFns.boolOr(d.ShowButton,cf.btn)
    cf.rgb=CatFns.boolOr(d.RgbBorder,cf.rgb)
    cf.grad=CatFns.boolOr(d.Gradient,cf.grad)
    cf.safe=CatFns.boolOr(d.SafeMode,cf.safe)
    if type(d.Avatar)=="string" and CatFns.nid(d.Avatar) then
        cf.avatar=CatFns.nid(d.Avatar)
    end
    if type(d.Crosshair)=="table" then
        cf.sl=d.Crosshair
    end
    if type(d.DiscordInvite)=="string" and d.DiscordInvite:match("^[%w%-_]+$") then
        cf.dc=d.DiscordInvite
    end
    if type(d.Key)=="string" then
        local f,k=pcall(function()
            return Enum.KeyCode[d.Key]
        end)
        if f and k then
            cf.key=k
        end
    end
    fs.game=CatFns.boolOr(d.GameText,fs.game)
    fs.menu=CatFns.boolOr(d.MenuText,fs.menu)
    if type(d.ActiveFont)=="string" then
        sv.active=d.ActiveFont
        fs.key=d.ActiveFont
    end
    -- [UPGRADE] luu trang thai overlay FPS/Ping/Mem
    local st=type(d.Stats)=="table" and d.Stats or {}
    for _,k in ipairs({"fps","png","mem"}) do
        s[k]=CatFns.boolOr(st[k],false)
        if sl[k] then
            sl[k].Visible=s[k]
        end
    end
    if type(d.Skies)=="table" then
        for _,e in ipairs(d.Skies) do
            if type(e)=="table" and type(e.Name)=="string" and type(e.Ids)=="table" then
                local ids={}
                for _,r in ipairs(e.Ids) do
                    local n=CatFns.nid(r)
                    if n then
                        table.insert(ids,n)
                    end
                end
                if #ids==1 or #ids==6 then
                    table.insert(sv.skies,{Name=e.Name,Ids=ids})
                end
            end
        end
    end
    -- [UPGRADE] khoi phuc sky dang dung
    if type(d.ActiveSky)=="table" and type(d.ActiveSky.Name)=="string" and type(d.ActiveSky.Ids)=="table" then
        local ids={}
        for _,r in ipairs(d.ActiveSky.Ids) do
            local n=CatFns.nid(r)
            if n then
                table.insert(ids,n)
            end
        end
        if #ids==1 or #ids==6 then
            sv.sky={Name=d.ActiveSky.Name,Ids=ids}
        end
    end
    if type(d.Fonts)=="table" then
        for _,e in ipairs(d.Fonts) do
            if type(e)=="table" and type(e.Name)=="string" and type(e.Value)=="string" then
                table.insert(sv.fonts,{Name=e.Name,Value=e.Value})
            end
        end
    end
    if type(d.Lighting)=="table" then
        for k,v in pairs(d.Lighting) do
            if type(v)=="number" or type(v)=="string" then
                V.light[k]=v
            end
        end
    end
    if type(d.Atmosphere)=="table" then
        for k,v in pairs(d.Atmosphere) do
            if type(v)=="number" or type(v)=="string" then
                V.atm[k]=v
            end
        end
    end
    if type(d.Sky)=="table" then
        for k,v in pairs(d.Sky) do
            if type(v)=="number" or type(v)=="boolean" then
                V.sky[k]=v
            end
        end
    end
    if type(d.ColorCorrection)=="table" then
        for k,v in pairs(d.ColorCorrection) do
            if type(v)=="number" or type(v)=="string" then
                V.cc[k]=v
            end
        end
    end
    if type(d.Effects)=="table" then
        for k,t in pairs(d.Effects) do
            if type(t)=="table" then
                V.fx[k]={}
                for p,v in pairs(t) do
                    if type(v)=="number" then
                        V.fx[k][p]=v
                    end
                end
            end
        end
    end
end

function CatFns.buildcfg()
    local d={
        Version=CFGVER,
        Theme={math.round(cf.theme.R*255),math.round(cf.theme.G*255),math.round(cf.theme.B*255)},
        MenuSize=cf.size,
        Transparency=cf.tr,
        Key=cf.key.Name,
        Notifications=cf.notif,
        ShowButton=cf.btn,
        RgbBorder=cf.rgb,
        Gradient=cf.grad,
        SafeMode=cf.safe,
        Avatar=cf.avatar,
        Crosshair=cf.sl,
        DiscordInvite=cf.dc,
        Skies=sv.skies,
        ActiveSky=sv.sky,
        Fonts=sv.fonts,
        ActiveFont=fs.key,
        GameText=fs.game,
        MenuText=fs.menu,
        Stats={fps=s.fps,png=s.png,mem=s.mem},
        Lighting=V.light,
        Atmosphere=V.atm,
        Sky=V.sky,
        ColorCorrection=V.cc,
        Effects=V.fx
    }
    return d
end

local col={
    bg=Color3.fromRGB(15,17,25),
    panel=Color3.fromRGB(23,26,37),
    card=Color3.fromRGB(31,35,49),
    text=Color3.fromRGB(245,246,255),
    sub=Color3.fromRGB(153,160,180),
    acc=cf.theme,
    green=Color3.fromRGB(90,220,155),
    yellow=Color3.fromRGB(255,205,90),
    red=Color3.fromRGB(255,100,110),
    off=Color3.fromRGB(65,68,82)
}

-- [FIX LOI NGHIEM TRONG] truoc day oobj/gtx/mtx cung tro vao 1 bang "weak"
-- nen du lieu bi tron lan va setkind() bao loi "attempt to index boolean". Moi bang phai rieng.
local olt={}
local owt={}
local oobj=setmetatable({},{__mode="k"})
local gtx=setmetatable({},{__mode="k"})
local mtx=setmetatable({},{__mode="k"})
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
local spos={}
local loglist={}
local searchreg={}
local pnames={"Performance","Sky","Lighting","Effects","Shift Lock","Servers","Fonts","Interface","Discord","General"}

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
local cpanel
local spanel
local sesc
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

function CatFns.mk(cls,p,par)
    local o=Instance.new(cls)
    for k,v in pairs(p) do
        o[k]=v
    end
    o.Parent=par
    return o
end

function CatFns.rc(o,r)
    local i=Instance.new("UICorner")
    i.CornerRadius=UDim.new(0,r or 12)
    i.Parent=o
    return i
end

function CatFns.st(o,cr,t)
    local i=Instance.new("UIStroke")
    i.Color=cr or Color3.new(1,1,1)
    i.Transparency=t or 0.85
    i.Thickness=1
    i.Parent=o
    return i
end

function CatFns.ast(o,t)
    local x=CatFns.st(o,col.acc,t)
    table.insert(astk,x)
    return x
end

function CatFns.pd(o,l)
    local i=Instance.new("UIPadding")
    i.PaddingLeft=UDim.new(0,l or 10)
    i.PaddingRight=UDim.new(0,10)
    i.Parent=o
end

function CatFns.tn(o,p,d)
    if o and o.Parent then
        local t=tw:Create(o,TweenInfo.new(d or 0.2,Enum.EasingStyle.Quart,Enum.EasingDirection.Out),p)
        t:Play()
        return t
    end
    return nil
end

function CatFns.apply(o,sp)
    if not o or not o.Parent or not sp then
        return
    end
    if sp.e then
        o.Font=sp.e
    elseif sp.f then
        o.FontFace=sp.f
    end
end

local bm={
    Gotham="GothamBold",
    GothamSemibold="GothamBold",
    GothamBlack="GothamBlack",
    SourceSans="SourceSansBold",
    SourceSansLight="SourceSansBold",
    SourceSansSemibold="SourceSansBold",
    Arial="ArialBold"
}

function CatFns.mspec(b)
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
        return {f=Font.new(sp.f.Family,Enum.FontWeight.Bold,sp.f.Style)}
    end
    return sp
end

function CatFns.tx(cls,p,par,b)
    local o=CatFns.mk(cls,p,par)
    mtx[o]=b==true
    pcall(CatFns.apply,o,CatFns.mspec(b==true))
    return o
end

function CatFns.menufont()
    for o,b in pairs(mtx) do
        if o.Parent then
            pcall(CatFns.apply,o,CatFns.mspec(b))
        else
            mtx[o]=nil
        end
    end
end

function CatFns.notify(m)
    if not m then
        return
    end
    table.insert(loglist,{t=os.clock(),m=m})
    if #loglist>40 then
        table.remove(loglist,1)
    end
    if dead or not cf.notif or not toast or not toast.Parent then
        return
    end
    toasttok=toasttok+1
    local id=toasttok
    toast.Text=m
    toast.Visible=true
    toast.Position=UDim2.new(0.5,0,0,-50)
    CatFns.tn(toast,{Position=UDim2.new(0.5,0,0,14)},0.25)
    task.delay(2.2,function()
        if dead or id~=toasttok or not toast.Parent then
            return
        end
        CatFns.tn(toast,{Position=UDim2.new(0.5,0,0,-60)},0.25)
        task.delay(0.3,function()
            if id==toasttok and toast.Parent then
                toast.Visible=false
            end
        end)
    end)
end

function CatFns.reg(label,frame)
    if type(label)~="string" then
        return
    end
    local t=CatFns.trim(label)
    if t=="" then
        return
    end
    local k=t:lower()
    if searchreg[k]==nil then
        searchreg[k]={label=t,page=cur,frame=frame}
    end
end

function CatFns.drag(o,h,scaled)
    local on=false
    local di=nil
    local ds=nil
    local sp=nil
    local mv=false
    table.insert(life,h.InputBegan:Connect(function(i)
        if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
            on=true
            mv=false
            di=i
            ds=i.Position
            sp=o.Position
        end
    end))
    table.insert(life,uis.InputChanged:Connect(function(i)
        if not on then
            return
        end
        if i.UserInputType==Enum.UserInputType.MouseMovement or i==di then
            local d=i.Position-ds
            if not mv and d.Magnitude<6 then
                return
            end
            mv=true
            local f=(scaled and scale and scale.Scale) or 1
            if f<=0 then
                f=1
            end
            o.Position=UDim2.new(sp.X.Scale,sp.X.Offset+d.X/f,sp.Y.Scale,sp.Y.Offset+d.Y/f)
        end
    end))
    table.insert(life,uis.InputEnded:Connect(function(i)
        if i==di or i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
            on=false
            di=nil
        end
    end))
    return function()
        return mv
    end
end

function CatFns.hover(b,list)
    list=list or conns
    local sc=CatFns.mk("UIScale",{Scale=1},b)
    table.insert(list,b.MouseEnter:Connect(function()
        CatFns.tn(b,{BackgroundTransparency=0.2},0.12)
    end))
    table.insert(list,b.MouseLeave:Connect(function()
        CatFns.tn(b,{BackgroundTransparency=0},0.12)
        tw:Create(sc,TweenInfo.new(0.16,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Scale=1}):Play()
    end))
    table.insert(list,b.MouseButton1Down:Connect(function()
        tw:Create(sc,TweenInfo.new(0.07,Enum.EasingStyle.Quart,Enum.EasingDirection.Out),{Scale=0.94}):Play()
    end))
    table.insert(list,b.MouseButton1Up:Connect(function()
        tw:Create(sc,TweenInfo.new(0.16,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Scale=1}):Play()
    end))
end

function CatFns.nxt()
    order=order+1
    return order
end

function CatFns.card(h)
    local f=CatFns.mk("Frame",{Size=UDim2.new(1,-5,0,h),BackgroundColor3=col.card,BackgroundTransparency=1,BorderSizePixel=0,LayoutOrder=CatFns.nxt()},holder)
    CatFns.rc(f,12)
    CatFns.st(f,Color3.new(1,1,1),0.94)
    if s.still then
        f.BackgroundTransparency=0.08
        return f
    end
    cseq=cseq+1
    local id=cseq
    local pop=CatFns.mk("UIScale",{Scale=0.94},f)
    task.delay(0.015*math.min(id,18),function()
        if f.Parent then
            CatFns.tn(f,{BackgroundTransparency=0.08},0.22)
            tw:Create(pop,TweenInfo.new(0.3,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Scale=1}):Play()
        end
    end)
    return f
end

function CatFns.section(n)
    local l=CatFns.tx("TextLabel",{Size=UDim2.new(1,-5,0,24),BackgroundTransparency=1,Text=n,TextColor3=col.acc,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,TextTransparency=1,LayoutOrder=CatFns.nxt()},holder,true)
    CatFns.pd(l,4)
    CatFns.reg(n,nil)
    if s.still then
        l.TextTransparency=0
        return l
    end
    cseq=cseq+1
    local id=cseq
    task.delay(0.015*math.min(id,18),function()
        if l.Parent then
            CatFns.tn(l,{TextTransparency=0},0.2)
        end
    end)
    return l
end

function CatFns.row(n,btns,pf,frame)
    local cd=CatFns.card(48)
    local off=12
    local made={}
    for i=#btns,1,-1 do
        local b=btns[i]
        local w=b.w or 78
        local bt=CatFns.tx("TextButton",{Size=UDim2.fromOffset(w,30),Position=UDim2.new(1,-(off+w),0.5,-15),BackgroundColor3=b.c or col.acc,Text=b.t,TextColor3=col.text,TextSize=12,AutoButtonColor=false},cd,true)
        CatFns.rc(bt,9)
        CatFns.hover(bt)
        table.insert(conns,bt.MouseButton1Click:Connect(b.f))
        made[i]=bt
        off=off+w+8
    end
    local p={
        Size=UDim2.new(1,-(off+14),1,0),
        Position=UDim2.fromOffset(14,0),
        BackgroundTransparency=1,
        Text=n,
        TextColor3=col.text,
        TextSize=13,
        TextXAlignment=Enum.TextXAlignment.Left,
        TextTruncate=Enum.TextTruncate.AtEnd
    }
    if pf then
        p.Font=pf
        CatFns.mk("TextLabel",p,cd)
    else
        CatFns.tx("TextLabel",p,cd)
    end
    CatFns.reg(n,cd)
    return cd,made
end

function CatFns.btn(n,t,f,c,frame)
    local _,m=CatFns.row(n,{{t=t or "Apply",f=f,c=c}},nil,frame)
    return m[1]
end

function CatFns.toggle(n,init,cb)
    local cd=CatFns.card(48)
    local on=init==true
    CatFns.tx("TextLabel",{Size=UDim2.new(1,-90,1,0),Position=UDim2.fromOffset(14,0),BackgroundTransparency=1,Text=n,TextColor3=col.text,TextSize=13,TextXAlignment=Enum.TextXAlignment.Left},cd)
    local tg=CatFns.mk("TextButton",{Size=UDim2.fromOffset(46,25),Position=UDim2.new(1,-60,0.5,-12),BackgroundColor3=on and col.acc or col.off,Text="",AutoButtonColor=false},cd)
    CatFns.rc(tg,20)
    local kn=CatFns.mk("Frame",{Size=UDim2.fromOffset(19,19),Position=on and UDim2.new(1,-22,0,3) or UDim2.fromOffset(3,3),BackgroundColor3=Color3.new(1,1,1),BorderSizePixel=0},tg)
    CatFns.rc(kn,20)
    local function paint(v)
        CatFns.tn(tg,{BackgroundColor3=v and col.acc or col.off})
        tw:Create(kn,TweenInfo.new(0.28,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=v and UDim2.new(1,-22,0,3) or UDim2.fromOffset(3,3)}):Play()
    end
    table.insert(conns,tg.MouseButton1Click:Connect(function()
        on=not on
        paint(on)
        cb(on)
    end))
    CatFns.reg(n,cd)
    return function(v)
        on=v==true
        paint(on)
    end
end

function CatFns.fmt(v)
    return tostring(math.floor(v*1000+0.5)/1000)
end

function CatFns.slider(n,mn,mx,init,cb,step)
    step=step or 0.1
    local cd=CatFns.card(64)
    local v=math.clamp(init,mn,mx)
    local rg=mx-mn
    CatFns.tx("TextLabel",{Size=UDim2.new(1,-90,0,22),Position=UDim2.fromOffset(14,6),BackgroundTransparency=1,Text=n,TextColor3=col.text,TextSize=13,TextXAlignment=Enum.TextXAlignment.Left},cd)
    local vl=CatFns.tx("TextLabel",{Size=UDim2.fromOffset(70,22),Position=UDim2.new(1,-84,0,6),BackgroundTransparency=1,Text=CatFns.fmt(v),TextColor3=col.acc,TextSize=12,TextXAlignment=Enum.TextXAlignment.Right},cd)
    local hit=CatFns.mk("Frame",{Size=UDim2.new(1,-28,0,26),Position=UDim2.fromOffset(14,32),BackgroundTransparency=1,Active=true},cd)
    local bar=CatFns.mk("Frame",{Size=UDim2.new(1,0,0,6),Position=UDim2.new(0,0,0.5,-3),BackgroundColor3=Color3.fromRGB(60,64,80),BorderSizePixel=0},hit)
    CatFns.rc(bar,10)
    local pc=rg==0 and 0 or (v-mn)/rg
    local fl=CatFns.mk("Frame",{Size=UDim2.new(pc,0,1,0),BackgroundColor3=col.acc,BorderSizePixel=0},bar)
    CatFns.rc(fl,10)
    local kn=CatFns.mk("Frame",{Size=UDim2.fromOffset(15,15),AnchorPoint=Vector2.new(0.5,0.5),Position=UDim2.new(pc,0,0.5,0),BackgroundColor3=Color3.new(1,1,1),BorderSizePixel=0},bar)
    CatFns.rc(kn,20)
    local on=false
    local di=nil
    local function set(x)
        local r=math.clamp((x-hit.AbsolutePosition.X)/math.max(hit.AbsoluteSize.X,1),0,1)
        local q=math.clamp(math.floor((mn+rg*r)/step+0.5)*step,mn,mx)
        q=math.floor(q*1000+0.5)/1000
        local p=rg==0 and 0 or (q-mn)/rg
        fl.Size=UDim2.new(p,0,1,0)
        kn.Position=UDim2.new(p,0,0.5,0)
        vl.Text=CatFns.fmt(q)
        if q~=v then
            v=q
            cb(v)
        end
    end
    table.insert(conns,hit.InputBegan:Connect(function(i)
        if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
            on=true
            di=i
            set(i.Position.X)
            CatFns.tn(kn,{Size=UDim2.fromOffset(20,20)},0.12)
        end
    end))
    table.insert(conns,uis.InputChanged:Connect(function(i)
        if on and (i.UserInputType==Enum.UserInputType.MouseMovement or i==di) then
            set(i.Position.X)
        end
    end))
    table.insert(conns,uis.InputEnded:Connect(function(i)
        if i==di or i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
            on=false
            di=nil
            CatFns.tn(kn,{Size=UDim2.fromOffset(15,15)},0.15)
        end
    end))
    CatFns.reg(n,cd)
    return cd
end

function CatFns.form(phs,bt,cb)
    local cnt=#phs
    local cd=CatFns.card(10+cnt*40)
    local bx={}
    for i,ph in ipairs(phs) do
        local b=CatFns.tx("TextBox",{Size=UDim2.new(1,i==cnt and -112 or -24,0,30),Position=UDim2.fromOffset(12,10+(i-1)*40),BackgroundColor3=col.panel,PlaceholderText=ph,PlaceholderColor3=col.sub,Text="",TextColor3=col.text,TextSize=12,ClearTextOnFocus=false,TextXAlignment=Enum.TextXAlignment.Left},cd)
        CatFns.rc(b,9)
        CatFns.pd(b,10)
        bx[i]=b
    end
    local function submit()
        local vals={}
        for i,b in ipairs(bx) do
            vals[i]=b.Text
        end
        cb(vals)
    end
    local sb=CatFns.tx("TextButton",{Size=UDim2.fromOffset(80,30),Position=UDim2.new(1,-92,0,10+(cnt-1)*40),BackgroundColor3=col.acc,Text=bt,TextColor3=col.text,TextSize=12,AutoButtonColor=false},cd,true)
    CatFns.rc(sb,9)
    CatFns.hover(sb)
    table.insert(conns,sb.MouseButton1Click:Connect(submit))
    table.insert(conns,bx[cnt].FocusLost:Connect(function(e)
        if e then
            submit()
        end
    end))
    CatFns.reg(phs[1],cd)
    return cd
end

function CatFns.textin(ph,cb)
    local cd=CatFns.card(48)
    local b=CatFns.tx("TextBox",{Size=UDim2.new(1,-24,0,30),Position=UDim2.fromOffset(12,9),BackgroundColor3=col.panel,PlaceholderText=ph,PlaceholderColor3=col.sub,Text="",TextColor3=col.text,TextSize=12,ClearTextOnFocus=false,TextXAlignment=Enum.TextXAlignment.Left},cd)
    CatFns.rc(b,9)
    CatFns.pd(b,10)
    local tok=0
    table.insert(conns,b:GetPropertyChangedSignal("Text"):Connect(function()
        tok=tok+1
        local id=tok
        task.delay(0.05,function()
            if id==tok and not dead then
                cb(b.Text)
            end
        end)
    end))
    CatFns.reg(ph,cd)
    return cd,b
end

function CatFns.colorin(n,init,cb)
    local cd=CatFns.card(48)
    CatFns.tx("TextLabel",{Size=UDim2.new(1,-212,1,0),Position=UDim2.fromOffset(14,0),BackgroundTransparency=1,Text=n,TextColor3=col.text,TextSize=13,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd},cd)
    local pv=CatFns.mk("Frame",{Size=UDim2.fromOffset(22,22),Position=UDim2.new(1,-190,0.5,-11),BackgroundColor3=init,BorderSizePixel=0},cd)
    CatFns.rc(pv,6)
    CatFns.st(pv,Color3.new(1,1,1),0.7)
    local bx=CatFns.tx("TextBox",{Size=UDim2.fromOffset(90,30),Position=UDim2.new(1,-160,0.5,-15),BackgroundColor3=col.panel,Text=CatFns.tohex(init),TextColor3=col.text,TextSize=12,ClearTextOnFocus=false},cd)
    CatFns.rc(bx,9)
    local sb=CatFns.tx("TextButton",{Size=UDim2.fromOffset(58,30),Position=UDim2.new(1,-64,0.5,-15),BackgroundColor3=col.acc,Text="Set",TextColor3=col.text,TextSize=12,AutoButtonColor=false},cd,true)
    CatFns.rc(sb,9)
    CatFns.hover(sb)
    local function submit()
        local cr=CatFns.hex(bx.Text)
        if not cr then
            CatFns.notify("Invalid color")
            return
        end
        pv.BackgroundColor3=cr
        bx.Text=CatFns.tohex(cr)
        cb(cr)
    end
    table.insert(conns,sb.MouseButton1Click:Connect(submit))
    table.insert(conns,bx.FocusLost:Connect(function(e)
        if e then
            submit()
        end
    end))
    CatFns.reg(n,cd)
    return cd
end

function CatFns.confirm(msg,onyes,oncancel)
    if dead then
        return
    end
    if cpanel then
        cpanel:Destroy()
        cpanel=nil
    end
    local block=CatFns.mk("Frame",{Size=UDim2.fromScale(1,1),BackgroundColor3=Color3.new(0,0,0),BackgroundTransparency=0.55,BorderSizePixel=0,ZIndex=55,Active=true},gui)
    local p=CatFns.mk("Frame",{Size=UDim2.fromOffset(380,170),AnchorPoint=Vector2.new(0.5,0.5),Position=UDim2.fromScale(0.5,0.5),BackgroundColor3=col.card,BorderSizePixel=0,ZIndex=60},block)
    CatFns.rc(p,14)
    CatFns.st(p,col.acc,0.5)
    CatFns.tx("TextLabel",{Size=UDim2.new(1,-28,0,40),Position=UDim2.fromOffset(14,14),BackgroundTransparency=1,Text=msg,TextColor3=col.text,TextSize=14,TextWrapped=true,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=61},p,true)
    -- [FIX] UDim2.new(1,-104,0,1,-46) co 5 doi so -> nut bi dat sai cho. Dung dung 4 doi so.
    local yes=CatFns.tx("TextButton",{Size=UDim2.fromOffset(90,30),Position=UDim2.new(1,-104,1,-46),BackgroundColor3=col.red,Text="Confirm",TextColor3=col.text,TextSize=12,AutoButtonColor=false,ZIndex=61},p,true)
    CatFns.rc(yes,9)
    CatFns.hover(yes,life)
    local no=CatFns.tx("TextButton",{Size=UDim2.fromOffset(90,30),Position=UDim2.new(1,-200,1,-46),BackgroundColor3=col.panel,Text="Cancel",TextColor3=col.text,TextSize=12,AutoButtonColor=false,ZIndex=61},p,true)
    CatFns.rc(no,9)
    CatFns.hover(no,life)
    local function close()
        if cpanel then
            cpanel:Destroy()
            cpanel=nil
        end
    end
    table.insert(life,yes.MouseButton1Click:Connect(function()
        close()
        if onyes then
            onyes()
        end
    end))
    table.insert(life,no.MouseButton1Click:Connect(function()
        close()
        if oncancel then
            oncancel()
        end
    end))
    cpanel=block
end

function CatFns.save()
    if not HASFS then
        return
    end
    CatFns.writejson(CatFns.cfgpath(),CatFns.buildcfg())
end

function CatFns.later()
    if not HASFS then
        return
    end
    savetok=savetok+1
    local id=savetok
    task.delay(0.6,function()
        if id==savetok and not dead then
            CatFns.save()
        end
    end)
end

local pg=lp:WaitForChild("PlayerGui",10)
if not pg then
    pg=lp:FindFirstChildOfClass("PlayerGui")
end

for _,p in ipairs({"Brightness","ClockTime","Ambient","OutdoorAmbient","ColorShift_Top","ColorShift_Bottom","FogColor","FogStart","FogEnd","ExposureCompensation","GlobalShadows","EnvironmentDiffuseScale","EnvironmentSpecularScale"}) do
    local ok,v=pcall(function()
        return lt[p]
    end)
    if ok then
        olt[p]=v
    end
end

local ter=workspace:FindFirstChildOfClass("Terrain")
if ter then
    for _,p in ipairs({"WaterWaveSize","WaterWaveSpeed","WaterReflectance"}) do
        pcall(function()
            owt[p]=ter[p]
        end)
    end
end

local oq
pcall(function()
    oq=settings().Rendering.QualityLevel
end)

local osky
local oatm
for _,o in ipairs(lt:GetChildren()) do
    if o:IsA("Sky") and not osky then
        osky=o
    end
    if o:IsA("Atmosphere") and not oatm then
        oatm=o
    end
end

-- [UPGRADE] uu tien gethui() de an GUI, sau do PlayerGui, cuoi cung CoreGui
local host
do
    local gh=CatFns.apifn("gethui")
    if gh then
        local ok,r=pcall(gh)
        if ok and typeof(r)=="Instance" then
            host=r
        end
    end
end
host=host or pg or game:GetService("CoreGui")

gui=CatFns.mk("ScreenGui",{Name="Cat",ResetOnSpawn=false,DisplayOrder=50,ZIndexBehavior=Enum.ZIndexBehavior.Sibling},host)
fade=CatFns.mk("Frame",{Name="CatFade",Size=UDim2.fromScale(1,1),Position=UDim2.fromOffset(0,0),BackgroundColor3=Color3.new(0,0,0),BackgroundTransparency=1,BorderSizePixel=0,ZIndex=30,Active=false},gui)
main=CatFns.mk("Frame",{Name="SettingsPanel",Size=UDim2.fromOffset(700,460),AnchorPoint=Vector2.new(0.5,0.5),Position=UDim2.fromScale(0.5,0.5),BackgroundColor3=col.bg,BackgroundTransparency=cf.tr,BorderSizePixel=0,ClipsDescendants=true,Visible=false,ZIndex=10},gui)
CatFns.rc(main,18)
CatFns.ast(main,0.5)
scale=CatFns.mk("UIScale",{Scale=cf.size},main)

function CatFns.fit()
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
    mscale=math.min(cf.size,math.max(0.4,(v.X-24)/700),math.max(0.4,(v.Y-24)/460))
    scale.Scale=mscale
end

function CatFns.watch()
    if camconn then
        camconn:Disconnect()
        camconn=nil
    end
    local cam=workspace.CurrentCamera
    if cam then
        camconn=cam:GetPropertyChangedSignal("ViewportSize"):Connect(CatFns.fit)
    end
    CatFns.fit()
end

side=CatFns.mk("Frame",{Size=UDim2.new(0,175,1,0),BackgroundColor3=col.panel,BorderSizePixel=0},main)
CatFns.rc(side,18)
CatFns.mk("Frame",{Size=UDim2.new(0,18,1,0),Position=UDim2.new(1,-18,0,0),BackgroundColor3=col.panel,BorderSizePixel=0},side)

function CatFns.fallback(img,size)
    local l=CatFns.tx("TextLabel",{Size=UDim2.fromScale(1,1),BackgroundTransparency=1,Text="CAT",TextColor3=col.text,TextSize=size,ZIndex=2,Visible=not img.IsLoaded},img,true)
    table.insert(life,img:GetPropertyChangedSignal("IsLoaded"):Connect(function()
        l.Visible=not img.IsLoaded
    end))
end

local av=CatFns.mk("ImageButton",{Size=UDim2.fromOffset(54,54),Position=UDim2.fromOffset(18,20),BackgroundColor3=col.card,Image="rbxassetid://"..cf.avatar,ScaleType=Enum.ScaleType.Crop,AutoButtonColor=false},side)
CatFns.rc(av,16)
CatFns.fallback(av,14)
CatFns.tx("TextLabel",{Size=UDim2.new(1,-25,0,24),Position=UDim2.fromOffset(18,82),BackgroundTransparency=1,Text="Cat",TextColor3=col.text,TextSize=15,TextXAlignment=Enum.TextXAlignment.Left},side,true)
nav=CatFns.mk("Frame",{Size=UDim2.new(1,-20,1,-126),Position=UDim2.fromOffset(10,116),BackgroundTransparency=1},side)
CatFns.mk("UIListLayout",{Padding=UDim.new(0,4),SortOrder=Enum.SortOrder.LayoutOrder},nav)
body=CatFns.mk("Frame",{Size=UDim2.new(1,-195,1,-28),Position=UDim2.fromOffset(185,14),BackgroundTransparency=1},main)
head=CatFns.mk("Frame",{Size=UDim2.new(1,0,0,48),BackgroundTransparency=1,Active=true},body)
title=CatFns.tx("TextLabel",{Size=UDim2.new(1,-190,0,32),Position=UDim2.fromOffset(4,0),BackgroundTransparency=1,Text="Performance",TextColor3=col.text,TextSize=20,TextXAlignment=Enum.TextXAlignment.Left},head,true)
local searchb=CatFns.tx("TextButton",{Size=UDim2.fromOffset(84,28),Position=UDim2.new(1,-170,0,2),BackgroundColor3=col.card,Text="Search",TextColor3=col.sub,TextSize=12,AutoButtonColor=false},head,true)
CatFns.rc(searchb,8)
CatFns.hover(searchb,life)
minb=CatFns.tx("TextButton",{Size=UDim2.fromOffset(32,32),Position=UDim2.new(1,-76,0,0),BackgroundColor3=col.card,Text="-",TextColor3=col.text,TextSize=20,AutoButtonColor=false},head,true)
CatFns.rc(minb,9)
closeb=CatFns.tx("TextButton",{Size=UDim2.fromOffset(32,32),Position=UDim2.new(1,-36,0,0),BackgroundColor3=col.card,Text="x",TextColor3=col.text,TextSize=20,AutoButtonColor=false},head,true)
CatFns.rc(closeb,9)
holder=CatFns.mk("ScrollingFrame",{Size=UDim2.new(1,0,1,-55),Position=UDim2.fromOffset(0,55),BackgroundTransparency=1,BorderSizePixel=0,ScrollBarThickness=3,ScrollBarImageColor3=col.acc,ScrollingDirection=Enum.ScrollingDirection.Y,AutomaticCanvasSize=Enum.AutomaticSize.Y,CanvasSize=UDim2.new()},body)
CatFns.mk("UIListLayout",{Padding=UDim.new(0,8),SortOrder=Enum.SortOrder.LayoutOrder},holder)
CatFns.mk("UIPadding",{PaddingBottom=UDim.new(0,12)},holder)

function CatFns.stat(k,y,init)
    local p=UDim2.new(1,-125,0,y)
    local l=CatFns.tx("TextLabel",{Size=UDim2.fromOffset(110,28),Position=p,BackgroundColor3=col.card,BackgroundTransparency=0.1,Text=init,TextColor3=col.green,TextSize=12,Visible=false,ZIndex=20,Active=true},gui,true)
    CatFns.rc(l,9)
    CatFns.drag(l,l,false)
    sl[k]=l
    spos[k]=p
end

CatFns.stat("fps",12,"FPS: --")
CatFns.stat("png",46,"PING: --")
CatFns.stat("mem",80,"MEM: --")
toast=CatFns.tx("TextLabel",{Size=UDim2.fromOffset(280,40),AnchorPoint=Vector2.new(0.5,0),Position=UDim2.new(0.5,0,0,-60),BackgroundColor3=col.card,BackgroundTransparency=0.05,TextColor3=col.text,Text="",TextSize=13,ZIndex=40,Visible=false},gui,false)
CatFns.rc(toast,12)
CatFns.ast(toast,0.6)

function CatFns.curatm()
    return lt:FindFirstChild("CatAtmosphere") or lt:FindFirstChildOfClass("Atmosphere")
end

function CatFns.cursky()
    return lt:FindFirstChild("CatSky") or lt:FindFirstChildOfClass("Sky")
end

function CatFns.rp(o,p,d)
    if o then
        local ok,v=pcall(function()
            return o[p]
        end)
        if ok and v~=nil then
            return v
        end
    end
    return d
end

function CatFns.hide(cls,store,keep)
    for _,o in ipairs(lt:GetChildren()) do
        if o:IsA(cls) and o.Name~=keep then
            table.insert(store,o)
            o.Parent=nil
        end
    end
end

local skyprops={"SkyboxBk","SkyboxDn","SkyboxFt","SkyboxLf","SkyboxRt","SkyboxUp","StarCount","CelestialBodiesShown","SunAngularSize","MoonAngularSize","SunTextureId","MoonTextureId","SkyboxOrientation"}
local atmprops={"Density","Offset","Color","Decay","Glare","Haze"}

function CatFns.catsky(fresh)
    local e=lt:FindFirstChild("CatSky")
    if e and not fresh then
        return e
    end
    if e then
        e:Destroy()
    end
    local sk=Instance.new("Sky")
    sk.Name="CatSky"
    if not fresh and osky and osky.Parent then
        for _,p in ipairs(skyprops) do
            pcall(function()
                sk[p]=osky[p]
            end)
        end
    end
    CatFns.hide("Sky",hsky,"CatSky")
    sk.Parent=lt
    return sk
end

function CatFns.catatm()
    local e=lt:FindFirstChild("CatAtmosphere")
    if e then
        return e
    end
    local a=Instance.new("Atmosphere")
    a.Name="CatAtmosphere"
    if oatm and oatm.Parent then
        for _,p in ipairs(atmprops) do
            pcall(function()
                a[p]=oatm[p]
            end)
        end
    end
    CatFns.hide("Atmosphere",hatm,"CatAtmosphere")
    a.Parent=lt
    return a
end

-- [FIX] type(v)=="Color3" luon sai (type tra ve "userdata"), phai dung typeof
function CatFns.setatm(p,v)
    local a=CatFns.catatm()
    pcall(function()
        a[p]=v
    end)
    if type(v)=="number" then
        V.atm[p]=v
    elseif type(v)=="string" then
        V.atm[p]=v
    elseif typeof(v)=="Color3" then
        V.atm[p]=CatFns.tohex(v)
    end
    CatFns.later()
end

function CatFns.lockclock()
    if lockconn then
        lockconn:Disconnect()
        lockconn=nil
    end
    if s.lck and not dead then
        lockconn=lt:GetPropertyChangedSignal("ClockTime"):Connect(function()
            if s.lck and math.abs(lt.ClockTime-s.lckt)>0.001 then
                lt.ClockTime=s.lckt
            end
        end)
    end
end

function CatFns.setclock(v)
    s.lckt=v
    lt.ClockTime=v
    V.light.ClockTime=v
    CatFns.later()
end

function CatFns.shadows()
    local b=olt.GlobalShadows
    if b==nil then
        b=true
    end
    pcall(function()
        lt.GlobalShadows=b and not s.shd
    end)
end

function CatFns.fog(on)
    s.fog=on
    if on then
        lt.FogStart=olt.FogStart or lt.FogStart
        lt.FogEnd=olt.FogEnd or lt.FogEnd
    else
        lt.FogStart=100000
        lt.FogEnd=100000
    end
    V.light.FogStart=lt.FogStart
    V.light.FogEnd=lt.FogEnd
    CatFns.later()
end

local ed={
    bloom={c="BloomEffect",n="CatBloom",d={Intensity=0.35,Size=32,Threshold=1}},
    blur={c="BlurEffect",n="CatBlur",d={Size=8}},
    rays={c="SunRaysEffect",n="CatSunRays",d={Intensity=0.25,Spread=1}},
    dof={c="DepthOfFieldEffect",n="CatDoF",d={FarIntensity=0.3,FocusDistance=50,InFocusRadius=30,NearIntensity=0}}
}

local ev={}
for k,d in pairs(ed) do
    ev[k]=table.clone(d.d)
end

local cv={
    Brightness=0,
    Contrast=0,
    Saturation=0,
    TintColor=Color3.new(1,1,1)
}

-- [UPGRADE] luu trang thai bat/tat cua hieu ung (_on) de khoi phuc dung khi load lai
function CatFns.seteffect(k,on)
    local d=ed[k]
    local e=lt:FindFirstChild(d.n)
    s[k]=on
    if V.fx[k]==nil then
        V.fx[k]={}
    end
    V.fx[k]._on=on and 1 or 0
    if on then
        if not e then
            e=Instance.new(d.c)
            e.Name=d.n
            e.Parent=lt
        end
        for p,v in pairs(ev[k]) do
            pcall(function()
                e[p]=v
            end)
        end
    elseif e then
        e:Destroy()
    end
    CatFns.later()
end

function CatFns.setev(k,p,v)
    ev[k][p]=v
    if V.fx[k]==nil then
        V.fx[k]={}
    end
    V.fx[k][p]=v
    local e=lt:FindFirstChild(ed[k].n)
    if e then
        pcall(function()
            e[p]=v
        end)
    end
    CatFns.later()
end

function CatFns.setcc(p,v)
    cv[p]=v
    if type(v)=="number" then
        V.cc[p]=v
    elseif typeof(v)=="Color3" then -- [FIX] truoc day dung type() nen TintColor khong bao gio duoc luu
        V.cc[p]=CatFns.tohex(v)
    end
    local e=lt:FindFirstChild("CatColorCorrection")
    if not e then
        e=Instance.new("ColorCorrectionEffect")
        e.Name="CatColorCorrection"
        e.Parent=lt
    end
    for n,x in pairs(cv) do
        pcall(function()
            e[n]=x
        end)
    end
    CatFns.later()
end

local ofov
function CatFns.fovbind()
    pcall(function()
        rs:UnbindFromRenderStep("CatSettingsFOV")
    end)
    if s.cfov and not dead then
        rs:BindToRenderStep("CatSettingsFOV",Enum.RenderPriority.Camera.Value+1,function()
            local cam=workspace.CurrentCamera
            if cam and cam.FieldOfView~=s.fov then
                cam.FieldOfView=math.clamp(s.fov,0.01,120)
            end
        end)
    end
end

function CatFns.setfov(on)
    local cam=workspace.CurrentCamera
    s.cfov=on
    if on then
        if cam and not ofov then
            ofov=cam.FieldOfView
        end
    else
        if cam and ofov then
            cam.FieldOfView=ofov
        end
        ofov=nil
    end
    CatFns.fovbind()
end

function CatFns.run(fn,msg)
    runtok=runtok+1
    local id=runtok
    if dead then
        return
    end
    local function exec()
        fn()
        if msg then
            CatFns.notify(msg)
        end
        task.delay(s.gl+0.1,function()
            if not dead and id==runtok then
                refresh()
            end
        end)
    end
    if s.gl<=0 then
        exec()
        return
    end
    CatFns.tn(fade,{BackgroundTransparency=0.45},0.18)
    task.delay(0.2,function()
        if dead or id~=runtok then
            return
        end
        exec()
        CatFns.tn(fade,{BackgroundTransparency=1},0.5)
    end)
end

function CatFns.applyd(o,d)
    local k=d.k
    if k=="e" then
        o.Enabled=d.en and not s.ptc
    elseif k=="p" then
        o.Enabled=d.en and not s.pst
    elseif k=="d" then
        o.Transparency=s.tex and 1 or d.t
    elseif k=="m" then
        if s.mat then
            o.Material=Enum.Material.SmoothPlastic
            o.Reflectance=0
        else
            o.Material=d.m
            o.Reflectance=d.r
        end
    end
end

function CatFns.kind(o)
    if not o or not o.Parent then
        return nil
    end
    if o:IsA("ParticleEmitter") or o:IsA("Trail") or o:IsA("Beam") then
        return "e"
    end
    if o:IsA("PostEffect") then
        if o.Name:sub(1,3)=="Cat" then
            return nil
        end
        return "p"
    end
    if o:IsA("Decal") then
        if o.Name:lower()=="face" then
            return nil
        end
        return "d"
    end
    if s.mat and o:IsA("BasePart") then
        return "m"
    end
    return nil
end

function CatFns.capture(o,k)
    local e=oobj[o]
    if e then
        return e
    end
    local d
    if k=="e" or k=="p" then
        d={k=k,en=o.Enabled}
    elseif k=="d" then
        d={k=k,t=o.Transparency}
    elseif k=="m" then
        d={k=k,m=o.Material,r=o.Reflectance}
    end
    oobj[o]=d
    return d
end

local addq={}
local addh=1
local addt=0
local rtok={e=0,p=0,d=0,m=0}
local scantok=0

-- [FIX LAG] chi 1 luot quet duy nhat cho moi co (truoc day moi co quet rieng toan bo workspace),
-- gop nhieu yeu cau lien tiep (debounce) va gioi han 2ms/frame.
function CatFns.scan()
    scantok=scantok+1
    local tok=scantok
    task.delay(0.05,function()
        if dead or tok~=scantok then
            return
        end
        if not (s.ptc or s.pst or s.tex or s.mat) then
            return
        end
        local roots={workspace,lt}
        local cam=workspace.CurrentCamera
        if cam then
            table.insert(roots,cam)
        end
        local t=os.clock()
        for _,r in ipairs(roots) do
            for _,o in ipairs(r:GetDescendants()) do
                if dead or tok~=scantok then
                    return
                end
                local k=CatFns.kind(o)
                if k and s[kf[k]] then
                    pcall(function()
                        local d=CatFns.capture(o,k)
                        if d then
                            CatFns.applyd(o,d)
                        end
                    end)
                end
                if os.clock()-t>0.002 then
                    task.wait()
                    t=os.clock()
                end
            end
        end
    end)
end

-- [FIX LAG] khoi phuc theo tung dot nho (truoc day khoi phuc het trong 1 frame => FPS tut khung).
function CatFns.restore(k)
    rtok[k]=rtok[k]+1
    local tok=rtok[k]
    local list={}
    for o,d in pairs(oobj) do
        if type(d)=="table" and d.k==k then
            list[#list+1]={o,d}
        end
    end
    if #list==0 then
        return
    end
    local function step(i)
        local o=list[i][1]
        local d=list[i][2]
        if o.Parent then
            pcall(CatFns.applyd,o,d)
        end
        if oobj[o]==d then
            oobj[o]=nil
        end
    end
    if dead then
        for i=1,#list do
            step(i)
        end
        return
    end
    task.spawn(function()
        local t=os.clock()
        for i=1,#list do
            if dead or rtok[k]~=tok then
                return
            end
            step(i)
            if os.clock()-t>0.002 then
                task.wait()
                t=os.clock()
            end
        end
    end)
end

function CatFns.setkind(k)
    if s[kf[k]] then
        rtok[k]=rtok[k]+1
        CatFns.scan()
    else
        CatFns.restore(k)
    end
end

-- Hang doi cho doi tuong moi sinh ra (xu ly toi da ~1.5ms/frame)
function CatFns.enq(o)
    addt=addt+1
    addq[addt]=o
end

function CatFns.pumpq()
    if addh>addt then
        if addt>0 then
            addh=1
            addt=0
        end
        return
    end
    local t=os.clock()
    while addh<=addt do
        local o=addq[addh]
        addq[addh]=nil
        addh=addh+1
        if o and o.Parent then
            CatFns.handle(o)
        end
        if os.clock()-t>0.0015 then
            break
        end
    end
end

function CatFns.water()
    if not ter then
        return
    end
    pcall(function()
        if s.wtr then
            ter.WaterWaveSize=0
            ter.WaterWaveSpeed=0
            ter.WaterReflectance=0
        else
            for p,v in pairs(owt) do
                ter[p]=v
            end
        end
    end)
end

function CatFns.quality()
    pcall(function()
        local r=settings().Rendering
        if s.low then
            r.QualityLevel=Enum.QualityLevel.Level01
        elseif oq then
            r.QualityLevel=oq
        end
    end)
end

function CatFns.setlag(v)
    s.lag=v
    s.shd=v
    s.ptc=v
    s.pst=v
    s.mat=v
    s.wtr=v
    s.low=v
    CatFns.shadows()
    for k in pairs(kf) do
        CatFns.setkind(k)
    end
    CatFns.water()
    CatFns.quality()
    refresh()
    CatFns.notify(v and "Fix Lag enabled" or "Fix Lag disabled")
end

function CatFns.rgraphics(q)
    s.lag=false
    s.shd=false
    s.ptc=false
    s.tex=false
    s.pst=false
    s.mat=false
    s.wtr=false
    s.low=false
    for k in pairs(kf) do
        CatFns.setkind(k)
    end
    CatFns.shadows()
    CatFns.water()
    CatFns.quality()
    if not q then
        CatFns.notify("Graphics restored")
    end
end

function CatFns.isgt(o)
    return (o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox")) and not o:IsDescendantOf(gui)
end

function CatFns.stylegt(o)
    local sp=fs.spec
    if not sp or not fs.game then
        return
    end
    if gtx[o]==nil then
        local ok,f=pcall(function()
            return o.FontFace
        end)
        if not ok then
            return
        end
        gtx[o]=f
    end
    pcall(CatFns.apply,o,sp)
end

function CatFns.sweep()
    sweeptok=sweeptok+1
    local tok=sweeptok
    task.spawn(function()
        local t=os.clock()
        -- [FIX] truoc day roots=workspace (Instance) khi pg=nil -> ipairs() bao loi
        local roots={workspace}
        if pg then
            roots={pg,workspace}
        end
        for _,r in ipairs(roots) do
            for _,o in ipairs(r:GetDescendants()) do
                if tok~=sweeptok or dead then
                    return
                end
                if CatFns.isgt(o) then
                    CatFns.stylegt(o)
                end
                if os.clock()-t>0.002 then
                    task.wait()
                    t=os.clock()
                end
            end
        end
    end)
end

function CatFns.rgt()
    sweeptok=sweeptok+1
    for o,f in pairs(gtx) do
        if o.Parent then
            pcall(function()
                o.FontFace=f
            end)
        end
    end
    table.clear(gtx)
end

function CatFns.same(o,sp)
    if sp.e then
        return o.Font==sp.e
    end
    local f=o.FontFace
    return f.Family==sp.f.Family and f.Weight==sp.f.Weight and f.Style==sp.f.Style
end

function CatFns.fbrefresh()
    for k,b in pairs(fb) do
        if b.Parent then
            local on=fs.key==k
            b.Text=on and "Active" or "Apply"
            b.BackgroundColor3=on and col.green or col.acc
        else
            fb[k]=nil
        end
    end
end

function CatFns.setfont(sp,q)
    fs.spec=sp
    fs.key=sp and sp.k or nil
    CatFns.menufont()
    if sp then
        if fs.game then
            CatFns.sweep()
        end
    else
        CatFns.rgt()
    end
    CatFns.fbrefresh()
    CatFns.later()
    if not q then
        CatFns.notify(sp and "Font applied" or "Font restored")
    end
end

function CatFns.setgame(on)
    fs.game=on
    if on and fs.spec then
        CatFns.sweep()
    else
        CatFns.rgt()
    end
    CatFns.later()
end

function CatFns.setmenu(on)
    fs.menu=on
    CatFns.menufont()
    CatFns.later()
end

function CatFns.theme(q)
    col.acc=cf.theme
    main.BackgroundTransparency=cf.tr
    holder.ScrollBarImageColor3=col.acc
    ind.BackgroundColor3=col.acc
    for i=#astk,1,-1 do
        local x=astk[i]
        if x.Parent then
            x.Color=col.acc
        else
            table.remove(astk,i)
        end
    end
    if s.setgrad then
        s.setgrad(cf.grad)
    end
    CatFns.fit()
    if not q then
        CatFns.later()
        CatFns.notify("Theme updated")
    end
end

function CatFns.clear()
    order=0
    cseq=0
    for _,o in ipairs(holder:GetChildren()) do
        if o:IsA("GuiObject") then
            o:Destroy()
        end
    end
end

local cycbase

function CatFns.rlight(q)
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
    for _,o in ipairs(hsky) do
        pcall(function()
            o.Parent=lt
        end)
    end
    for _,o in ipairs(hatm) do
        pcall(function()
            o.Parent=lt
        end)
    end
    table.clear(hsky)
    table.clear(hatm)
    for p,v in pairs(olt) do
        pcall(function()
            lt[p]=v
        end)
    end
    s.fog=true
    s.lck=false
    s.lckt=lt.ClockTime
    cycbase=nil
    CatFns.lockclock()
    CatFns.shadows()
    if s.setfb then
        s.setfb(false)
    end
    for k in pairs(V.light) do
        V.light[k]=nil
    end
    for k in pairs(V.atm) do
        V.atm[k]=nil
    end
    for k in pairs(V.sky) do
        V.sky[k]=nil
    end
    sv.sky=nil
    if not q then
        CatFns.notify("Lighting restored")
        CatFns.later()
    end
end

-- [UPGRADE] nut "Default Sky" truoc day goi rlight() lam reset ca anh sang. Gio chi khoi phuc sky.
function CatFns.rsky(q)
    s.spin=0
    local a=lt:FindFirstChild("CatSky")
    if a then
        a:Destroy()
    end
    for _,o in ipairs(hsky) do
        pcall(function()
            o.Parent=lt
        end)
    end
    table.clear(hsky)
    for k in pairs(V.sky) do
        V.sky[k]=nil
    end
    sv.sky=nil
    if not q then
        CatFns.notify("Sky restored")
        CatFns.later()
    end
end

function CatFns.reffects(q)
    for k in pairs(ed) do
        CatFns.seteffect(k,false)
        ev[k]=table.clone(ed[k].d)
    end
    local cc=lt:FindFirstChild("CatColorCorrection")
    if cc then
        cc:Destroy()
    end
    cv.Brightness=0
    cv.Contrast=0
    cv.Saturation=0
    cv.TintColor=Color3.new(1,1,1)
    CatFns.setfov(false)
    if s.xreset then
        s.xreset()
    end
    for k in pairs(V.fx) do
        V.fx[k]=nil
    end
    for k in pairs(V.cc) do
        V.cc[k]=nil
    end
    if not q then
        CatFns.notify("Effects reset")
        CatFns.later()
    end
end

local faces={"SkyboxBk","SkyboxDn","SkyboxFt","SkyboxLf","SkyboxRt","SkyboxUp"}
local skysnap

function CatFns.applycustom(e)
    local csk=CatFns.cursky()
    if csk then
        local snap={}
        local n=0
        for i,f in ipairs(faces) do
            local v=CatFns.nid(csk[f])
            if v then
                snap[i]=v
                n=n+1
            end
        end
        if n==6 then
            skysnap={Ids=snap,Name="Previous sky"}
        end
    end
    CatFns.run(function()
        local sk=CatFns.catsky(true)
        for i,f in ipairs(faces) do
            sk[f]="rbxassetid://"..(e.Ids[i] or e.Ids[1])
        end
        sv.sky={Name=e.Name,Ids=e.Ids}
        CatFns.later()
    end,e.Name.." applied")
end

local lroot="CatSkies"
local lcache="CatSkiesCache"
local fk={bk="SkyboxBk",dn="SkyboxDn",ft="SkyboxFt",lf="SkyboxLf",rt="SkyboxRt",up="SkyboxUp"}
local lfs=HASLOCALSKY
local lsk

function CatFns.scanlocal()
    lsk={}
    if not lfs then
        return
    end
    local function walk(dir,depth)
        local fl={}
        local n=0
        local subs={}
        for _,fp in ipairs(FIO.listfiles(dir)) do
            if FIO.isfolder(fp) then
                table.insert(subs,fp)
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
        if n==6 then
            table.insert(lsk,{n=dir:match("[^/\\]+$") or dir,f=fl})
        end
        if depth<3 then
            for _,sd in ipairs(subs) do
                walk(sd,depth+1)
            end
        end
    end
    pcall(function()
        if not FIO.isfolder(lroot) then
            FIO.makefolder(lroot)
        end
        walk(lroot,0)
    end)
    table.sort(lsk,function(a,b)
        return a.n<b.n
    end)
end

function CatFns.applylocal(e)
    task.spawn(function()
        local urls={}
        local ok=pcall(function()
            local base=e.n:gsub("[^%w]","_")
            if not FIO.isfolder(lcache) then
                FIO.makefolder(lcache)
            end
            for k,prop in pairs(fk) do
                local dst=lcache.."/"..base.."_"..k..".png"
                if not FIO.isfile(dst) then
                    FIO.write(dst,FIO.read(e.f[k]))
                end
                urls[prop]=FIO.customasset(dst)
            end
        end)
        if not ok then
            CatFns.notify("Local sky failed")
            return
        end
        CatFns.run(function()
            local sk=CatFns.catsky(true)
            for prop,u in pairs(urls) do
                sk[prop]=u
            end
            sv.sky=nil -- sky local khong luu duoc (asset tam)
            CatFns.later()
        end,e.n.." applied")
    end)
end

local bskies={
    {Name="SKY 01 - CLASSIC",Ids={"15983968922","15983966825","15983965025","15983967420","15983966246","15983964246"}},
    {Name="SKY 02 - NOVA",Ids={"11362875469","11362888971","11362882324","11362894347","11362878718","11363654446"}},
    {Name="SKY 03 - HALO",Ids={"13107325341","13107329809","13107334845","13107337703","13107340396","13107344387"}},
    {Name="SKY 04 - ECLIPSE",Ids={"5260808177","5260653793","5260817288","5260800833","5260811073","5260824661"}}
}

function CatFns.randomsky()
    if lfs and not lsk then
        CatFns.scanlocal()
    end
    local pool={}
    for _,e in ipairs(sv.skies) do
        local e=e
        table.insert(pool,function()
            CatFns.applycustom(e)
        end)
    end
    for _,e in ipairs(bskies) do
        local e=e
        table.insert(pool,function()
            CatFns.applycustom(e)
        end)
    end
    for _,e in ipairs(lsk or {}) do
        local e=e
        table.insert(pool,function()
            CatFns.applylocal(e)
        end)
    end
    if #pool==0 then
        CatFns.notify("No sky available")
        return
    end
    pool[math.random(#pool)]()
end

function CatFns.pids(v)
    local ids={}
    for t in tostring(v):gmatch("[^,%s;]+") do
        local i=CatFns.nid(t)
        if not i then
            return nil
        end
        table.insert(ids,i)
    end
    if #ids==1 or #ids==6 then
        return ids
    end
    return nil
end

function CatFns.modelsky(id)
    local ok,res=pcall(function()
        return game:GetObjects("rbxassetid://"..id)
    end)
    if not ok or type(res)~="table" then
        return nil
    end
    for _,o in ipairs(res) do
        local sk=o:IsA("Sky") and o or o:FindFirstChildWhichIsA("Sky",true)
        if sk then
            local ids={}
            for i,f in ipairs(faces) do
                local n=CatFns.nid(sk[f])
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

local lpresets={
    {n="Day",t=13.5,b=2,amb=Color3.fromRGB(150,158,175),oamb=Color3.fromRGB(180,188,205),fogc=Color3.fromRGB(196,206,220),exp=0,dens=0.32,glare=0,color=Color3.fromRGB(199,170,107)},
    {n="Golden",t=17.4,b=2.4,amb=Color3.fromRGB(178,142,104),oamb=Color3.fromRGB(214,168,110),fogc=Color3.fromRGB(224,168,110),exp=0.1,dens=0.36,glare=1.4,color=Color3.fromRGB(226,168,94)},
    {n="Sunset",t=19.1,b=1.6,amb=Color3.fromRGB(120,88,96),oamb=Color3.fromRGB(196,120,104),fogc=Color3.fromRGB(186,116,104),exp=-0.05,dens=0.44,glare=2.6,color=Color3.fromRGB(206,124,92)},
    {n="Night",t=0.5,b=0.6,amb=Color3.fromRGB(28,32,52),oamb=Color3.fromRGB(38,44,70),fogc=Color3.fromRGB(24,28,44),exp=-0.15,dens=0.4,glare=0,color=Color3.fromRGB(44,48,72)},
    {n="Dawn",t=6.6,b=1.4,amb=Color3.fromRGB(128,122,142),oamb=Color3.fromRGB(166,142,150),fogc=Color3.fromRGB(170,150,152),exp=0,dens=0.5,glare=1.1,color=Color3.fromRGB(186,158,156)},
    {n="Overcast",t=10,b=1.8,amb=Color3.fromRGB(168,170,176),oamb=Color3.fromRGB(186,188,194),fogc=Color3.fromRGB(190,192,198),exp=-0.05,dens=0.62,glare=0,color=Color3.fromRGB(186,188,196)}
}

function CatFns.applylpreset(p)
    CatFns.setclock(p.t)
    lt.Brightness=p.b
    lt.Ambient=p.amb
    lt.OutdoorAmbient=p.oamb
    lt.FogColor=p.fogc
    lt.ExposureCompensation=p.exp
    local a=CatFns.catatm()
    a.Density=p.dens
    a.Glare=p.glare
    a.Haze=0
    a.Color=p.color
    V.light.ClockTime=p.t
    V.light.Brightness=p.b
    V.light.Ambient=CatFns.tohex(p.amb)
    V.light.OutdoorAmbient=CatFns.tohex(p.oamb)
    V.light.FogColor=CatFns.tohex(p.fogc)
    V.light.ExposureCompensation=p.exp
    V.atm.Density=p.dens
    V.atm.Glare=p.glare
    V.atm.Haze=0
    V.atm.Color=CatFns.tohex(p.color)
    s.lck=false
    CatFns.lockclock()
    CatFns.later()
end

local vg
local vcol=Color3.new(0,0,0)
local vint=0.6

function CatFns.vfix()
    if not (vg and vg.Parent) then
        return
    end
    for _,f in ipairs(vg:GetChildren()) do
        if f:IsA("Frame") then
            f.BackgroundColor3=vcol
            f.UIGradient.Transparency=NumberSequence.new(1-vint,1)
        end
    end
end

function CatFns.vset(on)
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
                {UDim2.new(1,0,0.3,0),UDim2.new(0,0,0,0),90},
                {UDim2.new(1,0,0.3,0),UDim2.new(0,0,0.7,0),270},
                {UDim2.new(0.22,0,1,0),UDim2.new(0,0,0,0),0},
                {UDim2.new(0.22,0,1,0),UDim2.new(0,0,0.78,0),180}
            }
            for _,d in ipairs(defs) do
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
        CatFns.vfix()
        vg.Enabled=true
    elseif vg then
        vg.Enabled=false
    end
end

local mb
local lv

function CatFns.setmb(on)
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
    {n="Cinematic",fx={bloom={Intensity=0.5,Size=36,Threshold=0.9},rays={Intensity=0.15,Spread=0.8}},cc={-0.02,0.2,-0.05,Color3.fromRGB(255,235,215)}},
    {n="Dreamy",fx={bloom={Intensity=0.9,Size=56,Threshold=0.7},dof={FarIntensity=0.25,FocusDistance=60,InFocusRadius=40,NearIntensity=0}},cc={0.05,-0.1,0.3,Color3.fromRGB(255,220,245)}},
    {n="Cyber Glow",fx={bloom={Intensity=1.2,Size=40,Threshold=0.6}},cc={0,0.25,0.5,Color3.fromRGB(200,220,255)}},
    {n="Clean",fx={},cc={0,0.05,0.15,Color3.new(1,1,1)}}
}

function CatFns.lookapply(d)
    for k in pairs(ed) do
        if d.fx[k] then
            CatFns.seteffect(k,true)
            for p,v in pairs(d.fx[k]) do
                CatFns.setev(k,p,v)
            end
        else
            CatFns.seteffect(k,false)
        end
    end
    CatFns.setcc("Brightness",d.cc[1])
    CatFns.setcc("Contrast",d.cc[2])
    CatFns.setcc("Saturation",d.cc[3])
    CatFns.setcc("TintColor",d.cc[4])
    CatFns.notify(d.n.." applied")
end

s.xreset=function()
    if vg then
        vg:Destroy()
        vg=nil
    end
    s.vig=false
    CatFns.setmb(false)
end

pages.performance=function()
    CatFns.section("PERFORMANCE")
    local lagset=CatFns.toggle("Fix Lag",s.lag,CatFns.setlag)
    local function sync()
        s.lag=s.shd and s.ptc and s.pst and s.mat and s.wtr and s.low
        lagset(s.lag)
    end
    for _,d in ipairs({{"Show FPS","fps"},{"Show Ping","png"},{"Show Memory","mem"}}) do
        local d=d
        CatFns.toggle(d[1],s[d[2]],function(v)
            s[d[2]]=v
            sl[d[2]].Visible=v
            CatFns.later()
        end)
    end
    CatFns.section("GRAPHICS")
    CatFns.toggle("Remove Shadows",s.shd,function(v)
        s.shd=v
        CatFns.shadows()
        sync()
    end)
    for _,d in ipairs({{"Remove Particles","ptc","e"},{"Remove Textures","tex","d"},{"Remove Post Effects","pst","p"},{"Smooth Materials","mat","m"}}) do
        local d=d
        CatFns.toggle(d[1],s[d[2]],function(v)
            s[d[2]]=v
            CatFns.setkind(d[3])
            sync()
        end)
    end
    CatFns.toggle("Simple Water",s.wtr,function(v)
        s.wtr=v
        CatFns.water()
        sync()
    end)
    CatFns.toggle("Low Quality Level",s.low,function(v)
        s.low=v
        CatFns.quality()
        sync()
    end)
    CatFns.btn("Restore Graphics","Restore",function()
        CatFns.rgraphics()
        refresh()
    end)
end

pages.sky=function()
    CatFns.section("SKY PACK")
    CatFns.btn("Random Sky","Random",CatFns.randomsky)
    for _,e in ipairs(bskies) do
        local e=e
        CatFns.row(e.Name,{{t="Apply",f=function()
            CatFns.applycustom(e)
        end}})
    end
    CatFns.section("SKY FROM LINK")
    CatFns.form({"Name","Sky link or ID (1 or 6, separated by comma)"},"Add",function(v)
        local ids=CatFns.pids(v[2])
        if not ids then
            CatFns.notify("Need a valid link or ID")
            return
        end
        local n=CatFns.trim(v[1])
        task.spawn(function()
            if #ids==1 then
                CatFns.notify("Loading sky...")
                ids=CatFns.modelsky(ids[1]) or ids
            end
            if n=="" then
                n="Sky "..(#sv.skies+1)
            end
            local e={Name=n,Ids=ids}
            table.insert(sv.skies,e)
            CatFns.later()
            CatFns.applycustom(e)
        end)
    end)
    for _,e in ipairs(sv.skies) do
        local e=e
        CatFns.row(e.Name,{
            {t="Apply",f=function()
                CatFns.applycustom(e)
            end},
            {t="X",w=34,c=col.red,f=function()
                for k,x in ipairs(sv.skies) do
                    if x==e then
                        table.remove(sv.skies,k)
                        break
                    end
                end
                CatFns.later()
                refresh()
            end}
        })
    end
    CatFns.section("LOCAL SKIES")
    if not lfs then
        CatFns.row("Executor has no local file support",{})
    else
        if not lsk then
            CatFns.scanlocal()
        end
        CatFns.btn("Rescan CatSkies folder","Rescan",function()
            CatFns.scanlocal()
            refresh()
        end)
        if #lsk==0 then
            CatFns.row("Put sky folders in the CatSkies folder",{})
        end
        for _,e in ipairs(lsk) do
            CatFns.btn(e.n,"Apply",function()
                CatFns.applylocal(e)
            end)
        end
    end
    CatFns.section("SKY SETTINGS")
    local sk=CatFns.cursky()
    CatFns.slider("Star Count",0,5000,CatFns.rp(sk,"StarCount",3000),function(v)
        CatFns.catsky().StarCount=v
        V.sky.StarCount=v
        CatFns.later()
    end,100)
    CatFns.slider("Sun Size",0,60,CatFns.rp(sk,"SunAngularSize",21),function(v)
        CatFns.catsky().SunAngularSize=v
        V.sky.SunAngularSize=v
        CatFns.later()
    end,1)
    CatFns.slider("Moon Size",0,60,CatFns.rp(sk,"MoonAngularSize",11),function(v)
        CatFns.catsky().MoonAngularSize=v
        V.sky.MoonAngularSize=v
        CatFns.later()
    end,1)
    CatFns.toggle("Sun and Moon",CatFns.rp(sk,"CelestialBodiesShown",true),function(v)
        CatFns.catsky().CelestialBodiesShown=v
        V.sky.CelestialBodiesShown=v
        CatFns.later()
    end)
    CatFns.section("ANIMATION")
    CatFns.slider("Transition Time",0,5,s.gl,function(v)
        s.gl=v
    end,0.1)
    CatFns.slider("Sky Rotation",-20,20,s.spin,function(v)
        s.spin=v
        if v~=0 then
            CatFns.catsky()
        end
    end,0.5)
    CatFns.slider("Time Flow",-1,1,s.flow,function(v)
        s.flow=v
    end,0.05)
    CatFns.slider("Color Cycle",0,1,s.cyc,function(v)
        local was=s.cyc
        s.cyc=v
        if v~=0 and was==0 then
            cycbase={Ambient=lt.Ambient,OutdoorAmbient=lt.OutdoorAmbient,FogColor=lt.FogColor}
        elseif v==0 and was~=0 and cycbase then
            pcall(function()
                lt.Ambient=cycbase.Ambient
                lt.OutdoorAmbient=cycbase.OutdoorAmbient
                lt.FogColor=cycbase.FogColor
            end)
            cycbase=nil
        end
    end,0.05)
    if skysnap then
        CatFns.btn("Undo last sky change","Undo",function()
            CatFns.applycustom(skysnap)
        end,col.yellow)
    end
    CatFns.btn("Default Sky","Restore",function()
        CatFns.rsky()
        refresh()
    end)
end

pages.lighting=function()
    CatFns.section("LIGHTING PRESETS")
    for _,p in ipairs(lpresets) do
        local p=p
        CatFns.row(p.n,{{t="Apply",f=function()
            CatFns.applylpreset(p)
            refresh()
        end}})
    end
    CatFns.section("TIME")
    CatFns.slider("Time of Day",0,24,lt.ClockTime,CatFns.setclock,0.1)
    CatFns.toggle("Lock Time",s.lck,function(v)
        s.lck=v
        s.lckt=lt.ClockTime
        CatFns.lockclock()
    end)
    CatFns.section("LIGHT")
    CatFns.slider("Brightness",0,6,lt.Brightness,function(v)
        lt.Brightness=v
        V.light.Brightness=v
        CatFns.later()
    end,0.1)
    CatFns.slider("Exposure",-3,3,lt.ExposureCompensation,function(v)
        lt.ExposureCompensation=v
        V.light.ExposureCompensation=v
        CatFns.later()
    end,0.1)
    for _,d in ipairs({{"Diffuse Scale","EnvironmentDiffuseScale"},{"Specular Scale","EnvironmentSpecularScale"}}) do
        local d=d
        CatFns.slider(d[1],0,1,CatFns.rp(lt,d[2],1),function(v)
            pcall(function()
                lt[d[2]]=v
            end)
            V.light[d[2]]=v
            CatFns.later()
        end,0.05)
    end
    CatFns.toggle("Fog",s.fog,CatFns.fog)
    CatFns.toggle("Fullbright",s.fb,function(v)
        if s.setfb then
            s.setfb(v)
        end
    end)
    CatFns.section("COLORS")
    for _,d in ipairs({{"Ambient","Ambient"},{"Outdoor Ambient","OutdoorAmbient"},{"Fog Color","FogColor"},{"Shift Top","ColorShift_Top"},{"Shift Bottom","ColorShift_Bottom"}}) do
        local d=d
        CatFns.colorin(d[1],lt[d[2]],function(cr)
            lt[d[2]]=cr
            V.light[d[2]]=CatFns.tohex(cr)
            CatFns.later()
        end)
    end
    CatFns.section("ATMOSPHERE")
    local a=CatFns.curatm()
    for _,d in ipairs({{"Density",0,1,0.395,0.01},{"Offset",0,1,0,0.01},{"Haze",0,10,0,0.1},{"Glare",0,10,0,0.1}}) do
        local d=d
        CatFns.slider(d[1],d[2],d[3],CatFns.rp(a,d[1],d[4]),function(v)
            CatFns.setatm(d[1],v)
        end,d[5])
    end
    CatFns.colorin("Atmosphere Color",CatFns.rp(a,"Color",Color3.fromRGB(199,170,107)),function(cr)
        CatFns.setatm("Color",cr)
    end)
    CatFns.colorin("Atmosphere Decay",CatFns.rp(a,"Decay",Color3.fromRGB(92,60,13)),function(cr)
        CatFns.setatm("Decay",cr)
    end)
    CatFns.btn("Restore Lighting","Restore",function()
        CatFns.rlight()
        refresh()
    end)
end

pages.effects=function()
    local cgp={
        {"Vivid",0,0.1,0.5,Color3.fromRGB(255,255,255)},
        {"Cinematic",-0.03,0.25,-0.1,Color3.fromRGB(255,235,215)},
        {"Dreamy",0.05,-0.1,0.3,Color3.fromRGB(255,220,245)},
        {"Cyber",0,0.2,0.4,Color3.fromRGB(200,220,255)},
        {"Noir",0,0.3,-1,Color3.fromRGB(255,255,255)}
    }
    local function fx(sec,k,lbl,list)
        CatFns.section(sec)
        CatFns.toggle(lbl,s[k],function(v)
            CatFns.seteffect(k,v)
        end)
        for _,d in ipairs(list) do
            local d=d
            CatFns.slider(d[1],d[2],d[3],ev[k][d[4]],function(v)
                CatFns.setev(k,d[4],v)
            end,d[5])
        end
    end
    fx("BLOOM","bloom","Bloom",{{"Bloom Intensity",0,3,"Intensity",0.05},{"Bloom Size",0,56,"Size",1},{"Bloom Threshold",0,4,"Threshold",0.05}})
    fx("BLUR","blur","Blur",{{"Blur Size",0,56,"Size",1}})
    fx("SUN RAYS","rays","Sun Rays",{{"Rays Intensity",0,1,"Intensity",0.01},{"Rays Spread",0,1,"Spread",0.01}})
    fx("DEPTH OF FIELD","dof","Depth of Field",{{"Far Intensity",0,1,"FarIntensity",0.01},{"Focus Distance",0,200,"FocusDistance",1},{"Focus Radius",0,100,"InFocusRadius",1}})
    CatFns.section("COLOR")
    for _,d in ipairs({{"Color Brightness",-0.5,0.5,"Brightness",0.01},{"Contrast",-1,1,"Contrast",0.05},{"Saturation",-1,1,"Saturation",0.05}}) do
        local d=d
        CatFns.slider(d[1],d[2],d[3],cv[d[4]],function(v)
            CatFns.setcc(d[4],v)
        end,d[5])
    end
    CatFns.colorin("Tint",cv.TintColor,function(cr)
        CatFns.setcc("TintColor",cr)
    end)
    CatFns.section("COLOR PRESETS")
    for _,d in ipairs(cgp) do
        local d=d
        CatFns.btn(d[1],"Apply",function()
            CatFns.setcc("Brightness",d[2])
            CatFns.setcc("Contrast",d[3])
            CatFns.setcc("Saturation",d[4])
            CatFns.setcc("TintColor",d[5])
            CatFns.notify(d[1].." applied")
            refresh()
        end)
    end
    CatFns.section("LOOK PRESETS")
    for _,d in ipairs(looks) do
        local d=d
        CatFns.btn(d.n,"Apply",function()
            CatFns.lookapply(d)
            refresh()
        end)
    end
    CatFns.section("VIGNETTE")
    CatFns.toggle("Vignette",s.vig,CatFns.vset)
    CatFns.slider("Vignette Strength",0,1,vint,function(v)
        vint=v
        CatFns.vfix()
    end,0.05)
    CatFns.colorin("Vignette Color",vcol,function(cr)
        vcol=cr
        CatFns.vfix()
    end)
    CatFns.section("MOTION BLUR")
    CatFns.toggle("Motion Blur",s.mb,CatFns.setmb)
    CatFns.slider("Blur Strength",0.2,3,s.mbi,function(v)
        s.mbi=v
    end,0.1)
    CatFns.section("CAMERA")
    CatFns.toggle("Custom FOV",s.cfov,CatFns.setfov)
    CatFns.slider("Field of View",30,120,s.fov,function(v)
        s.fov=v
    end,1)
    CatFns.btn("Reset Effects","Reset",function()
        CatFns.reffects()
        refresh()
    end)
end

local fitems={}
for _,i in ipairs(Enum.Font:GetEnumItems()) do
    if i.Name~="Legacy" and i.Name~="Unknown" then
        table.insert(fitems,i)
    end
end
table.sort(fitems,function(a,b)
    return a.Name<b.Name
end)

function CatFns.fontloads(face)
    local done=false
    local bad=false
    local missing=false
    local res=nil
    task.spawn(function()
        local ok,r=pcall(function()
            local p=Instance.new("GetTextBoundsParams")
            p.Text="Test"
            p.Size=14
            p.Width=1000
            -- [FIX] GetTextBoundsParams dung thuoc tinh "Font" (khong phai "FontFace"), nen truoc day
            -- viec kiem tra font luon bi bo qua.
            if not pcall(function()
                p.Font=face
            end) then
                error("Font API unavailable")
            end
            return tsv:GetTextBoundsAsync(p)
        end)
        if not ok then
            local msg=tostring(r)
            if msg:find("GetTextBoundsParams",1,true) or msg:find("API unavailable",1,true) then
                missing=true
            end
        end
        bad=not ok
        res=r
        done=true
    end)
    local t=0
    while not done and t<0.8 do
        task.wait(0.05)
        t=t+0.05
    end
    if not done then
        return true
    end
    if missing then
        return true
    end
    return not bad and typeof(res)=="Vector2"
end

function CatFns.pfont(v,check)
    v=CatFns.trim(v)
    if v=="" then
        return nil
    end
    for _,i in ipairs(fitems) do
        if i.Name:lower()==v:lower() then
            return {e=i,k="Enum:"..i.Name}
        end
    end
    local id=CatFns.nid(v)
    local fam=id and ("rbxassetid://"..id) or (v:find("^rbxasset") and v)
    if not fam then
        return nil
    end
    local ok,face=pcall(Font.new,fam,Enum.FontWeight.Regular,Enum.FontStyle.Normal)
    if not ok then
        return nil
    end
    if check and not CatFns.fontloads(face) then
        return nil
    end
    return {f=face,k="Face:"..fam}
end

function CatFns.skey(k)
    return CatFns.pfont(k:match("^%a+:(.+)$") or "")
end

local fbs
s.setfb=function(on)
    local was=s.fb
    s.fb=on
    if on then
        if not was then
            fbs={}
            for _,p in ipairs({"Brightness","Ambient","OutdoorAmbient","FogStart","FogEnd"}) do
                fbs[p]=lt[p]
            end
        end
        lt.Brightness=2.5
        lt.Ambient=Color3.fromRGB(200,200,200)
        lt.OutdoorAmbient=Color3.fromRGB(200,200,200)
        lt.FogStart=100000
        lt.FogEnd=100000
    elseif was and fbs then
        for p,v in pairs(fbs) do
            pcall(function()
                lt[p]=v
            end)
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
                ColorSequenceKeypoint.new(0,Color3.fromRGB(255,80,120)),
                ColorSequenceKeypoint.new(0.17,Color3.fromRGB(255,200,70)),
                ColorSequenceKeypoint.new(0.33,Color3.fromRGB(90,255,140)),
                ColorSequenceKeypoint.new(0.5,Color3.fromRGB(70,220,255)),
                ColorSequenceKeypoint.new(0.67,Color3.fromRGB(110,120,255)),
                ColorSequenceKeypoint.new(0.83,Color3.fromRGB(230,90,255)),
                ColorSequenceKeypoint.new(1,Color3.fromRGB(255,80,120))
            })
            rgg.Parent=sk
        end
        sk.Transparency=0.15
    else
        if rgg then
            rgg:Destroy()
            rgg=nil
        end
        if not cf.grad then
            sk.Transparency=0.5
        end
    end
end

local rgg2
function CatFns.gradseq(c)
    local h=math.round(math.clamp(c.R,0,1)*255)
    local g=math.round(math.clamp(c.G,0,1)*255)
    local b=math.round(math.clamp(c.B,0,1)*255)
    return ColorSequence.new({
        ColorSequenceKeypoint.new(0,Color3.fromRGB(h,g,b)),
        ColorSequenceKeypoint.new(0.5,Color3.fromRGB(math.min(255,h+70),math.min(255,g+70),math.min(255,b+70))),
        ColorSequenceKeypoint.new(1,Color3.fromRGB(h,g,b))
    })
end

s.setgrad=function(on)
    local sk=main:FindFirstChildOfClass("UIStroke")
    if not sk then
        return
    end
    if on then
        if not (rgg2 and rgg2.Parent) then
            rgg2=Instance.new("UIGradient")
            rgg2.Parent=sk
        end
        rgg2.Color=CatFns.gradseq(cf.theme)
        sk.Transparency=0.3
    else
        if rgg2 then
            rgg2:Destroy()
            rgg2=nil
        end
        if not cf.rgb then
            sk.Transparency=0.5
        end
    end
end

local sc=type(cf.sl)=="table" and cf.sl or {}

function CatFns.nm(v,d,a,b)
    v=tonumber(v)
    if not v then
        return d
    end
    return math.clamp(v,a,b)
end

local C={
    on=sc.on==true,
    m=math.floor(CatFns.nm(sc.m,1,1,2)),
    st=math.floor(CatFns.nm(sc.st,1,1,7)),
    sz=CatFns.nm(sc.sz,10,4,40),
    th=CatFns.nm(sc.th,2,1,6),
    tr=CatFns.nm(sc.tr,0,0,0.9),
    c={255,255,255},
    ol=sc.ol~=false,
    al=sc.al==true,
    rb=sc.rb==true,
    fl=sc.fl==true,
    off=CatFns.nm(sc.off,1.75,0,4)
}

if type(sc.c)=="table" then
    C.c={math.floor(CatFns.nm(sc.c[1],255,0,255)),math.floor(CatFns.nm(sc.c[2],255,0,255)),math.floor(CatFns.nm(sc.c[3],255,0,255))}
end

cf.sl=C
local snames={"Dot","Ring","Dot + Ring","Cross","Crosshair","Plus","Diamond"}
local canfile=HASFS and FIO.customasset~=nil
local curl
local ours={}
local flon=false
local cg
local box
local els={}
local hid=false
local ocur=true
local pvt=0

function CatFns.paint(c)
    for _,e in ipairs(els) do
        if e.k=="f" then
            e.o.BackgroundColor3=c
        else
            e.o.Color=c
        end
    end
end

function CatFns.fr(w,h,x,y,round)
    local f=Instance.new("Frame")
    f.AnchorPoint=Vector2.new(0.5,0.5)
    f.Size=UDim2.fromOffset(w,h)
    f.Position=UDim2.fromOffset(x,y)
    f.BorderSizePixel=0
    f.BackgroundTransparency=C.tr
    if round then
        local u=Instance.new("UICorner")
        u.CornerRadius=UDim.new(1,0)
        u.Parent=f
    end
    if C.ol then
        local k=Instance.new("UIStroke")
        k.Color=Color3.new(0,0,0)
        k.Thickness=1
        k.Transparency=math.min(1,C.tr+0.2)
        k.Parent=f
    end
    f.Parent=box
    table.insert(els,{k="f",o=f})
end

function CatFns.ring(d,round)
    local f=Instance.new("Frame")
    f.AnchorPoint=Vector2.new(0.5,0.5)
    f.Size=UDim2.fromOffset(d,d)
    f.Position=UDim2.fromOffset(0,0)
    f.BackgroundTransparency=1
    f.BorderSizePixel=0
    if round then
        local u=Instance.new("UICorner")
        u.CornerRadius=UDim.new(1,0)
        u.Parent=f
    end
    local k=Instance.new("UIStroke")
    k.Thickness=C.th
    k.Transparency=C.tr
    k.Parent=f
    f.Parent=box
    table.insert(els,{k="s",o=k})
    return f
end

function CatFns.build()
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
    box.Position=UDim2.fromScale(0.5,0.5)
    box.Size=UDim2.fromOffset(0,0)
    box.BackgroundTransparency=1
    box.Parent=cg
    local sz=C.sz
    local th=C.th
    local st=C.st
    if st==1 then
        local d=math.max(2,sz*0.5)
        CatFns.fr(d,d,0,0,true)
    elseif st==2 then
        CatFns.ring(sz,true)
    elseif st==3 then
        CatFns.ring(sz,true)
        CatFns.fr(th+2,th+2,0,0,true)
    elseif st==4 or st==5 then
        local g=sz*0.35
        local l=sz*0.65
        CatFns.fr(l,th,-(g+l/2),0)
        CatFns.fr(l,th,g+l/2,0)
        CatFns.fr(th,l,0,-(g+l/2))
        CatFns.fr(th,l,0,g+l/2)
        if st==5 then
            CatFns.fr(th+1,th+1,0,0,true)
        end
    elseif st==6 then
        CatFns.fr(sz*1.6,th,0,0)
        CatFns.fr(th,sz*1.6,0,0)
    else
        CatFns.ring(sz*0.8,false).Rotation=45
    end
    CatFns.paint(Color3.fromRGB(C.c[1],C.c[2],C.c[3]))
    cg.Enabled=false
    cg.Parent=gui.Parent
end

local crct
function CatFns.crc32(str)
    if not crct then
        crct={}
        for i=0,255 do
            local x=i
            for _=1,8 do
                if bit32.band(x,1)==1 then
                    x=bit32.bxor(bit32.rshift(x,1),0xEDB88320)
                else
                    x=bit32.rshift(x,1)
                end
            end
            crct[i]=x
        end
    end
    local x=0xFFFFFFFF
    for i=1,#str do
        x=bit32.bxor(crct[bit32.band(bit32.bxor(x,string.byte(str,i)),255)],bit32.rshift(x,8))
    end
    return bit32.bxor(x,0xFFFFFFFF)
end

function CatFns.u32(n)
    return string.char(bit32.band(bit32.rshift(n,24),255),bit32.band(bit32.rshift(n,16),255),bit32.band(bit32.rshift(n,8),255),bit32.band(n,255))
end

function CatFns.chunk(t,d)
    return CatFns.u32(#d)..t..d..CatFns.u32(CatFns.crc32(t..d))
end

function CatFns.makepng()
    local N=96
    local half=N/2
    local SS=3
    local sz=C.sz
    local th=C.th
    local st=C.st
    local fns={}
    local function circ(r)
        table.insert(fns,function(x,y)
            return math.sqrt(x*x+y*y)-r
        end)
    end
    local function annulus(rm,w)
        table.insert(fns,function(x,y)
            return math.abs(math.sqrt(x*x+y*y)-rm)-w/2
        end)
    end
    local function bx(cx,cy,hx,hy)
        table.insert(fns,function(x,y)
            local qx=math.abs(x-cx)-hx
            local qy=math.abs(y-cy)-hy
            local ox=math.max(qx,0)
            local oy=math.max(qy,0)
            return math.sqrt(ox*ox+oy*oy)+math.min(math.max(qx,qy),0)
        end)
    end
    if st==1 then
        circ(math.max(1,sz*0.25))
    elseif st==2 then
        annulus(sz/2-th/2,th)
    elseif st==3 then
        annulus(sz/2-th/2,th)
        circ((th+2)/2)
    elseif st==4 or st==5 then
        local g=sz*0.35
        local l=sz*0.65
        bx(-(g+l/2),0,l/2,th/2)
        bx(g+l/2,0,l/2,th/2)
        bx(0,-(g+l/2),th/2,l/2)
        bx(0,g+l/2,th/2,l/2)
        if st==5 then
            circ((th+1)/2)
        end
    elseif st==6 then
        bx(0,0,sz*0.8,th/2)
        bx(0,0,th/2,sz*0.8)
    else
        table.insert(fns,function(x,y)
            return math.abs((math.abs(x)+math.abs(y))*0.70711-sz*0.4)-th/2
        end)
    end
    local base=1-C.tr
    local r0=C.c[1]
    local g0=C.c[2]
    local b0=C.c[3]
    local rows={}
    for py=0,N-1 do
        local row={string.char(0)}
        for px=0,N-1 do
            local sa=0
            local sr=0
            local sg=0
            local sb=0
            for iy=0,SS-1 do
                for ix=0,SS-1 do
                    local x=px+(ix+0.5)/SS-half
                    local y=py+(iy+0.5)/SS-half
                    local d=1e9
                    for _,f in ipairs(fns) do
                        local v=f(x,y)
                        if v<d then
                            d=v
                        end
                    end
                    if d<=0 then
                        sa=sa+base
                        sr=sr+r0*base
                        sg=sg+g0*base
                        sb=sb+b0*base
                    elseif C.ol and d<=1 then
                        sa=sa+base*0.85
                    end
                end
            end
            if sa>0 then
                row[#row+1]=string.char(math.floor(sr/sa+0.5),math.floor(sg/sa+0.5),math.floor(sb/sa+0.5),math.floor(sa/(SS*SS)*255+0.5))
            else
                row[#row+1]=string.char(0,0,0,0)
            end
        end
        rows[#rows+1]=table.concat(row)
    end
    local raw=table.concat(rows)
    local a=1
    local b=0
    for i=1,#raw do
        a=(a+string.byte(raw,i))%65521
        b=(b+a)%65521
    end
    local ln=#raw
    local nl=65535-ln
    -- [FIX LOI NGHIEM TRONG] LEN/NLEN cua khoi deflate "stored" phai la little-endian (byte thap truoc).
    -- Truoc day viet big-endian nen PNG bi hong va con tro he thong khong hien thi.
    local z=string.char(120,1,1,ln%256,math.floor(ln/256),nl%256,math.floor(nl/256))..raw..CatFns.u32(b*65536+a)
    return string.char(137,80,78,71,13,10,26,10)..CatFns.chunk("IHDR",CatFns.u32(N)..CatFns.u32(N)..string.char(8,6,0,0,0))..CatFns.chunk("IDAT",z)..CatFns.chunk("IEND","")
end

function CatFns.genurl()
    if not canfile then
        return nil
    end
    local key=C.st.."_"..C.sz.."_"..C.th.."_"..math.floor(C.tr*100+0.5).."_"..C.c[1].."_"..C.c[2].."_"..C.c[3].."_"..(C.ol and 1 or 0)
    -- v2: doi ten file de bo qua cac PNG hong do ban cu da tao ra
    local path="CatCursor2_"..key..".png"
    local ok,url=pcall(function()
        if not FIO.isfile(path) then
            FIO.write(path,CatFns.makepng())
        end
        return FIO.customasset(path)
    end)
    if ok and type(url)=="string" and url~="" then
        return url
    end
    return nil
end

local cicreated=false
local ciorig=nil

function CatFns.setci(url)
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
        elseif not cicreated and ciorig==nil then
            ciorig=v.Value
        end
        v.Value=url
    elseif v then
        if cicreated then
            v:Destroy()
        elseif ciorig~=nil then
            v.Value=ciorig
        end
        cicreated=false
        ciorig=nil
    end
end

function CatFns.sysapply()
    if C.on and C.m==1 and curl then
        CatFns.setci(curl)
    else
        CatFns.setci(nil)
    end
end

local gtk=0
function CatFns.regen(now)
    local function go()
        curl=nil
        if C.on and C.m==1 then
            curl=CatFns.genurl()
            if curl then
                ours[curl]=true
            end
        end
        CatFns.sysapply()
        if cg then
            CatFns.build()
        end
        CatFns.later()
    end
    if now then
        go()
        return
    end
    gtk=gtk+1
    local id=gtk
    task.delay(0.35,function()
        if id==gtk and not dead then
            go()
        end
    end)
end

function CatFns.showcur()
    if hid then
        hid=false
        uis.MouseIconEnabled=ocur
    end
end

function CatFns.flset(on)
    local was=flon
    flon=on
    if on or not was then
        return
    end
    local ch=lp.Character
    local hum=ch and ch:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.AutoRotate=true
        hum.CameraOffset=Vector3.new(0,0,0)
    end
    if uis.MouseBehavior==Enum.MouseBehavior.LockCenter then
        uis.MouseBehavior=Enum.MouseBehavior.Default
    end
end

function CatFns.forceupd()
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
    if hum and root and cam and hum.Health>0 and not hum.Sit then
        hum.AutoRotate=false
        hum.CameraOffset=hum.CameraOffset:Lerp(Vector3.new(C.off,0,0),0.2)
        local lv2=cam.CFrame.LookVector
        root.CFrame=CFrame.new(root.Position)*CFrame.Angles(0,math.atan2(-lv2.X,-lv2.Z),0)
    end
end

function CatFns.crossupd()
    CatFns.forceupd()
    local locked=uis.MouseBehavior==Enum.MouseBehavior.LockCenter
    local sys=C.on and C.m==1 and curl~=nil
    local ov=C.on and not sys
    if sys then
        if locked and (flon or uis.MouseIcon~="") then
            if uis.MouseIcon~=curl then
                uis.MouseIcon=curl
            end
        end
    end
    if not locked and ours[uis.MouseIcon] and os.clock()>pvt then
        uis.MouseIcon=""
    end
    local prev=not curl and os.clock()<pvt
    local want=prev or (ov and (locked or C.al) and (locked or not main.Visible))
    if want then
        if not cg then
            CatFns.build()
        end
        if not cg.Enabled then
            cg.Enabled=true
        end
        if C.rb then
            CatFns.paint(Color3.fromHSV((os.clock()*0.25)%1,0.8,1))
        end
    elseif cg and cg.Enabled then
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
        CatFns.showcur()
    end
end

s.kill=function()
    CatFns.flset(false)
    CatFns.setci(nil)
    if cg then
        cg:Destroy()
        cg=nil
    end
    CatFns.showcur()
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
    C.c={255,255,255}
    C.ol=true
    C.al=false
    C.rb=false
    C.fl=false
    C.off=1.75
    s.kill()
end

function CatFns.resetcfg()
    cf.theme=dtheme
    cf.size=1
    cf.tr=0.12
    cf.key=Enum.KeyCode.RightShift
    cf.notif=true
    cf.btn=true
    cf.rgb=false
    cf.grad=false
    cf.safe=false
    cf.avatar=DEFAVATAR
    cf.dc=DEFDISCORD
    cf.sl={}
    fs.spec=nil
    fs.key=nil
    fs.game=true
    fs.menu=true
    sv.active=nil
    sv.sky=nil
    s.gl=1.5
    table.clear(sv.skies)
    table.clear(sv.fonts)
    for _,t in pairs(V) do
        if type(t)=="table" then
            for k in pairs(t) do
                t[k]=nil
            end
        end
    end
end

function CatFns.synccross()
    local s2=type(cf.sl)=="table" and cf.sl or {}
    C.on=s2.on==true
    C.m=math.floor(CatFns.nm(s2.m,1,1,2))
    C.st=math.floor(CatFns.nm(s2.st,1,1,7))
    C.sz=CatFns.nm(s2.sz,10,4,40)
    C.th=CatFns.nm(s2.th,2,1,6)
    C.tr=CatFns.nm(s2.tr,0,0,0.9)
    C.ol=s2.ol~=false
    C.al=s2.al==true
    C.rb=s2.rb==true
    C.fl=s2.fl==true
    C.off=CatFns.nm(s2.off,1.75,0,4)
    C.c={255,255,255}
    if type(s2.c)=="table" then
        C.c={math.floor(CatFns.nm(s2.c[1],255,0,255)),math.floor(CatFns.nm(s2.c[2],255,0,255)),math.floor(CatFns.nm(s2.c[3],255,0,255))}
    end
    cf.sl=C
end

function CatFns.applyvisuals()
    if cf.safe then
        return
    end
    for k,v in pairs(V.light) do
        if type(v)=="number" then
            pcall(function()
                lt[k]=v
            end)
        elseif type(v)=="string" then
            local c=CatFns.hex(v)
            if c then
                pcall(function()
                    lt[k]=c
                end)
            end
        end
    end
    -- dong bo cong tac Fog voi gia tri da load
    s.fog=not (lt.FogStart>=100000 and lt.FogEnd>=100000 and V.light.FogStart~=nil and V.light.FogStart>=100000)
    if next(V.atm)~=nil then
        local a=CatFns.catatm()
        for k,v in pairs(V.atm) do
            if type(v)=="number" then
                pcall(function()
                    a[k]=v
                end)
            elseif type(v)=="string" then
                local c=CatFns.hex(v)
                if c then
                    pcall(function()
                        a[k]=c
                    end)
                end
            end
        end
    end
    -- [UPGRADE] khoi phuc sky da chon lan truoc
    if sv.sky and type(sv.sky.Ids)=="table" then
        local sk=CatFns.catsky(true)
        for i,f in ipairs(faces) do
            pcall(function()
                sk[f]="rbxassetid://"..(sv.sky.Ids[i] or sv.sky.Ids[1])
            end)
        end
    end
    if next(V.sky)~=nil then
        local sk=CatFns.catsky()
        for k,v in pairs(V.sky) do
            pcall(function()
                sk[k]=v
            end)
        end
    end
    -- [FIX] chi bat hieu ung khi _on ~= 0, va chi nhan thuoc tinh hop le
    for k,t in pairs(V.fx) do
        if type(t)=="table" and ed[k] then
            for p,v in pairs(t) do
                if ed[k].d[p]~=nil and type(v)=="number" then
                    ev[k][p]=v
                end
            end
            if t._on~=0 then
                CatFns.seteffect(k,true)
            end
        end
    end
    if next(V.cc)~=nil then
        for p,v in pairs(V.cc) do
            if type(v)=="number" then
                CatFns.setcc(p,v)
            elseif type(v)=="string" then
                local c=CatFns.hex(v)
                if c then
                    CatFns.setcc(p,c)
                end
            end
        end
    end
end

function CatFns.loadprofile(name)
    -- [FIX] reset anh sang/hieu ung cua profile cu truoc khi nap profile moi
    CatFns.rlight(true)
    CatFns.reffects(true)
    CatFns.resetcfg()
    local d=CatFns.migr(CatFns.readjson(CatFns.proffile(name)))
    if type(d)=="table" then
        CatFns.applycfg(d)
    end
    CatFns.synccross()
    local sp=CatFns.skey(fs.key or "")
    CatFns.setfont(sp or nil,true)
    CatFns.applyvisuals()
    CatFns.theme(true)
    if s.setrgb then
        s.setrgb(cf.rgb)
    end
    if s.setgrad then
        s.setgrad(cf.grad)
    end
    CatFns.applyui()
    CatFns.regen(true)
    refresh()
end

function CatFns.switchprofile(name)
    if name==profiles.active or dead then
        return
    end
    CatFns.save()
    profiles.active=name
    CatFns.saveprofiles()
    CatFns.loadprofile(name)
    CatFns.notify("Profile loaded: "..name)
end

function CatFns.setavatar(id)
    local n=CatFns.nid(id)
    if not n then
        CatFns.notify("Invalid avatar id")
        return
    end
    cf.avatar=n
    CatFns.applyui()
    CatFns.later()
end

pages.fonts=function()
    table.clear(fb)
    CatFns.section("TARGET")
    CatFns.toggle("Game Text",fs.game,CatFns.setgame)
    CatFns.toggle("Menu Text",fs.menu,CatFns.setmenu)
    CatFns.section("CUSTOM FONT")
    CatFns.form({"Font name or Font ID"},"Add",function(v)
        local val=CatFns.trim(v[1])
        local sp=CatFns.pfont(val,true)
        if not sp then
            CatFns.notify("Font not found")
            return
        end
        if sp.f then
            local ex=false
            for _,e in ipairs(sv.fonts) do
                if e.Value==val then
                    ex=true
                    break
                end
            end
            if not ex then
                table.insert(sv.fonts,{Name=val,Value=val})
            end
        end
        CatFns.setfont(sp)
        refresh()
    end)
    for _,e in ipairs(sv.fonts) do
        local e=e
        local sp=CatFns.pfont(e.Value)
        if sp then
            local _,m=CatFns.row(e.Name,{
                {t="Apply",f=function()
                    CatFns.setfont(sp)
                end},
                {t="X",w=34,c=col.red,f=function()
                    for k,x in ipairs(sv.fonts) do
                        if x==e then
                            table.remove(sv.fonts,k)
                            break
                        end
                    end
                    CatFns.later()
                    refresh()
                end}
            })
            fb[sp.k]=m[1]
        end
    end
    CatFns.section("FONTS")
    local rws={}
    CatFns.textin("Search font",function(q)
        q=CatFns.trim(q):lower()
        for _,r in ipairs(rws) do
            r.c.Visible=q=="" or r.n:find(q,1,true)~=nil
        end
    end)
    CatFns.btn("Default","Reset",function()
        CatFns.setfont(nil)
    end,col.red)
    for _,it in ipairs(fitems) do
        local sp={e=it,k="Enum:"..it.Name}
        local cd,m=CatFns.row(it.Name,{{t="Apply",f=function()
            CatFns.setfont(sp)
        end}},it)
        fb[sp.k]=m[1]
        table.insert(rws,{c=cd,n=it.Name:lower()})
    end
    CatFns.fbrefresh()
end

pages.interface=function()
    local themes={
        {"Purple",Color3.fromRGB(120,105,255)},
        {"Blue",Color3.fromRGB(70,150,255)},
        {"Cyan",Color3.fromRGB(55,210,220)},
        {"Pink",Color3.fromRGB(255,105,175)},
        {"Red",Color3.fromRGB(245,85,100)},
        {"Green",Color3.fromRGB(70,200,140)},
        {"Orange",Color3.fromRGB(255,150,75)},
        {"White",Color3.fromRGB(230,230,245)}
    }
    CatFns.section("WINDOW")
    CatFns.slider("Menu Size",0.7,1.2,cf.size,function(v)
        cf.size=v
        CatFns.fit()
        CatFns.later()
    end,0.05)
    CatFns.slider("Transparency",0,0.5,cf.tr,function(v)
        cf.tr=v
        main.BackgroundTransparency=v
        CatFns.later()
    end,0.01)
    CatFns.toggle("Notifications",cf.notif,function(v)
        cf.notif=v
        CatFns.later()
    end)
    CatFns.toggle("Show Button",cf.btn,function(v)
        cf.btn=v
        obtn.Visible=v
        CatFns.later()
        if not v then
            CatFns.notify("Press "..cf.key.Name.." to open")
        end
    end)
    CatFns.toggle("RGB Border",cf.rgb,function(v)
        cf.rgb=v
        if v then
            cf.grad=false
        end
        if s.setrgb then
            s.setrgb(v)
        end
        if s.setgrad then
            s.setgrad(cf.grad)
        end
        CatFns.later()
    end)
    CatFns.toggle("Gradient Accent",cf.grad,function(v)
        cf.grad=v
        if v then
            cf.rgb=false
        end
        if s.setrgb then
            s.setrgb(cf.rgb)
        end
        if s.setgrad then
            s.setgrad(v)
        end
        CatFns.later()
    end)
    CatFns.form({"Toggle key ("..cf.key.Name..")"},"Set",function(v)
        local ok,k=pcall(function()
            return Enum.KeyCode[CatFns.trim(v[1])]
        end)
        if not ok or not k then
            CatFns.notify("Invalid key")
            return
        end
        cf.key=k
        CatFns.later()
        CatFns.notify("Key set to "..k.Name)
        refresh()
    end)
    CatFns.form({"Avatar image id"},"Set",function(v)
        CatFns.setavatar(CatFns.trim(v[1]))
    end)
    CatFns.btn("Reset Position","Reset",function()
        resetpos()
    end)
    CatFns.section("THEME")
    for _,t in ipairs(themes) do
        local t=t
        CatFns.btn(t[1],"Apply",function()
            cf.theme=t[2]
            CatFns.theme()
            refresh()
        end,t[2])
    end
    CatFns.colorin("Custom Color",cf.theme,function(cr)
        cf.theme=cr
        CatFns.theme()
        refresh()
    end)
end

function CatFns.dcode(v)
    v=CatFns.trim(v)
    return v:match("discord%.gg/([%w%-_]+)") or v:match("discord%.com/invite/([%w%-_]+)") or v:match("discordapp%.com/invite/([%w%-_]+)") or (v:match("^[%w%-_]+$") and v)
end

function CatFns.djoin(code)
    local r=CatFns.httpreq({
        Url="http://127.0.0.1:6463/rpc?v=1",
        Method="POST",
        Headers={["Content-Type"]="application/json",["Origin"]="https://discord.com"},
        Body=http:JSONEncode({cmd="INVITE_BROWSER",args={code=code},nonce=http:GenerateGUID(false)})
    })
    if r and (r.StatusCode==200 or r.Success) then
        CatFns.notify("Opening Discord...")
        return
    end
    if CatFns.clipcopy("https://discord.gg/"..code) then
        CatFns.notify("Discord app not found, link copied")
    else
        CatFns.notify("Could not open Discord")
    end
end

pages.discord=function()
    CatFns.section("DISCORD")
    CatFns.row(cf.dc~="" and ("discord.gg/"..cf.dc) or "No invite set",{})
    if cf.dc~="" then
        CatFns.btn("Join Discord","Join",function()
            task.spawn(CatFns.djoin,cf.dc)
        end,col.green)
        CatFns.btn("Copy invite link","Copy",function()
            CatFns.notify(CatFns.clipcopy("https://discord.gg/"..cf.dc) and "Link copied" or "Executor has no clipboard support")
        end)
    end
    CatFns.section("SET INVITE")
    CatFns.form({"Invite link or code"},"Save",function(v)
        local c=CatFns.dcode(v[1])
        if not c then
            CatFns.notify("Invalid invite")
            return
        end
        cf.dc=c
        CatFns.later()
        CatFns.notify("Invite saved")
        refresh()
    end)
end

local srv={}
local busy=false
local scanpages=2
local minfree=1
local maxping=0
local maxpl=0
local view=1
local vnames={"Ping","Players","FPS"}
local vmodes={"ping","players","fps"}
local tplist
local tpi=0

function CatFns.req(url)
    local r=CatFns.httpreq({Url=url,Method="GET"})
    if r then
        if r.StatusCode==200 then
            return r.Body
        end
        return nil,r.StatusCode
    end
    local ok,b=pcall(function()
        return http:GetAsync(url)
    end)
    if ok and type(b)=="string" then
        return b
    end
    return nil,"nohttp"
end

function CatFns.fetch()
    local out={}
    local cursor=""
    local code=nil
    for p=1,scanpages do
        local url="https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&excludeFullGames=true&limit=100"
        if cursor~="" then
            url=url.."&cursor="..http:UrlEncode(cursor)
        end
        local body,c=CatFns.req(url)
        if not body then
            code=c
            break
        end
        local ok,d=pcall(function()
            return http:JSONDecode(body)
        end)
        if not ok or type(d)~="table" or type(d.data)~="table" then
            code="bad"
            break
        end
        for _,v in ipairs(d.data) do
            if type(v)=="table" and v.id~=game.JobId and type(v.playing)=="number" and type(v.maxPlayers)=="number" and v.maxPlayers-v.playing>=minfree then
                local pn=tonumber(v.ping)
                if (maxping<=0 or (pn and pn<=maxping)) and (maxpl<=0 or v.playing<=maxpl) then
                    table.insert(out,{id=v.id,ping=pn,fps=tonumber(v.fps),playing=v.playing,max=v.maxPlayers})
                end
            end
        end
        if type(d.nextPageCursor)~="string" or d.nextPageCursor=="" then
            break
        end
        cursor=d.nextPageCursor
        if p<scanpages then
            task.wait(0.6)
        end
    end
    return out,code
end

function CatFns.sortlist(l,m)
    local out={}
    for _,v in ipairs(l or {}) do
        if (m~="ping" or v.ping) and (m~="fps" or v.fps) then
            table.insert(out,v)
        end
    end
    if m=="random" then
        for i=#out,2,-1 do
            local j=math.random(i)
            out[i],out[j]=out[j],out[i]
        end
    else
        table.sort(out,function(a,b)
            if m=="players" then
                return a.playing<b.playing
            elseif m=="fps" then
                return a.fps>b.fps
            end
            if a.ping==b.ping then
                return a.playing<b.playing
            end
            return a.ping<b.ping
        end)
    end
    return out
end

function CatFns.tpgo()
    tpi=tpi+1
    local v=tplist and tplist[tpi]
    if not v or tpi>6 then
        tplist=nil
        busy=false
        CatFns.notify("No server available")
        return
    end
    CatFns.notify("Joining "..(v.ping and (math.floor(v.ping+0.5).." ms") or "server").."...")
    local ok=pcall(function()
        TS:TeleportToPlaceInstance(game.PlaceId,v.id,lp)
    end)
    if not ok then
        CatFns.tpgo()
        return
    end
    task.delay(12,function()
        busy=false
        tplist=nil
    end)
end

-- [UPGRADE] TeleportToPlaceInstance that bai bat dong bo (server day, het han...) -> tu thu server ke tiep
table.insert(life,TS.TeleportInitFailed:Connect(function(plr)
    if plr==lp and not dead and tplist then
        CatFns.tpgo()
    end
end))

function CatFns.hop(mode,listonly)
    if busy then
        return
    end
    busy=true
    CatFns.notify("Scanning servers...")
    task.spawn(function()
        local list,code=CatFns.fetch()
        if dead then
            return
        end
        if #list==0 then
            busy=false
            if code==429 then
                CatFns.notify("Rate limited, try again later")
            elseif code=="nohttp" then
                CatFns.notify("Executor has no HTTP support")
            elseif maxping>0 then
                CatFns.notify("No server under "..maxping.." ms")
            else
                CatFns.notify("No server found")
            end
            return
        end
        if listonly then
            srv=list
            busy=false
            CatFns.notify(#list.." servers found")
            refresh()
            return
        end
        local l=CatFns.sortlist(list,mode)
        if #l==0 then
            busy=false
            CatFns.notify("Servers have no "..mode.." data")
            return
        end
        tplist=l
        tpi=0
        CatFns.tpgo()
    end)
end

function CatFns.pcolor(p)
    if not p then
        return nil
    end
    if p<=80 then
        return col.green
    elseif p<=150 then
        return col.yellow
    end
    return col.red
end

pages.servers=function()
    CatFns.section("CURRENT SERVER")
    local ok,p=pcall(function()
        return lp:GetNetworkPing()*1000
    end)
    CatFns.row("Ping: "..((ok and p) and (math.floor(p).." ms") or "--").."   Players: "..#plrs:GetPlayers().."/"..plrs.MaxPlayers,{})
    CatFns.row("Job ID: "..string.sub(game.JobId,1,8).."...",{
        {t="Copy",w=64,f=function()
            CatFns.notify(CatFns.clipcopy(game.JobId) and "Job ID copied" or "Executor has no clipboard support")
        end}
    })
    CatFns.btn("Rejoin this server","Rejoin",function()
        CatFns.notify("Rejoining...")
        tplist=nil
        pcall(function()
            TS:TeleportToPlaceInstance(game.PlaceId,game.JobId,lp)
        end)
    end)
    CatFns.section("QUICK HOP")
    CatFns.btn("Lowest ping","Hop",function()
        CatFns.hop("ping")
    end,col.green)
    CatFns.btn("Fewest players","Hop",function()
        CatFns.hop("players")
    end)
    CatFns.btn("Highest FPS","Hop",function()
        CatFns.hop("fps")
    end)
    CatFns.btn("Random server","Hop",function()
        CatFns.hop("random")
    end)
    CatFns.section("JOIN BY ID")
    CatFns.form({"Job ID"},"Join",function(v)
        local id=CatFns.trim(v[1])
        if not id:match("^[%w%-]+$") then
            CatFns.notify("Invalid Job ID")
            return
        end
        if busy then
            return
        end
        busy=true
        tplist={{id=id}}
        tpi=0
        CatFns.tpgo()
    end)
    CatFns.section("FILTERS")
    CatFns.slider("Pages to Scan",1,8,scanpages,function(v)
        scanpages=v
    end,1)
    CatFns.slider("Min Free Slots",1,10,minfree,function(v)
        minfree=v
    end,1)
    CatFns.slider("Max Ping (0 = off)",0,400,maxping,function(v)
        maxping=v
    end,10)
    CatFns.slider("Max Players (0 = off)",0,100,maxpl,function(v)
        maxpl=v
    end,1)
    CatFns.section("SERVER LIST")
    CatFns.btn("Scan servers","Scan",function()
        CatFns.hop("ping",true)
    end)
    CatFns.btn("Sort: "..vnames[view],"Change",function()
        view=view%#vnames+1
        refresh()
    end)
    local shown=CatFns.sortlist(srv,vmodes[view])
    for i=1,math.min(#shown,12) do
        local v=shown[i]
        CatFns.row((v.ping and (math.floor(v.ping+0.5).." ms") or "? ms").."  |  "..v.playing.."/"..v.max.."  |  "..(v.fps and (math.floor(v.fps+0.5).." fps") or "? fps"),{
            {t="Join",w=64,c=CatFns.pcolor(v.ping),f=function()
                if busy then
                    return
                end
                busy=true
                tplist={v}
                tpi=0
                CatFns.tpgo()
            end}
        })
    end
end

pages["shift lock"]=function()
    CatFns.section("CUSTOM CURSOR")
    CatFns.toggle("Enable",C.on,function(v)
        C.on=v
        CatFns.regen(true)
    end)
    CatFns.btn("Mode: "..(C.m==1 and "System cursor" or "Overlay (GUI)"),"Change",function()
        C.m=C.m%2+1
        CatFns.regen(true)
        refresh()
    end)
    if C.m==1 and not canfile then
        CatFns.row("No file support, overlay is used instead",{})
    end
    CatFns.btn("Preview (3s)","Show",function()
        pvt=os.clock()+3
        if curl then
            uis.MouseIcon=curl
        end
    end)
    CatFns.section("SHIFT LOCK")
    CatFns.toggle("Force Shift Lock (Left Shift key)",C.fl,function(v)
        C.fl=v
        if not v then
            CatFns.flset(false)
        end
        CatFns.later()
    end)
    CatFns.slider("Camera Offset",0,4,C.off,function(v)
        C.off=v
        CatFns.later()
    end,0.25)
    CatFns.section("STYLE")
    for i,n in ipairs(snames) do
        local i=i
        CatFns.btn(n,C.st==i and "Active" or "Use",function()
            C.st=i
            CatFns.regen(true)
            pvt=os.clock()+2
            if curl then
                uis.MouseIcon=curl
            end
            refresh()
        end,C.st==i and col.green or nil)
    end
    CatFns.section("LOOK")
    CatFns.slider("Size",4,40,C.sz,function(v)
        C.sz=v
        CatFns.regen()
    end,1)
    CatFns.slider("Thickness",1,6,C.th,function(v)
        C.th=v
        CatFns.regen()
    end,1)
    CatFns.slider("Transparency",0,0.9,C.tr,function(v)
        C.tr=v
        CatFns.regen()
    end,0.05)
    CatFns.colorin("Color",Color3.fromRGB(C.c[1],C.c[2],C.c[3]),function(cr)
        C.c={math.round(cr.R*255),math.round(cr.G*255),math.round(cr.B*255)}
        CatFns.regen()
    end)
    CatFns.toggle("Outline",C.ol,function(v)
        C.ol=v
        CatFns.regen()
    end)
    if C.m==2 then
        CatFns.toggle("Always Show",C.al,function(v)
            C.al=v
            CatFns.later()
        end)
        CatFns.toggle("Rainbow",C.rb,function(v)
            C.rb=v
            if not v and cg then
                CatFns.paint(Color3.fromRGB(C.c[1],C.c[2],C.c[3]))
            end
            CatFns.later()
        end)
    end
    CatFns.btn("Restore cursor settings","Reset",function()
        s.slreset()
        refresh()
    end,col.red)
end

local renameTarget

function CatFns.createprofile(name)
    name=CatFns.trim(name)
    if name=="" or name:find("%W") then
        CatFns.notify("Invalid profile name")
        return
    end
    for _,p in ipairs(profiles.list) do
        if p==name then
            CatFns.notify("Profile already exists")
            return
        end
    end
    CatFns.save()
    table.insert(profiles.list,name)
    CatFns.saveprofiles()
    CatFns.writejson(CatFns.proffile(name),CatFns.buildcfg())
    CatFns.switchprofile(name)
end

function CatFns.renameprofile(old,new)
    new=CatFns.trim(new)
    if old=="Default" then
        CatFns.notify("Default profile cannot be renamed")
        renameTarget=nil
        refresh()
        return
    end
    if new=="" or new:find("%W") then
        CatFns.notify("Invalid profile name")
        return
    end
    if old==new then
        renameTarget=nil
        refresh()
        return
    end
    for _,p in ipairs(profiles.list) do
        if p==new then
            CatFns.notify("Profile already exists")
            return
        end
    end
    if profiles.active==old then
        CatFns.save()
    end
    local oldfile=CatFns.proffile(old)
    local newfile=CatFns.proffile(new)
    local d=CatFns.readjson(oldfile)
    if type(d)=="table" then
        CatFns.writejson(newfile,d)
    end
    -- [FIX] xoa file cu de khong de lai file rac
    local rm=CatFns.apifn("delfile")
    if rm then
        pcall(rm,oldfile)
        pcall(rm,oldfile..".bak")
    end
    CatFns.lastjson[oldfile]=nil
    local idx=nil
    for i,p in ipairs(profiles.list) do
        if p==old then
            idx=i
            break
        end
    end
    if idx then
        profiles.list[idx]=new
    end
    if profiles.active==old then
        profiles.active=new
    end
    CatFns.saveprofiles()
    renameTarget=nil
    CatFns.notify("Profile renamed")
    refresh()
end

function CatFns.deleteprofile(p)
    if p=="Default" then
        CatFns.notify("Default profile cannot be deleted")
        return
    end
    CatFns.confirm("Delete profile '"..p.."'?",function()
        local file=CatFns.proffile(p)
        local rm=CatFns.apifn("delfile")
        if rm then
            pcall(rm,file)
            pcall(rm,file..".bak")
        end
        CatFns.lastjson[file]=nil
        for i,x in ipairs(profiles.list) do
            if x==p then
                table.remove(profiles.list,i)
                break
            end
        end
        local wasactive=profiles.active==p
        if wasactive then
            profiles.active="Default"
        end
        CatFns.saveprofiles()
        if wasactive then
            -- [FIX] truoc day chi applycfg, khong nap lai day du giao dien/hieu ung
            CatFns.loadprofile("Default")
        end
        CatFns.notify("Profile deleted")
        refresh()
    end)
end

function CatFns.exportcfg()
    local ok,txt=pcall(function()
        return http:JSONEncode(CatFns.buildcfg())
    end)
    if not ok then
        CatFns.notify("Export failed")
        return
    end
    if HASFS then
        pcall(function()
            FIO.write("Cat_export.json",txt)
        end)
    end
    if CatFns.clipcopy(txt) then
        CatFns.notify("Config copied to clipboard")
    else
        CatFns.notify("Config saved to Cat_export.json")
    end
end

function CatFns.importcfg(txt)
    txt=CatFns.trim(txt)
    if txt=="" then
        CatFns.notify("Nothing to import")
        return
    end
    local ok,d=pcall(function()
        return http:JSONDecode(txt)
    end)
    if not ok or type(d)~="table" then
        CatFns.notify("Invalid config JSON")
        return
    end
    d=CatFns.migr(d)
    if not d then
        CatFns.notify("Invalid config data")
        return
    end
    -- [FIX] reset trang thai cu truoc khi nhap de khong bi "dinh" hieu ung cu
    CatFns.rlight(true)
    CatFns.reffects(true)
    CatFns.applycfg(d)
    CatFns.synccross()
    CatFns.applyvisuals()
    CatFns.theme(true)
    if s.setrgb then
        s.setrgb(cf.rgb)
    end
    if s.setgrad then
        s.setgrad(cf.grad)
    end
    local sp=CatFns.skey(fs.key or "")
    CatFns.setfont(sp or nil,true)
    CatFns.applyui()
    CatFns.regen(true)
    CatFns.save()
    refresh()
    CatFns.notify("Config imported")
end

function CatFns.resetall()
    CatFns.rgraphics(true)
    CatFns.rlight(true)
    CatFns.reffects(true)
    CatFns.rgt()
    CatFns.resetcfg()
    CatFns.synccross()
    if s.slreset then
        s.slreset()
    end
    if s.setrgb then
        s.setrgb(false)
    end
    if s.setgrad then
        s.setgrad(false)
    end
    if s.setfb then
        s.setfb(false)
    end
    s.fps=false
    s.png=false
    s.mem=false
    for _,l in pairs(sl) do
        l.Visible=false
    end
    CatFns.menufont()
    CatFns.applyui()
    CatFns.theme(true)
    resetpos()
    refresh()
    CatFns.later()
    CatFns.notify("Defaults restored")
end

pages.general=function()
    CatFns.section("RESTORE")
    CatFns.btn("Restore Graphics","Restore",function()
        CatFns.rgraphics()
        refresh()
    end)
    CatFns.btn("Restore Lighting","Restore",function()
        CatFns.rlight()
        refresh()
    end)
    CatFns.btn("Restore Effects","Restore",function()
        CatFns.reffects()
        refresh()
    end)
    CatFns.btn("Restore Fonts","Restore",function()
        CatFns.setfont(nil)
    end)
    CatFns.btn("Restore Defaults","Reset",function()
        CatFns.confirm("Restore every setting to default?",function()
            CatFns.resetall()
        end)
    end,col.red)
    CatFns.section("PROFILES")
    CatFns.row("Active: "..profiles.active,{})
    for _,p in ipairs(profiles.list) do
        local p=p
        local btns={}
        if p~=profiles.active then
            table.insert(btns,{t="Use",w=64,c=col.green,f=function()
                CatFns.switchprofile(p)
            end})
        end
        if p~="Default" then
            table.insert(btns,{t="Rename",w=74,f=function()
                renameTarget=p
                refresh()
            end})
            table.insert(btns,{t="X",w=34,c=col.red,f=function()
                CatFns.deleteprofile(p)
            end})
        end
        CatFns.row(p,btns)
    end
    CatFns.form({renameTarget and ("New name for "..renameTarget) or "New profile name"},renameTarget and "Rename" or "Create",function(v)
        local n=CatFns.trim(v[1])
        if renameTarget then
            CatFns.renameprofile(renameTarget,n)
        else
            CatFns.createprofile(n)
        end
    end)
    CatFns.btn("Export config","Export",CatFns.exportcfg)
    local icd=CatFns.card(140)
    CatFns.tx("TextLabel",{Size=UDim2.new(1,-24,0,18),Position=UDim2.fromOffset(12,8),BackgroundTransparency=1,Text="Import config",TextColor3=col.text,TextSize=13,TextXAlignment=Enum.TextXAlignment.Left},icd)
    local itb=CatFns.tx("TextBox",{Size=UDim2.new(1,-24,0,92),Position=UDim2.fromOffset(12,30),BackgroundColor3=col.panel,PlaceholderText="Paste exported config JSON here",PlaceholderColor3=col.sub,Text="",TextColor3=col.text,TextSize=11,ClearTextOnFocus=false,MultiLine=true,TextXAlignment=Enum.TextXAlignment.Left},icd)
    CatFns.rc(itb,9)
    CatFns.pd(itb,8)
    -- [FIX] UDim2.new(1,-92,0,1,-36) co 5 doi so -> nut Import bi sai vi tri
    local ib=CatFns.tx("TextButton",{Size=UDim2.fromOffset(80,28),Position=UDim2.new(1,-92,1,-36),BackgroundColor3=col.acc,Text="Import",TextColor3=col.text,TextSize=12,AutoButtonColor=false},icd,true)
    CatFns.rc(ib,9)
    CatFns.hover(ib)
    table.insert(conns,ib.MouseButton1Click:Connect(function()
        CatFns.importcfg(itb.Text)
    end))
    CatFns.section("MODE")
    CatFns.toggle("Safe Mode",cf.safe,function(v)
        cf.safe=v
        if v then
            CatFns.rgraphics(true)
            CatFns.reffects(true)
            CatFns.rlight(true)
            CatFns.notify("Safe Mode on, visuals reset")
        end
        CatFns.later()
    end)
    CatFns.section("INFO")
    CatFns.row("Cat",{})
    CatFns.row("Executor: "..EXECNAME,{})
    CatFns.row("Config: "..CatFns.cfgpath(),{})
    CatFns.row("File support: "..(HASFS and "yes" or "no"),{})
    CatFns.row("Local sky support: "..(lfs and "yes" or "no"),{})
    CatFns.row("HTTP support: "..(CatFns.httpcapable() and "yes" or "no"),{})
    CatFns.row("Clipboard support: "..(CatFns.clipcapable() and "yes" or "no"),{})
    CatFns.section("RECENT ACTIONS")
    local n=#loglist
    if n==0 then
        CatFns.row("No actions yet",{})
    else
        for i=math.max(1,n-11),n do
            local e=loglist[i]
            CatFns.row(e.m,{})
        end
    end
    CatFns.section("SCRIPT")
    CatFns.btn("Unload Script","Unload",function()
        CatFns.confirm("Close Cat and restore the game?",function()
            shutdown()
        end)
    end,col.red)
end

CatFns.hover(minb,life)
CatFns.hover(closeb,life)
ind=CatFns.mk("Frame",{Size=UDim2.fromOffset(4,20),Position=UDim2.fromOffset(4,120),BackgroundColor3=col.acc,BorderSizePixel=0},side)
CatFns.rc(ind,2)
for i,n in ipairs(pnames) do
    local n=n
    local b=CatFns.tx("TextButton",{Name=n,Size=UDim2.new(1,0,0,28),BackgroundColor3=col.panel,Text=n,TextColor3=col.text,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,AutoButtonColor=false,LayoutOrder=i},nav)
    CatFns.rc(b,9)
    CatFns.pd(b,12)
    CatFns.hover(b,life)
    b.MouseButton1Click:Connect(function()
        openpage(n)
    end)
    navs[n]=b
end

obtn=CatFns.mk("ImageButton",{Name="CatButton",Size=UDim2.fromOffset(58,58),Position=UDim2.new(0,25,0.5,-29),BackgroundColor3=col.panel,Image="rbxassetid://"..cf.avatar,ScaleType=Enum.ScaleType.Crop,AutoButtonColor=false,Visible=cf.btn,ZIndex=25},gui)
CatFns.rc(obtn,18)
CatFns.ast(obtn,0.45)
CatFns.fallback(obtn,13)
local obs=CatFns.mk("UIScale",{Scale=1},obtn)
table.insert(life,obtn.MouseButton1Down:Connect(function()
    tw:Create(obs,TweenInfo.new(0.08,Enum.EasingStyle.Quart,Enum.EasingDirection.Out),{Scale=0.88}):Play()
    CatFns.tn(obtn,{Rotation=8},0.08)
end))
table.insert(life,obtn.MouseButton1Up:Connect(function()
    tw:Create(obs,TweenInfo.new(0.2,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Scale=1}):Play()
    CatFns.tn(obtn,{Rotation=4},0.2)
end))
table.insert(life,obtn.MouseEnter:Connect(function()
    CatFns.tn(obtn,{Rotation=4},0.15)
end))
table.insert(life,obtn.MouseLeave:Connect(function()
    CatFns.tn(obtn,{Rotation=0},0.15)
    tw:Create(obs,TweenInfo.new(0.2,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Scale=1}):Play()
end))
local moved=CatFns.drag(obtn,obtn,false)
CatFns.drag(main,head,true)

-- [FIX] avatar va trang thai nut mo menu truoc day khong duoc ap dung khi load config
function CatFns.applyui()
    local img="rbxassetid://"..cf.avatar
    if av then
        av.Image=img
    end
    if obtn then
        obtn.Image=img
        obtn.Visible=cf.btn
    end
end

resetpos=function()
    main.Position=UDim2.fromScale(0.5,0.5)
    obtn.Position=UDim2.new(0,25,0.5,-29)
    for k,l in pairs(sl) do
        l.Position=spos[k]
    end
    CatFns.notify("Position reset")
end

function CatFns.layout(m)
    mini=m
    side.Visible=not m
    holder.Visible=not m
    CatFns.tn(body,{Position=m and UDim2.fromOffset(12,10) or UDim2.fromOffset(185,14),Size=m and UDim2.new(1,-24,0,40) or UDim2.new(1,-195,1,-28)},0.22)
    minb.Text=m and "+" or "-"
    searchb.Visible=not m
end

function CatFns.setmin(m)
    CatFns.closesearch()
    CatFns.layout(m)
    if sztween then
        sztween:Cancel()
        sztween=nil
    end
    if m then
        title.Text="Cat"
        sztween=CatFns.tn(main,{Size=UDim2.fromOffset(250,60)},0.22)
    else
        sztween=CatFns.tn(main,{Size=UDim2.fromOffset(700,460)},0.25)
        openpage(cur or "Performance",true)
    end
end

function CatFns.openmenu()
    if dead then
        return
    end
    anim=anim+1
    if sztween then
        sztween:Cancel()
        sztween=nil
    end
    CatFns.layout(false)
    main.Visible=true
    CatFns.fit()
    main.Size=UDim2.fromOffset(700,460)
    scale.Scale=mscale*0.82
    main.BackgroundTransparency=1
    CatFns.tn(main,{BackgroundTransparency=cf.tr},0.2)
    scaletw=tw:Create(scale,TweenInfo.new(0.3,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Scale=mscale})
    scaletw:Play()
    openpage(cur or "Performance",true)
    if s.navin then
        s.navin()
    end
end

function CatFns.closemenu()
    if dead then
        return
    end
    anim=anim+1
    local id=anim
    CatFns.closesearch()
    if sztween then
        sztween:Cancel()
        sztween=nil
    end
    CatFns.tn(main,{BackgroundTransparency=math.min(1,cf.tr+0.3)},0.16)
    scaletw=tw:Create(scale,TweenInfo.new(0.18,Enum.EasingStyle.Back,Enum.EasingDirection.In),{Scale=mscale*0.86})
    scaletw:Play()
    task.delay(0.2,function()
        if anim==id and main.Parent then
            main.Visible=false
            main.BackgroundTransparency=cf.tr
            scale.Scale=mscale
        end
    end)
end

function CatFns.togglemenu()
    if main.Visible then
        CatFns.closemenu()
    else
        CatFns.openmenu()
    end
end

local sres
function CatFns.findcard(label)
    for _,o in ipairs(holder:GetChildren()) do
        if o:IsA("Frame") then
            for _,c in ipairs(o:GetChildren()) do
                if (c:IsA("TextLabel") and c.Text==label) or (c:IsA("TextBox") and c.PlaceholderText==label) then
                    return o
                end
            end
        end
    end
    return nil
end

function CatFns.flash(f)
    if not f then
        return
    end
    local s2=f:FindFirstChildOfClass("UIStroke")
    if not s2 then
        return
    end
    local old=s2.Color
    s2.Color=col.acc
    s2.Transparency=0.2
    task.delay(0.9,function()
        if s2.Parent then
            s2.Color=old
            s2.Transparency=0.94
        end
    end)
end

function CatFns.closesearch(instant)
    if sesc then
        sesc:Disconnect()
        sesc=nil
    end
    local b=spanel
    spanel=nil
    if not b then
        return
    end
    if instant or dead or not b.Parent then
        b:Destroy()
        return
    end
    local pn=b:FindFirstChild("Panel")
    local ps=pn and pn:FindFirstChildOfClass("UIScale")
    CatFns.tn(b,{BackgroundTransparency=1},0.14)
    if pn then
        CatFns.tn(pn,{GroupTransparency=1},0.14)
    end
    if ps then
        tw:Create(ps,TweenInfo.new(0.14,Enum.EasingStyle.Quart,Enum.EasingDirection.In),{Scale=0.92}):Play()
    end
    task.delay(0.16,function()
        if b.Parent then
            b:Destroy()
        end
    end)
end

function CatFns.searchresult(entry)
    CatFns.closesearch()
    if not entry.page then
        return
    end
    openpage(entry.page)
    task.delay(0.08,function()
        if dead then
            return
        end
        local f=CatFns.findcard(entry.label)
        if f and f.Parent then
            local top=holder.AbsolutePosition.Y-holder.CanvasPosition.Y
            holder.CanvasPosition=Vector2.new(holder.CanvasPosition.X,math.max(0,f.AbsolutePosition.Y-top-10))
            CatFns.flash(f)
        end
    end)
end

function CatFns.buildsearch()
    CatFns.closesearch(true)
    local block=CatFns.mk("TextButton",{Name="CatSearch",Size=UDim2.fromScale(1,1),BackgroundColor3=Color3.new(0,0,0),BackgroundTransparency=1,BorderSizePixel=0,Text="",AutoButtonColor=false,ZIndex=44},gui)
    local pn=CatFns.mk("CanvasGroup",{Name="Panel",Size=UDim2.fromOffset(430,360),AnchorPoint=Vector2.new(0.5,0.5),Position=UDim2.fromScale(0.5,0.5),BackgroundColor3=col.card,BorderSizePixel=0,GroupTransparency=1,Active=true,ZIndex=45},block)
    CatFns.rc(pn,14)
    CatFns.st(pn,col.acc,0.5)
    local ps=CatFns.mk("UIScale",{Scale=0.92},pn)
    spanel=block
    CatFns.tn(block,{BackgroundTransparency=0.6},0.18)
    CatFns.tn(pn,{GroupTransparency=0},0.18)
    tw:Create(ps,TweenInfo.new(0.28,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Scale=1}):Play()
    block.MouseButton1Click:Connect(function()
        if spanel==block then
            CatFns.closesearch()
        end
    end)
    local tb=CatFns.tx("TextBox",{Size=UDim2.new(1,-24,0,32),Position=UDim2.fromOffset(12,12),BackgroundColor3=col.panel,PlaceholderText="Search settings...",PlaceholderColor3=col.sub,Text="",TextColor3=col.text,TextSize=13,ClearTextOnFocus=false,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=46},pn,true)
    CatFns.rc(tb,9)
    CatFns.pd(tb,10)
    sres=CatFns.mk("ScrollingFrame",{Size=UDim2.new(1,-24,1,-60),Position=UDim2.fromOffset(12,52),BackgroundTransparency=1,BorderSizePixel=0,ScrollBarThickness=3,ScrollBarImageColor3=col.acc,ScrollingDirection=Enum.ScrollingDirection.Y,AutomaticCanvasSize=Enum.AutomaticSize.Y,CanvasSize=UDim2.new(),ZIndex=46},pn)
    CatFns.mk("UIListLayout",{Padding=UDim.new(0,4),SortOrder=Enum.SortOrder.LayoutOrder},sres)
    local firstE=nil
    local function fill(q,animate)
        for _,o in ipairs(sres:GetChildren()) do
            if o:IsA("GuiObject") then
                o:Destroy()
            end
        end
        firstE=nil
        q=CatFns.trim(q):lower()
        local n=0
        local keys={}
        for k in pairs(searchreg) do
            table.insert(keys,k)
        end
        table.sort(keys,function(a,b)
            return searchreg[a].label:lower()<searchreg[b].label:lower()
        end)
        for _,k in ipairs(keys) do
            local e=searchreg[k]
            if q=="" or e.label:lower():find(q,1,true) then
                n=n+1
                if n>30 then
                    break
                end
                if not firstE then
                    firstE=e
                end
                local tr=animate and 1 or 0
                local b=CatFns.mk("TextButton",{Size=UDim2.new(1,-4,0,28),BackgroundColor3=col.panel,BackgroundTransparency=tr,Text="",AutoButtonColor=false,LayoutOrder=n,ZIndex=47},sres)
                CatFns.rc(b,7)
                local lab=CatFns.tx("TextLabel",{Size=UDim2.new(1,-120,1,0),Position=UDim2.fromOffset(10,0),BackgroundTransparency=1,Text=e.label,TextColor3=col.text,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd,TextTransparency=tr,ZIndex=48},b)
                local tag=CatFns.tx("TextLabel",{Size=UDim2.fromOffset(100,28),Position=UDim2.new(1,-108,0,0),BackgroundTransparency=1,Text=e.page or "",TextColor3=col.sub,TextSize=11,TextXAlignment=Enum.TextXAlignment.Right,TextTransparency=tr,ZIndex=48},b)
                b.MouseEnter:Connect(function()
                    CatFns.tn(b,{BackgroundColor3=col.acc},0.12)
                end)
                b.MouseLeave:Connect(function()
                    CatFns.tn(b,{BackgroundColor3=col.panel},0.12)
                end)
                b.MouseButton1Click:Connect(function()
                    CatFns.searchresult(e)
                end)
                if animate then
                    task.delay(0.03+0.018*math.min(n,12),function()
                        if b.Parent then
                            CatFns.tn(b,{BackgroundTransparency=0},0.2)
                            CatFns.tn(lab,{TextTransparency=0},0.2)
                            CatFns.tn(tag,{TextTransparency=0},0.2)
                        end
                    end)
                end
            end
        end
        if n==0 then
            local l=CatFns.tx("TextLabel",{Size=UDim2.new(1,-4,0,28),BackgroundTransparency=1,Text="No match",TextColor3=col.sub,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=47},sres)
            CatFns.pd(l,10)
        end
    end
    fill("",true)
    local tok=0
    tb:GetPropertyChangedSignal("Text"):Connect(function()
        tok=tok+1
        local id=tok
        task.delay(0.08,function()
            if id==tok and spanel==block and tb.Parent then
                fill(tb.Text,false)
            end
        end)
    end)
    tb.FocusLost:Connect(function(enter)
        if enter and firstE and spanel==block then
            CatFns.searchresult(firstE)
        end
    end)
    task.defer(function()
        if tb.Parent and spanel==block then
            tb:CaptureFocus()
        end
    end)
    sesc=uis.InputBegan:Connect(function(i,gp)
        if dead or not spanel then
            return
        end
        if i.KeyCode==Enum.KeyCode.Escape then
            CatFns.closesearch()
        end
    end)
end

table.insert(life,searchb.MouseButton1Click:Connect(function()
    CatFns.buildsearch()
end))

openpage=function(name,keep)
    local build=pages[name:lower()]
    if not build or dead then
        return
    end
    local pos=holder.CanvasPosition
    cur=name
    if not mini then
        title.Text=name
    end
    for _,x in ipairs(conns) do
        x:Disconnect()
    end
    table.clear(conns)
    CatFns.clear()
    build()
    for i,n in ipairs(pnames) do
        local on=n==name
        CatFns.tn(navs[n],{BackgroundColor3=on and col.acc or col.panel})
        if on then
            CatFns.tn(ind,{Position=UDim2.fromOffset(4,120+(i-1)*32)},0.25)
        end
    end
    if keep then
        holder.Position=UDim2.fromOffset(0,55)
    else
        holder.Position=UDim2.fromOffset(18,55)
        CatFns.tn(holder,{Position=UDim2.fromOffset(0,55)},0.25)
        holder.CanvasPosition=Vector2.new(0,0)
        return
    end
    task.spawn(function()
        rs.Heartbeat:Wait()
        rs.Heartbeat:Wait()
        if not dead and holder.Parent then
            holder.CanvasPosition=pos
        end
    end)
end

refresh=function()
    if cur and not mini and main.Visible then
        s.still=true
        openpage(cur,true)
        s.still=false
    end
end

s.navin=function()
    for i,n in ipairs(pnames) do
        local b=navs[n]
        b.BackgroundTransparency=1
        b.TextTransparency=1
        task.delay(0.03*i,function()
            if b.Parent then
                CatFns.tn(b,{BackgroundTransparency=0,TextTransparency=0},0.25)
            end
        end)
    end
end

shutdown=function()
    if dead then
        return
    end
    CatFns.save()
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
    for _,x in ipairs(conns) do
        x:Disconnect()
    end
    for _,x in ipairs(life) do
        x:Disconnect()
    end
    table.clear(conns)
    table.clear(life)
    pcall(function()
        rs:UnbindFromRenderStep("CatSettingsFOV")
    end)
    CatFns.rgraphics(true)
    CatFns.rlight(true)
    CatFns.reffects(true)
    CatFns.rgt()
    if s.kill then
        s.kill()
    end
    if cpanel then
        cpanel:Destroy()
        cpanel=nil
    end
    CatFns.closesearch()
    if env.CatSettingsUnload==shutdown then
        env.CatSettingsUnload=nil
    end
    if gui then
        gui:Destroy()
    end
end

minb.MouseButton1Click:Connect(function()
    CatFns.setmin(not mini)
end)
closeb.MouseButton1Click:Connect(function()
    CatFns.confirm("Close Cat and restore the game?",function()
        shutdown()
    end)
end)
obtn.MouseButton1Click:Connect(function()
    if moved() then
        return
    end
    CatFns.togglemenu()
end)

table.insert(life,uis.InputBegan:Connect(function(i,processed)
    if processed or dead then
        return
    end
    if i.KeyCode==cf.key then
        CatFns.togglemenu()
        return
    end
    -- [FIX] "Force Shift Lock" truoc day khong co phim nao de bat/tat (flset(true) khong bao gio duoc goi)
    if C.fl and i.KeyCode==Enum.KeyCode.LeftShift then
        CatFns.flset(not flon)
        CatFns.notify(flon and "Shift lock on" or "Shift lock off")
    end
end))

local ff=0
local ft=0
local t2=0
local t3=0
local t4=0

table.insert(life,rs.RenderStepped:Connect(function(dt)
    if dead then
        return
    end
    ff=ff+1
    ft=ft+dt
    if s.spin~=0 then
        local sk=lt:FindFirstChild("CatSky")
        if sk then
            local o=sk.SkyboxOrientation
            sk.SkyboxOrientation=Vector3.new(o.X,(o.Y+s.spin*dt)%360,o.Z)
        end
    end
    if ft>=0.5 then
        if s.fps and sl.fps.Visible then
            local n=math.floor(ff/ft+0.5)
            sl.fps.Text="FPS: "..n
            sl.fps.TextColor3=n>=50 and col.green or (n>=30 and col.yellow or col.red)
        end
        ff=0
        ft=0
    end
end))

table.insert(life,rs.Heartbeat:Connect(function(dt)
    if dead then
        return
    end
    t2=t2+dt
    t3=t3+dt
    t4=t4+dt
    CatFns.pumpq()
    if s.flow~=0 and not s.lck then
        lt.ClockTime=(lt.ClockTime+s.flow*dt)%24
    end
    if s.cyc~=0 and t4>=0.05 then
        t4=0
        s.hue=(s.hue+s.cyc*0.1*dt)%1
        local at=lt:FindFirstChild("CatAtmosphere") or CatFns.catatm()
        at.Color=Color3.fromHSV(s.hue,0.55,1)
        lt.OutdoorAmbient=Color3.fromHSV(s.hue,0.45,0.85)
        lt.FogColor=Color3.fromHSV(s.hue,0.5,1)
    end
    if t2>=1 then
        t2=0
        if s.png and sl.png.Visible then
            local ms
            local ok,p=pcall(function()
                return lp:GetNetworkPing()*1000
            end)
            if ok and p then
                ms=p
            else
                local ok2,v=pcall(function()
                    return stats.Network.ServerStatsItem["Data Ping"]:GetValue()
                end)
                if ok2 and v then
                    ms=v
                end
            end
            if ms then
                sl.png.Text="PING: "..math.floor(ms).." ms"
            end
        end
        if s.mem and sl.mem.Visible then
            local ok,v=pcall(function()
                return stats:GetTotalMemoryUsageMb()
            end)
            if ok and v then
                sl.mem.Text="MEM: "..math.floor(v).." MB"
            end
        end
    end
    if t3>=1 then
        t3=0
        local sp=fs.spec
        if sp and fs.game then
            for o in pairs(gtx) do
                if o.Parent then
                    local ok,m=pcall(CatFns.same,o,sp)
                    if ok and not m then
                        pcall(CatFns.apply,o,sp)
                    end
                else
                    gtx[o]=nil
                end
            end
        end
    end
end))

function CatFns.handle(o)
    if dead or not o.Parent then
        return
    end
    local k=CatFns.kind(o)
    if k and s[kf[k]] then
        pcall(function()
            local d=CatFns.capture(o,k)
            if d then
                CatFns.applyd(o,d)
            end
        end)
    end
    if fs.spec and fs.game and CatFns.isgt(o) then
        CatFns.stylegt(o)
    end
end

function CatFns.added(o)
    if dead then
        return
    end
    local fa=fs.spec~=nil and fs.game
    if not (fa or s.ptc or s.tex or s.pst or s.mat) then
        return
    end
    local k=CatFns.kind(o)
    if (k~=nil and s[kf[k]]) or (fa and CatFns.isgt(o)) then
        CatFns.enq(o)
    end
end

table.insert(life,workspace.DescendantAdded:Connect(CatFns.added))
table.insert(life,lt.DescendantAdded:Connect(CatFns.added))
if pg then
    table.insert(life,pg.DescendantAdded:Connect(function(o)
        if not dead and fs.spec~=nil and fs.game and CatFns.isgt(o) then
            CatFns.enq(o)
        end
    end))
end

table.insert(life,rs.RenderStepped:Connect(function(dt)
    if dead then
        return
    end
    CatFns.crossupd()
    if s.mb and mb and mb.Parent then
        local cam=workspace.CurrentCamera
        if cam then
            local v=cam.CFrame.LookVector
            if lv and dt>0 then
                local target=math.clamp((v-lv).Magnitude/dt*4*s.mbi,0,28)
                mb.Size=mb.Size+(target-mb.Size)*math.min(dt*12,1)
            end
            lv=v
        end
    end
    if cf.rgb and rgg and rgg.Parent and main.Visible then
        rgg.Rotation=(rgg.Rotation+80*dt)%360
    end
    if cf.grad and rgg2 and rgg2.Parent and main.Visible then
        rgg2.Rotation=(rgg2.Rotation+40*dt)%360
    end
end))

local ost=obtn:FindFirstChildOfClass("UIStroke")
local ostv=0.45
table.insert(life,rs.RenderStepped:Connect(function()
    if dead or not ost or not obtn.Visible then
        return
    end
    local nv=0.45+0.2*math.sin(os.clock()*2.5)
    if math.abs(nv-ostv)>0.005 then
        ostv=nv
        ost.Transparency=nv
    end
end))

-- ===== Khoi dong =====
CatFns.loadprofiles()
local cfgd=CatFns.migr(CatFns.readjson(CatFns.cfgpath()))
if type(cfgd)=="table" then
    CatFns.applycfg(cfgd)
end
CatFns.synccross()
if sv.active then
    local sp=CatFns.skey(sv.active)
    if sp then
        CatFns.setfont(sp,true)
    end
end
CatFns.theme(true)
if s.setrgb then
    s.setrgb(cf.rgb)
end
if s.setgrad then
    s.setgrad(cf.grad)
end
CatFns.applyui()
CatFns.applyvisuals()
if C.on then
    CatFns.regen(true)
end
CatFns.watch()
table.insert(life,workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(CatFns.watch))

function CatFns.buildregistry()
    local keep=cur
    s.still=true
    for _,n in ipairs(pnames) do
        openpage(n,true)
    end
    openpage(keep or "Performance",true)
    s.still=false
end

CatFns.buildregistry()
openpage("Performance")
env.CatSettingsUnload=shutdown
CatFns.notify("Cat loaded")
