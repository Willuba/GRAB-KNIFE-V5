function love.conf(t)
    t.window.title = "Grab Knife V4 — Simulación"
    t.window.width = 1280
    t.window.height = 720
    t.window.resizable = true
end-- Grab Knife V4 visual simulation for LÖVE (LOVE2D)
-- Updated: Equip with X (pull knife directly to hand) + melee attack on left click
-- Author: ChatGPT (for Luna)

local Knife = {}
Knife.__index = Knife

function Knife.new(x, y)
    local self = setmetatable({}, Knife)
    self.x = x or 0
    self.y = y or 0
    self.angle = 0
    self.scale = 1
    -- states: idle, grabbed, thrown, returning, equipped, stowed, melee
    self.state = "idle"
    self.timer = 0
    self.orbitRadius = 140

-- KnifeServer (Script) dentro del Tool
local Tool = script.Parent
local knifeRemote = Tool:WaitForChild("KnifeAction")

local DAMAGE = 25
local MELEE_RANGE = 6 -- studs para R6, ajusta
local COOLDOWN = 0.5
local lastUse = {}

local function canUse(player)
    local t = tick()
    if lastUse[player] and t - lastUse[player] < COOLDOWN then
        return false
    end
    lastUse[player] = t
    return true
end

local function onMeleeRequest(player, action, targetPos)
    -- validaciones básicas
    if not canUse(player) then return end
    local char = player.Character
    if not char then return end
    local humanoid = char:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end

    -- raycast desde torso/head hacia targetPos para detectar hit (R6)
    local root = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local origin = root.Position
    local direction = (targetPos - origin).unit * (MELEE_RANGE + 2)
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {char, Tool}
    params.FilterType = Enum.RaycastFilterType.Blacklist
    local hit = workspace:Raycast(origin, direction, params)

    if hit and hit.Instance then
        local hitChar = hit.Instance:FindFirstAncestorOfClass("Model")
        if hitChar then
            local hitHum = hitChar:FindFirstChildWhichIsA("Humanoid")
            if hitHum and hitHum.Health > 0 and hitChar ~= char then
                -- aplicar daño
                hitHum:TakeDamage(DAMAGE)
                -- opcional: Reproducir efectos en servidor (sonidos, partículas replicadas)
            end
        end
    end
end

local function onThrowRequest(player, action, targetPos)
    if not canUse(player) then return end
    -- Implementación simple: crear proyectil físico o efecto visual y realizar raycast/area
    -- Por seguridad, validamos que el jugador tiene el Tool en la mano:
    if not player.Character or not player.Character:FindFirstChildOfClass("Tool") then return end

    -- Ejemplo sencillo: raycast similar al melee con mayor rango
    local char = player.Character
    local root = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local origin = root.Position
    local dir = (targetPos - origin).unit * 100
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {char, Tool}
    params.FilterType = Enum.RaycastFilterType.Blacklist
    local hit = workspace:Raycast(origin, dir, params)

    if hit and hit.Instance then
        local hitChar = hit.Instance:FindFirstAncestorOfClass("Model")
        if hitChar then
            local hitHum = hitChar:FindFirstChildWhichIsA("Humanoid")
            if hitHum and hitHum.Health > 0 and hitChar ~= char then
                hitHum:TakeDamage(DAMAGE * 1.5) -- más daño por proyectil
            end
        end
    end
end

