-- 👤 Stalker + Seek Music Changer

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera
local humanoid = char:WaitForChild("Humanoid")

-- ================== STALKER ==================
local stalkerImages = {
    "rbxassetid://115047824955445",
    "rbxassetid://73629850429939",
    "rbxassetid://71725176156204",
    "rbxassetid://122094429760163",
    "rbxassetid://100917760105588",
    "rbxassetid://127842346062233",
    "rbxassetid://94733115025990"
}

local currentStalker = nil
local activated = false
local achievementGiven = false

local spawnSound = 136833080474934
local randomScarySounds = {136350971091939, 113917217579668}

local function caption(text)
    local MainUI = player:WaitForChild("PlayerGui"):WaitForChild("MainUI")
    local func = require(MainUI.Initiator.Main_Game)
    func.caption(text, true)
end

local function playSound(id, volume)
    local sound = Instance.new("Sound", Workspace)
    sound.SoundId = "rbxassetid://" .. id
    sound.Volume = volume or 3.5
    sound:Play()
    Debris:AddItem(sound, 6)
end

local function giveAchievement()
    if achievementGiven then return end
    achievementGiven = true

    local achievementGiver = loadstring(game:HttpGet("https://raw.githubusercontent.com/RegularVynixu/Utilities/main/Doors/Custom%20Achievements/Source.lua"))()
    achievementGiver({
        Title = "You Survived the Stalker",
        Desc = "Stalker was watching you.",
        Reason = "You got away from the Stalker.",
        Image = "rbxassetid://112480536020076"
    })
end

