-- ============================================
-- Delta Hub Mobile - نسخة محسنة للجوال (الجزء 1/3)
-- ============================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- متغيرات عامة
local savedPosition = nil
local isTeleportActive = false
local currentSpeed = 16
local isTeleporting = false
local ignoredPlayers = {}
local espEnabled = false
local espObjects = {}
local autoAimEnabled = false
local hitboxEnabled = false
local hitboxSize = 8

-- إنشاء الواجهة الرئيسية (ستظهر بشكل جميل على الجوال)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DeltaMobile"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- حاوية ESP (لن تتأثر بحجم الشاشة)
local espContainer = Instance.new("Frame")
espContainer.Size = UDim2.new(1, 0, 1, 0)
espContainer.BackgroundTransparency = 1
espContainer.Parent = screenGui

print("✅ الجزء الأول تم تحميله بنجاح - انتقل للجزء الثاني")-- ============================================
-- الجزء 2/3: القائمة الرئيسية بتصميم جوال
-- ============================================

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0.9, 0, 0, 0.7) -- 90% عرض الشاشة، 70% ارتفاع
mainFrame.Position = UDim2.new(0.05, 0, 0.15, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainFrame.BackgroundTransparency = 0.1
mainFrame.Draggable = true
mainFrame.Active = true
mainFrame.Visible = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)

-- شريط عنوان كبير
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
title.Text = "⚡ DELTA MOBILE ⚡"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame
Instance.new("UICorner", title).CornerRadius = UDim.new(0, 12)

-- حاوية الأزرار (ScrollingFrame لأن القائمة طويلة)
local btnContainer = Instance.new("ScrollingFrame")
btnContainer.Size = UDim2.new(1, -10, 1, -60)
btnContainer.Position = UDim2.new(0, 5, 0, 55)
btnContainer.BackgroundTransparency = 1
btnContainer.ScrollBarThickness = 5
btnContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
btnContainer.Parent = mainFrame

-- دالة مساعدة لإنشاء الأزرار بشكل جميل ومناسب للجوال
local function createBtn(text, yPos, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 55)
    btn.Position = UDim2.new(0, 5, 0, yPos)
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamBold
    btn.Parent = btnContainer
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    return btn
end

-- الأزرار (سنضيف لها وظائف لاحقاً)
local espBtn = createBtn("🔍 ESP: OFF", 0, Color3.fromRGB(0, 150, 200))
local ignoreBtn = createBtn("👤 IGNORE PLAYER", 65, Color3.fromRGB(200, 100, 50))
local hitboxBtn = createBtn("📦 HITBOX: OFF", 130, Color3.fromRGB(150, 50, 150))
local aimBtn = createBtn("🎯 AUTO AIM: OFF", 195, Color3.fromRGB(255, 80, 80))
local speedBtn = createBtn("⚡ SPEED: OFF (50)", 260, Color3.fromRGB(100, 150, 255))
local tpActivateBtn = createBtn("🌀 ACTIVATE TELEPORT", 325, Color3.fromRGB(100, 150, 255))
local closeBtn = createBtn("✖️ CLOSE MENU", 390, Color3.fromRGB(100, 100, 100))

-- تحديث حجم الـ ScrollingFrame
btnContainer.CanvasSize = UDim2.new(0, 0, 0, 460)

