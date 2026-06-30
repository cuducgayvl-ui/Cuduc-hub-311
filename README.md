--========================================================================--
--                           CUDUC HUB - UNIVERSAL                        --
--                       Sub-text: by cuduc / tugubuvn                    --
--========================================================================--

-- // 1. HIỂN THỊ THÔNG BÁO LÚC LOAD SCRIPT
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "CuHub Universal",
    Text = "script di tugubuvn\nNếu script lỗi hãy nhắn trên tik cho tôi",
    Icon = "rbxassetid://121632477410656", -- Sử dụng ID ảnh logo cún quen thuộc của bạn
    Duration = 7
})

-- // KHỞI TẠO CẤU HÌNH HỆ THỐNG
getgenv().CuHubConfig = {
    AuraEnabled = false,       -- Mặc định ban đầu là tắt
    AuraRange = 25,            -- Khoảng cách quét mục tiêu (đấm)
    AttackSpeed = 0.1,         -- Tốc độ quét & đấm (0.1 giây giúp máy mát)
    AttackPlayers = false      -- false: Chỉ đấm Quái/NPC | true: Đấm cả Player
}

-- // CÁC HÀM HỆ THỐNG CẦN THIẾT
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local VirtualUser = game:GetService("VirtualUser")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")

-- // TỰ ĐỘNG CHỐNG KICK AFK (Hỗ trợ treo máy 24/7)
if getgenv().AntiAfkConnected == nil then
    getgenv().AntiAfkConnected = true
    LocalPlayer.Idled:Connect(function()
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new(0, 0))
    end)
end

-- // 2. TẠO GIAO DIỆN UI (Hình chữ nhật nút "Kill")
if CoreGui:FindFirstChild("CuHub_KillUI") then
    CoreGui["CuHub_KillUI"]:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CuHub_KillUI"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

-- Nút bấm hình chữ nhật (Mặc định màu Đỏ khi tắt)
local KillButton = Instance.new("TextButton")
KillButton.Name = "KillButton"
KillButton.Parent = ScreenGui
KillButton.Size = UDim2.new(0, 120, 0, 50)
KillButton.Position = UDim2.new(0.5, -60, 0.2, 0)
KillButton.BackgroundColor3 = Color3.fromRGB(220, 53, 69) -- Màu đỏ gốc
KillButton.Text = "Kill"
KillButton.TextColor3 = Color3.fromRGB(255, 255, 255)
KillButton.Font = Enum.Font.SourceSansBold
KillButton.TextSize = 22
KillButton.BorderSizePixel = 0
KillButton.AutoButtonColor = false

-- Thêm bo góc nhẹ cho nút hình chữ nhật
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = KillButton

-- Tạo viền nhẹ cho nút
local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(255, 255, 255)
UIStroke.Thickness = 1.5
UIStroke.Transparency = 0.5
UIStroke.Parent = KillButton

-- [Tính năng kéo/di chuyển nút trên màn hình]
local dragging, dragInput, dragStart, startPos
KillButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = KillButton.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
KillButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        KillButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- [Sự kiện xử lý khi bấm nút: Đổi màu + Bật/Tắt Melee Aura]
KillButton.MouseButton1Click:Connect(function()
    getgenv().CuHubConfig.AuraEnabled = not getgenv().CuHubConfig.AuraEnabled
    
    if getgenv().CuHubConfig.AuraEnabled then
        -- Khi bật: Chuyển sang màu Xanh lá
        TweenService:Create(KillButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(40, 167, 69)}):Play()
    else
        -- Khi tắt: Quay về màu Đỏ
        TweenService:Create(KillButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(220, 53, 69)}):Play()
    end
end)

-- // 3. HÀM KIỂM TRA MỤC TIÊU HỢP LỆ (Quét mọi game)
local function checkTarget(model)
    if not model:IsA("Model") or model == LocalPlayer.Character then return nil end
    if not getgenv().CuHubConfig.AttackPlayers and Players:GetPlayerFromCharacter(model) then 
        return nil 
    end
    
    local humanoid = model:FindFirstChildOfClass("Humanoid")
    local rootPart = model:FindFirstChild("HumanoidRootPart")
    
    if humanoid and rootPart and humanoid.Health > 0 then
        return rootPart
    end
    return nil
end

-- // 4. HÀM TỰ ĐỘNG LẤY VŨ KHÍ TRÊN TAY (Auto Equip)
local function autoEquipWeapon()
    local char = LocalPlayer.Character
    if not char then return nil end
    
    local currentTool = char:FindFirstChildOfClass("Tool")
    if currentTool then return currentTool end
    
    local backpackTool = LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
    if backpackTool then
        backpackTool.Parent = char
        return backpackTool
    end
    return nil
end

-- // 5. VÒNG LẶP CHÍNH CỦA MELEE AURA
task.spawn(function()
    while task.wait(getgenv().CuHubConfig.AttackSpeed) do
        if getgenv().CuHubConfig.AuraEnabled then
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                local myPos = char.HumanoidRootPart.Position
                
                -- Quét sâu toàn bộ Folder/Thư mục trong Workspace của mọi game
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj:IsA("Model") then
                        local targetPart = checkTarget(obj)
                        if targetPart then
                            local distance = (myPos - targetPart.Position).Magnitude
                            
                            -- Nếu mục tiêu nằm trong tầm đánh
                            if distance <= getgenv().CuHubConfig.AuraRange then
                                local weapon = autoEquipWeapon()
                                if weapon then
                                    weapon:Activate() -- Thực hiện đấm/chém
                                    VirtualUser:Button1Down(Vector2.new(0, 0), workspace.CurrentCamera.CFrame) -- Click ảo bypass anti
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end)
