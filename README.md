local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

--==================================================
-- CONFIGURAÇÃO
--==================================================

local ACCENT = Color3.fromRGB(255, 140, 0)          -- Laranja
local ACCENT_DARK = Color3.fromRGB(200, 100, 0)     -- Laranja escuro
local COPIED_COLOR = Color3.fromRGB(255, 190, 60)   -- Dourado (estado copiado)

local SCRIPT_LINK = "https://discord.gg/SE3Fg6nfWD"

local FRAME_WIDTH = 380
local FRAME_HEIGHT = 400

--==================================================
-- FUNÇÃO DE CÓPIA COM FALLBACK
--==================================================

local function copiarTexto(texto)
    -- 1) setclipboard (padrão da maioria dos executores)
    if type(setclipboard) == "function" then
        local ok = pcall(setclipboard, texto)
        if ok then return true, "setclipboard" end
    end

    -- 2) toclipboard (alguns executores usam esse nome)
    if type(toclipboard) == "function" then
        local ok = pcall(toclipboard, texto)
        if ok then return true, "toclipboard" end
    end

    -- 3) Clipboard.set (Synapse/outros)
    if type(Clipboard) == "table" and type(Clipboard.set) == "function" then
        local ok = pcall(Clipboard.set, texto)
        if ok then return true, "Clipboard.set" end
    end

    -- 4) Fallback nativo do Roblox: TextBox + Selection
    local ok, err = pcall(function()
        local holder = Instance.new("TextBox")
        holder.Size = UDim2.fromOffset(1, 1)
        holder.BackgroundTransparency = 1
        holder.TextTransparency = 1
        holder.Text = texto
        holder.ClearTextOnFocus = false
        holder.Parent = player:WaitForChild("PlayerGui")

        holder:CaptureFocus()
        task.wait(0.05)
        holder:ReleaseFocus()

        local sel = holder.SelectionStart
        local selEnd = holder.SelectionStart
        holder.SelectionStart = 1
        holder.SelectionEnd = #texto
        task.wait(0.05)

        holder:Destroy()
    end)

    if ok then return true, "nativo" end
    return false, tostring(err or "método desconhecido")
end

--==================================================
-- INTERFACE NA TELA
--==================================================

local gui = Instance.new("ScreenGui")
gui.Name = "LebronHub"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = player:WaitForChild("PlayerGui")

--==================================================
-- CARTÃO PRINCIPAL
--==================================================

local frame = Instance.new("Frame")
frame.Name = "MainCard"
frame.Size = UDim2.fromOffset(FRAME_WIDTH, FRAME_HEIGHT)
frame.Position = UDim2.new(0.5, 0, 0.5, 40)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
frame.BorderSizePixel = 0
frame.ClipsDescendants = true
frame.Active = true
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 16)
corner.Parent = frame

--==================================================
-- GRADIENTE DE FUNDO
--==================================================

local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 20, 10)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(15, 12, 10)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(5, 5, 5))
})
gradient.Rotation = 135
gradient.Parent = frame

--==================================================
-- BORDA
--==================================================

local stroke = Instance.new("UIStroke")
stroke.Color = ACCENT
stroke.Transparency = 0.55
stroke.Thickness = 1.2
stroke.Parent = frame

--==================================================
-- LINHA DE DESTAQUE SUPERIOR
--==================================================

local topLine = Instance.new("Frame")
topLine.Size = UDim2.new(0.55, 0, 0, 3)
topLine.Position = UDim2.new(0.225, 0, 0, 0)
topLine.BackgroundColor3 = ACCENT
topLine.BorderSizePixel = 0
topLine.ZIndex = 2
topLine.Parent = frame

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(1, 0)
topCorner.Parent = topLine

--==================================================
-- BRILHO SUPERIOR SUAVE
--==================================================

local glow = Instance.new("Frame")
glow.Size = UDim2.fromOffset(220, 120)
glow.Position = UDim2.new(0.5, -110, 0, -75)
glow.BackgroundColor3 = ACCENT
glow.BackgroundTransparency = 0.94
glow.BorderSizePixel = 0
glow.ZIndex = 1
glow.Parent = frame

local glowCorner = Instance.new("UICorner")
glowCorner.CornerRadius = UDim.new(1, 0)
glowCorner.Parent = glow

--==================================================
-- ÍCONE DE ATUALIZAÇÃO
--==================================================

local icon = Instance.new("Frame")
icon.Size = UDim2.fromOffset(42, 42)
icon.Position = UDim2.new(0, 20, 0, 22)
icon.BackgroundColor3 = ACCENT
icon.BackgroundTransparency = 0.88
icon.BorderSizePixel = 0
icon.ZIndex = 3
icon.Parent = frame

