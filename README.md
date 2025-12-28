local Players = game:GetService("Players")
локальный RunService = игра:GetService("RunService")
локальный игрок = Игроки.ЛокальныйИгрок
local backpack = player:WaitForChild("Backpack")

-- Создаем инструмент Grab
local grabTool = Instance.new("Tool")
grabTool.Name = "Grab"
grabTool.ToolTip = "Захватить детали"
grabTool.CanBeDropped = false
grabTool.RequiresHandle = false

-- Переменные для управления
local currentPart = nil
local bodyPosition = nil
local bodyGyro = nil
локальное соединение = nil

локальная функция cleanupForces()
	если bodyPosition тогда
		bodyPosition:Destroy()
		bodyPosition = nil
	конец
	если bodyGyro тогда
		bodyGyro:Destroy()
		bodyGyro = nil
	конец
	если соединение установлено, то
		соединение:Отключиться()
		соединение = nil
	конец
	currentPart = nil
конец

локальная функция createBodyForces(part)
	-- Сначала очищаем холодную силу, если она есть.
	если part:FindFirstChild("BodyPosition") then
		part.BodyPosition:Destroy()
	конец
	если part:FindFirstChild("BodyGyro") then
		part.BodyGyro:Destroy()
	конец
	
	-- УЛУВЕЛИЧИЛИ СИ BodyPosition для поднятия объектов
	local bp = Instance.new("BodyPosition")
	bp.MaxForce = Vector3.new(1000000, 1000000, 1000000) -- УВЕЛИЧЕНО С 40000
	бп.П = 10000 -- УВЕЛИЧЕНО
	б.д.= 1000 -- УВЕЛИЧЕНО
	
	-- УВЕЛИЧИЛИ СИЛУ BodyGyro для лучшего качества
	local bg = Instance.new("BodyGyro")
	bg.MaxTorque = Vector3.new(1000000, 1000000, 1000000) -- УВЕЛИЧЕНО С 40000
	bg.P = 16000 -- УВЕЛИЧЕНО
	bg.D = 1600 -- УВЕЛИЧЕНО
	
	bp.Parent = part
	bg.Parent = часть
	
	возвращать bp, bg
конец

локальная функция getFollowPosition()
	локальный персонаж = игрок.Персонаж
	если символ тогда
		local root = character:FindFirstChild("HumanoidRootPart")
		если root, то
			-- Позиция перед игроком
			return root.Position + root.CFrame.LookVector * 5 + Vector3.new(0, 2, 0)
		конец
	конец
	return Vector3.new(0, 0, 0)
конец

локальная функция updatePartPosition()
	Если не currentPart или не bodyPosition, то вернуть end
	
	local targetPos = getFollowPosition()
	bodyPosition.Position = targetPos
	
	-- Мягко останавливаем физику
	currentPart.Velocity = currentPart.Velocity * 0.5
	currentPart.RotVelocity = currentPart.RotVelocity * 0.5
конец

-- Улучшенная доработка функциональной части
локальная функция findPartNearCharacter()
	локальный персонаж = игрок.Персонаж
	если не является символом, то вернуть нулевое значение.
	
	local root = character:FindFirstChild("HumanoidRootPart")
	если не root, то вернуть nil end
	
	local closestPart = nil
	local closestDistance = 15
	
	-- Ищем самую нижнюю близкую часть
	for _, obj in pairs(workspace:GetDescendants()) do
		Если obj:IsA("BasePart") и не obj.Anchored и obj.Parent ~= character, то
			локальное расстояние = (obj.Position - root.Position).Magnitude
			если расстояние < closestDistance, то
				-- Проверяем, что часть не от другого персонажа
				local isInCharacter = false
				локальный родитель = obj.Parent
				пока родитель делает
					если родитель:FindFirstChild("Humanoid") тогда
						isInCharacter = true
						перерыв
					конец
					родитель = родитель.родитель
				конец
				
				если isInCharacter, то
					closestPart = obj
					closestDistance = distance
				конец
			конец
		конец
	конец
	
	return closestPart
конец

-- Когда инструмент берут в руки
grabTool.Equipped:Connect(function()
	-- Автоматически берем ближайшую часть при взятии инструмента в руки
	local targetPart = findPartNearCharacter()
	если targetPart тогда
		cleanupForces()
		
		currentPart = targetPart
		bodyPosition, bodyGyro = createBodyForces(currentPart)
		
		-- Немедленно установить позицию
		bodyPosition.Position = getFollowPosition()
		
		connection = RunService.Heartbeat:Connect(updatePartPosition)
		grabTool.TextureId = "rbxassetid://"
		print("Автоматически получена деталь: " .. targetPart.Name)
	еще
		print("При экипировке инструмента поблизости не обнаружено ни одной детали")
	конец
конец)

-- Когда инструмент убирают из рук
grabTool.Unequipped:Connect(function()
	-- Автоматически отпускаем часть при убирании инструмента.
	cleanupForces()
	print("Часть автоматически удаляется при снятии инструмента")
конец)

-- Очистка при смерти
player.CharacterAdded:Connect(function(character)
	cleanupForces()
конец)

-- Очистка при удалении инструмента
grabTool.Destroying:Connect(function()
	cleanupForces()
конец)

-- Помещаем инструмент в инвентарь
grabTool.Parent = backpack

print("Установлен инструмент захвата - автоматический захват/снятие захвата при экипировке/снятии экипировки")
