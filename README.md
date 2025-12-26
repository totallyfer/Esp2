--========================================================--
-- 🖥️ GET UP + PC PROGRESS (XENO)
--========================================================--

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local PC_ENABLED = false

--========================================================--
-- 🧠 GUI
--========================================================--

local gui = Instance.new("ScreenGui")
gui.Name = "PC_GETUP_GUI"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

--========================================================--
-- 🔘 BOTÃO (CANTO SUPERIOR DIREITO)
--========================================================--

local btn = Instance.new("TextButton", gui)
btn.Size = UDim2.new(0,160,0,40)
btn.Position = UDim2.new(1,-170,0,10)
btn.BackgroundColor3 = Color3.fromRGB(25,25,25)
btn.BorderSizePixel = 0
btn.Text = "PC / GET UP: OFF"
btn.TextColor3 = Color3.fromRGB(255,80,80)
btn.Font = Enum.Font.GothamBold
btn.TextScaled = true
btn.Active = true

local corner = Instance.new("UICorner", btn)
corner.CornerRadius = UDim.new(0,10)

--========================================================--
-- 🖥️ PC PROGRESS
--========================================================--

local function corGradiente(p)
	if p >= 1 then return Color3.fromRGB(0,255,0) end
	if p < 0.5 then
		return Color3.fromRGB(255,255*p*2,255*p*2)
	else
		return Color3.fromRGB(255,255-((p-0.5)*2*255),0)
	end
end

local function setupPC(pc)
	if pc:FindFirstChild("ProgressBar") then return end

	local guiB = Instance.new("BillboardGui", pc)
	guiB.Name = "ProgressBar"
	guiB.Size = UDim2.new(0,160,0,16)
	guiB.StudsOffset = Vector3.new(0,2.6,0)
	guiB.AlwaysOnTop = true
	guiB.Enabled = PC_ENABLED

	local bg = Instance.new("Frame", guiB)
	bg.Size = UDim2.new(1,0,1,0)
	bg.BackgroundColor3 = Color3.fromRGB(0,0,0)

	local bar = Instance.new("Frame", bg)
	bar.Size = UDim2.new(0,0,1,0)

	local txt = Instance.new("TextLabel", bg)
	txt.Size = UDim2.new(1,0,1,0)
	txt.BackgroundTransparency = 1
	txt.TextScaled = true
	txt.Font = Enum.Font.SciFi
	txt.TextColor3 = Color3.new(1,1,1)

	local hl = Instance.new("Highlight", pc)
	hl.Name = "ComputerHighlight"
	hl.Enabled = PC_ENABLED

	local save = 0

	RunService.Heartbeat:Connect(function()
		if not PC_ENABLED then return end

		local highest = 0
		for _,p in ipairs(pc:GetChildren()) do
			if p:IsA("BasePart") and p.Name:match("ComputerTrigger") then
				for _,t in ipairs(p:GetTouchingParts()) do
					local pl = Players:GetPlayerFromCharacter(t.Parent)
					if pl then
						local m = pl:FindFirstChild("TempPlayerStatsModule")
						if m and not m.Ragdoll.Value then
							highest = math.max(highest, m.ActionProgress.Value)
						end
					end
				end
			end
		end

		save = math.max(save, highest)
		bar.Size = UDim2.new(save,0,1,0)
		bar.BackgroundColor3 = corGradiente(save)

		if save >= 1 then
			txt.Text = "COMPLETED"
			hl.FillColor = Color3.fromRGB(0,255,0)
		else
			txt.Text = math.floor(save*100).."%"
		end
	end)
end

task.spawn(function()
	while true do
		local map = Workspace:FindFirstChild(tostring(ReplicatedStorage.CurrentMap.Value))
		if map then
			for _,o in ipairs(map:GetChildren()) do
				if o.Name == "ComputerTable" then
					setupPC(o)
				end
			end
		end
		task.wait(1)
	end
end)

--========================================================--
-- 🔁 GET UP
--========================================================--

local getupGui = Instance.new("ScreenGui", gui)
getupGui.Enabled = false
getupGui.ResetOnSpawn = false

local lista = Instance.new("Frame", getupGui)
lista.Size = UDim2.new(0,250,0,400)
lista.Position = UDim2.new(1,-20,1,-20)
lista.AnchorPoint = Vector2.new(1,1)
lista.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", lista)
layout.VerticalAlignment = Enum.VerticalAlignment.Bottom
layout.Padding = UDim.new(0,5)

local function gradiente(p)
	if p >= 1 then return Color3.fromRGB(0,255,0) end
	if p > 0.5 then
		local t = (p-0.5)*2
		return Color3.fromRGB(255*(1-t),255,0)
	else
		local t = p*2
		return Color3.fromRGB(255,255*t,0)
	end
end

local function criarGetUp(plr)
	local lbl = Instance.new("TextLabel", lista)
	lbl.Size = UDim2.new(1,0,0,35)
	lbl.BackgroundTransparency = 1
	lbl.TextScaled = true
	lbl.Font = Enum.Font.GothamBold
	lbl.TextXAlignment = Enum.TextXAlignment.Right
	lbl.TextStrokeTransparency = 0.4

	local inicio = tick()
	local duracao = 28

	local conn
	conn = RunService.RenderStepped:Connect(function()
		lbl.Visible = PC_ENABLED
		local restante = duracao - (tick()-inicio)
		local progress = math.clamp(restante/duracao,0,1)

		if restante > 0 then
			lbl.Text = string.format("%s - %.2fs", plr.Name, restante)
			lbl.TextColor3 = gradiente(progress)
		else
			lbl.Text = plr.Name.." levantou!"
			lbl.TextColor3 = Color3.fromRGB(0,200,255)
			conn:Disconnect()
			task.delay(2,function()
				lbl:Destroy()
			end)
		end
	end)
end

local function monitor(plr)
	local function char(c)
		local hum = c:WaitForChild("Humanoid")
		hum:GetPropertyChangedSignal("PlatformStand"):Connect(function()
			if hum.PlatformStand then
				criarGetUp(plr)
			end
		end)
	end
	if plr.Character then char(plr.Character) end
	plr.CharacterAdded:Connect(char)
end

for _,p in ipairs(Players:GetPlayers()) do monitor(p) end
Players.PlayerAdded:Connect(monitor)

--========================================================--
-- 🔘 BOTÃO
--========================================================--

btn.MouseButton1Click:Connect(function()
	PC_ENABLED = not PC_ENABLED

	btn.Text = PC_ENABLED and "PC / GET UP: ON" or "PC / GET UP: OFF"
	btn.TextColor3 = PC_ENABLED and Color3.fromRGB(80,255,80) or Color3.fromRGB(255,80,80)
	btn.BackgroundColor3 = PC_ENABLED and Color3.fromRGB(25,40,25) or Color3.fromRGB(40,20,20)

	for _,d in ipairs(Workspace:GetDescendants()) do
		if d:IsA("BillboardGui") and d.Name == "ProgressBar" then
			d.Enabled = PC_ENABLED
		elseif d:IsA("Highlight") and d.Name == "ComputerHighlight" then
			d.Enabled = PC_ENABLED
		end
	end

	getupGui.Enabled = PC_ENABLED
end)

print("✅ GET UP + PC PROGRESS carregado (Xeno)")
