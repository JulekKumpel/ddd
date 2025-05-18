-- Load Rayfield Library
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Create Window
local Window = Rayfield:CreateWindow({
   Name = "amir x polewej",
   LoadingTitle = "for amir my friendchi",
   LoadingSubtitle = "for amir my friendchi",
   Theme = "Default",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "amir and polegoat",
      FileName = "settings"
   }
})

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local StarterPack = game:GetService("StarterPack")
local StarterPlayer = game:GetService("StarterPlayer")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- Variables
local legSizeValue = 1
local fakeLegsEnabled = false
local fakeLegColor = Color3.fromRGB(255, 255, 255)
local legTransparency = 0
local fakeLeftLeg, fakeRightLeg

local handSizeValue = 1
local fakeHandsEnabled = false
local fakeHandColor = Color3.fromRGB(255, 255, 255)
local handTransparency = 0
local fakeLeftHand, fakeRightHand

local deleteConnection = nil
local autoDeleteEnabled = false

local fileTargets = {
    function() return workspace[player.Name] and workspace[player.Name]:FindFirstChild("CustomCollisionClient") end,
    function() return workspace[player.Name] and workspace[player.Name]:FindFirstChild("Custom Collision stuff") end,
    function() return workspace[player.Name] and workspace[player.Name]:FindFirstChild("AntiCheat") end,
    function() return workspace[player.Name] and workspace[player.Name]:FindFirstChild("ClientServerResponse") end,
    function() return StarterPack:FindFirstChild("ToolManagment") end,
    function() return StarterPlayer.StarterCharacterScripts:FindFirstChild("AntiCheat") end,
    function() return StarterPlayer.StarterCharacterScripts:FindFirstChild("Custom Collision stuff") end,
    function() return StarterPlayer.StarterCharacterScripts:FindFirstChild("CustomCollisionClient") end,
    function() return StarterPlayer.StarterCharacterScripts:FindFirstChild("ClientServerResponse") end,
    function() return player.Backpack:FindFirstChild("ToolManagment") end
}

-- Function to get body parts
local function GetBodyParts()
    if not character then return nil, nil, nil, nil end

    local leftLeg = character:FindFirstChild("LeftLeg") or character:FindFirstChild("Left Leg")
    local rightLeg = character:FindFirstChild("RightLeg") or character:FindFirstChild("Right Leg")
    local leftHand = character:FindFirstChild("LeftHand") or character:FindFirstChild("Left Arm")
    local rightHand = character:FindFirstChild("RightHand") or character:FindFirstChild("Right Arm")

    return leftLeg, rightLeg, leftHand, rightHand
end

-- Function to create fake limbs
local function CreateFakeLimb(realLimb, name, color)
    if not realLimb then return nil end

    local fakeLimb = Instance.new("Part")
    fakeLimb.Name = name
    fakeLimb.Size = Vector3.new(1, 2, 1)
    fakeLimb.Color = color
    fakeLimb.Material = Enum.Material.Plastic
    fakeLimb.CanCollide = false
    fakeLimb.Anchored = false
    fakeLimb.Massless = true
    fakeLimb.Transparency = 0
    fakeLimb.Parent = character

    local weld = Instance.new("Weld")
    weld.Part0 = realLimb
    weld.Part1 = fakeLimb
    weld.C0 = CFrame.new()
    weld.Parent = fakeLimb

    return fakeLimb
end

-- Function to update legs
local function UpdateLegs()
    local leftLeg, rightLeg = GetBodyParts()
    if not leftLeg or not rightLeg then return end

    leftLeg.Size = Vector3.new(legSizeValue, 2, legSizeValue)
    rightLeg.Size = Vector3.new(legSizeValue, 2, legSizeValue)
    leftLeg.Transparency = legTransparency
    rightLeg.Transparency = legTransparency

    if fakeLegsEnabled then
        if not fakeLeftLeg then fakeLeftLeg = CreateFakeLimb(leftLeg, "FakeLeftLeg", fakeLegColor) end
        if not fakeRightLeg then fakeRightLeg = CreateFakeLimb(rightLeg, "FakeRightLeg", fakeLegColor) end
    else
        if fakeLeftLeg then fakeLeftLeg:Destroy() fakeLeftLeg = nil end
        if fakeRightLeg then fakeRightLeg:Destroy() fakeRightLeg = nil end
    end
end

-- Function to update hands
local function UpdateHands()
    local _, _, leftHand, rightHand = GetBodyParts()
    if not leftHand or not rightHand then return end

    leftHand.Size = Vector3.new(handSizeValue, handSizeValue, handSizeValue)
    rightHand.Size = Vector3.new(handSizeValue, handSizeValue, handSizeValue)
    leftHand.Transparency = handTransparency
    rightHand.Transparency = handTransparency

    if fakeHandsEnabled then
        if not fakeLeftHand then fakeLeftHand = CreateFakeLimb(leftHand, "FakeLeftHand", fakeHandColor) end
        if not fakeRightHand then fakeRightHand = CreateFakeLimb(rightHand, "FakeRightHand", fakeHandColor) end
    else
        if fakeLeftHand then fakeLeftHand:Destroy() fakeLeftHand = nil end
        if fakeRightHand then fakeRightHand:Destroy() fakeRightHand = nil end
    end
