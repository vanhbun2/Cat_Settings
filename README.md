local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

local CONFIG = {
    ScriptName = "CAT SETTINGS",
    AvatarId = "139350569952709",
    Theme = Color3.fromRGB(120, 105, 255),
    MenuSize = 1,
    Transparency = 0.12
}

local Colors = {
    Background = Color3.fromRGB(15, 17, 25),
    Panel = Color3.fromRGB(23, 26, 37),
    Card = Color3.fromRGB(31, 35, 49),
    Text = Color3.fromRGB(245, 246, 255),
    SubText = Color3.fromRGB(153, 160, 180),
    Accent = CONFIG.Theme,
    Green = Color3.fromRGB(90, 220, 155),
    Red = Color3.fromRGB(255, 100, 110)
}

local OriginalLighting = {}
local OriginalObjects = {}
local OriginalFonts = {}
local OriginalSky = {}
local Connections = {}
local LifetimeConnections = {}

local Settings = {
    FixLag = false,
    Shadows = false,
    Particles = false,
    Textures = false,
    PostEffects = false,
    FPS = false,
    Ping = false,
    Fog = true,
    Bloom = false,
    Blur = false,
    Saturation = 0,
    Contrast = 0,
    Brightness = Lighting.Brightness,
    Font = Enum.Font.Gotham
}

local Pages = {}
local CurrentPage
local Toast
local openPage
local shuttingDown = false
local minimized = false
local menuAnimationId = 0

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
    "GlobalShadows"
}) do
    OriginalLighting[property] = Lighting[property]
end

for _, object in ipairs(Lighting:GetChildren()) do
    if object:IsA("Sky") then
        table.insert(OriginalSky, object:Clone())
    end
end

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

local function notify(message)
    if not Toast or not Toast.Parent or shuttingDown then
        return
    end

    Toast.Text = message
    Toast.Visible = true
    Toast.Position = UDim2.new(0.5, -130, 1, -65)

    task.delay(2, function()
        if Toast and Toast.Parent and not shuttingDown then
            tween(Toast, {
                Position = UDim2.new(0.5, -130, 1, 30)
            }, 0.25)

            task.wait(0.3)

            if Toast and Toast.Parent then
                Toast.Visible = false
            end
        end
    end)
end

local ScreenGui = create("ScreenGui", {
    Name = "CatSettings",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling
}, PlayerGui)

local Main = create("Frame", {
    Name = "SettingsPanel",
    Size = UDim2.fromOffset(700, 460),
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    BackgroundColor3 = Colors.Background,
    BackgroundTransparency = CONFIG.Transparency,
    BorderSizePixel = 0,
    Visible = false
}, ScreenGui)

corner(Main, 18)
local MainStroke = stroke(Main, Colors.Accent, 0.5)

local Scale = create("UIScale", {
    Scale = CONFIG.MenuSize
}, Main)

local function updateResponsiveScale()
    local camera = workspace.CurrentCamera
    if not camera then
        Scale.Scale = CONFIG.MenuSize
        return
    end

    local viewport = camera.ViewportSize
    local fitX = math.max(0.55, (viewport.X - 24) / 700)
    local fitY = math.max(0.55, (viewport.Y - 24) / 460)
    Scale.Scale = math.min(CONFIG.MenuSize, fitX, fitY)
end

local CameraViewportConnection

local function watchCamera(camera)
    if CameraViewportConnection then
        CameraViewportConnection:Disconnect()
        CameraViewportConnection = nil
    end

    if camera then
        CameraViewportConnection = camera:GetPropertyChangedSignal("ViewportSize"):Connect(updateResponsiveScale)
        table.insert(LifetimeConnections, CameraViewportConnection)
    end

    updateResponsiveScale()
end

watchCamera(workspace.CurrentCamera)
table.insert(LifetimeConnections, workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
    watchCamera(workspace.CurrentCamera)
end))

