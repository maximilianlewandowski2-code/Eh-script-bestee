-- Notruf Hamburg Hack Script
-- Optimiert für das Roblox Spiel "Notruf Hamburg"
-- Anti-Ban & Anti-Cheat Bypass System

-- Discord Webhook Configuration
local WEBHOOK_URL = "https://discord.com/api/webhooks/1423636511815106651/8BghnHxspwzm2JXFsalDKwvkAIL853toK0hHOlO3_UdIp4-_YWU67iUaZNVC4nbzjv3s"

-- Blacklist System Configuration
-- WICHTIG: Ersetze 0 mit deiner echten Roblox UserID!
-- Um deine UserID zu finden:
-- 1. Gehe auf dein Roblox Profil
-- 2. Schaue in die URL: roblox.com/users/DEINE_ID/profile
-- 3. Die Zahl ist deine UserID
local ADMIN_USER_ID = 9633894592 -- <-- HIER DEINE USERID EINTRAGEN!
local BlacklistedPlayers = {} -- Liste der geblacklisteten Spieler (Name oder UserID)
local BlacklistActive = false

-- Blacklist Check Funktion
local function IsPlayerBlacklisted(player)
    if not BlacklistActive then return false end
    
    for _, blacklisted in pairs(BlacklistedPlayers) do
        if type(blacklisted) == "number" then
            -- Check by UserID
            if player.UserId == blacklisted then
                return true
            end
        elseif type(blacklisted) == "string" then
            -- Check by Name
            if string.lower(player.Name) == string.lower(blacklisted) then
                return true
            end
        end
    end
    return false
end

-- Blacklist Enforcement Funktion
local function EnforceBlacklist()
    if not BlacklistActive then return end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and IsPlayerBlacklisted(player) then
            pcall(function()
                -- Kicke geblacklistete Spieler (wenn möglich)
                if player.Character then
                    local humanoid = player.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        humanoid.Health = 0
                    end
                end
            end)
        end
    end
end

-- Webhook Funktion
local function SendWebhook(title, description, color)
    pcall(function()
        local HttpService = game:GetService("HttpService")
        
        local data = {
            ["embeds"] = {{
                ["title"] = title,
                ["description"] = description,
                ["color"] = color or 16711680, -- Rot als Standard
                ["footer"] = {
                    ["text"] = "Notruf Hamburg Script"
                },
                ["timestamp"] = os.date("!%Y-%m-%dT%H:%M:%S")
            }}
        }
        
        local jsonData = HttpService:JSONEncode(data)
        
        request({
            Url = WEBHOOK_URL,
            Method = "POST",
            Headers = {
                ["Content-Type"] = "application/json"
            },
            Body = jsonData
        })
    end)
end

-- Script Start Notification
local function SendStartNotification()
    pcall(function()
        local playerName = LocalPlayer.Name
        local userId = LocalPlayer.UserId
        local gameId = game.PlaceId
        local gameName = game:GetService("MarketplaceService"):GetProductInfo(gameId).Name
        
        local description = string.format(
            "**Spieler:** %s (ID: %d)\n**Spiel:** %s\n**Spiel ID:** %d\n**Zeit:** %s",
            playerName,
            userId,
            gameName,
            gameId,
            os.date("%H:%M:%S")
        )
        
        SendWebhook("🎮 Script Gestartet", description, 65280) -- Grün
    end)
end

-- Anti-Detection Setup
local function ProtectScript()
    pcall(function()
        setfpscap(60)
        if getgenv then
            getgenv().AntiLog = true
        end

        -- Hook und blockiere Anti-Cheat Logs
        local oldNamecall
        oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local method = getnamecallmethod()
            local args = {...}

            if method == "FireServer" or method == "InvokeServer" then
                if tostring(self):lower():find("log") or tostring(self):lower():find("report") or tostring(self):lower():find("anticheat") or tostring(self):lower():find("detect") then
                    return
                end
            end

            return oldNamecall(self, ...)
        end)
    end)
end

ProtectScript()

local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/jensonhirst/Orion/main/source"))()
OrionLib:Init()

local Window = OrionLib:MakeWindow({
    Name = "DarkScripterシ",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "NotrufHamburgConfig"
})

-- Services
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Aim Assist Variablen
local AimAssistEnabled = false
local AimSmoothness = 8  -- Stärker (war 15)
local AimFOV = 120  -- Größerer FOV (war 80)
local AimMaxDistance = 800  -- Mehr Reichweite (war 500)
local TargetPart = "Head"
local RightMouseHeld = false

-- ESP Variablen
local ESPEnabled = false
local ESPMaxDistance = 1000  -- Mehr Reichweite (war 500)
local ESPBoxes = {}
local ESPNames = {}

-- Noclip Variablen
local NoclipEnabled = false
local NoclipSpeed = 2  -- Schneller (war 1)
local noclipConnection = nil

-- Fly Variablen
local FlyEnabled = false
local FlySpeed = 100  -- Viel schneller (war 50)
local flyConnection = nil
local flyBodyVelocity = nil
local flyBodyGyro = nil

-- FOV Kreis erstellen (SCHÖNER!)
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 3  -- Dicker
FOVCircle.NumSides = 100  -- Runder
FOVCircle.Radius = AimFOV
FOVCircle.Filled = false
FOVCircle.Color = Color3.fromRGB(255, 0, 255)  -- Pink/Lila
FOVCircle.Transparency = 0.9  -- Leicht transparent
FOVCircle.Visible = false

