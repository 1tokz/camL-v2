# camL-v2
--// COMBINED KEY SYSTEM + AIM LOCK GUI
--// For your own Roblox experience
--// Put this LocalScript in StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

--==================================================
-- SETTINGS
--==================================================

local REQUIRED_KEY = "ZAKIPOGI123"

local MAX_DISTANCE = 250
local FOV_RADIUS = 180
local LOCK_SPEED = 35
local TARGET_PART = "Head"

local aiming = false
local lockedTarget = nil

--==================================================
-- MAIN GUI
--==================================================

local gui = Instance.new("ScreenGui")
gui.Name = "AimLockSystem"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

--==================================================
-- KEY GUI
--==================================================

local keyFrame = Instance.new("Frame")
keyFrame.Size = UDim2.fromOffset(320, 190)
keyFrame.Position = UDim2.new(0.5, -160, 0.5, -95)
keyFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
keyFrame.BorderSizePixel = 0
keyFrame.Parent = gui

local keyCorner = Instance.new("UICorner")
keyCorner.CornerRadius = UDim.new(0, 12)
keyCorner.Parent = keyFrame

local keyTitle = Instance.new("TextLabel")
keyTitle.Size = UDim2.new(1, 0, 0, 45)
keyTitle.BackgroundTransparency = 1
keyTitle.Text = "🔐 AIM LOCK KEY"
keyTitle.TextColor3 = Color3.new(1, 1, 1)
keyTitle.TextSize = 20
keyTitle.Font = Enum.Font.GothamBold
keyTitle.Parent = keyFrame

local keyBox = Instance.new("TextBox")
keyBox.Position = UDim2.fromOffset(20, 60)
keyBox.Size = UDim2.new(1, -40, 0, 40)
keyBox.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
keyBox.BorderSizePixel = 0
keyBox.PlaceholderText = "Enter key..."
keyBox.Text = ""
keyBox.TextColor3 = Color3.new(1, 1, 1)
keyBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 150)
keyBox.TextSize = 14
keyBox.Font = Enum.Font.Gotham
keyBox.ClearTextOnFocus = false
keyBox.Parent = keyFrame

local keyBoxCorner = Instance.new("UICorner")
keyBoxCorner.CornerRadius = UDim.new(0, 8)
keyBoxCorner.Parent = keyBox

local verifyButton = Instance.new("TextButton")
verifyButton.Position = UDim2.fromOffset(20, 112)
verifyButton.Size = UDim2.new(1, -40, 0, 40)
verifyButton.BackgroundColor3 = Color3.fromRGB(60, 140, 255)
verifyButton.BorderSizePixel = 0
verifyButton.Text = "VERIFY KEY"
verifyButton.TextColor3 = Color3.new(1, 1, 1)
verifyButton.TextSize = 14
verifyButton.Font = Enum.Font.GothamBold
verifyButton.Parent = keyFrame

local verifyCorner = Instance.new("UICorner")
verifyCorner.CornerRadius = UDim.new(0, 8)
verifyCorner.Parent = verifyButton

local keyStatus = Instance.new("TextLabel")
keyStatus.Position = UDim2.fromOffset(20, 158)
keyStatus.Size = UDim2.new(1, -40, 0, 20)
keyStatus.BackgroundTransparency = 1
keyStatus.Text = ""
keyStatus.TextSize = 12
keyStatus.Font = Enum.Font.Gotham
keyStatus.Parent = keyFrame

--==================================================
-- AIM LOCK GUI
--==================================================

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(300, 270)
main.Position = UDim2.new(0, 30, 0.5, -135)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
main.BorderSizePixel = 0
main.Visible = false
main.Parent = gui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundTransparency = 1
title.Text = "🎯 AIM LOCK"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = main

local status = Instance.new("TextLabel")
status.Position = UDim2.fromOffset(15, 50)
status.Size = UDim2.new(1, -30, 0, 25)
status.BackgroundTransparency = 1
status.Text = "Status: OFF"
status.TextColor3 = Color3.fromRGB(255, 80, 80)
status.TextSize = 14
status.Font = Enum.Font.Gotham
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = main

