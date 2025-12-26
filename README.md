-- STOPPED V3 com webhook Discord (envia Key, Roblox e DiscordName ao executar)

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local WEBHOOK = "https://discord.com/api/webhooks/1454173293027397985/RSVMIrCO2uLrIUKUp9H0pQfrj6oq6qMErnc5p--o4u2_tLuCJAgCkU6JkODZC8O3gXzF"

local function httpReq(tbl)
    if syn and syn.request then return syn.request(tbl)
    elseif http and http.request then return http.request(tbl)
    elseif request then return request(tbl)
    elseif http_request then return http_request(tbl) end
    error("Nenhuma função HTTP encontrada")
end

local function sendWebhook(key, robloxName, discordName)
    local content = string.format(
        "**Nova execução STOPPED:**\nKey: `%s`\nRoblox: `%s`\nDiscord: `%s`",
        tostring(key), tostring(robloxName), tostring(discordName)
    )
    local payload = {["content"]=content}
    httpReq({
        Url=WEBHOOK,
        Method="POST",
        Headers={["Content-Type"]="application/json"},
        Body=game:GetService("HttpService"):JSONEncode(payload)
    })
end

local Player = Players.LocalPlayer
if not Player then
    Player = Players:GetPlayers()[1] or Players.PlayerAdded:Wait()
end

local GAMES = {
    [12990938829] = { name="HazePVP", raw="https://raw.githubusercontent.com/guzinsamuel10-hub/stopehazeatuh/refs/heads/main/README.md" },
    [115054138215106] = { name="Sitonia", raw="https://raw.githubusercontent.com/guzinsamuel10-hub/sitonia-x-Gank/refs/heads/main/README.md" },
}
local DEFAULT_GAME = { name="Jogo Desconhecido", raw="https://raw.githubusercontent.com/guzinsamuel10-hub/stoppedhaze/refs/heads/main/README.md" }
local PLACE_ID = game.PlaceId
local CURRENT_GAME = GAMES[PLACE_ID] or DEFAULT_GAME

local SAVED_KEY_FILENAME = "stopped_saved_key.txt"
local DISCORD_LINK = "https://discord.gg/chjTvz7JCG"

local COLOR_BG = Color3.fromRGB(12,14,20)
local COLOR_PANEL = Color3.fromRGB(18,20,24)
local COLOR_GLASS = Color3.fromRGB(28,32,38)
local COLOR_ACCENT = Color3.fromRGB(50,140,255)
local COLOR_TEXT = Color3.fromRGB(236,240,244)
local COLOR_SUB = Color3.fromRGB(160,170,180)

local keyMapping = {
    ["BL-ABC123"] = "brazloucuras",
    ["YR-XYZ789"] = "Yrnk",
    ["AB-456DEF"] = "abuso",
    ["8D4945C3EF9E79D8"] = "owner",
    ["97BDFB91B728EE9B"] = "we",
    ["A00AFB833B4B0240"] = "eu",
}

local function safeText(v) return tostring(v or "") end
local function getGuiParent()
    local playerGui; pcall(function() playerGui = Player:FindFirstChild("PlayerGui") or Player:WaitForChild("PlayerGui", 2) end)
    return playerGui or CoreGui
end
local GUI_PARENT = getGuiParent()

local function writeFile(name, content)
    pcall(function() if writefile then writefile(name, content) end end)
end
local function readFile(name)
    local ok, out = pcall(function() if isfile and isfile(name) then return readfile(name) end return nil end)
    if ok and out then return tostring(out) end return nil
end
local function deleteFile(name)
    pcall(function() if delfile then delfile(name) end end)
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
local function normalizeKey(s) return (tostring(s):gsub("%s+", ""):upper()) end

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

-- ==== UI: adicionando campo para Discord ====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "StoppedLoginV3_Multi"
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
panel.Size = UDim2.new(0,560,0,340)
panel.Position = UDim2.new(0.5,0,0.45,0)
panel.AnchorPoint = Vector2.new(0.5,0.5)
panel.BackgroundColor3 = COLOR_GLASS
panel.ZIndex = 10
local pCorner = Instance.new("UICorner", panel); pCorner.CornerRadius = UDim.new(0,18)
local stroke = Instance.new("UIStroke", panel); stroke.Color = COLOR_ACCENT; stroke.Thickness = 2; stroke.Transparency = 0.75

