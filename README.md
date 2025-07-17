local Players = game:GetService("Players")
local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

-- Salvar posição inicial levemente acima
local originalPos = hrp.Position + Vector3.new(0, 10, 0)

-- Criar GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "StealGui"

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 160, 0, 50)
button.Position = UDim2.new(0.5, -80, 0.8, 0)
button.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 20
button.Text = "Steal"
button.Parent = gui

-- Criação de attachments e aligners
local a0 = Instance.new("Attachment", hrp)

local alignPos = Instance.new("AlignPosition")
alignPos.Attachment0 = a0
alignPos.Mode = Enum.PositionAlignmentMode.OneAttachment
alignPos.Responsiveness = 200
alignPos.MaxForce = 500000
alignPos.RigidityEnabled = false
alignPos.Enabled = false
alignPos.Parent = hrp

local alignOri = Instance.new("AlignOrientation")
alignOri.Attachment0 = a0
alignOri.Mode = Enum.OrientationAlignmentMode.OneAttachment
alignOri.Responsiveness = 50
alignOri.MaxTorque = 500000
alignOri.RigidityEnabled = false
alignOri.Enabled = false
alignOri.Parent = hrp

-- Função de voo até posição
local flying = false
local cancel = false

button.MouseButton1Click:Connect(function()
	if not flying then
		button.Text = "Cancel"
		alignPos.Position = originalPos
		alignPos.Enabled = true
		alignOri.Enabled = true
		flying = true
		cancel = false

		local stayTime = 0
		local lastCheck = tick()

		while flying and not cancel do
			wait(0.1)
			local dist = (hrp.Position - originalPos).Magnitude
			if dist < 3 then
				stayTime += tick() - lastCheck
			else
				stayTime = 0
			end
			lastCheck = tick()

			if stayTime >= 2 then
				break
			end
		end

		alignPos.Enabled = false
		alignOri.Enabled = false
		flying = false
		button.Text = "Steal"
	else
		cancel = true
		button.Text = "Steal"
	end
end)
