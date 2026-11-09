local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

if PlayerGui:FindFirstChild("Matcha_ProHub") then
	PlayerGui.Matcha_ProHub:Destroy()
end

local Camera = Workspace.CurrentCamera

-- Estados Globales de Toggles
local states = {
	diamonds = false, masterpiece = false, gamingpc = false, safes = false,
	guards = false, npcs = false, police = false, cameras = false
}

local tracked = {
	diamonds = {}, masterpiece = {}, gamingpc = {}, safes = {},
	guards = {}, npcs = {}, police = {}, cameras = {}
}

-- Configuración General
local cfg = {
	ShowDistance = true,
	MaxDistance = 500,        -- Distancia máxima (10m a 2000m)
	FillTrans = 0.5,         -- Transparencia del Highlight
	TracersEnabled = true,   -- Tracers globales
	CameraTracers = false,   -- Opcional: Tracers específicos para cámaras
	TracerWidth = 0.04,
	TracerColorMode = "Match",
	CustomTracerColor = Color3.fromRGB(255, 255, 255),
	ThemeMode = "Purple",
	BaseColor = Color3.fromRGB(160, 80, 255),
	DarkColor = Color3.fromRGB(60, 20, 100)
}

--------------------------------------------------------------------------------
-- 1. PANTALLA DE CARGA
--------------------------------------------------------------------------------
local LoadGui = Instance.new("ScreenGui", PlayerGui)
LoadGui.Name = "Matcha_Loading"
local LoadFrame = Instance.new("Frame", LoadGui)
LoadFrame.Size = UDim2.new(0, 300, 0, 100)
LoadFrame.Position = UDim2.new(0.5, -150, 0.5, -50)
LoadFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
LoadFrame.BackgroundTransparency = 1
LoadFrame.Selectable = false
Instance.new("UICorner", LoadFrame).CornerRadius = UDim.new(0, 8)

local LoadTitle = Instance.new("TextLabel", LoadFrame)
LoadTitle.Size = UDim2.new(1, 0, 1, 0)
LoadTitle.BackgroundTransparency = 1
LoadTitle.Text = "Matcha - Fixing ESP Persistence & Camera Tracers..."
LoadTitle.TextColor3 = Color3.fromRGB(255, 105, 180)
LoadTitle.Font = Enum.Font.Code
LoadTitle.TextSize = 11
LoadTitle.TextTransparency = 1

TweenService:Create(LoadFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0.1}):Play()
TweenService:Create(LoadTitle, TweenInfo.new(0.3), {TextTransparency = 0}):Play()
task.wait(0.4)
TweenService:Create(LoadFrame, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
TweenService:Create(LoadTitle, TweenInfo.new(0.3), {TextTransparency = 1}):Play()
task.wait(0.2)
LoadGui:Destroy()

--------------------------------------------------------------------------------
-- 2. INTERFAZ PRINCIPAL
--------------------------------------------------------------------------------
local ScreenGui = Instance.new("ScreenGui", PlayerGui)
ScreenGui.Name = "Matcha_ProHub"
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 360, 0, 520)
MainFrame.Position = UDim2.new(0.05, 0, 0.15, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
MainFrame.Active = true
MainFrame.Selectable = false
MainFrame.ClipsDescendants = false
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 6)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = Color3.fromRGB(45, 45, 60)
MainStroke.Thickness = 1.2

local ModalBtn = Instance.new("TextButton", MainFrame)
ModalBtn.Size = UDim2.new(0, 0, 0, 0)
ModalBtn.Visible = true
ModalBtn.Modal = true
ModalBtn.Selectable = false

-- Header
local Header = Instance.new("Frame", MainFrame)
Header.Size = UDim2.new(1, 0, 0, 32)
Header.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
Header.Selectable = false
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 6)

