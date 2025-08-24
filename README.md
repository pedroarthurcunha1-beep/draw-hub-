-- PD Hub (educacional, para uso em jogos próprios no Roblox Studio)
-- Coloque este LocalScript dentro do Frame do seu ScreenGui (StarterGui/PD_Hub/Frame)

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

local frame = script.Parent
local titleBar = frame:FindFirstChildWhichIsA("TextLabel") or frame:WaitForChild("Title")
local btnPlus = frame:FindFirstChild("PlusSpeed") or frame:WaitForChild("PlusSpeed")
local btnMinus = frame:FindFirstChild("MinusSpeed") or frame:WaitForChild("MinusSpeed")
local btnReset = frame:FindFirstChild("ResetSpeed") or frame:WaitForChild("ResetSpeed")

-- Configurações
local MIN_SPEED = 8
local MAX_SPEED = 24
local STEP = 2
local DEFAULT_SPEED = 16

-- Função util para obter o Humanoid do jogador local
local function getHumanoid()
	local char = player.Character or player.CharacterAdded:Wait()
	return char:WaitForChild("Humanoid")
end

-- Garantir velocidade padrão quando o personagem spawnar
player.CharacterAdded:Connect(function(char)
	local hum = char:WaitForChild("Humanoid")
	hum.WalkSpeed = DEFAULT_SPEED
end)

-- Inicializa
task.defer(function()
	local hum = getHumanoid()
	hum.WalkSpeed = DEFAULT_SPEED
end)

-- Controles de velocidade
local function clampSpeed(s)
	return math.clamp(s, MIN_SPEED, MAX_SPEED)
end

btnPlus.MouseButton1Click:Connect(function()
	local hum = getHumanoid()
	hum.WalkSpeed = clampSpeed(hum.WalkSpeed + STEP)
end)

btnMinus.MouseButton1Click:Connect(function()
	local hum = getHumanoid()
	hum.WalkSpeed = clampSpeed(hum.WalkSpeed - STEP)
end)

btnReset.MouseButton1Click:Connect(function()
	local hum = getHumanoid()
	hum.WalkSpeed = DEFAULT_SPEED
end)

-- Tornar a janela arrastável (sem usar a propriedade Draggable, que é obsoleta)
local dragging = false
local dragStart, startPos

local function updateDrag(input)
	local delta = input.Position - dragStart
	frame.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)
end

titleBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or
	   input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

UIS.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or
	                 input.UserInputType == Enum.UserInputType.Touch) then
		updateDrag(input)
	end
end)

-- Opcional: mostrar a velocidade atual no título
local function updateTitle()
	local hum = getHumanoid()
	if titleBar and titleBar:IsA("TextLabel") then
		titleBar.Text = string.format("PD Hub — WalkSpeed: %d", hum.WalkSpeed)
	end
end

btnPlus.MouseButton1Click:Connect(updateTitle)
btnMinus.MouseButton1Click:Connect(updateTitle)
btnReset.MouseButton1Click:Connect(updateTitle)
RunService.Heartbeat:Connect(function()
	-- Mantém o texto sincronizado mesmo após respawn
	updateTitle()
end)
