-- Tela de login aprimorada (LocalScript)
-- Coloque em StarterPlayerScripts ou dentro de PlayerGui

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Tema
local COLOR_BG = Color3.fromRGB(20, 24, 28)
local COLOR_PANEL = Color3.fromRGB(28, 33, 38)
local COLOR_ACCENT = Color3.fromRGB(40, 120, 255)
local COLOR_TEXT = Color3.fromRGB(230, 235, 240)
local COLOR_SUB = Color3.fromRGB(170, 175, 180)

-- Key mapping (edite conforme desejar)
local keyMapping = {
    ["BL-ABC123"] = "brazloucuras",
    ["YR-XYZ789"] = "Yrnk",
    ["AB-456DEF"] = "abuso",
    ["8D4945C3EF9E79D8"] = "owner",
    ["97BDFB91B728EE9B"] = "we",
}

local function safeText(t) return tostring(t or "") end

-- Notificações simples (temporárias)
local function notifyShort(title, text, duration)
    duration = duration or 2
    pcall(function()
        local gui = Instance.new("ScreenGui")
        gui.Name = "StoppedLoginNotify"
        gui.ResetOnSpawn = false
        gui.Parent = PlayerGui

        local frame = Instance.new("Frame", gui)
        frame.Size = UDim2.new(0, 360, 0, 72)
        frame.Position = UDim2.new(0.5, -180, 0, 36)
        frame.AnchorPoint = Vector2.new(0.5, 0)
        frame.BackgroundColor3 = COLOR_BG
        frame.BackgroundTransparency = 0.02
        frame.ZIndex = 10
        Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

        local titleLbl = Instance.new("TextLabel", frame)
        titleLbl.Size = UDim2.new(1, -24, 0, 20)
        titleLbl.Position = UDim2.new(0, 12, 0, 8)
        titleLbl.BackgroundTransparency = 1
        titleLbl.Font = Enum.Font.GothamBold
        titleLbl.TextSize = 14
        titleLbl.Text = safeText(title)
        titleLbl.TextColor3 = COLOR_TEXT
        titleLbl.TextXAlignment = Enum.TextXAlignment.Left

        local body = Instance.new("TextLabel", frame)
        body.Size = UDim2.new(1, -24, 0, 36)
        body.Position = UDim2.new(0, 12, 0, 28)
        body.BackgroundTransparency = 1
        body.Font = Enum.Font.Gotham
        body.TextSize = 13
        body.Text = safeText(text)
        body.TextColor3 = COLOR_SUB
        body.TextXAlignment = Enum.TextXAlignment.Left
        body.TextYAlignment = Enum.TextYAlignment.Top
        body.TextWrapped = true

        task.delay(duration, function()
            pcall(function() gui:Destroy() end)
        end)
    end)
end

-- Construção da UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "StoppedLoginGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local main = Instance.new("Frame", screenGui)
main.Name = "Main"
main.Size = UDim2.new(0, 620, 0, 220)
main.Position = UDim2.new(0.5, 0, 0.4, 0)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.BackgroundColor3 = COLOR_PANEL
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 10)

-- header com logo, título e botão fechar (X)
local header = Instance.new("Frame", main)
header.Size = UDim2.new(1, -24, 0, 72)
header.Position = UDim2.new(0, 12, 0, 12)
header.BackgroundColor3 = COLOR_BG
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 8)

local logo = Instance.new("ImageLabel", header)
logo.Position = UDim2.new(0, 12, 0.5, 0)
logo.AnchorPoint = Vector2.new(0, 0.5)
logo.Size = UDim2.new(0, 48, 0, 48)
logo.Image = "rbxassetid://117130789916072"
logo.BackgroundTransparency = 1
logo.ImageColor3 = COLOR_ACCENT

local title = Instance.new("TextLabel", header)
title.Position = UDim2.new(0, 72, 0.32, 0)
title.Size = UDim2.new(0.7, 0, 0, 24)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.Text = "STOPPED - Login"
title.TextColor3 = COLOR_TEXT
title.TextXAlignment = Enum.TextXAlignment.Left

local subtitle = Instance.new("TextLabel", header)
subtitle.Position = UDim2.new(0, 72, 0.68, 0)
subtitle.Size = UDim2.new(0.8, 0, 0, 16)
subtitle.BackgroundTransparency = 1
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 12
subtitle.Text = "Insira sua key de apoiador para desbloquear recursos"
subtitle.TextColor3 = COLOR_SUB
subtitle.TextXAlignment = Enum.TextXAlignment.Left

