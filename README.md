-- LocalScript: SCRIPT GRÁFICO - HG System (Completo & Corrigido)
local Players, Lighting, TweenService, UserInputService, Workspace, HttpService, RunService = 
	game:GetService("Players"), game:GetService("Lighting"), game:GetService("TweenService"), 
	game:GetService("UserInputService"), game:GetService("Workspace"), game:GetService("HttpService"), game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Arquivos de Armazenamento Local
local SAVE_FILE_NAME = "HG_Graphics_Preset.json"
local KEY_FILE_NAME = "HG_Key_Storage_v2.json"

-- Chaves de Acesso e URL do Gerador
local MASTER_KEY = "HG-GRAPHICS-2026" -- Key Mestre
local GET_KEY_URL = "https://hg-script-keys.netlify.app"
local TEST_DURATION = 86400 -- 24 horas em segundos

---------------------------------------------------------
-- SISTEMA DE VALIDAÇÃO E MEMÓRIA DE KEY POR DISPOSITIVO
---------------------------------------------------------

local function getKeyData()
	local success, result = pcall(function()
		if readfile and isfile and isfile(KEY_FILE_NAME) then
			return HttpService:JSONDecode(readfile(KEY_FILE_NAME))
		end
	end)
	return (success and type(result) == "table") and result or {}
end

local function saveKeyData(data)
	pcall(function()
		if writefile then
			writefile(KEY_FILE_NAME, HttpService:JSONEncode(data))
		end
	end)
end

local function clearKeyData()
	pcall(function()
		if delfile and isfile and isfile(KEY_FILE_NAME) then
			delfile(KEY_FILE_NAME)
		else
			saveKeyData({})
		end
	end)
end

local function validateKey(inputKey)
	local cleanKey = string.gsub(inputKey, "%s+", "")
	local currentTime = os.time()
	local keyData = getKeyData()

	if cleanKey == MASTER_KEY then
		keyData.SavedKey = cleanKey
		keyData.ExpireTime = nil
		saveKeyData(keyData)
		return true, "Master Key Aceita! Acesso Total Concedido."
	end

	if string.len(cleanKey) >= 20 then
		if keyData.SavedKey == cleanKey and keyData.ExpireTime then
			if currentTime < keyData.ExpireTime then
				local remainingHours = math.ceil((keyData.ExpireTime - currentTime) / 3600)
				return true, "Key válida! (" .. remainingHours .. "h restantes neste dispositivo)."
			else
				clearKeyData()
				return false, "Esta Key expirou após 24 horas neste dispositivo."
			end
		end

		keyData.SavedKey = cleanKey
		keyData.ExpireTime = currentTime + TEST_DURATION
		saveKeyData(keyData)
		return true, "Key ativada com sucesso! Válida por 24 horas."
	end

	return false, "Key inválida! Gere uma nova key no site."
end

---------------------------------------------------------
-- INICIALIZAÇÃO DO MENU PRINCIPAL DE GRÁFICOS
---------------------------------------------------------

local startMainGraphicsScript

local function checkSavedKeyAndStart()
	local keyData = getKeyData()
	if keyData.SavedKey then
		if keyData.SavedKey == MASTER_KEY then
			startMainGraphicsScript()
			return true
		elseif keyData.ExpireTime then
			if os.time() < keyData.ExpireTime then
				startMainGraphicsScript()
				return true
			else
				clearKeyData()
			end
		end
	end
	return false
end

---------------------------------------------------------
-- INTERFACE DE KEY COM BOTÃO "GET KEY"
---------------------------------------------------------

local function showKeyUI()
	local keyScreenGui = Instance.new("ScreenGui")
	keyScreenGui.Name = "HG_KeySystem"
	keyScreenGui.ResetOnSpawn = false
	keyScreenGui.Parent = playerGui

	local keyFrame = Instance.new("Frame", keyScreenGui)
	keyFrame.Size = UDim2.new(0, 0, 0, 0)
	keyFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
	keyFrame.AnchorPoint = Vector2.new(0.5, 0.5)
	keyFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	keyFrame.ClipsDescendants = true
	Instance.new("UICorner", keyFrame).CornerRadius = UDim.new(0, 14)

	local keyStroke = Instance.new("UIStroke", keyFrame)
	keyStroke.Color = Color3.fromRGB(160, 32, 255)
	keyStroke.Thickness = 2

	local keyTitle = Instance.new("TextLabel", keyFrame)
	keyTitle.Size = UDim2.new(1, 0, 0, 40)
	keyTitle.BackgroundTransparency = 1
	keyTitle.Text = "SCRIPT GRÁFICO - Acesso"
	keyTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
	keyTitle.Font = Enum.Font.GothamBold
	keyTitle.TextSize = 15

	local keyInput = Instance.new("TextBox", keyFrame)
	keyInput.Size = UDim2.new(1, -40, 0, 38)
	keyInput.Position = UDim2.new(0, 20, 0, 48)
	keyInput.BackgroundColor3 = Color3.fromRGB(10, 10, 12)
	keyInput.PlaceholderText = "Cole sua Key aqui..."
	keyInput.Text = ""
	keyInput.TextColor3 = Color3.fromRGB(255, 255, 255)
	keyInput.Font = Enum.Font.GothamBold
	keyInput.TextSize = 12
	Instance.new("UICorner", keyInput).CornerRadius = UDim.new(0, 8)

	local inputStroke = Instance.new("UIStroke", keyInput)
	inputStroke.Color = Color3.fromRGB(40, 40, 50)
	inputStroke.Thickness = 1

	local verifyBtn = Instance.new("TextButton", keyFrame)
	verifyBtn.Size = UDim2.new(1, -40, 0, 35)
	verifyBtn.Position = UDim2.new(0, 20, 0, 94)
	verifyBtn.BackgroundColor3 = Color3.fromRGB(35, 10, 55)
	verifyBtn.Text = "ENTRAR"
	verifyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	verifyBtn.Font = Enum.Font.GothamBold
	verifyBtn.TextSize = 12
	Instance.new("UICorner", verifyBtn).CornerRadius = UDim.new(0, 8)

	local btnStroke = Instance.new("UIStroke", verifyBtn)
	btnStroke.Color = Color3.fromRGB(160, 32, 255)
	btnStroke.Thickness = 1

	local getKeyBtn = Instance.new("TextButton", keyFrame)
	getKeyBtn.Size = UDim2.new(1, -40, 0, 32)
	getKeyBtn.Position = UDim2.new(0, 20, 0, 135)
	getKeyBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 12)
	getKeyBtn.Text = "OBTER KEY (GET KEY)"
	getKeyBtn.TextColor3 = Color3.fromRGB(160, 32, 255)
	getKeyBtn.Font = Enum.Font.GothamBold
	getKeyBtn.TextSize = 11
	Instance.new("UICorner", getKeyBtn).CornerRadius = UDim.new(0, 8)
	
	local getKeyStroke = Instance.new("UIStroke", getKeyBtn)
	getKeyStroke.Color = Color3.fromRGB(80, 40, 120)
	getKeyStroke.Thickness = 1

	local keyStatus = Instance.new("TextLabel", keyFrame)
	keyStatus.Size = UDim2.new(1, -40, 0, 20)
	keyStatus.Position = UDim2.new(0, 20, 0, 172)
	keyStatus.BackgroundTransparency = 1
	keyStatus.Text = ""
	keyStatus.TextColor3 = Color3.fromRGB(255, 255, 255)
	keyStatus.Font = Enum.Font.GothamBold
	keyStatus.TextSize = 10

	TweenService:Create(keyFrame, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
		Size = UDim2.new(0, 320, 0, 205)
	}):Play()

	getKeyBtn.MouseButton1Click:Connect(function()
		pcall(function()
			if setclipboard then
				setclipboard(GET_KEY_URL)
				keyStatus.Text = "Link copiado para a área de transferência!"
				keyStatus.TextColor3 = Color3.fromRGB(100, 200, 255)
			else
				keyStatus.Text = "Acesse: " .. GET_KEY_URL
				keyStatus.TextColor3 = Color3.fromRGB(100, 200, 255)
			end
		end)
	end)

	verifyBtn.MouseButton1Click:Connect(function()
		local isValid, msg = validateKey(keyInput.Text)
		keyStatus.Text = msg

		if isValid then
			keyStatus.TextColor3 = Color3.fromRGB(120, 255, 150)
			TweenService:Create(keyStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(120, 255, 150)}):Play()
			TweenService:Create(btnStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(120, 255, 150)}):Play()
			TweenService:Create(verifyBtn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(20, 80, 40)}):Play()

			task.wait(0.4)

			local closeTween = TweenService:Create(keyFrame, TweenInfo.new(0.25, Enum.EasingStyle.Exponential, Enum.EasingDirection.In), {
				Size = UDim2.new(0, 0, 0, 0)
			})
			closeTween:Play()
			closeTween.Completed:Wait()
			keyScreenGui:Destroy()

			startMainGraphicsScript()
		else
			keyStatus.TextColor3 = Color3.fromRGB(255, 90, 90)
			local origPos = UDim2.new(0.5, 0, 0.5, 0)
			TweenService:Create(keyFrame, TweenInfo.new(0.05), {Position = origPos + UDim2.new(0, -8, 0, 0)}):Play()
			task.wait(0.05)
			TweenService:Create(keyFrame, TweenInfo.new(0.05), {Position = origPos + UDim2.new(0, 8, 0, 0)}):Play()
			task.wait(0.05)
			TweenService:Create(keyFrame, TweenInfo.new(0.05), {Position = origPos}):Play()
		end
	end)