local targetLabel = Instance.new("TextLabel")
targetLabel.Position = UDim2.fromOffset(15, 78)
targetLabel.Size = UDim2.new(1, -30, 0, 25)
targetLabel.BackgroundTransparency = 1
targetLabel.Text = "Target: None"
targetLabel.TextColor3 = Color3.fromRGB(190, 190, 200)
targetLabel.TextSize = 14
targetLabel.Font = Enum.Font.Gotham
targetLabel.TextXAlignment = Enum.TextXAlignment.Left
targetLabel.Parent = main

--==================================================
-- AIM TOGGLE
--==================================================

local toggle = Instance.new("TextButton")
toggle.Position = UDim2.fromOffset(15, 112)
toggle.Size = UDim2.new(1, -30, 0, 42)
toggle.BackgroundColor3 = Color3.fromRGB(170, 45, 45)
toggle.BorderSizePixel = 0
toggle.Text = "AIM LOCK: OFF"
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.TextSize = 15
toggle.Font = Enum.Font.GothamBold
toggle.Parent = main

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 8)
toggleCorner.Parent = toggle

--==================================================
-- FOV
--==================================================

local fovLabel = Instance.new("TextLabel")
fovLabel.Position = UDim2.fromOffset(15, 165)
fovLabel.Size = UDim2.new(0.5, -15, 0, 25)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "FOV: 180"
fovLabel.TextColor3 = Color3.fromRGB(210, 210, 220)
fovLabel.TextSize = 13
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextXAlignment = Enum.TextXAlignment.Left
fovLabel.Parent = main

--==================================================
-- SPEED
--==================================================

local speedLabel = Instance.new("TextLabel")
speedLabel.Position = UDim2.new(0.5, 0, 165, 0)
speedLabel.Size = UDim2.new(0.5, -15, 0, 25)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Speed: 35"
speedLabel.TextColor3 = Color3.fromRGB(210, 210, 220)
speedLabel.TextSize = 13
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextXAlignment = Enum.TextXAlignment.Right
speedLabel.Parent = main

--==================================================
-- FOV SLIDER
--==================================================

local fovBar = Instance.new("TextButton")
fovBar.Position = UDim2.fromOffset(15, 198)
fovBar.Size = UDim2.new(1, -30, 0, 12)
fovBar.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
fovBar.BorderSizePixel = 0
fovBar.Text = ""
fovBar.AutoButtonColor = false
fovBar.Parent = main

local fovFill = Instance.new("Frame")
fovFill.Size = UDim2.new(0.37, 0, 1, 0)
fovFill.BackgroundColor3 = Color3.fromRGB(80, 140, 255)
fovFill.BorderSizePixel = 0
fovFill.Parent = fovBar

--==================================================
-- TARGET CHECK
--==================================================

local function isValidTarget(character)

	if not character then
		return false
	end

	local humanoid =
		character:FindFirstChildOfClass("Humanoid")

	local root =
		character:FindFirstChild("HumanoidRootPart")

	local targetPart =
		character:FindFirstChild(TARGET_PART)

	if not humanoid or humanoid.Health <= 0 then
		return false
	end

	if not root or not targetPart then
		return false
	end

	local distance =
		(root.Position - camera.CFrame.Position).Magnitude

	return distance <= MAX_DISTANCE
end

--==================================================
-- FIND TARGET
--==================================================

local function getClosestTarget()

	local closest = nil
	local closestDistance = FOV_RADIUS

	local center = Vector2.new(
		camera.ViewportSize.X / 2,
		camera.ViewportSize.Y / 2
	)

	for _, otherPlayer in ipairs(Players:GetPlayers()) do

		if otherPlayer ~= player then

			local character = otherPlayer.Character

			if isValidTarget(character) then

				local targetPart =
					character:FindFirstChild(TARGET_PART)

				local position, visible =
					camera:WorldToViewportPoint(
						targetPart.Position
					)

				if visible and position.Z > 0 then

					local screenDistance = (
						Vector2.new(
							position.X,
							position.Y
						) - center
					).Magnitude

					if screenDistance < closestDistance then
						closestDistance = screenDistance
						closest = character
					end
				end
			end
		end
	end

	return closest
end

--==================================================
-- AIM TOGGLE
--==================================================

