### [Preview - Youtube](https://youtu.be/83jV2z_Ime4)

### Add this function to qb-core/client/functions.lua
```
function QBCore.Functions.CreateBlip(coords, sprite, scale, color, text)
	local blip = AddBlipForCoord(coords)
	SetBlipSprite(blip, sprite)
	SetBlipDisplay(blip, 4)
	SetBlipScale(blip, scale)
	SetBlipAsShortRange(blip, true)
	SetBlipColour(blip, color)
	SetBlipAlpha(Blip, 0.7)
	BeginTextCommandSetBlipName('STRING')
	AddTextComponentString("<font face='Oswald'>"..text.."</font>")
	EndTextCommandSetBlipName(blip)
end

QBCore.Functions.GetVehiclesInArea = function(coords, maxDistance) return EnumerateEntitiesWithinDistance(QBCore.Functions.GetVehicles(), false, coords, maxDistance) end
QBCore.Functions.IsSpawnPointClear = function(coords, maxDistance) return #QBCore.Functions.GetVehiclesInArea(coords, maxDistance) == 0 end
function EnumerateEntitiesWithinDistance(entities, isPlayerEntities, coords, maxDistance)
	local nearbyEntities = {}

	if coords then
		coords = vector3(coords.x, coords.y, coords.z)
	else
		local playerPed = PlayerPedId()
		coords = GetEntityCoords(playerPed)
	end

	for k,entity in pairs(entities) do
		local distance = #(coords - GetEntityCoords(entity))

		if distance <= maxDistance then
			table.insert(nearbyEntities, isPlayerEntities and k or entity)
		end
	end

	return nearbyEntities
end
```
### Edit in qb-phone\server\main.lua:
```
QBCore.Functions.CreateCallback('qb-phone:server:GetGarageVehicles', function(source, cb)
    local Player = QBCore.Functions.GetPlayer(source)
    local Vehicles = {}

    local result = exports.oxmysql:executeSync('SELECT * FROM player_vehicles WHERE citizenid = ?',
        {Player.PlayerData.citizenid})
    if result[1] ~= nil then
        for k, v in pairs(result) do
            local VehicleData = QBCore.Shared.Vehicles[v.vehicle]

            local VehicleGarage = "None"
            if v.garage ~= nil then
                if Garages[v.garage] ~= nil then
                    VehicleGarage = Garages[v.garage]["label"]
                --elseif GangGarages[v.garage] ~= nil then
                    --VehicleGarage = GangGarages[v.garage]["label"]
                --elseif JobGarages[v.garage] ~= nil then
                    --VehicleGarage = JobGarages[v.garage]["label"]
                end
            end

            local VehicleState = "In"
            if v.state == 0 then
                VehicleState = "Out"
            elseif v.state == 2 then
                VehicleState = "Impounded"
            end

            local vehdata = {}

            if VehicleData["brand"] ~= nil then
                vehdata = {
                    fullname = VehicleData["brand"] .. " " .. VehicleData["name"],
                    brand = VehicleData["brand"],
                    model = VehicleData["name"],
                    plate = v.plate,
                    garage = VehicleGarage,
                    state = VehicleState,
                    fuel = v.fuel,
                    engine = v.engine,
                    body = v.body
                }
            else
                vehdata = {
                    fullname = VehicleData["name"],
                    brand = VehicleData["name"],
                    model = VehicleData["name"],
                    plate = v.plate,
                    garage = VehicleGarage,
                    state = VehicleState,
                    fuel = v.fuel,
                    engine = v.engine,
                    body = v.body
                }
            end
            Vehicles[#Vehicles+1] = vehdata
        end
        cb(Vehicles)
    else
        cb(nil)
    end
end)
```
### Add event to F1 menu:
Open Garage:
```
'Garage:openGarage'
```
Store Vehicle:
```
'Garage:storeVehicle'
```