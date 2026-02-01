local Players = game:GetService("Players")
local player = Players.LocalPlayer
local runService = game:GetService("RunService")

local autoFarmActive = false
local antiAfkActive = false
local flyActive = false
local flySpeed = 50 
local itemName = "GoldBar"

local function startFly()
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local hum = char:WaitForChild("Humanoid")
    
    for _, v in pairs(hrp:GetChildren()) do
        if v.Name == "IYFlyVelocity" or v.Name == "IYFlyGyro" then v:Destroy() end
    end

    local velocity = Instance.new("BodyVelocity")
    velocity.Name = "IYFlyVelocity"
    velocity.Parent = hrp
    velocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    velocity.Velocity = Vector3.new(0, 0, 0)

    local gyro = Instance.new("BodyGyro")
    gyro.Name = "IYFlyGyro"
    gyro.Parent = hrp
    gyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    gyro.CFrame = hrp.CFrame

    hum.PlatformStand = true 

    task.spawn(function()
        while flyActive and char.Parent do
            local camera = workspace.CurrentCamera
            local moveDir = hum.MoveDirection
            
            if moveDir.Magnitude > 0 then
                velocity.Velocity = camera.CFrame.LookVector * flySpeed
            else
                velocity.Velocity = Vector3.new(0, 0, 0)
            end
            
            gyro.CFrame = camera.CFrame
            runService.Heartbeat:Wait()
        end
        if velocity then velocity:Destroy() end
        if gyro then gyro:Destroy() end
        if hum then hum.PlatformStand = false end
    end)
end

-- Interface
local function criarInterface()
    if player.PlayerGui:FindFirstChild("VermelhoHubGui") then player.PlayerGui.VermelhoHubGui:Destroy() end

    local sg = Instance.new("ScreenGui", player.PlayerGui)
    sg.Name = "VermelhoHubGui"
    sg.ResetOnSpawn = false
    sg.DisplayOrder = 9999

    local main = Instance.new("Frame", sg)
    main.Size = UDim2.new(0, 420, 0, 280) -- Aumentado um pouco para caber o texto embaixo
    main.Position = UDim2.new(0.5, -210, 0.5, -140)
    main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    main.Active = true
    main.Draggable = true 
    Instance.new("UICorner", main)

    local hubTitle = Instance.new("TextLabel", main)
    hubTitle.Size = UDim2.new(1, 0, 0, 30)
    hubTitle.Position = UDim2.new(0, 0, 1, -30)
    hubTitle.Text = "VERMELHO HUB"
    hubTitle.TextColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho vivo
    hubTitle.Font = Enum.Font.GothamBold
    hubTitle.TextSize = 18
    hubTitle.BackgroundTransparency = 1

    local minBtn = Instance.new("TextButton", sg)
    minBtn.Size = UDim2.new(0, 50, 0, 50)
    minBtn.Position = UDim2.new(0, 10, 0.5, -25)
    minBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    minBtn.Text = "V" -- Trocado de R para V
    minBtn.TextColor3 = Color3.fromRGB(255, 0, 0)
    minBtn.Font = "GothamBold"
    minBtn.TextSize = 25
    minBtn.ZIndex = 10000
    minBtn.Active = true
    minBtn.Draggable = true 
    Instance.new("UICorner", minBtn).CornerRadius = UDim.new(1, 0)

    minBtn.MouseButton1Click:Connect(function()
        main.Visible = not main.Visible
    end)

    local sidebar = Instance.new("Frame", main)
    sidebar.Size = UDim2.new(0, 100, 1, -35) -- Espaço para o título embaixo
    sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Instance.new("UICorner", sidebar)
    local sideLayout = Instance.new("UIListLayout", sidebar)
    sideLayout.Padding = UDim.new(0, 5)
    sideLayout.HorizontalAlignment = "Center"

    local content = Instance.new("ScrollingFrame", main)
    content.Position = UDim2.new(0, 110, 0, 10)
    content.Size = UDim2.new(1, -120, 1, -50)
    content.BackgroundTransparency = 1
    content.ScrollBarThickness = 2
    local contentLayout = Instance.new("UIListLayout", content)
    contentLayout.Padding = UDim.new(0, 10)

    local function clearContent()
        for _, v in pairs(content:GetChildren()) do
            if not v:IsA("UIListLayout") then v:Destroy() end
        end
    end

    local function createToggle(text, callback, state)
        local btn = Instance.new("TextButton", content)
        btn.Size = UDim2.new(0.95, 0, 0, 40)
        btn.BackgroundColor3 = state and Color3.fromRGB(200, 0, 0) or Color3.fromRGB(35, 35, 35)
        btn.Text = text .. (state and ": ON" or ": OFF")
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.Font = "Gotham"
        Instance.new("UICorner", btn)
        btn.MouseButton1Click:Connect(function()
            state = not state
            btn.Text = text .. (state and ": ON" or ": OFF")
            btn.BackgroundColor3 = state and Color3.fromRGB(200, 0, 0) or Color3.fromRGB(35, 35, 35)
            callback(state)
        end)
    end

    -- Abas
    local function addTab(name, func)
        local b = Instance.new("TextButton", sidebar)
        b.Size = UDim2.new(0.9, 0, 0, 35)
        b.Text = name
        b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        b.TextColor3 = Color3.new(1, 1, 1)
        b.Font = "GothamBold"
        Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function()
            clearContent()
            func()
        end)
    end

    addTab("Evento", function()
        createToggle("Coletar Moedas", function(v) autoFarmActive = v end, autoFarmActive)
    end)

    addTab("Misc", function()
        createToggle("Fly (IY Style)", function(v) 
            flyActive = v 
            if v then startFly() end
        end, flyActive)
        
        local label = Instance.new("TextLabel", content)
        label.Size = UDim2.new(0.95, 0, 0, 20)
        label.Text = "Velocidade de:" -- Alterado conforme pedido
        label.TextColor3 = Color3.new(1, 1, 1)
        label.BackgroundTransparency = 1
        label.Font = "Gotham"

        local speedInput = Instance.new("TextBox", content)
        speedInput.Size = UDim2.new(0.95, 0, 0, 35)
        speedInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        speedInput.Text = tostring(flySpeed)
        speedInput.TextColor3 = Color3.new(1, 1, 1)
        speedInput.PlaceholderText = "Digite a velocidade..."
        speedInput.Font = "Gotham"
        Instance.new("UICorner", speedInput)

        speedInput.FocusLost:Connect(function()
            local num = tonumber(speedInput.Text)
            if num then
                flySpeed = num
            else
                speedInput.Text = tostring(flySpeed)
            end
        end)

        createToggle("Anti-AFK", function(v) antiAfkActive = v end, antiAfkActive)
    end)

    addTab("FPS Boost", function()
        local b = Instance.new("TextButton", content)
        b.Size = UDim2.new(0.95, 0, 0, 40)
        b.Text = "Ativar Anti-Lag"
        b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        b.TextColor3 = Color3.new(1, 1, 1)
        Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function()
            for _, v in pairs(game:GetDescendants()) do
                if v:IsA("BasePart") then v.Material = Enum.Material.SmoothPlastic
                elseif v:IsA("Decal") then v:Destroy() end
            end
        end)
    end)
    
    clearContent()
    createToggle("Coletar Moedas", function(v) autoFarmActive = v end, autoFarmActive)
