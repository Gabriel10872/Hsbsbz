local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humRoot = char:WaitForChild("HumanoidRootPart")
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 130, 0, 40)
button.Position = UDim2.new(0.5, -65, 0.6, 0)
button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.Font = Enum.Font.GothamSemibold
button.TextSize = 18
button.Text = "Steal"
button.Parent = gui

local isTrying = false
local startPos = humRoot.Position
local stayTimer = 0
local heartbeatConnection

local function tryTeleport()
	isTrying = true
	button.Text = "Cancel"
	stayTimer = 0

	local maxTries = 30
	local tries = 0

	heartbeatConnection = RunService.Heartbeat:Connect(function(dt)
		if not isTrying then return end

		local pos = humRoot.Position
		local dist = (Vector3.new(pos.X, 0, pos.Z) - Vector3.new(startPos.X, 0, startPos.Z)).Magnitude

		if dist < 2 and math.abs(pos.Y - startPos.Y) < 2 then
			stayTimer += dt
			if stayTimer >= 2 then
				isTrying = false
				button.Text = "Steal"
				heartbeatConnection:Disconnect()
			end
		else
			stayTimer = 0
			if tries < maxTries then
				-- sobe e tenta descer
				humRoot.CFrame = CFrame.new(startPos + Vector3.new(0, 5, 0))
				tries += 1
			else
				isTrying = false
				button.Text = "Steal"
				heartbeatConnection:Disconnect()
			end
		end
	end)
end

button.MouseButton1Click:Connect(function()
	if isTrying then
		isTrying = false
		button.Text = "Steal"
		if heartbeatConnection then
			heartbeatConnection:Disconnect()
		end
	else
		tryTeleport()
	end
end)