local function spawnStalker()
    if currentStalker then currentStalker:Destroy() end

    local stalkerPart = Instance.new("Part")
    stalkerPart.Name = "Stalker"
    stalkerPart.Size = Vector3.new(9, 12, 2.5)
    stalkerPart.Transparency = 1
    stalkerPart.Anchored = true
    stalkerPart.CanCollide = false
    stalkerPart.Parent = Workspace

    local randomImage = stalkerImages[math.random(1, #stalkerImages)]

    for _, face in pairs(Enum.NormalId:GetEnumItems()) do
        local decal = Instance.new("Decal")
        decal.Texture = randomImage
        decal.Face = face
        decal.Parent = stalkerPart
    end

    stalkerPart.CFrame = hrp.CFrame * CFrame.new(0, 3, 24)

    task.spawn(function()
        while stalkerPart.Parent do
            stalkerPart.CFrame = CFrame.new(stalkerPart.Position, camera.CFrame.Position)
            task.wait(0.03)
        end
    end)

    playSound(spawnSound, 4)

    if math.random(1, 3) == 1 then
        playSound(randomScarySounds[math.random(1, #randomScarySounds)], 3.5)
    end

    currentStalker = stalkerPart

    task.delay(math.random(45, 90), function()
        if currentStalker then currentStalker:Destroy() end
    end)
end

-- JUMPSCARE (Stalker 3)
local function jumpscare()
    local js = Instance.new("Part")
    js.Name = "StalkerJumpscare"
    js.Size = Vector3.new(7, 10, 1.5)
    js.Transparency = 1
    js.Anchored = true
    js.CanCollide = false
    js.Parent = Workspace

    local img = "rbxassetid://73629850429939"

    for _, face in pairs(Enum.NormalId:GetEnumItems()) do
        local decal = Instance.new("Decal")
        decal.Texture = img
        decal.Face = face
        decal.Parent = js
    end

    js.CFrame = hrp.CFrame * CFrame.new(0, 3, -6)  -- Mais baixo

    task.spawn(function()
        while js.Parent do
            js.CFrame = CFrame.new(js.Position, camera.CFrame.Position)
            task.wait(0.03)
        end
    end)

    task.wait(1.4)
    js:Destroy()
end

-- Sombra + Cego + Dano
task.spawn(function()
    while true do
        if currentStalker then
            local distance = (currentStalker.Position - hrp.Position).Magnitude
            if distance < 9 then
                local oldFogEnd = Lighting.FogEnd
                local oldAmbient = Lighting.Ambient
                local oldBrightness = Lighting.Brightness
                
                Lighting.FogColor = Color3.new(0,0,0)
                Lighting.FogEnd = 18
                Lighting.Ambient = Color3.new(0.01,0.01,0.01)
                Lighting.Brightness = 0.05

                caption("...")

                humanoid:TakeDamage(20)

                task.wait(2.3)

                Lighting.FogEnd = oldFogEnd
                Lighting.Ambient = oldAmbient
                Lighting.Brightness = oldBrightness
                
                giveAchievement()

                if currentStalker then
                    currentStalker:Destroy()
                    currentStalker = nil
                end
            end
        end
        task.wait(0.1)
    end
end)

-- ================== SEEK MUSIC ==================
local NEW_SEEK_ID = "rbxassetid://125959136412325"

local changedSounds = {}

local function changeSeekMusic()
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("Sound") and not changedSounds[obj] then
            local nameLower = obj.Name:lower()
            local parentName = obj.Parent and obj.Parent.Name:lower() or ""

            if nameLower:find("seek") or nameLower:find("chase") or parentName:find("seek") or parentName:find("chase") then
                changedSounds[obj] = true
                
                local wasPlaying = obj.IsPlaying
                local oldTime = obj.TimePosition
                
                obj.SoundId = NEW_SEEK_ID
                obj.Volume = 3.8
                obj.PlaybackSpeed = 1
                obj.Looped = true
                
                if wasPlaying then
                    obj:Stop()
                    task.wait(0.05)
                    obj.TimePosition = oldTime
                    obj:Play()
                end
            end
        end
    end
end

task.spawn(function()
    while true do
        changeSeekMusic()
        task.wait(2)
    end
end)

ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    task.wait(1)
    changeSeekMusic()
end)

task.wait(3)
changeSeekMusic()

-- ================== ATIVAÇÃO ==================
ReplicatedStorage.GameData.LatestRoom.Changed:Connect(function()
    if not activated then
        activated = true
        caption("Stalker Mod Activated!!!")
        task.wait(2)
        caption("Credit: Twixxel")
    end
end)

-- Spawn aleatório
task.spawn(function()
    while true do
        task.wait(math.random(35, 70))
        if activated and math.random(1, 4) == 1 then
            spawnStalker()
        end
    end
end)

-- Jumpscare aleatório
task.spawn(function()
    while true do
        task.wait(math.random(40, 85))
        if activated and math.random(1, 5) == 1 then
            jumpscare()
        end
    end
end)

player.Chatted:Connect(function(msg)
    local m = msg:lower()
    if m == "/stalker" then
        spawnStalker()
    elseif m == "/stalker3" then
        jumpscare()
    end
end)

print("✅ Stalker + Seek Music carregado!")
print("/stalker = Stalker normal")
print("/stalker3 = Jumpscare")


-- 👤 Stalker 2 - Chase Mode (Dano 40)

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera
local humanoid = char:WaitForChild("Humanoid")

local CHASE_IMAGE = "rbxassetid://91688967181531"
local CHASE_MUSIC = "rbxassetid://129085629036594"

local chaseEntity = nil
local chaseMusic = nil

local function caption(text)
    local MainUI = player:WaitForChild("PlayerGui"):WaitForChild("MainUI")
    local func = require(MainUI.Initiator.Main_Game)
    func.caption(text, true)
end

local function startChaseMode()
    if chaseEntity then return end

    caption("It is 499 blocks away from you")
    
    chaseMusic = Instance.new("Sound", Workspace)
    chaseMusic.SoundId = CHASE_MUSIC
    chaseMusic.Volume = 3.2
    chaseMusic.Looped = true
    chaseMusic:Play()

    local chaser = Instance.new("Part")
    chaser.Size = Vector3.new(7, 13, 2)
    chaser.Transparency = 1
    chaser.Anchored = true
    chaser.CanCollide = false
    chaser.Parent = Workspace

    local decal = Instance.new("Decal")
    decal.Texture = CHASE_IMAGE
    decal.Face = Enum.NormalId.Front
    decal.Parent = chaser

    chaser.CFrame = hrp.CFrame * CFrame.new(0, 5, 70)

    chaseEntity = chaser

    local distance = 499
    local speed = 8

    task.spawn(function()
        while chaseEntity do
            distance = math.max(5, distance - 24)
            speed = speed + 2.1

            caption("It is " .. math.floor(distance) .. " blocks away from you")

            Lighting.Ambient = Color3.fromRGB(140, 0, 0)
            Lighting.Brightness = 0.35

            local dir = (hrp.Position - chaser.Position).Unit
            chaser.CFrame = CFrame.new(chaser.Position, hrp.Position)

            chaser.Position = chaser.Position + dir * speed * 0.18

            if distance <= 10 then
                caption("RUN")
                speed = speed + 10
            end

            if (chaser.Position - hrp.Position).Magnitude < 9 then
                humanoid:TakeDamage(40)   -- Dano 40 ao invés de matar
                
                task.wait(1.5)
                if chaseEntity then chaseEntity:Destroy() end
                if chaseMusic then chaseMusic:Stop() end
                
                Lighting.Ambient = Color3.new(1,1,1)
                Lighting.Brightness = 1
                break
            end
            task.wait(0.18)
        end
    end)
end

-- Spawn aleatório BEM RARO
task.spawn(function()
    while true do
        task.wait(math.random(80, 160))
        if math.random(1, 6) == 1 then
            startChaseMode()
        end
    end
end)

player.Chatted:Connect(function(msg)
    if msg:lower() == "/stalker2" then
        startChaseMode()
    end
end)

print("👤 Stalker 2 carregado!")
print("Use /stalker2 para forçar")
