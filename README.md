--[[
    RANDOM HUB - Ultimate Optimizer
    Compatível com Roblox (executores que permitem execução client-side).
    Tema: Preto / Azul / Branco.
    
    Aviso: Recursos dependem do jogo, do dispositivo e das permissões do executor.
    Propriedades locais só podem ser alteradas quando o cliente tem permissão.
--]]

-- =====================================================================
-- SERVIÇOS
-- =====================================================================
local Players           = game:GetService("Players")
local Lighting          = game:GetService("Lighting")
local RunService        = game:GetService("RunService")
local UserInputService  = game:GetService("UserInputService")
local TweenService      = game:GetService("TweenService")
local CoreGui           = pcall(function() return game:GetService("CoreGui") end) and game:GetService("CoreGui") or nil
local StarterGui        = game:GetService("StarterGui")
local Stats             = pcall(function() return game:GetService("Stats") end) and game:GetService("Stats") or nil

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer and LocalPlayer:WaitForChild("PlayerGui")
local Camera      = workspace.CurrentCamera

-- =====================================================================
-- ESTADO GLOBAL
-- =====================================================================
local RH = {
    Connections   = {},
    Original      = {},      -- valores originais
    Flags         = {
        FPSBoost       = false,
        FPSMode        = "Off", -- Off / Low / Medium / High
        RemoveFog      = false,
        ReduceFog      = false,
        GraphicsMode   = "Off", -- Off / Min / Balanced / HighPerf
        FPSCounter     = false,
        PingMonitor    = false,
        ReduceParticles= false,
        ReduceLights   = false,
        ReducePostFX   = false,
        FOV            = 70,
        InterfaceScale = 1,
    },
}

-- =====================================================================
-- UTILITÁRIOS
-- =====================================================================
local function safe(fn, ...)
    local ok, err = pcall(fn, ...)
    if not ok then
        warn("[RANDOM HUB] erro:", err)
    end
    return ok
end

local function connect(signal, fn)
    local c = signal:Connect(fn)
    table.insert(RH.Connections, c)
    return c
end

local function isProperty(obj, prop)
    if not obj then return false end
    local ok = pcall(function() return obj[prop] end)
    return ok
end

local function setProp(obj, prop, value)
    if not obj or not prop then return false end
    if not isProperty(obj, prop) then return false end
    return (pcall(function() obj[prop] = value end))
end

local function saveOriginal(obj, prop)
    if not obj then return end
    if not isProperty(obj, prop) then return end
    local key = tostring(obj) .. "." .. prop
    if RH.Original[key] == nil then
        local ok, val = pcall(function() return obj[prop] end)
        if ok then RH.Original[key] = val end
    end
end

local function restoreOriginal(obj, prop)
    if not obj then return end
    local key = tostring(obj) .. "." .. prop
    if RH.Original[key] ~= nil then
        pcall(function() obj[prop] = RH.Original[key] end)
        RH.Original[key] = nil
    end
end

local function color3(r, g, b) return Color3.fromRGB(r, g, b) end

local function applyGradient(gui, c1, c2)
    local grad = Instance.new("UIGradient")
    grad.Color = ColorSequence.new(c1, c2)
    grad.Rotation = 90
    grad.Parent = gui
    return grad
end

-- =====================================================================
-- EVITAR DUPLICAÇÃO
-- =====================================================================
local parentGui = CoreGui or PlayerGui
if parentGui then
    local old = parentGui:FindFirstChild("RandomHubUI")
    if old then old:Destroy() end
end

-- =====================================================================
-- INTERFACE BASE
-- =====================================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RandomHubUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 999
ScreenGui.Parent = parentGui

-- Escala responsiva
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local baseWidth = isMobile and 320 or 460
local baseHeight = isMobile and 380 or 460

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.new(0, baseWidth, 0, baseHeight)
Main.Position = UDim2.new(0.5, -baseWidth/2, 0.5, -baseHeight/2)
Main.BackgroundColor3 = color3(10, 10, 14)
Main.BorderSizePixel = 0
Main.ClipsDescendants = true
Main.Active = true
Main.Parent = ScreenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 10)
mainCorner.Parent = Main

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = color3(0, 140, 255)
mainStroke.Thickness = 1.5
mainStroke.Transparency = 0.2
mainStroke.Parent = Main

-- Header
local Header = Instance.new("Frame")
Header.Name = "Header"
Header.Size = UDim2.new(1, 0, 0, isMobile and 44 or 50)
Header.BackgroundColor3 = color3(14, 18, 28)
Header.BorderSizePixel = 0
Header.Parent = Main
applyGradient(Header, color3(0, 90, 200), color3(0, 30, 70))

local headerTitle = Instance.new("TextLabel")
headerTitle.Text = "RANDOM HUB"
headerTitle.Font = Enum.Font.GothamBlack
headerTitle.TextSize = isMobile and 16 or 20
headerTitle.TextColor3 = color3(255, 255, 255)
headerTitle.BackgroundTransparency = 1
headerTitle.Size = UDim2.new(1, -110, 1, 0)
headerTitle.Position = UDim2.new(0, 14, 0, 0)
headerTitle.TextXAlignment = Enum.TextXAlignment.Left
headerTitle.Parent = Header

