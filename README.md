-- [[ Buu Hub v7.6 - Thaiban ]] --
local P = game:GetService("Players")
local TS = game:GetService("TweenService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")
local lp = P.LocalPlayer

local fOn, ncOn, savedCF = false, false, nil
local TreePos = {
    Vector3.new(186.08, 9.17, 227.97), 
    Vector3.new(182.56, 9.00, 213.30), 
    Vector3.new(169.69, 9.00, 211.43), 
    Vector3.new(166.20, 9.01, 224.03), 
    Vector3.new(150.13, 9.00, 215.99)
}

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

local MF = Instance.new("Frame", SG)
MF.Size = UDim2.new(0, 240, 0, 290)
MF.Position = UDim2.new(0.5, -120, 0.5, -145)
MF.BackgroundColor3 = Color3.fromRGB(15,15,15)
MF.BackgroundTransparency = 0.4
Instance.new("UICorner", MF).CornerRadius = UDim.new(0, 10)

local TL = Instance.new("TextLabel", MF)
TL.Size = UDim2.new(1, -50, 0, 35)
TL.Position = UDim2.new(0, 15, 0, 5)
TL.Text = "Buu Hub"
TL.TextColor3 = Color3.fromRGB(255,255,255)
TL.TextSize = 20
TL.Font = 4
TL.BackgroundTransparency = 1
TL.TextXAlignment = 0

local CB = Instance.new("TextButton", MF)
CB.Text = "✕"
CB.Size = UDim2.new(0, 26, 0, 26)
CB.Position = UDim2.new(0.85, 0, 0.03, 0)
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

local function btn(txt, pos, c)
    local B = Instance.new("TextButton", MF)
    B.Size = UDim2.new(0, 210, 0, 38)
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

local function BG(c)
    local h = c:WaitForChild("Humanoid", 5) 
    if h then h.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None end
    for _, v in ipairs(c:GetDescendants()) do 
        if v:IsA("BillboardGui") and v.Name ~= "BuuName" then v.Enabled = false end 
    end
    local head = c:WaitForChild("Head", 5) 
    if not head or head:FindFirstChild("BuuName") then return end
    local b = Instance.new("BillboardGui", head)
    b.Name = "BuuName"
    b.Size = UDim2.new(0, 200, 0, 50)
    b.StudsOffset = Vector3.new(0, 2.0, 0)
    b.AlwaysOnTop = true
    local t = Instance.new("TextLabel", b)
    t.Size = UDim2.new(1, 0, 1, 0)
    t.BackgroundTransparency = 1
    t.Text = "BuuHub"
    t.TextColor3 = Color3.fromRGB(0, 150, 255)
    t.TextSize = 18
    t.Font = 4
    t.TextStrokeTransparency = 0.5
end
if lp.Character then BG(lp.Character) end 
lp.CharacterAdded:Connect(BG)

local function walk(pos)
    local char = lp.Character or lp.CharacterAdded:Wait()
    local rp = char:WaitForChild("HumanoidRootPart")
    local t = TS:Create(rp, TweenInfo.new((rp.Position - pos).Magnitude / 16, Enum.EasingStyle.Linear), {CFrame = CFrame.new(pos)})
    t:Play() 
    t.Completed:Wait() 
    task.wait(0.5)
end

btn("ตัดไม้ออโต้: ปิดอยู่", UDim2.new(0.06, 0, 0.16, 0), function(b)
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

btn("ทะลุกำแพง: ปิดอยู่", UDim2.new(0.06, 0, 0.32, 0), function(b)
    ncOn = not ncOn 
    setUI(b, ncOn and "ทะลุกำแพง: เปิดอยู่" or "ทะลุกำแพง: ปิดอยู่", ncOn and Color3.fromRGB(0,150,255) or Color3.fromRGB(200,200,200))
end)

RS.Stepped:Connect(function() 
    if ncOn and lp.Character then 
        for _, p in ipairs(lp.Character:GetDescendants()) do 
            if p:IsA("BasePart") then p.CanCollide = false end 
        end 
    end 
end)

local SP = Instance.new("TextBox", MF)
SP.Size = UDim2.new(0, 210, 0, 38)
SP.Position = UDim2.new(0.06, 0, 0.48, 0)
SP.BackgroundColor3 = Color3.fromRGB(30,30,30)
SP.BackgroundTransparency = 0.3
SP.PlaceholderText = "ใส่ตัวเลขความเร็ว... (ปกติ 16)"
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

btn("📍 บันทึกจุดพิกัดปัจจุบัน", UDim2.new(0.06, 0, 0.66, 0), function(b)
    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then 
        savedCF = lp.Character.HumanoidRootPart.CFrame 
        b.Text = "✅ บันทึกจุดสำเร็จ!" 
        b.TextColor3 = Color3.fromRGB(0,255,150) 
        task.wait(1) 
        b.Text = "📍 บันทึกจุดพิกัดปัจจุบัน" 
        b.TextColor3 = Color3.fromRGB(255,215,0) 
    end
end)

btn("🌀 วาร์ปกลับจุดที่เซ็ตไว้", UDim2.new(0.06, 0, 0.82, 0), function(b)
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
