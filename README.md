--==================================================
-- CAM-LOCK-V2 FIXED
-- For your own Roblox experience
-- StarterPlayer > StarterPlayerScripts
--==================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local playerGui = player:WaitForChild("PlayerGui")

--==================================================
-- SETTINGS
--==================================================

local REQUIRED_KEY = "ZAKIPOGI123"
local KEY_LINK = "https://guns.lol/zakiwya"

local DEFAULTS = {
	MAX_DISTANCE = 250,
	FOV_RADIUS = 180,
	LOCK_SPEED = 35,
	TARGET_PART = "Head",

	TEAM_CHECK = true,
	LINE_OF_SIGHT = false,
	AUTO_UNLOCK_FAR = true,
	AUTO_UNLOCK_DEATH = true,

	PRIORITY = "Crosshair",
	HOLD_MODE = false,
	ACTIVATION_KEY = Enum.KeyCode.Q,
}

local MAX_DISTANCE = DEFAULTS.MAX_DISTANCE
local FOV_RADIUS = DEFAULTS.FOV_RADIUS
local LOCK_SPEED = DEFAULTS.LOCK_SPEED
local TARGET_PART = DEFAULTS.TARGET_PART

local TEAM_CHECK = DEFAULTS.TEAM_CHECK
local LINE_OF_SIGHT = DEFAULTS.LINE_OF_SIGHT
local AUTO_UNLOCK_FAR = DEFAULTS.AUTO_UNLOCK_FAR
local AUTO_UNLOCK_DEATH = DEFAULTS.AUTO_UNLOCK_DEATH

local PRIORITY = DEFAULTS.PRIORITY
local HOLD_MODE = DEFAULTS.HOLD_MODE
local ACTIVATION_KEY = DEFAULTS.ACTIVATION_KEY

local aiming = false
local lockedTarget = nil

--==================================================
-- GUI
--==================================================

local gui = Instance.new("ScreenGui")
gui.Name = "AimLockSystem"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = playerGui

--==================================================
-- HELPERS
--==================================================

local function createCorner(parent, radius)
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, radius)
	corner.Parent = parent
	return corner
end

local function tween(object, properties, duration)
	local t = TweenService:Create(
		object,
		TweenInfo.new(
			duration or 0.15,
			Enum.EasingStyle.Quad,
			Enum.EasingDirection.Out
		),
		properties
	)

	t:Play()
	return t
end

--==================================================
-- NOTIFICATION
--==================================================

local notification = Instance.new("TextLabel")
notification.Name = "Notification"
notification.Size = UDim2.fromOffset(300, 45)
notification.Position = UDim2.new(0.5, -150, 0, -60)
notification.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
notification.TextColor3 = Color3.new(1, 1, 1)
notification.TextSize = 14
notification.Font = Enum.Font.GothamBold
notification.Text = ""
notification.Visible = false
notification.Parent = gui
createCorner(notification, 10)

local notificationId = 0

local function notify(message, color)
	notificationId += 1
	local id = notificationId

	notification.Text = message
	notification.TextColor3 = color or Color3.new(1, 1, 1)
	notification.Visible = true

	tween(
		notification,
		{
			Position = UDim2.new(0.5, -150, 0, 20)
		},
		0.2
	)

	task.delay(1.5, function()
		if id ~= notificationId then
			return
		end

		local t = tween(
			notification,
			{
				Position = UDim2.new(0.5, -150, 0, -60)
			},
			0.2
		)

		t.Completed:Connect(function()
			if id == notificationId then
				notification.Visible = false
			end
		end)
	end)
end

--==================================================
-- KEY GUI
--==================================================

local keyFrame = Instance.new("Frame")
keyFrame.Name = "KeyFrame"
keyFrame.Size = UDim2.fromOffset(320, 235)
keyFrame.Position = UDim2.new(0.5, -160, 0.5, -117)
keyFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
keyFrame.BorderSizePixel = 0
keyFrame.Parent = gui
createCorner(keyFrame, 12)

