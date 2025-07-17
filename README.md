local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

local savedPosition = HumanoidRootPart.Position
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "TeleportStealGUI"
gui.ResetOnSpawn = false

-- Main Frame
local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 200, 0, 100)
main.Position = UDim2.new(0.5, -100, 0.4, 0)
main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true

-- UI Corner
local corner = Instance.new("UICorner", main)
corner.CornerRadius = UDim.new(0, 12)

-- UI Stroke
local stroke = Instance.new("UIStroke", main)
stroke.Color = Color3.fromRGB(0, 200, 255)
stroke.Thickness = 2

-- Title Label
local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "Steal Teleport"
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Button
local button = Instance.new("TextButton", main)
button.Size = UDim2.new(0.8, 0, 0, 35)
button.Position = UDim2.new(0.1, 0, 0.5, 0)
button.Text = "Steal"
button.Font = Enum.Font.Gotham
button.TextSize = 16
button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.AutoButtonColor = true

local buttonCorner = Instance.new("UICorner", button)
buttonCorner.CornerRadius = UDim.new(0, 8)

-- Lógica de teleporte
local function tentarTeleportar()
	local tentativas = 0
	local maxTentativas = 30
	local tempoFixo = 2 -- segundos

	task.spawn(function()
		while tentativas < maxTentativas do
			if not LocalPlayer.Character then break end
			local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				hrp.CFrame = CFrame.new(savedPosition)
			end

			tentativas += 1
			local tempoNaPosicao = 0

			while hrp and (hrp.Position - savedPosition).Magnitude < 5 do
				task.wait(0.2)
				tempoNaPosicao += 0.2
				if tempoNaPosicao >= tempoFixo then
					print("✅ Sucesso: ficou na posição por 2s.")
					return
				end
			end

			task.wait(0.1)
		end

		print("❌ Falhou: não conseguiu permanecer 2s na posição.")
	end)
end

-- Ação ao clicar no botão
button.MouseButton1Click:Connect(function()
	tentarTeleportar()
end)