local Title = Instance.new("TextLabel", Header)
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0.03, 0, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Matcha - <3 dyl"
Title.TextColor3 = cfg.BaseColor
Title.Font = Enum.Font.Code
Title.TextSize = 13
Title.TextXAlignment = Enum.TextXAlignment.Left

local rainbowHue = 0
RunService.RenderStepped:Connect(function(dt)
	if cfg.ThemeMode == "Rainbow" then
		rainbowHue = (rainbowHue + dt * 0.2) % 1
		local rainbowColor = Color3.fromHSV(rainbowHue, 0.8, 1)
		Title.TextColor3 = rainbowColor
		MainStroke.Color = rainbowColor
	end
end)

local CloseBtn = Instance.new("TextButton", Header)
CloseBtn.Size = UDim2.new(0, 24, 0, 24)
CloseBtn.Position = UDim2.new(1, -28, 0, 4)
CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 45, 45)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.Code
CloseBtn.TextSize = 11
CloseBtn.Selectable = true
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)

local MinimizeBtn = Instance.new("TextButton", Header)
MinimizeBtn.Size = UDim2.new(0, 24, 0, 24)
MinimizeBtn.Position = UDim2.new(1, -56, 0, 4)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
MinimizeBtn.Text = "—"
MinimizeBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
MinimizeBtn.Font = Enum.Font.Code
MinimizeBtn.TextSize = 11
MinimizeBtn.Selectable = true
Instance.new("UICorner", MinimizeBtn).CornerRadius = UDim.new(0, 4)

--------------------------------------------------------------------------------
-- 3. PESTAÑAS Y CONTENEDORES
--------------------------------------------------------------------------------
local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(0.94, 0, 0, 26)
TopBar.Position = UDim2.new(0.03, 0, 0, 38)
TopBar.BackgroundTransparency = 1
TopBar.Selectable = false

local function createTabBtn(posX, text)
	local btn = Instance.new("TextButton", TopBar)
	btn.Size = UDim2.new(0.31, 0, 1, 0)
	btn.Position = UDim2.new(posX, 0, 0, 0)
	btn.BackgroundColor3 = Color3.fromRGB(24, 24, 32)
	btn.TextColor3 = Color3.fromRGB(150, 150, 170)
	btn.Font = Enum.Font.Code
	btn.TextSize = 11
	btn.Text = text
	btn.Selectable = true
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
	return btn
end

local ESPTabBtn = createTabBtn(0, "ESP")
local ObjTabBtn = createTabBtn(0.345, "OBJECTS")
local SetTabBtn = createTabBtn(0.69, "SETTINGS")
ESPTabBtn.BackgroundColor3 = Color3.fromRGB(36, 36, 48)
ESPTabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)

local function createContainer()
	local cont = Instance.new("ScrollingFrame", MainFrame)
	cont.Size = UDim2.new(0.94, 0, 1, -74)
	cont.Position = UDim2.new(0.03, 0, 0, 68)
	cont.BackgroundTransparency = 1
	cont.AutomaticCanvasSize = Enum.AutomaticSize.Y
	cont.CanvasSize = UDim2.new(0, 0, 0, 0)
	cont.ScrollBarThickness = 3
	cont.Visible = false
	cont.Selectable = false
	
	local layout = Instance.new("UIListLayout", cont)
	layout.SortOrder = Enum.SortOrder.LayoutOrder
	layout.Padding = UDim.new(0, 6)
	return cont
end

local ESPContainer = createContainer()
ESPContainer.Visible = true
local ObjContainer = createContainer()
local SetContainer = createContainer()

local function switchTab(activeBtn, activeCont)
	for _, c in pairs({ESPContainer, ObjContainer, SetContainer}) do
		c.Visible = false
	end
	for _, b in pairs({ESPTabBtn, ObjTabBtn, SetTabBtn}) do
		b.BackgroundColor3 = Color3.fromRGB(24, 24, 32)
		b.TextColor3 = Color3.fromRGB(150, 150, 170)
	end
	activeCont.Visible = true
	activeBtn.BackgroundColor3 = Color3.fromRGB(36, 36, 48)
	activeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
