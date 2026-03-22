local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera
local gui = Instance.new("ScreenGui")
gui.Name = "AimbotGUI"
gui.Parent = player:WaitForChild("PlayerGui")

-- Configuration menu
local menu = Instance.new("Frame")
menu.Name = "AimbotMenu"
menu.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
menu.BorderSizePixel = 2
menu.Size = UDim2.new(0, 350, 0, 250)
menu.Position = UDim2.new(0.5, -175, 0.5, -125)
menu.Visible = true
menu.Parent = gui

-- Title label
local title = Instance.new("TextLabel")
title.Name = "Title"
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.Text = "Asus Aimbot Universal v1.0"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Size = UDim2.new(0, 350, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.Parent = menu

-- Configuration options
local config = {
    FOV = 80,
    AimKey = Enum.KeyCode.E,
    ESPEnabled = true,
    Smoothness = 3,
    FollowMode = true,
    AimbotEnabled = true
}

-- Button factory
local function createButton(text, callback)
    local button = Instance.new("TextButton")
    button.Name = text
    button.BackgroundTransparency = 0
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Size = UDim2.new(0, 330, 0, 30)
    button.Position = UDim2.new(0, 10, 0, #menu:GetChildren() * 35 + 30)
    button.MouseButton1Click:Connect(callback)
    button.Parent = menu
    return button
end

-- Create toggle buttons
createButton("Toggle ESP: "..tostring(config.ESPEnabled), function()
    config.ESPEnabled = not config.ESPEnabled
    menu:FindFirstChild("ESPLabel").Text = "ESP: "..tostring(config.ESPEnabled)
    print("ESP: " .. tostring(config.ESPEnabled))
end)

createButton("Toggle Aimbot: "..tostring(config.AimbotEnabled), function()
    config.AimbotEnabled = not config.AimbotEnabled
    menu:FindFirstChild("AimbotLabel").Text = "Aimbot: "..tostring(config.AimbotEnabled)
    print("Aimbot: " .. tostring(config.AimbotEnabled))
end)

-- Create slider controls
local function createSlider(label, min, max, defaultValue, onChange)
    local sliderContainer = Instance.new("Frame")
    sliderContainer.Name = label.."Slider"
    sliderContainer.BackgroundTransparency = 1
    sliderContainer.Size = UDim2.new(0, 330, 0, 40)
    sliderContainer.Position = UDim2.new(0, 10, 0, #menu:GetChildren() * 35 + 30)
    sliderContainer.Parent = menu
    
    local sliderLabel = Instance.new("TextLabel")
    sliderLabel.Name = label.."Label"
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.Text = label..": "..defaultValue
    sliderLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    sliderLabel.Size = UDim2.new(0, 330, 0, 20)
    sliderLabel.Position = UDim2.new(0, 0, 0, 0)
    sliderLabel.Parent = sliderContainer
    
    local slider = Instance.new("TextBox")
    slider.Name = label.."Slider"
    slider.BackgroundTransparency = 0
    slider.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    slider.Text = defaultValue
    slider.TextColor3 = Color3.fromRGB(255, 255, 255)
    slider.Size = UDim2.new(0, 330, 0, 20)
    slider.Position = UDim2.new(0, 0, 0, 20)
    slider.Parent = sliderContainer
    
    slider.FocusLost:Connect(function()
        local value = tonumber(slider.Text)
        if value >= min and value <= max then
            onChange(value)
            sliderLabel.Text = label..": "..value
        else
            slider.Text = defaultValue
        end
    end)
    
    return sliderContainer
end

-- Create configuration sliders
createSlider("FOV", 40, 160, config.FOV, function(value)
    config.FOV = value
    print("FOV set to: " .. tostring(value))
end)

createSlider("Smoothness", 1, 10, config.Smoothness, function(value)
    config.Smoothness = value
    print("Smoothness set to: " .. tostring(value))
end)

-- Create static FOV circle
local fovCircle = Instance.new("Frame")
fovCircle.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
fovCircle.BorderSizePixel = 2
fovCircle.Size = UDim2.new(0, config.FOV, 0, config.FOV)
fovCircle.Position = UDim2.new(0.5, -config.FOV/2, 0.5, -config.FOV/2)
fovCircle.Visible = true
fovCircle.Parent = gui

-- Player tracking
local players = {}
local espObjects = {}

-- Create ESP for each player
local function createESP(player)
    local character = player.Character
    if not character then return end
    
    local head = character:FindFirstChild("Head")
    if not head then return end
    
    local box = Instance.new("Frame")
    box.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    box.BorderSizePixel = 2
    box.Size = UDim2.new(0, 30, 0, 30)
    box.Visible = false
    box.Parent = gui
    
    table.insert(espObjects, {player = player, frame = box})
    return box
end

-- Update ESP positions
local function updateESP()
    for _, data in pairs(espObjects) do
        local player = data.player
        local frame = data.frame
        
        local character = player.Character
        if not character or not character:FindFirstChild("Head") then
            frame.Visible = false
            return
        end
        
        local head = character.Head
        local position = head.Position
        local screenPos = camera:WorldToViewportPoint(position)
        
        if screenPos.Visible then
            frame.Visible = true
            frame.Position = UDim2.new(0, screenPos.X - 15, 0, screenPos.Y - 15)
        else
            frame.Visible = false
        end
    end
end

-- Fixed aim function
local function fixedAim(target)
    if not target then return end
    
    local head = target:FindFirstChild("Head")
    if not head then return end
    
    local position = head.Position
    local screenPos = camera:WorldToViewportPoint(position)
    
    if screenPos.Visible then
        mouse.Target = target
        mouse.X = screenPos.X
        mouse.Y = screenPos.Y
    end
end

-- Main loop
game:GetService("RunService").Heartbeat:Connect(function()
    if not config.AimbotEnabled or not config.AimKey:IsPressed() then return end
    
    local closestPlayer = nil
    local closestDistance = math.huge
    
    for _, p in pairs(game.Players:GetPlayers()) do
        if p == player then continue end
        
        local character = p.Character
        if not character or not character:FindFirstChild("Head") then continue end
        
        local head = character.Head
        local screenPos = camera:WorldToViewportPoint(head.Position)
        
        if screenPos.Visible then
            local distance = Vector2.new(screenPos.X, screenPos.Y).Magnitude
            if distance < config.FOV/2 and distance < closestDistance then
                closestDistance = distance
                closestPlayer = head
            end
        end
    end
    
    if closestPlayer then
        fixedAim(closestPlayer)
    end
    
    updateESP()
end)

-- Initialize
for _, p in pairs(game.Players:GetPlayers()) do
    p.CharacterAdded:Connect(function()
        createESP(p)
    end)
end

print("Asus Aimbot Universal loaded with full control panel!")
