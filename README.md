if not game:IsLoaded() then game.Loaded:Wait() end

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local playerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Evento remoto
local Remote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("Effects"):WaitForChild("BellyFlopFX")

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BellyFlopDual_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = playerGui

-- Frame para segurar os 2 botões grudados
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 100, 0, 25)
Frame.Position = UDim2.new(0.7, 0, 0.8, 0)
Frame.BackgroundTransparency = 1
Frame.Parent = ScreenGui

-- Botão esquerdo (em você)
local ButtonSelf = Instance.new("TextButton")
ButtonSelf.Size = UDim2.new(0, 50, 0, 25)
ButtonSelf.Position = UDim2.new(0, 0, 0, 0)
ButtonSelf.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
ButtonSelf.TextColor3 = Color3.new(1, 1, 1)
ButtonSelf.TextScaled = true
ButtonSelf.Font = Enum.Font.SourceSansBold
ButtonSelf.Text = "SELF: OFF"
ButtonSelf.Parent = Frame

-- Botão direito (em todos)
local ButtonAll = Instance.new("TextButton")
ButtonAll.Size = UDim2.new(0, 50, 0, 25)
ButtonAll.Position = UDim2.new(0, 50, 0, 0)
ButtonAll.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
ButtonAll.TextColor3 = Color3.new(1, 1, 1)
ButtonAll.TextScaled = true
ButtonAll.Font = Enum.Font.SourceSansBold
ButtonAll.Text = "ALL: OFF"
ButtonAll.Parent = Frame

-- Função arrastável
local dragging = false
local dragInput, dragStart, startPos

local function update(input)
	local delta = input.Position - dragStart
	Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

local function dragBegin(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = Frame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end

local function dragChanged(input)
	if input == dragInput and dragging then
		update(input)
	end
end

-- Conectar arrasto ao frame e aos botões
for _, obj in pairs({Frame, ButtonSelf, ButtonAll}) do
	obj.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragBegin(input)
		end
	end)
	obj.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)
end

UIS.InputChanged:Connect(dragChanged)

-- Variáveis de loop
local ativoSelf = false
local ativoAll = false

-- Botão esquerdo (em você)
ButtonSelf.MouseButton1Click:Connect(function()
	ativoSelf = not ativoSelf
	if ativoSelf then
		ButtonSelf.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
		ButtonSelf.Text = "SELF: ON"
	else
		ButtonSelf.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
		ButtonSelf.Text = "SELF: OFF"
	end
end)

-- Botão direito (em todos)
ButtonAll.MouseButton1Click:Connect(function()
	ativoAll = not ativoAll
	if ativoAll then
		ButtonAll.BackgroundColor3 = Color3.fromRGB(0, 0, 255)
		ButtonAll.Text = "ALL: ON"
	else
		ButtonAll.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
		ButtonAll.Text = "ALL: OFF"
	end
end)

-- Função para spam 1000 vezes
local function SpamRemote(target)
    pcall(function()
        for i = 1, 1000 do
            Remote:FireServer("E", target)
        end
    end)
end

-- Loop infinito
RunService.Heartbeat:Connect(function()
	if ativoSelf then
		local target = workspace:FindFirstChild(LocalPlayer.Name)
		if target then
			SpamRemote(target)
		end
	end
	
	if ativoAll then
		for _, player in pairs(Players:GetPlayers()) do
			if player ~= LocalPlayer then
				local target = workspace:FindFirstChild(player.Name)
				if target then
					SpamRemote(target)
				end
			end
		end
	end
end)
