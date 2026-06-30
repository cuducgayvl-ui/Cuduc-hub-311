--========================================================================--
--                           CUDUC HUB - UNIVERSAL                        --
--                       Sub-text: script di tugubuvn                     --
--========================================================================--

-- // 1. THÔNG BÁO LÚC LOAD
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "CuHub Universal",
    Text = "script di tugubuvn\nNếu script lỗi hãy nhắn trên tik cho tôi",
    Duration = 7
})

-- // KHỞI TẠO CẤU HÌNH
getgenv().CuHubConfig = {
    AuraEnabled = false,
    AuraRange = 25,
    AttackSpeed = 0.05, -- Để 0.05 cho tốc độ đánh tối đa
    AttackPlayers = false
}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local VirtualUser = game:GetService("VirtualUser")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")

-- // CHỐNG AFK
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new(0, 0))
end)

-- // GIAO DIỆN UI (NÚT KILL)
if CoreGui:FindFirstChild("CuHub_KillUI") then CoreGui["CuHub_KillUI"]:Destroy() end

local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "CuHub_KillUI"
local KillButton = Instance.new("TextButton", ScreenGui)
KillButton.Size = UDim2.new(0, 120, 0, 50)
KillButton.Position = UDim2.new(0.5, -60, 0.2, 0)
KillButton.BackgroundColor3 = Color3.fromRGB(220, 53, 69)
KillButton.Text = "Kill"
KillButton.TextColor3 = Color3.fromRGB(255, 255, 255)
KillButton.Font = Enum.Font.SourceSansBold
KillButton.TextSize = 22
KillButton.BorderSizePixel = 0
Instance.new("UICorner", KillButton).CornerRadius = UDim.new(0, 8)

-- Logic Bật/Tắt UI
KillButton.MouseButton1Click:Connect(function()
    getgenv().CuHubConfig.AuraEnabled = not getgenv().CuHubConfig.AuraEnabled
    local color = getgenv().CuHubConfig.AuraEnabled and Color3.fromRGB(40, 167, 69) or Color3.fromRGB(220, 53, 69)
    TweenService:Create(KillButton, TweenInfo.new(0.2), {BackgroundColor3 = color}):Play()
end)

-- // HÀM ĐÁNH NHANH (FAST ATTACK)
local function fastAttack(targetPart)
    local found = false
    -- Tự động quét tìm RemoteEvent của game để spam sát thương
    for _, v in pairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
        if v:IsA("RemoteEvent") and (v.Name:lower():find("combat") or v.Name:lower():find("attack") or v.Name:lower():find("damage")) then
            v:FireServer(targetPart)
            found = true
        end
    end
    -- Nếu không thấy Remote, dùng cách mặc định
    if not found then
        local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
        if tool then tool:Activate() end
    end
end

-- // VÒNG LẶP CHÍNH
task.spawn(function()
    while task.wait(getgenv().CuHubConfig.AttackSpeed) do
        if getgenv().CuHubConfig.AuraEnabled and LocalPlayer.Character then
            local myPos = LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.Position
            if myPos then
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj ~= LocalPlayer.Character then
                        local hum = obj:FindFirstChildOfClass("Humanoid")
                        local root = obj:FindFirstChild("HumanoidRootPart")
                        if hum and root and hum.Health > 0 then
                            if not getgenv().CuHubConfig.AttackPlayers and Players:GetPlayerFromCharacter(obj) then continue end
                            
                            if (myPos - root.Position).Magnitude <= getgenv().CuHubConfig.AuraRange then
                                fastAttack(root)
                            end
                        end
                    end
                end
            end
        end
    end
end)

print("-[ CuHub Loaded Successfully! ]-")

