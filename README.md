local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

-- Posição de onde nasceu
local spawnPos = hrp.Position

-- GUI moderna
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "TPGui"
gui.ResetOnSpawn = false

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 160, 0, 50)
button.Position = UDim2.new(0.5, -80, 0.7, 0)
button.AnchorPoint = Vector2.new(0.5, 0.5)
button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.Font = Enum.Font.GothamBold
button.TextSize = 20
button.Text = "Steal"
button.Parent = gui

-- Estados
local flying = false
local connection = nil
local bodyVelocity = nil

-- Função que inicia voo suave até o ponto
local function startFlyingTo(targetPos)
	if bodyVelocity then bodyVelocity:Destroy() end

	bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
	bodyVelocity.P = 15000
	bodyVelocity.Velocity = Vector3.zero
	bodyVelocity.Parent = hrp

	local timeInPlace = 0

	connection = RunService.Heartbeat:Connect(function(dt)
		if not flying then return end

		local currentPos = hrp.Position
		local distance = (targetPos - currentPos)

		-- Aponta suavemente na direção e reduz velocidade ao chegar
		bodyVelocity.Velocity = distance.Unit * math.clamp(distance.Magnitude * 1.5, 5, 50)

		-- Detecta se chegou e ficou por 2 segundos
		if distance.Magnitude <= 2 then
			timeInPlace += dt
			if timeInPlace >= 2 then
				flying = false
				button.Text = "Steal"
				if connection then connection:Disconnect() end
				if bodyVelocity then bodyVelocity:Destroy() end
			end
		else
			timeInPlace = 0
		end
	end)
end

-- Botão
button.MouseButton1Click:Connect(function()
	if flying then
		flying = false
		button.Text = "Steal"
		if connection then connection:Disconnect() end
		if bodyVelocity then bodyVelocity:Destroy() end
	else
		flying = true
		button.Text = "Cancel"
		startFlyingTo(spawnPos + Vector3.new(0, 5, 0)) -- Vai 5 studs acima e desce suavemente
	end
end)