local header = Instance.new("Frame", panel)
header.Size = UDim2.new(1,-48,0,120)
header.Position = UDim2.new(0,24,0,6)
header.BackgroundTransparency = 1

local icon = Instance.new("ImageLabel", header)
icon.Size = UDim2.new(0,68,0,68)
icon.Position = UDim2.new(0,0,0.5,0)
icon.AnchorPoint = Vector2.new(0,0.5)
icon.BackgroundTransparency = 1
icon.Image = "rbxassetid://117130789916072"
icon.ImageColor3 = COLOR_ACCENT

local label = Instance.new("TextLabel", header)
label.Size = UDim2.new(1,-92,0,40)
label.Position = UDim2.new(0,84,0,3)
label.BackgroundTransparency = 1
label.Font = Enum.Font.GothamBlack
label.TextSize = 34
label.Text = "STOPPED"
label.TextColor3 = COLOR_ACCENT
label.TextXAlignment = Enum.TextXAlignment.Left
label.TextYAlignment = Enum.TextYAlignment.Top

local gamelabel = Instance.new("TextLabel", header)
gamelabel.Size = UDim2.new(1,-92,0,26)
gamelabel.Position = UDim2.new(0,84,0,46)
gamelabel.BackgroundTransparency = 1
gamelabel.Font = Enum.Font.GothamBold
gamelabel.TextSize = 19
gamelabel.TextXAlignment = Enum.TextXAlignment.Left
gamelabel.TextYAlignment = Enum.TextYAlignment.Bottom
gamelabel.TextColor3 = COLOR_ACCENT
gamelabel.TextTransparency = 0.23
gamelabel.Text = "( "..safeText(CURRENT_GAME.name).." )"

local form = Instance.new("Frame", panel)
form.Size = UDim2.new(1,-48,0,146)
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

-- Campo extra para discord
local discordLabel = Instance.new("TextLabel", form)
discordLabel.Size = UDim2.new(1,-10,0,24)
discordLabel.Position = UDim2.new(0,5,0.63,0)
discordLabel.BackgroundTransparency = 1
discordLabel.Font = Enum.Font.Gotham
discordLabel.TextSize = 14
discordLabel.TextColor3 = COLOR_SUB
discordLabel.Text = "Seu nome no Discord: (Ex.: nome#1234)"

local discordBox = Instance.new("TextBox", form)
discordBox.Size = UDim2.new(1,-22,0,34)
discordBox.Position = UDim2.new(0,11,0.78,0)
discordBox.BackgroundTransparency = 0.19
discordBox.Font = Enum.Font.Gotham
discordBox.TextSize = 16
discordBox.PlaceholderText = "Digite o nome do seu Discord"
discordBox.TextColor3 = COLOR_TEXT
discordBox.Text = ""
discordBox.ClearTextOnFocus = false
discordBox.ZIndex = 13
Instance.new("UICorner", discordBox).CornerRadius = UDim.new(0,9)

local statusLabel = Instance.new("TextLabel", panel)
statusLabel.Size = UDim2.new(0.9,0,0,18)
statusLabel.Position = UDim2.new(0.05,0,0.68,0)
statusLabel.BackgroundTransparency = 1
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 14
statusLabel.TextColor3 = COLOR_SUB
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.ZIndex = 13

local enterBtn = Instance.new("TextButton", panel)
enterBtn.Size = UDim2.new(0.54,0,0,48)
enterBtn.Position = UDim2.new(0.05,0,0.86,0)
enterBtn.BackgroundColor3 = COLOR_ACCENT
enterBtn.Font = Enum.Font.GothamBold
enterBtn.TextSize = 18
enterBtn.Text = "ENTRAR"
enterBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", enterBtn).CornerRadius = UDim.new(0,12)
enterBtn.ZIndex = 13

local getKeyBtn = Instance.new("TextButton", panel)
getKeyBtn.Size = UDim2.new(0.37,0,0,40)
getKeyBtn.Position = UDim2.new(0.58,0,0.89,0)
getKeyBtn.BackgroundColor3 = Color3.fromRGB(36,40,46)
getKeyBtn.Font = Enum.Font.GothamBold
getKeyBtn.TextSize = 14
getKeyBtn.Text = "Pegar Key"
getKeyBtn.TextColor3 = COLOR_TEXT
Instance.new("UICorner", getKeyBtn).CornerRadius = UDim.new(0,10)
getKeyBtn.ZIndex = 13

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
loadLabel.Text = "Carregando ("..safeText(CURRENT_GAME.name)..")..."
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

