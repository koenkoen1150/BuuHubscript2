-- [[ Buu Hub v7.6 - Thaiban (Mobile Fly + Speed Box Edition) ]] --
local P = game:GetService("Players")
local TS = game:GetService("TweenService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")
local lp = P.LocalPlayer

local fOn, ncOn, savedCF = false, false, nil
local flyOn = false
local flySpeed = 30 -- ความเร็วบินเริ่มต้น
local Camera = workspace.CurrentCamera

local TreePos = {
    Vector3.new(186.08, 9.17, 227.97), 
    Vector3.new(182.56, 9.00, 213.30), 
    Vector3.new(169.69, 9.00, 211.43), 
    Vector3.new(166.20, 9.01, 224.03), 
    Vector3.new(150.13, 9.00, 215.99)
}

-- ฟังก์ชันลบชื่อบนหัว
local function removeNames(c)
    local h = c:FindFirstChild("Humanoid")
    if h then h.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None end
    for _, v in ipairs(c:GetDescendants()) do 
        if v:IsA("BillboardGui") then v.Enabled = false end 
    end
end
if lp.Character then removeNames(lp.Character) end
lp.CharacterAdded:Connect(removeNames)

local targetGui = game:GetService("CoreGui")
if not pcall(function() local a = targetGui.Name end) then 
    targetGui = lp:WaitForChild("PlayerGui") 
end

if targetGui:FindFirstChild("BuuHub_Short") then 
    targetGui["BuuHub_Short"]:Destroy() 
end

local SG = Instance.new("ScreenGui", targetGui)
SG.Name = "BuuHub_Short"
SG.ResetOnSpawn = false

-- หน้าต่างหลัก (ขยายกว้าง 450 เพื่อแบ่ง 2 ฝั่งให้ชัดเจนและจิ้มง่ายบนมือถือ)
local MF = Instance.new("Frame", SG)
MF.Size = UDim2.new(0, 450, 0, 310)
MF.Position = UDim2.new(0.5, -225, 0.5, -155)
MF.BackgroundColor3 = Color3.fromRGB(15,15,15)
MF.BackgroundTransparency = 0.2
Instance.new("UICorner", MF).CornerRadius = UDim.new(0, 10)

local TL = Instance.new("TextLabel", MF)
TL.Size = UDim2.new(1, -50, 0, 35)
TL.Position = UDim2.new(0, 15, 0, 5)
TL.Text = "BUUHUB" 
TL.TextColor3 = Color3.fromRGB(255,255,255)
TL.TextSize = 20
TL.Font = 4
TL.BackgroundTransparency = 1
TL.TextXAlignment = 0

local CB = Instance.new("TextButton", MF)
CB.Text = "✕"
CB.Size = UDim2.new(0, 26, 0, 26)
CB.Position = UDim2.new(0.92, 0, 0.03, 0)
CB.BackgroundColor3 = Color3.fromRGB(20,20,20)
CB.TextColor3 = Color3.fromRGB(255,100,100)
CB.Font = 4
Instance.new("UICorner", CB).CornerRadius = UDim.new(0,6)
CB.MouseButton1Click:Connect(function() SG:Destroy() end)

local function setUI(b, txt, clr) 
    b.Text = txt 
    b.TextColor3 = clr 
    b.BackgroundColor3 = (clr == Color3.fromRGB(200,200,200) and Color3.fromRGB(30,30,30) or Color3.fromRGB(0,50,150)) 
end

local function btn(txt, pos, sizeX, c)
    local B = Instance.new("TextButton", MF)
    B.Size = UDim2.new(0, sizeX or 200, 0, 38)
    B.Position = pos
    B.Text = txt
    B.TextColor3 = Color3.fromRGB(200,200,200)
    B.BackgroundColor3 = Color3.fromRGB(30,30,30)
    B.Font = 4
    B.TextSize = 14
    Instance.new("UICorner", B)
    B.MouseButton1Click:Connect(function() c(B) end) 
    return B
end

local function walk(pos)
    local char = lp.Character or lp.CharacterAdded:Wait()
    local rp = char:WaitForChild("HumanoidRootPart")
    local t = TS:Create(rp, TweenInfo.new((rp.Position - pos).Magnitude / 16, Enum.EasingStyle.Linear), {CFrame = CFrame.new(pos)})
    t:Play() 
    t.Completed:Wait() 
    task.wait(0.5)
end

-- ================= ระบบล็อกอนิเมชัน / ล็อกท่าทาง (Infinity Yield ของแท้) =================
local oldAnimState = nil

