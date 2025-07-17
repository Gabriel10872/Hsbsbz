local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

-- SALVA posição de spawn corretamente logo no início
local spawnPos = hrp.Position

-- Criação do GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false
gui.Name = "TPGui"

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 160, 0, 50)
button.Position = UDim2.new(0.5, -80, 0.7, 0)
button.AnchorPoint = Vector2.new(0.5, 0.5)
button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.Font = Enum.Font.GothamBold
button.TextSize = 20
button.Text = "Steal"
button.Parent = gui
button.ZIndex = 2
button.AutoButtonColor = false

local trying = false
local stayingTime = 0
local attempts = 0
local maxTries = 30
local connection

local function tryTeleport()
	if connection then connection:Disconnect() end
	attempts = 0
	stayingTime = 0

	connection = RunService.Heartbeat:Connect(function(dt)
		if not trying then return end

		local currentPos = hrp.Position
		local horizontal = (Vector3.new(currentPos.X, 0, currentPos.Z) - Vector3.new(spawnPos.X, 0, spawnPos.Z)).Magnitude
		local vertical = math.abs(currentPos.Y - spawnPos.Y)

		if horizontal <= 2 and vertical <= 2 then
			stayingTime += dt
			if stayingTime >= 2 then
				trying = false
				button.Text = "Steal"
				connection:Disconnect()
			end
		else
			stayingTime = 0
			if attempts < maxTries then
				hrp.CFrame = CFrame.new(spawnPos + Vector3.new(0, 5, 0))
				attempts += 1
			else
				trying = false
				button.Text = "Steal"
				connection:Disconnect()
			end
		end
	end)
end

button.MouseButton1Click:Connect(function()
	if not trying then
		trying = true
		button.Text = "Cancel"
		tryTeleport()
	else
		trying = false
		button.Text = "Steal"
		if connection then connection:Disconnect() end
	end
end)