-- botão fechar (X)
local closeBtn = Instance.new("TextButton", header)
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -40, 0.5, 0)
closeBtn.AnchorPoint = Vector2.new(0.5, 0.5)
closeBtn.BackgroundColor3 = Color3.fromRGB(40, 44, 50)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(240, 100, 100)
closeBtn.TextSize = 16
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)
closeBtn.AutoButtonColor = true
closeBtn.MouseButton1Click:Connect(function()
    pcall(function() screenGui:Destroy() end)
end)

-- area principal (status em cima do label Key)
local statusLabel = Instance.new("TextLabel", main)
statusLabel.Size = UDim2.new(1, -24, 0, 18)
statusLabel.Position = UDim2.new(0, 12, 0, 96)
statusLabel.BackgroundTransparency = 1
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextSize = 14
statusLabel.Text = "Status: Nenhuma key validada."
statusLabel.TextColor3 = COLOR_SUB
statusLabel.TextXAlignment = Enum.TextXAlignment.Left

local keyLabel = Instance.new("TextLabel", main)
keyLabel.Size = UDim2.new(0, 48, 0, 16)
keyLabel.Position = UDim2.new(0, 12, 0, 116)
keyLabel.BackgroundTransparency = 1
keyLabel.Font = Enum.Font.Gotham
keyLabel.TextSize = 13
keyLabel.Text = "Key:"
keyLabel.TextColor3 = COLOR_TEXT
keyLabel.TextXAlignment = Enum.TextXAlignment.Left

-- linha com input e botão validar
local inputRow = Instance.new("Frame", main)
inputRow.Size = UDim2.new(1, -24, 0, 36)
inputRow.Position = UDim2.new(0, 12, 0, 132)
inputRow.BackgroundTransparency = 1

local inputBox = Instance.new("TextBox", inputRow)
inputBox.Size = UDim2.new(0.78, -8, 1, 0)
inputBox.Position = UDim2.new(0, 0, 0, 0)
inputBox.BackgroundColor3 = Color3.fromRGB(40, 44, 50)
inputBox.PlaceholderText = "Cole sua key aqui..."
inputBox.Text = ""
inputBox.TextColor3 = COLOR_TEXT
inputBox.Font = Enum.Font.Gotham
inputBox.TextSize = 14
Instance.new("UICorner", inputBox).CornerRadius = UDim.new(0, 6)

local validateBtn = Instance.new("TextButton", inputRow)
validateBtn.Size = UDim2.new(0.22, 0, 1, 0)
validateBtn.Position = UDim2.new(0.78, 6, 0, 0)
validateBtn.BackgroundColor3 = COLOR_ACCENT
validateBtn.Font = Enum.Font.GothamBold
validateBtn.Text = "Validar"
validateBtn.TextColor3 = Color3.fromRGB(255,255,255)
validateBtn.TextSize = 14
Instance.new("UICorner", validateBtn).CornerRadius = UDim.new(0, 8)

-- overlay de "Carregando..."
local overlay = Instance.new("Frame", main)
overlay.Size = UDim2.new(1, 0, 1, 0)
overlay.Position = UDim2.new(0, 0, 0, 0)
overlay.BackgroundColor3 = COLOR_BG
overlay.BackgroundTransparency = 1
overlay.ZIndex = 50
overlay.Visible = false
Instance.new("UICorner", overlay).CornerRadius = UDim.new(0, 10)

local carregandoLabel = Instance.new("TextLabel", overlay)
carregandoLabel.Size = UDim2.new(1, 0, 0, 36)
carregandoLabel.Position = UDim2.new(0, 0, 0.45, 0)
carregandoLabel.AnchorPoint = Vector2.new(0.5, 0.5)
carregandoLabel.BackgroundTransparency = 1
carregandoLabel.Font = Enum.Font.GothamBold
carregandoLabel.TextSize = 20
carregandoLabel.Text = "Carregando..."
carregandoLabel.TextColor3 = COLOR_ACCENT
carregandoLabel.TextStrokeTransparency = 0.85
carregandoLabel.ZIndex = 51
carregandoLabel.TextXAlignment = Enum.TextXAlignment.Center
carregandoLabel.Position = UDim2.new(0.5, 0, 0.45, 0)

