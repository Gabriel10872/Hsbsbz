-- ProximityPrompt: torna todos instantâneos
for _, p in ipairs(game:GetDescendants()) do
    if p:IsA("ProximityPrompt") then p.HoldDuration = 0 end
end
game.DescendantAdded:Connect(function(obj)
    if obj:IsA("ProximityPrompt") then obj.HoldDuration = 0 end
end)

-- GUI
local StarterGui = game:GetService("StarterGui")
pcall(function() StarterGui:SetCore("SendNotification",{Title="Teleport System",Text="Carregando Interface...",Duration=3}) end)

local Player = game.Players.LocalPlayer
local Mouse = Player:GetMouse()
local Teleporting = false
local InitialPos = nil
local TargetReachedTime = 0

-- GUI Criar
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 160, 0, 60)
Frame.Position = UDim2.new(0.5, -80, 0.85, 0)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0
Frame.BackgroundTransparency = 0.2
Frame.AnchorPoint = Vector2.new(0.5, 0.5)
Frame.Active = true
Frame.Draggable = true
Frame.Name = "SmoothTPGui"

local UICorner = Instance.new("UICorner", Frame)
UICorner.CornerRadius = UDim.new(0, 10)

local Button = Instance.new("TextButton", Frame)
Button.Size = UDim2.new(1, -10, 1, -10)
Button.Position = UDim2.new(0, 5, 0, 5)
Button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
Button.TextColor3 = Color3.fromRGB(255,255,255)
Button.TextScaled = true
Button.Text = "Steal"
Button.Font = Enum.Font.GothamBold

Instance.new("UICorner", Button).CornerRadius = UDim.new(0, 8)

-- Função de voo suave atravessando tudo
local function flyTo(position)
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    local hrp = char.HumanoidRootPart
    local flying = true
    local BodyGyro = Instance.new("BodyGyro", hrp)
    BodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    BodyGyro.P = 100000
    BodyGyro.CFrame = hrp.CFrame

    local BodyVelocity = Instance.new("BodyVelocity", hrp)
    BodyVelocity.Velocity = Vector3.zero
    BodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)

    local reached = false
    local arrivedTime = 0

    while flying and Teleporting do
        task.wait(0.03)
        if not char or not char:FindFirstChild("HumanoidRootPart") then break end

        local dir = (position - hrp.Position)
        local dist = dir.Magnitude

        if dist < 1 then
            if arrivedTime == 0 then arrivedTime = tick() end
            if tick() - arrivedTime >= 2 then
                flying = false
                break
            end
        else
            arrivedTime = 0
        end

        BodyVelocity.Velocity = dir.Unit * math.clamp(dist * 2.5, 20, 100)
        BodyGyro.CFrame = CFrame.new(hrp.Position, position)
    end

    BodyVelocity:Destroy()
    BodyGyro:Destroy()
end

-- Ação do botão
Button.MouseButton1Click:Connect(function()
    if Teleporting then
        Teleporting = false
        Button.Text = "Steal"
        return
    end

    Teleporting = true
    Button.Text = "Cancel"

    if not InitialPos then
        local char = Player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            InitialPos = char.HumanoidRootPart.Position + Vector3.new(0, 15, 0)
        end
    end

    task.delay(0.2, function()
        if InitialPos then
            flyTo(InitialPos)

            -- Congelar no ar por 6 segundos
            local hrp = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local freeze = Instance.new("BodyPosition", hrp)
                freeze.Position = hrp.Position
                freeze.MaxForce = Vector3.new(9e9, 9e9, 9e9)
                task.wait(6)
                freeze:Destroy()
            end

            Button.Text = "Steal"
            Teleporting = false
        end
    end)
end)
