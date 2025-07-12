local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local humanoid = char:WaitForChild("Humanoid")

local posMarcada = nil
local puxando = false
local noclip = false
local plano = nil
local puxaoForce = nil
local highlightAtivo = false
local highlightsLoop = nil

-- GUI
local gui = Instance.new("ScreenGui", plr.PlayerGui)
gui.Name = "GuiCompleta"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 190, 0, 410)
frame.Position = UDim2.new(0.05, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.Active = true
frame.Draggable = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

-- Criar botões
local function criarBotao(nome, ordem, func)
	local botao = Instance.new("TextButton", frame)
	botao.Size = UDim2.new(1, -20, 0, 30)
	botao.Position = UDim2.new(0, 10, 0, (ordem - 1) * 40 + 10)
	botao.Text = nome
	botao.Font = Enum.Font.GothamBold
	botao.TextSize = 14
	botao.TextColor3 = Color3.new(1,1,1)
	botao.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	Instance.new("UICorner", botao).CornerRadius = UDim.new(0, 6)
	botao.MouseButton1Click:Connect(func)
end

-- BOTÃO 1 - MARCAR POSIÇÃO
criarBotao("📍 Marcar Posição", 1, function()
	char = plr.Character or plr.CharacterAdded:Wait()
	hrp = char:WaitForChild("HumanoidRootPart")
	posMarcada = hrp.Position
end)

-- BOTÃO 2 - PUXAR / CANCELAR
criarBotao("🧵 Puxar/Cancelar", 2, function()
	if puxando then
		puxando = false
		if puxaoForce then puxaoForce:Destroy() end
		puxaoForce = nil
		humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
		return
	end
	if not posMarcada then return end
	puxando = true
	humanoid:ChangeState(Enum.HumanoidStateType.Physics)
	puxaoForce = Instance.new("BodyVelocity", hrp)
	puxaoForce.MaxForce = Vector3.new(1e5, 1e5, 1e5)
	coroutine.wrap(function()
		while puxando and (hrp.Position - posMarcada).Magnitude > 5 do
			local dir = (posMarcada - hrp.Position).Unit
			puxaoForce.Velocity = dir * 40
			wait(0.05)
		end
		if puxaoForce then puxaoForce:Destroy() end
		puxando = false
		humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
	end)()
end)

-- BOTÃO 3 - NOCLIP
criarBotao("🚪 Noclip", 3, function()
	noclip = not noclip
	if noclip then
		game:GetService("RunService").Stepped:Connect(function()
			if noclip and plr.Character then
				for _, v in pairs(plr.Character:GetDescendants()) do
					if v:IsA("BasePart") then
						v.CanCollide = false
					end
				end
			end
		end)
	end
end)

-- BOTÃO 4 - PLANÍCIE FLUTUANTE
criarBotao("☁️ Planície Toggle", 4, function()
	if plano and plano.Parent then
		plano:Destroy()
		plano = nil
		return
	end
	local y = hrp.Position.Y - 6
	plano = Instance.new("Part", workspace)
	plano.Anchored = true
	plano.CanCollide = true
	plano.Size = Vector3.new(5000, 1, 5000)
	plano.Position = Vector3.new(hrp.Position.X, y, hrp.Position.Z)
	plano.Transparency = 0.4
	plano.Color = Color3.fromRGB(160,160,160)
	plano.Material = Enum.Material.SmoothPlastic
	plano.Name = "PlanícieFlutuante"
end)

-- BOTÃO 5 - TOOL DELETAR MUNDO
criarBotao("🗑️ Tool Deletar Mundo", 5, function()
	local tool = Instance.new("Tool")
	tool.RequiresHandle = false
	tool.Name = "Deletar Mundo"
	tool.Parent = plr.Backpack
	local mouse = plr:GetMouse()
	tool.Activated:Connect(function()
		if mouse and mouse.Target then
			local alvo = mouse.Target
			if alvo:IsA("BasePart") and alvo:IsDescendantOf(workspace) then
				alvo.Anchored = false
				alvo.CanCollide = false
				alvo:BreakJoints()
				wait(0.05)
				alvo:Destroy()
			end
		end
	end)
end)

-- BOTÃO 6 - DESTACAR PLAYERS (CORRIGIDO)
criarBotao("👥 Destacar Players", 6, function()
	highlightAtivo = not highlightAtivo
	if not highlightAtivo and highlightsLoop then
		highlightsLoop:Disconnect()
		highlightsLoop = nil
	end

	-- Loop constante para manter highlights mesmo após respawn
	if highlightAtivo then
		highlightsLoop = game:GetService("RunService").RenderStepped:Connect(function()
			for _, p in pairs(game.Players:GetPlayers()) do
				if p ~= plr and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
					if not p.Character:FindFirstChild("HighlightESP") then
						local hl = Instance.new("Highlight")
						hl.Name = "HighlightESP"
						hl.FillColor = Color3.fromRGB(255, 0, 0)
						hl.OutlineColor = Color3.new(1, 1, 1)
						hl.FillTransparency = 0.4
						hl.OutlineTransparency = 0
						hl.Parent = p.Character
					end
				elseif p.Character and p.Character:FindFirstChild("HighlightESP") and not highlightAtivo then
					p.Character.HighlightESP:Destroy()
				end
			end
		end)
	else
		for _, p in pairs(game.Players:GetPlayers()) do
			if p.Character and p.Character:FindFirstChild("HighlightESP") then
				p.Character.HighlightESP:Destroy()
			end
		end
	end
end)

-- BOTÃO 7 - REMOVER PROTEÇÕES VERMELHAS
criarBotao("🚫 Remover Barreiras", 7, function()
	for _, obj in pairs(workspace:GetDescendants()) do
		if obj:IsA("BasePart") and obj.Color == Color3.fromRGB(255, 0, 0) then
			obj:Destroy()
		end
	end
end)

-- BOTÃO 8 - TELEPORTAR PARA POSIÇÃO MARCADA
criarBotao("📌 Teleportar Marcado", 8, function()
	if posMarcada then
		char = plr.Character or plr.CharacterAdded:Wait()
		hrp = char:WaitForChild("HumanoidRootPart")
		hrp.CFrame = CFrame.new(posMarcada + Vector3.new(0, 3, 0))
	end
end)