local keyTitle = Instance.new("TextLabel")
keyTitle.Size = UDim2.new(1, 0, 0, 45)
keyTitle.BackgroundTransparency = 1
keyTitle.Text = "AIM LOCK KEY"
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
keyBox.TextSize = 14
keyBox.Font = Enum.Font.Gotham
keyBox.ClearTextOnFocus = false
keyBox.Parent = keyFrame
createCorner(keyBox, 8)

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
createCorner(verifyButton, 8)

local getKeyButton = Instance.new("TextButton")
getKeyButton.Position = UDim2.fromOffset(20, 158)
getKeyButton.Size = UDim2.new(1, -40, 0, 30)
getKeyButton.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
getKeyButton.BorderSizePixel = 0
getKeyButton.Text = "GET KEY LINK"
getKeyButton.TextColor3 = Color3.fromRGB(100, 180, 255)
getKeyButton.TextSize = 12
getKeyButton.Font = Enum.Font.GothamBold
getKeyButton.Parent = keyFrame
createCorner(getKeyButton, 7)

local keyStatus = Instance.new("TextLabel")
keyStatus.Position = UDim2.fromOffset(20, 195)
keyStatus.Size = UDim2.new(1, -40, 0, 25)
keyStatus.BackgroundTransparency = 1
keyStatus.Text = ""
keyStatus.TextColor3 = Color3.fromRGB(190, 190, 200)
keyStatus.TextSize = 12
keyStatus.Font = Enum.Font.Gotham
keyStatus.Parent = keyFrame

getKeyButton.MouseButton1Click:Connect(function()
	keyStatus.Text = "Key link: " .. KEY_LINK
	keyStatus.TextColor3 = Color3.fromRGB(255, 200, 80)
end)

--==================================================
-- MAIN GUI
--==================================================

local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.fromOffset(340, 500)
main.Position = UDim2.new(0, 30, 0.5, -250)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
main.BorderSizePixel = 0
main.Visible = false
main.Parent = gui
createCorner(main, 12)

local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, -80, 0, 45)
title.BackgroundTransparency = 1
title.Text = "AIM LOCK"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = main

--==================================================
-- MINIMIZE
--==================================================

local minimize = Instance.new("TextButton")
minimize.Size = UDim2.fromOffset(32, 32)
minimize.Position = UDim2.new(1, -42, 0, 7)
minimize.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
minimize.Text = "—"
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.TextSize = 18
minimize.Font = Enum.Font.GothamBold
minimize.Parent = main
createCorner(minimize, 7)

local minimized = false

--==================================================
-- STATUS
--==================================================

local status = Instance.new("TextLabel")
status.Position = UDim2.fromOffset(15, 50)
status.Size = UDim2.new(1, -30, 0, 24)
status.BackgroundTransparency = 1
status.Text = "Status: OFF"
status.TextColor3 = Color3.fromRGB(255, 80, 80)
status.TextSize = 14
status.Font = Enum.Font.Gotham
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = main

local targetLabel = Instance.new("TextLabel")
targetLabel.Position = UDim2.fromOffset(15, 76)
targetLabel.Size = UDim2.new(1, -30, 0, 22)
targetLabel.BackgroundTransparency = 1
targetLabel.Text = "Target: None"
targetLabel.TextColor3 = Color3.fromRGB(190, 190, 200)
targetLabel.TextSize = 13
targetLabel.Font = Enum.Font.Gotham
targetLabel.TextXAlignment = Enum.TextXAlignment.Left
targetLabel.Parent = main

local distanceLabel = Instance.new("TextLabel")
distanceLabel.Position = UDim2.fromOffset(15, 100)
distanceLabel.Size = UDim2.new(1, -30, 0, 22)
distanceLabel.BackgroundTransparency = 1
distanceLabel.Text = "Distance: --"
distanceLabel.TextColor3 = Color3.fromRGB(160, 160, 175)
distanceLabel.TextSize = 13
distanceLabel.Font = Enum.Font.Gotham
distanceLabel.TextXAlignment = Enum.TextXAlignment.Left
distanceLabel.Parent = main