local subtitle = Instance.new("TextLabel")
subtitle.Text = " Ultimate Optimizer"
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 11
subtitle.TextColor3 = color3(180, 210, 255)
subtitle.BackgroundTransparency = 1
subtitle.Size = UDim2.new(1, -110, 0, 14)
subtitle.Position = UDim2.new(0, 14, 1, -16)
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Parent = Header

-- Botões do header (Min, Close)
local function makeHeaderBtn(text, xOffset, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 30, 0, 30)
    btn.Position = UDim2.new(1, xOffset, 0.5, -15)
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.TextColor3 = color3(255,255,255)
    btn.AutoButtonColor = false
    btn.Parent = Header
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 6); c.Parent = btn
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 0.2}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 0}):Play()
    end)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local btnMin   = makeHeaderBtn("–", -72, color3(30, 60, 110), function()
    Main.Visible = false
    ToggleBtn.Visible = true
end)
local btnClose = makeHeaderBtn("✕", -36, color3(150, 30, 30), function()
    safe(function()
        if RH.Flags.FPSCounter then RH:ToggleFPSCounter(false) end
        if RH.Flags.PingMonitor then RH:TogglePing(false) end
    end)
    for _, c in ipairs(RH.Connections) do pcall(function() c:Disconnect() end) end
    RH.Connections = {}
    ScreenGui:Destroy()
end)

-- Botão flutuante para reabrir
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 20, 0.5, -25)
ToggleBtn.BackgroundColor3 = color3(0, 120, 230)
ToggleBtn.Text = "RH"
ToggleBtn.Font = Enum.Font.GothamBlack
ToggleBtn.TextSize = 16
ToggleBtn.TextColor3 = color3(255,255,255)
ToggleBtn.Visible = false
ToggleBtn.AutoButtonColor = false
ToggleBtn.Parent = ScreenGui
local tbCorner = Instance.new("UICorner"); tbCorner.CornerRadius = UDim.new(1,0); tbCorner.Parent = ToggleBtn
local tbStroke = Instance.new("UIStroke"); tbStroke.Color = color3(0,180,255); tbStroke.Thickness = 2; tbStroke.Parent = ToggleBtn
ToggleBtn.MouseButton1Click:Connect(function()
    Main.Visible = true
    ToggleBtn.Visible = false
end)

-- Arrastar
local dragging, dragStart, startPos
Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = Main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Conteúdo
local Content = Instance.new("Frame")
Content.Name = "Content"
Content.Size = UDim2.new(1, -12, 1, -(Header.Size.Y.Offset + 60))
Content.Position = UDim2.new(0, 6, 0, Header.Size.Y.Offset + 6)
Content.BackgroundTransparency = 1
Content.Parent = Main

-- Barra de abas (scroll horizontal)
local TabBar = Instance.new("ScrollingFrame")
TabBar.Size = UDim2.new(1, -12, 0, 40)
TabBar.Position = UDim2.new(0, 6, 1, -46)
TabBar.BackgroundColor3 = color3(16, 20, 30)
TabBar.BorderSizePixel = 0
TabBar.ScrollBarThickness = 2
TabBar.ScrollBarImageColor3 = color3(0, 140, 255)
TabBar.CanvasSize = UDim2.new(0, 0, 0, 0)
TabBar.AutomaticCanvasSize = Enum.AutomaticSize.X
TabBar.ScrollingDirection = Enum.ScrollingDirection.X
TabBar.Parent = Main
local tbC = Instance.new("UICorner"); tbC.CornerRadius = UDim.new(0,8); tbC.Parent = TabBar

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 4)
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
tabLayout.Parent = TabBar

local tabPad = Instance.new("UIPadding")
tabPad.PaddingLeft = UDim.new(0, 4)
tabPad.PaddingRight = UDim.new(0, 4)
tabPad.Parent = TabBar

-- =====================================================================
-- SISTEMA DE PÁGINAS
-- =====================================================================
local Pages = {}
local TabButtons = {}

local function createPage(name)
    local page = Instance.new("ScrollingFrame")
    page.Name = name
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.ScrollBarThickness = 3
    page.ScrollBarImageColor3 = color3(0, 140, 255)
    page.CanvasSize = UDim2.new(0, 0, 0, 0)
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.Visible = false
    page.Parent = Content

    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = page

    local pad = Instance.new("UIPadding")
    pad.PaddingTop = UDim.new(0, 4)
    pad.PaddingBottom = UDim.new(0, 8)
    pad.PaddingLeft = UDim.new(0, 4)
    pad.PaddingRight = UDim.new(0, 4)
    pad.Parent = page

    Pages[name] = page
    return page
end

local function createTabButton(name, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 82, 0, 30)
    btn.BackgroundColor3 = color3(22, 30, 45)
    btn.Text = name
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.TextColor3 = color3(200, 220, 255)
    btn.AutoButtonColor = false
    btn.LayoutOrder = order
    btn.Parent = TabBar
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 6); c.Parent = btn

    btn.MouseButton1Click:Connect(function()
        for n, p in pairs(Pages) do p.Visible = (n == name) end
        for n, b in pairs(TabButtons) do
            TweenService:Create(b, TweenInfo.new(0.15), {
                BackgroundColor3 = (n == name) and color3(0, 130, 240) or color3(22, 30, 45)
            }):Play()
        end
        createPage(name)
    end)

    TabButtons[name] = btn
    return btn
