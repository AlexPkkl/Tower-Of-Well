-- Coloque este script em StarterPlayerScripts

local player = game.Players.LocalPlayer
local repStorage = game:GetService("ReplicatedStorage")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Cria a GUI (se já existir, reaproveita para persistir na morte)
local screenGui = player.PlayerGui:FindFirstChild("YayCatGUI") or Instance.new("ScreenGui")
screenGui.Name = "YayCatGUI"
screenGui.ResetOnSpawn = false -- para não sumir quando morre
screenGui.Parent = player.PlayerGui

-- Criar Frame principal (ou reaproveitar)
local frame = screenGui:FindFirstChild("MainFrame") or Instance.new("Frame")
frame.Name = "MainFrame"
if not frame.Parent then
    frame.Size = UDim2.new(0, 300, 0, 150)
    frame.Position = UDim2.new(0.5, -150, 0.5, -75)
    frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    frame.BorderSizePixel = 0
    frame.Active = true
    frame.Draggable = true -- permite mover a interface
    frame.Parent = screenGui
end

-- Função para limpar filhos extras se reaproveitando o frame em re-spawn
frame:ClearAllChildren()

-- Título / Créditos
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 25)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
title.BorderSizePixel = 0
title.Text = "Yay Cat's Parkour GUI"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(180, 180, 180)
title.Parent = frame

-- Botão fechar
local btnClose = Instance.new("TextButton")
btnClose.Size = UDim2.new(0, 25, 0, 25)
btnClose.Position = UDim2.new(1, -30, 0, 0)
btnClose.BackgroundColor3 = Color3.fromRGB(170, 50, 50)
btnClose.Text = "X"
btnClose.Font = Enum.Font.SourceSansBold
btnClose.TextSize = 20
btnClose.TextColor3 = Color3.new(1, 1, 1)
btnClose.Parent = frame

btnClose.MouseButton1Click:Connect(function()
    screenGui.Enabled = false
end)

-- Botão minimizar
local btnMinimize = Instance.new("TextButton")
btnMinimize.Size = UDim2.new(0, 25, 0, 25)
btnMinimize.Position = UDim2.new(1, -60, 0, 0)
btnMinimize.BackgroundColor3 = Color3.fromRGB(50, 50, 170)
btnMinimize.Text = "-"
btnMinimize.Font = Enum.Font.SourceSansBold
btnMinimize.TextSize = 24
btnMinimize.TextColor3 = Color3.new(1,1,1)
btnMinimize.Parent = frame

local isMinimized = false
btnMinimize.MouseButton1Click:Connect(function()
    if isMinimized then
        -- Restaurar tamanho
        frame.Size = UDim2.new(0, 300, 0, 150)
        -- Mostrar todos filhos menos title e botões
        for _,child in ipairs(frame:GetChildren()) do
            if child ~= title and child ~= btnClose and child ~= btnMinimize then
                child.Visible = true
            end
        end
        isMinimized = false
    else
        -- Minimizar tamanho
        frame.Size = UDim2.new(0, 300, 0, 30)
        -- Esconder todos filhos menos controles
        for _,child in ipairs(frame:GetChildren()) do
            if child ~= title and child ~= btnClose and child ~= btnMinimize then
                child.Visible = false
            end
        end
        isMinimized = true
    end
end)

-- Função auxiliar para criar toggle buttons
local function createToggle(name, position, parent)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(0, 260, 0, 40)
    toggleFrame.Position = position
    toggleFrame.BackgroundColor3 = Color3.fromRGB(60,60,60)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = 20
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = toggleFrame

    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0, 60, 0, 30)
    toggle.Position = UDim2.new(0.75, 0, 0.15, 0)
    toggle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
    toggle.Text = "OFF"
    toggle.Font = Enum.Font.SourceSansBold
    toggle.TextSize = 18
    toggle.TextColor3 = Color3.new(0,0,0)
    toggle.Parent = toggleFrame

    local state = false
    toggle.MouseButton1Click:Connect(function()
        state = not state
        if state then
            toggle.Text = "ON"
            toggle.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        else
            toggle.Text = "OFF"
            toggle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
        end
        toggleFrame:SetAttribute("ToggleState", state)
        -- Evento para ser usado depois se necessário
        toggleFrame:FireAllClients()
    end)

    -- Função para acessar estado
    return toggleFrame, function() return state end, function(newState)
        state = newState
        if state then
            toggle.Text = "ON"
            toggle.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        else
            toggle.Text = "OFF"
            toggle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
        end
        toggleFrame:SetAttribute("ToggleState", state)
    end
end

-- Criar toggles
local finishToggleFrame, getFinishState, setFinishState = createToggle("Finish Parkour", UDim2.new(0, 20, 0, 35), frame)
local gearToggleFrame, getGearState, setGearState = createToggle("Get All Gears", UDim2.new(0, 20, 0, 80), frame)

-- Logica do Finish Parkour
local function teleportToFinish()
    local tower = workspace:FindFirstChild("tower")
    if not tower then return end

    local sections = tower:FindFirstChild("sections")
    if not sections then return end

    local finish = sections:FindFirstChild("finish")
    if not finish then return end

    local exit = finish:FindFirstChild("exit")
    if not exit then return end

    local particleBrick = exit:FindFirstChild("ParticleBrick")
    if not particleBrick then return end

    local character = player.Character
    if character then
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.CFrame = CFrame.new(particleBrick.Position + Vector3.new(0, 3, 0)) -- Isso fará o player "acima" do bloco
        end
    end
end

-- Logica do Get All Gears
local function getAllGears()
    local gearFolder = repStorage:FindFirstChild("Assets")
    if not gearFolder then return end
    gearFolder = gearFolder:FindFirstChild("Gear")
    if not gearFolder then return end

    for _, gearItem in ipairs(gearFolder:GetChildren()) do
        if gearItem:IsA("Tool") then
            -- Clona para garantir que o player recebe o item dele próprio
            local gearClone = gearItem:Clone()
            gearClone.Parent = player.Backpack
        end
    end
end

-- Timer para checar toggles em loop e aplicar ações automaticamente
RunService.RenderStepped:Connect(function()
    -- Finish Parkour toggle ativo ? Teleporta (uma vez por click é mais legal, mas se quiser pode ficar em loop)
    -- Para evitar múltiplos teleports, aqui estará apenas um flag.
    -- Vamos detectar clique no toggle para teleportar

    -- O código para "Finish Parkour" só teleporta ao clicar no botão,
    -- Alternativamente aqui não repete.

    -- "Get All Gears" faz o "dar" só ao alternar para ON (uma vez)
    -- então podemos adicionar evento nos toggles:

    -- Isto é apenas loop para expandir se quiser futuramente

end)

-- Conectar ação ao clique pelo toggle (em vez de loop)
finishToggleFrame.ChildAdded:Connect(function(child)
    -- ignora adicionados
end)

-- No toggle, vamos conectar MouseButton1Click dos botões para as ações
-- Já dentro da função createToggle tem toggle.MouseButton1Click

-- Para executar ações na hora do click:

finishToggleFrame:GetChildren()[2].MouseButton1Click:Connect(function()
    if getFinishState() then
        teleportToFinish()
    end
end)

gearToggleFrame:GetChildren()[2].MouseButton1Click:Connect(function()
    if getGearState() then
        getAllGears()
    end
end)


-- Evento para reaparecer manter a GUI visível e carregada
player.CharacterAdded:Connect(function(char)
    -- Garante a GUI se mantenha ativa após death e respawn
    screenGui.Parent = player.PlayerGui
    screenGui.Enabled = true
end)
