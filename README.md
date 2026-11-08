local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local HttpService = game:GetService("HttpService")
local TextService = game:GetService("TextService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local Shared = (getgenv and getgenv()) or _G

if Shared.CatSettingsUnload then
    pcall(Shared.CatSettingsUnload)
    Shared.CatSettingsUnload = nil
end

local previousGui = PlayerGui:FindFirstChild("CatSettings")
if previousGui then
    previousGui:Destroy()
end

local FILE_NAME = "CatSettings.json"
local CAN_SAVE = type(writefile) == "function" and type(readfile) == "function" and type(isfile) == "function"
local DEFAULT_THEME = Color3.fromRGB(120, 105, 255)

local CONFIG = {
    AvatarId = "139350569952709",
    Theme = DEFAULT_THEME,
    MenuSize = 1,
    Transparency = 0.12,
    ToggleKey = Enum.KeyCode.RightShift,
    Notifications = true,
    ShowButton = true
}

local FontState = {
    Spec = nil,
    Key = nil,
    Game = true,
    Menu = true
}

local Saved = {
    Skies = {},
    Fonts = {},
    ActiveFont = nil
}

local function trim(value)
    return (string.match(tostring(value), "^%s*(.-)%s*$"))
end

local function normalizeId(value)
    local source = tostring(value)
    return source:match("^%s*(%d+)%s*$") or source:match("rbxassetid://(%d+)") or source:match("[?&]id=(%d+)")
end

local function parseHex(value)
    local hex = tostring(value):gsub("#", ""):gsub("%s", "")

    if #hex ~= 6 or not hex:match("^%x+$") then
        return nil
    end

    return Color3.fromRGB(
        tonumber(hex:sub(1, 2), 16),
        tonumber(hex:sub(3, 4), 16),
        tonumber(hex:sub(5, 6), 16)
    )
end

local function toHex(color)
    return string.format(
        "#%02X%02X%02X",
        math.round(color.R * 255),
        math.round(color.G * 255),
        math.round(color.B * 255)
    )
end

local function loadProfile()
    if not CAN_SAVE then
        return
    end

    local ok, data = pcall(function()
        if isfile(FILE_NAME) then
            return HttpService:JSONDecode(readfile(FILE_NAME))
        end

        return nil
    end)

    if not ok or type(data) ~= "table" then
        return
    end

    if type(data.Theme) == "table" then
        local r = tonumber(data.Theme[1])
        local g = tonumber(data.Theme[2])
        local b = tonumber(data.Theme[3])

        if r and g and b then
            CONFIG.Theme = Color3.fromRGB(math.clamp(r, 0, 255), math.clamp(g, 0, 255), math.clamp(b, 0, 255))
        end
    end

    if type(data.MenuSize) == "number" then
        CONFIG.MenuSize = math.clamp(data.MenuSize, 0.7, 1.2)
    end

    if type(data.Transparency) == "number" then
        CONFIG.Transparency = math.clamp(data.Transparency, 0, 0.5)
    end

    if type(data.Notifications) == "boolean" then
        CONFIG.Notifications = data.Notifications
    end

    if type(data.ShowButton) == "boolean" then
        CONFIG.ShowButton = data.ShowButton
    end

    if type(data.Key) == "string" then
        local found, key = pcall(function()
            return Enum.KeyCode[data.Key]
        end)

        if found and key then
            CONFIG.ToggleKey = key
        end
    end

    if type(data.GameText) == "boolean" then
        FontState.Game = data.GameText
    end

    if type(data.MenuText) == "boolean" then
        FontState.Menu = data.MenuText
    end

    if type(data.ActiveFont) == "string" then
        Saved.ActiveFont = data.ActiveFont
    end

    if type(data.Skies) == "table" then
        for _, entry in ipairs(data.Skies) do
            if type(entry) == "table" and type(entry.Name) == "string" and type(entry.Ids) == "table" then
                local ids = {}

                for _, rawId in ipairs(entry.Ids) do
                    local clean = normalizeId(rawId)

                    if clean then
                        table.insert(ids, clean)
                    end
                end

                if #ids == 1 or #ids == 6 then
                    table.insert(Saved.Skies, {Name = entry.Name, Ids = ids})
                end
            end
        end
    end

    if type(data.Fonts) == "table" then
        for _, entry in ipairs(data.Fonts) do
            if type(entry) == "table" and type(entry.Name) == "string" and type(entry.Value) == "string" then
                table.insert(Saved.Fonts, {Name = entry.Name, Value = entry.Value})
            end
        end
    end
end

loadProfile()

local Colors = {
    Background = Color3.fromRGB(15, 17, 25),
    Panel = Color3.fromRGB(23, 26, 37),
    Card = Color3.fromRGB(31, 35, 49),
    Text = Color3.fromRGB(245, 246, 255),
    SubText = Color3.fromRGB(153, 160, 180),
    Accent = CONFIG.Theme,
    Green = Color3.fromRGB(90, 220, 155),
    Yellow = Color3.fromRGB(255, 205, 90),
    Red = Color3.fromRGB(255, 100, 110),
    Off = Color3.fromRGB(65, 68, 82)
}

local function weak()
    return setmetatable({}, {__mode = "k"})
end

local OriginalLighting = {}
local OriginalObjects = weak()
local OriginalWater = {}
local GameTexts = weak()
local MenuTexts = weak()
local HiddenSky = {}
local HiddenAtmosphere = {}
local Connections = {}
local LifetimeConnections = {}
local AccentStrokes = {}
local Scanned = {}
local NavButtons = {}
local Pages = {}
local PageNames = {"Performance", "Sky", "Lighting", "Effects", "Fonts", "Interface", "General"}

local startCamera = workspace.CurrentCamera

local Settings = {
    FixLag = false,
    Shadows = false,
    Particles = false,
    Textures = false,
    PostEffects = false,
    Materials = false,
    SimpleWater = false,
    LowQuality = false,
    FPS = false,
    Ping = false,
    Memory = false,
    Fog = true,
    LockTime = false,
    LockedTime = Lighting.ClockTime,
    Bloom = false,
    Blur = false,
    SunRays = false,
    DoF = false,
    CustomFOV = false,
    FOV = startCamera and startCamera.FieldOfView or 70
}

local KindFlag = {
    Emitter = "Particles",
    Post = "PostEffects",
    Decal = "Textures",
    Part = "Materials"
}

local ScreenGui
local Main
local Scale
local MainStroke
local Sidebar
local Navigation
local Content
local Header
local PageHolder
local PageTitle
local OpenButton
local Toast
local MinimizeButton
local CloseButton

local CurrentPage
local openPage
local refreshPage
local shutdown
local resetPositions

local shuttingDown = false
local minimized = false
local menuAnimationId = 0
local layoutOrder = 0
local SweepToken = 0
local ToastToken = 0
local SaveToken = 0

local BoldMap = {
    Gotham = "GothamBold",
    GothamSemibold = "GothamBold",
    GothamBlack = "GothamBlack",
    SourceSans = "SourceSansBold",
    SourceSansLight = "SourceSansBold",
    SourceSansSemibold = "SourceSansBold",
    Arial = "ArialBold"
}

local function create(class, properties, parent)
    local object = Instance.new(class)

    for property, value in pairs(properties) do
        object[property] = value
    end

    object.Parent = parent
    return object
end

local function corner(object, radius)
    local item = Instance.new("UICorner")
    item.CornerRadius = UDim.new(0, radius or 12)
    item.Parent = object
    return item
end

local function stroke(object, color, transparency)
    local item = Instance.new("UIStroke")
    item.Color = color or Color3.new(1, 1, 1)
    item.Transparency = transparency or 0.85
    item.Thickness = 1
    item.Parent = object
    return item
end

local function accentStroke(object, transparency)
    local item = stroke(object, Colors.Accent, transparency)
    table.insert(AccentStrokes, item)
    return item
end

local function padding(object, left)
    local item = Instance.new("UIPadding")
    item.PaddingLeft = UDim.new(0, left or 10)
    item.PaddingRight = UDim.new(0, 10)
    item.Parent = object
    return item
end

local function tween(object, properties, duration)
    if not object or not object.Parent then
        return
    end

    TweenService:Create(
        object,
        TweenInfo.new(duration or 0.2, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
        properties
    ):Play()
end

local function applySpec(object, spec)
    if spec.Enum then
        object.Font = spec.Enum
    else
        object.FontFace = spec.Face
    end
end

local function menuSpec(bold)
    local spec = FontState.Menu and FontState.Spec

    if not spec then
        return {Enum = bold and Enum.Font.GothamBold or Enum.Font.Gotham}
    end

    if spec.Enum then
        local boldName = BoldMap[spec.Enum.Name]

        if bold and boldName then
            return {Enum = Enum.Font[boldName]}
        end

        return spec
    end

    if bold then
        return {Face = Font.new(spec.Face.Family, Enum.FontWeight.Bold, spec.Face.Style)}
    end

    return spec
end

local function text(class, properties, parent, bold)
    local object = create(class, properties, parent)
    MenuTexts[object] = bold == true
    pcall(applySpec, object, menuSpec(bold == true))
    return object
end

local function applyMenuFont()
    for object, bold in pairs(MenuTexts) do
        if object.Parent then
            pcall(applySpec, object, menuSpec(bold))
        end
    end
end

local function notify(message)
    if shuttingDown or not CONFIG.Notifications or not Toast or not Toast.Parent then
        return
    end

    ToastToken += 1
    local id = ToastToken

    Toast.Text = message
    Toast.Visible = true
    Toast.Position = UDim2.new(0.5, 0, 0, -50)

    tween(Toast, {
        Position = UDim2.new(0.5, 0, 0, 14)
    }, 0.25)

    task.delay(2.2, function()
        if shuttingDown or id ~= ToastToken or not Toast.Parent then
            return
        end

        tween(Toast, {
            Position = UDim2.new(0.5, 0, 0, -60)
        }, 0.25)

        task.delay(0.3, function()
            if id == ToastToken and Toast.Parent then
                Toast.Visible = false
            end
        end)
    end)
end

local FontItems = {}

for _, item in ipairs(Enum.Font:GetEnumItems()) do
    if item.Name ~= "Legacy" and item.Name ~= "Unknown" then
        table.insert(FontItems, item)
    end
end

table.sort(FontItems, function(a, b)
    return a.Name < b.Name
end)

local function fontLoads(face)
    local ok = pcall(function()
        local params = Instance.new("GetTextBoundsParams")
        params.Text = "Test"
        params.Font = face
        params.Size = 14
        params.Width = 1000
        TextService:GetTextBoundsAsync(params)
    end)

    return ok
end

local function parseFont(value)
    value = trim(value)

    if value == "" then
        return nil
    end

    for _, item in ipairs(FontItems) do
        if item.Name:lower() == value:lower() then
            return {Enum = item, Key = "Enum:" .. item.Name}
        end
    end

    local id = normalizeId(value)
    local family

    if id then
        family = "rbxassetid://" .. id
    elseif value:find("^rbxasset") then
        family = value
    end

    if not family then
        return nil
    end

    local ok, face = pcall(Font.new, family, Enum.FontWeight.Regular, Enum.FontStyle.Normal)

    if not ok or not fontLoads(face) then
        return nil
    end

    return {Face = face, Key = "Face:" .. family}
end

local function specFromKey(key)
    local kind, value = string.match(key, "^(%a+):(.+)$")

    if kind == "Enum" then
        for _, item in ipairs(FontItems) do
            if item.Name == value then
                return {Enum = item, Key = key}
            end
        end
    elseif kind == "Face" then
        local ok, face = pcall(Font.new, value, Enum.FontWeight.Regular, Enum.FontStyle.Normal)

        if ok then
            return {Face = face, Key = key}
        end
    end

    return nil
end

local function scheduleSave()
    if not CAN_SAVE then
        return
    end

    SaveToken += 1
    local id = SaveToken

    task.delay(0.6, function()
        if id ~= SaveToken then
            return
        end

        local data = {
            Theme = {
                math.round(CONFIG.Theme.R * 255),
                math.round(CONFIG.Theme.G * 255),
                math.round(CONFIG.Theme.B * 255)
            },
            MenuSize = CONFIG.MenuSize,
            Transparency = CONFIG.Transparency,
            Key = CONFIG.ToggleKey.Name,
            Notifications = CONFIG.Notifications,
            ShowButton = CONFIG.ShowButton,
            Skies = Saved.Skies,
            Fonts = Saved.Fonts,
            ActiveFont = FontState.Key,
            GameText = FontState.Game,
            MenuText = FontState.Menu
        }

        pcall(function()
            writefile(FILE_NAME, HttpService:JSONEncode(data))
        end)
    end)
end

local function makeDraggable(object, handle)
    local dragging = false
    local dragInput
    local dragStart
    local startPosition
    local moved = false

    table.insert(LifetimeConnections, handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            moved = false
            dragInput = input
            dragStart = input.Position
            startPosition = object.Position
        end
    end))

    table.insert(LifetimeConnections, UserInputService.InputChanged:Connect(function(input)
        if not dragging then
            return
        end

        if input.UserInputType == Enum.UserInputType.MouseMovement or input == dragInput then
            local delta = input.Position - dragStart

            if not moved and delta.Magnitude < 6 then
                return
            end

            moved = true

            object.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end))

    table.insert(LifetimeConnections, UserInputService.InputEnded:Connect(function(input)
        if input == dragInput or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            dragInput = nil
        end
    end))

    return function()
        return moved
    end
end

for _, property in ipairs({
    "Brightness",
    "ClockTime",
    "Ambient",
    "OutdoorAmbient",
    "ColorShift_Top",
    "ColorShift_Bottom",
    "FogColor",
    "FogStart",
    "FogEnd",
    "ExposureCompensation",
    "GlobalShadows",
    "EnvironmentDiffuseScale",
    "EnvironmentSpecularScale"
}) do
    local ok, value = pcall(function()
        return Lighting[property]
    end)

    if ok then
        OriginalLighting[property] = value
    end
end

local Terrain = workspace:FindFirstChildOfClass("Terrain")

if Terrain then
    for _, property in ipairs({"WaterWaveSize", "WaterWaveSpeed", "WaterReflectance"}) do
        pcall(function()
            OriginalWater[property] = Terrain[property]
        end)
    end
end

local OriginalQuality

pcall(function()
    OriginalQuality = settings().Rendering.QualityLevel
end)

ScreenGui = create("ScreenGui", {
    Name = "CatSettings",
    ResetOnSpawn = false,
    DisplayOrder = 50,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling
}, PlayerGui)

Main = create("Frame", {
    Name = "SettingsPanel",
    Size = UDim2.fromOffset(700, 460),
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    BackgroundColor3 = Colors.Background,
    BackgroundTransparency = CONFIG.Transparency,
    BorderSizePixel = 0,
    ClipsDescendants = true,
    Visible = false
}, ScreenGui)

corner(Main, 18)
MainStroke = accentStroke(Main, 0.5)

Scale = create("UIScale", {
    Scale = CONFIG.MenuSize
}, Main)

local function updateResponsiveScale()
    local camera = workspace.CurrentCamera

    if not camera then
        Scale.Scale = CONFIG.MenuSize
        return
    end

    local viewport = camera.ViewportSize
    local fitX = math.max(0.4, (viewport.X - 24) / 700)
    local fitY = math.max(0.4, (viewport.Y - 24) / 460)
    Scale.Scale = math.min(CONFIG.MenuSize, fitX, fitY)
end

local CameraConnection

local function watchCamera()
    if CameraConnection then
        CameraConnection:Disconnect()
        CameraConnection = nil
    end

    local camera = workspace.CurrentCamera

    if camera then
        CameraConnection = camera:GetPropertyChangedSignal("ViewportSize"):Connect(updateResponsiveScale)
    end

    updateResponsiveScale()
end

Sidebar = create("Frame", {
    Size = UDim2.new(0, 175, 1, 0),
    BackgroundColor3 = Colors.Panel,
    BorderSizePixel = 0
}, Main)

corner(Sidebar, 18)

create("Frame", {
    Size = UDim2.new(0, 18, 1, 0),
    Position = UDim2.new(1, -18, 0, 0),
    BackgroundColor3 = Colors.Panel,
    BorderSizePixel = 0
}, Sidebar)

local function imageFallback(image, size)
    local label = text("TextLabel", {
        Size = UDim2.fromScale(1, 1),
        BackgroundTransparency = 1,
        Text = "CAT",
        TextColor3 = Colors.Text,
        TextSize = size,
        ZIndex = 2,
        Visible = not image.IsLoaded
    }, image, true)

    image:GetPropertyChangedSignal("IsLoaded"):Connect(function()
        label.Visible = not image.IsLoaded
    end)
end

local Avatar = create("ImageButton", {
    Size = UDim2.fromOffset(54, 54),
    Position = UDim2.fromOffset(18, 20),
    BackgroundColor3 = Colors.Card,
    Image = "rbxassetid://" .. CONFIG.AvatarId,
    ScaleType = Enum.ScaleType.Crop,
    AutoButtonColor = false
}, Sidebar)

corner(Avatar, 16)
imageFallback(Avatar, 14)

text("TextLabel", {
    Size = UDim2.new(1, -25, 0, 24),
    Position = UDim2.fromOffset(18, 82),
    BackgroundTransparency = 1,
    Text = "CAT SETTINGS",
    TextColor3 = Colors.Text,
    TextSize = 15,
    TextXAlignment = Enum.TextXAlignment.Left
}, Sidebar, true)

Navigation = create("Frame", {
    Size = UDim2.new(1, -20, 1, -126),
    Position = UDim2.fromOffset(10, 116),
    BackgroundTransparency = 1
}, Sidebar)

create("UIListLayout", {
    Padding = UDim.new(0, 6),
    SortOrder = Enum.SortOrder.LayoutOrder
}, Navigation)

Content = create("Frame", {
    Size = UDim2.new(1, -195, 1, -28),
    Position = UDim2.fromOffset(185, 14),
    BackgroundTransparency = 1
}, Main)

Header = create("Frame", {
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundTransparency = 1
}, Content)

PageTitle = text("TextLabel", {
    Size = UDim2.new(1, -100, 0, 32),
    Position = UDim2.fromOffset(4, 0),
    BackgroundTransparency = 1,
    Text = "Performance",
    TextColor3 = Colors.Text,
    TextSize = 20,
    TextXAlignment = Enum.TextXAlignment.Left
}, Header, true)

MinimizeButton = text("TextButton", {
    Size = UDim2.fromOffset(32, 32),
    Position = UDim2.new(1, -76, 0, 0),
    BackgroundColor3 = Colors.Card,
    Text = "−",
    TextColor3 = Colors.Text,
    TextSize = 20,
    AutoButtonColor = false
}, Header, true)

corner(MinimizeButton, 9)

CloseButton = text("TextButton", {
    Size = UDim2.fromOffset(32, 32),
    Position = UDim2.new(1, -36, 0, 0),
    BackgroundColor3 = Colors.Card,
    Text = "×",
    TextColor3 = Colors.Text,
    TextSize = 20,
    AutoButtonColor = false
}, Header, true)

corner(CloseButton, 9)

PageHolder = create("ScrollingFrame", {
    Size = UDim2.new(1, 0, 1, -55),
    Position = UDim2.fromOffset(0, 55),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 3,
    ScrollBarImageColor3 = Colors.Accent,
    ScrollingDirection = Enum.ScrollingDirection.Y,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    CanvasSize = UDim2.new()
}, Content)

create("UIListLayout", {
    Padding = UDim.new(0, 8),
    SortOrder = Enum.SortOrder.LayoutOrder
}, PageHolder)

create("UIPadding", {
    PaddingBottom = UDim.new(0, 12)
}, PageHolder)

local StatLabels = {}
local StatPositions = {}

local function makeStat(key, y, initialText)
    local position = UDim2.new(1, -125, 0, y)

    local label = text("TextLabel", {
        Size = UDim2.fromOffset(110, 28),
        Position = position,
        BackgroundColor3 = Colors.Card,
        BackgroundTransparency = 0.1,
        Text = initialText,
        TextColor3 = Colors.Green,
        TextSize = 12,
        Visible = false
    }, ScreenGui, true)

    corner(label, 9)
    makeDraggable(label, label)

    StatLabels[key] = label
    StatPositions[key] = position
end

makeStat("FPS", 12, "FPS: --")
makeStat("Ping", 46, "PING: --")
makeStat("Memory", 80, "MEM: --")

Toast = text("TextLabel", {
    Size = UDim2.fromOffset(280, 40),
    AnchorPoint = Vector2.new(0.5, 0),
    Position = UDim2.new(0.5, 0, 0, -60),
    BackgroundColor3 = Colors.Card,
    BackgroundTransparency = 0.05,
    TextColor3 = Colors.Text,
    Text = "",
    TextSize = 13,
    ZIndex = 20,
    Visible = false
}, ScreenGui, false)

corner(Toast, 12)
accentStroke(Toast, 0.6)

local function getAtmosphereObject()
    local cat = Lighting:FindFirstChild("CatAtmosphere")

    if cat then
        return cat
    end

    return Lighting:FindFirstChildOfClass("Atmosphere")
end

local function getSkyObject()
    local cat = Lighting:FindFirstChild("CatSky")

    if cat then
        return cat
    end

    return Lighting:FindFirstChildOfClass("Sky")
end

local function readProperty(object, property, default)
    if object then
        local ok, value = pcall(function()
            return object[property]
        end)

        if ok and value ~= nil then
            return value
        end
    end

    return default
end

local function hideGame(className, store, keepName)
    for _, object in ipairs(Lighting:GetChildren()) do
        if object:IsA(className) and object.Name ~= keepName then
            table.insert(store, object)
            object.Parent = nil
        end
    end
end

local SkyProps = {
    "SkyboxBk",
    "SkyboxDn",
    "SkyboxFt",
    "SkyboxLf",
    "SkyboxRt",
    "SkyboxUp",
    "StarCount",
    "CelestialBodiesShown",
    "SunAngularSize",
    "MoonAngularSize",
    "SunTextureId",
    "MoonTextureId"
}

local AtmosphereProps = {"Density", "Offset", "Color", "Decay", "Glare", "Haze"}

local function getCatSky(fresh)
    local existing = Lighting:FindFirstChild("CatSky")

    if existing and not fresh then
        return existing
    end

    if existing then
        existing:Destroy()
    end

    local sky = Instance.new("Sky")
    sky.Name = "CatSky"

    if not fresh then
        local source = Lighting:FindFirstChildOfClass("Sky")

        if source then
            for _, property in ipairs(SkyProps) do
                pcall(function()
                    sky[property] = source[property]
                end)
            end
        end
    end

    hideGame("Sky", HiddenSky, "CatSky")
    sky.Parent = Lighting
    return sky
end

local function getCatAtmosphere()
    local existing = Lighting:FindFirstChild("CatAtmosphere")

    if existing then
        return existing
    end

    local atmosphere = Instance.new("Atmosphere")
    atmosphere.Name = "CatAtmosphere"

    local source = Lighting:FindFirstChildOfClass("Atmosphere")

    if source then
        for _, property in ipairs(AtmosphereProps) do
            pcall(function()
                atmosphere[property] = source[property]
            end)
        end
    end

    hideGame("Atmosphere", HiddenAtmosphere, "CatAtmosphere")
    atmosphere.Parent = Lighting
    return atmosphere
end

local function setAtmosphere(property, value)
    getCatAtmosphere()[property] = value
end

local function setClock(value)
    Lighting.ClockTime = value
    Settings.LockedTime = value
end

local function applyShadows()
    local base = OriginalLighting.GlobalShadows

    if base == nil then
        base = true
    end

    pcall(function()
        Lighting.GlobalShadows = base and not Settings.Shadows
    end)
end

local function applyFog(enabled)
    Settings.Fog = enabled

    if enabled then
        Lighting.FogStart = OriginalLighting.FogStart or Lighting.FogStart
        Lighting.FogEnd = OriginalLighting.FogEnd or Lighting.FogEnd
    else
        Lighting.FogStart = 100000
        Lighting.FogEnd = 100000
    end
end

local SkyPresets = {
    {Name = "REALISTIC DAY", Clock = 13, Brightness = 3, Ambient = Color3.fromRGB(170, 180, 200), Outdoor = Color3.fromRGB(190, 195, 205), Fog = Color3.fromRGB(180, 215, 245)},
    {Name = "SUNRISE", Clock = 6.2, Brightness = 2.4, Ambient = Color3.fromRGB(150, 130, 140), Outdoor = Color3.fromRGB(200, 160, 150), Fog = Color3.fromRGB(255, 190, 150)},
    {Name = "GOLDEN HOUR", Clock = 17, Brightness = 2.8, Ambient = Color3.fromRGB(180, 140, 115), Outdoor = Color3.fromRGB(210, 155, 120), Fog = Color3.fromRGB(255, 180, 125)},
    {Name = "SUNSET", Clock = 18.3, Brightness = 2.5, Ambient = Color3.fromRGB(145, 105, 130), Outdoor = Color3.fromRGB(190, 130, 140), Fog = Color3.fromRGB(245, 145, 135)},
    {Name = "EVENING", Clock = 19.5, Brightness = 2, Ambient = Color3.fromRGB(100, 105, 145), Outdoor = Color3.fromRGB(125, 135, 180), Fog = Color3.fromRGB(100, 120, 175)},
    {Name = "NIGHT", Clock = 0, Brightness = 1.5, Ambient = Color3.fromRGB(65, 75, 115), Outdoor = Color3.fromRGB(90, 105, 150), Fog = Color3.fromRGB(35, 45, 90)},
    {Name = "MOONLIGHT", Clock = 0, Brightness = 1.2, Ambient = Color3.fromRGB(50, 60, 100), Outdoor = Color3.fromRGB(70, 85, 130), Fog = Color3.fromRGB(25, 35, 75)},
    {Name = "CLOUDY", Clock = 12, Brightness = 2, Ambient = Color3.fromRGB(130, 140, 150), Outdoor = Color3.fromRGB(160, 170, 180), Fog = Color3.fromRGB(145, 160, 180)}
}

local SpacePresets = {
    {Name = "MILKY WAY", Color = Color3.fromRGB(55, 65, 110)},
    {Name = "DEEP SPACE", Color = Color3.fromRGB(35, 40, 85)},
    {Name = "NEBULA", Color = Color3.fromRGB(100, 55, 140)},
    {Name = "AURORA", Color = Color3.fromRGB(40, 115, 110)},
    {Name = "GALAXY", Color = Color3.fromRGB(80, 45, 125)}
}

local EffectDefs = {
    Bloom = {Class = "BloomEffect", Name = "CatBloom", Defaults = {Intensity = 0.35, Size = 32, Threshold = 1}},
    Blur = {Class = "BlurEffect", Name = "CatBlur", Defaults = {Size = 8}},
    SunRays = {Class = "SunRaysEffect", Name = "CatSunRays", Defaults = {Intensity = 0.25, Spread = 1}},
    DoF = {Class = "DepthOfFieldEffect", Name = "CatDoF", Defaults = {FarIntensity = 0.3, FocusDistance = 50, InFocusRadius = 30, NearIntensity = 0}}
}

local EffectValues = {}

for key, definition in pairs(EffectDefs) do
    EffectValues[key] = table.clone(definition.Defaults)
end

local ColorValues = {
    Brightness = 0,
    Contrast = 0,
    Saturation = 0,
    TintColor = Color3.new(1, 1, 1)
}

local function setEffect(key, enabled)
    local definition = EffectDefs[key]
    local effect = Lighting:FindFirstChild(definition.Name)
    Settings[key] = enabled

    if enabled then
        if not effect then
            effect = Instance.new(definition.Class)
            effect.Name = definition.Name
            effect.Parent = Lighting
        end

        for property, value in pairs(EffectValues[key]) do
            effect[property] = value
        end
    elseif effect then
        effect:Destroy()
    end
end

local function setEffectValue(key, property, value)
    EffectValues[key][property] = value

    local effect = Lighting:FindFirstChild(EffectDefs[key].Name)

    if effect then
        effect[property] = value
    end
end

local function setColorValue(property, value)
    ColorValues[property] = value

    local effect = Lighting:FindFirstChild("CatColorCorrection")

    if not effect then
        effect = Instance.new("ColorCorrectionEffect")
        effect.Name = "CatColorCorrection"
        effect.Parent = Lighting
    end

    for name, current in pairs(ColorValues) do
        effect[name] = current
    end
end

local OriginalFOV

local function setFOV(enabled)
    local camera = workspace.CurrentCamera
    Settings.CustomFOV = enabled

    if enabled then
        if camera and not OriginalFOV then
            OriginalFOV = camera.FieldOfView
        end
    elseif camera and OriginalFOV then
        camera.FieldOfView = OriginalFOV
        OriginalFOV = nil
    end
end

local function applySkyPreset(preset)
    getCatSky(true)

    setClock(preset.Clock)
    Lighting.Brightness = preset.Brightness
    Lighting.Ambient = preset.Ambient
    Lighting.OutdoorAmbient = preset.Outdoor
    Lighting.FogColor = preset.Fog

    local atmosphere = getCatAtmosphere()
    atmosphere.Density = 0.25
    atmosphere.Haze = 1
    atmosphere.Glare = 0.2
    atmosphere.Offset = 0.1
    atmosphere.Color = preset.Fog
    atmosphere.Decay = Color3.fromRGB(90, 100, 130)

    notify(preset.Name .. " applied")
    refreshPage()
end

local function applySpace(preset)
    local sky = getCatSky(true)
    sky.StarCount = 5000
    sky.SunAngularSize = 0
    sky.MoonAngularSize = 11
    sky.CelestialBodiesShown = true

    setClock(0)
    Lighting.Brightness = 1.5
    Lighting.Ambient = preset.Color
    Lighting.OutdoorAmbient = preset.Color
    Lighting.FogColor = preset.Color

    local atmosphere = getCatAtmosphere()
    atmosphere.Density = 0.05
    atmosphere.Haze = 0
    atmosphere.Color = preset.Color

    notify(preset.Name .. " applied")
    refreshPage()
end

local function applyCustomSky(entry)
    local sky = getCatSky(true)
    local faces = {"SkyboxBk", "SkyboxDn", "SkyboxFt", "SkyboxLf", "SkyboxRt", "SkyboxUp"}

    for index, face in ipairs(faces) do
        local id = entry.Ids[index] or entry.Ids[1]
        sky[face] = "rbxassetid://" .. id
    end

    notify(entry.Name .. " applied")
    refreshPage()
end

local function parseIds(value)
    local ids = {}

    for token in tostring(value):gmatch("[^,%s;]+") do
        local id = normalizeId(token)

        if not id then
            return nil
        end

        table.insert(ids, id)
    end

    if #ids == 1 or #ids == 6 then
        return ids
    end

    return nil
end

local function restoreLighting(silent)
    local catSky = Lighting:FindFirstChild("CatSky")

    if catSky then
        catSky:Destroy()
    end

    local catAtmosphere = Lighting:FindFirstChild("CatAtmosphere")

    if catAtmosphere then
        catAtmosphere:Destroy()
    end

    for _, object in ipairs(HiddenSky) do
        pcall(function()
            object.Parent = Lighting
        end)
    end

    for _, object in ipairs(HiddenAtmosphere) do
        pcall(function()
            object.Parent = Lighting
        end)
    end

    table.clear(HiddenSky)
    table.clear(HiddenAtmosphere)

    for property, value in pairs(OriginalLighting) do
        pcall(function()
            Lighting[property] = value
        end)
    end

    Settings.Fog = true
    Settings.LockTime = false
    Settings.LockedTime = Lighting.ClockTime
    applyShadows()

    if not silent then
        notify("Lighting restored")
    end
end

local function restoreEffects()
    for key in pairs(EffectDefs) do
        setEffect(key, false)
        EffectValues[key] = table.clone(EffectDefs[key].Defaults)
    end

    local correction = Lighting:FindFirstChild("CatColorCorrection")

    if correction then
        correction:Destroy()
    end

    ColorValues.Brightness = 0
    ColorValues.Contrast = 0
    ColorValues.Saturation = 0
    ColorValues.TintColor = Color3.new(1, 1, 1)

    setFOV(false)
end

local function graphicsKind(object)
    if object:IsA("ParticleEmitter") or object:IsA("Trail") or object:IsA("Beam") then
        return "Emitter"
    end

    if object:IsA("PostEffect") then
        if object.Name:sub(1, 3) == "Cat" then
            return nil
        end

        return "Post"
    end

    if object:IsA("Decal") then
        if object.Name:lower() == "face" then
            return nil
        end

        return "Decal"
    end

    if Settings.Materials and object:IsA("BasePart") and not object:IsA("Terrain") then
        return "Part"
    end

    return nil
end

local function captureObject(object, kind)
    local existing = OriginalObjects[object]

    if existing then
        return existing
    end

    local data

    if kind == "Emitter" or kind == "Post" then
        data = {Kind = kind, Enabled = object.Enabled}
    elseif kind == "Decal" then
        data = {Kind = kind, Transparency = object.Transparency}
    elseif kind == "Part" then
        data = {Kind = kind, Material = object.Material, Reflectance = object.Reflectance}
    end

    OriginalObjects[object] = data
    return data
end

local function applyData(object, data)
    local kind = data.Kind

    if kind == "Emitter" then
        object.Enabled = data.Enabled and not Settings.Particles
    elseif kind == "Post" then
        object.Enabled = data.Enabled and not Settings.PostEffects
    elseif kind == "Decal" then
        object.Transparency = Settings.Textures and 1 or data.Transparency
    elseif kind == "Part" then
        if Settings.Materials then
            object.Material = Enum.Material.SmoothPlastic
            object.Reflectance = 0
        else
            object.Material = data.Material
            object.Reflectance = data.Reflectance
        end
    end
end

local function scanKind(kind)
    task.spawn(function()
        local count = 0

        for _, object in ipairs(game:GetDescendants()) do
            if shuttingDown or not Scanned[kind] then
                return
            end

            if graphicsKind(object) == kind then
                pcall(function()
                    local data = captureObject(object, kind)

                    if data then
                        applyData(object, data)
                    end
                end)
            end

            count += 1

            if count % 2000 == 0 then
                task.wait()
            end
        end
    end)
end

local function setKind(kind)
    if Settings[KindFlag[kind]] then
        if not Scanned[kind] then
            Scanned[kind] = true
            scanKind(kind)
        end
    else
        Scanned[kind] = nil

        for object, data in pairs(OriginalObjects) do
            if data.Kind == kind then
                if object.Parent then
                    pcall(applyData, object, data)
                end

                OriginalObjects[object] = nil
            end
        end
    end
end

local function applyWater()
    if not Terrain then
        return
    end

    pcall(function()
        if Settings.SimpleWater then
            Terrain.WaterWaveSize = 0
            Terrain.WaterWaveSpeed = 0
            Terrain.WaterReflectance = 0
        else
            for property, value in pairs(OriginalWater) do
                Terrain[property] = value
            end
        end
    end)
end

local function applyQuality()
    pcall(function()
        local rendering = settings().Rendering

        if Settings.LowQuality then
            rendering.QualityLevel = Enum.QualityLevel.Level01
        elseif OriginalQuality then
            rendering.QualityLevel = OriginalQuality
        end
    end)
end

local function setFixLag(value)
    Settings.FixLag = value
    Settings.Shadows = value
    Settings.Particles = value
    Settings.PostEffects = value
    Settings.Materials = value
    Settings.SimpleWater = value
    Settings.LowQuality = value

    applyShadows()

    for kind in pairs(KindFlag) do
        setKind(kind)
    end

    applyWater()
    applyQuality()
    refreshPage()
    notify(value and "Fix Lag enabled" or "Fix Lag disabled")
end

local function restoreGraphics(silent)
    Settings.FixLag = false
    Settings.Shadows = false
    Settings.Particles = false
    Settings.Textures = false
    Settings.PostEffects = false
    Settings.Materials = false
    Settings.SimpleWater = false
    Settings.LowQuality = false

    for kind in pairs(KindFlag) do
        setKind(kind)
    end

    applyShadows()
    applyWater()
    applyQuality()

    if not silent then
        notify("Graphics restored")
    end
end

local function isGameText(object)
    return (object:IsA("TextLabel") or object:IsA("TextButton") or object:IsA("TextBox"))
        and not object:IsDescendantOf(ScreenGui)
end

local function styleGameText(object)
    local spec = FontState.Spec

    if not spec or not FontState.Game then
        return
    end

    if GameTexts[object] == nil then
        local ok, face = pcall(function()
            return object.FontFace
        end)

        if not ok then
            return
        end

        GameTexts[object] = face
    end

    pcall(applySpec, object, spec)
end

local function sweepGameText()
    SweepToken += 1
    local token = SweepToken

    task.spawn(function()
        local count = 0

        for _, root in ipairs({PlayerGui, workspace}) do
            for _, object in ipairs(root:GetDescendants()) do
                if token ~= SweepToken or shuttingDown then
                    return
                end

                if isGameText(object) then
                    styleGameText(object)
                end

                count += 1

                if count % 2000 == 0 then
                    task.wait()
                end
            end
        end
    end)
end

local function restoreGameText()
    SweepToken += 1

    for object, face in pairs(GameTexts) do
        if object.Parent then
            pcall(function()
                object.FontFace = face
            end)
        end
    end

    table.clear(GameTexts)
end

local function specMatches(object, spec)
    if spec.Enum then
        return object.Font == spec.Enum
    end

    local face = object.FontFace
    return face.Family == spec.Face.Family and face.Weight == spec.Face.Weight and face.Style == spec.Face.Style
end

local FontButtons = {}

local function refreshFontButtons()
    for key, button in pairs(FontButtons) do
        if button.Parent then
            local active = FontState.Key == key
            button.Text = active and "Active" or "Apply"
            button.BackgroundColor3 = active and Colors.Green or Colors.Accent
        end
    end
end

local function setFont(spec, silent)
    FontState.Spec = spec
    FontState.Key = spec and spec.Key or nil

    applyMenuFont()

    if spec then
        if FontState.Game then
            sweepGameText()
        end
    else
        restoreGameText()
    end

    refreshFontButtons()
    scheduleSave()

    if not silent then
        notify(spec and "Font applied" or "Font restored")
    end
end

local function setGameFont(enabled)
    FontState.Game = enabled

    if enabled and FontState.Spec then
        sweepGameText()
    else
        restoreGameText()
    end

    scheduleSave()
end

local function setMenuFont(enabled)
    FontState.Menu = enabled
    applyMenuFont()
    scheduleSave()
end

local function applyTheme(silent)
    Colors.Accent = CONFIG.Theme

    Main.BackgroundTransparency = CONFIG.Transparency
    PageHolder.ScrollBarImageColor3 = Colors.Accent

    for _, item in ipairs(AccentStrokes) do
        if item.Parent then
            item.Color = Colors.Accent
        end
    end

    updateResponsiveScale()
    scheduleSave()

    if not silent then
        notify("Theme updated")
    end
end

local function clearPage()
    layoutOrder = 0

    for _, child in ipairs(PageHolder:GetChildren()) do
        if child:IsA("GuiObject") then
            child:Destroy()
        end
    end
end

local function nextOrder()
    layoutOrder += 1
    return layoutOrder
end

local function addCard(height)
    local card = create("Frame", {
        Size = UDim2.new(1, -5, 0, height),
        BackgroundColor3 = Colors.Card,
        BackgroundTransparency = 0.08,
        BorderSizePixel = 0,
        LayoutOrder = nextOrder()
    }, PageHolder)

    corner(card, 12)
    stroke(card, Color3.new(1, 1, 1), 0.94)

    return card
end

local function addSection(name)
    text("TextLabel", {
        Size = UDim2.new(1, -5, 0, 24),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Accent,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Left,
        LayoutOrder = nextOrder()
    }, PageHolder, true)
end

local function addRow(name, buttons, previewFont)
    local card = addCard(48)
    local offset = 12
    local created = {}

    for index = #buttons, 1, -1 do
        local info = buttons[index]
        local width = info.Width or 78

        local button = text("TextButton", {
            Size = UDim2.fromOffset(width, 30),
            Position = UDim2.new(1, -(offset + width), 0.5, -15),
            BackgroundColor3 = info.Color or Colors.Accent,
            Text = info.Text,
            TextColor3 = Colors.Text,
            TextSize = 12,
            AutoButtonColor = false
        }, card, true)

        corner(button, 9)
        button.MouseButton1Click:Connect(info.Callback)

        created[index] = button
        offset += width + 8
    end

    local properties = {
        Size = UDim2.new(1, -(offset + 14), 1, 0),
        Position = UDim2.fromOffset(14, 0),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextTruncate = Enum.TextTruncate.AtEnd
    }

    if previewFont then
        properties.Font = previewFont
        create("TextLabel", properties, card)
    else
        text("TextLabel", properties, card, false)
    end

    return card, created
end

local function addButton(name, buttonText, callback, color)
    local _, created = addRow(name, {{Text = buttonText or "Apply", Callback = callback, Color = color}})
    return created[1]
end

local function addToggle(name, initial, callback)
    local card = addCard(48)
    local enabled = initial == true

    text("TextLabel", {
        Size = UDim2.new(1, -90, 1, 0),
        Position = UDim2.fromOffset(14, 0),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card, false)

    local toggle = create("TextButton", {
        Size = UDim2.fromOffset(46, 25),
        Position = UDim2.new(1, -60, 0.5, -12),
        BackgroundColor3 = enabled and Colors.Accent or Colors.Off,
        Text = "",
        AutoButtonColor = false
    }, card)

    corner(toggle, 20)

    local knob = create("Frame", {
        Size = UDim2.fromOffset(19, 19),
        Position = enabled and UDim2.new(1, -22, 0, 3) or UDim2.fromOffset(3, 3),
        BackgroundColor3 = Color3.new(1, 1, 1),
        BorderSizePixel = 0
    }, toggle)

    corner(knob, 20)

    local function update(value)
        enabled = value

        tween(toggle, {
            BackgroundColor3 = enabled and Colors.Accent or Colors.Off
        })

        tween(knob, {
            Position = enabled and UDim2.new(1, -22, 0, 3) or UDim2.fromOffset(3, 3)
        })

        callback(enabled)
    end

    toggle.MouseButton1Click:Connect(function()
        update(not enabled)
    end)

    return {
        Set = update,
        Get = function()
            return enabled
        end
    }
end

local function formatNumber(value)
    return tostring(math.floor(value * 1000 + 0.5) / 1000)
end

local function addSlider(name, minimum, maximum, initial, callback, step)
    step = step or 0.1

    local card = addCard(64)
    local value = math.clamp(initial, minimum, maximum)
    local range = maximum - minimum

    text("TextLabel", {
        Size = UDim2.new(1, -90, 0, 22),
        Position = UDim2.fromOffset(14, 6),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card, false)

    local valueLabel = text("TextLabel", {
        Size = UDim2.fromOffset(70, 22),
        Position = UDim2.new(1, -84, 0, 6),
        BackgroundTransparency = 1,
        Text = formatNumber(value),
        TextColor3 = Colors.Accent,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Right
    }, card, false)

    local hit = create("Frame", {
        Size = UDim2.new(1, -28, 0, 26),
        Position = UDim2.fromOffset(14, 32),
        BackgroundTransparency = 1,
        Active = true
    }, card)

    local bar = create("Frame", {
        Size = UDim2.new(1, 0, 0, 6),
        Position = UDim2.new(0, 0, 0.5, -3),
        BackgroundColor3 = Color3.fromRGB(60, 64, 80),
        BorderSizePixel = 0
    }, hit)

    corner(bar, 10)

    local percent = (value - minimum) / range

    local fill = create("Frame", {
        Size = UDim2.new(percent, 0, 1, 0),
        BackgroundColor3 = Colors.Accent,
        BorderSizePixel = 0
    }, bar)

    corner(fill, 10)

    local knob = create("Frame", {
        Size = UDim2.fromOffset(15, 15),
        Position = UDim2.new(percent, -7, 0.5, -7),
        BackgroundColor3 = Color3.new(1, 1, 1),
        BorderSizePixel = 0
    }, bar)

    corner(knob, 20)

    local dragging = false
    local dragInput

    local function setValue(x)
        local ratio = math.clamp((x - hit.AbsolutePosition.X) / math.max(hit.AbsoluteSize.X, 1), 0, 1)
        local raw = minimum + range * ratio
        local snapped = math.clamp(math.floor(raw / step + 0.5) * step, minimum, maximum)
        snapped = math.floor(snapped * 1000 + 0.5) / 1000

        local shown = (snapped - minimum) / range
        fill.Size = UDim2.new(shown, 0, 1, 0)
        knob.Position = UDim2.new(shown, -7, 0.5, -7)
        valueLabel.Text = formatNumber(snapped)

        if snapped ~= value then
            value = snapped
            callback(value)
        end
    end

    table.insert(Connections, hit.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragInput = input
            setValue(input.Position.X)
        end
    end))

    table.insert(Connections, UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input == dragInput) then
            setValue(input.Position.X)
        end
    end))

    table.insert(Connections, UserInputService.InputEnded:Connect(function(input)
        if input == dragInput or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            dragInput = nil
        end
    end))
