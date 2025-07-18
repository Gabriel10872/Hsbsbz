local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")

-- Armazena posição inicial (um pouco acima)
local initialPos = hrp.Position + Vector3.new(0, 7, 0)

-- Setar todos ProximityPrompt para HoldDuration = 0 (instant)
for _, prompt in ipairs(workspace:GetDescendants()) do
	if prompt:IsA("ProximityPrompt") then
		prompt.HoldDuration = 0
	end
end
workspace.DescendantAdded:Connect(function(obj)
	if obj:IsA("ProximityPrompt") then
		obj.HoldDuration = 0
	end
end)

-- Criar GUI moderna
local gui = Instance.new("ScreenGui")
gui.Name = "StealTopGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Background blur (sutil)
local blur = Instance.new("ImageLabel")
blur.Size = UDim2.new(0, 220, 0, 80)
blur.Position = UDim2.new(0.5, -110, 0.8, 0)
blur.BackgroundTransparency = 1
blur.Image = "rbxassetid://3570695787" -- sutil blur image
blur.ImageColor3 = Color3.fromRGB(15,15,15)
blur.ImageTransparency = 0.7
blur.ScaleType = Enum.ScaleType.Slice
blur.SliceCenter = Rect.new(100, 100, 100, 100)
blur.Parent = gui

-- Frame principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(1, -20, 1, -20)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(35,35,40)
frame.BorderSizePixel = 0
frame.ClipsDescendants = true
frame.Parent = blur
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,16)

-- Status label
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

-- Botão steal
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

-- Som de clique
local clickSound = Instance.new("Sound", btn)
clickSound.SoundId = "rbxassetid://12222030"
clickSound.Volume = 0.5

-- Hover effect
btn.MouseEnter:Connect(function()
	btn.BackgroundColor3 = Color3.fromRGB(115,115,255)
end)
btn.MouseLeave:Connect(function()
	btn.BackgroundColor3 = Color3.fromRGB(90,90,255)
end)

-- Flags controle
local stealing = false
local moving = false
local maxAttempts = 30
local attempts = 0

-- NoClip while moving
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

-- Função voo suave com TweenService e lerp para smoothness
local function smoothFly(destination)
	moving = true
	local hrp = character and character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	local duration = 4
	local startPos = hrp.Position
	local goal = destination

	-- Tween para suavizar movimento
	local tween = TweenService:Create(hrp, TweenInfo.new(duration, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {CFrame = CFrame.new(goal)})
	tween:Play()
	tween.Completed:Wait()

	-- Alternativa: lerp manual (mais controle)
	--[[
	local t = 0
	while t < duration and stealing do
		local dt = RunService.RenderStepped:Wait()
		t += dt
		local alpha = math.clamp(t/duration, 0,1)
		hrp.CFrame = CFrame.new(startPos:Lerp(goal, alpha))
	end
	]]

	moving = false
end

-- Função para esperar 2 segundos parado na posição
local function waitInPlace(pos)
	local timer = 0
	while timer < 2 and stealing do
		RunService.RenderStepped:Wait()
		if (hrp.Position - pos).Magnitude < 3 then
			timer += RunService.RenderStepped:Wait()
		else
			timer = 0
		end
	end
end

-- Função principal que tenta teleportar (máx 30 tentativas)
local function attemptSteal()
	attempts = 0
	status.Text = "Status: Starting..."
	while stealing and attempts < maxAttempts do
		attempts += 1
		status.Text = ("Status: Teleport attempt %d/%d"):format(attempts, maxAttempts)
		clickSound:Play()
		smoothFly(initialPos)
		waitInPlace(initialPos)
		if not stealing then break end
		task.wait(0.2)
	end
	status.Text = "Status: Done!"
	stealing = false
	btn.Text = "Steal"
end

-- Botão clicado
btn.MouseButton1Click:Connect(function()
	if stealing then
		-- Cancelar
		stealing = false
		status.Text = "Status: Cancelled"
		btn.Text = "Steal"
	else
		-- Iniciar
		stealing = true
		btn.Text = "Cancel"
		task.spawn(attemptSteal)
	end
end)
