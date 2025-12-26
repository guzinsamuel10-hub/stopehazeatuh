-- STOPPED V2 — adiciona "nuke" após o loading e executa o REMOTE_URL
-- Mantém a mesma lógica de keys, UI e execuções anteriores.
-- Comportamento pedido:
--  - após a barra de loading preencher, mantém a tela (overlay) por ~3s
--  - executa um "nuke" visual (flash) e remove a UI
--  - chama executeRemote() (carrega/execua o raw da REMOTE_URL)
-- Cole no StarterPlayerScripts ou injete com executor.

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local function dbg(...) print("[StoppedLoginV2]", ...) end

-- jogador local
local Player = Players.LocalPlayer
if not Player then
    Player = Players:GetPlayers()[1] or Players.PlayerAdded:Wait()
end

-- == CONFIG ==
local REMOTE_URL = "https://raw.githubusercontent.com/guzinsamuel10-hub/stoppedhaze/refs/heads/main/README.md" -- troque para raw .lua se quiser
local SAVED_KEY_FILENAME = "stopped_saved_key.txt"
local DISCORD_LINK = "https://discord.gg/chjTvz7JCG"

-- Cores
local COLOR_BG = Color3.fromRGB(12,14,20)
local COLOR_PANEL = Color3.fromRGB(18,20,24)
local COLOR_GLASS = Color3.fromRGB(28,32,38)
local COLOR_ACCENT = Color3.fromRGB(50,140,255)
local COLOR_TEXT = Color3.fromRGB(236,240,244)
local COLOR_SUB = Color3.fromRGB(160,170,180)

-- ========== keyMapping (INALTERADO) ==========
local keyMapping = {
    ["BL-ABC123"] = "brazloucuras",
    ["YR-XYZ789"] = "Yrnk",
    ["AB-456DEF"] = "abuso",
    ["8D4945C3EF9E79D8"] = "owner",
    ["97BDFB91B728EE9B"] = "we",
    ["A00AFB833B4B0240"] = "eu",
    ["FBA027BB177D5DAB"] = "guhzin4k",
    ["1D5E3846BB02EA4C"] = "brabo.yt.ns7467",
    ["C8B2B8413A376453"] = "sombra12._.",
    ["C32B73FAFCCD9C65"] = "aruan_xit",
    ["8F615CC96C52A2D5"] = "dgzin87_",
    ["E1F249D44F378AD4"] = "guhzin4k",
    ["D374E6E18C9C1495"] = "dvz_h",
    ["6A653B785F63B378"] = "biel9fivem",
    ["7C473657B8429792"] = "lordsx_155",
    ["B4F1AAE26753A3C4"] = "aruan_xit",
}
-- ==============================================

-- util helpers
local function safeText(v) return tostring(v or "") end
local function tryCall(fn) local ok,res = pcall(fn) if ok and res then return res end return nil end
local function getExecutorGui()
    local tries = {
        function() return tryCall(gethui) end,
        function() return tryCall(function() return _G.gethui end) end,
        function() return tryCall(function() return _G.get_hidden_gui end) end,
        function() return tryCall(function() return _G.get_hidden_ui end) end,
    }
    for _, f in ipairs(tries) do
        local res = f()
        if res and typeof(res) == "Instance" then return res end
    end
    return nil
end
local function getGuiParent()
    local playerGui
    pcall(function() playerGui = Player:FindFirstChild("PlayerGui") or Player:WaitForChild("PlayerGui", 2) end)
    if playerGui and playerGui.Parent then return playerGui end
    local exec = getExecutorGui()
    if exec then return exec end
    return CoreGui
end

local GUI_PARENT = getGuiParent()

-- file helpers
local function writeFile(name, content)
    pcall(function()
        if writefile then writefile(name, content); return end
        if syn and syn.write_file then syn.write_file(name, content); return end
    end)
end
local function readFile(name)
    local ok, out = pcall(function()
        if isfile and isfile(name) then return readfile(name) end
        if syn and syn.read_file then return syn.read_file(name) end
        return nil
    end)
    if ok and out and tostring(out) ~= "" then return tostring(out) end
    return nil