end

local function addForm(placeholders, buttonText, callback)
    local count = #placeholders
    local card = addCard(10 + count * 40)
    local boxes = {}

    for index, placeholder in ipairs(placeholders) do
        local isLast = index == count

        local box = text("TextBox", {
            Size = UDim2.new(1, isLast and -112 or -24, 0, 30),
            Position = UDim2.fromOffset(12, 10 + (index - 1) * 40),
            BackgroundColor3 = Colors.Panel,
            PlaceholderText = placeholder,
            PlaceholderColor3 = Colors.SubText,
            Text = "",
            TextColor3 = Colors.Text,
            TextSize = 12,
            ClearTextOnFocus = false,
            TextXAlignment = Enum.TextXAlignment.Left
        }, card, false)

        corner(box, 9)
        padding(box, 10)
        boxes[index] = box
    end

    local function submit()
        local values = {}

        for index, box in ipairs(boxes) do
            values[index] = box.Text
        end

        callback(values, boxes)
    end

    local button = text("TextButton", {
        Size = UDim2.fromOffset(80, 30),
        Position = UDim2.new(1, -92, 0, 10 + (count - 1) * 40),
        BackgroundColor3 = Colors.Accent,
        Text = buttonText,
        TextColor3 = Colors.Text,
        TextSize = 12,
        AutoButtonColor = false
    }, card, true)

    corner(button, 9)
    button.MouseButton1Click:Connect(submit)

    boxes[count].FocusLost:Connect(function(enterPressed)
        if enterPressed then
            submit()
        end
    end)

    return boxes