end

startMainGraphicsScript = function()
	local ACCENT_COLORS = {
		{Name = "Roxo Neon", Color = Color3.fromRGB(160, 32, 255), DarkColor = Color3.fromRGB(35, 10, 55)},
		{Name = "Azul Ciano", Color = Color3.fromRGB(0, 210, 255), DarkColor = Color3.fromRGB(5, 45, 65)},
		{Name = "Verde Neon", Color = Color3.fromRGB(50, 255, 120), DarkColor = Color3.fromRGB(10, 55, 25)},
		{Name = "Laranja Chama", Color = Color3.fromRGB(255, 100, 30), DarkColor = Color3.fromRGB(55, 20, 5)},
		{Name = "Rosa Chiclete", Color = Color3.fromRGB(255, 60, 160), DarkColor = Color3.fromRGB(55, 10, 35)},
		{Name = "Amarelo Ouro", Color = Color3.fromRGB(255, 210, 40), DarkColor = Color3.fromRGB(55, 45, 5)},
		{Name = "Vermelho Rubi", Color = Color3.fromRGB(255, 35, 50), DarkColor = Color3.fromRGB(55, 8, 12)},
		{Name = "Azul Elétrico", Color = Color3.fromRGB(30, 110, 255), DarkColor = Color3.fromRGB(8, 25, 55)}
	}

	local BG_COLORS = {
		{Name = "Preto OLED", Color = Color3.fromRGB(0, 0, 0)},
		{Name = "Escuro Absoluto", Color = Color3.fromRGB(5, 5, 8)},
		{Name = "Grafite Moderno", Color = Color3.fromRGB(18, 18, 22)},
		{Name = "Azul Noturno", Color = Color3.fromRGB(8, 12, 20)},
		{Name = "Cinza Escuro", Color = Color3.fromRGB(25, 25, 30)}
	}

	local currentAccent = ACCENT_COLORS[1]
	local currentBg = BG_COLORS[1]
	local CONTAINER_BG = Color3.fromRGB(10, 10, 14)

	local SKYBOX_LIST = {
		{Name = "Skybox 1", Bk = "rbxassetid://134717391832839", Dn = "rbxassetid://138320691207438", Ft = "rbxassetid://81506174447121", Lf = "rbxassetid://125698421172830", Rt = "rbxassetid://91324281705923", Up = "rbxassetid://73134102008280"},
		{Name = "Skybox 2", Bk = "rbxassetid://101159783687825", Dn = "rbxassetid://92913988053256", Ft = "rbxassetid://116864531179800", Lf = "rbxassetid://75247946852063", Rt = "rbxassetid://93712932926297", Up = "rbxassetid://98207357187404"},
		{Name = "Skybox 3", Bk = "rbxassetid://127175348355839", Dn = "rbxassetid://128653521173042", Ft = "rbxassetid://128822567226853", Lf = "rbxassetid://112330209086791", Rt = "rbxassetid://83810622307483", Up = "rbxassetid://87882399714583"}
	}

	local AA_LEVELS = {
		{Name = "Desativado", ContrastMod = 0, Specular = 0.2},
		{Name = "FXAA (Nível 1)", ContrastMod = 0.02, Specular = 0.4},
		{Name = "MSAA x2 (Nível 2)", ContrastMod = 0.04, Specular = 0.6},
		{Name = "MSAA x4 (Nível 3)", ContrastMod = 0.06, Specular = 0.8},
		{Name = "SMAA / Ultra", ContrastMod = 0.08, Specular = 1.0}
	}

	local DLSS2_MODES = {
		{Name = "Desativado", Sharpness = 0, ExposureOffset = 0, SpecularMult = 1.0, ContrastBoost = 0},
		{Name = "DLSS 2.0 - Qualidade", Sharpness = 0.35, ExposureOffset = 0.08, SpecularMult = 1.3, ContrastBoost = 0.08},
		{Name = "DLSS 2.0 - Ultra", Sharpness = 0.50, ExposureOffset = 0.12, SpecularMult = 1.5, ContrastBoost = 0.12}
	}

	local keybindMenu = Enum.KeyCode.V
	local keybindLoad = Enum.KeyCode.T
	local keybindReset = Enum.KeyCode.Y

	local screenGui = Instance.new("ScreenGui") 
	screenGui.Name = "HG_GraphicsMenu" 
	screenGui.ResetOnSpawn = false 
	screenGui.Parent = playerGui

	local mobileToggleBtn = Instance.new("ImageButton")
	mobileToggleBtn.Size, mobileToggleBtn.Position = UDim2.new(0, 48, 0, 48), UDim2.new(0.02, 0, 0.4, 0)
	mobileToggleBtn.BackgroundColor3 = currentBg.Color
	mobileToggleBtn.Image = "rbxassetid://120859596919883"
	mobileToggleBtn.Active, mobileToggleBtn.Draggable = true, true
	mobileToggleBtn.Parent = screenGui
	Instance.new("UICorner", mobileToggleBtn).CornerRadius = UDim.new(0, 10)
	local mStroke = Instance.new("UIStroke", mobileToggleBtn) 
	mStroke.Color, mStroke.Thickness = currentAccent.Color, 2

	local mainGroup = Instance.new("Frame")
	mainGroup.Size, mainGroup.Position, mainGroup.AnchorPoint = UDim2.new(0, 0, 0, 0), UDim2.new(0.5, 0, 0.5, 0), Vector2.new(0.5, 0.5)
	mainGroup.BackgroundColor3, mainGroup.BorderSizePixel, mainGroup.ClipsDescendants = currentBg.Color, 0, true
	mainGroup.Visible = true
	mainGroup.Parent = screenGui

	Instance.new("UICorner", mainGroup).CornerRadius = UDim.new(0, 16)
	local mainStroke = Instance.new("UIStroke", mainGroup) 
	mainStroke.Color, mainStroke.Thickness = currentAccent.Color, 2

	local uiConstraint = Instance.new("UIAspectRatioConstraint", mainGroup) uiConstraint.AspectRatio = 1.33
	local uiSize = Instance.new("UISizeConstraint", mainGroup) uiSize.MinSize, uiSize.MaxSize = Vector2.new(340, 420), Vector2.new(540, 620)

	local isOpen, isAnimating = false, false
	local function toggleMenu()
		if isAnimating then return end
		isAnimating = true

		if isOpen then
			local tween = TweenService:Create(mainGroup, TweenInfo.new(0.2, Enum.EasingStyle.Exponential, Enum.EasingDirection.Out), {Size = UDim2.new(0, 0, 0, 0)})
			tween:Play()
			tween.Completed:Connect(function()
				mainGroup.Visible = false
				isOpen, isAnimating = false, false
			end)
		else
			mainGroup.Visible = true
			mainGroup.Size = UDim2.new(0, 0, 0, 0)
			local tween = TweenService:Create(mainGroup, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0.44, 0, 0.72, 0)})
			tween:Play()
			tween.Completed:Connect(function()
				isOpen, isAnimating = true, false
			end)
		end
	end

	toggleMenu()
	mobileToggleBtn.MouseButton1Click:Connect(toggleMenu)

	local header = Instance.new("Frame", mainGroup) 
	header.Size, header.Position, header.BackgroundTransparency = UDim2.new(1, -24, 0, 45), UDim2.new(0, 12, 0, 10), 1

	local t2 = Instance.new("TextLabel", header) 
	t2.Size, t2.Position, t2.BackgroundTransparency, t2.Text, t2.TextColor3, t2.Font, t2.TextSize = UDim2.new(1, -40, 0, 22), UDim2.new(0, 0, 0, 0), 1, "SCRIPT GRÁFICO", currentAccent.Color, Enum.Font.GothamBold, 16

	local expiryLabel = Instance.new("TextLabel", header)
	expiryLabel.Size, expiryLabel.Position, expiryLabel.BackgroundTransparency = UDim2.new(1, -40, 0, 16), UDim2.new(0, 0, 0, 22), 1
	expiryLabel.Text = "Sincronizando Key..."
	expiryLabel.TextColor3 = Color3.fromRGB(220, 225, 240)
	expiryLabel.Font = Enum.Font.GothamBold
	expiryLabel.TextSize = 11
	expiryLabel.TextXAlignment = Enum.TextXAlignment.Left

	task.spawn(function()
		while task.wait(1) do
			local keyData = getKeyData()
			if keyData.SavedKey == MASTER_KEY then
				expiryLabel.Text = "Status: Master Key (Ilimitada)"
				expiryLabel.TextColor3 = Color3.fromRGB(120, 255, 150)
			elseif keyData.ExpireTime then
				local timeLeft = keyData.ExpireTime - os.time()
				if timeLeft > 0 then
					local hours = math.floor(timeLeft / 3600)
					local minutes = math.floor((timeLeft % 3600) / 60)
					local seconds = timeLeft % 60
					expiryLabel.Text = string.format("⚡ Expira em: %02d:%02d:%02d", hours, minutes, seconds)
					expiryLabel.TextColor3 = Color3.fromRGB(255, 215, 80)
				else
					expiryLabel.Text = "⚠️ KEY EXPIRADA! Redirecionando..."
					expiryLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
					clearKeyData()
					task.wait(1.5)
					screenGui:Destroy()
					showKeyUI()
					break
				end
			else
				expiryLabel.Text = "Status: Sem Key ativa"
				expiryLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
			end
		end
	end)

	local closeBtn = Instance.new("TextButton", header) 
	closeBtn.Size, closeBtn.Position, closeBtn.BackgroundColor3, closeBtn.Text, closeBtn.TextColor3, closeBtn.Font = UDim2.new(0, 32, 0, 32), UDim2.new(1, -32, 0.5, -16), Color3.fromRGB(10, 10, 14), "✕", Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold
	Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)
	closeBtn.MouseButton1Click:Connect(toggleMenu)

	local tabFrame = Instance.new("Frame", mainGroup) 
	tabFrame.Size, tabFrame.Position, tabFrame.BackgroundTransparency = UDim2.new(1, -24, 0, 36), UDim2.new(0, 12, 0, 60), 1

	local tabLayout = Instance.new("UIListLayout", tabFrame) 
	tabLayout.FillDirection, tabLayout.Padding = Enum.FillDirection.Horizontal, UDim.new(0, 4)

	local function createTab(text)
		local btn = Instance.new("TextButton", tabFrame) 
		btn.Size, btn.BackgroundColor3, btn.Text, btn.TextColor3, btn.Font, btn.TextSize = UDim2.new(0.165, -3, 1, 0), CONTAINER_BG, text, Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 8
		Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
		local s = Instance.new("UIStroke", btn) 
		s.Color, s.Thickness, s.Transparency = currentAccent.Color, 1, 0.8
		return btn, s
	end

	local geralBtn, gS = createTab("GERAL") 
	local rtxBtn, rtxS = createTab("RTX / DLSS")
	local coresBtn, cS = createTab("CORES") 
	local skyBtn, skyS = createTab("SKYBOX")
	local customBtn, custS = createTab("PERSONALIZAR")
	local savesBtn, sS = createTab("SAVES")

	local function createScroll()
		local sc = Instance.new("ScrollingFrame", mainGroup) 
		sc.Size, sc.Position, sc.BackgroundTransparency, sc.BorderSizePixel, sc.ScrollBarThickness, sc.ScrollBarImageColor3, sc.Visible = UDim2.new(1, -24, 1, -110), UDim2.new(0, 12, 0, 102), 1, 0, 4, currentAccent.Color, false
		
		local l = Instance.new("UIListLayout", sc) 
		l.SortOrder, l.Padding = Enum.SortOrder.LayoutOrder, UDim.new(0, 8)
		
		l:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
			sc.CanvasSize = UDim2.new(0, 0, 0, l.AbsoluteContentSize.Y + 20)
		end)
		
		return sc
	end

	local geralScroll, rtxScroll, coresScroll, skyScroll, customScroll, savesScroll = createScroll(), createScroll(), createScroll(), createScroll(), createScroll(), createScroll()

	local tabs = {
		{geralBtn, gS, geralScroll},
		{rtxBtn, rtxS, rtxScroll},
		{coresBtn, cS, coresScroll},
		{skyBtn, skyS, skyScroll},
		{customBtn, custS, customScroll},
		{savesBtn, sS, savesScroll}
	}
	local dynamicStrokes, dynamicTexts = {mainStroke, mStroke}, {t2}
	local activeToggles = {}
	
	for _, t in ipairs(tabs) do table.insert(dynamicStrokes, t[2]) end

	local activeTab = geralBtn
	local function switchTab(target)
		activeTab = target
		for _, t in ipairs(tabs) do
			local active = (t[1] == target)
			t[3].Visible = active
			TweenService:Create(t[1], TweenInfo.new(0.15), {BackgroundColor3 = active and currentAccent.DarkColor or CONTAINER_BG}):Play()
			TweenService:Create(t[2], TweenInfo.new(0.15), {Transparency = active and 0 or 0.8}):Play()
		end
	end
	for _, t in ipairs(tabs) do t[1].MouseButton1Click:Connect(function() switchTab(t[1]) end) end

	local function applyTheme(accentObj, bgObj)
		if accentObj then currentAccent = accentObj end
		if bgObj then currentBg = bgObj end

		mainGroup.BackgroundColor3 = currentBg.Color
		mobileToggleBtn.BackgroundColor3 = currentBg.Color

		for _, s in ipairs(dynamicStrokes) do s.Color = currentAccent.Color end
		for _, txt in ipairs(dynamicTexts) do txt.TextColor3 = currentAccent.Color end

		for _, toggleData in ipairs(activeToggles) do toggleData.RefreshTheme() end
		switchTab(activeTab)
	end

	local CC = Lighting:FindFirstChildOfClass("ColorCorrectionEffect") or Instance.new("ColorCorrectionEffect", Lighting)
	local sunRays = Lighting:FindFirstChildOfClass("SunRaysEffect") or Instance.new("SunRaysEffect", Lighting)
	local bloom = Lighting:FindFirstChildOfClass("BloomEffect") or Instance.new("BloomEffect", Lighting)
	local dof = Lighting:FindFirstChildOfClass("DepthOfFieldEffect") or Instance.new("DepthOfFieldEffect", Lighting)

	dof.FocusDistance, dof.InFocusRadius, dof.NearIntensity, dof.FarIntensity, dof.Enabled = 20, 25, 0.1, 0.5, false
	sunRays.Enabled = false

	local cachedLights = {}
	local settingsState, uiElements = {
		RTXMode=false, Shadows=false, SunRays=false, SunRaysIntensity=0.15, Reflections=false, GlassReflections=false, VehicleReflections=false, RemoveBlur=false, HighDefinition=false, Saturation=0, Contrast=0, Sharpness=0,
		SelectedSkybox=0, TimeOfDay=12, CustomTimeEnabled=false, AntiAliasingLevel=1, BackgroundBlur=false, BlurIntensity=0.5, Exposure=0, Highlights=0, LightIntensity=0, NullHighlight=false, AccentIndex=1, BgIndex=1,
		DLSS1Mode=false, DLSS2Mode=1, DLSS3Mode=false
	}, {}

	local function applyAntiAliasing()
		local aaData = AA_LEVELS[settingsState.AntiAliasingLevel or 1] or AA_LEVELS[1]
		local dlss2Data = DLSS2_MODES[settingsState.DLSS2Mode or 1] or DLSS2_MODES[1]
		
		if settingsState.AntiAliasingLevel > 1 or settingsState.DLSS1Mode or settingsState.DLSS2Mode > 1 or settingsState.DLSS3Mode then
			Lighting.Technology = Enum.Technology.Future
			local hlVal = settingsState.NullHighlight and 0 or (settingsState.Highlights or 0)
			local dlss1Bonus = settingsState.DLSS1Mode and 1.5 or 1.0
			local dlss3Bonus = settingsState.DLSS3Mode and 1.8 or 1.0
			Lighting.EnvironmentSpecularScale = math.clamp(aaData.Specular * hlVal * dlss2Data.SpecularMult * dlss1Bonus * dlss3Bonus, 0, 5)
		elseif not settingsState.RTXMode then
			Lighting.Technology = Enum.Technology.ShadowMap
		end
	end

	local function updateColorCorrection()
		local rtxContrast = settingsState.RTXMode and 0.08 or 0
		local rtxSat = settingsState.RTXMode and 0.08 or 0
		local sharpnessVal = settingsState.Sharpness or 0
		local aaData = AA_LEVELS[settingsState.AntiAliasingLevel or 1] or AA_LEVELS[1]
		
		local dlss1Sharp = settingsState.DLSS1Mode and 0.35 or 0
		local dlss1Sat = settingsState.DLSS1Mode and 0.12 or 0
		
		local dlss2Data = DLSS2_MODES[settingsState.DLSS2Mode or 1] or DLSS2_MODES[1]
		local dlss3Sharp = settingsState.DLSS3Mode and 0.60 or 0
		local dlss3Sat = settingsState.DLSS3Mode and 0.18 or 0
		
		local hl = settingsState.NullHighlight and 0 or (settingsState.Highlights or 0)
		
		CC.Contrast = settingsState.Contrast + (settingsState.HighDefinition and 0.15 or 0) + rtxContrast + ((sharpnessVal + dlss1Sharp + dlss2Data.Sharpness + dlss3Sharp) * 0.45) + dlss2Data.ContrastBoost + aaData.ContrastMod
		CC.Saturation = settingsState.Saturation + (settingsState.HighDefinition and 0.1 or 0) + rtxSat + dlss1Sat + dlss3Sat + (dlss2Data.ContrastBoost * 0.5)
		Lighting.ExposureCompensation = (settingsState.Exposure or 0) + dlss2Data.ExposureOffset + (settingsState.DLSS1Mode and 0.08 or 0) + (settingsState.DLSS3Mode and 0.15 or 0)
		
		if hl > 0 or settingsState.DLSS2Mode > 1 or settingsState.DLSS1Mode or settingsState.DLSS3Mode then
			bloom.Enabled = true
			local dlssMultiplier = (settingsState.DLSS2Mode > 1 or settingsState.DLSS3Mode) and 1.5 or 1.0
			bloom.Intensity = (hl > 0 and hl or 0.5) * 1.8 * dlssMultiplier
			bloom.Size = 28 + (hl * 18)
			bloom.Threshold = math.clamp(1.8 - (hl * 1.5), 0.05, 2.0)
		else
			bloom.Enabled = false
			bloom.Intensity = 0
		end
	end

	local function updateBlur()
		dof.Enabled = settingsState.BackgroundBlur
		dof.FarIntensity = settingsState.BlurIntensity
		dof.NearIntensity = settingsState.BlurIntensity * 0.3
	end

	local isUpdatingLights = false
	local function updateLightIntensity(multiplier)
		if isUpdatingLights then return end
		isUpdatingLights = true
		
		task.spawn(function()
			if #cachedLights == 0 then
				local instances = Workspace:GetDescendants()
				for i = 1, #instances do
					local item = instances[i]
					if item:IsA("PointLight") or item:IsA("SpotLight") or item:IsA("SurfaceLight") then
						table.insert(cachedLights, {Light = item, Base = item.Brightness})
					end
					if i % 250 == 0 then task.wait() end
				end
			end
			
			for i = 1, #cachedLights do
				local data = cachedLights[i]
				if data.Light and data.Light.Parent then
					data.Light.Brightness = data.Base * multiplier
				end
				if i % 150 == 0 then task.wait() end
			end
			isUpdatingLights = false
		end)
	end

	local function applyVehicleReflections(enabled)
		task.spawn(function()
			local items = Workspace:GetDescendants()
			for i = 1, #items do
				local item = items[i]
				if item:IsA("Model") and (item.Name:lower():find("car") or item.Name:lower():find("vehicle") or item:FindFirstChild("DriveSeat")) then
					local parts = item:GetDescendants()
					for p = 1, #parts do
						local part = parts[p]
						if part:IsA("BasePart") then
							local mat = part.Material
							if mat == Enum.Material.SmoothPlastic or mat == Enum.Material.Metal or mat == Enum.Material.Glass then
								part.Reflectance = enabled and 0.75 or 0
							end
						end
					end
				end
				if i % 200 == 0 then task.wait() end
			end
		end)
	end

	RunService.Heartbeat:Connect(function()
		if settingsState.CustomTimeEnabled then
			if math.abs(Lighting.ClockTime - settingsState.TimeOfDay) > 0.1 then
				Lighting.ClockTime = settingsState.TimeOfDay
			end
		end
	end)

	local function lockCustomTime(hour)
		settingsState.TimeOfDay = hour
		settingsState.CustomTimeEnabled = true
		Lighting.ClockTime = hour
	end

	local skyboxToggles = {}
	local function applySkyboxIndex(index)
		settingsState.SelectedSkybox = index
		for _, child in ipairs(Lighting:GetChildren()) do
			if child:IsA("Sky") then child:Destroy() end
		end
		if index > 0 and SKYBOX_LIST[index] then
			local data = SKYBOX_LIST[index]
			local newSky = Instance.new("Sky")
			newSky.Name = "HG_CustomSky"
			newSky.SkyboxBk, newSky.SkyboxDn, newSky.SkyboxFt = data.Bk, data.Dn, data.Ft
			newSky.SkyboxLf, newSky.SkyboxRt, newSky.SkyboxUp = data.Lf, data.Rt, data.Up
			newSky.SunTextureId, newSky.MoonTextureId = "", ""
			newSky.Parent = Lighting
		end
		for i = 1, #skyboxToggles do
			skyboxToggles[i].SetVisual(i == index)
		end
	end

	local function createToggle(key, text, parentFrame, layoutOrder, callback)
		local container = Instance.new("Frame", parentFrame) 
		container.Size, container.BackgroundColor3, container.LayoutOrder = UDim2.new(1, -6, 0, 42), CONTAINER_BG, layoutOrder
		Instance.new("UICorner", container).CornerRadius = UDim.new(0, 8)
		
		local lbl = Instance.new("TextLabel", container) 
		lbl.Size, lbl.Position, lbl.BackgroundTransparency, lbl.Text, lbl.TextColor3, lbl.Font, lbl.TextSize, lbl.TextXAlignment = UDim2.new(0.75, 0, 1, 0), UDim2.new(0, 12, 0, 0), 1, text, Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 12, Enum.TextXAlignment.Left

		local box = Instance.new("TextButton", container) 
		box.Size, box.Position, box.BackgroundColor3, box.Text = UDim2.new(0, 22, 0, 22), UDim2.new(1, -34, 0.5, -11), Color3.fromRGB(8, 8, 12), ""
		Instance.new("UICorner", box).CornerRadius = UDim.new(0, 5)
		local boxStroke = Instance.new("UIStroke", box) 
		boxStroke.Color, boxStroke.Thickness = Color3.fromRGB(50, 50, 65), 1.5

		local check = Instance.new("TextLabel", box) 
		check.Size, check.BackgroundTransparency, check.Text, check.TextColor3, check.Font, check.TextTransparency = UDim2.new(1, 0, 1, 0), 1, "✓", Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 1

		local active = false
		local function setVisual(st)
			active = st 
			settingsState[key] = active
			TweenService:Create(box, TweenInfo.new(0.15), {BackgroundColor3 = active and currentAccent.Color or Color3.fromRGB(8, 8, 12)}):Play()
			TweenService:Create(boxStroke, TweenInfo.new(0.15), {Color = active and currentAccent.Color or Color3.fromRGB(50, 50, 65)}):Play()
			check.TextTransparency = active and 0 or 1
			if callback then callback(active) end
		end

		table.insert(activeToggles, {
			RefreshTheme = function()
				if active then
					box.BackgroundColor3 = currentAccent.Color
					boxStroke.Color = currentAccent.Color
				end
			end
		})

		box.MouseButton1Click:Connect(function() setVisual(not active) end)
		uiElements[key] = {Set = setVisual}
	end

	local function createSlider(key, text, minV, maxV, defV, parentFrame, layoutOrder, isInteger, callback)
		local container = Instance.new("Frame", parentFrame) 
		container.Size, container.BackgroundColor3, container.LayoutOrder = UDim2.new(1, -6, 0, 52), CONTAINER_BG, layoutOrder
		Instance.new("UICorner", container).CornerRadius = UDim.new(0, 8)
		
		local lbl = Instance.new("TextLabel", container) 
		lbl.Size, lbl.Position, lbl.BackgroundTransparency, lbl.Text, lbl.TextColor3, lbl.Font, lbl.TextSize, lbl.TextXAlignment = UDim2.new(0.6, 0, 0, 22), UDim2.new(0, 12, 0, 4), 1, text, Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 12, Enum.TextXAlignment.Left
		
		local valLbl = Instance.new("TextLabel", container) 
		valLbl.Size, valLbl.Position, valLbl.BackgroundTransparency, valLbl.TextColor3, valLbl.Font, valLbl.TextSize, valLbl.TextXAlignment = UDim2.new(0.3, 0, 0, 22), UDim2.new(0.7, -12, 0, 4), 1, currentAccent.Color, Enum.Font.GothamBold, 12, Enum.TextXAlignment.Right
		table.insert(dynamicTexts, valLbl)

		local track = Instance.new("Frame", container) 
		track.Size, track.Position, track.BackgroundColor3 = UDim2.new(1, -24, 0, 5), UDim2.new(0, 12, 0, 34), Color3.fromRGB(8, 8, 12)
		Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)
		
		local fill = Instance.new("Frame", track) 
		fill.BackgroundColor3 = currentAccent.Color Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

		local knob = Instance.new("Frame", track) 
		knob.Size, knob.BackgroundColor3 = UDim2.new(0, 12, 0, 12), Color3.fromRGB(255, 255, 255) Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

		table.insert(activeToggles, {
			RefreshTheme = function()
				fill.BackgroundColor3 = currentAccent.Color
			end
		})

		local function setVal(v)
			v = math.clamp(v, minV, maxV)
			if isInteger then v = math.floor(v + 0.5) end
			settingsState[key] = v
			local pct = (v - minV) / (maxV - minV)
			fill.Size = UDim2.new(pct, 0, 1, 0)
			knob.Position = UDim2.new(pct, -6, 0.5, -6)
			valLbl.Text = isInteger and string.format("%d", v) or string.format("%.2f", v)
			callback(v)
		end

		local isDrag = false
		local function updateInput(i) setVal(minV + (maxV - minV) * math.clamp((i.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)) end
		track.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then isDrag = true updateInput(i) end end)
		UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then isDrag = false end end)
		UserInputService.InputChanged:Connect(function(i) if isDrag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then updateInput(i) end end)

		setVal(defV)
		uiElements[key] = {Set = setVal}
	end

	local function createSelector(key, text, optionsList, parentFrame, layoutOrder, callback)
		local container = Instance.new("Frame", parentFrame)
		container.Size, container.BackgroundColor3, container.LayoutOrder = UDim2.new(1, -6, 0, 52), CONTAINER_BG, layoutOrder
		Instance.new("UICorner", container).CornerRadius = UDim.new(0, 8)

		local lbl = Instance.new("TextLabel", container)
		lbl.Size, lbl.Position, lbl.BackgroundTransparency, lbl.Text, lbl.TextColor3, lbl.Font, lbl.TextSize, lbl.TextXAlignment = UDim2.new(0.5, 0, 1, 0), UDim2.new(0, 12, 0, 0), 1, text, Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 12, Enum.TextXAlignment.Left

		local btnPrev = Instance.new("TextButton", container)
		btnPrev.Size, btnPrev.Position, btnPrev.BackgroundColor3, btnPrev.Text, btnPrev.TextColor3, btnPrev.Font = UDim2.new(0, 24, 0, 24), UDim2.new(0.5, 0, 0.5, -12), Color3.fromRGB(8, 8, 12), "<", currentAccent.Color, Enum.Font.GothamBold
		Instance.new("UICorner", btnPrev).CornerRadius = UDim.new(0, 6)
		table.insert(dynamicTexts, btnPrev)

		local valLbl = Instance.new("TextLabel", container)
		valLbl.Size, valLbl.Position, valLbl.BackgroundTransparency, valLbl.TextColor3, valLbl.Font, valLbl.TextSize = UDim2.new(0.38, -54, 0, 24), UDim2.new(0.5, 28, 0.5, -12), 1, Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 11

		local btnNext = Instance.new("TextButton", container)
		btnNext.Size, btnNext.Position, btnNext.BackgroundColor3, btnNext.Text, btnNext.TextColor3, btnNext.Font = UDim2.new(0, 24, 0, 24), UDim2.new(1, -30, 0.5, -12), Color3.fromRGB(8, 8, 12), ">", currentAccent.Color, Enum.Font.GothamBold
		Instance.new("UICorner", btnNext).CornerRadius = UDim.new(0, 6)
		table.insert(dynamicTexts, btnNext)

		local currentIndex = 1
		local function setIndex(idx)
			currentIndex = math.clamp(idx, 1, #optionsList)
			settingsState[key] = currentIndex
			valLbl.Text = optionsList[currentIndex].Name
			callback(currentIndex)
		end

		btnPrev.MouseButton1Click:Connect(function() setIndex(currentIndex - 1 < 1 and #optionsList or currentIndex - 1) end)
		btnNext.MouseButton1Click:Connect(function() setIndex(currentIndex + 1 > #optionsList and 1 or currentIndex + 1) end)

		setIndex(1)
		uiElements[key] = {Set = setIndex}
	end

	-- Geral
	createToggle("Shadows", "Sombras Realistas HD", geralScroll, 1, function(e) if not settingsState.RTXMode then Lighting.GlobalShadows = e end end)
	
	-- SUN RAYS CORRIGIDO NO MENU GERAL
	createToggle("SunRays", "🌅 Ativar Sun Rays (Raios de Sol)", geralScroll, 2, function(e) 
		sunRays.Enabled = e 
		if e then
			Lighting.Technology = Enum.Technology.Future
			Lighting.ClockTime = 14.5
		end
	end)
	
	createSlider("SunRaysIntensity", "✨ Intensidade dos Sun Rays", 0.05, 1.0, 0.15, geralScroll, 3, false, function(v) sunRays.Intensity = v end)
	
	createToggle("Reflections", "Reflexos HD no Cenário", geralScroll, 4, function(e) if not settingsState.RTXMode then Lighting.EnvironmentSpecularScale = e and 1 or 0 bloom.Enabled = e end end)
	createToggle("GlassReflections", "Reflexos Ultra em Vidros", geralScroll, 5, function(e)
		task.spawn(function()
			local items = Workspace:GetDescendants()
			for i = 1, #items do
				local p = items[i]
				if p:IsA("BasePart") and (p.Material == Enum.Material.Glass or p.Name:lower():find("glass")) then 
					p.Reflectance = e and 0.85 or 0 
				end
				if i % 300 == 0 then task.wait() end
			end
		end)
	end)
	createToggle("VehicleReflections", "Reflexos nos Veículos HD", geralScroll, 6, function(e) applyVehicleReflections(e) end)
	createToggle("RemoveBlur", "Remover Névoa Distante", geralScroll, 7, function(e) Lighting.FogEnd = e and 1000000 or 10000 end)
	createToggle("HighDefinition", "Melhor Definição / Nitidez", geralScroll, 8, function() updateColorCorrection() end)

	createSelector("AntiAliasingLevel", "Anti-Serrilhamento Ultra", AA_LEVELS, geralScroll, 9, function(level)
		settingsState.AntiAliasingLevel = level
		applyAntiAliasing()
		updateColorCorrection()
	end)

	createToggle("BackgroundBlur", "🔍 Desfoque de Fundo (Depth of Field)", geralScroll, 10, function() updateBlur() end)
	createSlider("BlurIntensity", "Intensidade do Desfoque", 0.1, 1.0, 0.5, geralScroll, 11, false, function() updateBlur() end)
	createSlider("LightIntensity", "💡 Intensidade das Luzes", 0, 5, 0, geralScroll, 12, false, function(v) updateLightIntensity(v) end)

	-- RTX / DLSS
	createToggle("RTXMode", "⚡ RTX + Iluminação Dinâmica (Future)", rtxScroll, 1, function(enabled)
		Lighting.Technology = enabled and Enum.Technology.Future or Enum.Technology.ShadowMap
		Lighting.GlobalShadows = enabled or settingsState.Shadows
		bloom.Enabled = enabled or settingsState.Reflections
		updateColorCorrection()
	end)

	createToggle("DLSS1Mode", "🤖 NVIDIA DLSS 1.0", rtxScroll, 2, function(enabled)
		settingsState.DLSS1Mode = enabled
		applyAntiAliasing()
		updateColorCorrection()
	end)

	createSelector("DLSS2Mode", "🚀 NVIDIA DLSS 2.0", DLSS2_MODES, rtxScroll, 3, function(idx)
		settingsState.DLSS2Mode = idx
		applyAntiAliasing()
		updateColorCorrection()
	end)

	createToggle("DLSS3Mode", "⚡ NVIDIA DLSS 3.0 (Frame Gen)", rtxScroll, 4, function(enabled)
		settingsState.DLSS3Mode = enabled
		applyAntiAliasing()
		updateColorCorrection()
	end)

	-- Cores
	createSlider("Saturation", "Saturação Geral", -1, 1, 0, coresScroll, 1, false, function() updateColorCorrection() end)
	createSlider("Contrast", "Contraste Geral", -1, 1, 0, coresScroll, 2, false, function() updateColorCorrection() end)
	createSlider("Sharpness", "Nitidez da Imagem", 0, 1, 0, coresScroll, 3, false, function() updateColorCorrection() end)
	createSlider("Exposure", "Exposição de Luz", -2, 2, 0, coresScroll, 4, false, function() updateColorCorrection() end)
	createSlider("Highlights", "Realces e Brilho", 0, 3, 0, coresScroll, 5, false, function() 
		applyAntiAliasing()
		updateColorCorrection() 
	end)
	createToggle("NullHighlight", "🚫 Realce Nulo (Desativar Brilhos)", coresScroll, 6, function()
		applyAntiAliasing()
		updateColorCorrection()
	end)

	-- Skybox
	createSlider("TimeOfDay", "🕒 Horário Local (00 - 23h)", 0, 23, 12, skyScroll, 1, true, function(v) lockCustomTime(v) end)

	for idx, skyData in ipairs(SKYBOX_LIST) do
		local container = Instance.new("Frame", skyScroll) 
		container.Size, container.BackgroundColor3, container.LayoutOrder = UDim2.new(1, -6, 0, 42), CONTAINER_BG, idx + 1
		Instance.new("UICorner", container).CornerRadius = UDim.new(0, 8)
		
		local lbl = Instance.new("TextLabel", container) 
		lbl.Size, lbl.Position, lbl.BackgroundTransparency, lbl.Text, lbl.TextColor3, lbl.Font, lbl.TextSize, lbl.TextXAlignment = UDim2.new(0.75, 0, 1, 0), UDim2.new(0, 12, 0, 0), 1, "🌌 " .. skyData.Name, Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 12, Enum.TextXAlignment.Left

		local box = Instance.new("TextButton", container) 
		box.Size, box.Position, box.BackgroundColor3, box.Text = UDim2.new(0, 22, 0, 22), UDim2.new(1, -34, 0.5, -11), Color3.fromRGB(8, 8, 12), ""
		Instance.new("UICorner", box).CornerRadius = UDim.new(1, 0)
		local boxStroke = Instance.new("UIStroke", box) boxStroke.Color, boxStroke.Thickness = Color3.fromRGB(50, 50, 65), 1.5

		local dot = Instance.new("Frame", box) 
		dot.Size, dot.Position, dot.BackgroundColor3, dot.Visible = UDim2.new(0, 10, 0, 10), UDim2.new(0.5, -5, 0.5, -5), Color3.fromRGB(255, 255, 255), false
		Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

		local function setVisual(active)
			TweenService:Create(box, TweenInfo.new(0.15), {BackgroundColor3 = active and currentAccent.Color or Color3.fromRGB(8, 8, 12)}):Play()
			TweenService:Create(boxStroke, TweenInfo.new(0.15), {Color = active and currentAccent.Color or Color3.fromRGB(50, 50, 65)}):Play()
			dot.Visible = active
		end

		box.MouseButton1Click:Connect(function()
			if settingsState.SelectedSkybox == idx then applySkyboxIndex(0) else applySkyboxIndex(idx) end
		end)

		table.insert(skyboxToggles, {SetVisual = setVisual})
	end

	-- Personalização
	createSelector("AccentIndex", "🎨 Cor dos Detalhes", ACCENT_COLORS, customScroll, 1, function(idx)
		applyTheme(ACCENT_COLORS[idx], nil)
	end)

	createSelector("BgIndex", "🖼️ Cor do Fundo do Menu", BG_COLORS, customScroll, 2, function(idx)
		applyTheme(nil, BG_COLORS[idx])
	end)

	-- Saves
	local statusLbl = Instance.new("TextLabel", savesScroll) 
	statusLbl.Size, statusLbl.BackgroundTransparency, statusLbl.Text, statusLbl.TextColor3, statusLbl.Font, statusLbl.LayoutOrder = UDim2.new(1, -6, 0, 22), 1, "Aguardando ação...", Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 1

	local function applyAllSettings(data)
		for k, v in pairs(data) do 
			if k == "SelectedSkybox" then applySkyboxIndex(v or 0)
			elseif k == "TimeOfDay" then lockCustomTime(v or 12)
			elseif k == "AccentIndex" and ACCENT_COLORS[v] then
				settingsState.AccentIndex = v
				if uiElements["AccentIndex"] and uiElements["AccentIndex"].Set then uiElements["AccentIndex"].Set(v) end
				applyTheme(ACCENT_COLORS[v], nil)
			elseif k == "BgIndex" and BG_COLORS[v] then
				settingsState.BgIndex = v
				if uiElements["BgIndex"] and uiElements["BgIndex"].Set then uiElements["BgIndex"].Set(v) end
				applyTheme(nil, BG_COLORS[v])
			elseif uiElements[k] and uiElements[k].Set then uiElements[k].Set(v) end 
		end
	end

	local function saveToFile()
		local success = pcall(function() if writefile then writefile(SAVE_FILE_NAME, HttpService:JSONEncode(settingsState)) end end)
		statusLbl.Text = success and "Preset e personalizações salvos com sucesso!" or "Erro ao salvar o preset."
		statusLbl.TextColor3 = success and Color3.fromRGB(120, 255, 150) or Color3.fromRGB(255, 90, 90)
	end

	local function loadFromFile()
		local success, result = pcall(function()
			if readfile and isfile and isfile(SAVE_FILE_NAME) then return HttpService:JSONDecode(readfile(SAVE_FILE_NAME)) end
		end)
		if success and type(result) == "table" then
			applyAllSettings(result)
			statusLbl.Text = "Preset e personalizações carregados!" statusLbl.TextColor3 = Color3.fromRGB(120, 255, 150)
		else
			statusLbl.Text = "Nenhum preset salvo encontrado." statusLbl.TextColor3 = Color3.fromRGB(255, 180, 80)
		end
	end

	local function resetToDefault()
		settingsState.CustomTimeEnabled = false
		applyAllSettings({
			RTXMode=false, Shadows=false, SunRays=false, SunRaysIntensity=0.15, Reflections=false, GlassReflections=false, VehicleReflections=false, RemoveBlur=false, HighDefinition=false, Saturation=0, Contrast=0, Sharpness=0,
			SelectedSkybox=0, TimeOfDay=12, AntiAliasingLevel=1, BackgroundBlur=false, BlurIntensity=0.5, Exposure=0, Highlights=0, LightIntensity=0, NullHighlight=false, DLSS1Mode=false, DLSS2Mode=1, DLSS3Mode=false
		})
		statusLbl.Text = "Gráficos redefinidos!" statusLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
	end

	local isBindingKey = false
	local function bindKey(keyType, keyBtn)
		if isBindingKey then return end
		isBindingKey = true keyBtn.Text = "Pressione..." keyBtn.TextColor3 = Color3.fromRGB(255, 200, 100)
		local connection
		connection = UserInputService.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.Keyboard then
				if keyType == "Load" then keybindLoad = input.KeyCode keyBtn.Text = "[" .. input.KeyCode.Name .. "]"
				elseif keyType == "Reset" then keybindReset = input.KeyCode keyBtn.Text = "[" .. input.KeyCode.Name .. "]" end
				keyBtn.TextColor3 = currentAccent.Color connection:Disconnect() isBindingKey = false
			end
		end)
	end

	local function createActionBtn(text, col, order, bindType, defaultKeyName, cb)
		local container = Instance.new("Frame", savesScroll) 
		container.Size, container.BackgroundTransparency, container.LayoutOrder = UDim2.new(1, -6, 0, 40), 1, order
		
		local b = Instance.new("TextButton", container) 
		b.Size, b.BackgroundColor3, b.Text, b.TextColor3, b.Font, b.TextSize = UDim2.new(1, 0, 1, 0), col, text, Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 12
		Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
		
		local s = Instance.new("UIStroke", b) s.Color, s.Thickness, s.Transparency = currentAccent.Color, 1, 0.5
		table.insert(dynamicStrokes, s)

		if bindType then
			local keyBtn = Instance.new("TextButton", b) 
			keyBtn.Size, keyBtn.Position, keyBtn.BackgroundTransparency = UDim2.new(0, 85, 1, 0), UDim2.new(1, -90, 0, 0), 1
			keyBtn.Text = "[" .. defaultKeyName .. "]" keyBtn.TextColor3, keyBtn.Font, keyBtn.TextSize = currentAccent.Color, Enum.Font.GothamBold, 11
			keyBtn.TextXAlignment = Enum.TextXAlignment.Right
			table.insert(dynamicTexts, keyBtn)
			keyBtn.MouseButton1Click:Connect(function() bindKey(bindType, keyBtn) end)
		end
		b.MouseButton1Click:Connect(function() cb() end)
	end

	createActionBtn("Salvar Configurações Atuais", Color3.fromRGB(15, 15, 20), 2, nil, nil, saveToFile)
	createActionBtn("Carregar Preset Salvo", Color3.fromRGB(10, 25, 18), 3, "Load", "T", loadFromFile)
	createActionBtn("Redefinir Para Padrão", Color3.fromRGB(25, 10, 15), 4, "Reset", "Y", resetToDefault)

	UserInputService.InputBegan:Connect(function(input, gameProcessed)
		if gameProcessed or isBindingKey then return end
		if input.KeyCode == keybindMenu then toggleMenu()
		elseif input.KeyCode == keybindLoad then loadFromFile()
		elseif input.KeyCode == keybindReset then resetToDefault() end
	end)

	pcall(function()
		if readfile and isfile and isfile(SAVE_FILE_NAME) then
			local savedData = HttpService:JSONDecode(readfile(SAVE_FILE_NAME))
			if savedData.AccentIndex then
				settingsState.AccentIndex = savedData.AccentIndex
				if uiElements["AccentIndex"] and uiElements["AccentIndex"].Set then uiElements["AccentIndex"].Set(savedData.AccentIndex) end
				if ACCENT_COLORS[savedData.AccentIndex] then applyTheme(ACCENT_COLORS[savedData.AccentIndex], nil) end
			end
			if savedData.BgIndex then
				settingsState.BgIndex = savedData.BgIndex
				if uiElements["BgIndex"] and uiElements["BgIndex"].Set then uiElements["BgIndex"].Set(savedData.BgIndex) end
				if BG_COLORS[savedData.BgIndex] then applyTheme(nil, BG_COLORS[savedData.BgIndex]) end
			end
		end
	end)
end

---------------------------------------------------------
-- VERIFICAÇÃO INICIAL AO EXECUTAR
---------------------------------------------------------

if not checkSavedKeyAndStart() then
	showKeyUI()
end