-- زر فتح القائمة المنفصل (صغير ويمكن تحريكه)
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 60, 0, 60)
toggleBtn.Position = UDim2.new(0.02, 0, 0.8, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
toggleBtn.Text = "⚡"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.TextSize = 24
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.Draggable = true
toggleBtn.Parent = screenGui
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(1, 0)

-- زر التلبورت المنفصل (سيظهر عند التفعيل)
local tpButton = Instance.new("TextButton")
tpButton.Size = UDim2.new(0, 70, 0, 70)
tpButton.Position = UDim2.new(0.8, 0, 0.7, 0)
tpButton.BackgroundColor3 = Color3.fromRGB(30, 40, 60)
tpButton.Text = "🔮\nTP"
tpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
tpButton.TextSize = 14
tpButton.Font = Enum.Font.GothamBold
tpButton.Visible = false
tpButton.Draggable = true
tpButton.Parent = screenGui
Instance.new("UICorner", tpButton).CornerRadius = UDim.new(1, 0)

print("✅ الجزء الثاني تم تحميله - انتقل للجزء الثالث (الأخير)")-- ============================================
-- الجزء 3/3: جميع الوظائف (ESP، هيت بوكس، تصويب، تلبورت)
-- ============================================

-- متغيرات السرعة
local speedEnabled = false
local currentSpeedVal = 50

-- وظيفة السرعة
speedBtn.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    if speedEnabled then
        speedBtn.Text = "⚡ SPEED: ON (" .. currentSpeedVal .. ")"
        speedBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
        humanoid.WalkSpeed = currentSpeedVal
    else
        speedBtn.Text = "⚡ SPEED: OFF (" .. currentSpeedVal .. ")"
        speedBtn.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
        humanoid.WalkSpeed = 16
    end
end)

-- مربع إدخال السرعة (يظهر عند الضغط المطول)
local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0, 150, 0, 40)
speedBox.Position = UDim2.new(0.5, -75, 0.5, -20)
speedBox.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
speedBox.Text = "50"
speedBox.TextColor3 = Color3.fromRGB(255, 255, 255)
speedBox.TextSize = 16
speedBox.Font = Enum.Font.Code
speedBox.Visible = false
speedBox.Parent = screenGui
Instance.new("UICorner", speedBox).CornerRadius = UDim.new(0, 8)

speedBtn.MouseButton2Click:Connect(function() -- الضغط بزر الفأرة الأيمن أو الضغط المطول على الجوال
    speedBox.Visible = true
    speedBox:CaptureFocus()
end)
speedBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newSpeed = tonumber(speedBox.Text)
        if newSpeed then
            currentSpeedVal = newSpeed
            if speedEnabled then
                humanoid.WalkSpeed = currentSpeedVal
                speedBtn.Text = "⚡ SPEED: ON (" .. currentSpeedVal .. ")"
            else
                speedBtn.Text = "⚡ SPEED: OFF (" .. currentSpeedVal .. ")"
            end
        end
    end
    speedBox.Visible = false
end)

-- ==================== هيت بوكس ====================
local function applyHitbox(plr)
    if not hitboxEnabled then return end
    if plr == player or ignoredPlayers[plr.Name] then return end
    local char = plr.Character
    if not char then return end
    local parts = {char:FindFirstChild("HumanoidRootPart"), char:FindFirstChild("Torso"), char:FindFirstChild("UpperTorso")}
    for _, part in pairs(parts) do
        if part then
            part.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
            part.Transparency = 0.5
            part.CanCollide = false
        end
    end
end

RunService.Heartbeat:Connect(function()
    if hitboxEnabled then
        for _, plr in pairs(Players:GetPlayers()) do
            pcall(applyHitbox, plr)
        end
    end
end)

hitboxBtn.MouseButton1Click:Connect(function()
    hitboxEnabled = not hitboxEnabled
    if hitboxEnabled then
        hitboxBtn.Text = "📦 HITBOX: ON"
        hitboxBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    else
        hitboxBtn.Text = "📦 HITBOX: OFF"
        hitboxBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 150)
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
                if hrp then hrp.Size = Vector3.new(2, 2, 1) end
                local torso = plr.Character:FindFirstChild("Torso")
                if torso then torso.Size = Vector3.new(2, 2, 1) end
                local utorso = plr.Character:FindFirstChild("UpperTorso")
                if utorso then utorso.Size = Vector3.new(2, 2, 1) end
            end
        end
    end
end)

