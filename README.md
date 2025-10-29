-- Grab Knife V4 visual simulation for LÖVE (LOVE2D)
-- Author: ChatGPT (for Luna)
-- Run with LÖVE 11.4+

local Knife = {}
Knife.__index = Knife

function Knife.new(x, y)
    local self = setmetatable({}, Knife)
    self.x = x
    self.y = y
    self.angle = 0
    self.scale = 1
    self.state = "idle" -- idle, grabbed, thrown, returning
    self.timer = 0
    self.orbitRadius = 140
    self.orbitSpeed = 1.2
    self.colorPulse = 0
    return self
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

    -- grabbed: lock to owner
    if self.state == "grabbed" then
        -- slight offset to show being held
        local holdDist = 40
        self.x = ownerX + math.cos(self.angle) * holdDist
        self.y = ownerY + math.sin(self.angle) * holdDist - 20
        self.scale = 1.1
    end

    -- thrown: move forward and spin, fade out a bit
    if self.state == "thrown" then
        local speed = 800
        self.x = self.x + math.cos(self.angle) * speed * dt
        self.y = self.y + math.sin(self.angle) * speed * dt
        self.scale = self.scale +
