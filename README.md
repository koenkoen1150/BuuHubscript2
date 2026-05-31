-- [[ Buu Hub v7.6 - Thaiban (Mobile Camera Fly Edition) ]] --
local P = game:GetService("Players")
local TS = game:GetService("TweenService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")
local lp = P.LocalPlayer

local fOn, ncOn, savedCF = false, false, nil
local flyOn = false
local flySpeed = 30 -- ความเร็วบินเริ่มต้น

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

-- หน้าต่างหลัก
local MF = Instance.new("Frame", SG)
MF.Size = UDim2.new(0, 450, 0, 250) -- ปรับขนาดลดความสูงลงมาเนื่องจากนำปุ่มขึ้นลงออกแล้ว
MF.Position = UDim2.new(0.5, -225, 0.5, -125)
MF.BackgroundColor3 = Color3.fromRGB(15,15,15)
MF.BackgroundTransparency = 0.2
Instance.new("UICorner", MF).CornerRadius = UDim.new(0, 10)

local TL = Instance.new("TextLabel", MF)
TL.Size = UDim2.new(1, -50, 0, 35)
TL.Position = UDim2.new(0, 15, 0, 5)
TL.Text = "Buu Hub (Mobile Camera Fly)"
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

-- ================= ฝั่งซ้าย (ฟังก์ชันเดิม) =================

btn("ตัดไม้ออโต้: ปิดอยู่", UDim2.new(0.04, 0, 0.20, 0), 200, function(b)
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

btn("ทะลุกำแพง: ปิดอยู่", UDim2.new(0.04, 0, 0.40, 0), 200, function(b)
    ncOn = not ncOn 
    setUI(b, ncOn and "ทะลุกำแพง: เปิดอยู่" or "ทะลุกำแพง: ปิดอยู่", ncOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
end)

local SP = Instance.new("TextBox", MF)
SP.Size = UDim2.new(0, 200, 0, 38)
SP.Position = UDim2.new(0.04, 0, 0.60, 0)
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

btn("📍 Save Pos", UDim2.new(0.04, 0, 0.80, 0), 95, function(b)
    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then 
        savedCF = lp.Character.HumanoidRootPart.CFrame 
        b.Text = "✅ Saved!" 
        b.TextColor3 = Color3.fromRGB(0,255,150) 
        task.wait(0.8) 
        b.Text = "📍 Save Pos" 
        b.TextColor3 = Color3.fromRGB(200,200,200) 
    end
end)

btn("🌀 Teleport", UDim2.new(0.27, 0, 0.80, 0), 95, function(b)
    if savedCF and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then 
        lp.Character.HumanoidRootPart.CFrame = savedCF 
    else 
        b.Text = "❌ No Pos!" 
        b.TextColor3 = Color3.fromRGB(255,100,100) 
        task.wait(0.8) 
        b.Text = "🌀 Teleport" 
        b.TextColor3 = Color3.fromRGB(200,200,200) 
    end
end)


-- ================= ฝั่งขวา (ระบบบินตามกล้องสำหรับมือถือ) =================

-- ปุ่มเปิด/ปิด บิน
btn("บิน (Fly): ปิดอยู่", UDim2.new(0.52, 0, 0.20, 0), 200, function(b)
    flyOn = not flyOn
    setUI(b, flyOn and "บิน (Fly): เปิดอยู่" or "บิน (Fly): ปิดอยู่", flyOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
end)

-- ช่องใส่ค่าความเร็วในการบิน
local FlySP = Instance.new("TextBox", MF)
FlySP.Size = UDim2.new(0, 200, 0, 38)
FlySP.Position = UDim2.new(0.52, 0, 0.40, 0)
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


-- ================= Loop ควบคุมตัวละครและการบินตามทิศกล้อง =================

RS.Stepped:Connect(function() 
    if (ncOn or flyOn) and lp.Character then 
        for _, p in ipairs(lp.Character:GetDescendants()) do 
            if p:IsA("BasePart") then p.CanCollide = false end 
        end 
    end 
end)

local Camera = workspace.CurrentCamera

RS.RenderStepped:Connect(function()
    if flyOn and lp.Character then
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        local hum = lp.Character:FindFirstChild("Humanoid")
        if hrp and hum then
            hrp.Velocity = Vector3.new(0, 0, 0) -- ล็อคไม่ให้ร่วงพื้น
            
            -- ระบบคำนวณการเคลื่อนที่บินตามหน้าจอ (Camera-Relative)
            if hum.MoveDirection.Magnitude > 0 then
                -- ดึงเวกเตอร์ทิศทางเดินของอนิเมชันปุ่มเดิน แล้วนำมาคูณกับทิศทางมุมกล้องจริงของหน้าจอมือถือ
                local flyDirection = Camera.CFrame:VectorToWorldSpace(Vector3.new(hum.MoveDirection.X, 0, hum.MoveDirection.Z))
                
                -- หากผู้เล่นดันปุ่มเดินไปด้านหน้า (ค่าแกน Z ต่ำกว่า 0) ระบบจะอิงองศากล้อง LookVector เพื่อให้บินขึ้น/ลงตามการก้มเงยหน้าจอได้
                if hum.MoveDirection.Z < 0 then
                    hrp.Position = hrp.Position + (Camera.CFrame.LookVector * (flySpeed / 50))
                else
                    -- กรณีเดินถอยหลังหรือไปด้านข้าง ให้บินระนาบตามปกติแต่หมุนแกนตามหน้าจอ
                    hrp.Position = hrp.Position + (flyDirection * (flySpeed / 50))
                end
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