local progBarBG = Instance.new("Frame", overlay)
progBarBG.Size = UDim2.new(0.6, 0, 0, 10)
progBarBG.Position = UDim2.new(0.2, 0, 0.62, 0)
progBarBG.BackgroundColor3 = Color3.fromRGB(40,40,44)
Instance.new("UICorner", progBarBG).CornerRadius = UDim.new(1, 0)
progBarBG.ZIndex = 51

local progFill = Instance.new("Frame", progBarBG)
progFill.Size = UDim2.new(0, 0, 1, 0)
progFill.Position = UDim2.new(0, 0, 0, 0)
progFill.BackgroundColor3 = COLOR_ACCENT
Instance.new("UICorner", progFill).CornerRadius = UDim.new(1, 0)
progFill.ZIndex = 52

-- validação e comportamento ao carregar
local function fadeOutAndDestroyGui(root)
    pcall(function()
        local tweenInfo = TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut)
        local t1 = TweenService:Create(root, tweenInfo, {BackgroundTransparency = 1})
        t1:Play()
        -- fade children text
        for _, obj in ipairs(root:GetDescendants()) do
            if obj:IsA("TextLabel") or obj:IsA("TextBox") or obj:IsA("TextButton") then
                pcall(function()
                    TweenService:Create(obj, tweenInfo, {TextTransparency = 1}):Play()
                end)
            elseif obj:IsA("ImageLabel") then
                pcall(function()
                    TweenService:Create(obj, tweenInfo, {ImageTransparency = 1}):Play()
                end)
            end
        end
        task.delay(0.5, function()
            pcall(function() screenGui:Destroy() end)
        end)
    end)
end

local function executeRemote()
    local ok, err = pcall(function()
        local url = "https://raw.githubusercontent.com/guzinsamuel10-hub/stoppedhaze/refs/heads/main/README.md"
        local resp = game:HttpGet(url)
        local f, loadErr = loadstring(resp)
        if not f then error("Loadstring falhou: "..tostring(loadErr)) end
        f()
    end)
    if not ok then
        notifyShort("Erro", "Falha ao executar código remoto: "..tostring(err), 5)
        warn("StoppedLogin: erro ao executar remote code:", err)
    end
end

local function startLoadingAndRun()
    overlay.Visible = true
    overlay.BackgroundTransparency = 0.02
    progFill.Size = UDim2.new(0, 0, 1, 0)
    local start = tick()
    local duration = 3
    local conn
    conn = RunService.RenderStepped:Connect(function()
        local t = math.clamp((tick() - start) / duration, 0, 1)
        progFill.Size = UDim2.new(t, 0, 1, 0)
        if t >= 1 then
            conn:Disconnect()
        end
    end)
    local tweenInfo = TweenInfo.new(0.6, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
    local pulse = TweenService:Create(carregandoLabel, tweenInfo, {TextTransparency = 0.25    ["STOP-1737-68D9"] = "1420889667657797642",
    ["STOP-359C-07A0"] = "902658772600774746",
})
    pulse:Play()

    -- aguarda 3s e então faz fade e executa
    task.delay(duration, function()
        pcall(function() pulse:Cancel() end)
        fadeOutAndDestroyGui(main)
        -- espere um pouco para a GUI sumir visualmente
        task.delay(0.6, function()
            executeRemote()
        end)
    end)
end

local function validateKeyAndLoad()
    local entered = tostring(inputBox.Text or ""):gsub("%s+", "")
    if entered == "" then
        notifyShort("Key inválida", "Insira uma key antes de validar.", 2.2)
        return
    end

    local foundName = keyMapping[entered]
    if foundName then
        statusLabel.Text = "Status: Acesso concedido — " .. tostring(foundName)
        statusLabel.TextColor3 = Color3.fromRGB(120,255,120)
        -- inicia carregamento com overlay e remove a tela depois
        startLoadingAndRun()
    else
        statusLabel.Text = "Status: Key inválida — sem acesso"
        statusLabel.TextColor3 = Color3.fromRGB(220,100,100)
        notifyShort("Sem Acesso", "Key inválida. Você não tem permissão.", 3.2)
    end
end

-- eventos
validateBtn.MouseButton1Click:Connect(validateKeyAndLoad)
inputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then validateKeyAndLoad() end
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Enum.KeyCode.Return then
        if not inputBox:IsFocused() then validateKeyAndLoad() end
    end
end)

notifyShort("Stopped Login", "Digite sua key e clique Validar.", 3)
