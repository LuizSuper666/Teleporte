-- LocalScript (StarterPlayerScripts)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Variáveis
local character = player.Character or player.CharacterAdded:Wait()
local armazenadoPos = nil
local tpConfusaoAtivo = false
local tpClickAtivo = false

-- Atualizar personagem sempre que respawnar
player.CharacterAdded:Connect(function(char)
	character = char
end)

-- Criação dos elementos visuais
local function criarBotao(pai, texto)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 120, 0, 40)
	btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 16
	btn.Text = texto
	btn.AutoButtonColor = true
	btn.BackgroundTransparency = 0.1
	btn.BorderSizePixel = 0

	local uiCorner = Instance.new("UICorner")
	uiCorner.CornerRadius = UDim.new(0, 12)
	uiCorner.Parent = btn

	btn.Parent = pai
	return btn
end

-- Menu principal
local mainGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
mainGui.Name = "TPMenuGui"
mainGui.ResetOnSpawn = false

local menuPrincipal = Instance.new("Frame", mainGui)
menuPrincipal.Size = UDim2.new(0, 400, 0, 100)
menuPrincipal.Position = UDim2.new(0.5, -200, 0.4, 0)
menuPrincipal.BackgroundTransparency = 0.3
menuPrincipal.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
local corner = Instance.new("UICorner", menuPrincipal)
corner.CornerRadius = UDim.new(0, 15)

local layout = Instance.new("UIListLayout", menuPrincipal)
layout.FillDirection = Enum.FillDirection.Horizontal
layout.Padding = UDim.new(0, 10)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.VerticalAlignment = Enum.VerticalAlignment.Center

-- Submenus
local function criarMenuMovel(nome, tamanho)
	local frame = Instance.new("Frame", mainGui)
	frame.Name = nome
	frame.Size = UDim2.new(0, tamanho.X, 0, tamanho.Y)
	frame.Position = UDim2.new(0.5, -tamanho.X / 2, 0.6, 0)
	frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	frame.BackgroundTransparency = 0.3
	frame.Visible = false

	local drag = false
	local dragInput, dragStart, startPos

	local function update(input)
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end

	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			drag = true
			dragStart = input.Position
			startPos = frame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					drag = false
				end
			end)
		end
	end)

	frame.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and drag then
			update(input)
		end
	end)

	local corner = Instance.new("UICorner", frame)
	corner.CornerRadius = UDim.new(0, 12)

	return frame
end

-- Submenu 1 - Armazenar / TP
local menu1 = criarMenuMovel("MenuArmazenado", Vector2.new(200, 100))

local btnArmazenar = criarBotao(menu1, "Armazenar")
btnArmazenar.Position = UDim2.new(0, 10, 0, 10)
btnArmazenar.MouseButton1Click:Connect(function()
	if character and character:FindFirstChild("HumanoidRootPart") then
		armazenadoPos = character.HumanoidRootPart.Position
	end
end)

local btnTP = criarBotao(menu1, "TP")
btnTP.Position = UDim2.new(0, 10, 0, 60)
btnTP.MouseButton1Click:Connect(function()
	if armazenadoPos and character and character:FindFirstChild("HumanoidRootPart") then
		character.HumanoidRootPart.CFrame = CFrame.new(armazenadoPos)
	end
end)

-- Submenu 2 - Confusão
local menu2 = criarMenuMovel("MenuConfusao", Vector2.new(200, 60))
local btnConfusao = criarBotao(menu2, "Ativar Confusão")
btnConfusao.Position = UDim2.new(0, 10, 0, 10)

local pontos = {
	Vector3.new(24, 0, 0),
	Vector3.new(-24, 0, 0),
	Vector3.new(0, 0, 24),
	Vector3.new(0, 0, -24),
}

btnConfusao.MouseButton1Click:Connect(function()
	tpConfusaoAtivo = not tpConfusaoAtivo
	btnConfusao.Text = tpConfusaoAtivo and "Desativar Confusão" or "Ativar Confusão"

	if tpConfusaoAtivo then
		spawn(function()
			while tpConfusaoAtivo do
				for _, offset in ipairs(pontos) do
					if character and character:FindFirstChild("HumanoidRootPart") then
						local basePos = character.HumanoidRootPart.Position
						character.HumanoidRootPart.CFrame = CFrame.new(basePos + offset)
						wait(0.1)
					end
				end
			end
		end)
	end
end)

-- Submenu 3 - TP por clique
local menu3 = criarMenuMovel("MenuClick", Vector2.new(200, 60))
local btnClick = criarBotao(menu3, "Ativar TP Click")
btnClick.Position = UDim2.new(0, 10, 0, 10)

btnClick.MouseButton1Click:Connect(function()
	tpClickAtivo = not tpClickAtivo
	btnClick.Text = tpClickAtivo and "Desativar TP Click" or "Ativar TP Click"
end)

mouse.Button1Down:Connect(function()
	if tpClickAtivo then
		local ray = workspace:Raycast(mouse.Origin.Position, mouse.Hit.Position - mouse.Origin.Position)
		if ray and ray.Position then
			if character and character:FindFirstChild("HumanoidRootPart") then
				character.HumanoidRootPart.CFrame = CFrame.new(ray.Position + Vector3.new(0, 3, 0))
			end
		end
	end
end)

-- Botões principais
local btn1 = criarBotao(menuPrincipal, "TP Armazenado")
btn1.MouseButton1Click:Connect(function()
	menuPrincipal.Visible = false
	menu1.Visible = true
end)

local btn2 = criarBotao(menuPrincipal, "TP Confusão")
btn2.MouseButton1Click:Connect(function()
	menuPrincipal.Visible = false
	menu2.Visible = true
end)

local btn3 = criarBotao(menuPrincipal, "TP Click")
btn3.MouseButton1Click:Connect(function()
	menuPrincipal.Visible = false
	menu3.Visible = true
end)
