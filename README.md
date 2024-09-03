local DrRayLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/AZYsGithub/DrRay-UI-Library/main/DrRay.lua"))()

local window = DrRayLibrary:Load("All things in one tab", "Default")
local tab = DrRayLibrary.newTab("Random shit", "ImageIdHere")

tab.newToggle("Magnets BLATENT", "Toggle! (prints the state)", true, function(toggleState)
    if toggleState then
        print("Magnets Enabled")
        local RunService = game:GetService("RunService")
        local Players = game:GetService("Players")
        local LocalPlayer = Players.LocalPlayer
        local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local CatchRight = Character:WaitForChild("CatchRight")

        local MagPower = 100
        local MagsEnabled = true

        RunService.Heartbeat:Connect(function()
            for _, v in pairs(workspace:GetChildren()) do
                if v:IsA("BasePart") and v.Name == "Football" then
                    if (CatchRight.Position - v.Position).Magnitude <= MagPower and MagsEnabled then
                        firetouchinterest(CatchRight, v, 0)
                        firetouchinterest(CatchRight, v, 1)
                        task.wait()
                        firetouchinterest(CatchRight, v, 0)
                        firetouchinterest(CatchRight, v, 1)
                    end
                end
            end
            task.wait()
        end)
    else
        print("Magnets Disabled")
    end
end)

tab.newToggle("Ball Prediction", "Toggle! (prints the state)", true, function(toggleState)
    if toggleState then
        print("Ball Prediction Enabled")
        local function beamProjectile(g, v0, x0, t1)
            local c = 0.5 * 0.5 * 0.5
            local p3 = 0.5 * g * t1 * t1 + v0 * t1 + x0
            local p2 = p3 - (g * t1 * t1 + v0 * t1) / 3
            local p1 = (c * g * t1 * t1 + 0.5 * v0 * t1 + x0 - c * (x0 + p3)) / (3 * c) - p2
            local curve0 = (p1 - x0).magnitude
            local curve1 = (p2 - p3).magnitude
            local b = (x0 - p3).unit
            local r1 = (p1 - x0).unit
            local u1 = r1:Cross(b).unit
            local r2 = (p2 - p3).unit
            local u2 = r2:Cross(b).unit
            b = u1:Cross(r1).unit
            local cf1 = CFrame.new(x0.x, x0.y, x0.z, r1.x, u1.x, b.x, r1.y, u1.y, b.y, r1.z, u1.z, b.z)
            local cf2 = CFrame.new(p3.x, p3.y, p3.z, r2.x, u2.x, b.x, r2.y, u2.y, b.y, r2.z, u2.z, b.z)
            return curve0, -curve1, cf1, cf2
        end

        local predictionColor = Color3.fromRGB(255, 255, 255)
        local eventConnection
        local Players = game:GetService("Players")
        local LocalPlayer = Players.LocalPlayer
        local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local RunService = game:GetService("RunService")

        eventConnection = workspace.ChildAdded:Connect(function(b)
            if b.Name == "Football" and b:IsA("BasePart") then
                task.wait()
                local vel = b.Velocity
                local pos = b.Position
                local c0, c1, cf1, cf2 = beamProjectile(Vector3.new(0, -28, 0), vel, pos, 10)
                local beam = Instance.new("Beam")
                local a0 = Instance.new("Attachment")
                local a1 = Instance.new("Attachment")
                beam.Color = ColorSequence.new(predictionColor)
                beam.Transparency = NumberSequence.new(0, 0)
                beam.CurveSize0 = c0
                beam.CurveSize1 = c1
                beam.Name = "Hitbox"
                beam.Parent = workspace.Terrain
                beam.Transparency = NumberSequence.new({
                    NumberSequenceKeypoint.new(0, 1),
                    NumberSequenceKeypoint.new(0.01, 0),
                    NumberSequenceKeypoint.new(1, 0),
                    NumberSequenceKeypoint.new(1, 0.01),
                })
                beam.Segments = 1750
                a0.Parent = workspace.Terrain
                a1.Parent = workspace.Terrain
                a0.CFrame = a0.Parent.CFrame:Inverse() * cf1
                a1.CFrame = a1.Parent.CFrame:Inverse() * cf2
                beam.Attachment0 = a0
                beam.Attachment1 = a1
                beam.Width0 = 1
                beam.Width1 = 1

                local landedConnection
                landedConnection = RunService.Heartbeat:Connect(function()
                    if b.Velocity.magnitude < 1 then
                        beam:Destroy()
                        a0:Destroy()
                        a1:Destroy()
                        landedConnection:Disconnect()
                    end
                end)
                repeat
                    task.wait()
                until b.Parent ~= workspace
                beam:Destroy()
                a0:Destroy()
                a1:Destroy()
            end
        end)
    else
        print("Ball Prediction Disabled")
    end
end)