end

-- Cria as páginas
local pageNames = {"Home", "FPS Boost", "Graphics", "Remove Fog", "Stretched", "FOV", "Monitor", "Settings"}
for i, n in ipairs(pageNames) do createPage(n); createTabButton(n, i) end

-- Ativa home
Pages["Home"].Visible = true
TabButtons["Home"].BackgroundColor3 = color3(0, 130, 240)

-- =====================================================================
-- HELPERS DE UI
-- =====================================================================
local function createSection(parent, title)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, 30)
    f.BackgroundTransparency = 1
    f.Parent = parent
    local lbl = Instance.new("TextLabel")
    lbl.Text = title
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 13
    lbl.TextColor3 = color3(0, 170, 255)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, -10, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = f
    return f
end

local function createToggle(parent, label, default, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 40)
    frame.BackgroundColor3 = color3(18, 22, 32)
    frame.BorderSizePixel = 0
    frame.Parent = parent
    local fc = Instance.new("UICorner"); fc.CornerRadius = UDim.new(0, 6); fc.Parent = frame

    local lbl = Instance.new("TextLabel")
    lbl.Text = label
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 12
    lbl.TextColor3 = color3(230, 235, 245)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, -80, 1, 0)
    lbl.Position = UDim2.new(0, 12, 0, 0)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = frame

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(0, 40, 0, 20)
    bg.Position = UDim2.new(1, -52, 0.5, -10)
    bg.BackgroundColor3 = default and color3(0,150,255) or color3(45, 50, 60)
    bg.Parent = frame
    local bgc = Instance.new("UICorner"); bgc.CornerRadius = UDim.new(1,0); bgc.Parent = bg

    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0, 16, 0, 16)
    knob.Position = default and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
    knob.BackgroundColor3 = color3(255,255,255)
    knob.Parent = bg
    local kc = Instance.new("UICorner"); kc.CornerRadius = UDim.new(1,0); kc.Parent = knob

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1,0,1,0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.Parent = frame

    local state = default
    btn.MouseButton1Click:Connect(function()
        state = not state
        TweenService:Create(bg, TweenInfo.new(0.2), {BackgroundColor3 = state and color3(0,150,255) or color3(45,50,60)}):Play()
        TweenService:Create(knob, TweenInfo.new(0.2), {Position = state and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8)}):Play()
        if callback then safe(callback, state) end
    end)

    return frame, function(v)
        if v ~= state then
            state = v
            bg.BackgroundColor3 = state and color3(0,150,255) or color3(45,50,60)
            knob.Position = state and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8)
        end
    end
end

local function createButton(parent, label, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 36)
    btn.BackgroundColor3 = color3(0, 100, 200)
    btn.Text = label
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.TextColor3 = color3(255,255,255)
    btn.AutoButtonColor = false
    btn.Parent = parent
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,6); c.Parent = btn
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = color3(0,140,240)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = color3(0,100,200)}):Play()
    end)
    btn.MouseButton1Click:Connect(function() safe(callback) end)
    return btn
end

local function createSlider(parent, label, minV, maxV, default, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 54)
    frame.BackgroundColor3 = color3(18, 22, 32)
    frame.BorderSizePixel = 0
    frame.Parent = parent
    local fc = Instance.new("UICorner"); fc.CornerRadius = UDim.new(0,6); fc.Parent = frame

    local lbl = Instance.new("TextLabel")
    lbl.Text = label .. ": " .. tostring(default)
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 12
    lbl.TextColor3 = color3(230,235,245)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, -20, 0, 20)
    lbl.Position = UDim2.new(0, 12, 0, 4)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = frame

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1, -24, 0, 6)
    bar.Position = UDim2.new(0, 12, 0, 34)
    bar.BackgroundColor3 = color3(40, 45, 55)
    bar.Parent = frame
    local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(1,0); bc.Parent = bar

    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((default-minV)/(maxV-minV), 0, 1, 0)
    fill.BackgroundColor3 = color3(0,150,255)
    fill.Parent = bar
    local fillc = Instance.new("UICorner"); fillc.CornerRadius = UDim.new(1,0); fillc.Parent = fill

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1,0,1,0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.Parent = frame

    local value = default
    local function updateFromX(x)
        local rel = math.clamp((x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
        value = math.floor(minV + rel * (maxV - minV) + 0.5)
        fill.Size = UDim2.new(rel, 0, 1, 0)
        lbl.Text = label .. ": " .. tostring(value)
        if callback then safe(callback, value) end
    end

    btn.MouseButton1Down:Connect(function()
        updateFromX(UserInputService:GetMouseLocation().X)
        local moveConn
        moveConn = UserInputService.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                updateFromX(input.Position.X)
            end
        end)
        local endConn
        endConn = UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                moveConn:Disconnect()
                endConn:Disconnect()
            end
        end)
    end)

    return frame, function(v)
        value = v
        local rel = math.clamp((v - minV) / (maxV - minV), 0, 1)
        fill.Size = UDim2.new(rel, 0, 1, 0)
        lbl.Text = label .. ": " .. tostring(v)
    end
end