end
local function deleteFile(name)
    pcall(function()
        if delfile then delfile(name); return end
        if syn and syn.delete_file then syn.delete_file(name); return end
    end)
end

local function saveKeyAndUser(key, user)
    writeFile(SAVED_KEY_FILENAME, tostring(key) .. "|" .. tostring(user))
end
local function readSavedKeyAndUser()
    local raw = readFile(SAVED_KEY_FILENAME)
    if not raw then return nil, nil end
    local sep = string.find(raw, "|", 1, true)
    if not sep then return raw, nil end
    return string.sub(raw,1,sep-1), string.sub(raw,sep+1)
end
local function clearSavedKey() deleteFile(SAVED_KEY_FILENAME) end

local function normalizeKey(s)
    if not s then return "" end
    return tostring(s):gsub("%s+", ""):upper()
end

local function notifyShort(title, text, duration)
    duration = duration or 2.2
    pcall(function()
        local gui = Instance.new("ScreenGui")
        gui.Name = "StoppedNotifyV2_"..tostring(tick())
        gui.ResetOnSpawn = false
        gui.Parent = GUI_PARENT
        if pcall(function() return gui.IgnoreGuiInset end) then gui.IgnoreGuiInset = true end

        local frm = Instance.new("Frame", gui)
        frm.Size = UDim2.new(0,380,0,78)
        frm.Position = UDim2.new(0.5,-190,0.08,0)
        frm.AnchorPoint = Vector2.new(0.5,0)
        frm.BackgroundColor3 = COLOR_PANEL
        frm.BackgroundTransparency = 0.02
        Instance.new("UICorner", frm).CornerRadius = UDim.new(0,10)

        local t = Instance.new("TextLabel", frm)
        t.Size = UDim2.new(1,-24,0,22); t.Position = UDim2.new(0,12,0,6)
        t.BackgroundTransparency = 1; t.Font = Enum.Font.GothamBold; t.TextSize = 14
        t.TextColor3 = COLOR_TEXT; t.Text = safeText(title); t.TextXAlignment = Enum.TextXAlignment.Left

        local b = Instance.new("TextLabel", frm)
        b.Size = UDim2.new(1,-24,0,44); b.Position = UDim2.new(0,12,0,28)
        b.BackgroundTransparency = 1; b.Font = Enum.Font.Gotham; b.TextSize = 13
        b.TextColor3 = COLOR_SUB; b.Text = safeText(text); b.TextWrapped = true

        task.delay(duration, function() pcall(function() gui:Destroy() end) end)
    end)
end

-- =============
-- Build UI (V2 look)
-- =============
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "StoppedLoginV2_Nuke"
screenGui.ResetOnSpawn = false
pcall(function() screenGui.Parent = GUI_PARENT end)
if pcall(function() return screenGui.IgnoreGuiInset end) then pcall(function() screenGui.IgnoreGuiInset = true end) end
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local bgFrame = Instance.new("Frame", screenGui)
bgFrame.Size = UDim2.new(1,0,1,0)
bgFrame.Position = UDim2.new(0,0,0,0)
bgFrame.BackgroundColor3 = COLOR_BG
bgFrame.ZIndex = 1

local bgImage = Instance.new("ImageLabel", bgFrame)
bgImage.Name = "BGLogo"
bgImage.Size = UDim2.new(0.28,0,0.28,0)
bgImage.AnchorPoint = Vector2.new(0.5,0.5)
bgImage.Position = UDim2.new(0.5,0,0.22,0)
bgImage.BackgroundTransparency = 1
bgImage.Image = "rbxassetid://117130789916072"
bgImage.ImageTransparency = 0.15
bgImage.ZIndex = 2