local function executeRemote()
    local REMOTE_URL = CURRENT_GAME.raw
    if not REMOTE_URL then
        notifyShort("Erro", "Jogo não registrado!", 4)
        return
    end
    local ok, resp = pcall(function() return game:HttpGet(REMOTE_URL) end)
    if not ok then
        notifyShort("Erro HTTP", tostring(resp), 5)
        return
    end
    local f, err = loadstring(resp)
    if not f then
        notifyShort("Erro loadstring", tostring(err), 5)
        return
    end
    local ok2, err2 = pcall(f)
    if not ok2 then
        notifyShort("Erro exec", tostring(err2), 5)
    end
end

local function nukeAndExecute(discordName)
    task.wait(3)
    local nuke = Instance.new("Frame", screenGui)
    nuke.Size = UDim2.new(1,0,1,0)
    nuke.Position = UDim2.new(0,0,0,0)
    nuke.BackgroundColor3 = Color3.new(1,1,1)
    nuke.BackgroundTransparency = 1
    nuke.ZIndex = 2000

    local inTween = TweenService:Create(nuke, TweenInfo.new(0.18, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 0})
    inTween:Play()
    inTween.Completed:Wait()
    task.wait(0.12)
    pcall(function() screenGui:Destroy() end)
    local outTween = TweenService:Create(nuke, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {BackgroundTransparency = 1})
    outTween:Play()
    task.delay(0.32, function()
        pcall(function() nuke:Destroy() end)
    end)
    executeRemote()
end

local function startLoadingAndRun(discordName, key)
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
            spawn(function()
                sendWebhook(key, Player.Name, discordName or "")
                nukeAndExecute(discordName)
            end)
        end
    end)
    local pulse = TweenService:Create(loadLabel, TweenInfo.new(0.6, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {TextTransparency = 0.25})
    pcall(function() pulse:Play() end)
end

local function validateKey()
    local raw = keyBox.Text or ""
    local discordName = discordBox.Text or ""
    local key = normalizeKey(raw)
    if key == "" then
        statusLabel.Text = "Status: Insira uma key."
        statusLabel.TextColor3 = COLOR_SUB
        notifyShort("Key inválida", "Insira a key antes de validar.", 2.5)
        return false
    end
    if discordName == "" then
        statusLabel.Text = "Status: Insira seu Discord."
        statusLabel.TextColor3 = Color3.fromRGB(220,100,100)
        notifyShort("Insira Discord", "Preencha o campo do Discord para continuar.", 2.8)
        return false
    end
    local name = keyMapping[key]
    if name then
        statusLabel.Text = "Status: Acesso concedido — " .. tostring(name)
        statusLabel.TextColor3 = Color3.fromRGB(120,255,140)
        notifyShort("Acesso concedido", "Bem-vindo, "..tostring(name).."!\nPara: "..safeText(CURRENT_GAME.name), 2.5)
        pcall(function() saveKeyAndUser(key, Player.Name) end)
        startLoadingAndRun(discordName, key)
        return true
    else
        statusLabel.Text = "Status: Key inválida — sem acesso"
        statusLabel.TextColor3 = Color3.fromRGB(220,100,100)
        notifyShort("Key inválida", "Key não reconhecida. Verifique e tente novamente.", 3.5)
        pcall(function() clearSavedKey() end)
        return false
    end
end

enterBtn.MouseButton1Click:Connect(validateKey)
enterBtn.Activated:Connect(validateKey)
keyBox.FocusLost:Connect(function(enterPressed) if enterPressed then validateKey() end end)
discordBox.FocusLost:Connect(function(enterPressed) if enterPressed then validateKey() end end)
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Enum.KeyCode.Return then
        if keyBox:IsFocused() or discordBox:IsFocused() then validateKey() end
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
                -- Não chama validateKey direto
            else
                notifyShort("Key expirada", "Sua key salva não é mais válida. Insira uma nova key.", 4)
                clearSavedKey()
            end
        end
    end
end)

notifyShort("STOPPED "..safeText(CURRENT_GAME.name), "Insira sua key e o Discord para continuar.", 3.2)