end

local function addTextInput(placeholder, onChange)
    local card = addCard(48)

    local box = text("TextBox", {
        Size = UDim2.new(1, -24, 0, 30),
        Position = UDim2.fromOffset(12, 9),
        BackgroundColor3 = Colors.Panel,
        PlaceholderText = placeholder,
        PlaceholderColor3 = Colors.SubText,
        Text = "",
        TextColor3 = Colors.Text,
        TextSize = 12,
        ClearTextOnFocus = false,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card, false)

    corner(box, 9)
    padding(box, 10)

    box:GetPropertyChangedSignal("Text"):Connect(function()
        onChange(box.Text)
    end)

    return box
end

local function addColorInput(name, initial, callback)
    local card = addCard(48)

    text("TextLabel", {
        Size = UDim2.new(1, -212, 1, 0),
        Position = UDim2.fromOffset(14, 0),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextTruncate = Enum.TextTruncate.AtEnd
    }, card, false)

    local preview = create("Frame", {
        Size = UDim2.fromOffset(22, 22),
        Position = UDim2.new(1, -190, 0.5, -11),
        BackgroundColor3 = initial,
        BorderSizePixel = 0
    }, card)

    corner(preview, 6)
    stroke(preview, Color3.new(1, 1, 1), 0.7)

    local box = text("TextBox", {
        Size = UDim2.fromOffset(90, 30),
        Position = UDim2.new(1, -160, 0.5, -15),
        BackgroundColor3 = Colors.Panel,
        Text = toHex(initial),
        TextColor3 = Colors.Text,
        TextSize = 12,
        ClearTextOnFocus = false
    }, card, false)

    corner(box, 9)

    local button = text("TextButton", {
        Size = UDim2.fromOffset(58, 30),
        Position = UDim2.new(1, -64, 0.5, -15),
        BackgroundColor3 = Colors.Accent,
        Text = "Set",
        TextColor3 = Colors.Text,
        TextSize = 12,
        AutoButtonColor = false
    }, card, true)

    corner(button, 9)

    local function submit()
        local color = parseHex(box.Text)

        if not color then
            notify("Invalid color")
            return
        end

        preview.BackgroundColor3 = color
        box.Text = toHex(color)
        callback(color)
    end

    button.MouseButton1Click:Connect(submit)

    box.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            submit()
        end
    end)
