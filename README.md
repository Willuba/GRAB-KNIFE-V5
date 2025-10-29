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