local iconCorner = Instance.new("UICorner")
iconCorner.CornerRadius = UDim.new(0, 12)
iconCorner.Parent = icon

local iconStroke = Instance.new("UIStroke")
iconStroke.Color = ACCENT
iconStroke.Transparency = 0.55
iconStroke.Thickness = 1
iconStroke.Parent = icon

local iconText = Instance.new("TextLabel")
iconText.Size = UDim2.fromScale(1, 1)
iconText.BackgroundTransparency = 1
iconText.Text = "↑"
iconText.TextColor3 = ACCENT
iconText.TextSize = 25
iconText.Font = Enum.Font.GothamBold
iconText.ZIndex = 4
iconText.Parent = icon

--==================================================
-- TÍTULO
--==================================================

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -115, 0, 30)
title.Position = UDim2.new(0, 75, 0, 20)
title.BackgroundTransparency = 1
title.Text = "LEBRONHUB"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 21
title.TextXAlignment = Enum.TextXAlignment.Left
title.Font = Enum.Font.GothamBold
title.ZIndex = 3
title.Parent = frame

--==================================================
-- RÓTULO DE VERSÃO
--==================================================

local version = Instance.new("TextLabel")
version.Size = UDim2.new(1, -115, 0, 18)
version.Position = UDim2.new(0, 76, 0, 47)
version.BackgroundTransparency = 1
version.Text = "NOVA VERSÃO DISPONÍVEL"
version.TextColor3 = ACCENT
version.TextSize = 10
version.TextXAlignment = Enum.TextXAlignment.Left
version.Font = Enum.Font.GothamBold
version.ZIndex = 3
version.Parent = frame

--==================================================
-- BOTÃO FECHAR
--==================================================

local close = Instance.new("TextButton")
close.Size = UDim2.fromOffset(32, 32)
close.Position = UDim2.new(1, -42, 0, 14)
close.BackgroundTransparency = 1
close.Text = "×"
close.TextColor3 = Color3.fromRGB(130, 130, 140)
close.TextSize = 24
close.Font = Enum.Font.Gotham
close.AutoButtonColor = false
close.Active = true
close.ZIndex = 5
close.Parent = frame

close.MouseEnter:Connect(function()
    TweenService:Create(close, TweenInfo.new(0.15), {
        TextColor3 = ACCENT
    }):Play()
end)

close.MouseLeave:Connect(function()
    TweenService:Create(close, TweenInfo.new(0.15), {
        TextColor3 = Color3.fromRGB(130, 130, 140)
    }):Play()
end)

--==================================================
-- DESCRIÇÃO
--==================================================

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, -40, 0, 25)
subtitle.Position = UDim2.new(0, 20, 0, 78)
subtitle.BackgroundTransparency = 1
subtitle.Text = "Um novo script ou loader do LeBronHub está disponível."
subtitle.TextColor3 = Color3.fromRGB(235, 235, 240)
subtitle.TextSize = 14
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Font = Enum.Font.GothamMedium
subtitle.ZIndex = 3
subtitle.Parent = frame

local description = Instance.new("TextLabel")
description.Size = UDim2.new(1, -40, 0, 20)
description.Position = UDim2.new(0, 20, 0, 101)
description.BackgroundTransparency = 1
description.Text = "Copie o link abaixo e abra no seu navegador."
description.TextColor3 = Color3.fromRGB(135, 135, 145)
description.TextSize = 11
description.TextXAlignment = Enum.TextXAlignment.Left
description.Font = Enum.Font.Gotham
description.ZIndex = 3
description.Parent = frame

--==================================================
-- BOTÃO DE COPIAR
--==================================================

local button = Instance.new("TextButton")
button.Name = "CopyButton"
button.Size = UDim2.new(1, -40, 0, 44)
button.Position = UDim2.new(0, 20, 0, 130)
button.BackgroundColor3 = ACCENT
button.BorderSizePixel = 0
button.AutoButtonColor = false
button.Text = ""
button.Active = true
button.ZIndex = 3
button.Parent = frame

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 10)
buttonCorner.Parent = button

local buttonGradient = Instance.new("UIGradient")
buttonGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, ACCENT),
    ColorSequenceKeypoint.new(1, ACCENT_DARK)
})
buttonGradient.Rotation = 90
buttonGradient.Parent = button