end

task.spawn(function()
    while true do
        if autoFarmActive then
            pcall(function()
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj.Name == itemName and obj:IsA("BasePart") then
                        obj.CFrame = player.Character.HumanoidRootPart.CFrame
                    end
                end
            end)
        end
        task.wait(0.4)
    end
end)

player.Idled:Connect(function()
    if antiAfkActive then
        game:GetService("VirtualUser"):CaptureController()
        game:GetService("VirtualUser"):ClickButton2(Vector2.new())
    end
end)

criarInterface()

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local autoFarmActive = false

local NOME_MOEDA = "GoldBar" 

local function teleportarMoedas()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    
    local fila = {}

    for _, obj in pairs(workspace:GetDescendants()) do
        -- Verifica se o nome é EXATAMENTE o da moeda e se não é um Brainrot
        if obj.Name == NOME_MOEDA and obj:IsA("BasePart") then
            -- Segurança extra: verifica se está dentro da pasta de eventos
            if obj:FindFirstAncestor("EventParts") or obj.Parent.Name == NOME_MOEDA then
                table.insert(fila, obj)
            end
        end
    end

    for _, moeda in ipairs(fila) do
        if not autoFarmActive then break end
        if moeda and moeda.Parent then
            moeda.CanCollide = false
            moeda.CFrame = hrp.CFrame
            task.wait(0.02) -- Velocidade máxima permitida pelo servidor
        end
    end
end

local function criarInterface()
    if player.PlayerGui:FindFirstChild("RedzStyleMenu") then
        player.PlayerGui.RedzStyleMenu:Destroy()
    end

    local sg = Instance.new("ScreenGui", player.PlayerGui)
    sg.Name = "RedzStyleMenu"
    sg.ResetOnSpawn = false

    local main = Instance.new("Frame", sg)
    main.Size = UDim2.new(0, 220, 0, 150)
    main.Position = UDim2.new(0.5, -110, 0.3, 0)
    main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    main.Active = true
    main.Draggable = true
    Instance.new("UICorner", main)

    local glow = Instance.new("Frame", main)
    glow.Size = UDim2.new(1, 0, 0, 2)
    glow.BackgroundColor3 = Color3.fromRGB(0, 255, 100) -- Cor Verde Radioativa
    glow.BorderSizePixel = 0

    local title = Instance.new("TextLabel", main)
    title.Size = UDim2.new(1, 0, 0, 40)
    title.Text = "ONLY COINS FARM"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.GothamBold
    title.TextSize = 13

    local btnAuto = Instance.new("TextButton", main)
    btnAuto.Size = UDim2.new(0.9, 0, 0, 50)
    btnAuto.Position = UDim2.new(0.05, 0, 0.4, 0)
    btnAuto.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btnAuto.Text = "AUTO-FARM: OFF"
    btnAuto.TextColor3 = Color3.new(1, 1, 1)
    btnAuto.Font = Enum.Font.GothamBold
    Instance.new("UICorner", btnAuto)

    btnAuto.MouseButton1Click:Connect(function()
        autoFarmActive = not autoFarmActive
        btnAuto.Text = autoFarmActive and "AUTO-FARM: ON" or "AUTO-FARM: OFF"
        btnAuto.BackgroundColor3 = autoFarmActive and Color3.fromRGB(0, 200, 80) or Color3.fromRGB(30, 30, 30)
    end)

    task.spawn(function()
        while true do
            if autoFarmActive then teleportarMoedas() end
            task.wait(0.5)
        end
    end)
end

criarInterface()
