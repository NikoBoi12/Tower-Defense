2023-4-6 This was my first ever attempt at UI and front end development this taught me a lot about scaling for different devices and to make nice looking visuals
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local TweenGoals = require(ReplicatedStorage.Module.NikoModules.TweenGoals)
local EasingStyle = require(ReplicatedStorage.Module.NikoModules.EasingStyle)
local Controls = require(game.Players.LocalPlayer.PlayerScripts:WaitForChild("PlayerModule")):GetControls()---GetControls

local LocalPlayer = Players.LocalPlayer
local PlayerUi = LocalPlayer.PlayerGui
local MainMenuScreen = PlayerUi:WaitForChild("MainMenu")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local HRP = Character:WaitForChild("HumanoidRootPart")

local Camera = workspace.CurrentCamera

local Sounds = ReplicatedStorage.Storage.Sounds

local LoadingScreen = MainMenuScreen.LoadingScreen
local LoadingBackground = LoadingScreen.Background
local LoadingText = LoadingScreen.Loading.LoadingText
local MainMenu = MainMenuScreen.MainMenu
local Background = MainMenu.Backgrounds

local Points = workspace.Intro

local MenuConnectionArray = {} -- Array with button connections
local PointArray = {Points.Point2, Points.Point3}
local OmoriAnimationArray = {.142, -.711, -1.579}
local LightAnimationArray = {0.041, -0.928, -1.89}
local LoadingArray = {"Loading .", "Loading . .", "Loading . . ."}

local BackgroundArray = {
	{
		BackgroundImage = Background.DeepWell,
		x = {-.5,.5,1.5},
		y = {0, 0, 0},
	},
	{
		BackgroundImage = Background.Ice,
		x = {-.5, .5, 1.5},
		y = {0, 0, 0}
	},
	{
		BackgroundImage = Background.OtherWorld,
		x = {.5, .5, .5},
		y = {-2, -1, 0},
	},
	{
		BackgroundImage = Background.OrangeOasis,
		x = {-.5, .5, 1.5},
		y = {0, 0, 0}
	},
}

local x = 0
local DotCount = 1
local BreakLoop = false

local BackgroundInfo = BackgroundArray[4] --math.random(1, 3)

local function Menu(Button)
	if Button.Name == "Start" then
		
		BreakLoop = true
		Sounds.Music.Title:Pause()
		
		Camera.CFrame = Points.Point1.CFrame
		
		MainMenu:Destroy()
		Character.Parent = workspace.Entities
		HRP.CFrame = Points.Spawn.CFrame
		Controls:Disable()
		
		Sounds.Music.WhiteSpace:Play()
		
		task.wait(2)
		
		for i, Points in PointArray do
			local CameraGoal = {}
			CameraGoal.CFrame = Points.CFrame
			
			local CameraTween = TweenService:Create(Camera, TweenInfo.new(4, EasingStyle.Linear), CameraGoal)
			CameraTween:Play()
			CameraTween.Completed:Wait()
			task.wait(1)
		end
		
		print("Completed Tweening of Camera")
		
		Camera.CameraType = Enum.CameraType.Custom
		Camera.CFrame = Points.Point3.CFrame
		Controls:Enable()
		PlayerUi.PlayerInterface.Enabled = true
	elseif Button.Name == "Changelog" then
		--Add code for changelogs later
		print("Change Logs appeared :D")
		
	elseif Button.Name == "Credits" then
		local Credits = MainMenuScreen.Credits
		
		for i, Button in MainMenu:GetChildren() do
			if Button:IsA("GuiButton") then
				Button.Visible = false
			end
		end
		
		Credits.Visible = true

		Credits.Frame.Back.Activated:Connect(function()
			MainMenuScreen.Credits.Visible = false
			for i, Button in MainMenu:GetChildren() do
				if Button:IsA("GuiButton") then
					Button.Visible = true
				end
			end
		end)
	end
end


MainMenuScreen.LoadingScreen.Visible = true

Camera.CameraType = Enum.CameraType.Scriptable
Character.Parent = game.Lighting



while x ~= 6 do
	if DotCount > 3 then
		DotCount = 1
	end
	print(DotCount)
	LoadingText.Text = LoadingArray[DotCount]
	task.wait(1)
	DotCount += 1
	x += 1
end

LoadingText:Destroy()

MainMenu.Visible = true
BackgroundInfo.BackgroundImage.Visible = true
Sounds.Music.Title:Play()


LoadingScreen:Destroy()

for i, Buttons in MainMenu:GetChildren() do
	if Buttons:IsA("GuiButton") then
		MenuConnectionArray[#MenuConnectionArray + 1] = Buttons.Activated:Connect(function()
			Menu(Buttons)
		end)
	end
end

local Cooldown = false

for i, Buttons in MainMenuScreen:GetDescendants() do
	if Buttons:IsA("GuiButton") then
		MenuConnectionArray[#MenuConnectionArray + 1] = Buttons.MouseEnter:Connect(function()
			if Cooldown == false then
				Cooldown = true
				print("Worked")
				Sounds.SFX.Select:Play()
				task.wait(.1)
				Cooldown = false
			end
		end)
	end
end

local i = 1

while not BreakLoop do
	if i == 4 then
		i = 1
	end
	print(BackgroundInfo.x[i], BackgroundInfo.y[i])
	MainMenu.OmoriTitle.OmoriSprite.Position = UDim2.fromScale(OmoriAnimationArray[i], 0)
	MainMenu.Title.Light.LightEffect.Position = UDim2.fromScale(LightAnimationArray[i], 0.527)
	BackgroundInfo.BackgroundImage.Position = UDim2.fromScale(BackgroundInfo.x[i], BackgroundInfo.y[i])
	print("Adding")
	i += 1
	
	task.wait(.3)
end
```