local buttonStroke = Instance.new("UIStroke")
buttonStroke.Color = Color3.fromRGB(255, 255, 255)
buttonStroke.Transparency = 0.82
buttonStroke.Thickness = 1
buttonStroke.Parent = button

local buttonText = Instance.new("TextLabel")
buttonText.Size = UDim2.fromScale(1, 1)
buttonText.BackgroundTransparency = 1
buttonText.Text = "COPIAR LINK DO SCRIPT"
buttonText.TextColor3 = Color3.fromRGB(255, 255, 255)
buttonText.TextSize = 13
buttonText.TextXAlignment = Enum.TextXAlignment.Center
buttonText.TextYAlignment = Enum.TextYAlignment.Center
buttonText.Font = Enum.Font.GothamBold
buttonText.ZIndex = 4
buttonText.Parent = button

--==================================================
-- TÍTULO DO CHANGELOG
--==================================================

local changelogTitle = Instance.new("TextLabel")
changelogTitle.Name = "ChangelogTitle"
changelogTitle.Size = UDim2.new(1, -40, 0, 18)
changelogTitle.Position = UDim2.new(0, 20, 0, 188)
changelogTitle.BackgroundTransparency = 1
changelogTitle.Text = "REGISTRO DE ATUALIZAÇÕES"
changelogTitle.TextColor3 = ACCENT
changelogTitle.TextSize = 11
changelogTitle.TextXAlignment = Enum.TextXAlignment.Left
changelogTitle.Font = Enum.Font.GothamBold
changelogTitle.ZIndex = 3
changelogTitle.Parent = frame

--==================================================
-- PAINEL DO CHANGELOG
--==================================================

local changelog = Instance.new("ScrollingFrame")
changelog.Name = "Changelog"
changelog.Size = UDim2.new(1, -40, 0, 155)
changelog.Position = UDim2.new(0, 20, 0, 211)
changelog.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
changelog.BackgroundTransparency = 0.15
changelog.BorderSizePixel = 0
changelog.ScrollBarThickness = 3
changelog.ScrollBarImageColor3 = ACCENT
changelog.CanvasSize = UDim2.new(0, 0, 0, 0)
changelog.AutomaticCanvasSize = Enum.AutomaticSize.Y
changelog.ScrollingDirection = Enum.ScrollingDirection.Y
changelog.Active = true
changelog.ZIndex = 3
changelog.Parent = frame

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 8)
panelCorner.Parent = changelog

local panelStroke = Instance.new("UIStroke")
panelStroke.Color = ACCENT
panelStroke.Transparency = 0.82
panelStroke.Thickness = 1
panelStroke.Parent = changelog

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 10)
padding.PaddingBottom = UDim.new(0, 10)
padding.PaddingLeft = UDim.new(0, 10)
padding.PaddingRight = UDim.new(0, 8)
padding.Parent = changelog

local list = Instance.new("UIListLayout")
list.Padding = UDim.new(0, 7)
list.SortOrder = Enum.SortOrder.LayoutOrder
list.Parent = changelog

--==================================================
-- ITENS DO CHANGELOG
--==================================================

local changes = {
    "Corrigido alguns bugs",
    "Loader alterado",
}

for i, change in ipairs(changes) do
    local row = Instance.new("TextLabel")
    row.Name = "Change_" .. i
    row.Size = UDim2.new(1, -2, 0, 20)
    row.BackgroundTransparency = 1
    row.Text = "+  " .. change
    row.TextColor3 = Color3.fromRGB(205, 207, 198)
    row.TextSize = 11
    row.Font = Enum.Font.Gotham
    row.TextXAlignment = Enum.TextXAlignment.Left
    row.TextYAlignment = Enum.TextYAlignment.Center
    row.TextWrapped = true
    row.TextTruncate = Enum.TextTruncate.AtEnd
    row.LayoutOrder = i
    row.ZIndex = 4
    row.Parent = changelog
end

--==================================================
-- INFORMAÇÃO INFERIOR
--==================================================

local info = Instance.new("TextLabel")
info.Size = UDim2.new(1, -40, 0, 20)
info.Position = UDim2.new(0, 20, 1, -25)
info.BackgroundTransparency = 1
info.Text = "LeBronHub — Última versão disponível."
info.TextColor3 = Color3.fromRGB(135, 135, 145)
info.TextSize = 10
info.Font = Enum.Font.Gotham
info.TextXAlignment = Enum.TextXAlignment.Center
info.TextTruncate = Enum.TextTruncate.AtEnd
info.ZIndex = 3
info.Parent = frame

--==================================================
-- EFEITO HOVER DO BOTÃO
--==================================================