-- ==================== AI ذكي للعثور على الهدف ====================
local function getSmartTarget()
    local bestTarget = nil
    local bestScore = -math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and not ignoredPlayers[p.Name] and p.Character then
            local hrp = p.Character:FindFirstChild("HumanoidRootPart")
            local hum = p.Character:FindFirstChild("Humanoid")
            if hrp and hum and hum.Health > 0 then
                local distance = (rootPart.Position - hrp.Position).Magnitude
                local healthFactor = (hum.Health / hum.MaxHealth) * 10
                local score = (50 - distance) + (10 - healthFactor)
                if score > bestScore then
                    bestScore = score
                    bestTarget = p
                end
            end
        end
    end
    return bestTarget
end

-- ==================== تصويب تلقائي ====================
local function updateAim()
    if not autoAimEnabled then return end
    local target = getSmartTarget()
    if target and target.Character then
        local targetPos = target.Character:FindFirstChild("HumanoidRootPart").Position
        local newCF = CFrame.new(Camera.CFrame.Position, targetPos)
        Camera.CFrame = Camera.CFrame:Lerp(newCF, 0.3)
        rootPart.CFrame = CFrame.new(rootPart.Position, Vector3.new(targetPos.X, rootPart.Position.Y, targetPos.Z))
    end
end

aimBtn.MouseButton1Click:Connect(function()
    autoAimEnabled = not autoAimEnabled
    if autoAimEnabled then
        aimBtn.Text = "🎯 AUTO AIM: ON"
        aimBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    else
        aimBtn.Text = "🎯 AUTO AIM: OFF"
        aimBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
    end
end)

-- ==================== ESP ====================
local function clearESP()
    for _, obj in pairs(espObjects) do
        pcall(function()
            if obj.line then obj.line:Destroy() end
            if obj.name then obj.name:Destroy() end
            if obj.bar then obj.bar:Destroy() end
        end)
    end
    espObjects = {}
end

local function updateESP()
    if not espEnabled then clearESP() return end
    local viewX, viewY = Camera.ViewportSize.X, Camera.ViewportSize.Y
    local centerX = viewX / 2
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and not ignoredPlayers[p.Name] and p.Character then
            local hrp = p.Character:FindFirstChild("HumanoidRootPart")
            local hum = p.Character:FindFirstChild("Humanoid")
            if hrp and hum and hum.Health > 0 then
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position + Vector3.new(0, 2, 0))
                if onScreen and pos.Z > 0 then
                    if not espObjects[p] then
                        espObjects[p] = {}
                        local line = Instance.new("Frame")
                        line.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
                        line.BorderSizePixel = 0
                        line.Parent = espContainer
                        espObjects[p].line = line
                        local name = Instance.new("TextLabel")
                        name.Size = UDim2.new(0, 140, 0, 22)
                        name.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
                        name.BackgroundTransparency = 0.5
                        name.TextColor3 = Color3.fromRGB(255, 255, 255)
                        name.TextSize = 12
                        name.Font = Enum.Font.GothamBold
                        name.Parent = espContainer
                        Instance.new("UICorner", name).CornerRadius = UDim.new(0, 4)
                        espObjects[p].name = name
                        local bar = Instance.new("Frame")
                        bar.Size = UDim2.new(0, 60, 0, 6)
                        bar.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
                        bar.BorderSizePixel = 0
                        bar.Parent = espContainer
                        Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)
                        espObjects[p].bar = bar
                    end
                    local esp = espObjects[p]
                    local healthPercent = hum.Health / hum.MaxHealth
                    local startY = viewY
                    local endY = pos.Y
                    local distance = startY - endY
                    local angle = math.atan2(endY - startY, pos.X - centerX)
                    esp.line.Size = UDim2.new(0, 2, 0, math.abs(distance))
                    esp.line.Position = UDim2.new(0, centerX, 0, endY)
                    esp.line.Rotation = math.deg(angle)
                    esp.line.Visible = true
                    esp.name.Text = "[" .. p.Name .. "]"
                    esp.name.Position = UDim2.new(0, pos.X - 70, 0, pos.Y - 40)
                    esp.name.Visible = true
                    local barWidth = 60 * healthPercent
                    esp.bar.Size = UDim2.new(0, barWidth, 0, 6)
                    esp.bar.Position = UDim2.new(0, pos.X - 30, 0, pos.Y - 18)
                    esp.bar.BackgroundColor3 = Color3.fromRGB(0, 200, 0):Lerp(Color3.fromRGB(200, 0, 0), 1 - healthPercent)
                    esp.bar.Visible = true
                else
                    if espObjects[p] then
                        if espObjects[p].line then espObjects[p].line.Visible = false end
                        if espObjects[p].name then espObjects[p].name.Visible = false end
                        if espObjects[p].bar then espObjects[p].bar.Visible = false end
                    end
                end
            else
                if espObjects[p] then
                    pcall(function()
                        if espObjects[p].line then espObjects[p].line:Destroy() end
                        if espObjects[p].name then espObjects[p].name:Destroy() end
                        if espObjects[p].bar then espObjects[p].bar:Destroy() end
                    end)
                    espObjects[p] = nil
                end
            end
        end
    end
