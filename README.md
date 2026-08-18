local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

local MonsterNames = {
	"Kus", "Gus", "Lurker", "Sandman", "Ashy",
	"Dus", "Bloodman", "Lost", "Scar", "Blud", "Dlud"
}

local isBringing = false
local bringConnection = nil
local savedPositions = {}

local function GetRoot(model)
	return model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart
end

local function FindMonsters()
	local found = {}
	
	for _, name in ipairs(MonsterNames) do
		for _, obj in ipairs(Workspace:GetDescendants()) do
			if obj:IsA("Model") and obj.Name:lower():find(name:lower()) then
				local root = GetRoot(obj)
				local humanoid = obj:FindFirstChildOfClass("Humanoid")
				
				if root and humanoid and humanoid.Health > 0 then
					table.insert(found, {
						model = obj,
						root = root,
						originalCFrame = root.CFrame
					})
					break -- pega só 1 de cada tipo (igual o original)
				end
			end
		end
	end
	
	return found
end

local function BringMonsters()
	if isBringing then
		print("Já está trazendo monstros. Aguarde.")
		return
	end
	
	local char = LocalPlayer.Character
	if not char then return end
	local myRoot = char:FindFirstChild("HumanoidRootPart")
	if not myRoot then return end

	local monsters = FindMonsters()
	
	if #monsters == 0 then
		print("Nenhum monstro encontrado.")
		return
	end

	isBringing = true
	savedPositions = monsters

	print("Trouxe", #monsters, "monstros. Segurando por 60 segundos...")

	local startTime = tick()
	local duration = 60 -- tempo em segundos

	-- Loop contínuo que força a posição todo frame
	bringConnection = RunService.Heartbeat:Connect(function()
		if not isBringing or (tick() - startTime) > duration then
			if bringConnection then
				bringConnection:Disconnect()
				bringConnection = nil
			end
			isBringing = false

			-- Devolve pros lugares originais
			for _, data in ipairs(savedPositions) do
				pcall(function()
					if data.root and data.root.Parent then
						data.root.CFrame = data.originalCFrame
					end
				end)
			end

			print("Monstros devolvidos.")
			return
		end

		local char = LocalPlayer.Character
		if not char then return end
		local myRoot = char:FindFirstChild("HumanoidRootPart")
		if not myRoot then return end

		local angle = 0
		for _, data in ipairs(savedPositions) do
			if data.root and data.root.Parent then
				local offset = Vector3.new(
					math.cos(math.rad(angle)) * 12,
					3,
					math.sin(math.rad(angle)) * 12
				)

				pcall(function()
					-- Força a posição + zera velocidade (ajuda a não voar)
					data.root.CFrame = CFrame.new(myRoot.Position + offset)
					data.root.AssemblyLinearVelocity = Vector3.zero
					data.root.AssemblyAngularVelocity = Vector3.zero
				end)

				angle = angle + 32
			end
		end
	end)
end

-- Comando no chat
LocalPlayer.Chatted:Connect(function(msg)
	if msg:lower() == "/bring" then
		BringMonsters()
	end
end)

-- Tecla B
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.KeyCode == Enum.KeyCode.B then
		BringMonsters()
	end
end)

print("Pronto. /bring ou B | Monstros agora são segurados por 60s")