end

local function resetAll()
    restoreGraphics(true)
    restoreLighting(true)
    restoreEffects()
    restoreGameText()

    FontState.Spec = nil
    FontState.Key = nil
    FontState.Game = true
    FontState.Menu = true
    applyMenuFont()

    CONFIG.Theme = DEFAULT_THEME
    CONFIG.MenuSize = 1
    CONFIG.Transparency = 0.12
    CONFIG.ToggleKey = Enum.KeyCode.RightShift
    CONFIG.Notifications = true
    CONFIG.ShowButton = true

    Settings.FPS = false
    Settings.Ping = false
    Settings.Memory = false

    for _, label in pairs(StatLabels) do
        label.Visible = false
    end

    OpenButton.Visible = true

    applyTheme(true)
    resetPositions()
    refreshPage()
    scheduleSave()
    notify("Defaults restored")
end

Pages["Performance"] = function()
    addSection("PERFORMANCE")

    addToggle("Fix Lag", Settings.FixLag, setFixLag)

    addToggle("Show FPS", Settings.FPS, function(value)
        Settings.FPS = value
        StatLabels.FPS.Visible = value
    end)

    addToggle("Show Ping", Settings.Ping, function(value)
        Settings.Ping = value
        StatLabels.Ping.Visible = value
    end)

    addToggle("Show Memory", Settings.Memory, function(value)
        Settings.Memory = value
        StatLabels.Memory.Visible = value
    end)

    addSection("GRAPHICS")

    addToggle("Remove Shadows", Settings.Shadows, function(value)
        Settings.Shadows = value
        applyShadows()
    end)

    addToggle("Remove Particles", Settings.Particles, function(value)
        Settings.Particles = value
        setKind("Emitter")
    end)

    addToggle("Remove Textures", Settings.Textures, function(value)
        Settings.Textures = value
        setKind("Decal")
    end)

    addToggle("Remove Post Effects", Settings.PostEffects, function(value)
        Settings.PostEffects = value
        setKind("Post")
    end)

    addToggle("Smooth Materials", Settings.Materials, function(value)
        Settings.Materials = value
        setKind("Part")
    end)

    addToggle("Simple Water", Settings.SimpleWater, function(value)
        Settings.SimpleWater = value
        applyWater()
    end)

    addToggle("Low Quality Level", Settings.LowQuality, function(value)
        Settings.LowQuality = value
        applyQuality()
    end)

    addButton("Restore Graphics", "Restore", function()
        restoreGraphics()
        refreshPage()
    end)