end

ESPTabBtn.MouseButton1Click:Connect(function() switchTab(ESPTabBtn, ESPContainer) end)
ObjTabBtn.MouseButton1Click:Connect(function() switchTab(ObjTabBtn, ObjContainer) end)
SetTabBtn.MouseButton1Click:Connect(function() switchTab(SetTabBtn, SetContainer) end)

--------------------------------------------------------------------------------
-- 4. FUNCIONES DE LIMPIEZA TOTAL Y TOGGLES
--------------------------------------------------------------------------------
local function clearGroup(key)
	for target, data in pairs(tracked[key]) do
		if data.hl then data.hl:Destroy() end
		if data.bb then data.bb:Destroy() end
		if data.beam then data.beam:Destroy() end
		if data.a0 then data.a0:Destroy() end
	end
	table.clear(tracked[key])
end

local function createSlider(parent, titleText, minVal, maxVal, defaultVal, isInteger, callback)
	local holder = Instance.new("Frame", parent)
	holder.Size = UDim2.new(1, 0, 0, 44)
	holder.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
	holder.Selectable = false
	Instance.new("UICorner", holder).CornerRadius = UDim.new(0, 4)

	local titleLabel = Instance.new("TextLabel", holder)
	titleLabel.Size = UDim2.new(0.6, 0, 0, 18)
	titleLabel.Position = UDim2.new(0.04, 0, 0.1, 0)
	titleLabel.BackgroundTransparency = 1
	titleLabel.Text = titleText
	titleLabel.TextColor3 = Color3.fromRGB(200, 200, 215)
	titleLabel.Font = Enum.Font.Code
	titleLabel.TextSize = 11
	titleLabel.TextXAlignment = Enum.TextXAlignment.Left

	local valueLabel = Instance.new("TextLabel", holder)
	valueLabel.Size = UDim2.new(0.34, 0, 0, 18)
	valueLabel.Position = UDim2.new(0.62, 0, 0.1, 0)
	valueLabel.BackgroundTransparency = 1
	valueLabel.Text = isInteger and string.format("%d / %d", math.floor(defaultVal), maxVal) or string.format("%.2f / %.2f", defaultVal, maxVal)
	valueLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
	valueLabel.Font = Enum.Font.Code
	valueLabel.TextSize = 10
	valueLabel.TextXAlignment = Enum.TextXAlignment.Right

	local trackBg = Instance.new("TextButton", holder)
	trackBg.Size = UDim2.new(0.92, 0, 0, 12)
	trackBg.Position = UDim2.new(0.04, 0, 0.6, 0)
	trackBg.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
	trackBg.Text = ""
	trackBg.AutoButtonColor = false
	trackBg.Selectable = true
	Instance.new("UICorner", trackBg).CornerRadius = UDim.new(1, 0)

	local trackFill = Instance.new("Frame", trackBg)
	local initScale = (defaultVal - minVal) / (maxVal - minVal)
	trackFill.Size = UDim2.new(initScale, 0, 1, 0)
	trackFill.BackgroundColor3 = cfg.BaseColor
	trackFill.Selectable = false
	Instance.new("UICorner", trackFill).CornerRadius = UDim.new(1, 0)

	local knob = Instance.new("Frame", trackFill)
	knob.Size = UDim2.new(0, 12, 0, 12)
	knob.Position = UDim2.new(1, -6, 0.5, -6)
	knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	knob.Selectable = false
	Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

	local isSliding = false

	local function updateSlider(inputPos)
		local barAbsPos = trackBg.AbsolutePosition.X
		local barAbsSize = trackBg.AbsoluteSize.X
		local pct = math.clamp((inputPos.X - barAbsPos) / barAbsSize, 0, 1)
		
		trackFill.Size = UDim2.new(pct, 0, 1, 0)
		local rawVal = minVal + pct * (maxVal - minVal)
		local currentVal = isInteger and math.floor(rawVal) or rawVal
		
		valueLabel.Text = isInteger and string.format("%d / %d", currentVal, maxVal) or string.format("%.2f / %.2f", currentVal, maxVal)
		callback(currentVal)
	end

	trackBg.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			isSliding = true
			updateSlider(input.Position)
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if isSliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			updateSlider(input.Position)
		end
	end)

	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			isSliding = false
		end
	end)