local function setAimLock(state)

	aiming = state

	if aiming then

		lockedTarget = getClosestTarget()

		status.Text = "Status: ON"
		status.TextColor3 =
			Color3.fromRGB(80, 255, 120)

		toggle.Text = "AIM LOCK: ON"
		toggle.BackgroundColor3 =
			Color3.fromRGB(45, 170, 80)

	else

		lockedTarget = nil

		status.Text = "Status: OFF"
		status.TextColor3 =
			Color3.fromRGB(255, 80, 80)

		targetLabel.Text = "Target: None"

		toggle.Text = "AIM LOCK: OFF"
		toggle.BackgroundColor3 =
			Color3.fromRGB(170, 45, 45)
	end
end

toggle.MouseButton1Click:Connect(function()
	setAimLock(not aiming)
end)

--==================================================
-- CONTINUOUS CAMERA LOCK
-- LOCKS ONTO ONE TARGET UNTIL IT BECOMES INVALID
--==================================================

RunService:BindToRenderStep(
	"AimLockCamera",
	Enum.RenderPriority.Camera.Value + 1,
	function(deltaTime)

		if not aiming then
			return
		end

		-- IMPORTANT:
		-- HUWAG kumuha ng bagong target habang valid pa
		-- ang kasalukuyang naka-lock na target.
		if not lockedTarget then
			lockedTarget = getClosestTarget()
		end

		-- Kapag namatay / nawala / lumampas sa range ang target,
		-- saka lang maghanap ng bagong target.
		if lockedTarget and not isValidTarget(lockedTarget) then
			lockedTarget = getClosestTarget()
		end

		if not lockedTarget then
			targetLabel.Text = "Target: None"
			return
		end

		local targetPart =
			lockedTarget:FindFirstChild(TARGET_PART)

		if not targetPart then
			lockedTarget = getClosestTarget()
			targetLabel.Text = "Target: None"
			return
		end

		targetLabel.Text =
			"Target: " .. lockedTarget.Name

		local desired =
			CFrame.lookAt(
				camera.CFrame.Position,
				targetPart.Position
			)

		local alpha =
			1 - math.exp(-LOCK_SPEED * deltaTime)

		camera.CFrame =
			camera.CFrame:Lerp(
				desired,
				alpha
			)
	end
)

--==================================================
-- FOV CONTROL
--==================================================

fovBar.MouseButton1Click:Connect(function()

	local mousePosition = UserInputService:GetMouseLocation()

	local x =
		math.clamp(
			mousePosition.X - fovBar.AbsolutePosition.X,
			0,
			fovBar.AbsoluteSize.X
		)

	local percent =
		x / fovBar.AbsoluteSize.X

	FOV_RADIUS =
		math.floor(50 + percent * 350)

	fovLabel.Text =
		"FOV: " .. FOV_RADIUS

	fovFill.Size =
		UDim2.new(percent, 0, 1, 0)
end)

--==================================================
-- KEYBOARD
--==================================================

UserInputService.InputBegan:Connect(function(input, processed)

	if processed then
		return
	end

	if input.KeyCode == Enum.KeyCode.Q
		and main.Visible then

		setAimLock(not aiming)
	end

	if input.KeyCode == Enum.KeyCode.RightShift
		and main.Visible then

		main.Visible = not main.Visible
	end
end)

--==================================================
-- VERIFY KEY
--==================================================

verifyButton.MouseButton1Click:Connect(function()

	if keyBox.Text == REQUIRED_KEY then

		keyStatus.Text = "✓ Key accepted!"
		keyStatus.TextColor3 =
			Color3.fromRGB(80, 255, 120)

		task.wait(0.5)

		keyFrame.Visible = false
		main.Visible = true

	else

		keyStatus.Text = "✕ Invalid key"
		keyStatus.TextColor3 =
			Color3.fromRGB(255, 80, 80)
	end
end)

--==================================================
-- DRAG GUI
--==================================================

local dragging = false
local dragStart
local startPosition

title.InputBegan:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1 then

		dragging = true
		dragStart = input.Position
		startPosition = main.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)

	if dragging and
		input.UserInputType ==
		Enum.UserInputType.MouseMovement then

		local delta =
			input.Position - dragStart

		main.Position = UDim2.new(
			startPosition.X.Scale,
			startPosition.X.Offset + delta.X,
			startPosition.Y.Scale,
			startPosition.Y.Offset + delta.Y
		)
	end
end)

UserInputService.InputEnded:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1 then

		dragging = false
	end
end)
