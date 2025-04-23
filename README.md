local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

local storedPosition = nil
local isConfusionActive = false
local isClickTPActive = false

-- UI PRINCIPAL
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false

local mainMenu = Instance.new("Frame", gui)
mainMenu.Size = UDim2.new(0, 300, 0, 60)
mainMenu.Position = UDim2.new(0.5, -150, 0.85, 0)
mainMenu.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainMenu.BackgroundTransparency = 0.2
mainMenu.BorderSizePixel = 0
mainMenu.AnchorPoint = Vector2.new(0.5, 0.5)
mainMenu.Active = true
mainMenu.Draggable = true

-- Função para criar botões
local function criarBotao(texto, posX)
	local botao = Instance.new("TextButton", mainMenu)
	botao.Size = UDim2.new(0.3, 0, 1, 0)
	botao.Position = UDim2.new(posX, 0, 0, 0)
	botao.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
	botao.TextColor3 = Color3.new(1, 1, 1)
	botao.Font = Enum.Font.GothamBold
	botao.TextScaled = true
	botao.BorderSizePixel = 0
	botao.AutoButtonColor = true
	botao.Text = texto
	botao.Name = texto
	botao.ClipsDescendants = true
	botao.UICorner = Instance.new("UICorner", botao)
	return botao
end

-- Função para criar submenus
local function criarSubMenu()
	local f = Instance.new("Frame", gui)
	f.Size = UDim2.new(0, 200, 0, 80)
	f.Position = UDim2.new(0.5, 0, 0.7, 0)
	f.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	f.BorderSizePixel = 0
	f.Visible = false
	f.Active = true
	f.Draggable = true
	Instance.new("UICorner", f)
	return f
end

-- Submenu TP Armazenado
local armazenadoMenu = criarSubMenu()

local btnArmazenar = criarBotao("Armazenar", 0.05)
btnArmazenar.Parent = armazenadoMenu
btnArmazenar.Size = UDim2.new(0.45, 0, 0.6, 0)
btnArmazenar.Position = UDim2.new(0.05, 0, 0.2, 0)

btnArmazenar.MouseButton1Click:Connect(function()
	local char = player.Character or player.CharacterAdded:Wait()
	local hrp = char:WaitForChild("HumanoidRootPart")
	storedPosition = hrp.Position
end)

local btnTP = criarBotao("TP", 0.5)
btnTP.Parent = armazenadoMenu
btnTP.Size = UDim2.new(0.45, 0, 0.6, 0)
btnTP.Position = UDim2.new(0.5, 0, 0.2, 0)

btnTP.MouseButton1Click:Connect(function()
	if storedPosition then
		local char = player.Character or player.CharacterAdded:Wait()
		local hrp = char:FindFirstChild("HumanoidRootPart")
		if hrp then
			hrp.CFrame = CFrame.new(storedPosition)
		end
	end
end)

-- Submenu TP Confusão
local confusaoMenu = criarSubMenu()

local confusaoBtn = criarBotao("Ativar", 0.25)
confusaoBtn.Parent = confusaoMenu
confusaoBtn.Size = UDim2.new(0.5, 0, 0.6, 0)
confusaoBtn.Position = UDim2.new(0.25, 0, 0.2, 0)

local pontos = {
	Vector3.new(5, 0, 0),
	Vector3.new(-5, 0, 0),
	Vector3.new(0, 0, 5),
	Vector3.new(0, 0, -5),
}

local function confusaoLoop()
	while isConfusionActive do
		local char = player.Character or player.CharacterAdded:Wait()
		local hrp = char:FindFirstChild("HumanoidRootPart")
		if not hrp then break end
		for _, offset in ipairs(pontos) do
			if not isConfusionActive then return end
			hrp.CFrame = hrp.CFrame + offset
			task.wait(0.15)
		end
	end
end

confusaoBtn.MouseButton1Click:Connect(function()
	isConfusionActive = not isConfusionActive
	confusaoBtn.Text = isConfusionActive and "Desativar" or "Ativar"
	if isConfusionActive then
		task.spawn(confusaoLoop)
	end
end)

-- Submenu TP Click
local clickMenu = criarSubMenu()

local clickBtn = criarBotao("Ativar", 0.25)
clickBtn.Parent = clickMenu
clickBtn.Size = UDim2.new(0.5, 0, 0.6, 0)
clickBtn.Position = UDim2.new(0.25, 0, 0.2, 0)

clickBtn.MouseButton1Click:Connect(function()
	isClickTPActive = not isClickTPActive
	clickBtn.Text = isClickTPActive and "Desativar" or "Ativar"
end)

UserInputService.InputBegan:Connect(function(input, processed)
	if isClickTPActive and input.UserInputType == Enum.UserInputType.Touch then
		local touch = input
		local pos = touch.Position
		local ray = workspace.CurrentCamera:ScreenPointToRay(pos.X, pos.Y)
		local raycast = workspace:Raycast(ray.Origin, ray.Direction * 1000)
		if raycast then
			local char = player.Character or player.CharacterAdded:Wait()
			local hrp = char:FindFirstChild("HumanoidRootPart")
			if hrp then
				hrp.CFrame = CFrame.new(raycast.Position + Vector3.new(0, 3, 0))
			end
		end
	end
end)

-- Menu principal
local btn1 = criarBotao("TP Armazenado", 0.02)
local btn2 = criarBotao("TP Confusão", 0.35)
local btn3 = criarBotao("TP Click", 0.68)

btn1.MouseButton1Click:Connect(function()
	mainMenu.Visible = false
	armazenadoMenu.Visible = true
end)

btn2.MouseButton1Click:Connect(function()
	mainMenu.Visible = false
	confusaoMenu.Visible = true
end)

btn3.MouseButton1Click:Connect(function()
	mainMenu.Visible = false
	clickMenu.Visible = true
end)
