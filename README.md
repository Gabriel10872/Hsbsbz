local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local spawnPos = hrp.Position

-- GUI Moderna
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "SmoothFlyGUI"

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 170, 0, 55)
button.Position = UDim2.new(0.5, -85, 0.8, 0)
button.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
button.TextColor3 = Color3.new(1, 1, 1)
button.TextSize = 22
button.Font = Enum.Font.GothamBold
button.Text = "Steal"
button.AutoButtonColor = true
button.Parent = gui

-- Função de voo ultra suave
local flying = false
local connection
local bv, bg

local function startSmoothFlyTo(pos)
	if bv then bv:Destroy() end
	if bg then bg:Destroy() end

	bv = Instance.new("BodyVelocity")
	bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
	bv.Velocity = Vector3.zero
	bv.P = 3000
	bv.Parent = hrp

	bg = Instance.new("BodyGyro")
	bg.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
	bg.CFrame = hrp.CFrame
	bg.P = 3000
	bg.Parent = hrp

	local timeStill = 0

	connection = RunService.Heartbeat:Connect(function(dt)
		if not flying then return end

		local target = pos + Vector3.new(0, 5, 0) -- Fica acima e desce
		local direction = (target - hrp.Position)
		local distance = direction.Magnitude

		-- Suavização da direção
		if distance > 1 then
			local speed = math.clamp(distance * 2, 10, 60)
			bv.Velocity = direction.Unit * speed
			bg.CFrame = CFrame.new(hrp.Position, target)
			timeStill = 0
		else
			timeStill += dt
			bv.Velocity = Vector3.zero
			if timeStill >= 2 then
				button.Text = "Steal"
				flying = false
				if connection then connection:Disconnect() end
				if bv then bv:Destroy() end
				if bg then bg:Destroy() end
			end
		end
	end)
end

-- Botão
button.MouseButton1Click:Connect(function()
	if flying then
		flying = false
		button.Text = "Steal"
		if connection then connection:Disconnect() end
		if bv then bv:Destroy() end
		if bg then bg:Destroy() end
	else
		flying = true
		button.Text = "Cancel"
		startSmoothFlyTo(spawnPos)
	end
end)
