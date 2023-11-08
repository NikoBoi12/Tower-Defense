10123-3-7 Tower Defense core code for  my towers this is one of my very first actual projects that involved me having to put logic and different recourses to all work togeather to get my desired result it helped me learn tables, logic, loops, math, and UI.

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")
local Players = game:GetService("Players")

local TowerStats = require(ReplicatedStorage.Module.Configs.TowerStats)
local EnemyStats = require(ReplicatedStorage.Module.Configs.EnemyStats)
local TowerEmotion = require(ReplicatedStorage.Module.Configs.TowerEmotions)
local PlayerData = require(ReplicatedStorage.Module.GameModules.PlayerData)
local TowerData = require(ReplicatedStorage.Module.TowerModules.TowerData)
local GameData = require(ReplicatedStorage.Module.GameModules.GameData)
local ServerData = require(ReplicatedStorage.Module.GameModules.ServerData)
local EnemyData = require(ReplicatedStorage.Module.EnemyModules.EnemyData)

local SellRemote = ReplicatedStorage.Remote.SellRemote
local PlaceTowerRemote = ReplicatedStorage.Remote.PlaceTower
local UpgradeRemote = ReplicatedStorage.Remote.UpgradeRemote

local ActiveTowers = workspace.TowerDefense.ActiveTowers
local ActiveEnemies = workspace.TowerDefense.ActiveEnemies
local VFXStorage = workspace.TowerDefense.NotActiveVFX

local TowerStorage = ReplicatedStorage.Storage.TowerStorage

local TowerAttacked = {}--[[
TowerAttacked[Tower] = (bool)

]]

local function TowerPlacment(Player, TowerCloneCFrame, TowerName, IsAHero) -- Places a Tower
	local TowerInfo = TowerStats[TowerName]
	local TowerType = TowerInfo["Tower Name"]
	local NumOfTowers = 0
	
	local TowerGameData = GameData.Get(Player)["PlacedTowers"]
	
	if IsAHero then
		TowerGameData = GameData.Get(Player)["PlacedHeros"]
	end
	local IsCapped = GameData.TowerCap(Player, TowerName, IsAHero)
	
	if IsCapped == true then
		return
	end
	
	if GameData.Get(Player)["Money"] >= TowerInfo["Cost"] then -- Rechecks if you can place this tower
		local Range = TowerInfo["Range"]
		local TowerObject = TowerStorage:FindFirstChild(TowerName) 
		local Tower = TowerObject:Clone()
		
		GameData.Increment(Player, "Money", -TowerInfo["Cost"])
		
		Tower.Parent = ActiveTowers
		Tower.TowerDefenseParts.HitBoxRadius.CFrame = TowerCloneCFrame
		Tower.TowerDefenseParts.Range.Size = Vector3.new(.1,Range*2,Range*2) -- Sets range size
		Tower.TowerDefenseParts.Range.Transparency = 1
		
		GameData.TowerPlaced(Player, Tower)
		TowerData.TowerStatus(Tower)
		
	end
end

PlaceTowerRemote.OnServerEvent:Connect(TowerPlacment)


local function TowerSell(Player, TowerName, Tower)
	local TowerInfo = TowerStats[TowerName]
	local i = 0
	local IsAHero = TowerInfo["Is A Hero"]
	GameData.Increment(Player, "Money", TowerInfo["Sell Price"])
	print(Tower)
	GameData.TowerRemoved(Player, Tower, IsAHero)
	TowerData.RemoveTowerStatus(Tower)
	Tower:Destroy()
end

SellRemote.OnServerEvent:Connect(TowerSell)


