local RS = game:GetService("ReplicatedStorage")

local uiChangeEvent = RS.Events.Remote.UiRoundEvent

local Time = game.ReplicatedStorage.RoundData.Time
local ball = workspace.Lobby.ball
local redTeam = game.Teams.RedTeam
local blueTeam = game.Teams.RedTeam
local player = game.Players.LocalPlayer
local Players = game:GetService("Players")
local Client = Players:GetPlayerFromCharacter(player)
local Teams = game:GetService("Teams")
local teams = Teams:GetTeams()
local winner

--SETTING--

local ITimeBeforeTeleport = 10
local IPreparingTime = 10
local IRoundTime = 20
local IChooseMapTime = 10

--END--

--dont touch--

local TimeBeforeTeleport = ITimeBeforeTeleport
local RoundTime = IRoundTime
local PreparingTime = IPreparingTime
local ChooseMapTime = IChooseMapTime

--till here--

--Functions--

function giveTeamCoins(teamColor,amount)
	for i,v in ipairs(game.Players:GetChildren()) do
		if v.TeamColor == BrickColor.new(teamColor) then
			v.leaderstats.Coins.Value = v.leaderstats.Coins.Value + amount
		end
	end
end

function killPlayers()
	for _, player in pairs(game.Players:GetChildren()) do
		local char = player.Character
		if char:WaitForChild("Humanoid").Health > 0 then
			char:WaitForChild("Humanoid").Health = 0
		end
	end
end

function ChangeToLobby()
	for _, player in pairs(game.Players:GetPlayers()) do
		player.Team = Teams.Lobby
		
	end
end

function AutoTeam()
	for i, plr in ipairs(game.Players:GetPlayers()) do
		if i%2 == 0 then 
			plr.Team = game.Teams.RedTeam
		else
			plr.Team = game.Teams.BlueTeam
		end
	end
end

Players.PlayerAdded:Connect(function(plr)
	plr.Team = Teams.Lobby
end)

RS.Events.Remote.Map1Vote.OnServerEvent:Connect(function()
	RS.Map1Votes.Value += 1
end)

RS.Events.Remote.Map2Vote.OnServerEvent:Connect(function()
	RS.Map2Votes.Value += 1
end)

--end--

--round--