local function startFly()
    local char = lp.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    if not hrp or not hum then return end

    -- ล็อกท่าเดิน: ปิดสคริปต์อนิเมชันเพื่อล็อกตัวตรงตัวแข็ง
    local animateScript = char:FindFirstChild("Animate")
    if animateScript then
        oldAnimState = animateScript.Enabled
        animateScript.Enabled = false
    end
    
    -- หยุดการขยับขาทุกอย่าง
    for _, track in ipairs(hum:GetPlayingAnimationTracks()) do
        track:Stop()
    end

    -- ปรับแต่งแรงยกสไตล์ IY ให้มั่นคงสูง ไม่แกว่ง
    local bv = Instance.new("BodyVelocity", hrp)
    bv.Velocity = Vector3.new(0, 0, 0)
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Name = "IYFlyVelocity"

    local bg = Instance.new("BodyGyro", hrp)
    bg.CFrame = hrp.CFrame
    bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bg.P = 15000 -- เพิ่มความหนืดในการล็อกมุมมองให้หันตามเป๊ะๆ
    bg.Name = "IYFlyGyro"
end

local function stopFly()
    local char = lp.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        local hrp = char.HumanoidRootPart
        if hrp:FindFirstChild("IYFlyVelocity") then hrp.IYFlyVelocity:Destroy() end
        if hrp:FindFirstChild("IYFlyGyro") then hrp.IYFlyGyro:Destroy() end
    end
    
    -- เปิดแอนิเมชันให้เดินขยับขาได้ปกติเมื่อปิดบิน
    if char and char:FindFirstChild("Animate") and oldAnimState ~= nil then
        char.Animate.Enabled = oldAnimState
    end
end

-- ================= ฝั่งซ้าย (ฟังก์ชันเดิมของคุณ) =================

