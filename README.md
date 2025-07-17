local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

-- Posição inicial (ligeiramente acima)
local originalPos = hrp.Position + Vector3.new(0, 10, 0)

-- GUI moderna
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "StealGui"
gui.ZIndexBehavior = Enum.ZIndexBehavior.Global

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 60)
frame.Position = UDim2.new(0.5, -100, 0.85, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.1
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.ClipsDescendants = true

-- Cantos arredondados e sombra
local uicorner = Instance.new("UICorner", frame)
uicorner.CornerRadius = UDim.new(0, 12)

local shadow = Instance.new("ImageLabel", frame)
shadow.Image = "rbxassetid://1316045217"
shadow.Size = UDim2.new(1, 30, 1, 30)
shadow.Position = UDim2.new(0.5, -15, 0.5, -15)
shadow.AnchorPoint = Vector2.new(0.5, 0.5)
shadow.ImageTransparency = 0.7
shadow.BackgroundTransparency = 1
shadow.ZIndex = 0

-- Botão
local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, 0, 1, 0)
button.BackgroundTransparency = 1
button.Text = "Steal"
button.TextColor3 = Color3.new(1, 1, 1)
button.TextSize = 24
button.Font = Enum.Font.GothamBold
button.ZIndex = 2

-- Fly suave
local a0 = Instance.new("Attachment", hrp)

local alignPos = Instance.new("AlignPosition")
alignPos.Attachment0 = a0
alignPos.Mode = Enum.PositionAlignmentMode.OneAttachment
alignPos.Responsiveness = 100
alignPos.MaxForce = 1000000
alignPos.RigidityEnabled = false
alignPos.Enabled = false
alignPos.Parent = hrp

local alignOri = Instance.new("AlignOrientation")
alignOri.Attachment0 = a0
alignOri.Mode = Enum.OrientationAlignmentMode.OneAttachment
alignOri.Responsiveness = 100
alignOri.MaxTorque = 1000000
alignOri.RigidityEnabled = false
alignOri.Enabled = false
alignOri.Parent = hrp

-- Controle
local flying = false
local cancel = false

button.MouseButton1Click:Connect(function()
	if flying then
		cancel = true
		button.Text = "Steal"
		return
	end

	flying = true
	cancel = false
	button.Text = "Cancel"

	alignPos.Position = originalPos
	alignPos.Enabled = true
	alignOri.Enabled = true

	local arrived = false
	local timer = 0
	local lastCheck = tick()

	while flying and not cancel do
		task.wait(0.1)
		local dist = (hrp.Position - originalPos).Magnitude
		if dist < 3 then
			timer += tick() - lastCheck
			if timer >= 2 then
				arrived = true
				break
			end
		else
			timer = 0
		end
		lastCheck = tick()
	end

	if arrived and not cancel then
		-- Fica parado no ar por 6 segundos
		alignPos.Position = originalPos + Vector3.new(0, 0.1, 0)
		for i = 1, 60 do
			if cancel then break end
			task.wait(0.1)
		end
	end

	alignPos.Enabled = false
	alignOri.Enabled = false
	button.Text = "Steal"
	flying = false
end)