end

-- Function to make arms non-collidable and massless (R6 support)
local function MakeArmsNonCollidable(character)
    RunService.RenderStepped:Connect(function()
        if character and character.Parent then
            local leftArm = character:FindFirstChild("Left Arm")
            local rightArm = character:FindFirstChild("Right Arm")

            if leftArm and leftArm:IsA("BasePart") then
                leftArm.CanCollide = false
                leftArm.Massless = true
            end

            if rightArm and rightArm:IsA("BasePart") then
                rightArm.CanCollide = false
                rightArm.Massless = true
            end
        end
    end)
end

-- File deletion logic
local function StartAutoDelete()
    if deleteConnection then deleteConnection:Disconnect() end
    deleteConnection = RunService.Heartbeat:Connect(function()
        for _, func in ipairs(fileTargets) do
            local obj = func()
            if obj and obj.Parent then
                obj:Destroy()
            end
        end
    end)
end

local function StopAutoDelete()
    if deleteConnection then
        deleteConnection:Disconnect()
        deleteConnection = nil
    end
end

-- GUI - Leg Tab
local LegTab = Window:CreateTab("Leg Modifier", "footprints")
LegTab:CreateSlider({
   Name = "Leg Size (X and Z)",
   Range = {1, 10},
   Increment = 0.5,
   Suffix = "Size",
   CurrentValue = legSizeValue,
   Flag = "LegSizeSlider",
   Callback = function(Value)
      legSizeValue = Value
      UpdateLegs()
   end
})

LegTab:CreateToggle({
   Name = "Enable Fake Legs",
   CurrentValue = fakeLegsEnabled,
   Flag = "FakeLegsToggle",
   Callback = function(Value)
      fakeLegsEnabled = Value
      UpdateLegs()
   end
})

LegTab:CreateColorPicker({
   Name = "Fake Leg Color",
   Color = fakeLegColor,
   Flag = "FakeLegColorPicker",
   Callback = function(Value)
      fakeLegColor = Value
      if fakeLeftLeg then fakeLeftLeg.Color = Value end
      if fakeRightLeg then fakeRightLeg.Color = Value end
   end
})

LegTab:CreateSlider({
   Name = "Leg Transparency",
   Range = {0, 1},
   Increment = 0.1,
   Suffix = "Transparency",
   CurrentValue = legTransparency,
   Flag = "LegTransparencySlider",
   Callback = function(Value)
      legTransparency = Value
      UpdateLegs()
   end
})

-- GUI - Hand Tab
local HandTab = Window:CreateTab("Hand Modifier", "hand")
HandTab:CreateSlider({
   Name = "Hand Size (X and Z)",
   Range = {1, 10},
   Increment = 0.5,
   Suffix = "Size",
   CurrentValue = handSizeValue,
   Flag = "HandSizeSlider",
   Callback = function(Value)
      handSizeValue = Value
      UpdateHands()
   end
})

HandTab:CreateToggle({
   Name = "Enable Fake Hands",
   CurrentValue = fakeHandsEnabled,
   Flag = "FakeHandsToggle",
   Callback = function(Value)
      fakeHandsEnabled = Value
      UpdateHands()
   end
})

HandTab:CreateColorPicker({
   Name = "Fake Hand Color",
   Color = fakeHandColor,
   Flag = "FakeHandColorPicker",
   Callback = function(Value)
      fakeHandColor = Value
      if fakeLeftHand then fakeLeftHand.Color = Value end
      if fakeRightHand then fakeRightHand.Color = Value end
   end
})

HandTab:CreateSlider({
   Name = "Hand Transparency",
   Range = {0, 1},
   Increment = 0.1,
   Suffix = "Transparency",
   CurrentValue = handTransparency,
   Flag = "HandTransparencySlider",
   Callback = function(Value)
      handTransparency = Value
      UpdateHands()
   end
})

-- GUI - Delete Tab
local DeleteTab = Window:CreateTab("File Cleaner", "trash-2")
DeleteTab:CreateToggle({
   Name = "Persistent File Deletion",
   CurrentValue = autoDeleteEnabled,
   Flag = "AutoDeleteToggle",
   Callback = function(Value)
      autoDeleteEnabled = Value
      if Value then
         StartAutoDelete()
      else
         StopAutoDelete()
      end
   end
})

-- Handle character updates
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    repeat task.wait() until newChar:FindFirstChild("Humanoid")
    MakeArmsNonCollidable(newChar)
    UpdateLegs()
    UpdateHands()
end)

-- Apply initial settings
MakeArmsNonCollidable(character)
UpdateLegs()
UpdateHands()

-- Initial Notification
Rayfield:Notify({
   Title = "VIP Tools Loaded",
   Content = "Fake limbs now stay visible!",
   Duration = 5,
   Image = "check-circle"
})