end

Pages["Sky"] = function()
    addSection("PRESETS")

    for _, preset in ipairs(SkyPresets) do
        addButton(preset.Name, "Apply", function()
            applySkyPreset(preset)
        end)
    end

    addSection("SPACE")

    for _, preset in ipairs(SpacePresets) do
        addButton(preset.Name, "Apply", function()
            applySpace(preset)
        end)
    end

    addSection("CUSTOM SKY")

    addForm({"Name", "Asset ID (1 or 6, separated by comma)"}, "Add", function(values)
        local ids = parseIds(values[2])

        if not ids then
            notify("Need 1 or 6 valid IDs")
            return
        end

        local name = trim(values[1])

        if name == "" then
            name = "Sky " .. (#Saved.Skies + 1)
        end

        local entry = {Name = name, Ids = ids}
        table.insert(Saved.Skies, entry)
        scheduleSave()
        applyCustomSky(entry)
    end)

    for index, entry in ipairs(Saved.Skies) do
        addRow(entry.Name, {
            {
                Text = "Apply",
                Callback = function()
                    applyCustomSky(entry)
                end
            },
            {
                Text = "X",
                Width = 34,
                Color = Colors.Red,
                Callback = function()
                    table.remove(Saved.Skies, index)
                    scheduleSave()
                    refreshPage()
                end
            }
        })
    end

    addSection("SKY SETTINGS")

    local sky = getSkyObject()

    addSlider("Star Count", 0, 5000, readProperty(sky, "StarCount", 3000), function(value)
        getCatSky().StarCount = value
    end, 100)

    addSlider("Sun Size", 0, 60, readProperty(sky, "SunAngularSize", 21), function(value)
        getCatSky().SunAngularSize = value
    end, 1)

    addSlider("Moon Size", 0, 60, readProperty(sky, "MoonAngularSize", 11), function(value)
        getCatSky().MoonAngularSize = value
    end, 1)

    addToggle("Sun and Moon", readProperty(sky, "CelestialBodiesShown", true), function(value)
        getCatSky().CelestialBodiesShown = value
    end)

    addButton("Default Sky", "Restore", function()
        restoreLighting()
        refreshPage()
    end)
end

Pages["Lighting"] = function()
    addSection("TIME")

    addSlider("Time of Day", 0, 24, Lighting.ClockTime, function(value)
        setClock(value)
    end, 0.1)

    addToggle("Lock Time", Settings.LockTime, function(value)
        Settings.LockTime = value
        Settings.LockedTime = Lighting.ClockTime
    end)

    addSection("LIGHT")

    addSlider("Brightness", 0, 6, Lighting.Brightness, function(value)
        Lighting.Brightness = value
    end, 0.1)

    addSlider("Exposure", -3, 3, Lighting.ExposureCompensation, function(value)
        Lighting.ExposureCompensation = value
    end, 0.1)

    addSlider("Diffuse Scale", 0, 1, readProperty(Lighting, "EnvironmentDiffuseScale", 1), function(value)
        pcall(function()
            Lighting.EnvironmentDiffuseScale = value
        end)
    end, 0.05)

    addSlider("Specular Scale", 0, 1, readProperty(Lighting, "EnvironmentSpecularScale", 1), function(value)
        pcall(function()
            Lighting.EnvironmentSpecularScale = value
        end)
    end, 0.05)

    addToggle("Fog", Settings.Fog, applyFog)

    addSection("COLORS")

    addColorInput("Ambient", Lighting.Ambient, function(color)
        Lighting.Ambient = color
    end)

    addColorInput("Outdoor Ambient", Lighting.OutdoorAmbient, function(color)
        Lighting.OutdoorAmbient = color
    end)

    addColorInput("Fog Color", Lighting.FogColor, function(color)
        Lighting.FogColor = color
    end)

    addColorInput("Shift Top", Lighting.ColorShift_Top, function(color)
        Lighting.ColorShift_Top = color
    end)

    addColorInput("Shift Bottom", Lighting.ColorShift_Bottom, function(color)
        Lighting.ColorShift_Bottom = color
    end)

    addSection("ATMOSPHERE")

    local atmosphere = getAtmosphereObject()

    addSlider("Density", 0, 1, readProperty(atmosphere, "Density", 0.395), function(value)
        setAtmosphere("Density", value)
    end, 0.01)

    addSlider("Offset", 0, 1, readProperty(atmosphere, "Offset", 0), function(value)
        setAtmosphere("Offset", value)
    end, 0.01)

    addSlider("Haze", 0, 10, readProperty(atmosphere, "Haze", 0), function(value)
        setAtmosphere("Haze", value)
    end, 0.1)

    addSlider("Glare", 0, 10, readProperty(atmosphere, "Glare", 0), function(value)
        setAtmosphere("Glare", value)
    end, 0.1)

    addColorInput("Atmosphere Color", readProperty(atmosphere, "Color", Color3.fromRGB(199, 170, 107)), function(color)
        setAtmosphere("Color", color)
    end)

    addColorInput("Atmosphere Decay", readProperty(atmosphere, "Decay", Color3.fromRGB(92, 60, 13)), function(color)
        setAtmosphere("Decay", color)
    end)

    addButton("Restore Lighting", "Restore", function()
        restoreLighting()
        refreshPage()
    end)
end

Pages["Effects"] = function()
    addSection("BLOOM")

    addToggle("Bloom", Settings.Bloom, function(value)
        setEffect("Bloom", value)
    end)

    addSlider("Bloom Intensity", 0, 3, EffectValues.Bloom.Intensity, function(value)
        setEffectValue("Bloom", "Intensity", value)
    end, 0.05)

    addSlider("Bloom Size", 0, 56, EffectValues.Bloom.Size, function(value)
        setEffectValue("Bloom", "Size", value)
    end, 1)

    addSlider("Bloom Threshold", 0, 4, EffectValues.Bloom.Threshold, function(value)
        setEffectValue("Bloom", "Threshold", value)
    end, 0.05)

    addSection("BLUR")

    addToggle("Blur", Settings.Blur, function(value)
        setEffect("Blur", value)
    end)

    addSlider("Blur Size", 0, 56, EffectValues.Blur.Size, function(value)
        setEffectValue("Blur", "Size", value)
    end, 1)

    addSection("SUN RAYS")

    addToggle("Sun Rays", Settings.SunRays, function(value)
        setEffect("SunRays", value)
    end)

    addSlider("Rays Intensity", 0, 1, EffectValues.SunRays.Intensity, function(value)
        setEffectValue("SunRays", "Intensity", value)
    end, 0.01)

    addSlider("Rays Spread", 0, 1, EffectValues.SunRays.Spread, function(value)
        setEffectValue("SunRays", "Spread", value)
    end, 0.01)

    addSection("DEPTH OF FIELD")

    addToggle("Depth of Field", Settings.DoF, function(value)
        setEffect("DoF", value)
    end)

    addSlider("Far Intensity", 0, 1, EffectValues.DoF.FarIntensity, function(value)
        setEffectValue("DoF", "FarIntensity", value)
    end, 0.01)

    addSlider("Focus Distance", 0, 200, EffectValues.DoF.FocusDistance, function(value)
        setEffectValue("DoF", "FocusDistance", value)
    end, 1)

    addSlider("Focus Radius", 0, 100, EffectValues.DoF.InFocusRadius, function(value)
        setEffectValue("DoF", "InFocusRadius", value)
    end, 1)

    addSection("COLOR")

    addSlider("Color Brightness", -0.5, 0.5, ColorValues.Brightness, function(value)
        setColorValue("Brightness", value)
    end, 0.01)

    addSlider("Contrast", -1, 1, ColorValues.Contrast, function(value)
        setColorValue("Contrast", value)
    end, 0.05)

    addSlider("Saturation", -1, 1, ColorValues.Saturation, function(value)
        setColorValue("Saturation", value)
    end, 0.05)

    addColorInput("Tint", ColorValues.TintColor, function(color)
        setColorValue("TintColor", color)
    end)

    addSection("CAMERA")

    addToggle("Custom FOV", Settings.CustomFOV, setFOV)

    addSlider("Field of View", 30, 120, Settings.FOV, function(value)
        Settings.FOV = value
    end, 1)

    addButton("Reset Effects", "Reset", function()
        restoreEffects()
        refreshPage()
        notify("Effects reset")
    end)
end

Pages["Fonts"] = function()
    table.clear(FontButtons)

    addSection("TARGET")

    addToggle("Game Text", FontState.Game, setGameFont)
    addToggle("Menu Text", FontState.Menu, setMenuFont)

    addSection("CUSTOM FONT")

    addForm({"Font name or Font ID"}, "Add", function(values)
        local value = trim(values[1])
        local spec = parseFont(value)

        if not spec then
            notify("Font not found")
            return
        end

        if spec.Face then
            local exists = false

            for _, entry in ipairs(Saved.Fonts) do
                if entry.Value == value then
                    exists = true
                end
            end

            if not exists then
                table.insert(Saved.Fonts, {Name = value, Value = value})
            end
        end

        setFont(spec)
        refreshPage()
    end)

    for index, entry in ipairs(Saved.Fonts) do
        local spec = parseFont(entry.Value)

        if spec then
            local _, created = addRow(entry.Name, {
                {
                    Text = "Apply",
                    Callback = function()
                        setFont(spec)
                    end
                },
                {
                    Text = "X",
                    Width = 34,
                    Color = Colors.Red,
                    Callback = function()
                        table.remove(Saved.Fonts, index)
                        scheduleSave()
                        refreshPage()
                    end
                }
            })

            FontButtons[spec.Key] = created[1]
        end
    end

    addSection("FONTS")

    local rows = {}

    addTextInput("Search font", function(query)
        query = query:lower()

        for _, row in ipairs(rows) do
            row.Card.Visible = query == "" or row.Name:find(query, 1, true) ~= nil
        end
    end)

    addButton("Default", "Reset", function()
        setFont(nil)
    end, Colors.Red)

    for _, item in ipairs(FontItems) do
        local spec = {Enum = item, Key = "Enum:" .. item.Name}

        local card, created = addRow(item.Name, {
            {
                Text = "Apply",
                Callback = function()
                    setFont(spec)
                end
            }
        }, item)

        FontButtons[spec.Key] = created[1]
        table.insert(rows, {Card = card, Name = item.Name:lower()})
    end

    refreshFontButtons()
end

local Themes = {
    {"Purple", Color3.fromRGB(120, 105, 255)},
    {"Blue", Color3.fromRGB(70, 150, 255)},
    {"Cyan", Color3.fromRGB(55, 210, 220)},
    {"Pink", Color3.fromRGB(255, 105, 175)},
    {"Red", Color3.fromRGB(245, 85, 100)},
    {"Green", Color3.fromRGB(70, 200, 140)},
    {"Orange", Color3.fromRGB(255, 150, 75)},
    {"White", Color3.fromRGB(230, 230, 245)}
}

Pages["Interface"] = function()
    addSection("WINDOW")

    addSlider("Menu Size", 0.7, 1.2, CONFIG.MenuSize, function(value)
        CONFIG.MenuSize = value
        updateResponsiveScale()
        scheduleSave()
    end, 0.05)

    addSlider("Transparency", 0, 0.5, CONFIG.Transparency, function(value)
        CONFIG.Transparency = value
        Main.BackgroundTransparency = value
        scheduleSave()
    end, 0.01)

    addToggle("Notifications", CONFIG.Notifications, function(value)
        CONFIG.Notifications = value
        scheduleSave()
    end)

    addToggle("Show Button", CONFIG.ShowButton, function(value)
        CONFIG.ShowButton = value
        OpenButton.Visible = value
        scheduleSave()

        if not value then
            notify("Press " .. CONFIG.ToggleKey.Name .. " to open")
        end
    end)

    addForm({"Toggle key (" .. CONFIG.ToggleKey.Name .. ")"}, "Set", function(values)
        local name = trim(values[1])
        local ok, key = pcall(function()
            return Enum.KeyCode[name]
        end)

        if not ok or not key then
            notify("Invalid key")
            return
        end

        CONFIG.ToggleKey = key
        scheduleSave()
        notify("Key set to " .. key.Name)
        refreshPage()
    end)

    addButton("Reset Position", "Reset", function()
        resetPositions()
    end)

    addSection("THEME")

    for _, theme in ipairs(Themes) do
        addButton(theme[1], "Apply", function()
            CONFIG.Theme = theme[2]
            applyTheme()
            refreshPage()
        end, theme[2])
    end

    addColorInput("Custom Color", CONFIG.Theme, function(color)
        CONFIG.Theme = color
        applyTheme()
        refreshPage()
    end)
end

Pages["General"] = function()
    addSection("RESTORE")

    addButton("Restore Graphics", "Restore", function()
        restoreGraphics()
        refreshPage()
    end)

    addButton("Restore Lighting", "Restore", function()
        restoreLighting()
        refreshPage()
    end)

    addButton("Restore Effects", "Restore", function()
        restoreEffects()
        refreshPage()
        notify("Effects reset")
    end)

    addButton("Restore Fonts", "Restore", function()
        setFont(nil)
    end)

    addButton("Restore Defaults", "Reset", resetAll)

    addSection("SCRIPT")

    addButton("Unload Script", "Unload", function()
        shutdown()
    end, Colors.Red)
end

openPage = function(name, keepScroll)
    if not Pages[name] or shuttingDown then
        return
    end

    local position = PageHolder.CanvasPosition

    CurrentPage = name
    PageTitle.Text = name

    for _, connection in ipairs(Connections) do
        connection:Disconnect()
    end

    table.clear(Connections)

    clearPage()
    Pages[name]()

    for buttonName, button in pairs(NavButtons) do
        tween(button, {
            BackgroundColor3 = buttonName == name and Colors.Accent or Colors.Panel
        })
    end

    if keepScroll then
        task.spawn(function()
            RunService.Heartbeat:Wait()
            RunService.Heartbeat:Wait()

            if not shuttingDown and PageHolder.Parent then
                PageHolder.CanvasPosition = position
            end
        end)
    else
        PageHolder.CanvasPosition = Vector2.new(0, 0)
    end
end

refreshPage = function()
    if CurrentPage then
        openPage(CurrentPage, true)
    end
end

for index, name in ipairs(PageNames) do
    local button = text("TextButton", {
        Name = name,
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = Colors.Panel,
        Text = name,
        TextColor3 = Colors.Text,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Left,
        AutoButtonColor = false,
        LayoutOrder = index
    }, Navigation, false)

    corner(button, 9)
    padding(button, 12)

    button.MouseButton1Click:Connect(function()
        openPage(name)
    end)

    NavButtons[name] = button
end

OpenButton = create("ImageButton", {
    Name = "CatButton",
    Size = UDim2.fromOffset(58, 58),
    Position = UDim2.new(0, 25, 0.5, -29),
    BackgroundColor3 = Colors.Panel,
    Image = "rbxassetid://" .. CONFIG.AvatarId,
    ScaleType = Enum.ScaleType.Crop,
    AutoButtonColor = false,
    Visible = CONFIG.ShowButton
}, ScreenGui)

corner(OpenButton, 18)
accentStroke(OpenButton, 0.45)
imageFallback(OpenButton, 13)

local openMoved = makeDraggable(OpenButton, OpenButton)
makeDraggable(Main, Header)

resetPositions = function()
    Main.Position = UDim2.fromScale(0.5, 0.5)
    OpenButton.Position = UDim2.new(0, 25, 0.5, -29)

    for key, label in pairs(StatLabels) do
        label.Position = StatPositions[key]
    end

    notify("Position reset")
end

local function setMinimized(state)
    minimized = state

    if minimized then
        Sidebar.Visible = false
        PageHolder.Visible = false

        Content.Position = UDim2.fromOffset(12, 10)
        Content.Size = UDim2.new(1, -24, 0, 40)

        PageTitle.Text = "CAT SETTINGS"
        MinimizeButton.Text = "+"
        Main.Size = UDim2.fromOffset(250, 60)
    else
        Sidebar.Visible = true
        PageHolder.Visible = true

        Content.Position = UDim2.fromOffset(185, 14)
        Content.Size = UDim2.new(1, -195, 1, -28)

        MinimizeButton.Text = "−"
        Main.Size = UDim2.fromOffset(700, 460)

        openPage(CurrentPage or "Performance", true)
    end
end

local function openMenu()
    if shuttingDown then
        return
    end

    menuAnimationId += 1
    minimized = false

    Sidebar.Visible = true
    Content.Visible = true
    PageHolder.Visible = true

    Content.Position = UDim2.fromOffset(185, 14)
    Content.Size = UDim2.new(1, -195, 1, -28)

    MinimizeButton.Text = "−"

    Main.Visible = true
    Main.Size = UDim2.fromOffset(620, 400)

    tween(Main, {
        Size = UDim2.fromOffset(700, 460)
    }, 0.25)

    openPage(CurrentPage or "Performance", true)
end

local function closeMenu()
    if shuttingDown then
        return
    end

    menuAnimationId += 1
    local currentId = menuAnimationId

    tween(Main, {
        Size = UDim2.fromOffset(620, 400)
    }, 0.18)

    task.delay(0.2, function()
        if menuAnimationId == currentId and Main.Parent then
            Main.Visible = false
        end
    end)
end

local function toggleMenu()
    if Main.Visible then
        closeMenu()
    else
        openMenu()
    end
end

shutdown = function()
    if shuttingDown then
        return
    end

    shuttingDown = true
    menuAnimationId += 1
    SweepToken += 1

    for _, connection in ipairs(Connections) do
        connection:Disconnect()
    end

    for _, connection in ipairs(LifetimeConnections) do
        connection:Disconnect()
    end

    table.clear(Connections)
    table.clear(LifetimeConnections)

    if CameraConnection then
        CameraConnection:Disconnect()
        CameraConnection = nil
    end

    pcall(function()
        RunService:UnbindFromRenderStep("CatSettingsFOV")
    end)

    restoreGraphics(true)
    restoreLighting(true)
    restoreEffects()
    restoreGameText()

    if Shared.CatSettingsUnload == shutdown then
        Shared.CatSettingsUnload = nil
    end

    if ScreenGui then
        ScreenGui:Destroy()
    end
end

MinimizeButton.MouseButton1Click:Connect(function()
    setMinimized(not minimized)
end)

CloseButton.MouseButton1Click:Connect(closeMenu)

OpenButton.MouseButton1Click:Connect(function()
    if openMoved() then
        return
    end

    toggleMenu()
end)

table.insert(LifetimeConnections, UserInputService.InputBegan:Connect(function(input, processed)
    if processed or shuttingDown then
        return
    end

    if input.KeyCode == CONFIG.ToggleKey then
        toggleMenu()
    end
end))

local fpsFrames = 0
local fpsTime = 0
local statTimer = 0
local fontTimer = 0

table.insert(LifetimeConnections, RunService.Heartbeat:Connect(function(deltaTime)
    if shuttingDown then
        return
    end

    fpsFrames += 1
    fpsTime += deltaTime
    statTimer += deltaTime
    fontTimer += deltaTime

    if fpsTime >= 0.5 then
        if Settings.FPS then
            local fps = math.floor(fpsFrames / fpsTime + 0.5)
            StatLabels.FPS.Text = "FPS: " .. fps
            StatLabels.FPS.TextColor3 = fps >= 50 and Colors.Green or (fps >= 30 and Colors.Yellow or Colors.Red)
        end

        fpsFrames = 0
        fpsTime = 0
    end

    if statTimer >= 1 then
        statTimer = 0

        if Settings.Ping then
            local ok, value = pcall(function()
                return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
            end)

            if ok then
                StatLabels.Ping.Text = "PING: " .. math.floor(value) .. " ms"
            end
        end

        if Settings.Memory then
            local ok, value = pcall(function()
                return Stats:GetTotalMemoryUsageMb()
            end)

            if ok then
                StatLabels.Memory.Text = "MEM: " .. math.floor(value) .. " MB"
            end
        end
    end

    if Settings.LockTime and math.abs(Lighting.ClockTime - Settings.LockedTime) > 0.001 then
        Lighting.ClockTime = Settings.LockedTime
    end

    if fontTimer >= 0.5 then
        fontTimer = 0

        local spec = FontState.Spec

        if spec and FontState.Game then
            for object in pairs(GameTexts) do
                if object.Parent then
                    local ok, matches = pcall(specMatches, object, spec)

                    if ok and not matches then
                        pcall(applySpec, object, spec)
                    end
                end
            end
        end
    end
end))

pcall(function()
    RunService:UnbindFromRenderStep("CatSettingsFOV")
end)

RunService:BindToRenderStep("CatSettingsFOV", Enum.RenderPriority.Last.Value, function()
    if Settings.CustomFOV then
        local camera = workspace.CurrentCamera

        if camera and camera.FieldOfView ~= Settings.FOV then
            camera.FieldOfView = Settings.FOV
        end
    end
end)

local function handleNew(object)
    if shuttingDown or not object.Parent then
        return
    end

    local kind = graphicsKind(object)

    if kind and Settings[KindFlag[kind]] then
        pcall(function()
            local data = captureObject(object, kind)

            if data then
                applyData(object, data)
            end
        end)
    end

    if FontState.Spec and FontState.Game and isGameText(object) then
        styleGameText(object)
    end
end

table.insert(LifetimeConnections, game.DescendantAdded:Connect(function(object)
    if shuttingDown then
        return
    end

    local fontActive = FontState.Spec ~= nil and FontState.Game

    if not (fontActive or Settings.Particles or Settings.Textures or Settings.PostEffects or Settings.Materials) then
        return
    end

    local kind = graphicsKind(object)
    local wantsGraphics = kind ~= nil and Settings[KindFlag[kind]]
    local wantsFont = fontActive and isGameText(object)

    if wantsGraphics or wantsFont then
        task.defer(handleNew, object)
    end
end))

watchCamera()
table.insert(LifetimeConnections, workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(watchCamera))

if Saved.ActiveFont then
    local spec = specFromKey(Saved.ActiveFont)

    if spec then
        setFont(spec, true)
    end
end

applyTheme(true)

Shared.CatSettingsUnload = shutdown

openPage("Performance")
notify("CAT SETTINGS loaded")
