local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- Variáveis principais
local ball = nil
local controlling = false
local rotating = false
local speed = 100
local keysDown = {}
local rotateRadius = 10
local rotateSpeed = 3
local rotateAngle = 0

-- Função para encontrar a bola
local function findBall()
    return Workspace:FindFirstChild("Football")
end

-- Criação da GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "BallControlGui"

local controlButton = Instance.new("TextButton")
controlButton.Size = UDim2.new(0, 180, 0, 50)
controlButton.Position = UDim2.new(0, 20, 0, 20)
controlButton.Text = "Controlar Bola: OFF"
controlButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
controlButton.TextColor3 = Color3.new(1, 1, 1)
controlButton.Font = Enum.Font.SourceSansBold
controlButton.TextSize = 20
controlButton.Parent = ScreenGui

local stealButton = Instance.new("TextButton")
stealButton.Size = UDim2.new(0, 180, 0, 50)
stealButton.Position = UDim2.new(0, 20, 0, 80)
stealButton.Text = "Roubar Bola"
stealButton.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
stealButton.TextColor3 = Color3.new(1, 1, 1)
stealButton.Font = Enum.Font.SourceSansBold
stealButton.TextSize = 20
stealButton.Parent = ScreenGui

-- Função de controle
local function toggleControl()
    controlling = not controlling
    ball = findBall()

    if controlling and ball then
        controlButton.Text = "Controlar Bola: ON"
        Camera.CameraSubject = ball
        Camera.CameraType = Enum.CameraType.Custom
    else
        controlButton.Text = "Controlar Bola: OFF"
        Camera.CameraSubject = LocalPlayer.Character:FindFirstChild("Humanoid") or LocalPlayer.Character.PrimaryPart
        Camera.CameraType = Enum.CameraType.Custom
        if ball then
            local bv = ball:FindFirstChild("BodyVelocity")
            if bv then bv:Destroy() end
        end
        ball = nil
        rotating = false
    end
end

controlButton.MouseButton1Click:Connect(toggleControl)

-- Roubar Bola
stealButton.MouseButton1Click:Connect(function()
    local b = findBall()
    if not b then return end

    local owner = nil
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if (player.Character.HumanoidRootPart.Position - b.Position).Magnitude < 10 then
                owner = player
                break
            end
        end
    end

    if owner then
        b.CFrame = CFrame.new(LocalPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, 2, 0))
    end
end)

-- Atualizar movimento
local function updateMoveDirection()
    local forward, right, up = 0, 0, 0
    if keysDown.W then forward += 1 end
    if keysDown.S then forward -= 1 end
    if keysDown.D then right += 1 end
    if keysDown.A then right -= 1 end
    if keysDown.Space then up += 1 end
    if keysDown.Ctrl then up -= 1 end

    local moveVec = (Camera.CFrame.LookVector * forward + Camera.CFrame.RightVector * right + Vector3.new(0, 1, 0) * up)
    return moveVec.Magnitude > 0 and moveVec.Unit or Vector3.zero
end

-- Teclas pressionadas
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then toggleControl() end
    if input.KeyCode == Enum.KeyCode.H then rotating = not rotating end
    if input.KeyCode == Enum.KeyCode.E then
        local b = findBall()
        if not b then return end

        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                if (player.Character.HumanoidRootPart.Position - b.Position).Magnitude < 10 then
                    local behind = player.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
                    LocalPlayer.Character:SetPrimaryPartCFrame(behind)
                    break
                end
            end
        end
    end

    keysDown[input.KeyCode.Name] = true
end)

UserInputService.InputEnded:Connect(function(input)
    keysDown[input.KeyCode.Name] = false
end)

-- Loop principal
RunService.Heartbeat:Connect(function()
    if controlling and ball then
        if rotating then
            rotateAngle += rotateSpeed * RunService.Heartbeat:Wait()
            local offset = Vector3.new(math.cos(rotateAngle), 0, math.sin(rotateAngle)) * rotateRadius
            ball.CFrame = CFrame.new(LocalPlayer.Character.HumanoidRootPart.Position + offset)
        else
            local bv = ball:FindFirstChild("BodyVelocity") or Instance.new("BodyVelocity", ball)
            bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
            bv.Velocity = updateMoveDirection() * speed
        end
    end
end)
