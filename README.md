-- [[ Buu Hub v7.6 - Thaiban (Mobile Fly Edition) ]] --
local P = game:GetService("Players")
local TS = game:GetService("TweenService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")
local lp = P.LocalPlayer

local fOn, ncOn, savedCF = false, false, nil
local flyOn = false
local flySpeed = 50 
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

-- หน้าต่างหลัก
local MF = Instance.new("Frame", SG)
MF.Size = UDim2.new(0, 450, 0, 250)
MF.Position = UDim2.new(0.5, -225, 0.5, -125)
MF.BackgroundColor3 = Color3.fromRGB(15,15,15)
MF.BackgroundTransparency = 0.2
Instance.new("UICorner", MF).CornerRadius = UDim.new(0, 10)

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

-- ระบบ Fly Infinity Yield
local function startFly()
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local bv = Instance.new("BodyVelocity", hrp); bv.Name = "FlyVel"; bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.zero
    local bg = Instance.new("BodyGyro", hrp); bg.Name = "FlyGyro"; bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge); bg.P = 10000; bg.CFrame = hrp.CFrame
    
    RS.RenderStepped:Connect(function()
        if not flyOn then return end
        bg.CFrame = Camera.CFrame
        local move = lp.Character.Humanoid.MoveDirection
        bv.Velocity = (Camera.CFrame.LookVector * (move.Z * flySpeed)) + (Camera.CFrame.RightVector * (move.X * flySpeed))
    end)
end

local function stopFly()
    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        for _, v in pairs(lp.Character.HumanoidRootPart:GetChildren()) do if v.Name == "FlyVel" or v.Name == "FlyGyro" then v:Destroy() end end
    end
end

-- ปุ่มต่างๆ
btn("ตัดไม้ออโต้: ปิดอยู่", UDim2.new(0.04, 0, 0.20, 0), 200, function(b)
    fOn = not fOn
    setUI(b, fOn and "ตัดไม้ออโต้: เปิดอยู่" or "ตัดไม้ออโต้: ปิดอยู่", fOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
    if fOn then task.spawn(function() while fOn do for _,p in ipairs(TreePos) do if not fOn then break end
        local rp = lp.Character:FindFirstChild("HumanoidRootPart")
        if rp then TS:Create(rp, TweenInfo.new((rp.Position-p).Magnitude/16), {CFrame=CFrame.new(p)}):Play(); task.wait((rp.Position-p).Magnitude/16 + 0.5) end
        if fOn then for i=1,18 do VIM:SendMouseButtonEvent(0,0,0,true,game,1); task.wait(0.02); VIM:SendMouseButtonEvent(0,0,0,false,game,1); task.wait(0.5) end end
    end task.wait(0.5) end end) end
end)

btn("ทะลุกำแพง: ปิดอยู่", UDim2.new(0.04, 0, 0.40, 0), 200, function(b)
    ncOn = not ncOn
    setUI(b, ncOn and "ทะลุกำแพง: เปิดอยู่" or "ทะลุกำแพง: ปิดอยู่", ncOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
end)

btn("บิน (Fly): ปิดอยู่", UDim2.new(0.52, 0, 0.20, 0), 200, function(b)
    flyOn = not flyOn
    setUI(b, flyOn and "บิน (Fly): เปิดอยู่" or "บิน (Fly): ปิดอยู่", flyOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
    if flyOn then startFly() else stopFly() end
end)

local FlySP = Instance.new("TextBox", MF)
FlySP.Size = UDim2.new(0, 200, 0, 38); FlySP.Position = UDim2.new(0.52, 0, 0.40, 0)
FlySP.PlaceholderText = "ความเร็วบิน (ปกติ 20)"; FlySP.Text = "20"; FlySP.BackgroundColor3 = Color3.fromRGB(30,30,30)
Instance.new("UICorner", FlySP)
FlySP.FocusLost:Connect(function() flySpeed = tonumber(FlySP.Text) or 50 end)

-- ลูป Noclip
RS.Stepped:Connect(function()
    if (ncOn or flyOn) and lp.Character then for _,p in ipairs(lp.Character:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end end
end)

-- ปุ่มเปิดปิด UI
local RB = Instance.new("TextButton", SG)
RB.Size = UDim2.new(0, 50, 0, 50); RB.Position = UDim2.new(0.05, 0, 0.05, 0)
RB.BackgroundColor3 = Color3.fromRGB(20,20,20); RB.Text = "Buu"; RB.TextColor3 = Color3.fromRGB(0,150,255); RB.Draggable = true
Instance.new("UICorner", RB).CornerRadius = UDim.new(0,25)
RB.MouseButton1Click:Connect(function() MF.Visible = not MF.Visible end)