end

local function createToggle(parent, labelText, key, scanFunc)
	local holder = Instance.new("Frame", parent)
	holder.Size = UDim2.new(1, 0, 0, 32)
	holder.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
	holder.Selectable = false
	Instance.new("UICorner", holder).CornerRadius = UDim.new(0, 4)

	local label = Instance.new("TextLabel", holder)
	label.Size = UDim2.new(0.7, 0, 1, 0)
	label.Position = UDim2.new(0.04, 0, 0, 0)
	label.BackgroundTransparency = 1
	label.Text = labelText
	label.TextColor3 = Color3.fromRGB(200, 200, 215)
	label.Font = Enum.Font.Code
	label.TextSize = 11
	label.TextXAlignment = Enum.TextXAlignment.Left

	local btnOverlay = Instance.new("TextButton", holder)
	btnOverlay.Size = UDim2.new(1, 0, 1, 0)
	btnOverlay.BackgroundTransparency = 1
	btnOverlay.Text = ""
	btnOverlay.Selectable = true

	local switchBg = Instance.new("Frame", holder)
	switchBg.Size = UDim2.new(0, 36, 0, 18)
	switchBg.Position = UDim2.new(1, -42, 0.5, -9)
	switchBg.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
	switchBg.Selectable = false
	Instance.new("UICorner", switchBg).CornerRadius = UDim.new(1, 0)

	local circle = Instance.new("Frame", switchBg)
	circle.Size = UDim2.new(0, 14, 0, 14)
	circle.Position = UDim2.new(0, 2, 0.5, -7)
	circle.BackgroundColor3 = Color3.fromRGB(140, 140, 160)
	circle.Selectable = false
	Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)

	btnOverlay.MouseButton1Click:Connect(function()
		states[key] = not states[key]
		local st = states[key]
		
		TweenService:Create(switchBg, TweenInfo.new(0.2), {BackgroundColor3 = st and cfg.BaseColor or Color3.fromRGB(35, 35, 45)}):Play()
		TweenService:Create(circle, TweenInfo.new(0.2), {Position = st and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)}):Play()

		if st then
			scanFunc()
		else
			clearGroup(key) -- Limpieza inmediata al desactivar
		end
	end)
end

local function createHeaderLabel(parent, text)
	local lbl = Instance.new("TextLabel", parent)
	lbl.Size = UDim2.new(1, 0, 0, 20)
	lbl.BackgroundTransparency = 1
	lbl.Text = "— " .. string.upper(text) .. " —"
	lbl.TextColor3 = Color3.fromRGB(120, 120, 140)
	lbl.Font = Enum.Font.Code
	lbl.TextSize = 10
	lbl.Selectable = false
end

