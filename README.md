-- Lethal Ape - Bring Monsters (Delta)
-- Comando: /bring ou tecla B

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local MonsterNames = {
	"Sandman", "Ashy", "Gus", "Dus", "Kus", "Lost",
	"Bloodman", "Lurker", "Scar", "Meat", "Billy",
	"SandGuy", "John", "Dlud", "Blud", "Unknown"
}

local function GetRoot(model)
	return model:FindFirstChild("HumanoidRootPart")
		or model.PrimaryPart
		or model:FindFirstChildWhichIsA("BasePart")
end

local function BringMonsters()
	local char = LocalPlayer.Character
	if not char then return end
	local myRoot = char:FindFirstChild("HumanoidRootPart")
	if not myRoot then return end

	local found = 0
	local angle = 0
	local radius = 10

	for _, name in ipairs(MonsterNames) do
		for _, obj in ipairs(Workspace:GetDescendants()) do
			if obj:IsA("Model") and obj.Name:lower():find(name:lower()) then
				local root = GetRoot(obj)
				if root and obj:FindFirstChildOfClass("Humanoid") then
					local offset = Vector3.new(
						math.cos(math.rad(angle)) * radius,
						2,
						math.sin(math.rad(angle)) * radius
					)

					pcall(function()
						root.CFrame = CFrame.new(myRoot.Position + offset)
						if obj.PrimaryPart then
							obj:SetPrimaryPartCFrame(CFrame.new(myRoot.Position + offset))
						end
					end)

					found += 1
					angle += 28
					break
				end
			end
		end
	end

	print("[Lethal Ape] Tentou trazer " .. found .. " monstros.")
end

-- Chat
LocalPlayer.Chatted:Connect(function(msg)
	if msg:lower() == "/bring" or msg:lower() == "/bringmonsters" then
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

print("Bring Monsters carregado. Use /bring ou aperte B.")
