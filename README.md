local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

local spawnPos = hrp.Position + Vector3.new(0, 10, 0)

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "UltraSmoothFlyGui"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 80)
frame.Position = UDim2.new(0.5, -110, 0.85, 0)
frame.BackgroundColor3 = Color3.fromRGB(28, 28, 35)
frame.BorderSizePixel = 0
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.ClipsDescendants = true
frame.BackgroundTransparency = 0.15

local uicorner = Instance.new("UICorner", frame)
uicorner.CornerRadius = UDim.new(0, 16)

local shadow = Instance.new("ImageLabel", frame)
shadow.Image = "rbxassetid://1316045217"
shadow.Size = UDim2.new(1, 30, 1, 30)
shadow.Position = UDim2.new(0.5, -15, 0.5, -15)
shadow.AnchorPoint = Vector2.new(0.5, 0.5)
shadow.ImageTransparency = 0.75
shadow.BackgroundTransparency = 1
shadow.ZIndex = 0

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(1, 0, 0.6, 0)
button.BackgroundTransparency = 1
button.Text = "Steal"
button.Font = Enum.Font.GothamBold
button.TextSize = 26
button.TextColor3 = Color3.new(1, 1, 1)
button.ZIndex = 2
button.AutoButtonColor = true

local progressBarBack = Instance.new("Frame", frame)
progressBarBack.Size = UDim2.new(0.9, 0, 0.25, 0)
progressBarBack.Position = UDim2.new(0.05, 0, 0.7, 0)
progressBarBack.BackgroundColor3 = Color3.fromRGB(60, 60, 75)
progressBarBack.BorderSizePixel = 0
progressBarBack.ZIndex = 2
progressBarBack.ClipsDescendants = true
progressBarBack.Visible = false

local progressBar = Instance.new("Frame", progressBarBack)
progressBar.Size = UDim2.new(0, 0, 1, 0)
progressBar.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
progressBar.BorderSizePixel = 0
progressBar.ZIndex = 3

-- Aligns para voo suave
local a0 = Instance.new("Attachment", hrp)

local alignPos = Instance.new("AlignPosition")
alignPos.Attachment0 = a0
alignPos.Mode = Enum.PositionAlignmentMode.OneAttachment
alignPos.Responsiveness = 20
alignPos.MaxForce = 5e5
alignPos.RigidityEnabled = false
alignPos.Enabled = false
alignPos.Parent = hrp

local alignOri = Instance.new("AlignOrientation")
alignOri.Attachment0 = a0
alignOri.Mode = Enum.OrientationAlignmentMode.OneAttachment
alignOri.Responsiveness = 20
alignOri.MaxTorque = 5e5
alignOri.RigidityEnabled = false
alignOri.Enabled = false
alignOri.Parent = hrp

local flying = false
local cancel = false

local function tweenProgress(duration)
	progressBarBack.Visible = true
	progressBar.Size = UDim2.new(0, 0, 1, 0)
	
	local tween = TweenService:Create(progressBar, TweenInfo.new(duration), {Size = UDim2.new(1, 0, 1, 0)})
	tween:Play()
	tween.Completed:Wait()
	
	progressBarBack.Visible = false
	progressBar.Size = UDim2.new(0, 0, 1, 0)
end

button.MouseButton1Click:Connect(function()
	if flying then
		cancel = true
		button.Text = "Steal"
		return
	end

	flying = true
	cancel = false
	button.Text = "Cancel"
	alignPos.Position = spawnPos
	alignPos.Enabled = true
	alignOri.Enabled = true

	-- Move suavemente até posição + 10 studs no ar
	local arrived = false
	local timer = 0
	local lastTick = tick()

	-- Loop do voo
	local heartbeatConn
	heartbeatConn = RunService.Heartbeat:Connect(function(dt)
		if cancel then
			flying = false
			alignPos.Enabled = false
			alignOri.Enabled = false
			heartbeatConn:Disconnect()
			button.Text = "Steal"
			progressBarBack.Visible = false
			return
		end
		
		local dist = (hrp.Position - spawnPos).Magnitude
		if dist < 3 then
			timer += tick() - lastTick
			if timer >= 2 then
				arrived = true
				heartbeatConn:Disconnect()
			end
		else
			timer = 0
		end
		lastTick = tick()
	end)

	-- Espera o voo até chegar
	while not arrived and not cancel do
		task.wait()
	end

	if cancel then return end

	-- Congela no ar por 6 segundos, mostrando barra
	alignPos.Position = spawnPos + Vector3.new(0, 0.1, 0)
	tweenProgress(6)

	alignPos.Enabled = false
	alignOri.Enabled = false
	button.Text = "Steal"
	flying = false
end)