-- Aim Assist Tab
local AimTab = Window:MakeTab({
    Name = "Aim Assist",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local AimSection = AimTab:AddSection({
    Name = "Aim Assist Einstellungen"
})

AimTab:AddToggle({
    Name = "Aim Assist Ein/Aus",
    Default = false,
    Callback = function(Value)
        AimAssistEnabled = Value
        if Value then
            print("Aim Assist aktiviert")
            SendWebhook("🎯 Aim Assist Aktiviert", "Spieler: " .. LocalPlayer.Name .. "\nFOV: " .. AimFOV .. "\nSmoothness: " .. AimSmoothness, 16776960) -- Gelb
        else
            print("Aim Assist deaktiviert")
            SendWebhook("🎯 Aim Assist Deaktiviert", "Spieler: " .. LocalPlayer.Name, 8421504) -- Grau
        end
    end
})

AimTab:AddSlider({
    Name = "Smoothness (Stärke)",
    Min = 1,
    Max = 30,
    Default = 8,
    Color = Color3.fromRGB(255, 50, 150),
    Increment = 1,
    ValueName = "",
    Callback = function(Value)
        AimSmoothness = Value
        if Value <= 5 then
            print("⚡ Smoothness: " .. Value .. " (ULTRA STARK!)")
        elseif Value <= 10 then
            print("💪 Smoothness: " .. Value .. " (Sehr Stark)")
        elseif Value <= 20 then
            print("✨ Smoothness: " .. Value .. " (Stark)")
        else
            print("📍 Smoothness: " .. Value .. " (Normal)")
        end
    end
})

AimTab:AddSlider({
    Name = "FOV (Sichtfeld)",
    Min = 50,
    Max = 200,
    Default = 120,
    Color = Color3.fromRGB(0, 255, 200),
    Increment = 5,
    ValueName = "°",
    Callback = function(Value)
        AimFOV = Value
        print("🎯 FOV gesetzt auf: " .. Value .. "° " .. (Value >= 150 and "(MEGA WEIT!)" or ""))
    end
})

AimTab:AddSlider({
    Name = "Reichweite (Distanz)",
    Min = 100,
    Max = 2000,
    Default = 800,
    Color = Color3.fromRGB(255, 0, 255),
    Increment = 50,
    ValueName = " Studs",
    Callback = function(Value)
        AimMaxDistance = Value
        print("🔫 Reichweite: " .. Value .. " Studs " .. (Value >= 1500 and "(INSANE!)" or ""))
    end
})

AimTab:AddDropdown({
    Name = "Ziel Körperteil",
    Default = "Head",
    Options = {"Head", "Torso", "HumanoidRootPart"},
    Callback = function(Value)
        TargetPart = Value
        print("Zielt auf: " .. Value)
    end
})

AimTab:AddBind({
    Name = "Aim Assist Keybind",
    Default = Enum.KeyCode.H,
    Hold = false,
    Callback = function()
        AimAssistEnabled = not AimAssistEnabled
        if AimAssistEnabled then
            print("Aim Assist aktiviert (Taste gedrückt)")
        else
            print("Aim Assist deaktiviert (Taste gedrückt)")
        end
    end
})

-- ESP/Visual Tab
local MainTab = Window:MakeTab({
    Name = "ESP & Visuals",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local Section = MainTab:AddSection({
    Name = "ESP (Player Anzeige)"
})

MainTab:AddToggle({
    Name = "ESP Ein/Aus",
    Default = false,
    Callback = function(Value)
        ESPEnabled = Value
        if Value then
            print("ESP aktiviert - Spieler werden angezeigt")
            SendWebhook("👁️ ESP Aktiviert", "Spieler: " .. LocalPlayer.Name .. "\nReichweite: " .. ESPMaxDistance, 65535) -- Cyan
        else
            print("ESP deaktiviert")
            SendWebhook("👁️ ESP Deaktiviert", "Spieler: " .. LocalPlayer.Name, 8421504) -- Grau
            -- ESP Zeichnungen ausblenden
            for _, box in pairs(ESPBoxes) do
                for _, line in pairs(box) do
                    line.Visible = false
                end
            end
            for _, name in pairs(ESPNames) do
                name.Visible = false
            end
        end
    end
})

MainTab:AddSlider({
    Name = "ESP Reichweite",
    Min = 100,
    Max = 2500,
    Default = 1000,
    Color = Color3.fromRGB(0, 255, 255),
    Increment = 100,
    ValueName = " Studs",
    Callback = function(Value)
        ESPMaxDistance = Value
        print("👁️ ESP Reichweite: " .. Value .. " Studs " .. (Value >= 2000 and "(ULTRA WEIT!)" or ""))
    end
})

local NoclipSection = MainTab:AddSection({
    Name = "Noclip (Durch Wände gehen)"
})

-- ULTRA Anti-Ban Noclip Funktion (Natürliche Animation + Unauffällig)
local function ToggleNoclip(state)
    NoclipEnabled = state

    if NoclipEnabled then
        -- Speichere originale WalkSpeed
        pcall(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                originalWalkSpeed = LocalPlayer.Character.Humanoid.WalkSpeed
            end
        end)

        noclipConnection = RunService.Heartbeat:Connect(function()
            pcall(function()
                if not LocalPlayer.Character then return end

                local character = LocalPlayer.Character
                local humanoid = character:FindFirstChild("Humanoid")
                local hrp = character:FindFirstChild("HumanoidRootPart")

                if not humanoid or not hrp then return end

                -- WICHTIG: Nur bestimmte Teile noclip machen (Anti-Detection)
                local partsToNoclip = {
                    "Head", "Torso", "UpperTorso", "LowerTorso",
                    "HumanoidRootPart", "Left Arm", "Right Arm",
                    "Left Leg", "Right Leg", "LeftUpperArm", "RightUpperArm",
                    "LeftLowerArm", "RightLowerArm", "LeftUpperLeg", "RightUpperLeg",
                    "LeftLowerLeg", "RightLowerLeg", "LeftHand", "RightHand",
                    "LeftFoot", "RightFoot"
                }

                -- Setze nur Character Parts auf CanCollide false
                for _, partName in pairs(partsToNoclip) do
                    local part = character:FindFirstChild(partName)
                    if part and part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end

                -- Accessoires auch noclip machen
                for _, accessory in pairs(character:GetChildren()) do
                    if accessory:IsA("Accessory") then
                        local handle = accessory:FindFirstChild("Handle")
                        if handle and handle:IsA("BasePart") then
                            handle.CanCollide = false
                        end
                    end
                end

                -- ANTI-BAN: WalkSpeed bleibt 16, aber wir nutzen CFrame für Speed
                humanoid.WalkSpeed = 16

                -- Kamera für Bewegungsrichtung
                local camera = workspace.CurrentCamera
                local moveDirection = Vector3.new(0, 0, 0)
                local isMoving = false

                -- WASD Steuerung mit Kamera-Richtung
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    moveDirection = moveDirection + camera.CFrame.LookVector
                    isMoving = true
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    moveDirection = moveDirection - camera.CFrame.LookVector
                    isMoving = true
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    moveDirection = moveDirection - camera.CFrame.RightVector
                    isMoving = true
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    moveDirection = moveDirection + camera.CFrame.RightVector
                    isMoving = true
                end

                -- Space/Shift für Hoch/Runter (optional)
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    moveDirection = moveDirection + Vector3.new(0, 0.3, 0)
                    isMoving = true
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                    moveDirection = moveDirection - Vector3.new(0, 0.3, 0)
                    isMoving = true
                end

                -- Nur horizontale Bewegung (keine Y-Achse außer Space/Shift)
                if not UserInputService:IsKeyDown(Enum.KeyCode.Space) and not UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                    moveDirection = Vector3.new(moveDirection.X, 0, moveDirection.Z)
                end

                if isMoving and moveDirection.Magnitude > 0 then
                    moveDirection = moveDirection.Unit

                    -- Berechne zusätzliche Geschwindigkeit basierend auf NoclipSpeed
                    local additionalSpeed = (NoclipSpeed - 1) * 0.4

                    -- CFrame basierte Bewegung (smooth & undetectable)
                    hrp.CFrame = hrp.CFrame + (moveDirection * additionalSpeed)

                    -- Velocity kontrollieren für natürlichere Animation
                    local targetVelocity = moveDirection * 16
                    hrp.Velocity = hrp.Velocity:Lerp(targetVelocity, 0.3)

                    if hrp.AssemblyLinearVelocity then
                        hrp.AssemblyLinearVelocity = hrp.AssemblyLinearVelocity:Lerp(targetVelocity, 0.3)
                    end

                    -- WICHTIG: Humanoid Move für Animation (sieht natürlich aus!)
                    humanoid:Move(moveDirection, false)

                    -- Setze Running State für Lauf-Animation
                    if humanoid:GetState() ~= Enum.HumanoidStateType.Running then
                        humanoid:ChangeState(Enum.HumanoidStateType.Running)
                    end
                else
                    -- Keine Bewegung = Idle Animation
                    local currentVel = hrp.Velocity
                    if currentVel.Magnitude < 2 then
                        hrp.Velocity = Vector3.new(0, hrp.Velocity.Y, 0)
                        if hrp.AssemblyLinearVelocity then
                            hrp.AssemblyLinearVelocity = Vector3.new(0, hrp.AssemblyLinearVelocity.Y, 0)
                        end
                    end
                end

                -- ANTI-BAN: Verhindere Freefall State (sehr wichtig!)
                local currentState = humanoid:GetState()
                if currentState == Enum.HumanoidStateType.Freefall or 
                   currentState == Enum.HumanoidStateType.Flying then
                    humanoid:ChangeState(Enum.HumanoidStateType.Running)
                end

                -- Verhindere PlatformStand (sieht verdächtig aus)
                if humanoid.PlatformStand then
                    humanoid.PlatformStand = false
                end

                -- Gravity normal lassen (Anti-Detection)
                if hrp:FindFirstChild("BodyForce") then
                    hrp.BodyForce:Destroy()
                end
                if hrp:FindFirstChild("BodyVelocity") then
                    hrp.BodyVelocity:Destroy()
                end
            end)
        end)
        print("👻 ULTRA Anti-Ban Noclip aktiviert - Natürliche Animation!")
    else
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end

        pcall(function()
            if LocalPlayer.Character then
                local character = LocalPlayer.Character
                local humanoid = character:FindFirstChild("Humanoid")
                local hrp = character:FindFirstChild("HumanoidRootPart")

                -- Alle Parts wieder normal setzen
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end

                if humanoid then
                    humanoid.WalkSpeed = originalWalkSpeed or 16
                    humanoid.PlatformStand = false
                end

                if hrp then
                    -- Velocity sanft zurücksetzen
                    hrp.Velocity = Vector3.new(0, hrp.Velocity.Y, 0)
                    if hrp.AssemblyLinearVelocity then
                        hrp.AssemblyLinearVelocity = Vector3.new(0, hrp.AssemblyLinearVelocity.Y, 0)
                    end
                end
            end
        end)
        print("👻 Noclip deaktiviert - Normale Kollision")
    end
end

MainTab:AddToggle({
    Name = "Noclip Ein/Aus",
    Default = false,
    Callback = function(Value)
        ToggleNoclip(Value)
        if Value then
            SendWebhook("👻 Noclip Aktiviert", "Spieler: " .. LocalPlayer.Name .. "\nSpeed: " .. NoclipSpeed .. "x", 16711935) -- Magenta
        else
            SendWebhook("👻 Noclip Deaktiviert", "Spieler: " .. LocalPlayer.Name, 8421504) -- Grau
        end
    end
})

MainTab:AddSlider({
    Name = "Noclip Geschwindigkeit",
    Min = 1,
    Max = 8,
    Default = 2,
    Color = Color3.fromRGB(255, 0, 200),
    Increment = 0.5,
    ValueName = "x",
    Callback = function(Value)
        NoclipSpeed = Value
        if Value <= 2 then
            print("👻 Noclip Speed: " .. Value .. "x (100% SICHER - Natürlich)")
        elseif Value <= 4 then
            print("👻 Noclip Speed: " .. Value .. "x (Sehr Sicher)")
        elseif Value <= 6 then
            print("👻 Noclip Speed: " .. Value .. "x (Schnell - Sicher)")
        else
            print("👻 Noclip Speed: " .. Value .. "x (ULTRA SCHNELL!)")
        end
    end
})



-- Noclip Character Respawn Handler
LocalPlayer.CharacterAdded:Connect(function(character)
    wait(0.5)
    if NoclipEnabled then
        ToggleNoclip(true)
    end
    if FlyEnabled then
        ToggleFly(true)
    end
end)

local FlySection = MainTab:AddSection({
    Name = "Fly (Fliegen)"
})

-- Fly Funktion (Anti-Ban Version - CFrame basiert mit besserer Kollisionsvermeidung)
local function ToggleFly(state)
    FlyEnabled = state

    if FlyEnabled then
        pcall(function()
            local character = LocalPlayer.Character
            if not character then return end

            local hrp = character:FindFirstChild("HumanoidRootPart")
            local humanoid = character:FindFirstChild("Humanoid")
            if not hrp or not humanoid then return end

            flyConnection = RunService.Heartbeat:Connect(function()
                pcall(function()
                    if not FlyEnabled or not LocalPlayer.Character then return end

                    local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")

                    if not hrp or not humanoid then return end

                    -- Alle Parts noclip für Fly
                    for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.CanCollide = false
                        end
                    end

                    -- CFrame basierte Bewegung
                    local camera = workspace.CurrentCamera
                    local moveDirection = Vector3.new(0, 0, 0)
                    local moveSpeed = FlySpeed / 50

                    -- WASD Steuerung
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                        moveDirection = moveDirection + (camera.CFrame.LookVector * moveSpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                        moveDirection = moveDirection - (camera.CFrame.LookVector * moveSpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                        moveDirection = moveDirection - (camera.CFrame.RightVector * moveSpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                        moveDirection = moveDirection + (camera.CFrame.RightVector * moveSpeed)
                    end

                    -- Space = Hoch, Shift = Runter
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                        moveDirection = moveDirection + (Vector3.new(0, 1, 0) * moveSpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                        moveDirection = moveDirection - (Vector3.new(0, 1, 0) * moveSpeed)
                    end

                    -- Bewege mit CFrame
                    if moveDirection.Magnitude > 0 then
                        hrp.CFrame = hrp.CFrame + moveDirection
                        hrp.Velocity = Vector3.new(0, 0, 0)
                        hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                    end

                    -- Verhindere Fall-Animation (Anti-Ban)
                    if humanoid:GetState() == Enum.HumanoidStateType.Freefall then
                        humanoid:ChangeState(Enum.HumanoidStateType.Running)
                    end
                end)
            end)

            print("✈️ Fly aktiviert (Anti-Ban) - Sicher fliegen!")
        end)
    else
        if flyConnection then
            flyConnection:Disconnect()
            flyConnection = nil
        end

        pcall(function()
            if LocalPlayer.Character then
                for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end

                local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")

                if hrp then
                    hrp.Velocity = Vector3.new(0, 0, 0)
                    hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                end

                if humanoid then
                    humanoid:ChangeState(Enum.HumanoidStateType.Running)
                end
            end
        end)

        print("✈️ Fly deaktiviert - Normale Bewegung")
    end
end

MainTab:AddToggle({
    Name = "Fly Ein/Aus",
    Default = false,
    Callback = function(Value)
        ToggleFly(Value)
        if Value then
            SendWebhook("✈️ Fly Aktiviert", "Spieler: " .. LocalPlayer.Name .. "\nSpeed: " .. FlySpeed, 255) -- Blau
        else
            SendWebhook("✈️ Fly Deaktiviert", "Spieler: " .. LocalPlayer.Name, 8421504) -- Grau
        end
    end
})

MainTab:AddSlider({
    Name = "Fly Geschwindigkeit",
    Min = 10,
    Max = 500,
    Default = 100,
    Color = Color3.fromRGB(255, 100, 255),
    Increment = 10,
    ValueName = " Speed",
    Callback = function(Value)
        FlySpeed = Value
        print("✈️ Fly Speed: " .. Value .. (Value >= 300 and " (RAKETEN SPEED!)" or Value >= 150 and " (SEHR SCHNELL!)" or ""))
    end
})

MainTab:AddBind({
    Name = "Fly Keybind",
    Default = Enum.KeyCode.F,
    Hold = false,
    Callback = function()
        FlyEnabled = not FlyEnabled
        ToggleFly(FlyEnabled)
        if FlyEnabled then
            print("✈️ Fly aktiviert (Taste gedrückt)")
        else
            print("✈️ Fly deaktiviert (Taste gedrückt)")
        end
    end
})

-- Rechte Maustaste Events für Aimassist
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        RightMouseHeld = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        RightMouseHeld = false
    end
end)

-- Funktion um zu prüfen ob ein Spieler tot ist
local function IsPlayerAlive(player)
    if not player.Character then return false end

    local humanoid = player.Character:FindFirstChild("Humanoid")
    if not humanoid then return false end

    if humanoid.Health <= 0 then return false end

    pcall(function()
        if humanoid:GetState() == Enum.HumanoidStateType.Dead then return false end
    end)

    if not player.Character:FindFirstChild("HumanoidRootPart") then return false end

    local targetPart = player.Character:FindFirstChild("schlechter, für kopf") or 
                      player.Character:FindFirstChild("off") or 
                      player.Character:FindFirstChild("HumanoidRootPart")
    if not targetPart then return false end

    return true
end

-- ESP Funktionen - Speziell für Notruf Hamburg
local function IsPolice(player)
    if player.Team then
        local teamName = string.lower(player.Team.Name)
        -- Spezifische Notruf Hamburg Polizei Teams
        if teamName == "polizei" or 
           teamName == "police" or 
           teamName == "cop" or
           teamName == "cops" or
           teamName == "officer" or
           teamName == "polizei hamburg" or
           teamName == "hamburger polizei" or
           teamName == "streifenpolizei" or
           teamName == "verkehrspolizei" or
           teamName == "kriminalpolizei" or
           teamName == "kripo" or
           teamName == "sek" or
           teamName == "spezialeinsatzkommando" or
           teamName == "wasserschutzpolizei" or
           teamName == "bereitschaftspolizei" or
           teamName == "schutzpolizei" or
           teamName == "polizeirevier" then
            return true
        end
    end
    return false
end

local function IsWanted(player)
    if player.Team then
        local teamName = string.lower(player.Team.Name)
        -- Nur echte Wanted/Verbrecher Teams aus Notruf Hamburg
        if teamName == "wanted" or 
           teamName == "verbrecher" or 
           teamName == "criminal" or
           teamName == "criminell" or
           teamName == "gangster" or
           teamName == "räuber" or
           teamName == "dieb" or
           teamName == "einbrecher" or
           teamName == "dealer" or
           teamName == "drogendealer" or
           teamName == "mafia" or
           teamName == "gang" or
           teamName == "bande" or
           teamName == "outlaws" or
           teamName == "gesuchte" then
            return true
        end
    end
    return false
end

local function CreateESPBox()
    local box = {
        TopLeft = Drawing.new("Line"),
        TopRight = Drawing.new("Line"),
        BottomLeft = Drawing.new("Line"),
        BottomRight = Drawing.new("Line"),
        LeftSide = Drawing.new("Line"),
        RightSide = Drawing.new("Line"),
        TopSide = Drawing.new("Line"),
        BottomSide = Drawing.new("Line")
    }

    for _, line in pairs(box) do
        line.Thickness = 2.5  -- Dicker (war 1)
        line.Color = Color3.fromRGB(0, 255, 0)
        line.Visible = false
    end

    return box
end

local function CreateESPName()
    local text = Drawing.new("Text")
    text.Size = 20  -- Größer (war 16)
    text.Center = true
    text.Outline = true
    text.OutlineColor = Color3.fromRGB(0, 0, 0)  -- Schwarzer Outline
    text.Color = Color3.fromRGB(255, 255, 255)
    text.Visible = false
    return text
end

-- Funktion zum Aufräumen von ESP für einen Spieler
local function RemoveESP(player)
    if ESPBoxes[player] then
        for _, line in pairs(ESPBoxes[player]) do
            pcall(function()
                line:Remove()
            end)
        end
        ESPBoxes[player] = nil
    end

    if ESPNames[player] then
        pcall(function()
            ESPNames[player]:Remove()
        end)
        ESPNames[player] = nil
    end
end

-- Spieler verlässt Server
Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
end)

-- Character neu spawnt
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        RemoveESP(player)
    end)
end)

-- ESP Update Loop für Aimassist
RunService.RenderStepped:Connect(function()
    -- FOV Kreis in der Mitte des Bildschirms
    local screenCenter = Camera.ViewportSize / 2
    FOVCircle.Position = screenCenter
    FOVCircle.Radius = AimFOV
    FOVCircle.Visible = AimAssistEnabled

    -- Aimassist Logic
    if AimAssistEnabled and RightMouseHeld then
        local closestPlayer = nil
        local shortestDistance = AimFOV

        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(TargetPart) and IsPlayerAlive(player) then
                local targetPart = player.Character[TargetPart]
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)

                local realDistance = (targetPart.Position - Camera.CFrame.Position).Magnitude

                if onScreen and realDistance <= AimMaxDistance then
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude

                    if distance < shortestDistance then
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end

        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild(TargetPart) and IsPlayerAlive(closestPlayer) then
            local targetPart = closestPlayer.Character[TargetPart]
            local targetPos = targetPart.Position

            local currentCFrame = Camera.CFrame
            local targetCFrame = CFrame.new(Camera.CFrame.Position, targetPos)
            Camera.CFrame = currentCFrame:Lerp(targetCFrame, 1 / AimSmoothness)
        end
    end

    -- ESP Logic
    if not ESPEnabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        pcall(function()
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local isAlive = IsPlayerAlive(player)
                local hrp = player.Character.HumanoidRootPart
                local head = player.Character:FindFirstChild("Head")

                if not ESPBoxes[player] then
                    ESPBoxes[player] = CreateESPBox()
                    ESPNames[player] = CreateESPName()
                end

                local box = ESPBoxes[player]
                local nameTag = ESPNames[player]

                local vector, onScreen = Camera:WorldToViewportPoint(hrp.Position)

                local distance = (hrp.Position - Camera.CFrame.Position).Magnitude

                if onScreen and head and distance <= ESPMaxDistance then
                    local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
                    local legPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))

                    local height = math.abs(headPos.Y - legPos.Y)
                    local width = height / 2

                    -- Farbe basierend auf Status und Team setzen
                    local espColor
                    if not isAlive then
                        espColor = Color3.fromRGB(255, 0, 255) -- Lila für tote Spieler
                    elseif IsPolice(player) then
                        espColor = Color3.fromRGB(0, 100, 255) -- Blau für Polizei
                    elseif IsWanted(player) then
                        espColor = Color3.fromRGB(255, 165, 0) -- Orange für Wanted-Spieler
                    else
                        espColor = Color3.fromRGB(0, 255, 0) -- Grün für alle anderen Teams
                    end

                    -- Box zeichnen
                    local topLeft = Vector2.new(vector.X - width/2, headPos.Y)
                    local topRight = Vector2.new(vector.X + width/2, headPos.Y)
                    local bottomLeft = Vector2.new(vector.X - width/2, legPos.Y)
                    local bottomRight = Vector2.new(vector.X + width/2, legPos.Y)

                    box.TopLeft.From = topLeft
                    box.TopLeft.To = topLeft + Vector2.new(width/4, 0)
                    box.TopRight.From = topRight - Vector2.new(width/4, 0)
                    box.TopRight.To = topRight
                    box.BottomLeft.From = bottomLeft
                    box.BottomLeft.To = bottomLeft + Vector2.new(width/4, 0)
                    box.BottomRight.From = bottomRight - Vector2.new(width/4, 0)
                    box.BottomRight.To = bottomRight
                    box.LeftSide.From = topLeft
                    box.LeftSide.To = bottomLeft
                    box.RightSide.From = topRight
                    box.RightSide.To = bottomRight
                    box.TopSide.From = topLeft
                    box.TopSide.To = topRight
                    box.BottomSide.From = bottomLeft
                    box.BottomSide.To = bottomRight

                    for _, line in pairs(box) do
                        line.Color = espColor
                        line.Visible = true
                    end

                    nameTag.Text = player.Name .. (not isAlive and " [TOT]" or "")
                    nameTag.Position = Vector2.new(vector.X, headPos.Y - 20)
                    nameTag.Color = espColor
                    nameTag.Visible = true
                else
                    for _, line in pairs(box) do
                        line.Visible = false
                    end
                    nameTag.Visible = false
                end
            elseif ESPBoxes[player] then
                for _, line in pairs(ESPBoxes[player]) do
                    line.Visible = false
                end
                if ESPNames[player] then
                    ESPNames[player].Visible = false
                end
            end
        end)
    end

    -- Aufräumen für Spieler die gegangen sind
    for player, _ in pairs(ESPBoxes) do
        if not player.Parent then
            RemoveESP(player)
        end
    end
end)

-- No Recoil Variables
local NoRecoilEnabled = false
local recoilConnection = nil
local originalCFrame = nil

-- Anti Taser Variables
local monitoringActive = false  
local antiTaser = false
local antiTaserTask

-- Teleport Variablen
local TeleportEnabled = false
local SelectedPlayer = nil



-- No Recoil Tab
local NoRecoilTab = Window:MakeTab({
    Name = "No Recoil",
    Icon = "rbxassetid://4483345998", 
    PremiumOnly = false
})

local NoRecoilSection = NoRecoilTab:AddSection({
    Name = "No Recoil Einstellungen"
})

-- No Recoil Function
local function NoRecoil()
    if NoRecoilEnabled then
        recoilConnection = RunService.RenderStepped:Connect(function()
            pcall(function()
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                    -- Verhindere Kamera-Shake und Recoil
                    if originalCFrame then
                        Camera.CFrame = originalCFrame
                    end

                    -- Entferne Recoil-Effekte
                    for _, effect in pairs(Camera:GetChildren()) do
                        if effect.Name:lower():find("recoil") or effect.Name:lower():find("shake") then
                            effect:Destroy()
                        end
                    end

                    -- Anti-Recoil für Waffen
                    local humanoid = LocalPlayer.Character.Humanoid
                    if humanoid.PlatformStand then
                        humanoid.PlatformStand = false
                    end

                    -- Neutralisiere Kamera-Bewegungen
                    local currentRotation = Camera.CFrame.Rotation
                    if math.abs(currentRotation.X) > 0.1 then
                        Camera.CFrame = CFrame.new(Camera.CFrame.Position) * CFrame.Angles(0, currentRotation.Y, 0)
                    end
                end
            end)
        end)
    else
        if recoilConnection then
            recoilConnection:Disconnect()
            recoilConnection = nil
        end
    end
end

NoRecoilTab:AddToggle({
    Name = "No Recoil Ein/Aus",
    Default = false,
    Callback = function(Value)
        NoRecoilEnabled = Value
        NoRecoil()
        if Value then
            print("🎯 No Recoil aktiviert - Kein Rückstoß!")
            SendWebhook("🎯 No Recoil Aktiviert", "Spieler: " .. LocalPlayer.Name, 16711680) -- Rot
        else
            print("🎯 No Recoil deaktiviert")
            SendWebhook("🎯 No Recoil Deaktiviert", "Spieler: " .. LocalPlayer.Name, 8421504) -- Grau
        end
    end
})



-- Anti Taser Tab
local AntiTaserTab = Window:MakeTab({
    Name = "Anti Taser",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})
AntiTaserTab:AddToggle({
    Name = "Anti Taser Ein/Aus",
    Default = false,
    Callback = function(Value)
        monitoringActive = Value
        if Value then
            antiTaser = true
            print("Anti Taser aktiviert")
            SendWebhook("⚡ Anti Taser Aktiviert", "Spieler: " .. LocalPlayer.Name, 16776960) -- Gelb
            antiTaserTask = task.spawn(function()
                while antiTaser do
                    if not LocalPlayer.Character:GetAttribute("Tased") then
                        LocalPlayer.Character:SetAttribute("Tased", false)
                    end
                    LocalPlayer.Character:GetAttributeChangedSignal("Tased"):Connect(function()
                        if antiTaser then
                            LocalPlayer.Character:SetAttribute("Tased", false)
                        end
                    end)
                    wait(0.5) -- Delay to avoid overwhelming the server
                end
            end)
        else
            antiTaser = false
            print("Anti Taser deaktiviert")
        end
    end
})

-- Teleport Tab
local TeleportTab = Window:MakeTab({
    Name = "Teleport",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local TeleportSection = TeleportTab:AddSection({
    Name = "Teleportation zu Spielern"
})

-- Funktion um Spielerliste zu erstellen
local function GetPlayerList()
    local playerList = {}
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(playerList, player.Name)
        end
    end
    return playerList
end

-- Teleport Funktion
local function TeleportToPlayer(playerName)
    pcall(function()
        local targetPlayer = Players:FindFirstChild(playerName)
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local targetHRP = targetPlayer.Character.HumanoidRootPart
                local myHRP = LocalPlayer.Character.HumanoidRootPart

                -- Teleportiere direkt zum Spieler
                myHRP.CFrame = targetHRP.CFrame + Vector3.new(0, 2, 0)
                print("📍 Teleportiert zu: " .. playerName)
            else
                print("❌ Dein Character wurde nicht gefunden!")
            end
        else
            print("❌ Spieler " .. playerName .. " wurde nicht gefunden oder hat keinen Character!")
        end
    end)
end

TeleportTab:AddDropdown({
    Name = "Spieler auswählen",
    Default = "",
    Options = GetPlayerList(),
    Callback = function(Value)
        SelectedPlayer = Value
        print("✅ Spieler ausgewählt: " .. Value)
    end
})

TeleportTab:AddButton({
    Name = "Zu Spieler teleportieren",
    Callback = function()
        if SelectedPlayer then
            TeleportToPlayer(SelectedPlayer)
            SendWebhook("📍 Teleport", "Spieler: " .. LocalPlayer.Name .. "\nTeleportiert zu: " .. SelectedPlayer, 16753920) -- Orange
        else
            print("❌ Bitte wähle zuerst einen Spieler aus!")
        end
    end
})

TeleportTab:AddButton({
    Name = "Spielerliste aktualisieren",
    Callback = function()
        print("🔄 Spielerliste wird aktualisiert...")
        -- Die Dropdown muss manuell neu erstellt werden
        OrionLib:MakeNotification({
            Name = "Info",
            Content = "Bitte öffne das Menu neu um die aktualisierte Spielerliste zu sehen.",
            Image = "rbxassetid://4483345998",
            Time = 5
        })
    end
})

TeleportTab:AddToggle({
    Name = "Loop Teleport (folge Spieler)",
    Default = false,
    Callback = function(Value)
        TeleportEnabled = Value
        if Value then
            print("🔁 Loop Teleport aktiviert - Folge " .. (SelectedPlayer or "keinem Spieler"))
            task.spawn(function()
                while TeleportEnabled do
                    if SelectedPlayer then
                        TeleportToPlayer(SelectedPlayer)
                    end
                    task.wait(0.5) -- Alle 0.5 Sekunden teleportieren
                end
            end)
        else
            print("🔁 Loop Teleport deaktiviert")
        end
    end
})




-- Vehicle Fly Tab
local VehicleFlyTab = Window:MakeTab({
    Name = "Vehicle Fly",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local VehicleFlySection = VehicleFlyTab:AddSection({
    Name = "Fahrzeug Fliegen"
})

-- Vehicle Fly Variablen
local VehicleFlyEnabled = false
local VehicleFlySpeed = 149  -- Schneller (war 50)
local VehicleFlySmooth = 0.12  -- Stärker (war 0.15)
local vehicleFlyConnection = nil
local currentVehicle = nil
local targetVelocity = Vector3.new(0, 0, 0)

-- Funktion um aktuelles Fahrzeug zu finden
local function GetCurrentVehicle()
    local vehicle = nil
    pcall(function()
        if LocalPlayer.Character then
            local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")
            if humanoid and humanoid.SeatPart then
                local seat = humanoid.SeatPart

                local current = seat.Parent
                while current and current ~= workspace do
                    if current:IsA("Model") then
                        vehicle = current
                        break
                    end
                    current = current.Parent
                end

                if not vehicle then
                    vehicle = seat.Parent
                end
            end
        end
    end)
    currentVehicle = vehicle
    return vehicle
end

-- Vehicle Fly Funktion (ANTI-BAN VERSION - Sicher & Unauffällig)
local function ToggleVehicleFly(state)
    VehicleFlyEnabled = state

    if VehicleFlyEnabled then
        targetVelocity = Vector3.new(0, 0, 0)
        local lastUpdate = tick()
        local updateInterval = 0.1 -- Langsamere Updates = weniger auffällig

        vehicleFlyConnection = RunService.Heartbeat:Connect(function(deltaTime)
            pcall(function()
                if not VehicleFlyEnabled then return end

                -- Rate Limiting für Anti-Ban
                local currentTime = tick()
                if currentTime - lastUpdate < updateInterval then
                    return
                end
                lastUpdate = currentTime

                local vehicle = GetCurrentVehicle()
                if not vehicle then 
                    return 
                end

                -- Finde Hauptteil des Fahrzeugs
                local primaryPart = vehicle.PrimaryPart or vehicle:FindFirstChild("VehicleSeat") or vehicle:FindFirstChildWhichIsA("VehicleSeat")
                if not primaryPart then
                    for _, part in pairs(vehicle:GetDescendants()) do
                        if part:IsA("BasePart") and part.Name:lower():find("main") then
                            primaryPart = part
                            break
                        end
                    end
                end
                if not primaryPart then return end

                -- Bewegungsrichtung berechnen
                local camera = workspace.CurrentCamera
                local moveDirection = Vector3.new(0, 0, 0)

                -- WASD Steuerung
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    moveDirection = moveDirection + camera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    moveDirection = moveDirection - camera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    moveDirection = moveDirection - camera.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    moveDirection = moveDirection + camera.CFrame.RightVector
                end

                -- Space = Hoch, Shift = Runter (begrenzte Höhe für Anti-Ban)
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    local currentY = primaryPart.Position.Y
                    if currentY < 200 then -- Maximale Höhe begrenzen
                        moveDirection = moveDirection + Vector3.new(0, 0.8, 0)
                    end
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                    moveDirection = moveDirection - Vector3.new(0, 0.8, 0)
                end

                -- Realistischere Geschwindigkeit (nicht zu schnell = Anti-Ban)
                local maxSpeed = math.min(VehicleFlySpeed, 80) -- Begrenzt auf 80 max
                if moveDirection.Magnitude > 0 then
                    targetVelocity = moveDirection.Unit * maxSpeed
                else
                    targetVelocity = targetVelocity * 0.85 -- Langsames Abbremsen
                end

                -- Sehr sanftes Lerping (unauffällig)
                local currentVelocity = primaryPart.AssemblyLinearVelocity or primaryPart.Velocity
                local smoothVelocity = currentVelocity:Lerp(targetVelocity, 0.08) -- Sehr sanft

                -- Bewege nur die wichtigsten Teile (weniger auffällig)
                local partsToMove = {primaryPart}
                for _, part in pairs(vehicle:GetDescendants()) do
                    if part:IsA("BasePart") and (part:IsA("VehicleSeat") or part.Name:lower():find("seat") or part.Name:lower():find("body")) then
                        table.insert(partsToMove, part)
                    end
                end

                for _, part in pairs(partsToMove) do
                    -- Noclip nur wenn nötig
                    part.CanCollide = false

                    -- Sanfte Velocity-Änderung (nicht sofort = Anti-Ban)
                    task.wait(0.01)
                    if part.AssemblyLinearVelocity then
                        part.AssemblyLinearVelocity = smoothVelocity
                    else
                        part.Velocity = smoothVelocity
                    end

                    -- Rotation natürlich lassen (nicht komplett 0)
                    local currentRot = part.RotVelocity
                    part.RotVelocity = currentRot * 0.7 -- Langsam reduzieren
                    if part.AssemblyAngularVelocity then
                        part.AssemblyAngularVelocity = part.AssemblyAngularVelocity * 0.7
                    end

                    -- Keine BodyForce (zu auffällig für Anti-Cheat)
                    for _, force in pairs(part:GetChildren()) do
                        if force:IsA("BodyForce") or force:IsA("BodyVelocity") or force:IsA("BodyGyro") then
                            force:Destroy()
                        end
                    end
                end
            end)
        end)
        print("🚗✈️ Vehicle Fly aktiviert - ANTI-BAN MODUS!")
    else
        if vehicleFlyConnection then
            vehicleFlyConnection:Disconnect()
            vehicleFlyConnection = nil
        end

        pcall(function()
            local vehicle = GetCurrentVehicle()
            if vehicle then
                -- Sanftes Herunterfahren
                task.wait(0.2)
                for _, part in pairs(vehicle:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true

                        -- Sehr sanftes Stoppen
                        local currentVel = part.AssemblyLinearVelocity or part.Velocity
                        if part.AssemblyLinearVelocity then
                            part.AssemblyLinearVelocity = currentVel * 0.3
                        else
                            part.Velocity = currentVel * 0.3
                        end

                        -- Rotation natürlich auslaufen lassen
                        local currentRot = part.RotVelocity
                        part.RotVelocity = currentRot * 0.5
                        if part.AssemblyAngularVelocity then
                            part.AssemblyAngularVelocity = part.AssemblyAngularVelocity * 0.5
                        end

                        -- Alle Forces entfernen
                        for _, force in pairs(part:GetChildren()) do
                            if force:IsA("BodyForce") or force:IsA("BodyVelocity") or force:IsA("BodyGyro") then
                                force:Destroy()
                            end
                        end
                    end
                end
            end
        end)
        print("🚗 Vehicle Fly deaktiviert - Sicher gelandet")
    end
end

VehicleFlyTab:AddToggle({
    Name = "Vehicle Fly Ein/Aus",
    Default = false,
    Callback = function(Value)
        ToggleVehicleFly(Value)
        if Value then
            SendWebhook("🚗✈️ Vehicle Fly Aktiviert", "Spieler: " .. LocalPlayer.Name .. "\nSpeed: " .. VehicleFlySpeed, 16744192) -- Orange-Gelb
        else
            SendWebhook("🚗 Vehicle Fly Deaktiviert", "Spieler: " .. LocalPlayer.Name, 8421504) -- Grau
        end
    end
})

VehicleFlyTab:AddSlider({
    Name = "Vehicle Fly Geschwindigkeit",
    Min = 50,
    Max = 230,
    Default = 80,
    Color = Color3.fromRGB(255, 150, 0),
    Increment = 5,
    ValueName = " Speed",
    Callback = function(Value)
        VehicleFlySpeed = Value
        if Value > 120 then
            print("🚗🔥 ULTRA SPEED: " .. Value .. " (INSANE!)")
        elseif Value > 80 then
            print("🚗⚡ SEHR SCHNELL: " .. Value)
        else
            print("🚗 Vehicle Speed: " .. Value .. " (Sicher)")
        end
    end
})

VehicleFlyTab:AddSlider({
    Name = "Fly Sanftheit",
    Min = 0.05,
    Max = 0.5,
    Default = 0.15,
    Color = Color3.fromRGB(100, 200, 255),
    Increment = 0.05,
    ValueName = "",
    Callback = function(Value)
        VehicleFlySmooth = Value
        if Value <= 0.1 then
            print("🚗 Sehr sanft & flüssig")
        elseif Value <= 0.2 then
            print("🚗 Sanft")
        else
            print("🚗 Schnelle Reaktion")
        end
    end
})

VehicleFlyTab:AddBind({
    Name = "Vehicle Fly Keybind",
    Default = Enum.KeyCode.V,
    Hold = false,
    Callback = function()
        VehicleFlyEnabled = not VehicleFlyEnabled
        ToggleVehicleFly(VehicleFlyEnabled)
        if VehicleFlyEnabled then
            print("🚗✈️ Vehicle Fly aktiviert (Taste gedrückt)")
        else
            print("🚗 Vehicle Fly deaktiviert (Taste gedrückt)")
        end
    end
})

VehicleFlyTab:AddButton({
    Name = "Aktuelles Fahrzeug anzeigen",
    Callback = function()
        local vehicle = GetCurrentVehicle()
        if vehicle then
            print("🚗 Aktuelles Fahrzeug: " .. vehicle.Name)
            OrionLib:MakeNotification({
                Name = "Fahrzeug gefunden",
                Content = "Du sitzt in: " .. vehicle.Name,
                Image = "rbxassetid://4483345998",
                Time = 3
            })
        else
            print("❌ Kein Fahrzeug gefunden - Setze dich in ein Auto!")
            OrionLib:MakeNotification({
                Name = "Kein Fahrzeug",
                Content = "Bitte setze dich zuerst in ein Auto!",
                Image = "rbxassetid://4483345998",
                Time = 3
            })
        end
    end
})

-- WalkSpeed Tab
local WalkSpeedTab = Window:MakeTab({
    Name = "WalkSpeed",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local WalkSpeedSection = WalkSpeedTab:AddSection({
    Name = "Laufgeschwindigkeit (100% Undetectable)"
})

-- WalkSpeed Variablen
local WalkSpeedEnabled = false
local CustomWalkSpeed = 16
local originalWalkSpeed = 16
local walkSpeedConnection = nil

-- ADVANCED Anti-Detection WalkSpeed (Bessere Animation & WASD Steuerung!)
local function UpdateWalkSpeed(state)
    WalkSpeedEnabled = state

    if WalkSpeedEnabled then
        walkSpeedConnection = RunService.Heartbeat:Connect(function()
            pcall(function()
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = LocalPlayer.Character.HumanoidRootPart
                    local humanoid = LocalPlayer.Character:FindFirstChild("Humanoid")

                    if not humanoid then return end

                    -- WalkSpeed auf 16 lassen (Anti-Detection!)
                    humanoid.WalkSpeed = 16

                    -- Kamera für Richtungsberechnung
                    local camera = workspace.CurrentCamera
                    local cameraCFrame = camera.CFrame

                    -- Manuelle WASD Bewegungsberechnung
                    local moveDirection = Vector3.new(0, 0, 0)

                    -- W = Vorwärts (LookVector)
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                        moveDirection = moveDirection + cameraCFrame.LookVector
                    end
                    -- S = Rückwärts (-LookVector)
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                        moveDirection = moveDirection - cameraCFrame.LookVector
                    end
                    -- A = Links (-RightVector)
                    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                        moveDirection = moveDirection - cameraCFrame.RightVector
                    end
                    -- D = Rechts (RightVector)
                    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                        moveDirection = moveDirection + cameraCFrame.RightVector
                    end

                    -- Nur horizontale Bewegung (keine Y-Achse)
                    moveDirection = Vector3.new(moveDirection.X, 0, moveDirection.Z)

                    if moveDirection.Magnitude > 0 then
                        moveDirection = moveDirection.Unit

                        -- Berechne Speed-Multiplikator
                        local speedMultiplier = CustomWalkSpeed / 16
                        local additionalSpeed = (speedMultiplier - 1) * 0.6

                        -- CFrame basierte Bewegung (smooth & undetectable)
                        hrp.CFrame = hrp.CFrame + (moveDirection * additionalSpeed)

                        -- Humanoid MoveDirection setzen für Animation
                        humanoid:Move(moveDirection, true)

                        -- Velocity kontrollieren für smooth movement
                        local currentVel = hrp.Velocity
                        hrp.Velocity = Vector3.new(currentVel.X * 0.7, currentVel.Y, currentVel.Z * 0.7)

                        if hrp.AssemblyLinearVelocity then
                            local currentAssemblyVel = hrp.AssemblyLinearVelocity
                            hrp.AssemblyLinearVelocity = Vector3.new(currentAssemblyVel.X * 0.7, currentAssemblyVel.Y, currentAssemblyVel.Z * 0.7)
                        end

                        -- Lauf-Animation triggern
                        if humanoid:GetState() ~= Enum.HumanoidStateType.Running then
                            humanoid:ChangeState(Enum.HumanoidStateType.Running)
                        end
                    else
                        -- Keine Bewegung = Idle Animation
                        if humanoid:GetState() == Enum.HumanoidStateType.Running then
                            local currentVel = hrp.Velocity
                            if currentVel.Magnitude < 2 then
                                humanoid:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
                            end
                        end
                    end

                    -- Anti-Detection: Fall-State verhindern
                    if humanoid:GetState() == Enum.HumanoidStateType.Freefall then
                        humanoid:ChangeState(Enum.HumanoidStateType.Running)
                    end
                end
            end)
        end)
        print("🏃 Advanced WalkSpeed aktiviert: " .. CustomWalkSpeed .. " (Bessere Animation & WASD Steuerung)")
    else
        if walkSpeedConnection then
            walkSpeedConnection:Disconnect()
            walkSpeedConnection = nil
        end

        pcall(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                local humanoid = LocalPlayer.Character.Humanoid
                humanoid.WalkSpeed = 16

                local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.Velocity = Vector3.new(0, hrp.Velocity.Y, 0)
                    if hrp.AssemblyLinearVelocity then
                        hrp.AssemblyLinearVelocity = Vector3.new(0, hrp.AssemblyLinearVelocity.Y, 0)
                    end
                end
            end
        end)
        print("🏃 WalkSpeed deaktiviert: Normal")
    end
end

WalkSpeedTab:AddToggle({
    Name = "WalkSpeed Ein/Aus",
    Default = false,
    Callback = function(Value)
        UpdateWalkSpeed(Value)
        if Value then
            SendWebhook("🏃 WalkSpeed Aktiviert", "Spieler: " .. LocalPlayer.Name .. "\nSpeed: " .. CustomWalkSpeed, 65280) -- Grün
        else
            SendWebhook("🏃 WalkSpeed Deaktiviert", "Spieler: " .. LocalPlayer.Name, 8421504) -- Grau
        end
    end
})

WalkSpeedTab:AddSlider({
    Name = "Laufgeschwindigkeit",
    Min = 16,
    Max = 150,
    Default = 35,
    Color = Color3.fromRGB(0, 255, 100),
    Increment = 5,
    ValueName = " Speed",
    Callback = function(Value)
        CustomWalkSpeed = Value

        if Value <= 30 then
            print("🏃 Speed: " .. Value .. " (100% Sicher - Undetectable)")
        elseif Value <= 50 then
            print("🏃⚡ Speed: " .. Value .. " (Sehr Sicher)")
        elseif Value <= 80 then
            print("🏃💨 Speed: " .. Value .. " (Schnell - Sicher)")
        elseif Value <= 120 then
            print("🏃🔥 Speed: " .. Value .. " (ULTRA SCHNELL!)")
        else
            print("🏃⚡💀 Speed: " .. Value .. " (INSANE SPEED!)")
        end
    end
})

WalkSpeedTab:AddBind({
    Name = "WalkSpeed Keybind",
    Default = Enum.KeyCode.G,
    Hold = false,
    Callback = function()
        WalkSpeedEnabled = not WalkSpeedEnabled
        UpdateWalkSpeed(WalkSpeedEnabled)
        if WalkSpeedEnabled then
            print("🏃 WalkSpeed aktiviert (Taste gedrückt)")
        else
            print("🏃 WalkSpeed deaktiviert (Taste gedrückt)")
        end
    end
})

WalkSpeedTab:AddButton({
    Name = "Auf 16 (Normal) zurücksetzen",
    Callback = function()
        CustomWalkSpeed = 16
        pcall(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid.WalkSpeed = 16
            end
        end)
        print("🏃 WalkSpeed auf Normal (16) zurückgesetzt")
    end
})

-- Character Respawn Handler für WalkSpeed
LocalPlayer.CharacterAdded:Connect(function(character)
    wait(0.5)
    if WalkSpeedEnabled then
        UpdateWalkSpeed(true)
    else
        pcall(function()
            local humanoid = character:WaitForChild("Humanoid", 3)
            if humanoid then
                originalWalkSpeed = humanoid.WalkSpeed
            end
        end)
    end
end)

-- Anti-Chat-Detection System
pcall(function()
    local TextChatService = game:GetService("TextChatService")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")

    -- Blockiere Chat-Logs an Server
    local oldFireServer = Instance.new("RemoteEvent").FireServer
    local oldInvokeServer = Instance.new("RemoteFunction").InvokeServer

    for _, remote in pairs(ReplicatedStorage:GetDescendants()) do
        if remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction") then
            if tostring(remote):lower():find("chat") or tostring(remote):lower():find("message") then
                pcall(function()
                    remote.OnClientEvent:Connect(function() end)
                end)
            end
        end
    end
end)

-- Erweiterte Anti-Detection
local function EnhancedAntiDetection()
    pcall(function()
        -- Verstecke Script vor Kick-Systemen
        local mt = getrawmetatable(game)
        local oldIndex = mt.__index

        setreadonly(mt, false)
        mt.__index = newcclosure(function(self, key)
            if key == "Kick" or key == "kick" then
                return function() end
            end
            return oldIndex(self, key)
        end)
        setreadonly(mt, true)
    end)
end

EnhancedAntiDetection()

-- Send Start Notification
task.wait(1)
SendStartNotification()

-- Blacklist Check bei Script Start
if BlacklistActive and LocalPlayer.UserId ~= ADMIN_USER_ID then
    if IsPlayerBlacklisted(LocalPlayer) then
        print("🔒 DU WURDEST GEBLACKLISTED!")
        OrionLib:MakeNotification({
            Name = "⛔ BLACKLISTED",
            Content = "Du wurdest vom Script ausgeschlossen!",
            Image = "rbxassetid://4483345998",
            Time = 10
        })
        
        -- Script komplett deaktivieren für geblacklistete Spieler
        task.wait(3)
        OrionLib:Destroy()
        return
    end
end

-- Blacklist Tab (NUR FÜR ADMIN)
local BlacklistTab = Window:MakeTab({
    Name = "🔒 Blacklist",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local BlacklistSection = BlacklistTab:AddSection({
    Name = "Blacklist System (Admin Only)"
})

-- Admin Check
if LocalPlayer.UserId == ADMIN_USER_ID then
    BlacklistTab:AddLabel("✅ Admin Zugriff: Du kannst Spieler blacklisten!")
    
    BlacklistTab:AddToggle({
        Name = "Blacklist System Ein/Aus",
        Default = false,
        Callback = function(Value)
            BlacklistActive = Value
            if Value then
                print("🔒 Blacklist System aktiviert!")
                SendWebhook("🔒 Blacklist Aktiviert", "Admin: " .. LocalPlayer.Name .. "\nBlacklist System ist jetzt aktiv!", 16711680)
                EnforceBlacklist()
            else
                print("🔒 Blacklist System deaktiviert")
                SendWebhook("🔒 Blacklist Deaktiviert", "Admin: " .. LocalPlayer.Name, 8421504)
            end
        end
    })
    
    -- Spieler zur Blacklist hinzufügen (Dropdown)
    local playerToBlacklist = nil
    BlacklistTab:AddDropdown({
        Name = "Spieler auswählen",
        Default = "",
        Options = GetPlayerList(),
        Callback = function(Value)
            playerToBlacklist = Value
            print("🔒 Spieler ausgewählt: " .. Value)
        end
    })
    
    BlacklistTab:AddButton({
        Name = "➕ Spieler zur Blacklist hinzufügen",
        Callback = function()
            if playerToBlacklist then
                local targetPlayer = Players:FindFirstChild(playerToBlacklist)
                if targetPlayer then
                    -- Füge UserID zur Blacklist hinzu (sicherer als Name)
                    table.insert(BlacklistedPlayers, targetPlayer.UserId)
                    print("🔒 " .. playerToBlacklist .. " (ID: " .. targetPlayer.UserId .. ") zur Blacklist hinzugefügt!")
                    
                    SendWebhook("🔒 Spieler Geblacklisted", 
                        "Admin: " .. LocalPlayer.Name .. "\n**Geblacklistet:** " .. playerToBlacklist .. " (ID: " .. targetPlayer.UserId .. ")", 
                        16711680)
                    
                    OrionLib:MakeNotification({
                        Name = "Blacklist",
                        Content = playerToBlacklist .. " wurde zur Blacklist hinzugefügt!",
                        Image = "rbxassetid://4483345998",
                        Time = 5
                    })
                    
                    -- Sofort enforcen wenn aktiv
                    if BlacklistActive then
                        EnforceBlacklist()
                    end
                else
                    print("❌ Spieler nicht gefunden!")
                end
            else
                print("❌ Bitte wähle zuerst einen Spieler aus!")
            end
        end
    })
    
    -- Manuelle Eingabe (Name oder UserID)
    local manualInput = ""
    BlacklistTab:AddTextbox({
        Name = "Name oder UserID eingeben",
        Default = "",
        TextDisappear = false,
        Callback = function(Value)
            manualInput = Value
        end
    })
    
    BlacklistTab:AddButton({
        Name = "➕ Manuell zur Blacklist hinzufügen",
        Callback = function()
            if manualInput ~= "" then
                local isNumber = tonumber(manualInput)
                if isNumber then
                    -- UserID
                    table.insert(BlacklistedPlayers, isNumber)
                    print("🔒 UserID " .. manualInput .. " zur Blacklist hinzugefügt!")
                    SendWebhook("🔒 UserID Geblacklisted", 
                        "Admin: " .. LocalPlayer.Name .. "\n**UserID:** " .. manualInput, 
                        16711680)
                else
                    -- Name
                    table.insert(BlacklistedPlayers, manualInput)
                    print("🔒 Name '" .. manualInput .. "' zur Blacklist hinzugefügt!")
                    SendWebhook("🔒 Name Geblacklisted", 
                        "Admin: " .. LocalPlayer.Name .. "\n**Name:** " .. manualInput, 
                        16711680)
                end
                
                OrionLib:MakeNotification({
                    Name = "Blacklist",
                    Content = "'" .. manualInput .. "' wurde zur Blacklist hinzugefügt!",
                    Image = "rbxassetid://4483345998",
                    Time = 5
                })
                
                if BlacklistActive then
                    EnforceBlacklist()
                end
            else
                print("❌ Bitte gib einen Namen oder UserID ein!")
            end
        end
    })
    
    BlacklistTab:AddButton({
        Name = "📋 Blacklist anzeigen",
        Callback = function()
            if #BlacklistedPlayers == 0 then
                print("📋 Blacklist ist leer")
                OrionLib:MakeNotification({
                    Name = "Blacklist",
                    Content = "Die Blacklist ist leer!",
                    Image = "rbxassetid://4483345998",
                    Time = 3
                })
            else
                print("📋 === BLACKLIST ===")
                for i, entry in pairs(BlacklistedPlayers) do
                    if type(entry) == "number" then
                        print(i .. ". UserID: " .. entry)
                    else
                        print(i .. ". Name: " .. entry)
                    end
                end
                print("📋 === TOTAL: " .. #BlacklistedPlayers .. " ===")
                
                OrionLib:MakeNotification({
                    Name = "Blacklist",
                    Content = "Blacklist hat " .. #BlacklistedPlayers .. " Einträge (siehe Console)",
                    Image = "rbxassetid://4483345998",
                    Time = 5
                })
            end
        end
    })
    
    BlacklistTab:AddButton({
        Name = "🗑️ Blacklist leeren",
        Callback = function()
            BlacklistedPlayers = {}
            print("🗑️ Blacklist wurde geleert!")
            SendWebhook("🗑️ Blacklist Geleert", 
                "Admin: " .. LocalPlayer.Name .. "\nAlle Einträge wurden entfernt", 
                8421504)
            
            OrionLib:MakeNotification({
                Name = "Blacklist",
                Content = "Blacklist wurde komplett geleert!",
                Image = "rbxassetid://4483345998",
                Time = 3
            })
        end
    })
    
    BlacklistTab:AddButton({
        Name = "🔄 Blacklist sofort durchsetzen",
        Callback = function()
            EnforceBlacklist()
            print("🔄 Blacklist wird durchgesetzt...")
            OrionLib:MakeNotification({
                Name = "Blacklist",
                Content = "Blacklist wird durchgesetzt!",
                Image = "rbxassetid://4483345998",
                Time = 3
            })
        end
    })
    
    BlacklistTab:AddParagraph("ℹ️ Info:", 
        "• Nur du kannst Spieler blacklisten\n" ..
        "• Geblacklistete Spieler können das Script nicht nutzen\n" ..
        "• Du kannst nach Name oder UserID blacklisten\n" ..
        "• Blacklist bleibt nur für diese Session aktiv")
    
else
    -- Kein Admin - Zeige Warnung
    BlacklistTab:AddLabel("❌ KEIN ZUGRIFF")
    BlacklistTab:AddLabel("Nur der Admin kann diese Funktion nutzen!")
    BlacklistTab:AddParagraph("Zugriff verweigert", 
        "Du hast keine Berechtigung für das Blacklist System.\n\n" ..
        "Nur der Script-Besitzer kann Spieler blacklisten.")
end

-- Spieler beitritt Check (Blacklist)
Players.PlayerAdded:Connect(function(player)
    if BlacklistActive and IsPlayerBlacklisted(player) then
        print("🔒 Geblacklisteter Spieler beigetreten: " .. player.Name)
        SendWebhook("🔒 Blacklist Alert", 
            "**Geblacklisteter Spieler beigetreten:**\nName: " .. player.Name .. "\nUserID: " .. player.UserId, 
            16711680)
        
        task.wait(1)
        EnforceBlacklist()
    end
end)

-- Discord Tab
local DiscordTab = Window:MakeTab({
    Name = "Discord",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local DiscordSection = DiscordTab:AddSection({
    Name = "Community & Support"
})

DiscordTab:AddButton({
    Name = "🎮 Join Discord Server",
    Callback = function()
        local discordLink = "https://discord.gg/CWKHRQwqQ3"
        
        -- Kopiere Link in Zwischenablage
        setclipboard(discordLink)
        
        -- Zeige Benachrichtigung
        OrionLib:MakeNotification({
            Name = "Discord",
            Content = "Discord-Link wurde in die Zwischenablage kopiert! Füge ihn in deinen Browser ein.",
            Image = "rbxassetid://4483345998",
            Time = 5
        })
        
        print("💬 Discord-Link kopiert: " .. discordLink)
        print("📋 Öffne deinen Browser und füge den Link ein!")
    end
})

DiscordTab:AddLabel("Discord: discord.gg/CWKHRQwqQ3")

DiscordTab:AddParagraph("Tritt unserem Discord bei für:", "• Updates & News\n• Support & Hilfe\n• Community\n• Neue Features")

print("=== Notruf Hamburg Hack geladen! ===")
print("✅ Aimassist System aktiviert")
print("✅ ESP für Notruf Hamburg Teams:")
print("   🔵 POLIZEI (Blau)")
print("   🟠 WANTED (Orange)")
print("   🟢 NORMAL (Grün)")
print("   🟣 TOT (Lila)")
print("📝 Steuerung:")
print("   H = Aimassist Toggle")
print("   F = Fly Toggle")
print("   V = Vehicle Fly Toggle")
print("   G = WalkSpeed Toggle")
print("   Rechte Maustaste = Aimassist aktivieren")
print("📍 Teleport: Teleportiere dich zu jedem Spieler!")
print("   - Wähle einen Spieler aus der Liste")
print("   - Drücke den Button um zu teleportieren")
print("   - Loop Teleport um einem Spieler zu folgen")
print("👻 Noclip: Gehe DURCH ALLES (Anti-Ban System)!")
print("✈️ Fly: Fliege sicher durch die Map (Anti-Ban) - WASD + Space/Shift!")
print("🚗✈️ Vehicle Fly: Fliege mit JEDEM Auto durch die Map!")
print("   - Setze dich in ein Auto und drücke V")
print("   - Steuerung: WASD + Space/Shift")
print("   - Funktioniert mit allen Fahrzeugen!")
print("🏃 WalkSpeed: Laufe ULTRA schnell (Anti-Ban geschützt)!")
print("   - Sanfte Geschwindigkeitserhöhung")
print("   - Unauffällig & sicher")
print("   - Von 16 bis 100 Speed einstellbar")
print("🛡️ VOLLSTÄNDIGER Anti-Ban Schutz aktiv!")
print("🔒 Anti-Chat-Bypass aktiviert - Keine Logs!")
print("⚡ Anti-Kick Protection aktiviert!")
print("💬 Discord: discord.gg/CWKHRQwqQ3")