tab.newToggle("Follow Carrier", "Toggle! (prints the state)", true, function(toggleState)
    if toggleState then
        print("Follow Carrier Enabled")
        local player = game.Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        local humanoid = character:WaitForChild("Humanoid")
        local hrp = character:WaitForChild("HumanoidRootPart")
        local catchLeft = character:FindFirstChild("CatchLeft")
        local catchRight = character:FindFirstChild("CatchRight")

        local function findOpponentFootball()
            local nearestFootball = nil
            local shortestDistance = math.huge
            local myTeam = player.Team
            if not myTeam then
                print("No opposite team detected")
                return nil
            end
            for _, v in pairs(workspace:GetChildren()) do
                if v.Name == "Football" then
                    local carrier = v.Parent
                    if carrier:IsA("Model") then
                        local carrierPlayer = game.Players:GetPlayerFromCharacter(carrier)
                        if carrierPlayer and carrierPlayer.Team ~= myTeam then
                            local distance = (hrp.Position - v.Position).Magnitude
                            if distance < shortestDistance then
                                shortestDistance = distance
                                nearestFootball = v
                            end
                        end
                    end
                end
            end
            return nearestFootball
        end

        local function updateFootballPosition()
            local football = findOpponentFootball()
            if football then
                local carrier = football.Parent
                if carrier:IsA("Model") then
                    local carrierPlayer = game.Players:GetPlayerFromCharacter(carrier)
                    if carrierPlayer == player then
                        if catchLeft and catchRight then
                            local footballPosition = (catchLeft.Position + catchRight.Position) / 2
                            football.Position = footballPosition
                        end
                    else
                        humanoid:MoveTo(football.Position)
                        humanoid.MoveToFinished:Wait(2)
                    end
                end
            end
        end

        local running = true
        spawn(function()
            while running do
                updateFootballPosition()
                wait(1)
            end
        end)
    else
        print("Follow Carrier Disabled")
    end
end)

tab.newToggle("WalkSpeed Power 30", "Toggle! (prints the state)", true, function(toggleState)
    if toggleState then
        print("WalkSpeed Power 30 Enabled")
        local Players = game:GetService("Players")
        local LocalPlayer = Players.LocalPlayer
        local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local Humanoid = Character:WaitForChild("Humanoid")

        local walkspeedPower = 30
        Humanoid.WalkSpeed = walkspeedPower
    else
        print("WalkSpeed Power 30 Disabled")
    end
end)

tab.newToggle("JumpPower 45", "Toggle! (prints the state)", true, function(toggleState)
    if toggleState then
        print("JumpPower 45 Enabled")
        local Players = game:GetService("Players")
        local LocalPlayer = Players.LocalPlayer
        local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local Humanoid = Character:WaitForChild("Humanoid")

        local jumpPower = 45
        Humanoid.JumpPower = jumpPower
    else
        print("JumpPower 45 Disabled")
    end
end)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ReplicatedFirst = game:GetService("ReplicatedFirst")
local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remote = ReplicatedStorage:Wait
