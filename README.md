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
					break
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

	-- Teleporta todos bem perto do jogador (um pouco na frente)
	for i, data in ipairs(savedPositions) do
		if data.root and data.root.Parent then
			-- Posição aleatória perto do player (entre 6 e 12 studs)
			local randomOffset = Vector3.new(
				math.random(-10, 10),
				2,
				math.random(-10, 10)
			)
			
			-- Garante que não fique muito longe nem muito em cima
			if randomOffset.Magnitude < 6 then
				randomOffset = randomOffset.Unit * 8
			end

			pcall(function()
				data.root.CFrame = CFrame.new(myRoot.Position + randomOffset)
			end)
		end
	end

	print("Trouxe", #monsters, "monstros perto de você! Eles estão livres.")

	local startTime = tick()
	local duration = 60

	-- Só puxa de volta se eles forem muito longe
	bringConnection = RunService.Heartbeat:Connect(function()
		if not isBringing or (tick() - startTime) > duration then
			if bringConnection then
				bringConnection:Disconnect()
				bringConnection = nil
			end
			isBringing = false

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

		for _, data in ipairs(savedPositions) do
			if data.root and data.root.Parent then
				local distance = (data.root.Position - myRoot.Position).Magnitude

				-- Só puxa se estiver muito longe
				if distance > 65 then
					local randomOffset = Vector3.new(
						math.random(-10, 10),
						2,
						math.random(-10, 10)
					)
					if randomOffset.Magnitude < 6 then
						randomOffset = randomOffset.Unit * 8
					end

					pcall(function()
						data.root.CFrame = CFrame.new(myRoot.Position + randomOffset)
					end)
				end
			end
		end
	end)
end

LocalPlayer.Chatted:Connect(function(msg)
	if msg:lower() == "/bring" then
		BringMonsters()
	end
end)

UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.KeyCode == Enum.KeyCode.B then
		BringMonsters()
	end
end)

print("Pronto. /bring ou B | Monstros teleportados perto de você")