local function createLabel(parent, text, color, size)
    local lbl = Instance.new("TextLabel")
    lbl.Text = text
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = size or 12
    lbl.TextColor3 = color or color3(180, 200, 230)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, -20, 0, 0)
    lbl.AutomaticSize = Enum.AutomaticSize.Y
    lbl.TextWrapped = true
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = parent
    local pad = Instance.new("UIPadding"); pad.PaddingLeft = UDim.new(0,10); pad.PaddingRight = UDim.new(0,10); pad.Parent = lbl
    return lbl
end

-- =====================================================================
-- NOTIFICAÇÕES
-- =====================================================================
local NotifContainer = Instance.new("Frame")
NotifContainer.Size = UDim2.new(0, 260, 1, 0)
NotifContainer.Position = UDim2.new(1, -270, 0, 0)
NotifContainer.BackgroundTransparency = 1
NotifContainer.Parent = ScreenGui

local notifLayout = Instance.new("UIListLayout")
notifLayout.Padding = UDim.new(0, 6)
notifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
notifLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
notifLayout.SortOrder = Enum.SortOrder.LayoutOrder
notifLayout.Parent = NotifContainer

local function notify(text)
    safe(function()
        local n = Instance.new("TextLabel")
        n.Size = UDim2.new(1, 0, 0, 36)
        n.BackgroundColor3 = color3(12, 16, 24)
        n.Text = "  " .. text
        n.Font = Enum.Font.GothamSemibold
        n.TextSize = 12
        n.TextColor3 = color3(220, 235, 255)
        n.TextXAlignment = Enum.TextXAlignment.Left
        n.Parent = NotifContainer
        local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,6); c.Parent = n
        local s = Instance.new("UIStroke"); s.Color = color3(0,150,255); s.Transparency = 0.3; s.Parent = n
        task.delay(2.5, function()
            if n and n.Parent then
                TweenService:Create(n, TweenInfo.new(0.3), {BackgroundTransparency = 1, TextTransparency = 1}):Play()
                task.wait(0.35)
                n:Destroy()
            end
        end)
    end)
end

-- =====================================================================
-- MÓDULO: FPS BOOST
-- =====================================================================
local function snapshotLighting()
    safe(function()
        saveOriginal(Lighting, "Brightness")
        saveOriginal(Lighting, "Ambient")
        saveOriginal(Lighting, "OutdoorAmbient")
        saveOriginal(Lighting, "GlobalShadows")
        saveOriginal(Lighting, "EnvironmentDiffuseScale")
        saveOriginal(Lighting, "EnvironmentSpecularScale")
        saveOriginal(Lighting, "ShadowSoftness")
        saveOriginal(Lighting, "ExposureCompensation")
    end)
end

local function applyFPSMode(mode)
    snapshotLighting()
    if mode == "Off" then
        restoreOriginal(Lighting, "GlobalShadows")
        restoreOriginal(Lighting, "Brightness")
        restoreOriginal(Lighting, "EnvironmentDiffuseScale")
        restoreOriginal(Lighting, "EnvironmentSpecularScale")
        notify("FPS Boost desativado")
        return
    end
    if mode == "Low" then
        setProp(Lighting, "GlobalShadows", false)
        setProp(Lighting, "Brightness", 1)
        setProp(Lighting, "EnvironmentDiffuseScale", 0)
        setProp(Lighting, "EnvironmentSpecularScale", 0)
        notify("FPS Boost: Low")
    elseif mode == "Medium" then
        setProp(Lighting, "GlobalShadows", true)
        setProp(Lighting, "Brightness", 1.5)
        setProp(Lighting, "EnvironmentDiffuseScale", 0.15)
        setProp(Lighting, "EnvironmentSpecularScale", 0.15)
        notify("FPS Boost: Medium")
    elseif mode == "High" then
        setProp(Lighting, "GlobalShadows", true)
        setProp(Lighting, "Brightness", 2)
        setProp(Lighting, "EnvironmentDiffuseScale", 0.4)
        setProp(Lighting, "EnvironmentSpecularScale", 0.4)
        notify("FPS Boost: High")
    end
    RH.Flags.FPSMode = mode
    RH.Flags.FPSBoost = (mode ~= "Off")
end

-- =====================================================================
-- MÓDULO: GRAPHICS
-- =====================================================================
local function setPostFXEnabled(enabled)
    for _, fx in ipairs(Lighting:GetChildren()) do
        if fx:IsA("BlurEffect") or fx:IsA("BloomEffect") or fx:IsA("ColorCorrectionEffect")
            or fx:IsA("SunRaysEffect") or fx:IsA("DepthOfFieldEffect") then
            saveOriginal(fx, "Enabled")
            setProp(fx, "Enabled", enabled)
        end
    end
end

local function setParticlesEnabled(enabled)
    safe(function()
        for _, d in ipairs(workspace:GetDescendants()) do
            if d:IsA("ParticleEmitter") or d:IsA("Smoke") or d:IsA("Fire") or d:IsA("Sparkles") then
                saveOriginal(d, "Enabled")
                setProp(d, "Enabled", enabled)
            end
        end
    end)
end

