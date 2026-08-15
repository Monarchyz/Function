local function SetupFastVent(Tab)
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local LP = Players.LocalPlayer

    local FastVentEnabled = false
    local isInVent = false
    local ventConnection = nil
    local CrawlAnimTrack = nil
    local cachedHitboxes = nil
    local lastMapCheck = 0
    local currentMap = nil
    local mapConnection = nil

    local function GetCurrentMap()
        local map = ReplicatedStorage:FindFirstChild("CurrentMap")
        if map and map:IsA("ObjectValue") and map.Value then
            return map.Value
        end
        return nil
    end

    local function IsBeast()
        local stats = LP:FindFirstChild("TempPlayerStatsModule")
        if stats then
            local isBeast = stats:FindFirstChild("IsBeast")
            if isBeast and isBeast.Value then
                return true
            end
        end
        return false
    end

    local function GetVentHitboxes()
        local now = tick()
        local map = GetCurrentMap()
        
        if map ~= currentMap then
            currentMap = map
            cachedHitboxes = nil
        end
        
        if cachedHitboxes and now - lastMapCheck < 5 then
            return cachedHitboxes
        end
        
        if not map then return {} end
        
        cachedHitboxes = {}
        for _, obj in pairs(map:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Name == "VentHitbox" then
                table.insert(cachedHitboxes, obj)
            end
        end
        lastMapCheck = now
        return cachedHitboxes
    end

    local function IsInsideAnyVent()
        local char = LP.Character
        if not char then return false end
        
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return false end
        
        local hitboxes = GetVentHitboxes()
        for _, hitbox in pairs(hitboxes) do
            if hitbox and hitbox.Parent then
                local distance = (hitbox.Position - hrp.Position).Magnitude
                if distance < 3 then
                    return true
                end
            end
        end
        return false
    end

    local function FindCrawlAnimation()
        local map = GetCurrentMap()
        if map then
            local anim = map:FindFirstChild("AnimCrawl", true)
            if anim and anim:IsA("Animation") then return anim end
        end
        
        local anim = workspace:FindFirstChild("AnimCrawl", true)
        if anim and anim:IsA("Animation") then return anim end
        
        anim = ReplicatedStorage:FindFirstChild("AnimCrawl", true)
        if anim and anim:IsA("Animation") then return anim end
        
        return nil
    end

    local function LoadCrawlAnimation(humanoid)
        if CrawlAnimTrack then return CrawlAnimTrack end
        local animation = FindCrawlAnimation()
        if not animation then return nil end
        
        local animator = humanoid:FindFirstChildOfClass("Animator")
        if not animator then
            animator = Instance.new("Animator")
            animator.Parent = humanoid
        end
        
        local success, track = pcall(function()
            return animator:LoadAnimation(animation)
        end)
        
        if success and track then
            CrawlAnimTrack = track
            CrawlAnimTrack.Priority = Enum.AnimationPriority.Action
        end
        
        return CrawlAnimTrack
    end

    local function SpinCharacter()
        local char = LP.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        
        hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(180), 0)
    end

    local function CreateVentHitboxes()
        local map = GetCurrentMap()
        if not map then return end

        for _, obj in pairs(map:GetDescendants()) do
            if obj:IsA("BasePart") and string.find(string.lower(obj.Name), "ventblock") then
                local old = obj:FindFirstChild("VentHitbox")
                if old then old:Destroy() end

                local s = obj.Size

                local depthAxis = "X"
                local smallest = s.X
                if s.Y < smallest then
                    depthAxis = "Y"
                    smallest = s.Y
                end
                if s.Z < smallest then
                    depthAxis = "Z"
                    smallest = s.Z
                end

                local newSize = s
                if depthAxis == "X" then
                    newSize = Vector3.new(s.X * 5, s.Y, s.Z)
                elseif depthAxis == "Y" then
                    newSize = Vector3.new(s.X, s.Y * 5, s.Z)
                else
                    newSize = Vector3.new(s.X, s.Y, s.Z * 5)
                end

                local hitbox = Instance.new("Part")
                hitbox.Name = "VentHitbox"
                hitbox.Size = newSize
                hitbox.CFrame = obj.CFrame
                hitbox.Anchored = true
                hitbox.CanCollide = false
                hitbox.CanTouch = false
                hitbox.Transparency = 1
                hitbox.BrickColor = BrickColor.new("Bright blue")
                hitbox.Material = Enum.Material.Neon
                hitbox.Parent = obj
            end
        end

        cachedHitboxes = nil
        lastMapCheck = 0
    end

    local function OnMapChanged()
        if FastVentEnabled then
            cachedHitboxes = nil
            currentMap = nil
            CreateVentHitboxes()
        end
    end

    local function CheckMapLoop()
        task.spawn(function()
            while FastVentEnabled do
                task.wait(1)
                local current = GetCurrentMap()
                if current ~= currentMap then
                    currentMap = current
                    cachedHitboxes = nil
                    CreateVentHitboxes()
                end
            end
        end)
    end

    local function StartVentLoop()
        if ventConnection then return end
        
        CreateVentHitboxes()
        CheckMapLoop()
        
        local mapValue = ReplicatedStorage:FindFirstChild("CurrentMap")
        if mapValue then
            if mapConnection then
                mapConnection:Disconnect()
                mapConnection = nil
            end
            mapConnection = mapValue:GetPropertyChangedSignal("Value"):Connect(OnMapChanged)
        end
        
        ventConnection = RunService.Heartbeat:Connect(function()
            if not FastVentEnabled then
                if ventConnection then
                    ventConnection:Disconnect()
                    ventConnection = nil
                end
                return
            end
            
            if IsBeast() then
                if isInVent then
                    isInVent = false
                    local char = LP.Character
                    if char then
                        local humanoid = char:FindFirstChildOfClass("Humanoid")
                        if humanoid then
                            humanoid.HipHeight = 0
                            humanoid.WalkSpeed = 16
                            if CrawlAnimTrack then
                                pcall(function() CrawlAnimTrack:Stop(0.15) end)
                            end
                        end
                    end
                end
                return
            end
            
            local char = LP.Character
            if not char then return end
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not humanoid or not hrp then return end
            
            local inside = IsInsideAnyVent()
            
            if inside and not isInVent then
                isInVent = true
                humanoid.HipHeight = -2
                humanoid.WalkSpeed = 18
                
                local crawl = LoadCrawlAnimation(humanoid)
                if crawl then
                    pcall(function() crawl:Play(0.15, 1, 3) end)
                end
                
                task.spawn(SpinCharacter)
                
            elseif not inside and isInVent then
                isInVent = false
                humanoid.HipHeight = 0
                humanoid.WalkSpeed = 16
                if CrawlAnimTrack then
                    pcall(function() CrawlAnimTrack:Stop(0.15) end)
                end
            end
        end)
    end

    local function StopVent()
        FastVentEnabled = false
        if ventConnection then
            ventConnection:Disconnect()
            ventConnection = nil
        end
        if mapConnection then
            mapConnection:Disconnect()
            mapConnection = nil
        end
        
        local char = LP.Character
        if char then
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.HipHeight = 0
                humanoid.WalkSpeed = 16
            end
        end
        
        if CrawlAnimTrack then
            pcall(function() CrawlAnimTrack:Stop(0.15) end)
            CrawlAnimTrack = nil
        end
        isInVent = false
        cachedHitboxes = nil
        currentMap = nil
    end

    return {
        Start = StartVentLoop,
        Stop = StopVent,
        IsEnabled = function() return FastVentEnabled end,
        SetEnabled = function(value)
            FastVentEnabled = value
            if value then
                StartVentLoop()
            else
                StopVent()
            end
        end
    }
end

return SetupFastVent
