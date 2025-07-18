-- Gui principal
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
ScreenGui.Name = "FlyBackGui"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 60)
Frame.Position = UDim2.new(0.5, -100, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.BackgroundTransparency = 0.1
Frame.Active = true
Frame.Draggable = true
Frame.AnchorPoint = Vector2.new(0.5, 0)

local UICorner = Instance.new("UICorner", Frame)
UICorner.CornerRadius = UDim.new(0, 12)

local Button = Instance.new("TextButton", Frame)
Button.Size = UDim2.new(1, 0, 1, 0)
Button.Text = "Steal"
Button.Font = Enum.Font.GothamBold
Button.TextSize = 22
Button.TextColor3 = Color3.fromRGB(255, 255, 255)
Button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Button.BorderSizePixel = 0

local UIStroke = Instance.new("UIStroke", Frame)
UIStroke.Thickness = 2
UIStroke.Color = Color3.fromRGB(255, 255, 255)
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

local UICornerBtn = Instance.new("UICorner", Button)
UICornerBtn.CornerRadius = UDim.new(0, 12)

-- Lógica
local Player = game.Players.LocalPlayer
local InitialPos = nil
local Teleporting = false

-- Força os ProximityPrompts a terem holdDuration 0
for _, p in ipairs(workspace:GetDescendants()) do
	if p:IsA("ProximityPrompt") then
		p.HoldDuration = 0
	end
end

-- Desativa colisão pra atravessar tudo
local function disableCollision(char)
	for _, part in ipairs(char:GetDescendants()) do
		if part:IsA("BasePart") then
			part.CanCollide = false
			part.Massless = true
		end
	end
end

-- Voo suave até a posição
local function flyTo(position)
	local char = Player.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end
	disableCollision(char)

	local hrp = char.HumanoidRootPart
	local flying = true
	local BodyGyro = Instance.new("BodyGyro", hrp)
	BodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
	BodyGyro.P = 100000
	BodyGyro.CFrame = hrp.CFrame

	local BodyVelocity = Instance.new("BodyVelocity", hrp)
	BodyVelocity.Velocity = Vector3.zero
	BodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)

	local arrivedTime = 0

	while flying and Teleporting do
		task.wait(0.03)
		if not char or not char:FindFirstChild("HumanoidRootPart") then break end

		local dir = (position - hrp.Position)
		local dist = dir.Magnitude

		if dist < 1 then
			if arrivedTime == 0 then arrivedTime = tick() end
			if tick() - arrivedTime >= 2 then
				-- Congela no ar por 6 segundos
				BodyVelocity.Velocity = Vector3.new(0, 0, 0)
				task.wait(6)
				flying = false
				break
			end
		else
			arrivedTime = 0
		end

		BodyVelocity.Velocity = dir.Unit * math.clamp(dist * 2.5, 20, 60)
		BodyGyro.CFrame = CFrame.new(hrp.Position, position)
	end

	BodyVelocity:Destroy()
	BodyGyro:Destroy()
end

-- Botão
Button.MouseButton1Click:Connect(function()
	local char = Player.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end

	if not InitialPos then
		InitialPos = char.HumanoidRootPart.Position + Vector3.new(0, 5, 0)
	end

	if not Teleporting then
		Teleporting = true
		Button.Text = "Cancel"
		coroutine.wrap(function()
			while Teleporting do
				flyTo(InitialPos)
			end
		end)()
	else
		Teleporting = false
		Button.Text = "Steal"
	end
end)