local hpLabel = Instance.new("TextLabel")
hpLabel.Position = UDim2.fromOffset(15, 124)
hpLabel.Size = UDim2.new(1, -30, 0, 22)
hpLabel.BackgroundTransparency = 1
hpLabel.Text = "HP: --"
hpLabel.TextColor3 = Color3.fromRGB(160, 220, 170)
hpLabel.TextSize = 13
hpLabel.Font = Enum.Font.Gotham
hpLabel.TextXAlignment = Enum.TextXAlignment.Left
hpLabel.Parent = main

--==================================================
-- BUTTONS
--==================================================

local toggle = Instance.new("TextButton")
toggle.Position = UDim2.fromOffset(15, 152)
toggle.Size = UDim2.new(1, -30, 0, 40)
toggle.BackgroundColor3 = Color3.fromRGB(170, 45, 45)
toggle.Text = "AIM LOCK: OFF"
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.TextSize = 14
toggle.Font = Enum.Font.GothamBold
toggle.BorderSizePixel = 0
toggle.Parent = main
createCorner(toggle, 8)

local function makeButton(y, text)
	local b = Instance.new("TextButton")
	b.Position = UDim2.fromOffset(15, y)
	b.Size = UDim2.new(1, -30, 0, 30)
	b.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
	b.BorderSizePixel = 0
	b.Text = text
	b.TextColor3 = Color3.new(1, 1, 1)
	b.TextSize = 12
	b.Font = Enum.Font.GothamBold
	b.Parent = main
	createCorner(b, 7)

	return b
end

local partButton = makeButton(200, "Target Part: Head")
local priorityButton = makeButton(236, "Priority: Crosshair")
local teamButton = makeButton(272, "Team Check: ON")
local losButton = makeButton(308, "Line of Sight: OFF")
local farButton = makeButton(344, "Auto Unlock Far: ON")
local holdButton = makeButton(380, "Mode: TOGGLE")
local resetButton = makeButton(416, "RESET SETTINGS")

--==================================================
-- TARGET PART
--==================================================

local targetParts = {
	"Head",
	"UpperTorso",
	"HumanoidRootPart",
}

local partIndex = 1

partButton.MouseButton1Click:Connect(function()
	partIndex += 1

	if partIndex > #targetParts then
		partIndex = 1
	end

	TARGET_PART = targetParts[partIndex]
	partButton.Text = "Target Part: " .. TARGET_PART
end)

--==================================================
-- PRIORITY
--==================================================

local priorities = {
	"Crosshair",
	"Distance",
	"Lowest HP",
}

local priorityIndex = 1

priorityButton.MouseButton1Click:Connect(function()
	priorityIndex += 1

	if priorityIndex > #priorities then
		priorityIndex = 1
	end

	PRIORITY = priorities[priorityIndex]
	priorityButton.Text = "Priority: " .. PRIORITY
end)

--==================================================
-- TOGGLE BUTTON HELPER
--==================================================

local function updateToggleButton(button, label, enabled)
	button.Text = label .. ": " .. (enabled and "ON" or "OFF")

	if enabled then
		button.BackgroundColor3 = Color3.fromRGB(45, 170, 80)
	else
		button.BackgroundColor3 = Color3.fromRGB(170, 45, 45)
	end
end

--==================================================
-- TEAM CHECK
--==================================================

teamButton.MouseButton1Click:Connect(function()
	TEAM_CHECK = not TEAM_CHECK
	updateToggleButton(teamButton, "Team Check", TEAM_CHECK)
end)

--==================================================
-- LINE OF SIGHT
--==================================================

losButton.MouseButton1Click:Connect(function()
	LINE_OF_SIGHT = not LINE_OF_SIGHT
	updateToggleButton(losButton, "Line of Sight", LINE_OF_SIGHT)
end)

--==================================================
-- AUTO UNLOCK FAR
--==================================================

farButton.MouseButton1Click:Connect(function()
	AUTO_UNLOCK_FAR = not AUTO_UNLOCK_FAR
	updateToggleButton(farButton, "Auto Unlock Far", AUTO_UNLOCK_FAR)
end)

--==================================================
-- HOLD MODE
--==================================================

holdButton.MouseButton1Click:Connect(function()
	HOLD_MODE = not HOLD_MODE

	if HOLD_MODE then
		holdButton.Text = "Mode: HOLD"
	else
		holdButton.Text = "Mode: TOGGLE"
	end
end)