local function applyGraphicsMode(mode)
    RH.Flags.GraphicsMode = mode
    if mode == "Off" then
        -- Restaura FX
        for _, fx in ipairs(Lighting:GetChildren()) do
            if fx:IsA("PostEffect") then restoreOriginal(fx, "Enabled") end
        end
        notify("Graphics: restaurado")
        return
    end
    if mode == "Min" then
        setPostFXEnabled(false)
        snapshotLighting()
        setProp(Lighting, "GlobalShadows", false)
        setProp(Lighting, "Brightness", 1)
        notify("Graphics: mínimo")
    elseif mode == "Balanced" then
        setPostFXEnabled(true)
        snapshotLighting()
        setProp(Lighting, "GlobalShadows", true)
        notify("Graphics: equilibrado")
    elseif mode == "HighPerf" then
        setPostFXEnabled(false)
        snapshotLighting()
        setProp(Lighting, "GlobalShadows", false)
        setProp(Lighting, "Brightness", 0.8)
        notify("Graphics: alto desempenho")
    end
end

-- =====================================================================
-- MÓDULO: FOG
-- =====================================================================
local function removeFog()
    snapshotLighting()
    saveOriginal(Lighting, "FogStart")
    saveOriginal(Lighting, "FogEnd")
    saveOriginal(Lighting, "FogColor")
    setProp(Lighting, "FogStart", 0)
    setProp(Lighting, "FogEnd", 100000)
    for _, a in ipairs(Lighting:GetChildren()) do
        if a:IsA("Atmosphere") then
            saveOriginal(a, "Density")
            saveOriginal(a, "Haze")
            saveOriginal(a, "Glare")
            setProp(a, "Density", 0)
            setProp(a, "Haze", 0)
            setProp(a, "Glare", 0)
        end
    end
    notify("Neblina removida")
end

local function reduceFog()
    snapshotLighting()
    saveOriginal(Lighting, "FogStart")
    saveOriginal(Lighting, "FogEnd")
    if isProperty(Lighting, "FogStart") and Lighting.FogStart then
        setProp(Lighting, "FogStart", math.min(Lighting.FogStart + 100, 10000))
    end
    if isProperty(Lighting, "FogEnd") and Lighting.FogEnd then
        setProp(Lighting, "FogEnd", math.max(Lighting.FogEnd * 2, 5000))
    end
    for _, a in ipairs(Lighting:GetChildren()) do
        if a:IsA("Atmosphere") then
            saveOriginal(a, "Density")
            setProp(a, "Density", (a.Density or 0) * 0.3)
        end
    end
    notify("Neblina reduzida")
end

local function restoreFog()
    restoreOriginal(Lighting, "FogStart")
    restoreOriginal(Lighting, "FogEnd")
    restoreOriginal(Lighting, "FogColor")
    for _, a in ipairs(Lighting:GetChildren()) do
        if a:IsA("Atmosphere") then
            restoreOriginal(a, "Density")
            restoreOriginal(a, "Haze")
            restoreOriginal(a, "Glare")
        end
    end
    notify("Neblina restaurada")
end

-- =====================================================================
-- MÓDULO: FOV
-- =====================================================================
local function setFOV(v)
    local cam = workspace.CurrentCamera
    if not cam then return end
    v = math.clamp(v, 40, 120)
    if isProperty(cam, "FieldOfView") then
        cam.FieldOfView = v
        RH.Flags.FOV = v
    end
end

-- =====================================================================
-- FPS COUNTER
-- =====================================================================
local FPSGui = Instance.new("TextLabel")
FPSGui.Name = "RandomHubFPS"
FPSGui.Size = UDim2.new(0, 130, 0, 28)
FPSGui.Position = UDim2.new(0, 10, 0, 10)
FPSGui.BackgroundColor3 = color3(0, 0, 0)
FPSGui.BackgroundTransparency = 0.35
FPSGui.TextColor3 = color3(0, 200, 255)
FPSGui.Font = Enum.Font.Code
FPSGui.TextSize = 14
FPSGui.Text = "FPS: --"
FPSGui.Visible = false
FPSGui.Parent = ScreenGui
local fgCorner = Instance.new("UICorner"); fgCorner.CornerRadius = UDim.new(0,6); fgCorner.Parent = FPSGui

local fpsConn
local function startFPS()
    if fpsConn then return end
    local frames, acc = 0, 0
    fpsConn = RunService.RenderStepped:Connect(function(dt)
        frames += 1
        acc += dt
        if acc >= 1 then
            FPSGui.Text = string.format("FPS: %d", frames)
            frames, acc = 0, 0
        end
    end)
end

local function stopFPS()
    if fpsConn then fpsConn:Disconnect(); fpsConn = nil end
    FPSGui.Text = "FPS: --"
end

-- =====================================================================
-- PING MONITOR
-- =====================================================================
local PingGui = Instance.new("TextLabel")
PingGui.Name = "RandomHubPing"
PingGui.Size = UDim2.new(0, 130, 0, 28)
PingGui.Position = UDim2.new(0, 10, 0, 44)
PingGui.BackgroundColor3 = color3(0, 0, 0)
PingGui.BackgroundTransparency = 0.35
PingGui.TextColor3 = color3(0, 255, 150)
PingGui.Font = Enum.Font.Code
PingGui.TextSize = 14
PingGui.Text = "Ping: --"
PingGui.Visible = false
PingGui.Parent = ScreenGui
local pgCorner = Instance.new("UICorner"); pgCorner.CornerRadius = UDim.new(0,6); pgCorner.Parent = PingGui

