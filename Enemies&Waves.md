2023-3-13 This was the core of my enemy logic which is where I learn to modualize code and use mathmatical equations like lerp and etc.


```lua
local Module = {}

local ReplicatedStorage = game:GetService("ReplicatedStorage")

local EnemyStats = require(ReplicatedStorage.Module.Configs.EnemyStats)
local EnemyData = require(ReplicatedStorage.Module.EnemyModules.EnemyData)
local ServerData = require(ReplicatedStorage.Module.GameModules.ServerData)
local ServerInfo = ServerData.Get()
Module.Spawn = function(EnemyType, Map, NumberOfEnemies, Seconds, Emotion)
	
	print(EnemyType, Map, NumberOfEnemies, Seconds)
	
	for i = 1, NumberOfEnemies do
		if ServerInfo["BaseHealth"] <= 0 then
			break
		end
		
		local EnemyClone = EnemyType:Clone()
		
		local EnemyCloneName = tostring(EnemyClone)
		local HealthBar = EnemyClone.Head.HealthBar
		
		EnemyClone.Parent = workspace.TowerDefense.ActiveEnemies
		EnemyClone.HumanoidRootPart.CFrame = Map.Path.Spawner.CFrame
		
		EnemyData.EnemyStatus(EnemyClone)
		EnemyData.Set(EnemyClone, "Emotion", Emotion)
		
		local EnemyInfo = EnemyData.Get(EnemyClone)
		
		HealthBar.MaxHealth.CurrentHealth.Health.Text = EnemyInfo["Health"].."/"..EnemyInfo["Health"]
		Module.Move(EnemyClone, Map, Emotion)
		task.wait(Seconds)
	end
end

local function Time(Distance, Speed) -- Distance/Speed = Time
	local Time = Distance/Speed
	return Time
end

Module.Move = function(Enemy, Map)
	task.defer(function()
		local BaseHealthUI = Map.Decorations.Terrain.Endpoint.Cave3.TopOfCave.HealthBar
		local BaseHealth = BaseHealthUI.MaxHealth.CurrentHealth.Health
		
		local EnemyInfo = EnemyData.Get(Enemy)
		
		local WayPointArray = Map.WayPoints:GetChildren()
		
		for x = 1, #WayPointArray do
			
			if Enemy:FindFirstChild("HumanoidRootPart") == nil then
				break
			end
			
			Enemy.HumanoidRootPart.CFrame = Map.WayPoints["WayPoint"..x].CFrame
			if x == #WayPointArray then
				ServerData.Increment("BaseHealth", -EnemyInfo["Health"])
				BaseHealth.Text = ServerInfo["BaseHealth"].."/"..100
				
				Enemy:Destroy()
				print("Destroyed")
				if ServerInfo["BaseHealth"] < 0 then
					BaseHealth.Text = "0/100"
				end
				break
			end
			
			local StartPoint = Map.WayPoints["WayPoint"..x].Position
			local EndPoint = Map.WayPoints["WayPoint"..x+1].Position
			local Alpha = 0
			
			while true do
				local Distance = (StartPoint - EndPoint).Magnitude
				local Time = Time(Distance, EnemyData.Get(Enemy)["Speed"])

				local Increment = 1 / Time
				
				if ServerInfo["BaseHealth"] <= 0 then
					print("GameOver")
					Enemy:Destroy()
					break
				end
				if Alpha >= 1 or x == #WayPointArray then
					break
				end
				
				local TimeElapsed = task.wait()
				Alpha += math.min(Increment * TimeElapsed, 1)
				
				EnemyData.Set(Enemy, "Total Alpha", x+Alpha, true)
				--EnemyData.Increment(Enemy, "Total Alpha", Alpha, true)
				
				if Enemy:FindFirstChild("HumanoidRootPart") == nil then
					break
				end
				local NewPos = Map.WayPoints["WayPoint"..x].CFrame:Lerp(Map.WayPoints["WayPoint"..x+1].CFrame, Alpha).Position
				Enemy.HumanoidRootPart.CFrame = CFrame.lookAt(NewPos, Vector3.new(EndPoint.X, Enemy.HumanoidRootPart.CFrame.Y, EndPoint.Z))
			end
		end
	end)
end


return Module
```