knifeRemote.OnServerEvent:Connect(function(player, action, targetPos)
    if typeof(action) ~= "string" then return end
    if action == "melee" then
        onMeleeRequest(player, action, targetPos)
    elseif
    self.orbitSpeed = 1.2
    self.colorPulse = 0
    -- melee params
    self.meleeTimer = 0
    self.meleeDuration = 0.25
    self.meleeRange = 90
    return self
end

function Knife:equip(ownerX, ownerY)
    self.state = "equipped"
    self.timer = 0
    -- place in front of player immediately
    self.x = ownerX + math.cos(self.angle) * 36
    self.y = ownerY + math.sin(self.angle) * 36 - 8
    self.scale = 1.05
end

function Knife:unequip()
    -- if currently grabbed/throw/returning, force return to stowed
    self.state = "stowed"
    self.timer = 0
end

function Knife:update(dt, ownerX, ownerY)
    self.timer = self.timer + dt

    -- idle orbit
    if self.state == "idle" then
        self.angle = self.angle + self.orbitSpeed * dt
        local ox = math.cos(self.angle) * self.orbitRadius
        local oy = math.sin(self.angle) * (self.orbitRadius * 0.45)
        self.x = ownerX + ox
        self.y = ownerY + oy - 40
        self.scale = 1 + 0.06 * math.sin(self.timer * 3)
    end

    -- grabbed: lock to owner with hold offset
    if self.state == "grabbed" then
        local holdDist = 40
        self.x = ownerX + math.cos(self.angle) * holdDist
        self.y = ownerY + math.sin(self.angle) * holdDist - 20
        self.scale = 1.1
    end

    -- equipped: attach directly to player's hand position
    if self.state == "equipped" then
        local holdDist = 36
        self.x = ownerX + math.cos(self.angle) * holdDist
        self.y = ownerY + math.sin(self.angle) * holdDist - 8
        self.scale = 1.08 + 0.03 * math.sin(self.timer * 6)
    end

    -- melee: short forward l

    -- KnifeLocal (dentro del Tool)
local Tool = script.Parent
local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local remote = Tool:WaitForChild("KnifeAction")

-- Detectar click izquierdo
Tool.Activated:Connect(function()
	local char = player.Character
	if not char then return end
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if not humanoid then return end

	-- Reproducir animación local de "agarre"
	local grabAnim = Instance.new("Animation")
	grabAnim.AnimationId = "rbxassetid://13721831968" -- 👈 reemplaza con tu ID de animación "grab"
	local track = humanoid:LoadAnimation(grabAnim)
	track:Play()

	-- Notificar al servidor que intentas agarrar a alguien
	remote:FireServer("grab", mouse.Hit.Position)
end)

grabAnim.AnimationId

-- KnifeServer (dentro del Tool)
local Tool = script.Parent
local remote = Tool:WaitForChild("KnifeAction")

local GRAB_RANGE = 10
local THROW_FORCE = 120

remote.OnServerEvent:Connect(function(player, action, targetPos)
	if action ~= "grab" then return end

	local char = player.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	-- Buscar jugador más cercano
	local closest = nil
	local minDist = GRAB_RANGE

	for _, other in ipairs(game.Players:GetPlayers()) do
		if other ~= player and other.Character and other.Character:FindFirstChild("HumanoidRootPart") then
			local dist = (hrp.Position - other.Character.HumanoidRootPart.Position).Magnitude
			if dist < minDist then
				minDist = dist
				closest = other
			end
		end
	end

	if closest then
		local otherHRP = closest.Character:FindFirstChild("HumanoidRootPart")
		local otherHum = closest.Character:FindFirstChildOfClass("Humanoid")
		if otherHRP and otherHum then
			-- Levantar al jugador un poco (efecto de agarre)
			otherHRP.CFrame = hrp.CFrame * CFrame.new(0, 2, -2)
			otherHum.PlatformStand = true -- los deja “tiesos”

			-- Esperar un momento y lanzar
			task.wait(0.8)

			local bodyVel = Instance.new("BodyVelocity")
			bodyVel.Velocity = hrp.CFrame.LookVector * THROW_FORCE + Vector3.new(0, 30, 0)
			bodyVel.MaxForce = Vector3.new(1e5, 1e5, 1e5)
			bodyVel.Parent = otherHRP
			game.Debris:AddItem(bodyVel, 0.5)

			-- Restaurar movimiento después de 1 seg
			task.delay(1, function()
				otherHum.PlatformStand = false
			end)
		end
	end
end)

-- KnifeServer (Script) dentro del Tool
local Tool = script.Parent
local knifeRemote = Tool:WaitForChild("KnifeAction")

local DAMAGE = 25
local MELEE_RANGE = 6 -- studs para R6, ajusta
local COOLDOWN = 0.5
local lastUse = {}

local function canUse(player)
    local t = tick()
    if lastUse[player] and t - lastUse[player] < COOLDOWN then
        return false
    end
    lastUse[player] = t
    return true
end

local function onMeleeRequest(player, action, targetPos)
    -- validaciones básicas
    if not canUse(player) then return end
    local char = player.Character
    if not char then return end
    local humanoid = char:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end

    -- raycast desde torso/head hacia targetPos para detectar hit (R6)
    local root = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local origin = root.Position
    local direction = (targetPos - origin).unit * (MELEE_RANGE + 2)
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {char, Tool}
    params.FilterType = Enum.RaycastFilterType.Blacklist
    local hit = workspace:Raycast(origin, direction, params)

    if hit and hit.Instance then
        local hitChar = hit.Instance:FindFirstAncestorOfClass("Model")
        if hitChar then
            local hitHum = hitChar:FindFirstChildWhichIsA("Humanoid")
            if hitHum and hitHum.Health > 0 and hitChar ~= char then
                -- aplicar daño
                hitHum:TakeDamage(DAMAGE)
                -- opcional: Reproducir efectos en servidor (sonidos, partículas replicadas)
            end
        end
    end
end

local function onThrowRequest(player, action, targetPos)
    if not canUse(player) then return end
    -- Implementación simple: crear proyectil físico o efecto visual y realizar raycast/area
    -- Por seguridad, validamos que el jugador tiene el Tool en la mano:
    if not player.Character or not player.Character:FindFirstChildOfClass("Tool") then return end

    -- Ejemplo sencillo: raycast similar al melee con mayor rango
    local char = player.Character
    local root = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local origin = root.Position
    local dir = (targetPos - origin).unit * 100
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {char, Tool}
    params.FilterType = Enum.RaycastFilterType.Blacklist
    local hit = workspace:Raycast(origin, dir, params)

    if hit and hit.Instance then
        local hitChar = hit.Instance:FindFirstAncestorOfClass("Model")
        if hitChar then
            local hitHum = hitChar:FindFirstChildWhichIsA("Humanoid")
            if hitHum and hitHum.Health > 0 and hitChar ~= char then
                hitHum:TakeDamage(DAMAGE * 1.5) -- más daño por proyectil
            end
        end
    end
end

knifeRemote.OnServerEvent:Connect(function(player, action, targetPos)
    if typeof(action) ~= "string" then return end
    if action == "melee" then
        onMeleeRequest(player, action, targetPos)
    