local function TowerUpgrade(Player, Tower, TowerCFrame)
	local TowerInfo = TowerStats[Tower.Name]
	local PlacedTower = GameData.Get(Player)["PlacedTowers"]
	
		
	
	if table.find(PlacedTower, Tower) and  GameData.Get(Player)["Money"] >= TowerStats[Tower.Name]["Upgrade Price"] then
		if TowerInfo["Next Upgrade"] ~= nil then
			
			GameData.TowerRemoved(Player, Tower)
			
			print("Tower is being Upgraded")
			
			local UpgradedTowerStorage = TowerInfo["Next Upgrade"]
			local TowerUpgradeLocation = TowerInfo["Tower Name"]
			local TowerUpgradeFolder = TowerStorage:FindFirstChild(TowerUpgradeLocation.." Upgrades")
			local TowerUpgradeFind = TowerUpgradeFolder:FindFirstChild(UpgradedTowerStorage)
			
			local UpgradedTower = TowerUpgradeFind:Clone()
			local UpgradedTowerParts = UpgradedTower.TowerDefenseParts
			
			local Range = TowerStats[UpgradedTower.Name]["Range"]
			
			UpgradedTower.Parent = ActiveTowers
			UpgradedTowerParts.Range.Transparency = 1
			UpgradedTowerParts.HitBoxRadius.CFrame = TowerCFrame
			UpgradedTowerParts.Range.Size = Vector3.new(.1,Range*2,Range*2)
			
			GameData.Increment(Player, "Money", -TowerInfo["Upgrade Price"])
			GameData.TowerPlaced(Player, UpgradedTower)
			TowerData.TowerStatus(UpgradedTower)
			TowerData.RemoveTowerStatus(Tower, UpgradedTower)
			Tower:Destroy()
		end
	end
end

UpgradeRemote.OnServerEvent:Connect(TowerUpgrade)


local function Effects(Tower, SpecialAbility)
	local SFX = Tower.Sounds.AttackAudio
	local HeroAbilities = TowerStats[Tower.Name]["Hero Ability"]
	
	if SpecialAbility then
		SFX = HeroAbilities["SFX"]
	end
	
	if SFX then
		SFX:Play()
	end

	if Tower.VFX:FindFirstChild("AttackVFX") then -- Triggers VFX
		--ModuleCodeHere
	end
end


local function Attack(Tower, Enemy, Damage)
	
	EnemyData.Increment(Enemy, "Health", -Damage)
	print(Enemy)
	local MaxHealth = EnemyStats[Enemy.Name]["Health"]
	local HealthBar = Enemy.Head.HealthBar
	
	HealthBar.MaxHealth.CurrentHealth.Health.Text = EnemyData.Get(Enemy)["Health"].."/"..MaxHealth
	
	if EnemyData.Get(Enemy)["Health"] <= 0 then -- Destroys the Enemy if there health reaches 0
		Enemy:Destroy()
		for _, Player in Players:GetChildren() do
			GameData.Increment(Player, "Money", EnemyStats[Enemy.Name]["MoneyPerKill"])
		end	
	end
end


local function TargetMode(Tower, Mode) -- Sets what the target will be

	local TowerPosition = Tower.HumanoidRootPart.Position
	
	local ActiveTowers = ActiveEnemies:GetChildren()
	
	local FirstEnemy
	local AllEnemies = {}
	local HealthyEnemy = {}
	local Distance
	local Health

	for i, Enemy in ActiveTowers do
		local EnemyPosition = Enemy.HumanoidRootPart.Position

		local EnemyDistance = (TowerPosition - EnemyPosition).Magnitude
		local TotalDistance = EnemyData.Get(Enemy)["Total Alpha"]
		local TotalHealth = EnemyData.Get(Enemy)["Health"]

		if EnemyDistance <= TowerStats[Tower.Name]["Range"] then
			if Mode == "First" then
				if not FirstEnemy or TotalDistance > Distance then
					FirstEnemy = Enemy
					Distance = TotalDistance
				end
			end
			if Mode == "Last" then
				if not FirstEnemy or TotalDistance < Distance then
					FirstEnemy = Enemy
					Distance = TotalDistance
				end
			end
			if Mode == "AOE" then
				table.insert(AllEnemies, Enemy)
			end
			if Mode == "Strongest" then
				if not Health or TotalHealth > Health then
					Health = TotalHealth
				end
			end
			if Mode == "Weakest" then
				if not Health or TotalHealth < Health then
					Health = TotalHealth
				end
			end
		end
	end

	if Mode == "Strongest" or Mode == "Weakest" then
		for i, Enemy in ActiveTowers do
			if EnemyData.Get(Enemy)["Health"] == Health then
				table.insert(HealthyEnemy, Enemy)
			end
		end 
		for i, Enemy in HealthyEnemy do
			local TotalDistance = EnemyData.Get(Enemy)["Total Alpha"]
			
			if not FirstEnemy or TotalDistance > Distance then
				FirstEnemy = Enemy
				Distance = TotalDistance
			end
		end
	end
	table.insert(AllEnemies, FirstEnemy)
	
	if #AllEnemies == 0 then
		return nil
	end

	
	return AllEnemies