end

espBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    if espEnabled then
        espBtn.Text = "🔍 ESP: ON"
        espBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    else
        espBtn.Text = "🔍 ESP: OFF"
        espBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
        clearESP()
    end
end)

-- ==================== قائمة تجاهل اللاعبين ====================
local selectGui = nil
ignoreBtn.MouseButton1Click:Connect(function()
    if selectGui then selectGui:Destroy() selectGui = nil return end
    selectGui = Instance.new("ScreenGui")
    selectGui.Name = "SelectPlayer"
    selectGui.Parent = screenGui
    local selectFrame = Instance.new("Frame")
    selectFrame.Size = UDim2.new(0.8, 0, 0, 0.6)
    selectFrame.Position = UDim2.new(0.1, 0, 0.2, 0)
    selectFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    selectFrame.Draggable = true
    selectFrame.Parent = selectGui
    Instance.new("UICorner", selectFrame).CornerRadius = UDim.new(0, 12)
    local selTitle = Instance.new("TextLabel")
    selTitle.Size = UDim2.new(1, 0, 0, 40)
    selTitle.BackgroundColor3 = Color3.fromRGB(200, 100, 50)
    selTitle.Text = "IGNORE PLAYER"
    selTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    selTitle.TextSize = 16
    selTitle.Font = Enum.Font.GothamBold
    selTitle.Parent = selectFrame
    Instance.new("UICorner", selTitle).CornerRadius = UDim.new(0, 12)
    local closeSel = Instance.new("TextButton")
    closeSel.Size = UDim2.new(0, 35, 0, 35)
    closeSel.Position = UDim2.new(1, -40, 0, 3)
    closeSel.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    closeSel.Text = "X"
    closeSel.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeSel.TextSize = 18
    closeSel.Font = Enum.Font.GothamBold
    closeSel.Parent = selTitle
    Instance.new("UICorner", closeSel).CornerRadius = UDim.new(1, 0)
    closeSel.MouseButton1Click:Connect(function() selectGui:Destroy() selectGui = nil end)
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -10, 0, -50)
    scroll.Position = UDim2.new(0, 5, 0, 45)
    scroll.BackgroundTransparency = 1
    scroll.ScrollBarThickness = 5
    scroll.Parent = selectFrame
    local y = 0
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 45)
            btn.Position = UDim2.new(0, 5, 0, y)
            btn.BackgroundColor3 = ignoredPlayers[p.Name] and Color3.fromRGB(0, 120, 40) or Color3.fromRGB(50, 50, 70)
            btn.Text = p.Name .. (ignoredPlayers[p.Name] and " ✓" or "")
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.TextSize = 14
            btn.Font = Enum.Font.Gotham
            btn.TextXAlignment = "Left"
            btn.Parent = scroll
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
            btn.MouseButton1Click:Connect(function()
                if ignoredPlayers[p.Name] then
                    ignoredPlayers[p.Name] = nil
                    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
                    btn.Text = p.Name
                else
                    ignoredPlayers[p.Name] = true
                    btn.BackgroundColor3 = Color3.fromRGB(0, 120, 40)
                    btn.Text = p.Name .. " ✓"
                end
            end)
            y = y + 50
        end
    end
    scroll.CanvasSize = UDim2.new(0, 0, 0, y + 20)