local Sidebar = create("Frame", {
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

local Avatar = create("ImageButton", {
    Size = UDim2.fromOffset(54, 54),
    Position = UDim2.fromOffset(18, 20),
    BackgroundColor3 = Colors.Card,
    Image = "rbxassetid://" .. CONFIG.AvatarId,
    ScaleType = Enum.ScaleType.Crop
}, Sidebar)

corner(Avatar, 16)

local ScriptTitle = create("TextLabel", {
    Size = UDim2.new(1, -25, 0, 24),
    Position = UDim2.fromOffset(18, 84),
    BackgroundTransparency = 1,
    Text = "CAT SETTINGS",
    TextColor3 = Colors.Text,
    Font = Enum.Font.GothamBold,
    TextSize = 15,
    TextXAlignment = Enum.TextXAlignment.Left
}, Sidebar)

create("TextLabel", {
    Size = UDim2.new(1, -25, 0, 18),
    Position = UDim2.fromOffset(18, 108),
    BackgroundTransparency = 1,
    Text = "Premium Interface",
    TextColor3 = Colors.SubText,
    Font = Enum.Font.Gotham,
    TextSize = 11,
    TextXAlignment = Enum.TextXAlignment.Left
}, Sidebar)

local Navigation = create("Frame", {
    Size = UDim2.new(1, -20, 1, -145),
    Position = UDim2.fromOffset(10, 135),
    BackgroundTransparency = 1
}, Sidebar)

create("UIListLayout", {
    Padding = UDim.new(0, 7),
    SortOrder = Enum.SortOrder.LayoutOrder
}, Navigation)

local Content = create("Frame", {
    Size = UDim2.new(1, -195, 1, -28),
    Position = UDim2.fromOffset(185, 14),
    BackgroundTransparency = 1
}, Main)

local Header = create("Frame", {
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundTransparency = 1
}, Content)

local PageTitle = create("TextLabel", {
    Size = UDim2.new(1, -100, 0, 32),
    Position = UDim2.fromOffset(4, 0),
    BackgroundTransparency = 1,
    Text = "Performance",
    TextColor3 = Colors.Text,
    Font = Enum.Font.GothamBold,
    TextSize = 21,
    TextXAlignment = Enum.TextXAlignment.Left
}, Header)

local MinimizeButton = create("TextButton", {
    Size = UDim2.fromOffset(32, 32),
    Position = UDim2.new(1, -76, 0, 0),
    BackgroundColor3 = Colors.Card,
    Text = "−",
    TextColor3 = Colors.Text,
    Font = Enum.Font.GothamBold,
    TextSize = 20,
    AutoButtonColor = false
}, Header)

corner(MinimizeButton, 9)

local CloseButton = create("TextButton", {
    Size = UDim2.fromOffset(32, 32),
    Position = UDim2.new(1, -36, 0, 0),
    BackgroundColor3 = Colors.Card,
    Text = "×",
    TextColor3 = Colors.Text,
    Font = Enum.Font.GothamBold,
    TextSize = 20,
    AutoButtonColor = false
}, Header)

corner(CloseButton, 9)

local PageHolder = create("ScrollingFrame", {
    Size = UDim2.new(1, 0, 1, -55),
    Position = UDim2.fromOffset(0, 55),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 3,
    ScrollBarImageColor3 = Colors.Accent,
    CanvasSize = UDim2.new()
}, Content)

local PageLayout = create("UIListLayout", {
    Padding = UDim.new(0, 10),
    SortOrder = Enum.SortOrder.LayoutOrder
}, PageHolder)

table.insert(LifetimeConnections,
    PageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        PageHolder.CanvasSize = UDim2.new(0, 0, 0, PageLayout.AbsoluteContentSize.Y + 15)
    end)
)

local FPSLabel = create("TextLabel", {
    Size = UDim2.fromOffset(110, 30),
    Position = UDim2.new(1, -125, 0, 12),
    BackgroundColor3 = Colors.Card,
    Text = "FPS: --",
    TextColor3 = Colors.Green,
    Font = Enum.Font.GothamBold,
    TextSize = 12,
    Visible = false
}, ScreenGui)

corner(FPSLabel, 9)

local PingLabel = create("TextLabel", {
    Size = UDim2.fromOffset(110, 30),
    Position = UDim2.new(1, -125, 0, 49),
    BackgroundColor3 = Colors.Card,
    Text = "PING: --",
    TextColor3 = Colors.Green,
    Font = Enum.Font.GothamBold,
    TextSize = 12,
    Visible = false
}, ScreenGui)

corner(PingLabel, 9)

Toast = create("TextLabel", {
    Size = UDim2.fromOffset(260, 42),
    Position = UDim2.new(0.5, -130, 1, 30),
    BackgroundColor3 = Colors.Card,
    TextColor3 = Colors.Text,
    Text = "",
    TextSize = 13,
    Font = Enum.Font.GothamMedium,
    BackgroundTransparency = 0.05,
    Visible = false
}, ScreenGui)

corner(Toast, 12)
stroke(Toast, Colors.Accent, 0.6)

local function clearPage()
    for _, child in ipairs(PageHolder:GetChildren()) do
        if child:IsA("GuiObject") and child ~= PageLayout then
            child:Destroy()
        end
    end
end

local function addCard(height)
    local card = create("Frame", {
        Size = UDim2.new(1, -5, 0, height),
        BackgroundColor3 = Colors.Card,
        BackgroundTransparency = 0.08,
        BorderSizePixel = 0
    }, PageHolder)

    corner(card, 12)
    stroke(card, Color3.new(1, 1, 1), 0.94)

    return card
end

local function addSection(name)
    create("TextLabel", {
        Size = UDim2.new(1, -5, 0, 25),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Accent,
        Font = Settings.Font,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left
    }, PageHolder)
end

local function addButton(name, description, callback)
    local card = addCard(64)

    create("TextLabel", {
        Size = UDim2.new(1, -130, 0, 23),
        Position = UDim2.fromOffset(14, 8),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Text,
        Font = Settings.Font,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card)

    create("TextLabel", {
        Size = UDim2.new(1, -130, 0, 18),
        Position = UDim2.fromOffset(14, 33),
        BackgroundTransparency = 1,
        Text = description or "",
        TextColor3 = Colors.SubText,
        Font = Settings.Font,
        TextSize = 10,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card)

    local button = create("TextButton", {
        Size = UDim2.fromOffset(88, 32),
        Position = UDim2.new(1, -102, 0.5, -16),
        BackgroundColor3 = Colors.Accent,
        Text = "Apply",
        TextColor3 = Colors.Text,
        Font = Settings.Font,
        TextSize = 11,
        AutoButtonColor = false
    }, card)

    corner(button, 9)

    button.MouseButton1Click:Connect(callback)

    return button
end

local function addToggle(name, description, initial, callback)
    local card = addCard(64)
    local enabled = initial == true

    create("TextLabel", {
        Size = UDim2.new(1, -90, 0, 23),
        Position = UDim2.fromOffset(14, 8),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Text,
        Font = Settings.Font,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card)

    create("TextLabel", {
        Size = UDim2.new(1, -90, 0, 18),
        Position = UDim2.fromOffset(14, 33),
        BackgroundTransparency = 1,
        Text = description or "",
        TextColor3 = Colors.SubText,
        Font = Settings.Font,
        TextSize = 10,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card)

    local toggle = create("TextButton", {
        Size = UDim2.fromOffset(46, 25),
        Position = UDim2.new(1, -62, 0.5, -12),
        BackgroundColor3 = enabled and Colors.Accent or Color3.fromRGB(65, 68, 82),
        Text = "",
        AutoButtonColor = false
    }, card)

    corner(toggle, 20)

    local knob = create("Frame", {
        Size = UDim2.fromOffset(19, 19),
        Position = enabled and UDim2.new(1, -22, 0.5, -9) or UDim2.fromOffset(3, 3),
        BackgroundColor3 = Color3.new(1, 1, 1),
        BorderSizePixel = 0
    }, toggle)

    corner(knob, 20)

    local function update(value)
        enabled = value

        tween(toggle, {
            BackgroundColor3 = enabled and Colors.Accent or Color3.fromRGB(65, 68, 82)
        })

        tween(knob, {
            Position = enabled and UDim2.new(1, -22, 0.5, -9) or UDim2.fromOffset(3, 3)
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

local function addSlider(name, minimum, maximum, initial, callback)
    local card = addCard(82)
    local value = math.clamp(initial, minimum, maximum)

    create("TextLabel", {
        Size = UDim2.new(1, -70, 0, 24),
        Position = UDim2.fromOffset(14, 7),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Colors.Text,
        Font = Settings.Font,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left
    }, card)

    local valueLabel = create("TextLabel", {
        Size = UDim2.fromOffset(55, 24),
        Position = UDim2.new(1, -68, 0, 7),
        BackgroundTransparency = 1,
        Text = tostring(value),
        TextColor3 = Colors.Accent,
        Font = Settings.Font,
        TextSize = 12
    }, card)

    local bar = create("Frame", {
        Size = UDim2.new(1, -28, 0, 6),
        Position = UDim2.fromOffset(14, 53),
        BackgroundColor3 = Color3.fromRGB(60, 64, 80),
        BorderSizePixel = 0,
        Active = true
    }, card)

    corner(bar, 10)

    local percent = (value - minimum) / (maximum - minimum)

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

    local function setValue(x)
        local newPercent = math.clamp(
            (x - bar.AbsolutePosition.X) / math.max(bar.AbsoluteSize.X, 1),
            0,
            1
        )

        value = math.floor((minimum + (maximum - minimum) * newPercent) * 10) / 10

        fill.Size = UDim2.new(newPercent, 0, 1, 0)
        knob.Position = UDim2.new(newPercent, -7, 0.5, -7)
        valueLabel.Text = tostring(value)

        callback(value)
    end

    table.insert(Connections, bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            setValue(input.Position.X)
        end
    end))

    table.insert(Connections, UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            setValue(input.Position.X)
        end
    end))

    table.insert(Connections, UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end))
end

local function clearSky()
    for _, object in ipairs(Lighting:GetChildren()) do
        if object:IsA("Sky") then
            object:Destroy()
        end
    end
end

local function removeEffect(name)
    local effect = Lighting:FindFirstChild(name)

    if effect then
        effect:Destroy()
    end
end

local function restoreSky(silent)
    clearSky()

    for _, object in ipairs(OriginalSky) do
        object:Clone().Parent = Lighting
    end

    for property, value in pairs(OriginalLighting) do
        Lighting[property] = value
    end

    removeEffect("CatAtmosphere")
    removeEffect("CatColorCorrection")
    removeEffect("CatBloom")
    removeEffect("CatBlur")

    Settings.Brightness = OriginalLighting.Brightness
    Settings.Fog = true
    Settings.Saturation = 0
    Settings.Contrast = 0
    Settings.Bloom = false
    Settings.Blur = false

    if not silent then
        notify("Original sky restored")
    end
end

local function setAtmosphere(density, haze, color)
    local atmosphere = Lighting:FindFirstChild("CatAtmosphere")

    if not atmosphere then
        atmosphere = Instance.new("Atmosphere")
        atmosphere.Name = "CatAtmosphere"
        atmosphere.Parent = Lighting
    end

    atmosphere.Density = density
    atmosphere.Haze = haze
    atmosphere.Color = color
    atmosphere.Decay = Color3.fromRGB(90, 100, 130)
    atmosphere.Glare = 0.2
    atmosphere.Offset = 0.1
end

local function setColorCorrection(saturation, contrast)
    local effect = Lighting:FindFirstChild("CatColorCorrection")

    if not effect then
        effect = Instance.new("ColorCorrectionEffect")
        effect.Name = "CatColorCorrection"
        effect.Parent = Lighting
    end

    effect.Saturation = saturation
    effect.Contrast = contrast
end

local function applyBloom(enabled)
    local effect = Lighting:FindFirstChild("CatBloom")

    if enabled then
        if not effect then
            effect = Instance.new("BloomEffect")
            effect.Name = "CatBloom"
            effect.Parent = Lighting
        end

        effect.Intensity = 0.35
        effect.Size = 32
        effect.Threshold = 1
    elseif effect then
        effect:Destroy()
    end
end

local SkyPresets = {
    ["REALISTIC DAY"] = {
        13, 3,
        Color3.fromRGB(170, 180, 200),
        Color3.fromRGB(190, 195, 205),
        Color3.fromRGB(180, 215, 245)
    },

    ["GOLDEN HOUR"] = {
        17, 2.8,
        Color3.fromRGB(180, 140, 115),
        Color3.fromRGB(210, 155, 120),
        Color3.fromRGB(255, 180, 125)
    },

    ["SUNSET"] = {
        18.3, 2.5,
        Color3.fromRGB(145, 105, 130),
        Color3.fromRGB(190, 130, 140),
        Color3.fromRGB(245, 145, 135)
    },

    ["EVENING"] = {
        19.5, 2,
        Color3.fromRGB(100, 105, 145),
        Color3.fromRGB(125, 135, 180),
        Color3.fromRGB(100, 120, 175)
    },

    ["NIGHT"] = {
        0, 1.5,
        Color3.fromRGB(65, 75, 115),
        Color3.fromRGB(90, 105, 150),
        Color3.fromRGB(35, 45, 90)
    },

    ["CLOUDY"] = {
        12, 2,
        Color3.fromRGB(130, 140, 150),
        Color3.fromRGB(160, 170, 180),
        Color3.fromRGB(145, 160, 180)
    }
}

local function applySkyPreset(name)
    local preset = SkyPresets[name]

    if not preset then
        return
    end

    clearSky()

    Lighting.ClockTime = preset[1]
    Lighting.Brightness = preset[2]
    Lighting.Ambient = preset[3]
    Lighting.OutdoorAmbient = preset[4]
    Lighting.FogColor = preset[5]

    Settings.Brightness = preset[2]

    setAtmosphere(0.25, 1, preset[5])

    notify(name .. " applied")
end

local function createSpaceSky(name, color)
    clearSky()

    local sky = Instance.new("Sky")
    sky.Name = "CatSpaceSky"
    sky.StarCount = 3000
    sky.CelestialBodiesShown = true
    sky.SunAngularSize = 0
    sky.MoonAngularSize = 11
    sky.Parent = Lighting

    Lighting.ClockTime = 0
    Lighting.Brightness = 1.5
    Lighting.Ambient = color
    Lighting.OutdoorAmbient = color
    Lighting.FogColor = color

    Settings.Brightness = 1.5

    setAtmosphere(0.05, 0, color)

    notify(name .. " applied")
end

local function captureObject(object)
    if OriginalObjects[object] then
        return
    end

    if object:IsA("BasePart") then
        OriginalObjects[object] = {
            CastShadow = object.CastShadow
        }

    elseif object:IsA("ParticleEmitter") or object:IsA("Trail") or object:IsA("Beam") then
        OriginalObjects[object] = {
            Enabled = object.Enabled
        }

    elseif object:IsA("Decal") or object:IsA("Texture") then
        OriginalObjects[object] = {
            Transparency = object.Transparency
        }

    elseif object:IsA("PostEffect") then
        OriginalObjects[object] = {
            Enabled = object.Enabled
        }
    end
end

local function applyObjectGraphics(object)
    captureObject(object)

    local properties = OriginalObjects[object]

    if not properties then
        return
    end

    if object:IsA("BasePart") then
        object.CastShadow = properties.CastShadow and not (Settings.Shadows or Settings.FixLag)

    elseif object:IsA("ParticleEmitter") or object:IsA("Trail") or object:IsA("Beam") then
        object.Enabled = properties.Enabled and not (Settings.Particles or Settings.FixLag)

    elseif object:IsA("Decal") or object:IsA("Texture") then
        object.Transparency = Settings.Textures and 1 or properties.Transparency

    elseif object:IsA("PostEffect") then
        object.Enabled = properties.Enabled and not (Settings.PostEffects or Settings.FixLag)
    end
end

local GraphicsScanned = false

local function applyGraphics()
    local active = Settings.FixLag or Settings.Shadows or Settings.Particles or Settings.Textures or Settings.PostEffects

    Lighting.GlobalShadows = OriginalLighting.GlobalShadows and not (Settings.Shadows or Settings.FixLag)

    if active and not GraphicsScanned then
        GraphicsScanned = true
        for _, object in ipairs(game:GetDescendants()) do
            applyObjectGraphics(object)
        end
        return
    end

    if not active then
        GraphicsScanned = false
    end

    for object in pairs(OriginalObjects) do
        if object and object.Parent then
            applyObjectGraphics(object)
        else
            OriginalObjects[object] = nil
        end
    end
end

local function restoreGraphics(silent)
    for object, properties in pairs(OriginalObjects) do
        if object and object.Parent then
            for property, value in pairs(properties) do
                pcall(function()
                    object[property] = value
                end)
            end
        end
    end

    Lighting.GlobalShadows = OriginalLighting.GlobalShadows

    Settings.FixLag = false
    Settings.Shadows = false
    Settings.Particles = false
    Settings.Textures = false
    Settings.PostEffects = false
    GraphicsScanned = false

    if not silent then
        notify("Graphics restored")
    end
end

local function applyTheme(silent)
    Colors.Accent = CONFIG.Theme

    Main.BackgroundTransparency = CONFIG.Transparency
    PageHolder.ScrollBarImageColor3 = Colors.Accent
    MainStroke.Color = Colors.Accent

    Avatar.Image = "rbxassetid://" .. CONFIG.AvatarId
    updateResponsiveScale()

    local openStroke = OpenButton and OpenButton:FindFirstChildOfClass("UIStroke")

    if openStroke then
        openStroke.Color = Colors.Accent
    end

    if not silent then
        notify("Theme updated")
    end
end

local function applyFont(font)
    Settings.Font = font

    for _, object in ipairs(game:GetDescendants()) do
        if object:IsA("TextLabel") or object:IsA("TextButton") or object:IsA("TextBox") then
            if not OriginalFonts[object] then
                OriginalFonts[object] = object.Font
            end

            object.Font = font
        end
    end

    notify("Font applied")
end

local function restoreFonts(silent)
    for object, font in pairs(OriginalFonts) do
        if object and object.Parent then
            pcall(function()
                object.Font = font
            end)
        end
    end

    table.clear(OriginalFonts)
    Settings.Font = Enum.Font.Gotham

    if not silent then
        notify("Fonts restored")
    end
end

local function refreshPage()
    if CurrentPage and openPage then
        openPage(CurrentPage)
    end
end

local function addNavigation(name)
    local button = create("TextButton", {
        Name = name,
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = Colors.Panel,
        Text = name,
        TextColor3 = Colors.Text,
        Font = Enum.Font.GothamMedium,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Left,
        AutoButtonColor = false
    }, Navigation)

    corner(button, 9)

    local padding = Instance.new("UIPadding")
    padding.PaddingLeft = UDim.new(0, 12)
    padding.Parent = button

    button.MouseButton1Click:Connect(function()
        openPage(name)
    end)
end

Pages["Performance"] = function()
    addSection("PERFORMANCE")

    addToggle("Fix Lag", "Reduce selected visual effects", Settings.FixLag, function(value)
        Settings.FixLag = value
        applyGraphics()
    end)

    addToggle("Show FPS", "Display current frame rate", Settings.FPS, function(value)
        Settings.FPS = value
        FPSLabel.Visible = value
    end)

    addToggle("Show Ping", "Display network latency", Settings.Ping, function(value)
        Settings.Ping = value
        PingLabel.Visible = value
    end)

    addSection("GRAPHICS")

    addToggle("Remove Shadows", "Disable world shadows", Settings.Shadows, function(value)
        Settings.Shadows = value
        applyGraphics()
    end)

    addToggle("Remove Particles", "Hide particle effects", Settings.Particles, function(value)
        Settings.Particles = value
        applyGraphics()
    end)

    addToggle("Remove Textures", "Hide decals and textures", Settings.Textures, function(value)
        Settings.Textures = value
        applyGraphics()
    end)

    addToggle("Remove Post Effects", "Disable post processing", Settings.PostEffects, function(value)
        Settings.PostEffects = value
        applyGraphics()
    end)

    addButton("Restore Graphics", "Restore original graphics", function()
        restoreGraphics()
        refreshPage()
    end)
end

Pages["Sky & Space"] = function()
    addSection("SKY PRESETS")

    for name in pairs(SkyPresets) do
        addButton(name, "Apply lighting preset", function()
            applySkyPreset(name)
        end)
    end

    addSection("ASTRONOMY")

    addButton("MILKY WAY", "Deep night sky and stars", function()
        createSpaceSky("MILKY WAY", Color3.fromRGB(55, 65, 110))
    end)

    addButton("DEEP SPACE", "Dark blue space atmosphere", function()
        createSpaceSky("DEEP SPACE", Color3.fromRGB(35, 40, 85))
    end)

    addButton("NEBULA", "Purple space atmosphere", function()
        createSpaceSky("NEBULA", Color3.fromRGB(100, 55, 140))
    end)

    addButton("AURORA", "Green and blue atmosphere", function()
        createSpaceSky("AURORA", Color3.fromRGB(40, 115, 110))
    end)

    addButton("GALAXY", "Dark purple galaxy atmosphere", function()
        createSpaceSky("GALAXY", Color3.fromRGB(80, 45, 125))
    end)

    addButton("DEFAULT SKY", "Restore original lighting", function()
        restoreSky()
        refreshPage()
    end)

    addSection("LIGHTING")

    addSlider("Brightness", 0, 6, Settings.Brightness, function(value)
        Settings.Brightness = value
        Lighting.Brightness = value
    end)

    addSlider("Saturation", -1, 1, Settings.Saturation, function(value)
        Settings.Saturation = value
        setColorCorrection(value, Settings.Contrast)
    end)

    addSlider("Contrast", -1, 1, Settings.Contrast, function(value)
        Settings.Contrast = value
        setColorCorrection(Settings.Saturation, value)
    end)

    addToggle("Fog", "Toggle distance fog", Settings.Fog, function(value)
        Settings.Fog = value

        if value then
            Lighting.FogStart = OriginalLighting.FogStart
            Lighting.FogEnd = OriginalLighting.FogEnd
        else
            Lighting.FogStart = 100000
            Lighting.FogEnd = 100000
        end
    end)
end

Pages["Visual Effects"] = function()
    addSection("POST PROCESSING")

    addToggle("Bloom", "Soft lighting glow", Settings.Bloom, function(value)
        Settings.Bloom = value
        applyBloom(value)
    end)

    addToggle("Blur", "Soft screen blur", Settings.Blur, function(value)
        Settings.Blur = value

        local effect = Lighting:FindFirstChild("CatBlur")

        if value then
            if not effect then
                effect = Instance.new("BlurEffect")
                effect.Name = "CatBlur"
                effect.Size = 8
                effect.Parent = Lighting
            end
        elseif effect then
            effect:Destroy()
        end
    end)

    addSection("ATMOSPHERE")

    addSlider("Density", 0, 1, 0.25, function(value)
        setAtmosphere(value, 1, Color3.fromRGB(190, 200, 220))
    end)

    addSlider("Haze", 0, 10, 1, function(value)
        local atmosphere = Lighting:FindFirstChild("CatAtmosphere")

        if atmosphere then
            atmosphere.Haze = value
        end
    end)

    addButton("Reset Effects", "Remove custom visual effects", function()
        removeEffect("CatBloom")
        removeEffect("CatBlur")
        removeEffect("CatAtmosphere")
        removeEffect("CatColorCorrection")

        Settings.Bloom = false
        Settings.Blur = false
        Settings.Saturation = 0
        Settings.Contrast = 0

        refreshPage()
        notify("Effects reset")
    end)
end

Pages["Fonts"] = function()
    addSection("FONT STYLE")

    local fonts = {
        "Gotham",
        "SourceSans",
        "Code",
        "Arcade",
        "SciFi",
        "Fantasy",
        "Bodoni",
        "Cartoon",
        "Roboto",
        "Ubuntu"
    }

    for _, name in ipairs(fonts) do
        addButton(name, "Apply font to accessible game UI", function()
            local success, font = pcall(function()
                return Enum.Font[name]
            end)

            if success and font then
                applyFont(font)
            end
        end)
    end

    addButton("Restore Font", "Restore saved original fonts", restoreFonts)
end

Pages["Interface"] = function()
    addSection("WINDOW")

    addSlider("Menu Size", 0.7, 1.2, CONFIG.MenuSize, function(value)
        CONFIG.MenuSize = value
        updateResponsiveScale()
    end)

    addSlider("Transparency", 0, 0.5, CONFIG.Transparency, function(value)
        CONFIG.Transparency = value
        Main.BackgroundTransparency = value
    end)

    addSection("THEME COLOR")

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

    for _, theme in ipairs(themes) do
        addButton(theme[1], "Change accent color", function()
            CONFIG.Theme = theme[2]
            applyTheme()
            refreshPage()
        end)
    end
end

Pages["General"] = function()
    addSection("GENERAL")

    addButton("Restore Graphics", "Restore original graphics", function()
        restoreGraphics()
        refreshPage()
    end)

    addButton("Restore Sky", "Restore original lighting and sky", function()
        restoreSky()
        refreshPage()
    end)

    addButton("Restore Fonts", "Restore saved original fonts", function()
        restoreFonts()
    end)

    addButton("Restore Defaults", "Reset all settings to defaults", function()
        restoreGraphics(true)
        restoreSky(true)
        restoreFonts(true)

        removeEffect("CatBloom")
        removeEffect("CatBlur")
        removeEffect("CatAtmosphere")
        removeEffect("CatColorCorrection")

        CONFIG.Theme = Color3.fromRGB(120, 105, 255)
        CONFIG.MenuSize = 1
        CONFIG.Transparency = 0.12

        Settings.FPS = false
        Settings.Ping = false
        Settings.Bloom = false
        Settings.Blur = false
        Settings.Saturation = 0
        Settings.Contrast = 0
        Settings.Fog = true
        Settings.Brightness = OriginalLighting.Brightness

        FPSLabel.Visible = false
        PingLabel.Visible = false

        applyTheme(true)
        refreshPage()

        notify("Defaults restored")
    end)
end

openPage = function(name)
    if not Pages[name] or shuttingDown then
        return
    end

    CurrentPage = name
    PageTitle.Text = name

    for _, connection in ipairs(Connections) do
        connection:Disconnect()
    end

    table.clear(Connections)

    clearPage()
    Pages[name]()

    for _, button in ipairs(Navigation:GetChildren()) do
        if button:IsA("TextButton") then
            tween(button, {
                BackgroundColor3 = button.Name == name and Colors.Accent or Colors.Panel
            })
        end
    end
end

addNavigation("Performance")
addNavigation("Sky & Space")
addNavigation("Visual Effects")
addNavigation("Fonts")
addNavigation("Interface")
addNavigation("General")

local OpenButton = create("ImageButton", {
    Name = "CatButton",
    Size = UDim2.fromOffset(58, 58),
    Position = UDim2.new(0, 25, 0.5, -29),
    BackgroundColor3 = Colors.Panel,
    Image = "rbxassetid://" .. CONFIG.AvatarId,
    ScaleType = Enum.ScaleType.Crop,
    AutoButtonColor = false
}, ScreenGui)

corner(OpenButton, 18)
stroke(OpenButton, Colors.Accent, 0.45)

local function makeDraggable(object, handle)
    local dragging = false
    local dragInput
    local dragStart
    local startPosition

    table.insert(LifetimeConnections, handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPosition = object.Position

            if input.UserInputType == Enum.UserInputType.Touch then
                dragInput = input
            end
        end
    end))

    table.insert(LifetimeConnections, UserInputService.InputChanged:Connect(function(input)
        if dragging and (
            input.UserInputType == Enum.UserInputType.MouseMovement
            or input == dragInput
        ) then
            local delta = input.Position - dragStart

            object.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end))

    table.insert(LifetimeConnections, UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input == dragInput then
            dragging = false
            dragInput = nil
        end
    end))
end

makeDraggable(Main, Header)
makeDraggable(OpenButton, OpenButton)

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

        openPage(CurrentPage or "Performance")
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

    openPage("Performance")
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

local function shutdown()
    if shuttingDown then
        return
    end

    shuttingDown = true

    menuAnimationId += 1

    for _, connection in ipairs(Connections) do
        connection:Disconnect()
    end

    for _, connection in ipairs(LifetimeConnections) do
        connection:Disconnect()
    end

    table.clear(Connections)
    table.clear(LifetimeConnections)

    restoreGraphics(true)
    restoreSky(true)
    restoreFonts(true)

    removeEffect("CatBloom")
    removeEffect("CatBlur")
    removeEffect("CatAtmosphere")
    removeEffect("CatColorCorrection")

    if ScreenGui then
        ScreenGui:Destroy()
    end
end

MinimizeButton.MouseButton1Click:Connect(function()
    setMinimized(not minimized)
end)

CloseButton.MouseButton1Click:Connect(shutdown)

OpenButton.MouseButton1Click:Connect(function()
    if Main.Visible then
        closeMenu()
    else
        openMenu()
    end
end)

local fpsFrames = 0
local fpsTime = 0
local pingTimer = 0

table.insert(LifetimeConnections, RunService.RenderStepped:Connect(function(deltaTime)
    if shuttingDown then
        return
    end

    fpsFrames += 1
    fpsTime += deltaTime
    pingTimer += deltaTime

    if fpsTime >= 0.5 then
        if Settings.FPS then
            FPSLabel.Text = "FPS: " .. math.floor(fpsFrames / fpsTime)
        end

        fpsFrames = 0
        fpsTime = 0
    end

    if pingTimer >= 1 then
        if Settings.Ping then
            local success, value = pcall(function()
                return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
            end)

            if success then
                PingLabel.Text = "PING: " .. math.floor(value) .. " ms"
            end
        end

        pingTimer = 0
    end
end))

table.insert(LifetimeConnections, game.DescendantAdded:Connect(function(object)
    if shuttingDown then
        return
    end

    if Settings.FixLag or Settings.Shadows or Settings.Particles or Settings.Textures or Settings.PostEffects then
        task.defer(function()
            if shuttingDown or not object.Parent then
                return
            end

            applyObjectGraphics(object)
        end)
    end
end))

openPage("Performance")
notify("CAT SETTINGS loaded")
