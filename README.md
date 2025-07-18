local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

-- NoClip
RunService.Stepped:Connect(function()
    for _, v in ipairs(char:GetDescendants()) do
        if v:IsA("BasePart") then
            v.CanCollide = false
        end
    end
end)

-- ProximityPrompt Auto Hold
for _, p in ipairs(workspace:GetDescendants()) do
    if p:IsA("ProximityPrompt") then
        p.HoldDuration = 0
    end
end

-- Save initial spawn position (slightly above ground)
local spawnPos = hrp.Position + Vector3.new(0, 5, 0)

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "StealGUI"

local main = Instance.new("Frame", gui)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.Position = UDim2.new(0.5, 0, 0.6, 0)
main.Size = UDim2.new(0, 180, 0, 65)
main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
main.BorderSizePixel = 0
main.BackgroundTransparency = 0.1
main.ClipsDescendants = true
main.Active = true
main.Draggable = true
main.UICorner = Instance.new("UICorner", main)
main.UICorner.CornerRadius = UDim.new(0, 12)

local btn = Instance.new("TextButton", main)
btn.Size = UDim2.new(1, -20, 1, -20)
btn.Position = UDim2.new(0, 10, 0, 10)
btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
btn.TextColor3 = Color3.fromRGB(255, 255, 255)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 16
btn.Text = "Steal"
btn.UICorner = Instance.new("UICorner", btn)
btn.UICorner.CornerRadius = UDim.new(0, 8)

-- Control flags
local isStealing = false
local teleportAttempts = 0
local MAX_ATTEMPTS = 30

local function attemptTeleport()
    local startTime = tick()
    local lastInZone = tick()
    local checking = true

    while checking and teleportAttempts < MAX_ATTEMPTS do
        teleportAttempts += 1

        local tween = TweenService:Create(hrp, TweenInfo.new(0.8, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
            CFrame = CFrame.new(spawnPos)
        })
        tween:Play()

        -- Verifica permanência
        local staying = true
        local stayStart = tick()

        while staying do
            RunService.RenderStepped:Wait()
            local dist = (hrp.Position - spawnPos).Magnitude
            if dist > 5 then
                stayStart = tick() -- Reset se saiu da posição
            elseif tick() - stayStart >= 2 then
                checking = false
                staying = false
                break
            end
        end

        task.wait(0.8)
    end

    btn.Text = "Steal"
    isStealing = false
end

btn.MouseButton1Click:Connect(function()
    if not isStealing then
        isStealing = true
        teleportAttempts = 0
        btn.Text = "Cancel"
        task.spawn(attemptTeleport)
    else
        isStealing = false
        btn.Text = "Steal"
    end
end)