local pingConn
local function getPing()
    if not Stats then return nil end
    local ok, val = pcall(function()
        local net = Stats.Network
        if net and net.ServerStatsItem then
            local item = net.ServerStatsItem["Data Ping"]
            if item then return item:GetValue() end
        end
        return nil
    end)
    if ok and val then return val end
    -- fallback: usar PerformanceStats se disponível
    local ok2, p = pcall(function()
        local perf = Stats.PerformanceStats
        if perf and perf.Ping then return perf.Ping:GetValue() end
    end)
    return ok2 and p or nil
end

local function startPing()
    if pingConn then return end
    local tick = 0
    pingConn = RunService.Heartbeat:Connect(function()
        tick += 1
        if tick < 60 then return end
        tick = 0
        local p = getPing()
        if p and p > 0 then
            PingGui.Text = string.format("Ping: %d ms", math.floor(p))
            if p < 80 then PingGui.TextColor3 = color3(0,255,150)
            elseif p < 150 then PingGui.TextColor3 = color3(255,220,0)
            else PingGui.TextColor3 = color3(255,80,80) end
        else
            PingGui.Text = "Ping: indisponível"
            PingGui.TextColor3 = color3(180,180,180)
        end
    end)
end

local function stopPing()
    if pingConn then pingConn:Disconnect(); pingConn = nil end
    PingGui.Text = "Ping: --"
end

-- =====================================================================
-- FLAGS DE API INTERNA
-- =====================================================================
function RH:ToggleFPSCounter(on)
    RH.Flags.FPSCounter = on
    FPSGui.Visible = on
    if on then startFPS() else stopFPS() end
end

function RH:TogglePing(on)
    RH.Flags.PingMonitor = on
    PingGui.Visible = on
    if on then startPing() else stopPing() end
end

-- =====================================================================
-- PÁGINA: HOME
-- =====================================================================
safe(function()
    local home = Pages["Home"]
    local title = Instance.new("TextLabel")
    title.Text = "RANDOM HUB"
    title.Font = Enum.Font.GothamBlack
    title.TextSize = 24
    title.TextColor3 = color3(0, 170, 255)
    title.BackgroundTransparency = 1
    title.Size = UDim2.new(1, 0, 0, 34)
    title.Parent = home

    createLabel(home, "Ultimate Optimizer — preto, azul e branco.", color3(180,200,230), 12)
    createLabel(home, "Ative recursos individualmente. Todos os valores originais são salvos e podem ser restaurados na aba Settings.", color3(150,170,200), 11)

    createSection(home, "Status")
    local statusLbl = Instance.new("TextLabel")
    statusLbl.Font = Enum.Font.Code
    statusLbl.TextSize = 11
    statusLbl.TextColor3 = color3(0, 200, 255)
    statusLbl.BackgroundColor3 = color3(14,18,28)
    statusLbl.Size = UDim2.new(1, 0, 0, 100)
    statusLbl.TextWrapped = true
    statusLbl.TextXAlignment = Enum.TextXAlignment.Left
    statusLbl.TextYAlignment = Enum.TextYAlignment.Top
    statusLbl.Parent = home
    local slc = Instance.new("UICorner"); slc.CornerRadius = UDim.new(0,6); slc.Parent = statusLbl
    local slp = Instance.new("UIPadding"); slp.PaddingLeft = UDim.new(0,10); slp.PaddingTop = UDim.new(0,8); slp.Parent = statusLbl

    task.spawn(function()
        while ScreenGui.Parent do
            statusLbl.Text = string.format(
                "FPS Boost: %s\nGraphics: %s\nFog: %s\nFOV: %d\nFPS Counter: %s\nPing: %s",
                RH.Flags.FPSBoost and RH.Flags.FPSMode or "Off",
                RH.Flags.GraphicsMode,
                RH.Flags.RemoveFog and "Removida" or (RH.Flags.ReduceFog and "Reduzida" or "Off"),
                RH.Flags.FOV,
                RH.Flags.FPSCounter and "ON" or "OFF",
                RH.Flags.PingMonitor and "ON" or "OFF"
            )
            task.wait(0.5)
        end
    end)

    createButton(home, "Aplicar configurações recomendadas", function()
        applyFPSMode("Medium")
        applyGraphicsMode("Balanced")
        RH.Flags.FPSCounter = true; FPSGui.Visible = true; startFPS()
        RH.Flags.PingMonitor = true; PingGui.Visible = true; startPing()
        setFOV(75)
        notify("Configurações recomendadas aplicadas")
    end)
end)

-- =====================================================================
-- PÁGINA: FPS BOOST
-- =====================================================================
safe(function()
    local p = Pages["FPS Boost"]
    createLabel(p, "Escolha um modo de desempenho. As alterações afetam apenas o seu cliente.", color3(150,170,200), 11)
    createSection(p, "Modos")
    createButton(p, "Low Performance", function() applyFPSMode("Low") end)
    createButton(p, "Medium Performance", function() applyFPSMode("Medium") end)
    createButton(p, "High Performance", function() applyFPSMode("High") end)
    createButton(p, "Desativar FPS Boost", function() applyFPSMode("Off") end)
    createSection(p, "Restauração")
    createButton(p, "Restaurar configurações originais", function()
        for k, v in pairs(RH.Original) do
            -- percorre e restaura
        end
        for _, key in ipairs({"Brightness","Ambient","OutdoorAmbient","GlobalShadows","EnvironmentDiffuseScale","EnvironmentSpecularScale","ShadowSoftness","ExposureCompensation"}) do
            restoreOriginal(Lighting, key)
        end
        RH.Flags.FPSBoost = false
        RH.Flags.FPSMode = "Off"
        notify("Configurações de iluminação restauradas")
    end)
end)

