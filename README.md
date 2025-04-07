local scriptContent = [[
-- SERVICES
local Players, RunService, VirtualInput, TPS = game:GetService("Players"), game:GetService("RunService"), game:GetService("VirtualInputManager"), game:GetService("TeleportService")
local LocalPlayer, HRP = Players.LocalPlayer, nil
repeat wait() until LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
HRP = LocalPlayer.Character.HumanoidRootPart

-- CONFIG
local skillKeys, panicHP = {"Z", "X", "C", "F", "T", "G"}, 250
local activeKill, activeAutoTrial, activeAimbot, activePanic = false, false, false, false
local conn, trialCompleted, currentRace = nil, false, "Angel"  -- Example for race check

-- UI Setup
local ScreenGui, Frame = Instance.new("ScreenGui", game.CoreGui), Instance.new("Frame", game.CoreGui)
Frame.Size, Frame.Position, Frame.BackgroundTransparency, Frame.BackgroundColor3 = UDim2.new(0, 250, 0, 220), UDim2.new(0.05, 0, 0.5, -110), 0.2, Color3.fromRGB(30, 30, 30)
local buttons = {}

local function createButton(name, pos, text)
    local btn = Instance.new("TextButton", Frame)
    btn.Text, btn.Size, btn.Position, btn.BackgroundColor3, btn.TextColor3, btn.Font, btn.TextSize = text, UDim2.new(0.9, 0, 0, 40), pos, Color3.fromRGB(255, 50, 50), Color3.fromRGB(255, 255, 255), Enum.Font.GothamBold, 16
    buttons[name] = btn
    return btn
end

createButton("StartKill", UDim2.new(0.05, 0, 0, 35), "Auto Kill: Off")
createButton("StartTrial", UDim2.new(0.05, 0, 0, 85), "Auto Trial: Off")
createButton("StartAimbot", UDim2.new(0.05, 0, 0, 135), "Aimbot: Off")
createButton("StartPanic", UDim2.new(0.05, 0, 0, 185), "Panic Mode: Off")

-- Smooth Button Toggle Function
local function toggleButton(btn, state)
    btn.Text, btn.BackgroundColor3 = state and btn.Text:match("(.+): .+") .. ": On" or btn.Text:match("(.+): .+") .. ": Off", state and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
end

-- Functions for Trial, Aimbot, and Panic
local function enterTrial() 
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "Blue Gear" and v:IsA("Model") then
            local prompt = v:FindFirstChildWhichIsA("ProximityPrompt", true)
            if prompt then fireproximityprompt(prompt) end
        end
    end
end

local function completeTrial()
    if trialCompleted then return end
    if currentRace == "Angel" then warn("👼 Completing Angel Trial") end
    trialCompleted = true
end

local function aimbot()
    conn = RunService.RenderStepped:Connect(function()
        if not activeAimbot then return end
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local target = plr.Character
                if target:FindFirstChild("Humanoid") and target.Humanoid.Health > 0 and target:FindFirstChild("TrialStatus") and target.TrialStatus.Value == "InTrial" then
                    HRP.CFrame = target.HumanoidRootPart.CFrame * CFrame.new(0, 0, 2)
                    for _, key in ipairs(skillKeys) do
                        VirtualInput:SendKeyEvent(true, key, false, game)
                        wait(0.05)
                        VirtualInput:SendKeyEvent(false, key, false, game)
                    end
                    break
                end
            end
        end
    end)
end

local function panic()
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
    if hum and hum.Health <= panicHP then
        HRP.CFrame = HRP.CFrame * CFrame.new(math.random(-60, 60), 0, math.random(-60, 60))
        VirtualInput:SendKeyEvent(true, "E", false, game)
        wait(0.1)
        VirtualInput:SendKeyEvent(false, "E", false, game)
    end
end

-- Button Handlers
buttons.StartKill.MouseButton1Click:Connect(function()
    activeKill = not activeKill
    toggleButton(buttons.StartKill, activeKill)
    if activeKill then warn("✅ Auto Kill Started") else if conn then conn:Disconnect() end warn("⛔ Auto Kill Stopped") end
end)

buttons.StartTrial.MouseButton1Click:Connect(function()
    activeAutoTrial = not activeAutoTrial
    toggleButton(buttons.StartTrial, activeAutoTrial)
    if activeAutoTrial then
        enterTrial()
        completeTrial()
        aimbot()
        warn("✅ Auto Trial Started and completed")
    else
        warn("⛔ Auto Trial Stopped")
    end
end)

buttons.StartAimbot.MouseButton1Click:Connect(function()
    activeAimbot = not activeAimbot
    toggleButton(buttons.StartAimbot, activeAimbot)
    if activeAimbot then
        aimbot()
        warn("✅ Aimbot Started")
    else
        warn("⛔ Aimbot Stopped")
    end
end)

buttons.StartPanic.MouseButton1Click:Connect(function()
    activePanic = not activePanic
    toggleButton(buttons.StartPanic, activePanic)
    if activePanic then
        panic()
        warn("✅ Panic Mode Activated")
    else
        warn("⛔ Panic Mode Deactivated")
    end
end)
]]

local loadedScript = loadstring(scriptContent)
if loadedScript then
    loadedScript()
else
    warn("Error: Invalid script")
end
