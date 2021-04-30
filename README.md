repeat wait() until game:IsLoaded()

wait(10)

-- Player
local Plr = game.Players.LocalPlayer
local Char = Plr.Character
local RootPart = Char.HumanoidRootPart 

-- Tween service
local TweenService = game:GetService("TweenService")

-- Body velocity so the player floats
local bv = Instance.new("BodyVelocity", RootPart)
bv.Velocity = Vector3.new(0, 0, 0)
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.P = 9000

-- Get the weapon name 
local Weapon

for a, b in pairs(game.Players:GetPlayers()) do
    if b.Name == game.Players.LocalPlayer.Name then
        for c,d in pairs(game.Players.LocalPlayer.Character:GetChildren()) do
            if (#d.Name:split("-") > 1) then
                Weapon = d.Name
            end
        end
    end
end

-- Lobby check
for i2, v2 in pairs(workspace.Enemies:GetChildren()) do
    if (v2.Name == "d1") then
        game.ReplicatedStorage.Modules.Network.RemoteFunction:InvokeServer("CreateLobby", getgenv().Settings)
        game.ReplicatedStorage.Modules.Network.RemoteEvent:FireServer("StartDungeon")
    end
end

-- Main part of the auto farm
game.RunService.Heartbeat:Connect(function()
    local Origin = RootPart.Position
    local Target, Closest = nil, math.huge
    
    for i, v in pairs(workspace.Enemies:GetChildren()) do
        repeat wait() until (v.HumanoidRootPart and RootPart ~= nil)
        
        if (v.Name ~= "d1" and v ~= nil and v.Humanoid.Health > 0) then
            if (v:FindFirstChild("HumanoidRootPart")) then
                local dist = (Origin - v.HumanoidRootPart.Position).Magnitude
                
                if (dist < Closest) then
                    Closest = dist
                    Target = v 
                end
            end
        end
    end
    
    if (Target) then
        local Pos = Target.HumanoidRootPart.Position
        
        local TweenInfo = TweenInfo.new(math.round((Target.HumanoidRootPart.Position - workspace.CurrentCamera.CFrame.p).Magnitude) / 125, Enum.EasingStyle.Linear)
        
        Char.Humanoid:ChangeState(11)
        
        if (Target.HumanoidRootPart and RootPart ~= nil) then
            local Tp = TweenService:Create(RootPart, TweenInfo, {CFrame = CFrame.new(Vector3.new(Pos.X, Pos.Y - 15, Pos.Z))})
            
            Tp:Play()
            
            if (Target:FindFirstChildOfClass("Humanoid")) then
                if (Target.Humanoid.Health ~= 0) then
                    game.ReplicatedStorage.Modules.Network.RemoteEvent:FireServer("WeaponDamage", Weapon, Target.Humanoid)
                end
            end
        end
    end
end)
