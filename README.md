local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- Criar GUI persistente
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BlackUpWalkerUI"
screenGui.ResetOnSpawn = false -- não some ao morrer
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Função para criar botão arrastável
local function criarBotao(nome, texto, posX, cor)
	local btn = Instance.new("TextButton")
	btn.Name = nome
	btn.Text = texto
	btn.Size = UDim2.new(0, 100, 0, 100)
	btn.Position = UDim2.new(0, posX, 0.7, 0)
	btn.BackgroundColor3 = cor
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.GothamBold
	btn.TextScaled = true
	btn.Active = true
	btn.Draggable = true -- permite arrastar
	btn.Parent = screenGui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(1, 0)
	corner.Parent = btn

	return btn
end

-- Criar botões
local upButton = criarBotao("UpWalke", "UpWalke", 50, Color3.fromRGB(0, 200, 0))
local stopButton = criarBotao("StopWalke", "Parar", 170, Color3.fromRGB(200, 0, 0))

-- Variáveis
local barrier = nil
local active = false
local followConnection = nil

-- Função: Ativar plataforma
local function ativarPlataforma()
	if active then return end
	active = true

	-- Criar barreira
	barrier = Instance.new("Part")
	barrier.Size = Vector3.new(8, 1, 8)
	barrier.Color = Color3.fromRGB(0, 255, 0)
	barrier.Anchored = true
	barrier.CanCollide = true
	barrier.Transparency = 0.3
	barrier.Name = "UpWalkerBarrier"
	barrier.Parent = workspace

	-- Seguir jogador
	followConnection = RunService.Heartbeat:Connect(function()
		if active and player.Character and barrier then
			local hrp = player.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				barrier.CFrame = CFrame.new(hrp.Position.X, hrp.Position.Y - 3, hrp.Position.Z)
			end
		end
	end)
end

-- Função: Parar plataforma e segurar no ar
local function pararPlataforma()
	if not active or not barrier then return end
	active = false
	if followConnection then
		followConnection:Disconnect()
		followConnection = nil
	end
	-- Congela a plataforma no ponto atual
	barrier.Anchored = true
	-- Garantir que o jogador fique em cima e não caia
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		local hrp = player.Character.HumanoidRootPart
		hrp.CFrame = CFrame.new(barrier.Position + Vector3.new(0, 3, 0))
		hrp.Velocity = Vector3.zero
	end
end

-- Conectar botões
upButton.MouseButton1Click:Connect(ativarPlataforma)
stopButton.MouseButton1Click:Connect(pararPlataforma)
