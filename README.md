local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local MonsterNames = {
	"Kus", "Gus", "Lurker", "Sandman", "Ashy",
	"Dus", "Bloodman", "Lost", "Scar", "Blud", "Dlud"
}

local savedPositions = {}

local function GetRoot(model)
	return model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart
end

local function BringMonsters()
	local char = LocalPlayer.Character
	if not char then return end
	local myRoot = char:FindFirstChild("HumanoidRootPart")
	if not myRoot then return end

	savedPositions = {}
	local angle = 0
	local count = 0

	for _, name in ipairs(MonsterNames) do
		for _, obj in ipairs(Workspace:GetDescendants()) do
			if obj:IsA("Model") and obj.Name:lower():find(name:lower()) then
				local root = GetRoot(obj)
				if root and obj:FindFirstChildOfClass("Humanoid") then
					-- Salva posição original
					table.insert(savedPositions, {
						root = root,
						cframe = root.CFrame
					})

					-- Teleporta uma vez só (eles ficam livres depois)
					local offset = Vector3.new(
						math.cos(math.rad(angle)) * 12,
						3,
						math.sin(math.rad(angle)) * 12
					)
					pcall(function()
						root.CFrame = CFrame.new(myRoot.Position + offset)
					end)

					angle = angle + 32
					count = count + 1
					break
				end
			end
		end
	end

	print("Trouxe", count, "monstros. Eles estão livres por 60 segundos...")

	-- Espera 60 segundos sem travar eles
	task.wait(60)

	-- Devolve pros lugares originais
	for _, data in ipairs(savedPositions) do
		pcall(function()
			data.root.CFrame = data.cframe
		end)
	end

	print("Monstros devolvidos.")
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

print("Pronto. /bring ou B")
