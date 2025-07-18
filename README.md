local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")

-- Salva posição inicial (pouco acima)
local initialPos = hrp.Position + Vector3.new(0, 7, 0)

-- Config GUI
local stealing = false
local moving = false
local maxAttempts = 30
local attempts = 0

-- NoClip enquanto move
RunService.Stepped:Connect(function()
	if moving and character and character.Parent then
		for _, part in ipairs(character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = false
				part.Massless = true
			end
		end
	end
end)

-- GUI existente (não modifiquei o visual, só adiciono draggable)
local gui = Instance.new("ScreenGui")
gui.Name = "StealTopGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local blur = Instance.new("ImageLabel")
blur.Size = UDim2.new(0, 220, 0, 80)
blur.Position = UDim2.new(0.5, -110, 0.8, 0)
blur.BackgroundTransparency = 1
blur.Image = "rbxassetid://3570695787"
blur.ImageColor3 = Color3.fromRGB(15,15,15)
blur.ImageTransparency = 0.7
blur.ScaleType = Enum.ScaleType.Slice
blur.SliceCenter = Rect.new(100, 100, 100, 100)
blur.Parent = gui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(1, -20, 1, -20)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(35,35,40)
frame.BorderSizePixel = 0
frame.ClipsDescendants = true
frame.Parent = blur
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,16)

-- Tornar frame movível
frame.Active = true
frame.Draggable = true

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, 0, 0, 25)
status.BackgroundTransparency = 1
status.TextColor3 = Color3.fromRGB(180, 180, 180)
status.TextStrokeColor3 = Color3.new(0,0,0)
status.TextStrokeTransparency = 0.8
status.Font = Enum.Font.Gotham
status.TextSize = 18
status.Text = "Status: Ready"
status.Parent = frame

local btn = Instance.new("TextButton")
btn.Size = UDim2.new(1, 0, 0, 40)
btn.Position = UDim2.new(0, 0, 0, 30)
btn.BackgroundColor3 = Color3.fromRGB(90, 90, 255)
btn.AutoButtonColor = false
btn.Font = Enum.Font.GothamBold
btn.TextSize = 20
btn.TextColor3 = Color3.new(1,1,1)
btn.Text = "Steal"
btn.Parent = frame
Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 16)

local clickSound = Instance.new("Sound", btn)
clickSound.SoundId = "rbxassetid://12222030"
clickSound.Volume = 0.5

btn.MouseEnter:Connect(function()
	btn.BackgroundColor3 = Color3.fromRGB(115,115,255)
end)
btn.MouseLeave:Connect(function()
	btn.BackgroundColor3 = Color3.fromRGB(90,90,255)
end)

local function smoothFly(destination)
	moving = true
	local hrp = character and character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	local duration = 4
	local startPos = hrp.Position
	local goal = destination

	local tween = TweenService:Create(hrp, TweenInfo.new(duration, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {CFrame = CFrame.new(goal)})
	tween:Play()
	tween.Completed:Wait()

	moving = false
end

local function waitInPlace(pos, waitTime)
	local timer = 0
	while timer < waitTime and stealing do
		RunService.RenderStepped:Wait()
		if (hrp.Position - pos).Magnitude < 3 then
			timer += RunService.RenderStepped:Wait()
		else
			timer = 0
		end
	end
end

local function attemptSteal()
	attempts = 0
	status.Text = "Status: Starting..."
	while stealing and attempts < maxAttempts do
		attempts += 1
		status.Text = ("Status: Teleport attempt %d/%d"):format(attempts, maxAttempts)
		clickSound:Play()
		smoothFly(initialPos)
		waitInPlace(initialPos, 2)
		if not stealing then break end
		status.Text = "Status: Hovering 6s..."
		waitInPlace(initialPos, 6) -- espera 6 segundos no ar antes de tentar de novo
	end
	status.Text = "Status: Done!"
	stealing = false
	btn.Text = "Steal"
end

btn.MouseButton1Click:Connect(function()
	if stealing then
		stealing = false
		status.Text = "Status: Cancelled"
		btn.Text = "Steal"
	else
		stealing = true
		btn.Text = "Cancel"
		task.spawn(attemptSteal)
	end
end)