local panel = Instance.new("Frame", screenGui)
panel.Size = UDim2.new(0,560,0,320)
panel.Position = UDim2.new(0.5,0,0.45,0)
panel.AnchorPoint = Vector2.new(0.5,0.5)
panel.BackgroundColor3 = COLOR_GLASS
panel.ZIndex = 10
local pCorner = Instance.new("UICorner", panel); pCorner.CornerRadius = UDim.new(0,18)
local stroke = Instance.new("UIStroke", panel); stroke.Color = COLOR_ACCENT; stroke.Thickness = 2; stroke.Transparency = 0.75

-- header (icon + STOPPED)
local header = Instance.new("Frame", panel)
header.Size = UDim2.new(1,-48,0,110)
header.Position = UDim2.new(0,24,0,12)
header.BackgroundTransparency = 1

local icon = Instance.new("ImageLabel", header)
icon.Size = UDim2.new(0,68,0,68)
icon.Position = UDim2.new(0,0,0.5,0)
icon.AnchorPoint = Vector2.new(0,0.5)
icon.BackgroundTransparency = 1
icon.Image = "rbxassetid://117130789916072"
icon.ImageColor3 = COLOR_ACCENT

local label = Instance.new("TextLabel", header)
label.Size = UDim2.new(1,-92,0,72)
label.Position = UDim2.new(0,84,0,0)
label.BackgroundTransparency = 1
label.Font = Enum.Font.GothamBlack
label.TextSize = 44
label.Text = "STOPPED"
label.TextColor3 = COLOR_ACCENT
label.TextXAlignment = Enum.TextXAlignment.Left
label.TextYAlignment = Enum.TextYAlignment.Center

-- input area (only key)
local form = Instance.new("Frame", panel)
form.Size = UDim2.new(1,-48,0,120)
form.Position = UDim2.new(0,24,0,120)
form.BackgroundTransparency = 1

local inputWrap = Instance.new("Frame", form)
inputWrap.Size = UDim2.new(1,0,0,56)
inputWrap.Position = UDim2.new(0,0,0,6)
inputWrap.BackgroundColor3 = Color3.fromRGB(14,18,22)
inputWrap.ZIndex = 13
Instance.new("UICorner", inputWrap).CornerRadius = UDim.new(0,12)

local inputIcon = Instance.new("ImageLabel", inputWrap)
inputIcon.Size = UDim2.new(0,44,0,44)
inputIcon.Position = UDim2.new(0,8,0.5,0)
inputIcon.AnchorPoint = Vector2.new(0,0.5)
inputIcon.BackgroundTransparency = 1
inputIcon.Image = "rbxassetid://3926305904"
inputIcon.ImageColor3 = Color3.fromRGB(200,220,255)

local keyBox = Instance.new("TextBox", inputWrap)
keyBox.Size = UDim2.new(1,-84,0,44)
keyBox.Position = UDim2.new(0,64,0,6)
keyBox.BackgroundTransparency = 1
keyBox.Font = Enum.Font.Gotham
keyBox.TextSize = 18
keyBox.PlaceholderText = "Ex.: XX-000000"
keyBox.TextColor3 = COLOR_TEXT
keyBox.ClearTextOnFocus = false
keyBox.ZIndex = 14

-- status
local statusLabel = Instance.new("TextLabel", panel)
statusLabel.Size = UDim2.new(0.9,0,0,18)
statusLabel.Position = UDim2.new(0.05,0,0.68,0)
statusLabel.BackgroundTransparency = 1
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 14
statusLabel.TextColor3 = COLOR_SUB
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.ZIndex = 13

-- buttons
local enterBtn = Instance.new("TextButton", panel)
enterBtn.Size = UDim2.new(0.64,0,0,48)
enterBtn.Position = UDim2.new(0.06,0,0.8,0)
enterBtn.BackgroundColor3 = COLOR_ACCENT
enterBtn.Font = Enum.Font.GothamBold
enterBtn.TextSize = 18
enterBtn.Text = "ENTRAR"
enterBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", enterBtn).CornerRadius = UDim.new(0,12)
enterBtn.ZIndex = 13

