repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local player = Players.LocalPlayer
repeat task.wait() until player:FindFirstChild("PlayerGui")

-- Global State Variables (Default values)
local currentID = 17121838966
local targetLeg = "Left Leg"
local offsetX, offsetY, offsetZ = 0, 0, 0
local rotateX, rotateY, rotateZ = 0, 0, 0
local scaleX, scaleY, scaleZ = 1, 1, 1

local activeAccessory = nil

-- Function to clean up old custom accessories
local function removeExistingAccessory(character)
	if character then
		local old = character:FindFirstChild("CustomR6LegAccessory")
		if old then old:Destroy() end
	end
end

-- Core Function to Build and Weld the Accessory
local function applyAccessory()
	local character = player.Character
	if not character then return end
	
	local humanoid = character:WaitForChild("Humanoid", 5)
	if not humanoid or humanoid.RigType ~= Enum.HumanoidRigType.R6 then return end
	
	removeExistingAccessory(character)
	
	local leg = character:FindFirstChild(targetLeg)
	if not leg then return end
	
	local success, assetModel = pcall(function()
		return game:GetObjects("rbxassetid://" .. currentID)[1]
	end)
	
	if not success or not assetModel then return end
	
	local handle = assetModel:FindFirstChild("Handle") or assetModel:FindFirstChildWhichIsA("Part", true) or assetModel:FindFirstChildWhichIsA("MeshPart", true)
	if not handle then return end
	
	handle.Anchored = false
	for _, obj in ipairs(assetModel:GetDescendants()) do
		if obj:IsA("BasePart") then
			obj.Anchored = false
			obj.CanCollide = false
			obj.Massless = true
		elseif obj:IsA("Weld") or obj:IsA("ManualWeld") or obj:IsA("WeldConstraint") or obj:IsA("Attachment") then
			obj:Destroy()
		elseif obj:IsA("SpecialMesh") then
			obj.Scale = Vector3.new(scaleX, scaleY, scaleZ)
		end
	end
	
	handle.Name = "Handle"
	handle.CanCollide = false
	handle.Massless = true
	
	if handle:IsA("MeshPart") or not handle:FindFirstChildWhichIsA("SpecialMesh") then
		handle.Size = Vector3.new(scaleX, scaleY, scaleZ)
	end
	
	local accessory = Instance.new("Accessory")
	accessory.Name = "CustomR6LegAccessory"
	handle.Parent = accessory
	
	local weld = Instance.new("Weld")
	weld.Name = "AccessoryWeld"
	weld.Part0 = handle
	weld.Part1 = leg
	
	local positionOffset = CFrame.new(offsetX, offsetY, offsetZ)
	local rotationOffset = CFrame.Angles(math.rad(rotateX), math.rad(rotateY), math.rad(rotateZ))
	
	weld.C0 = positionOffset * rotationOffset
	weld.Parent = handle
	
	accessory.Parent = character
	activeAccessory = accessory
end

-- UI Construction
local sgui = Instance.new("ScreenGui")
sgui.Name = "LegAccessoryGUI"
sgui.ResetOnSpawn = false
sgui.Parent = player:WaitForChild("PlayerGui")

-- Toggle Button
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 100, 0, 35)
toggleBtn.Position = UDim2.new(0, 10, 0, 10)
toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Text = "Toggle Menu"
toggleBtn.Font = Enum.Font.SourceSansBold
toggleBtn.TextSize = 16
toggleBtn.Parent = sgui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 340, 0, 520)
frame.Position = UDim2.new(0.5, -170, 0.5, -260)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 2
frame.Visible = true
frame.Parent = sgui

toggleBtn.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

local function createInputRow(labelName, defaultVal, yPos, callback)
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0, 140, 0, 25)
	label.Position = UDim2.new(0, 10, 0, yPos)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.fromRGB(200, 200, 200)
	label.Text = labelName
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Font = Enum.Font.SourceSans
	label.TextSize = 16
	label.Parent = frame

	local box = Instance.new("TextBox")
	box.Size = UDim2.new(0, 160, 0, 25)
	box.Position = UDim2.new(0, 160, 0, yPos)
	box.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	box.TextColor3 = Color3.fromRGB(255, 255, 255)
	box.Text = tostring(defaultVal)
	box.Font = Enum.Font.SourceSans
	box.TextSize = 16
	box.Parent = frame

	box.FocusLost:Connect(function()
		callback(box.Text)
	end)
end

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Text = "Leg Accessory Customizer (R6)"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.Parent = frame

createInputRow("Accessory ID:", currentID, 45, function(val) currentID = tonumber(val) or currentID end)
createInputRow("Offset X (L/R):", offsetX, 85, function(val) offsetX = tonumber(val) or offsetX end)
createInputRow("Offset Y (Up/Dn):", offsetY, 115, function(val) offsetY = tonumber(val) or offsetY end)
createInputRow("Offset Z (Fwd/Bkd):", offsetZ, 145, function(val) offsetZ = tonumber(val) or offsetZ end)

createInputRow("Rotate X (Tilt):", rotateX, 185, function(val) rotateX = tonumber(val) or rotateX end)
createInputRow("Rotate Y (Spin):", rotateY, 215, function(val) rotateY = tonumber(val) or rotateY end)
createInputRow("Rotate Z (Side):", rotateZ, 245, function(val) rotateZ = tonumber(val) or rotateZ end)

createInputRow("Scale X (Width):", scaleX, 285, function(val) scaleX = tonumber(val) or scaleX end)
createInputRow("Scale Y (Height):", scaleY, 315, function(val) scaleY = tonumber(val) or scaleY end)
createInputRow("Scale Z (Depth):", scaleZ, 345, function(val) scaleZ = tonumber(val) or scaleZ end)

local leftLegBtn = Instance.new("TextButton")
leftLegBtn.Size = UDim2.new(0, 150, 0, 30)
leftLegBtn.Position = UDim2.new(0, 10, 0, 390)
leftLegBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
leftLegBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
leftLegBtn.Text = "Target: Left Leg"
leftLegBtn.Font = Enum.Font.SourceSansBold
leftLegBtn.Parent = frame

local rightLegBtn = Instance.new("TextButton")
rightLegBtn.Size = UDim2.new(0, 150, 0, 30)
rightLegBtn.Position = UDim2.new(0, 180, 0, 390)
rightLegBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
rightLegBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
rightLegBtn.Text = "Target: Right Leg"
rightLegBtn.Font = Enum.Font.SourceSansBold
rightLegBtn.Parent = frame

leftLegBtn.MouseButton1Click:Connect(function()
	targetLeg = "Left Leg"
	leftLegBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
	rightLegBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
end)

rightLegBtn.MouseButton1Click:Connect(function()
	targetLeg = "Right Leg"
	rightLegBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
	leftLegBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
end)

local applyBtn = Instance.new("TextButton")
applyBtn.Size = UDim2.new(0, 320, 0, 40)
applyBtn.Position = UDim2.new(0, 10, 0, 440)
applyBtn.BackgroundColor3 = Color3.fromRGB(0, 100, 200)
applyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
applyBtn.Text = "APPLY / REFRESH"
applyBtn.Font = Enum.Font.SourceSansBold
applyBtn.TextSize = 20
applyBtn.Parent = frame

applyBtn.MouseButton1Click:Connect(function()
	applyAccessory()
end)

player.CharacterAdded:Connect(function(newCharacter)
	task.wait(1) 
	applyAccessory()
end)

if player.Character then
	task.spawn(applyAccessory)
end
