local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local playerGui = player:WaitForChild("PlayerGui")

local spawnPos = hrp.Position

-- GUI Setup (usando seu último visual top)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UltraModernSteal"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 280, 0, 120)
frame.Position = UDim2.new(0.5, -140, 0.75, 0)
frame.BackgroundColor3 = Color3.fromRGB(28, 28, 30)
frame.BorderSizePixel = 0
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.ClipsDescendants = true
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 16)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(0, 170, 255)
stroke.Thickness = 3

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -20, 0, 30)
title.Position = UDim2.new(0, 10, 0, 10)
title.BackgroundTransparency = 1
title.Text = "Steal Teleport"
title.TextColor3 = Color3.fromRGB(0, 170, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 24
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = frame

local statusText = Instance.new("TextLabel")
statusText.Size = UDim2.new(1, -20, 0, 20)
statusText.Position = UDim2.new(0, 10, 0, 45)
statusText.BackgroundTransparency = 1
statusText.Text = "Clique em Steal para começar"
statusText.TextColor3 = Color3.fromRGB(180, 180, 180)
statusText.Font = Enum.Font.Gotham
statusText.TextSize = 16
statusText.TextXAlignment = Enum.TextXAlignment.Left
statusText.Parent = frame

local loadingFrame = Instance.new("Frame")
loadingFrame.Size = UDim2.new(0, 48, 0, 48)
loadingFrame.Position = UDim2.new(1, -60, 0.5, -24)
loadingFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
loadingFrame.AnchorPoint = Vector2.new(0, 0.5)
loadingFrame.Parent = frame

local loadingCorner = Instance.new("UICorner", loadingFrame)
loadingCorner.CornerRadius = UDim.new(1, 0)

local loadingStroke = Instance.new("UIStroke", loadingFrame)
loadingStroke.Color = Color3.fromRGB(0, 170, 255)
loadingStroke.Thickness = 2

local arc = Instance.new("Frame")
arc.Size = UDim2.new(1, 0, 1, 0)
arc.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
arc.AnchorPoint = Vector2.new(0.5, 0.5)
arc.Position = UDim2.new(0.5, 0, 0.5, 0)
arc.Parent = loadingFrame

local arcCorner = Instance.new("UICorner", arc)
arcCorner.CornerRadius = UDim.new(1, 0)

local button = Instance.new("TextButton")
button.Size = UDim2.new(1, -20, 0, 50)
button.Position = UDim2.new(0, 10, 1, -60)
button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
button.Font = Enum.Font.GothamBold
button.TextSize = 22
button.TextColor3 = Color3.new(1,1,1)
button.Text = "Steal"
button.AutoButtonColor = false
button.Parent = frame

local buttonCorner = Instance.new("UICorner", button)
buttonCorner.CornerRadius = UDim.new(0, 12)

local buttonStroke = Instance.new("UIStroke", button)
buttonStroke.Color = Color3.fromRGB(255, 255, 255)
buttonStroke.Thickness = 1

button.MouseEnter:Connect(function()
	TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 200, 255)}):Play()
end)
button.MouseLeave:Connect(function()
	TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 170, 255)}):Play()
end)

local teleporting = false
local cancelRequested = false
local stayTimer = 0
local maxStayTime = 2
local maxAttempts = 30
local attempts = 0
local heartbeatConn

local function setProgress(progress)
	progress = math.clamp(progress, 0, 1)
	arc.Size = UDim2.new(progress, 0, 1, 0)
end

local function resetProgress()
	setProgress(0)
end

local function updateStatus(text)
	statusText.Text = text
end

local function disconnectHeartbeat()
	if heartbeatConn then
		heartbeatConn:Disconnect()
		heartbeatConn = nil
	end
end

local function teleportLoop()
	teleporting = true
	cancelRequested = false
	stayTimer = 0
	attempts = 0
	setProgress(0)
	updateStatus("Teleportando...")
	button.Text = "Cancel"
	button.BackgroundColor3 = Color3.fromRGB(255, 70, 70)

	-- Start teleport above
	hrp.CFrame = CFrame.new(spawnPos + Vector3.new(0, 5, 0))

	heartbeatConn = RunService.Heartbeat:Connect(function(dt)
		if cancelRequested then
			disconnectHeartbeat()
			teleporting = false
			resetProgress()
			updateStatus("Teleportação cancelada.")
			button.Text = "Steal"
			button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
			return
		end

		if not hrp or not hrp.Parent then
			disconnectHeartbeat()
			teleporting = false
			resetProgress()
			updateStatus("Personagem não encontrado.")
			button.Text = "Steal"
			button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
			return
		end

		local pos = hrp.Position
		local horizontalDist = (Vector3.new(pos.X, 0, pos.Z) - Vector3.new(spawnPos.X, 0, spawnPos.Z)).Magnitude
		local verticalDist = pos.Y - spawnPos.Y

		if horizontalDist <= 2 and math.abs(verticalDist) <= 2 then
			stayTimer += dt
			updateStatus(string.format("Parado %.1f/%.1f segundos", stayTimer, maxStayTime))
			setProgress(stayTimer / maxStayTime)
			if stayTimer >= maxStayTime then
				disconnectHeartbeat()
				teleporting = false
				updateStatus("Teleportação concluída!")
				button.Text = "Done!"
				button.BackgroundColor3 = Color3.fromRGB(80, 220, 80)
				task.wait(2)
				resetProgress()
				updateStatus("Clique em Steal para começar")
				button.Text = "Steal"
				button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
			end
		else
			stayTimer = 0
			if attempts < maxAttempts then
				-- Descendo pouco a pouco para evitar "teleporte brusco"
				local newY = math.max(pos.Y - 0.5, spawnPos.Y)
				hrp.CFrame = CFrame.new(Vector3.new(spawnPos.X, newY, spawnPos.Z))
				attempts += 1
				updateStatus(string.format("Descendo... tentativa %d/%d", attempts, maxAttempts))
				setProgress(attempts / maxAttempts)
			else
				disconnectHeartbeat()
				teleporting = false
				updateStatus("Falha ao teleportar.")
				button.Text = "Failed"
				button.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
				task.wait(2)
				resetProgress()
				updateStatus("Clique em Steal para começar")
				button.Text = "Steal"
				button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
			end
		end
	end)
end

button.MouseButton1Click:Connect(function()
	if teleporting then
		cancelRequested = true
	else
		teleportLoop()
	end
end)