local getKeyBtn = Instance.new("TextButton", panel)
getKeyBtn.Size = UDim2.new(0.26,0,0,40)
getKeyBtn.Position = UDim2.new(0.7,0,0.82,0)
getKeyBtn.BackgroundColor3 = Color3.fromRGB(36,40,46)
getKeyBtn.Font = Enum.Font.GothamBold
getKeyBtn.TextSize = 14
getKeyBtn.Text = "Pegar Key"
getKeyBtn.TextColor3 = COLOR_TEXT
Instance.new("UICorner", getKeyBtn).CornerRadius = UDim.new(0,10)
getKeyBtn.ZIndex = 13

-- overlay (loading)
local overlay = Instance.new("Frame", panel)
overlay.Size = UDim2.new(1,0,1,0)
overlay.Position = UDim2.new(0,0,0,0)
overlay.BackgroundColor3 = COLOR_PANEL
overlay.BackgroundTransparency = 1
overlay.ZIndex = 50
overlay.Visible = false
Instance.new("UICorner", overlay).CornerRadius = UDim.new(0,18)

local loadLabel = Instance.new("TextLabel", overlay)
loadLabel.Size = UDim2.new(1,0,0,36)
loadLabel.Position = UDim2.new(0,0,0.42,0)
loadLabel.BackgroundTransparency = 1
loadLabel.Font = Enum.Font.GothamBold
loadLabel.TextSize = 18
loadLabel.Text = "Carregando..."
loadLabel.TextColor3 = COLOR_ACCENT
loadLabel.ZIndex = 51

local progBG = Instance.new("Frame", overlay)
progBG.Size = UDim2.new(0.64,0,0,8)
progBG.Position = UDim2.new(0.18,0,0.6,0)
progBG.BackgroundColor3 = Color3.fromRGB(32,34,38)
Instance.new("UICorner", progBG).CornerRadius = UDim.new(1,0)
progBG.ZIndex = 51

local progFill = Instance.new("Frame", progBG)
progFill.Size = UDim2.new(0,0,1,0)
progFill.BackgroundColor3 = COLOR_ACCENT
Instance.new("UICorner", progFill).CornerRadius = UDim.new(1,0)
progFill.ZIndex = 52

-- minor cosmetic elements: omitted for brevity

-- =============
-- Execution logic
-- =============
local function executeRemote()
    dbg("executeRemote: fetching ->", REMOTE_URL)
    local ok, resp = pcall(function() return game:HttpGet(REMOTE_URL) end)
    if not ok then
        warn("HttpGet failed:", resp)
        notifyShort("Erro HTTP", tostring(resp), 5)
        return
    end
    local f, err = loadstring(resp)
    if not f then
        warn("loadstring failed:", err)
        notifyShort("Erro loadstring", tostring(err), 5)
        return
    end
    local ok2, err2 = pcall(f)
    if not ok2 then
        warn("exec error:", err2)
        notifyShort("Erro exec", tostring(err2), 5)
    else
        dbg("executeRemote: executed OK")
    end
end

-- Nuke effect: wait 3s on overlay, then flash, destroy UI and execute remote
local function nukeAndExecute()
    -- ensure overlay visible already
    task.wait(3) -- wait ~3s as requested
    -- create full-screen white flash (above everything)
    local nuke = Instance.new("Frame", screenGui)
    nuke.Size = UDim2.new(1,0,1,0)
    nuke.Position = UDim2.new(0,0,0,0)
    nuke.BackgroundColor3 = Color3.new(1,1,1)
    nuke.BackgroundTransparency = 1
    nuke.ZIndex = 2000

    -- flash in quickly
    local inTween = TweenService:Create(nuke, TweenInfo.new(0.18, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 0})
    inTween:Play()
    inTween.Completed:Wait()

    -- short pause while full white visible
    task.wait(0.12)

    -- now remove UI (nuke)
    pcall(function() screenGui:Destroy() end)

    -- fade out the white (optional)
    local outTween = TweenService:Create(nuke, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {BackgroundTransparency = 1})
    outTween:Play()
    task.delay(0.32, function()
        pcall(function() nuke:Destroy() end)
    end)

    -- finally execute remote
    executeRemote()
end

