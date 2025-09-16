# 6669178
反重力脚本测试
lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = game:GetService("Players").LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

_G.processedParts = {}
_G.floatSpeed = 10 -- 默认漂浮速度
_G.moveDirection = Vector3.new(0, 1, 0) -- 默认向上移动

RunService.Heartbeat:Connect(function()
    sethiddenproperty(LocalPlayer, "SimulationRadius", math.huge)
end)

local function Parts(v)
    if v:IsA("Part") and not v.Anchored and not v.Parent:FindFirstChild("Humanoid") and not v.Parent:FindFirstChild("Head") then
        if _G.processedParts[v] then
            local existingBV = _G.processedParts[v].bodyVelocity
            if existingBV and existingBV.Parent then
                -- 使用标量速度和方向向量计算最终速度
                local finalVelocity = _G.moveDirection.Unit * _G.floatSpeed
                if existingBV.Velocity ~= finalVelocity then
                    existingBV.Velocity = finalVelocity
                end
                return
            else
                _G.processedParts[v] = nil
            end
        end
        
        for _, x in next, v:GetChildren() do
            if x:IsA("BodyAngularVelocity") or x:IsA("BodyForce") or x:IsA("BodyGyro") or x:IsA("BodyPosition") or x:IsA("BodyThrust") or x:IsA("BodyVelocity") then
                x:Destroy()
            end
        end
        
        if v:FindFirstChild("Torque") then
            v:FindFirstChild("Torque"):Destroy()
        end
        
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Parent = v
        -- 使用标量速度和方向向量计算最终速度
        bodyVelocity.Velocity = _G.moveDirection.Unit * _G.floatSpeed
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        _G.processedParts[v] = { bodyVelocity = bodyVelocity }
    end
end

local anActivity = false

local function kqan()
    if anActivity then
        for _, v in next, Workspace:GetDescendants() do
            Parts(v)
        end
    end
end

Workspace.DescendantAdded:Connect(function(v)
    if anActivity then
        Parts(v)
    end
end)

local function cleanupParts()
    for _, data in pairs(_G.processedParts) do
        if data.bodyVelocity then
            data.bodyVelocity:Destroy()
        end
    end
    _G.processedParts = {}
end

-- 更新所有物体的速度
local function updateAllPartsVelocity()
    for part, data in pairs(_G.processedParts) do
        if data.bodyVelocity and data.bodyVelocity.Parent then
            data.bodyVelocity.Velocity = _G.moveDirection.Unit * _G.floatSpeed
        end
    end
end

-- 使GUI元素可拖动的函数
local function makeDraggable(gui)
    gui.Active = true
    gui.Draggable = true
    
    -- 添加拖动指示器（右上角的小手柄）
    local dragHandle = Instance.new("Frame")
    dragHandle.Name = "DragHandle"
    dragHandle.Size = UDim2.new(0, 20, 0, 20)
    dragHandle.Position = UDim2.new(1, -20, 0, 0)
    dragHandle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
    dragHandle.BorderSizePixel = 0
    dragHandle.Parent = gui
    
    local gripIcon = Instance.new("TextLabel")
    gripIcon.Name = "GripIcon"
    gripIcon.Size = UDim2.new(1, 0, 1, 0)
    gripIcon.Position = UDim2.new(0, 0, 0, 0)
    gripIcon.Text = "≡"
    gripIcon.TextColor3 = Color3.new(1, 1, 1)
    gripIcon.BackgroundTransparency = 1
    gripIcon.TextSize = 14
    gripIcon.Parent = dragHandle
    
    -- 让拖动手柄也能拖动整个GUI
    dragHandle.Active = true
    dragHandle.Draggable = true
    dragHandle.MouseButton1Down:Connect(function()
        gui:Drag()
    end)
end