local originalSize = button.Size
local originalPosition = button.Position
local isCopied = false
local copyToken = 0

button.MouseEnter:Connect(function()
    if isCopied or not gui.Parent then return end

    TweenService:Create(button, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundColor3 = ACCENT:Lerp(Color3.fromRGB(255, 255, 255), 0.10),
        Size = UDim2.new(1, -36, 0, 46),
        Position = UDim2.new(0, 18, 0, 129)
    }):Play()

    TweenService:Create(buttonStroke, TweenInfo.new(0.18), {
        Transparency = 0.55
    }):Play()
end)

button.MouseLeave:Connect(function()
    if isCopied or not gui.Parent then return end

    TweenService:Create(button, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundColor3 = ACCENT,
        Size = originalSize,
        Position = originalPosition
    }):Play()

    TweenService:Create(buttonStroke, TweenInfo.new(0.18), {
        Transparency = 0.82
    }):Play()
end)

--==================================================
-- AÇÃO DE COPIAR
--==================================================

button.MouseButton1Click:Connect(function()
    if isCopied or not gui.Parent then return end

    local sucesso, metodo = copiarTexto(SCRIPT_LINK)

    if not sucesso then
        warn("Não foi possível copiar o link do script (método: " .. tostring(metodo) .. ")")
        buttonText.Text = "FALHA AO COPIAR"
        buttonText.TextColor3 = Color3.fromRGB(255, 230, 230)

        task.delay(1.5, function()
            if gui and gui.Parent then
                buttonText.Text = "COPIAR LINK DO SCRIPT"
                buttonText.TextColor3 = Color3.fromRGB(255, 255, 255)
            end
        end)
        return
    end

    isCopied = true
    copyToken += 1
    local currentToken = copyToken

    TweenService:Create(button, TweenInfo.new(0.08), {
        Size = UDim2.new(1, -44, 0, 40)
    }):Play()

    task.wait(0.08)

    if not gui or not gui.Parent then return end

    buttonText.Text = "LINK COPIADO!"

    TweenService:Create(button, TweenInfo.new(0.2), {
        BackgroundColor3 = COPIED_COLOR,
        Size = originalSize,
        Position = originalPosition
    }):Play()

    TweenService:Create(buttonStroke, TweenInfo.new(0.2), {
        Transparency = 0.55
    }):Play()

    task.delay(1.5, function()
        if not gui or not gui.Parent or currentToken ~= copyToken then
            return
        end

        isCopied = false
        buttonText.Text = "COPIAR LINK DO SCRIPT"

        TweenService:Create(button, TweenInfo.new(0.2), {
            BackgroundColor3 = ACCENT
        }):Play()

        TweenService:Create(buttonStroke, TweenInfo.new(0.2), {
            Transparency = 0.82
        }):Play()
    end)
end)

--==================================================
-- ANIMAÇÃO DE FECHAMENTO
--==================================================

local isClosing = false

local function closeGui()
    if isClosing or not gui or not gui.Parent then
        return
    end

    isClosing = true
    copyToken += 1  -- invalida callbacks pendentes

    pcall(function()
        TweenService:Create(frame, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
            Size = UDim2.fromOffset(340, 340),
            Position = UDim2.new(0.5, 0, 0.5, 15)
        }):Play()

        TweenService:Create(stroke, TweenInfo.new(0.2), {
            Transparency = 1
        }):Play()
    end)

    task.wait(0.25)

    if gui then
        gui:Destroy()
    end
end

close.MouseButton1Click:Connect(closeGui)

--==================================================
-- ANIMAÇÃO DE ABERTURA
--==================================================

frame.Size = UDim2.fromOffset(330, 340)

TweenService:Create(
    frame,
    TweenInfo.new(
        0.45,
        Enum.EasingStyle.Back,
        Enum.EasingDirection.Out
    ),
    {
        Size = UDim2.fromOffset(FRAME_WIDTH, FRAME_HEIGHT),
        Position = UDim2.new(0.5, 0, 0.5, 0)
    }
):Play()

--==================================================
-- PULSO DA BORDA
--==================================================

task.spawn(function()
    while gui and gui.Parent and stroke do
        local ok1 = pcall(function()
            TweenService:Create(stroke, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
                Transparency = 0.75
            }):Play()
        end)
        if not ok1 then break end

        task.wait(1.2)

        if not gui or not gui.Parent or not stroke then break end

        local ok2 = pcall(function()
            TweenService:Create(stroke, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
                Transparency = 0.35
            }):Play()
        end)
        if not ok2 then break end

        task.wait(1.2)
    end
end)# Lebronhub