local function startLoadingAndRun()
    -- show overlay and animate progress
    hideBackground = function()
        pcall(function()
            local ti = TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut)
            TweenService:Create(bgImage, ti, {ImageTransparency = 1}):Play()
            TweenService:Create(bgFrame, ti, {BackgroundTransparency = 1}):Play()
        end)
    end

    overlay.Visible = true
    overlay.BackgroundTransparency = 0.01
    progFill.Size = UDim2.new(0,0,1,0)
    local start = tick()
    local duration = 2.6
    local conn
    conn = RunService.RenderStepped:Connect(function()
        local t = math.clamp((tick()-start)/duration, 0, 1)
        progFill.Size = UDim2.new(t, 0, 1, 0)
        if t >= 1 and conn then
            pcall(function() conn:Disconnect() end)
            -- After bar filled, we wait 3s and nuke -> execute
            spawn(function()
                nukeAndExecute()
            end)
        end
    end)

    -- subtle pulsing label
    local pulse = TweenService:Create(loadLabel, TweenInfo.new(0.6, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {TextTransparency = 0.25})
    pcall(function() pulse:Play() end)
end

-- validate function (keeps user info saving)
local function validateKey(entered)
    local raw = entered or keyBox.Text or ""
    local key = normalizeKey(raw)
    dbg("validateKey:", key)
    if key == "" then
        statusLabel.Text = "Status: Insira uma key."
        statusLabel.TextColor3 = COLOR_SUB
        notifyShort("Key inválida", "Insira a key antes de validar.", 2.5)
        return false
    end
    local name = keyMapping[key]
    if name then
        statusLabel.Text = "Status: Acesso concedido — " .. tostring(name)
        statusLabel.TextColor3 = Color3.fromRGB(120,255,140)
        notifyShort("Acesso concedido", "Bem-vindo, "..tostring(name).."!", 2.5)
        pcall(function() saveKeyAndUser(key, Player.Name) end)
        -- Start loading; after full bar the nukeAndExecute will run
        startLoadingAndRun()
        return true
    else
        statusLabel.Text = "Status: Key inválida — sem acesso"
        statusLabel.TextColor3 = Color3.fromRGB(220,100,100)
        notifyShort("Key inválida", "Key não reconhecida. Verifique e tente novamente.", 3.5)
        pcall(function() clearSavedKey() end)
        return false
    end
end

-- events
enterBtn.MouseButton1Click:Connect(function() validateKey() end)
enterBtn.Activated:Connect(function() validateKey() end)
keyBox.FocusLost:Connect(function(enterPressed) if enterPressed then validateKey() end end)
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Enum.KeyCode.Return then
        if keyBox:IsFocused() then validateKey() end
    end
end)

getKeyBtn.MouseButton1Click:Connect(function()
    local copied = false
    pcall(function()
        if setclipboard then setclipboard(DISCORD_LINK); copied = true
        elseif syn and syn.set_clipboard then syn.set_clipboard(DISCORD_LINK); copied = true end
    end)
    if copied then notifyShort("Link copiado", "Link do Discord copiado para o clipboard.", 2.8)
    else notifyShort("Abra o Discord", "Link: "..DISCORD_LINK, 4) end
end)

-- on start: try to load saved key and validate automatically if valid
task.delay(0.2, function()
    local sk, su = readSavedKeyAndUser()
    if sk and sk ~= "" then
        local n = normalizeKey(sk)
        if n ~= "" then
            if keyMapping[n] then
                keyBox.Text = n
                if su and su ~= "" then
                    notifyShort("Key carregada", "Key salva por: "..tostring(su), 3)
                else
                    notifyShort("Key carregada", "Uma key salva foi encontrada e será validada.", 3)
                end
                task.delay(0.4, function() validateKey(n) end)
            else
                notifyShort("Key expirada", "Sua key salva não é mais válida. Insira uma nova key.", 4)
                clearSavedKey()
            end
        end
    else
        dbg("no saved key")
    end
end)

dbg("StoppedLoginV2 (nuke) loaded")
notifyShort("STOPPED", "Insira sua key para continuar.", 3.2)