end


local function Abilities(Tower, Special)
	local TowerInfo = TowerStats[Tower.Name]
	local GetInfo = TowerData.Get(Tower)
	local Emotion = GetInfo["Emotion"]
	local Ability = "Special Ability"
	local Damage = TowerInfo["Attack"]

	if Special == true then
		Ability = "Hero Ability"
		Damage = TowerInfo[Ability]["Damage"] * TowerEmotion[Emotion]["Attack"]
		print(TowerInfo[Ability]["Damage"], TowerEmotion[Emotion]["Attack"])
		print(TowerInfo[Ability]["Damage"] * TowerEmotion[Emotion]["Attack"])
		print(Emotion)
		
	end
	
	local CurrentTarget = GetInfo["CurrentTarget"]
	local TargetOptions = GetInfo["TargetOptions"]

	local HeroAbilities = TowerInfo[Ability]
	local AbilitySFX = HeroAbilities["SFX"]
	local AbilityType = HeroAbilities["Ability"]
	
	local Target = TargetMode(Tower, TargetOptions[CurrentTarget])
	
	if AbilityType == "Single" then
		if TargetOptions[CurrentTarget] == "AOE" then
			Target = TargetMode(Tower, "First")
		end
		for i, Enemy in Target do
			EnemyData.Slowness(Tower, Enemy, Ability)
			
			Attack(Tower, Enemy, Damage)
		end
	elseif AbilityType == "AOE" then
		local Target = TargetMode(Tower, "AOE")
		for i, Enemy in Target do
			EnemyData.Slowness(Tower, Enemy, Ability)
			Attack(Tower, Enemy, Damage)
		end
	end
	
	for _, Player in Players:GetChildren() do
		GameData.Increment(Player, "Money", HeroAbilities["Money"])
	end	
	ServerData.Increment("BaseHealth", HeroAbilities["Healing"])
	Effects(Tower, true)
end


while true do 
	
	for _,Tower in ActiveTowers:GetChildren() do
		if TowerAttacked[Tower] == nil then
			local CurrentTarget = TowerData.Get(Tower)["CurrentTarget"]
			local TargetOptions = TowerData.Get(Tower)["TargetOptions"]
			local Target = TargetMode(Tower, TargetOptions[CurrentTarget])
			
			local TowerInfo = TowerStats[Tower.Name]
			local HeroTower = TowerStats[Tower.Name]["Is A Hero"]
			local SFX = Tower.Sounds
			if Target then
		
				local GetData = TowerData.Get(Tower)
				local SpecialAbility = false
				
				if HeroTower then
					TowerData.Increment(Tower, "Juice", 25)
					print(TowerData.Get(Tower)["Juice"])
					if GetData["Juice"] >= 100 then
						print("Juiced Up!")
						
						TowerData.Increment(Tower, "Juice", -100)
						Abilities(Tower, true)
						SpecialAbility = true
					end
				end
				if SpecialAbility == false then
					for i, Enemy in Target do
						if not TowerInfo["Special Ability"] then
							Attack(Tower, Enemy, GetData["Attack"])
						end
					end
					if TowerInfo["Special Ability"] then
						Abilities(Tower)
					end
					Effects(Tower)
				end		
				
				TowerAttacked[Tower] = true
				task.defer(function() -- Attack Cooldown
					task.wait(TowerStats[Tower.Name]["Attack Speed"]) -- Waits for the AttackSpeed
					TowerAttacked[Tower] = nil -- Allows the tower to attack again
				end)
			end
		end
	end
	task.wait()
end
```