btn("ตัดไม้ออโต้: ปิดอยู่", UDim2.new(0.04, 0, 0.16, 0), 200, function(b)
    fOn = not fOn 
    setUI(b, fOn and "ตัดไม้ออโต้: เปิดอยู่" or "ตัดไม้ออโต้: ปิดอยู่", fOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
    if fOn then 
        task.spawn(function() 
            while fOn do 
                for _, p in ipairs(TreePos) do 
                    if not fOn then break end 
                    walk(p) 
                    if fOn then 
                        for i=1,18 do 
                            if not fOn then break end 
                            VIM:SendMouseButtonEvent(0,0,0,true,game,1) 
                            task.wait(0.02) 
                            VIM:SendMouseButtonEvent(0,0,0,false,game,1) 
                            task.wait(0.5) 
                        end 
                    end 
                end 
                task.wait(0.5) 
            end 
        end) 
    end
end)

btn("ทะลุกำแพง: ปิดอยู่", UDim2.new(0.04, 0, 0.32, 0), 200, function(b)
    ncOn = not ncOn 
    setUI(b, ncOn and "ทะลุกำแพง: เปิดอยู่" or "ทะลุกำแพง: ปิดอยู่", ncOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
end)

local SP = Instance.new("TextBox", MF)
SP.Size = UDim2.new(0, 200, 0, 38)
SP.Position = UDim2.new(0.04, 0, 0.48, 0)
SP.BackgroundColor3 = Color3.fromRGB(30,30,30)
SP.BackgroundTransparency = 0.3
SP.PlaceholderText = "ใส่ความเร็วเดิน... (ปกติ 16)"
SP.Text = ""
SP.TextColor3 = Color3.fromRGB(0,150,255)
SP.Font = 4
SP.TextSize = 14
Instance.new("UICorner", SP)

SP.FocusLost:Connect(function() 
    if tonumber(SP.Text) and lp.Character and lp.Character:FindFirstChild("Humanoid") then 
        lp.Character.Humanoid.WalkSpeed = tonumber(SP.Text) 
    end 
end)

btn("📍 บันทึกจุดพิกัดปัจจุบัน", UDim2.new(0.04, 0, 0.66, 0), 200, function(b)
    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then 
        savedCF = lp.Character.HumanoidRootPart.CFrame 
        b.Text = "✅ บันทึกจุดสำเร็จ!" 
        b.TextColor3 = Color3.fromRGB(0,255,150) 
        task.wait(1) 
        b.Text = "📍 บันทึกจุดพิกัดปัจจุบัน" 
        b.TextColor3 = Color3.fromRGB(255,215,0) 
    end
end)

btn("🌀 วาร์ปกลับจุดที่เซ็ตไว้", UDim2.new(0.04, 0, 0.82, 0), 200, function(b)
    if savedCF and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then 
        lp.Character.HumanoidRootPart.CFrame = savedCF 
    else 
        b.Text = "❌ ยังไม่ได้เซ็ตจุด!" 
        b.TextColor3 = Color3.fromRGB(255,100,100) 
        task.wait(1) 
        b.Text = "🌀 วาร์ปกลับจุดที่เซ็ตไว้" 
        b.TextColor3 = Color3.fromRGB(200,200,200) 
    end
end)


-- ================= ฝั่งขวา (ระบบบินตรงตามทิศกล้อง) =================

-- ปุ่มเปิด/ปิด บิน
btn("บิน (Fly): ปิดอยู่", UDim2.new(0.52, 0, 0.16, 0), 200, function(b)
    flyOn = not flyOn
    setUI(b, flyOn and "บิน (Fly): เปิดอยู่" or "บิน (Fly): ปิดอยู่", flyOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
    if flyOn then
        startFly()
    else
        stopFly()
    end
end)

-- ช่องใส่ค่าความเร็วในการบิน
local FlySP = Instance.new("TextBox", MF)
FlySP.Size = UDim2.new(0, 200, 0, 38)
FlySP.Position = UDim2.new(0.52, 0, 0.32, 0)
FlySP.BackgroundColor3 = Color3.fromRGB(30,30,30)
FlySP.BackgroundTransparency = 0.3
FlySP.PlaceholderText = "ใส่ความเร็วบิน... (ปกติ 30)"
FlySP.Text = "30"
FlySP.TextColor3 = Color3.fromRGB(0,255,150)
FlySP.Font = 4
FlySP.TextSize = 14
Instance.new("UICorner", FlySP)

FlySP.FocusLost:Connect(function()
    if tonumber(FlySP.Text) then
        flySpeed = tonumber(FlySP.Text)
    else
        FlySP.Text = tostring(flySpeed) 
    end
end)


-- ================= Loop ควบคุมการเคลื่อนที่และแก้แกนทิศทางมั่ว =================

RS.Stepped:Connect(function() 
    if ncOn and lp.Character then 
        for _, p in ipairs(lp.Character:GetDescendants()) do 
            if p:IsA("BasePart") then p.CanCollide = false end 
        end 
    elseif flyOn and lp.Character then
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CanCollide = false end 
    end 
end)

RS.RenderStepped:Connect(function()
    if flyOn and lp.Character then
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        local hum = lp.Character:FindFirstChild("Humanoid")
        
        local IYVel = hrp and hrp:FindFirstChild("IYFlyVelocity")
        local IYGyro = hrp and hrp:FindFirstChild("IYFlyGyro")
        
        if hrp and hum and IYVel and IYGyro then
            -- ล็อกให้ตัวละครหันหน้าตรงตามกล้องมองตลอดเวลา
            IYGyro.CFrame = Camera.CFrame
            
            -- ตรวจจับปุ่มกดเดิน
            local moveDir = hum.MoveDirection
            if moveDir.Magnitude > 0 then
                -- แก้ไขสูตรล็อกทิศทางใหม่: นำเอาเวกเตอร์ทิศทางปุ่มเดิน แปลงเข้าสู่ทิศทางกล้องโลก (World Space) ตรงๆ 
                -- และปรับทิศทางลบแกน Z เพื่อให้กดดันเดินหน้าแล้วตัวพุ่งตรงไปข้างหน้ากล้องทันที หันขึ้นบินขึ้น หันลงบินลง
                local direction = Camera.CFrame:VectorToWorldSpace(Vector3.new(moveDir.X, 0, moveDir.Z))
                
                -- ตรวจสอบหากเป็นการกดดันเดินไปด้านหน้า (Z น้อยกว่า 0) ให้พุ่งไปตาม LookVector ของกล้องตรงๆ
                if hum.MoveDirection.Z < 0 then
                    IYVel.Velocity = Camera.CFrame.LookVector * flySpeed
                elseif hum.MoveDirection.Z > 0 then
                    -- กดถอยหลัง ให้พุ่งถอยจากทิศกล้องตรงๆ
                    IYVel.Velocity = Camera.CFrame.LookVector * -flySpeed
                else
                    -- บินสไลด์ข้างซ้าย-ขวา ตามองศากล้องปัจจุบัน
                    IYVel.Velocity = direction * flySpeed
                end
            else
                -- ถ้าไม่ได้กดเดินให้ลอยตัวตรงล็อกท่านิ่งอยู่กับที่
                IYVel.Velocity = Vector3.new(0, 0, 0)
            end
        end
    end
end)


-- ================= ปุ่มย่อเปิด/ปิด UI หลัก =================
local RB = Instance.new("TextButton", SG)
RB.Size = UDim2.new(0, 50, 0, 50)
RB.Position = UDim2.new(0.05, 0, 0.05, 0)
RB.BackgroundColor3 = Color3.fromRGB(20,20,20)
RB.BackgroundTransparency = 0.3
RB.Text = "Buu"
RB.TextColor3 = Color3.fromRGB(0,150,255)
RB.Font = 4
RB.TextSize = 16
RB.Draggable = true
Instance.new("UICorner", RB).CornerRadius = UDim.new(0,25)
RB.MouseButton1Click:Connect(function() MF.Visible = not MF.Visible end)