--==================================================
-- HIGHLIGHT
--==================================================

local highlight = Instance.new("Highlight")
highlight.Name = "AimLockHighlight"
highlight.FillColor = Color3.fromRGB(255, 70, 70)
highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
highlight.FillTransparency = 0.75
highlight.OutlineTransparency = 0
highlight.Enabled = false
highlight.Parent = workspace

--==================================================
-- TEAM CHECK
--==================================================

local function isEnemy(otherPlayer)
	if not TEAM_CHECK then
		return true
	end

	if player.Team ~= nil and otherPlayer.Team == player.Team then
		return false
	end

	return true
end

--==================================================
-- LINE OF SIGHT
--==================================================

local function hasLineOfSight(targetPart, character)
	if not LINE_OF_SIGHT then
		return true
	end

	local origin = camera.CFrame.Position
	local direction = targetPart.Position - origin

	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = {
		player.Character,
	}

	local result = workspace:Raycast(
		origin,
		direction,
		params
	)

	if not result then
		return true
	end

	return result.Instance:IsDescendantOf(character)
end

--==================================================
-- VALID TARGET
--==================================================

local function isValidTarget(character)
	if not character then
		return false
	end

	local humanoid = character:FindFirstChildOfClass("Humanoid")
	local root = character:FindFirstChild("HumanoidRootPart")
	local targetPart = character:FindFirstChild(TARGET_PART)

	if not humanoid or humanoid.Health <= 0 then
		return false
	end

	if not root or not targetPart then
		return false
	end

	local distance = (
		root.Position - camera.CFrame.Position
	).Magnitude

	if distance > MAX_DISTANCE then
		return false
	end

	if not hasLineOfSight(targetPart, character) then
		return false
	end

	return true
end

--==================================================
-- TARGET SCORE
--==================================================

local function getTargetScore(character)
	local root = character:FindFirstChild("HumanoidRootPart")
	local targetPart = character:FindFirstChild(TARGET_PART)
	local humanoid = character:FindFirstChildOfClass("Humanoid")

	if not root or not targetPart or not humanoid then
		return math.huge
	end

	local distance = (
		root.Position - camera.CFrame.Position
	).Magnitude

	if PRIORITY == "Distance" then
		return distance
	end

	if PRIORITY == "Lowest HP" then
		return humanoid.Health
	end

	local center = Vector2.new(
		camera.ViewportSize.X / 2,
		camera.ViewportSize.Y / 2
	)

	local screenPos, visible =
		camera:WorldToViewportPoint(targetPart.Position)

	if not visible or screenPos.Z <= 0 then
		return math.huge
	end

	return (
		Vector2.new(screenPos.X, screenPos.Y) - center
	).Magnitude
end

--==================================================
-- FIND TARGET
--==================================================

local function getClosestTarget(excludeTarget)
	local closest = nil
	local bestScore = math.huge

	for _, otherPlayer in ipairs(Players:GetPlayers()) do
		if otherPlayer ~= player and isEnemy(otherPlayer) then

			local character = otherPlayer.Character

			if character
				and character ~= excludeTarget
				and isValidTarget(character) then

				local score = getTargetScore(character)

				if score < bestScore and score <= FOV_RADIUS then
					bestScore = score
					closest = character
				end
			end
		end
	end

	return closest
end

--==================================================
-- CLEAR LOCK
--==================================================

local function clearLock()
	lockedTarget = nil

	highlight.Enabled = false
	highlight.Adornee = nil

	targetLabel.Text = "Target: None"
	distanceLabel.Text = "Distance: --"
	hpLabel.Text = "HP: --"
end

--==================================================
-- UPDATE AIM BUTTON
--==================================================

local function updateAimUI()
	if aiming then
		status.Text = "Status: ON"
		status.TextColor3 = Color3.fromRGB(80, 255, 120)

		toggle.Text = "AIM LOCK: ON"
		toggle.BackgroundColor3 = Color3.fromRGB(45, 170, 80)
	else
		status.Text = "Status: OFF"
		status.TextColor3 = Color3.fromRGB(255, 80, 80)

		toggle.Text = "AIM LOCK: OFF"
		toggle.BackgroundColor3 = Color3.fromRGB(170, 45, 45)
	end