while true do
	RoundTime = IRoundTime
	
	Time.Value = ITimeBeforeTeleport
	TimeBeforeTeleport = ITimeBeforeTeleport
	RS.Events.Remote.Intermission:FireAllClients(player, nil)
	
	-- intermission --
	
	repeat 
		task.wait(1)
		Time.Value -= 1
		TimeBeforeTeleport -= 1
	until TimeBeforeTeleport <= 0


	TimeBeforeTeleport = ITimeBeforeTeleport
		
	Time.Value = IChooseMapTime
	RS.Events.Remote.VotingTime:FireAllClients(player, nil)
	AutoTeam()
	
	-- Voting --
	
	repeat
		task.wait(1)
		Time.Value -= 1
		TimeBeforeTeleport -= 1
	until TimeBeforeTeleport <= 0
	
	RS.Events.Remote.PrepTime:FireAllClients(player, nil)
	
	if RS.Map1Votes.Value > RS.Map2Votes.Value then
		workspace.Lobby.Map.floor.Color = Color3.fromRGB(44, 101, 29)
		
		for i, model in pairs(workspace.Lobby.Lines.Map2Lines:GetChildren()) do
			if model:IsA('Part') then
				model.Transparency = 1
			end
		end
		
		for i, model in pairs(workspace.Lobby.Lines:GetChildren()) do
			if model:IsA('Part') then
				model.Color = Color3.fromRGB(255, 255, 255)
			end
		end
		RS.Map1Votes.Value = 0
		RS.Map2Votes.Value = 0
	elseif RS.Map2Votes.Value > RS.Map1Votes.Value then
		workspace.Lobby.Map.floor.Color = Color3.fromRGB(128, 187, 219)

		for i, model in pairs(workspace.Lobby.Lines.Map2Lines:GetChildren()) do
			if model:IsA('Part') then
				model.Transparency = 0
			end
		end

		for i, model in pairs(workspace.Lobby.Lines:GetChildren()) do
			if model:IsA('Part') then
				model.Color = Color3.fromRGB(255, 0, 0)
			end
		end
		RS.Map1Votes.Value = 0
		RS.Map2Votes.Value = 0
	end
	
	ChooseMapTime = IChooseMapTime
	Time.Value = IPreparingTime

	workspace.RedSpawn.Position = Vector3.new(60.452, 10, -15.976)
	workspace.SpawnLocation.Transparency = 1
	workspace.SpawnLocation.Decal.Transparency = 1
	workspace.BlueSpawn.Position = Vector3.new(60.452, 10, -15.976)
	workspace.SpawnLocation.Transparency = 1
	workspace.SpawnLocation.Decal.Transparency = 1

	wait(1)
	killPlayers()
	task.wait(0.5)

	task.wait(1)

	workspace.RedSpawn.Position = Vector3.new(60.489, 5, 149.788)
	workspace.SpawnLocation.Transparency = 0
	workspace.SpawnLocation.Decal.Transparency = 0
	workspace.BlueSpawn.Position = Vector3.new(60.489, 5, -181.826)
	workspace.SpawnLocation.Transparency = 0
	workspace.SpawnLocation.Decal.Transparency = 0
	task.wait(0.1)

	-- preptime --
	
	repeat
		task.wait(1)
		Time.Value -= 1
		PreparingTime -= 1
	until PreparingTime <= 0

	PreparingTime = IPreparingTime
	Time.Value = IRoundTime

	ball.CanCollide = true
	ball.Anchored = false
	ball.Transparency = 0
	ball.Velocity = Vector3.new()
	ball.AssemblyAngularVelocity = Vector3.new()
	RS.Events.Remote.RoundTime:FireAllClients()
	
	-- round time --
	
	repeat -- round time
		task.wait(1)
		Time.Value -= 1
		RoundTime -= 1
	until RoundTime <= 0
	
	ball.Anchored = true
	ball.Position = Vector3.new(60.366, 7.596, -16.019)
	ball.Velocity = Vector3.new()
	ball.AssemblyAngularVelocity = Vector3.new()
	ball.Transparency = 1
	
	if game.PrivateServerId ~= "" and game.PrivateServerOwnerId ~= 0 then

		-- listen for new players being added
		
		Players.PlayerAdded:Connect(function(player)

			-- check if the player is the server owner
			
			if player.UserId == game.PrivateServerOwnerId then
				local part = Instance.new("Part")
				part.Parent = workspace
				part.Position = workspace.SpawnLocation.Position
			end
		end)
	-- give coins to the team that won --
	else
		if RS.BlueScore.Value > RS.RedScore.Value then
			giveTeamCoins("Deep blue", 20)
			RS.BlueScore.Value = 0
			RS.RedScore.Value = 0
			winner = "blue"
			RS.Events.Remote.RoundFinished:FireAllClients(winner)
		elseif RS.RedScore.Value > RS.BlueScore.Value then
			giveTeamCoins("Really red", 20)
			RS.BlueScore.Value = 0
			RS.RedScore.Value = 0
			winner = "red"
			RS.Events.Remote.RoundFinished:FireAllClients(winner)
		elseif RS.BlueScore.Value == RS.RedScore.Value and RS.BlueScore.Value >= 1 then
			giveTeamCoins("Deep blue", 5)
			giveTeamCoins("Really red", 5)
			RS.BlueScore.Value = 0
			RS.RedScore.Value = 0
			winner = "tie"
			RS.Events.Remote.RoundFinished:FireAllClients(winner)
		end
	end
	
	-- teleport to lobby --
	
	task.wait(3)
	ChangeToLobby()
	killPlayers()
end

--end of round--