-- =====================================================================
-- PÁGINA: GRAPHICS
-- =====================================================================
safe(function()
    local p = Pages["Graphics"]
    createSection(p, "Modos Gráficos")
    createButton(p, "Gráfico Mínimo", function() applyGraphicsMode("Min") end)
    createButton(p, "Gráfico Equilibrado", function() applyGraphicsMode("Balanced") end)
    createButton(p, "Alto Desempenho", function() applyGraphicsMode("HighPerf") end)
    createButton(p, "Restaurar gráficos", function() applyGraphicsMode("Off") end)

    createSection(p, "Efeitos")
    createToggle(p, "Reduzir partículas", false, function(on)
        RH.Flags.ReduceParticles = on
        setParticlesEnabled(not on)
        notify(on and "Partículas reduzidas" or "Partículas restauradas")
    end)
    createToggle(p, "Reduzir efeitos de pós-processamento", false, function(on)
        RH.Flags.ReducePostFX = on
        setPostFXEnabled(not on)
        notify(on and "Pós-processamento reduzido" or "Pós-processamento restaurado")
    end)
    createToggle(p, "Reduzir sombras", false, function(on)
        snapshotLighting()
        setProp(Lighting, "GlobalShadows", not on)
        notify(on and "Sombras reduzidas" or "Sombras restauradas")
    end)
end)

-- =====================================================================
-- PÁGINA: REMOVE FOG
-- =====================================================================
safe(function()
    local p = Pages["Remove Fog"]
    createLabel(p, "Controle de neblina local e atmosfera.", color3(150,170,200), 11)
    createToggle(p, "Remover neblina", false, function(on)
        RH.Flags.RemoveFog = on
        if on then removeFog() else restoreFog() end
    end)
    createToggle(p, "Reduzir intensidade da neblina", false, function(on)
        RH.Flags.ReduceFog = on
        if on then reduceFog() else restoreFog() end
    end)
    createButton(p, "Restaurar neblina", restoreFog)
end)

-- =====================================================================
-- PÁGINA: STRETCHED SCREEN
-- =====================================================================
safe(function()
    local p = Pages["Stretched"]
    createLabel(p, "Ajustes de tela e visualização. O Roblox não permite forçar resolução real fora do que o cliente oferece — use FOV e escala para efeito similar.", color3(150,170,200), 11)
    createSection(p, "Interface")
    local _, scaleSetter = createSlider(p, "Escala da interface", 70, 130, 100, function(v)
        RH.Flags.InterfaceScale = v/100
        Main.Size = UDim2.new(0, baseWidth * (v/100), 0, baseHeight * (v/100))
        Main.Position = UDim2.new(0.5, -baseWidth*(v/100)/2, 0.5, -baseHeight*(v/100)/2)
    end)

    createButton(p, "Visualização compacta", function()
        Main.Size = UDim2.new(0, baseWidth * 0.7, 0, baseHeight * 0.7)
        Main.Position = UDim2.new(0.5, -baseWidth*0.35, 0.5, -baseHeight*0.35)
        notify("Modo compacto")
    end)
    createButton(p, "Visualização expandida", function()
        Main.Size = UDim2.new(0, baseWidth * 1.3, 0, baseHeight * 1.3)
        Main.Position = UDim2.new(0.5, -baseWidth*0.65, 0.5, -baseHeight*0.65)
        notify("Modo expandido")
    end)
    createButton(p, "Restaurar tamanho padrão", function()
        Main.Size = UDim2.new(0, baseWidth, 0, baseHeight)
        Main.Position = UDim2.new(0.5, -baseWidth/2, 0.5, -baseHeight/2)
        notify("Tamanho restaurado")
    end)
end)

-- =====================================================================
-- PÁGINA: FOV
-- =====================================================================
safe(function()
    local p = Pages["FOV"]
    if not workspace.CurrentCamera then
        createLabel(p, "Câmera indisponível no momento.", color3(255,100,100), 12)
        return
    end
    saveOriginal(workspace.CurrentCamera, "FieldOfView")
    local default = workspace.CurrentCamera.FieldOfView
    RH.Flags.FOV = default
    createSlider(p, "Field of View", 40, 120, default, function(v) setFOV(v) end)
    createButton(p, "Aumentar FOV (+5)", function() setFOV(RH.Flags.FOV + 5) end)
    createButton(p, "Diminuir FOV (-5)", function() setFOV(RH.Flags.FOV - 5) end)
    createButton(p, "Restaurar FOV padrão", function()
        setFOV(default)
        notify("FOV restaurado para " .. tostring(default))
    end)
end)