end

--==================================================
-- AIM STATE
--==================================================

local function setAimLock(state)
	aiming = state

	if not aiming then
		clearLock()
		updateAimUI()
		return
	end

	lockedTarget = getClosestTarget()

	updateAimUI()

	if lockedTarget then
		highlight.Adornee = lockedTarget
		highlight.Enabled = true

		notify(
			"LOCKED: " .. lockedTarget.Name,
			Color3.fromRGB(80, 255, 120)
		)
	else
		notify(
			"NO TARGET",
			Color3.fromRGB(255, 180, 80)
		)
	end
end

toggle.MouseButton1Click:Connect(function()
	setAimLock(not aiming)
end)

--==================================================
-- SWITCH TARGET
--==================================================

local function switchTarget()
	if not aiming then
		return
	end

	local newTarget = getClosestTarget(lockedTarget)

	if newTarget then
		lockedTarget = newTarget

		highlight.Adornee = newTarget
		highlight.Enabled = true

		notify(
			"TARGET: " .. newTarget.Name,
			Color3.fromRGB(80, 170, 255)
		)
	end
end

--==================================================
-- RESET
--==================================================

resetButton.MouseButton1Click:Connect(function()
	MAX_DISTANCE = DEFAULTS.MAX_DISTANCE
	FOV_RADIUS = DEFAULTS.FOV_RADIUS
	LOCK_SPEED = DEFAULTS.LOCK_SPEED
	TARGET_PART = DEFAULTS.TARGET_PART

	TEAM_CHECK = DEFAULTS.TEAM_CHECK
	LINE_OF_SIGHT = DEFAULTS.LINE_OF_SIGHT
	AUTO_UNLOCK_FAR = DEFAULTS.AUTO_UNLOCK_FAR
	AUTO_UNLOCK_DEATH = DEFAULTS.AUTO_UNLOCK_DEATH

	PRIORITY = DEFAULTS.PRIORITY
	HOLD_MODE = DEFAULTS.HOLD_MODE
	ACTIVATION_KEY = DEFAULTS.ACTIVATION_KEY

	partIndex = 1
	priorityIndex = 1

	partButton.Text = "Target Part: Head"
	priorityButton.Text = "Priority: Crosshair"
	holdButton.Text = "Mode: TOGGLE"

	updateToggleButton(
		teamButton,
		"Team Check",
		TEAM_CHECK
	)

	updateToggleButton(
		losButton,
		"Line of Sight",
		LINE_OF_SIGHT
	)

	updateToggleButton(
		farButton,
		"Auto Unlock Far",
		AUTO_UNLOCK_FAR
	)

	setAimLock(false)

	notify(
		"SETTINGS RESET",
		Color3.fromRGB(80, 170, 255)
	)
end)

--==================================================
-- CAMERA LOCK
--==================================================