end)

-- ==================== التلبورت ====================
local function getHandPos(target)
    local char = target.Character
    if not char then return nil end
    local hand = char:FindFirstChild("RightHand") or char:FindFirstChild("Right Arm") or char:FindFirstChild("ArmR")
    if hand then return hand.Position end
    local head = char:FindFirstChild("Head")
    if head then return head.Position + Vector3.new(1, 0, 0) end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if hrp then return hrp.Position + Vector3.new(1.5, 1, 0) end
    return nil
end

local function teleportTo(pos)
    if rootPart and pos then rootPart.CFrame = CFrame.new(pos) end
end

local function doTeleport()
    if isTeleporting then return end
    isTeleporting = true
    local target = getSmartTarget()
    if not target then
        tpButton.Text = "❌"
        tpButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        task.wait(0.5)
        tpButton.Text = "🔮\nTP"
        tpButton.BackgroundColor3 = Color3.fromRGB(30, 40, 60)
        isTeleporting = false
        return
    end
    local handPos = getHandPos(target)
    if not handPos then
        tpButton.Text = "⚠️"
        tpButton.BackgroundColor3 = Color3.fromRGB(200, 100, 50)
        task.wait(0.5)
        tpButton.Text = "🔮\nTP"
        tpButton.BackgroundColor3 = Color3.fromRGB(30, 40, 60)
        isTeleporting = false
        return
    end
    savedPosition = rootPart.CFrame
    teleportTo(handPos)
    tpButton.Text = "🔄"
    tpButton.BackgroundColor3 = Color3.fromRGB(200, 150, 0)
    for ang = 0, 300, 60 do
        local rad = math.rad(ang)
        local offset = Vector3.new(math.cos(rad) * 2, 0, math.sin(rad) * 2)
        pcall(function() rootPart.CFrame = CFrame.new(handPos + offset, handPos) end)
        task.wait(0.01)
    end
    tpButton.Text = "📍"
    tpButton.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    task.wait(0.05)
    if savedPosition then
        teleportTo(savedPosition.Position)
        savedPosition = nil
    end
    tpButton.Text = "🔮\nTP"
    tpButton.BackgroundColor3 = Color3.fromRGB(30, 40, 60)
    isTeleporting = false
end

-- تفعيل زر التلبورت المنفصل
tpActivateBtn.MouseButton1Click:Connect(function()
    isTeleportActive = not isTeleportActive
    if isTeleportActive then
        tpActivateBtn.Text = "🌀 TELEPORT: ON"
        tpActivateBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
        tpButton.Visible = true
    else
        tpActivateBtn.Text = "🌀 ACTIVATE TELEPORT"
        tpActivateBtn.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
        tpButton.Visible = false
    end
end)

tpButton.MouseButton1Click:Connect(function()
    if isTeleportActive and not isTeleporting then doTeleport() end
end)

-- ==================== إظهار/إخفاء القائمة ====================
local menuVisible = true
toggleBtn.MouseButton1Click:Connect(function()
    menuVisible = not menuVisible
    mainFrame.Visible = menuVisible
    toggleBtn.Text = menuVisible and "⚡" or "✖"
end)

-- ==================== حلقات التحديث ====================
RunService.RenderStepped:Connect(function()
    if espEnabled then pcall(updateESP) end
    if autoAimEnabled then pcall(updateAim) end
end)

RunService.Heartbeat:Connect(function()
    if speedEnabled and humanoid and humanoid.WalkSpeed ~= currentSpeedVal then
        humanoid.WalkSpeed = currentSpeedVal
    end
end)

player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    if speedEnabled then
        task.wait(0.3)
        humanoid.WalkSpeed = currentSpeedVal
    end
end)

-- زر الإغلاق في القائمة
closeBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    menuVisible = false
    toggleBtn.Text = "✖"
end)

print("✅ تم تحميل Delta Hub Mobile بالكامل! استمتع!")