-- =====================================================================
-- PÁGINA: MONITOR
-- =====================================================================
safe(function()
    local p = Pages["Monitor"]
    createToggle(p, "Contador de FPS", false, function(on) RH:ToggleFPSCounter(on) end)
    createToggle(p, "Monitor de Ping", false, function(on) RH:TogglePing(on) end)
    createLabel(p, "O ping depende de Stats.Network. Caso o jogo/executor não exponha, será exibido 'indisponível'.", color3(150,170,200), 11)

    createSection(p, "Posição do indicador")
    local positions = {
        {"Topo Esquerdo", UDim2.new(0,10,0,10), UDim2.new(0,10,0,44)},
        {"Topo Direito",  UDim2.new(1,-140,0,10), UDim2.new(1,-140,0,44)},
        {"Baixo Esquerdo",UDim2.new(0,10,1,-74), UDim2.new(0,10,1,-40)},
        {"Baixo Direito", UDim2.new(1,-140,1,-74), UDim2.new(1,-140,1,-40)},
    }
    for _, info in ipairs(positions) do
        createButton(p, info[1], function()
            FPSGui.Position = info[2]
            PingGui.Position = info[3]
            notify("Posição: " .. info[1])
        end)
    end
end)

-- =====================================================================
-- PÁGINA: SETTINGS
-- =====================================================================
safe(function()
    local p = Pages["Settings"]
    createSection(p, "Preferências")
    createButton(p, "Aplicar configurações recomendadas", function()
        applyFPSMode("Medium")
        applyGraphicsMode("Balanced")
        RH:ToggleFPSCounter(true)
        RH:TogglePing(true)
        setFOV(75)
        notify("Configurações recomendadas aplicadas")
    end)
    createButton(p, "Restaurar configurações padrão", function()
        -- FPS
        applyFPSMode("Off")
        -- Graphics
        applyGraphicsMode("Off")
        -- Fog
        restoreFog()
        RH.Flags.RemoveFog = false
        RH.Flags.ReduceFog = false
        -- FOV
        if workspace.CurrentCamera then
            local key = tostring(workspace.CurrentCamera) .. ".FieldOfView"
            if RH.Original[key] then
                workspace.CurrentCamera.FieldOfView = RH.Original[key]
            end
        end
        -- Monitores
        RH:ToggleFPSCounter(false)
        RH:TogglePing(false)
        notify("Configurações padrão restauradas")
    end)

    createSection(p, "Restauração Completa")
    createButton(p, "Restaurar TODAS as alterações", function()
        safe(function()
            -- Lighting
            for _, key in ipairs({"Brightness","Ambient","OutdoorAmbient","GlobalShadows",
                "EnvironmentDiffuseScale","EnvironmentSpecularScale","ShadowSoftness",
                "ExposureCompensation","FogStart","FogEnd","FogColor"}) do
                restoreOriginal(Lighting, key)
            end
            for _, fx in ipairs(Lighting:GetChildren()) do
                if fx:IsA("PostEffect") then restoreOriginal(fx, "Enabled") end
                if fx:IsA("Atmosphere") then
                    restoreOriginal(fx, "Density")
                    restoreOriginal(fx, "Haze")
                    restoreOriginal(fx, "Glare")
                end
            end
            -- FOV
            if workspace.CurrentCamera then
                local k = tostring(workspace.CurrentCamera) .. ".FieldOfView"
                if RH.Original[k] then
                    workspace.CurrentCamera.FieldOfView = RH.Original[k]
                    RH.Original[k] = nil
                end
            end
            RH.Flags.FPSBoost = false
            RH.Flags.FPSMode = "Off"
            RH.Flags.GraphicsMode = "Off"
            RH.Flags.RemoveFog = false
            RH.Flags.ReduceFog = false
            notify("Todas as alterações foram restauradas")
        end)
    end)

    createButton(p, "Fechar RANDOM HUB", function()
        safe(function()
            RH:ToggleFPSCounter(false)
            RH:TogglePing(false)
        end)
        for _, c in ipairs(RH.Connections) do pcall(function() c:Disconnect() end) end
        ScreenGui:Destroy()
    end)

    createSection(p, "Informação")
    createLabel(p, "Recursos dependem do jogo, dispositivo e executor. Nenhuma alteração modifica servidores ou a memória protegida do Roblox.", color3(150,170,200), 11)
end)

-- =====================================================================
-- RESIZE RESPONSIVO
-- =====================================================================
local function updateResponsive()
    local vp = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1280,720)
    local w = math.clamp(vp.X * 0.9, 280, baseWidth)
    local h = math.clamp(vp.Y * 0.8, 320, baseHeight)
    Main.Size = UDim2.new(0, w, 0, h)
    Main.Position = UDim2.new(0.5, -w/2, 0.5, -h/2)
end

if workspace.CurrentCamera then
    connect(workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"), updateResponsive)
    updateResponsive()
end

-- =====================================================================
-- HUD DE MONITOR - atalhos de teclado (opcional)
-- =====================================================================
connect(UserInputService.InputBegan, function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        Main.Visible = not Main.Visible
        ToggleBtn.Visible = not Main.Visible
    end
end)

-- =====================================================================
-- LIMPEZA AO FECHAR O JOGO
-- =====================================================================
game:BindToClose(function()
    safe(function()
        RH:ToggleFPSCounter(false)
        RH:TogglePing(false)
        for _, c in ipairs(RH.Connections) do pcall(function() c:Disconnect() end) end
    end)
end)

notify("RANDOM HUB carregado com sucesso")