RunService:BindToRenderStep(
	"AimLockCamera",
	Enum.RenderPriority.Camera.Value + 1,
	function(deltaTime)

		if not aiming or not lockedTarget then
			return
		end

		if not lockedTarget.Parent then
			setAimLock(false)
			return
		end

		local humanoid =
			lockedTarget:FindFirstChildOfClass("Humanoid")

		local root =
			lockedTarget:FindFirstChild("HumanoidRootPart")

		local targetPart =
			lockedTarget:FindFirstChild(TARGET_PART)

		-- DEATH
		if not humanoid or humanoid.Health <= 0 then
			clearLock()

			if AUTO_UNLOCK_DEATH then
				aiming = false
				updateAimUI()

				notify(
					"TARGET LOST",
					Color3.fromRGB(255, 100, 100)
				)
			end

			return
		end

		if not root or not targetPart then
			setAimLock(false)
			return
		end

		-- DISTANCE
		local distance = (
			root.Position - camera.CFrame.Position
		).Magnitude

		distanceLabel.Text =
			"Distance: " .. math.floor(distance) .. " studs"

		-- HP
		hpLabel.Text =
			"HP: "
			.. math.floor(humanoid.Health)
			.. " / "
			.. math.floor(humanoid.MaxHealth)

		-- FAR
		if AUTO_UNLOCK_FAR and distance > MAX_DISTANCE then
			setAimLock(false)

			notify(
				"TARGET TOO FAR",
				Color3.fromRGB(255, 180, 80)
			)

			return
		end

		-- LOS
		if LINE_OF_SIGHT
			and not hasLineOfSight(targetPart, lockedTarget) then

			highlight.FillColor =
				Color3.fromRGB(255, 180, 50)
		else
			highlight.FillColor =
				Color3.fromRGB(255, 70, 70)
		end

		-- INFO
		targetLabel.Text =
			"Target: " .. lockedTarget.Name

		highlight.Adornee = lockedTarget
		highlight.Enabled = true

		-- CAMERA
		local desired = CFrame.lookAt(
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
-- KEYBOARD
--==================================================

UserInputService.InputBegan:Connect(function(input, processed)
	if processed then
		return
	end

	if input.KeyCode == ACTIVATION_KEY then
		if HOLD_MODE then
			setAimLock(true)
		else
			setAimLock(not aiming)
		end

		return
	end

	if input.KeyCode == Enum.KeyCode.E then
		switchTarget()
		return
	end

	if input.KeyCode == Enum.KeyCode.RightShift then
		if main.Visible then
			main.Visible = not main.Visible
		end
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.KeyCode == ACTIVATION_KEY and HOLD_MODE then
		setAimLock(false)
	end
end)

--==================================================
-- MINIMIZE
--==================================================

minimize.MouseButton1Click:Connect(function()
	minimized = not minimized

	for _, child in ipairs(main:GetChildren()) do
		if child:IsA("GuiObject")
			and child ~= title
			and child ~= minimize then

			child.Visible = not minimized
		end
	end

	if minimized then
		main.Size = UDim2.fromOffset(340, 55)
		minimize.Text = "+"
	else
		main.Size = UDim2.fromOffset(340, 500)
		minimize.Text = "—"
	end
end)

--==================================================
-- DRAG
--==================================================

local dragging = false
local dragStart
local startPosition

title.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPosition = main.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging
		and input.UserInputType == Enum.UserInputType.MouseMovement then

		local delta = input.Position - dragStart

		main.Position = UDim2.new(
			startPosition.X.Scale,
			startPosition.X.Offset + delta.X,
			startPosition.Y.Scale,
			startPosition.Y.Offset + delta.Y
		)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

--==================================================
-- VERIFY KEY
--==================================================

verifyButton.MouseButton1Click:Connect(function()
	if keyBox.Text == REQUIRED_KEY then

		keyStatus.Text = "Key accepted!"
		keyStatus.TextColor3 =
			Color3.fromRGB(80, 255, 120)

		task.wait(0.4)

		keyFrame.Visible = false
		main.Visible = true

		notify(
			"AIM LOCK READY",
			Color3.fromRGB(80, 255, 120)
		)

	else
		keyStatus.Text = "Invalid key"
		keyStatus.TextColor3 =
			Color3.fromRGB(255, 80, 80)
	end
end)

--==================================================
-- MOBILE
--==================================================

local mobileButton = Instance.new("TextButton")
mobileButton.Name = "MobileLock"
mobileButton.Size = UDim2.fromOffset(65, 65)
mobileButton.Position = UDim2.new(1, -85, 1, -100)
mobileButton.BackgroundColor3 = Color3.fromRGB(170, 45, 45)
mobileButton.Text = "LOCK"
mobileButton.TextColor3 = Color3.new(1, 1, 1)
mobileButton.TextSize = 13
mobileButton.Font = Enum.Font.GothamBold
mobileButton.Visible = UserInputService.TouchEnabled
mobileButton.Parent = gui
createCorner(mobileButton, 999)

mobileButton.MouseButton1Click:Connect(function()
	setAimLock(not aiming)
end)

--==================================================
-- INITIAL STATE
--==================================================

updateToggleButton(teamButton, "Team Check", TEAM_CHECK)
updateToggleButton(losButton, "Line of Sight", LINE_OF_SIGHT)
updateToggleButton(farButton, "Auto Unlock Far", AUTO_UNLOCK_FAR)

print("[CAM-LOCK-V2] Loaded successfully")