-- 创建手机友好的GUI
local function createMobileGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MobileFloatingControl"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    
    -- 主开关按钮
    local mainButton = Instance.new("TextButton")
    mainButton.Name = "MainToggle"
    mainButton.Size = UDim2.new(0, 120, 0, 50)
    mainButton.Position = UDim2.new(0.5, -60, 0, 10)
    mainButton.Text = "漂浮: 关闭"
    mainButton.TextSize = 16
    mainButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    mainButton.TextColor3 = Color3.new(1, 1, 1)
    mainButton.Parent = screenGui
    
    -- 使主按钮可拖动
    makeDraggable(mainButton)
    
    -- 控制面板
    local controlPanel = Instance.new("Frame")
    controlPanel.Name = "ControlPanel"
    controlPanel.Size = UDim2.new(0, 300, 0, 400)
    controlPanel.Position = UDim2.new(0.5, -150, 0.5, -200)
    controlPanel.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    controlPanel.BackgroundTransparency = 0.3
    controlPanel.BorderSizePixel = 0
    controlPanel.Visible = false
    controlPanel.Parent = screenGui
    
    -- 使控制面板可拖动
    makeDraggable(controlPanel)
    
    -- 速度控制
    local speedLabel = Instance.new("TextLabel")
    speedLabel.Name = "SpeedLabel"
    speedLabel.Size = UDim2.new(1, 0, 0, 40)
    speedLabel.Position = UDim2.new(0, 0, 0, 10)
    speedLabel.Text = "速度: " .. _G.floatSpeed
    speedLabel.TextColor3 = Color3.new(1, 1, 1)
    speedLabel.BackgroundTransparency = 1
    speedLabel.TextSize = 20
    speedLabel.Parent = controlPanel
    
    -- 速度增加按钮
    local speedUpButton = Instance.new("TextButton")
    speedUpButton.Name = "SpeedUp"
    speedUpButton.Size = UDim2.new(0, 60, 0, 60)
    speedUpButton.Position = UDim2.new(0.7, 0, 0, 60)
    speedUpButton.Text = "+"
    speedUpButton.TextSize = 30
    speedUpButton.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
    speedUpButton.TextColor3 = Color3.new(1, 1, 1)
    speedUpButton.Parent = controlPanel
    
    -- 速度减少按钮
    local speedDownButton = Instance.new("TextButton")
    speedDownButton.Name = "SpeedDown"
    speedDownButton.Size = UDim2.new(0, 60, 0, 60)
    speedDownButton.Position = UDim2.new(0.3, 0, 0, 60)
    speedDownButton.Text = "-"
    speedDownButton.TextSize = 30
    speedDownButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    speedDownButton.TextColor3 = Color3.new(1, 1, 1)
    speedDownButton.Parent = controlPanel
    
    -- 方向控制标题
    local directionLabel = Instance.new("TextLabel")
    directionLabel.Name = "DirectionLabel"
    directionLabel.Size = UDim2.new(1, 0, 0, 40)
    directionLabel.Position = UDim2.new(0, 0, 0, 140)
    directionLabel.Text = "移动方向"
    directionLabel.TextColor3 = Color3.new(1, 1, 1)
    directionLabel.BackgroundTransparency = 1
    directionLabel.TextSize = 20
    directionLabel.Parent = controlPanel
    
    -- 方向按钮网格
    local directions = {
        {name = "向上", dir = Vector3.new(0, 1, 0), pos = UDim2.new(0.5, -30, 0, 190)},
        {name = "向下", dir = Vector3.new(0, -1, 0), pos = UDim2.new(0.5, -30, 0, 260)},
        {name = "向前", dir = Vector3.new(0, 0, 1), pos = UDim2.new(0.2, -30, 0, 225)},
        {name = "向后", dir = Vector3.new(0, 0, -1), pos = UDim2.new(0.8, -30, 0, 225)},
        {name = "向左", dir = Vector3.new(-1, 0, 0), pos = UDim2.new(0.05, -30, 0, 225)},
        {name = "向右", dir = Vector3.new(1, 0, 0), pos = UDim2.new(0.95, -30, 0, 225)}
    }
    
    local directionButtons = {}
    
    for i, dirInfo in ipairs(directions) do
        local button = Instance.new("TextButton")
        button.Name = dirInfo.name
        button.Size = UDim2.new(0, 60, 0, 60)
        button.Position = dirInfo.pos
        button.Text = dirInfo.name
        button.TextSize = 14
        button.BackgroundColor3 = Color3.fromRGB(100, 100, 200)
        button.TextColor3 = Color3.new(1, 1, 1)
        button.Parent = controlPanel
        
        button.MouseButton1Click:Connect(function()
            _G.moveDirection = dirInfo.dir
            updateAllPartsVelocity()
            
            -- 视觉反馈
            local originalColor = button.BackgroundColor3
            button.BackgroundColor3 = Color3.fromRGB(255, 255, 0)
            wait(0.2)
            button.BackgroundColor3 = originalColor
        end)
        
        directionButtons[dirInfo.name] = button
    end
    
    -- 关闭面板按钮
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "ClosePanel"
    closeButton.Size = UDim2.new(0, 100, 0, 40)
    closeButton.Position = UDim2.new(0.5, -50, 0, 340)
    closeButton.Text = "关闭面板"
    closeButton.TextSize = 16
    closeButton.BackgroundColor3 = Color3.fromRGB(200, 100, 100)
    closeButton.TextColor3 = Color3.new(1, 1, 1)
    closeButton.Parent = controlPanel
    
    -- 速度按钮功能
    speedUpButton.MouseButton1Click:Connect(function()
        _G.floatSpeed = math.clamp(_G.floatSpeed + 5, 1, 50)
        speedLabel.Text = "速度: " .. _G.floatSpeed
        updateAllPartsVelocity()
        
        -- 视觉反馈
        local originalColor = speedUpButton.BackgroundColor3
        speedUpButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        wait(0.2)
        speedUpButton.BackgroundColor3 = originalColor
    end)
    
    speedDownButton.MouseButton1Click:Connect(function()
        _G.floatSpeed = math.clamp(_G.floatSpeed - 5, 1, 50)
        speedLabel.Text = "速度: " .. _G.floatSpeed
        updateAllPartsVelocity()
        
        -- 视觉反馈
        local originalColor = speedDownButton.BackgroundColor3
        speedDownButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        wait(0.2)
        speedDownButton.BackgroundColor3 = originalColor
    end)
    
    -- 主开关功能
    mainButton.MouseButton1Click:Connect(function()
        anActivity = not anActivity
        kqan()
        if anActivity then
            mainButton.Text = "漂浮: 开启"
            mainButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            controlPanel.Visible = true
            updateAllPartsVelocity()
        else
            cleanupParts()
            mainButton.Text = "漂浮: 关闭"
            mainButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            controlPanel.Visible = false
        end
    end)
    
    -- 关闭面板功能
    closeButton.MouseButton1Click:Connect(function()
        controlPanel.Visible = false
    end)
    
    -- 添加一个打开面板的按钮，当面板关闭时显示
    local openPanelButton = Instance.new("TextButton")
    openPanelButton.Name = "OpenPanel"
    openPanelButton.Size = UDim2.new(0, 120, 0, 40)
    openPanelButton.Position = UDim2.new(0.5, -60, 0, 70)
    openPanelButton.Text = "打开控制面板"
    openPanelButton.TextSize = 14
    openPanelButton.BackgroundColor3 = Color3.fromRGB(100, 100, 200)
    openPanelButton.TextColor3 = Color3.new(1, 1, 1)
    openPanelButton.Visible = false
    openPanelButton.Parent = screenGui
    
    -- 使打开面板按钮可拖动
    makeDraggable(openPanelButton)
    
    openPanelButton.MouseButton1Click:Connect(function()
        controlPanel.Visible = true
        openPanelButton.Visible = false
    end)
    
    closeButton.MouseButton1Click:Connect(function()
        controlPanel.Visible = false
        openPanelButton.Visible = true
    end)
    
    return screenGui
end

-- 创建并显示GUI
createMobileGUI()