local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "neptune.lol " .. Fluent.Version,
    SubTitle = "by Dot",
    TabWidth = 110,
    Size = UDim2.fromOffset(500, 300),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "" }),
    Misc = Window:AddTab({ Title = "Misc", Icon = "" })
}

local Options = Fluent.Options
local Toggle = Tabs.Main:AddToggle("CamlockToggle", { Title = "Enable Camlock", Default = false })
local CamlockEnabled = false

Toggle:OnChanged(function()
    CamlockEnabled = Options.CamlockToggle.Value
end)

-- Function to calculate Low Arc adjustment
local function LowArc(dist)
    if dist >= 59 and dist <= 61 then
        return -17
    elseif dist >= 62 and dist <= 63 then
        return -16
    elseif dist >= 64 and dist <= 65 then
        return -13
    elseif dist >= 66 and dist <= 67 then
        return -11
    elseif dist >= 68 and dist <= 69 then
        return -8
    elseif dist >= 70 and dist <= 71 then
        return -5
    elseif dist == 72 then
        return -2
    else
        return 0
    end
end

-- Camlock Function (With Low Arc for Shooting)
local function Camlock()
    local Players = game:GetService("Players")
    local Workspace = game:GetService("Workspace")
    local Player = Players.LocalPlayer
    local Character = Player.Character or Player.CharacterAdded:Wait()
    local Camera = Workspace.CurrentCamera
    local Humanoid = Character:WaitForChild("Humanoid")
    
    -- Find the nearest hoop
    local function GetNearestHoop()
        local Distance, TargetHoop = math.huge, nil
        local CharacterPosition = Character.PrimaryPart.Position  

        local function CheckHoops(container)
            if not container then return end
            for _, court in ipairs(container:GetChildren()) do
                for _, Obj in ipairs(court:GetDescendants()) do
                    if Obj.Name == "Swish" and Obj.Parent:FindFirstChildOfClass("TouchTransmitter") then
                        local HoopPosition = Obj.Parent.Position
                        local Magnitude = (CharacterPosition - HoopPosition).Magnitude
                        if Magnitude < Distance then
                            Distance = Magnitude
                            TargetHoop = Obj.Parent
                        end
                    end
                end
            end
        end

        CheckHoops(Workspace:FindFirstChild("Courts"))
        CheckHoops(Workspace:FindFirstChild("PracticeArea"))

        return TargetHoop, Distance
    end

    -- Function to adjust aim (With Low Arc)
    local function AimAtHoop()
        local Hoop, Distance = GetNearestHoop()
        if Hoop then
            local HoopPosition = Hoop.Position
            local PlayerPosition = Character.PrimaryPart.Position
            local ArcAdjustment = LowArc(math.floor(Distance))  -- Apply Low Arc
            local AdjustedHoopPosition = HoopPosition + Vector3.new(0, ArcAdjustment, 0)

            -- Smoothly adjust camera towards the hoop with arc correction
            local NewCameraCFrame = CFrame.new(Camera.CFrame.Position, AdjustedHoopPosition)
            Camera.CFrame = NewCameraCFrame
        end
    end

    -- Detect Jumping and Landing
    Humanoid.StateChanged:Connect(function(_, NewState)
        if CamlockEnabled and NewState == Enum.HumanoidStateType.Jumping then
            Camera.CameraType = Enum.CameraType.Follow
            AimAtHoop()
        elseif NewState == Enum.HumanoidStateType.Landed then
            Camera.CameraType = Enum.CameraType.Custom
        end
    end)
end

-- Activate Camlock
Camlock()
