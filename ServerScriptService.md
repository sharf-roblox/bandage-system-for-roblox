Server Local Script :
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- Создание RemoteEvent для связи клиент-сервер
local BandageEvent = Instance.new("RemoteEvent")
BandageEvent.Name = "BandageEvent"
BandageEvent.Parent = ReplicatedStorage

-- Таблица для хранения времени последнего использования бинта
local cooldowns = {}

-- Функция для отключения регенерации здоровья
local function disableHealthRegeneration(character)
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.HealthRegenerationTime = 0 -- Отключает регенерацию здоровья
	end
end

-- Отключение регенерации для новых игроков
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(disableHealthRegeneration)
	if player.Character then
		disableHealthRegeneration(player.Character)
	end
end)

-- Обработка использования бинта
BandageEvent.OnServerEvent:Connect(function(player)
	local character = player.Character
	if not character then return end

	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if not humanoid or humanoid.Health <= 0 or humanoid.Health >= humanoid.MaxHealth then return end

	-- Проверка кулдауна (3 секунды)
	local currentTime = tick()
	if cooldowns[player.UserId] and currentTime - cooldowns[player.UserId] < 3 then
		return
	end

	-- Проверка наличия бинта
	local bandage = character:FindFirstChild("Bandage")
	if not bandage or not bandage:IsA("Tool") then return end

	-- Применение лечения (+25 HP, не превышая максимум)
	local newHealth = math.min(humanoid.Health + 25, humanoid.MaxHealth)
	humanoid.Health = newHealth

	-- Удаление бинта
	bandage:Destroy()

	-- Установка кулдауна
	cooldowns[player.UserId] = currentTime
end)

-- Очистка кулдауна при уходе игрока
Players.PlayerRemoving:Connect(function(player)
	cooldowns[player.UserId] = nil
end)