local function createDropdown(parent, titleText, options)
	local dropFrame = Instance.new("Frame", parent)
	dropFrame.Size = UDim2.new(1, 0, 0, 32)
	dropFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 30)
	dropFrame.ClipsDescendants = true
	dropFrame.Selectable = false
	Instance.new("UICorner", dropFrame).CornerRadius = UDim.new(0, 4)

	local btn = Instance.new("TextButton", dropFrame)
	btn.Size = UDim2.new(1, 0, 0, 32)
	btn.BackgroundTransparency = 1
	btn.Text = titleText .. " ▼"
	btn.TextColor3 = Color3.fromRGB(210, 210, 230)
	btn.Font = Enum.Font.Code
	btn.TextSize = 11
	btn.Selectable = true

	local list = Instance.new("Frame", dropFrame)
	list.Size = UDim2.new(1, 0, 0, #options * 26)
	list.Position = UDim2.new(0, 0, 0, 32)
	list.BackgroundTransparency = 1
	list.Selectable = false

	local layout = Instance.new("UIListLayout", list)
	layout.SortOrder = Enum.SortOrder.LayoutOrder

	local isOpen = false
	btn.MouseButton1Click:Connect(function()
		isOpen = not isOpen
		btn.Text = titleText .. (isOpen and " ▲" or " ▼")
		TweenService:Create(dropFrame, TweenInfo.new(0.25), {Size = UDim2.new(1, 0, 0, isOpen and (32 + #options * 26) or 32)}):Play()
	end)

	for _, opt in ipairs(options) do
		local optBtn = Instance.new("TextButton", list)
		optBtn.Size = UDim2.new(1, 0, 0, 26)
		optBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
		optBtn.Text = opt.Name
		optBtn.TextColor3 = Color3.fromRGB(180, 180, 200)
		optBtn.Font = Enum.Font.Code
		optBtn.TextSize = 10
		optBtn.Selectable = true
		
		optBtn.MouseButton1Click:Connect(function()
			opt.Callback()
			isOpen = false
			btn.Text = titleText .. " ▼"
			TweenService:Create(dropFrame, TweenInfo.new(0.2), {Size = UDim2.new(1, 0, 0, 32)}):Play()
		end)
	end
end

--------------------------------------------------------------------------------
-- 5. OPCIONES DE PESTAÑAS Y SETTINGS
--------------------------------------------------------------------------------
createHeaderLabel(ESPContainer, "Entities")
createToggle(ESPContainer, "⬛ Guards ESP", "guards", function() scanGroup("Guards", "guards", Color3.fromRGB(50, 255, 100), "⬛ Guards") end)
createToggle(ESPContainer, "⬜ NPCs ESP", "npcs", function() scanGroup("Npcs", "npcs", Color3.fromRGB(50, 255, 100), "⬜ NPCs") end)
createToggle(ESPContainer, "🔵 Police ESP", "police", function() scanGroup("Police", "police", Color3.fromRGB(50, 255, 100), "🔵 Police") end)
createHeaderLabel(ESPContainer, "Surveillance")
createToggle(ESPContainer, "📷 Security Cameras", "cameras", function() scanCameras() end)

createHeaderLabel(ObjContainer, "Valuables")
createToggle(ObjContainer, "💎 Diamonds (1200)", "diamonds", function() scanObjects("DiamondNecklace", "diamonds", Color3.fromRGB(0, 220, 255), "💎 Diamonds\nPrice: 1200 | Liviano") end)
createToggle(ObjContainer, "🖼️ Masterpiece (7500)", "masterpiece", function() scanObjects("Masterpiece", "masterpiece", Color3.fromRGB(255, 150, 0), "🖼️ Masterpiece\nPrice: 7500 | Pesado") end)
createToggle(ObjContainer, "💻 GamingPC (1000-1300)", "gamingpc", function() scanObjects("GamingPC", "gamingpc", Color3.fromRGB(180, 80, 255), "💻 GamingPC\nPrice: 1000-1300 | Pesado") end)
createHeaderLabel(ObjContainer, "Safes")
createToggle(ObjContainer, "🔐 Safes", "safes", function() scanObjects("Safe", "safes", Color3.fromRGB(50, 255, 120), "🔐 Safe") end)

createHeaderLabel(SetContainer, "Themes & Colors")
createDropdown(SetContainer, "🎨 Select UI Theme", {
	{Name = "Rainbow Dynamic (RGB)", Callback = function() cfg.ThemeMode = "Rainbow" end},
	{Name = "Red / Crimson Tactical", Callback = function() cfg.ThemeMode = "Red"; cfg.BaseColor = Color3.fromRGB(230, 40, 40); cfg.DarkColor = Color3.fromRGB(90, 10, 10); Title.TextColor3 = cfg.BaseColor; MainStroke.Color = cfg.BaseColor end},
	{Name = "Purple Matcha", Callback = function() cfg.ThemeMode = "Purple"; cfg.BaseColor = Color3.fromRGB(160, 80, 255); cfg.DarkColor = Color3.fromRGB(60, 20, 100); Title.TextColor3 = cfg.BaseColor; MainStroke.Color = Color3.fromRGB(45, 45, 60) end},
	{Name = "Pink Matcha", Callback = function() cfg.ThemeMode = "Pink"; cfg.BaseColor = Color3.fromRGB(255, 105, 180); cfg.DarkColor = Color3.fromRGB(120, 30, 80); Title.TextColor3 = cfg.BaseColor; MainStroke.Color = Color3.fromRGB(45, 45, 60) end},
	{Name = "Cyber Cyan", Callback = function() cfg.ThemeMode = "Cyan"; cfg.BaseColor = Color3.fromRGB(0, 220, 255); cfg.DarkColor = Color3.fromRGB(0, 80, 120); Title.TextColor3 = cfg.BaseColor; MainStroke.Color = Color3.fromRGB(45, 45, 60) end}
})

createHeaderLabel(SetContainer, "Interactive Sliders")
createSlider(SetContainer, "ESP Render Distance", 10, 2000, cfg.MaxDistance, true, function(val) cfg.MaxDistance = val end)
createSlider(SetContainer, "Fill Transparency", 0.00, 1.00, cfg.FillTrans, false, function(val) cfg.FillTrans = val end)
createSlider(SetContainer, "Tracer Line Width", 0.01, 0.15, cfg.TracerWidth, false, function(val) cfg.TracerWidth = val end)

createHeaderLabel(SetContainer, "Camera & Tracer Options")
createDropdown(SetContainer, "⚡ Global Tracers Toggle", {
	{Name = "Tracers: Enabled", Callback = function() cfg.TracersEnabled = true end},
	{Name = "Tracers: Disabled", Callback = function() cfg.TracersEnabled = false end}
})
createDropdown(SetContainer, "📷 Camera Tracers Toggle", {
	{Name = "Camera Tracers: OFF", Callback = function() cfg.CameraTracers = false end},
	{Name = "Camera Tracers: ON", Callback = function() cfg.CameraTracers = true end}
})

--------------------------------------------------------------------------------
-- DRAG, RESIZE Y CIERRE
--------------------------------------------------------------------------------
local ResizeGrip = Instance.new("Frame", MainFrame)
ResizeGrip.Size = UDim2.new(0, 12, 0, 12)
ResizeGrip.Position = UDim2.new(1, -12, 1, -12)
ResizeGrip.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
ResizeGrip.Selectable = false
Instance.new("UICorner", ResizeGrip).CornerRadius = UDim.new(0, 3)

local dragging, resizing = false, false
local dragStart, startPos, resizeStart, startSize

Header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true; dragStart = input.Position; startPos = MainFrame.Position
	end
end)

ResizeGrip.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		resizing = true; resizeStart = input.Position; startSize = MainFrame.Size
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	elseif resizing and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - resizeStart
		MainFrame.Size = UDim2.new(0, math.clamp(startSize.X.Offset + delta.X, 300, 450), 0, math.clamp(startSize.Y.Offset + delta.Y, 400, 650))
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false; resizing = false end
end)

CloseBtn.MouseButton1Click:Connect(function()
	for key, _ in pairs(tracked) do
		clearGroup(key)
	end
	ScreenGui:Destroy()
end)

local isMinimized = false
MinimizeBtn.MouseButton1Click:Connect(function()
	isMinimized = not isMinimized
	MainFrame.Size = UDim2.new(0, MainFrame.Size.X.Offset, 0, isMinimized and 32 or 520)
	MinimizeBtn.Text = isMinimized and "+" or "—"
	TopBar.Visible = not isMinimized
	ESPContainer.Visible = not isMinimized and ESPTabBtn.BackgroundColor3 == Color3.fromRGB(36, 36, 48)
	ObjContainer.Visible = not isMinimized and ObjTabBtn.BackgroundColor3 == Color3.fromRGB(36, 36, 48)
	SetContainer.Visible = not isMinimized and SetTabBtn.BackgroundColor3 == Color3.fromRGB(36, 36, 48)
	ResizeGrip.Visible = not isMinimized
end)

--------------------------------------------------------------------------------
-- 6. LÓGICA DE ESP, TRACERS Y ESCANEO DE CÁMARAS
--------------------------------------------------------------------------------
function addESP(target, listKey, color, isDynamic, text)
	if tracked[listKey][target] then return end
	
	-- Selección precisa de la parte principal (Hitbox -> Cabeza -> Base -> PrimaryPart)[cite: 3]
	local root = target:IsA("Model") and (target:FindFirstChild("Hitbox") or target:FindFirstChild("Cabeza") or target:FindFirstChild("Base") or target.PrimaryPart or target:FindFirstChildWhichIsA("BasePart")) or (target:IsA("BasePart") and target)
	if not root then return end

	local hl = Instance.new("Highlight")
	hl.Adornee = target
	hl.FillColor = color
	hl.FillTransparency = cfg.FillTrans
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Parent = target

	local bb = Instance.new("BillboardGui")
	bb.Adornee = root
	bb.Size = UDim2.new(0, 180, 0, 50)
	bb.StudsOffset = Vector3.new(0, 3, 0)
	bb.AlwaysOnTop = true

	local lbl = Instance.new("TextLabel", bb)
	lbl.Size = UDim2.new(1, 0, 1, 0)
	lbl.BackgroundTransparency = 1
	lbl.TextColor3 = color
	lbl.Font = Enum.Font.Code
	lbl.TextSize = 11
	bb.Parent = target

	local a0 = Instance.new("Attachment", Workspace.Terrain)
	local a1 = Instance.new("Attachment", root)
	local beam = Instance.new("Beam", Workspace.Terrain)
	beam.Attachment0 = a0; beam.Attachment1 = a1
	beam.Color = ColorSequence.new(color)
	beam.Width0 = cfg.TracerWidth; beam.Width1 = cfg.TracerWidth
	beam.Transparency = NumberSequence.new(0)
	beam.FaceCamera = true

	tracked[listKey][target] = {hl = hl, bb = bb, lbl = lbl, root = root, dynamic = isDynamic, beam = beam, a0 = a0, text = text, col = color}
end

function scanObjects(name, key, col, text)
	for _, obj in pairs(Workspace:GetDescendants()) do
		if (obj:IsA("Model") or obj:IsA("BasePart")) and obj.Name == name then
			addESP(obj, key, col, true, text)
		end
	end
end

function scanGroup(folderName, key, col, text)
	local folder = Workspace:FindFirstChild(folderName)
	if folder then
		for _, obj in pairs(folder:GetDescendants()) do
			if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") then
				addESP(obj, key, col, true, text)
			end
		end
	end
end

function scanCameras()
	local town = Workspace:FindFirstChild("Town")
	local secCamerasFolder = town and town:FindFirstChild("SecurityCameras") or Workspace:FindFirstChild("SecurityCameras")
	
	if secCamerasFolder then
		for _, obj in pairs(secCamerasFolder:GetChildren()) do
			if obj:IsA("Model") and (obj.Name == "SecurityCamera" or obj.Name:lower():find("camera")) then
				addESP(obj, "cameras", Color3.fromRGB(255, 240, 50), true, "📷 SecurityCamera\n[Active]")
			end
		end
	end
end

-- LOOP DE RENDIMIENTO
RunService.RenderStepped:Connect(function()
	if not ScreenGui.Parent then return end
	local char = LocalPlayer.Character
	local pRoot = char and char:FindFirstChild("HumanoidRootPart")
	local bottomScreenPos = Camera.CFrame.Position - (Camera.CFrame.UpVector * 2.5)

	for key, isAct in pairs(states) do
		if isAct then
			if next(tracked[key]) == nil then
				if key == "diamonds" then scanObjects("DiamondNecklace", "diamonds", Color3.fromRGB(0, 220, 255), "💎 Diamonds\nPrice: 1200 | Liviano")
				elseif key == "masterpiece" then scanObjects("Masterpiece", "masterpiece", Color3.fromRGB(255, 150, 0), "🖼️ Masterpiece\nPrice: 7500 | Pesado")
				elseif key == "gamingpc" then scanObjects("GamingPC", "gamingpc", Color3.fromRGB(180, 80, 255), "💻 GamingPC\nPrice: 1000-1300 | Pesado")
				elseif key == "safes" then scanObjects("Safe", "safes", Color3.fromRGB(50, 255, 120), "🔐 Safe")
				elseif key == "guards" then scanGroup("Guards", "guards", Color3.fromRGB(50, 255, 100), "⬛ Guards")
				elseif key == "npcs" then scanGroup("Npcs", "npcs", Color3.fromRGB(50, 255, 100), "⬜ NPCs")
				elseif key == "police" then scanGroup("Police", "police", Color3.fromRGB(50, 255, 100), "🔵 Police")
				elseif key == "cameras" then scanCameras() end
			end

			for target, data in pairs(tracked[key]) do
				if target and target.Parent and data.root and data.root.Parent then
					if pRoot then
						local dist = math.floor((pRoot.Position - data.root.Position).Magnitude)

						if dist <= cfg.MaxDistance then
							data.hl.Enabled = true
							data.bb.Enabled = true

							-- Control de Tracers (Global vs Cámara Opcional)
							local shouldShowTracer = cfg.TracersEnabled
							if key == "cameras" and not cfg.CameraTracers then
								shouldShowTracer = false
							end

							if data.beam and data.a0 then
								data.beam.Enabled = shouldShowTracer
								data.a0.WorldPosition = bottomScreenPos
								data.beam.Width0 = cfg.TracerWidth
								data.beam.Width1 = cfg.TracerWidth
								data.beam.Color = (cfg.TracerColorMode == "Rainbow" and ColorSequence.new(Color3.fromHSV(rainbowHue, 0.9, 1))) or ColorSequence.new(data.col)
							end

							data.hl.FillTransparency = cfg.FillTrans
							data.lbl.Text = data.text .. "\n[" .. dist .. "m]"

							-- Raycast para ver si se asoma un poquito
							local rayParams = RaycastParams.new()
							rayParams.FilterType = Enum.RaycastFilterType.Exclude
							rayParams.FilterDescendantsInstances = {char, target}
							rayParams.IgnoreWater = true
							
							local res = Workspace:Raycast(Camera.CFrame.Position, data.root.Position - Camera.CFrame.Position, rayParams)

							if not res or (res.Position - data.root.Position).Magnitude < 3 then
								data.hl.FillColor = data.col
								data.lbl.TextColor3 = data.col
							else
								data.hl.FillColor = Color3.fromRGB(255, 50, 50)
								data.lbl.TextColor3 = Color3.fromRGB(255, 80, 80)
							end
						else
							data.hl.Enabled = false
							data.bb.Enabled = false
							if data.beam then data.beam.Enabled = false end
						end
					end
				else
					if data.hl then data.hl:Destroy() end
					if data.bb then data.bb:Destroy() end
					if data.beam then data.beam:Destroy() end
					if data.a0 then data.a0:Destroy() end
					tracked[key][target] = nil
				end
			end
		end
	end
end)
