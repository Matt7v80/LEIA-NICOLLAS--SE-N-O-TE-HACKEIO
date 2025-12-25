--[[
    SERVER-SIDE ADMIN - TRITE SPIRIT
    Local: ServerScriptService > Script "ServerAdmin"
]]

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- Conexão com a pasta criada no Passo 1
local RemotesFolder = ReplicatedStorage:WaitForChild("TriteRemotes")
local AdminRemote = RemotesFolder:WaitForChild("AdminAction")
local ClientEffectRemote = RemotesFolder:WaitForChild("ClientEffect")

-- === LISTA DE ADMINS ===
-- Coloque SEU ID aqui. (Para pegar seu ID, vá no seu perfil do Roblox e copie o número na URL)
local Admins = {
    [3650838602] = true, -- Substitua pelo seu ID
    [987654321] = true  -- ID de um amigo (opcional)
}

local function IsAdmin(player)
    if Admins[player.UserId] or player.UserId == game.CreatorId then
        return true
    end
    return false
end

-- === COMANDOS ===
local Commands = {}

Commands["kick"] = function(target)
    target:Kick("Você foi expulso pelo Sistema Trite Spirit.")
end

Commands["ban"] = function(target)
    -- Num jogo real você usaria DataStore. Aqui é um kick permanente na sessão.
    target:Kick("BANIDO! (Trite Spirit Admin)")
end

Commands["punishment"] = function(target)
    -- Manda efeito visual APENAS para a vítima
    ClientEffectRemote:FireClient(target, "Glitch")
end

Commands["jumpscare"] = function(target)
    -- Manda susto APENAS para a vítima
    ClientEffectRemote:FireClient(target, "Jumpscare")
end

Commands["troll"] = function(target)
    if target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        target.Character.HumanoidRootPart.CFrame = CFrame.new(0, 50000, 0) -- Joga pro céu
    end
end

Commands["f3x"] = function(player)
    -- Para isso funcionar, você precisa ter uma Tool chamada "Building Tools" no ServerStorage
    -- Ou usar este código para dar uma tool temporária:
    local backpack = player:WaitForChild("Backpack")
    -- (Nota: Inserir tools via script requer que a tool exista no jogo)
end

-- === RECEBER COMANDOS DO PAINEL ===
AdminRemote.OnServerEvent:Connect(function(player, action, targetName)
    -- 1. Verifica se quem mandou o comando é Admin
    if not IsAdmin(player) then
        warn(player.Name .. " tentou usar admin sem permissão!")
        return
    end

    -- 2. Acha o jogador alvo (Target)
    local targetPlayer = nil
    if targetName then
        for _, p in pairs(Players:GetPlayers()) do
            if string.sub(string.lower(p.Name), 1, string.len(targetName)) == string.lower(targetName) or
               string.sub(string.lower(p.DisplayName), 1, string.len(targetName)) == string.lower(targetName) then
                targetPlayer = p
                break
            end
        end
    end

    -- 3. Executa
    if action == "f3x" then
        Commands["f3x"](player) -- F3X é para quem pediu
    elseif targetPlayer and Commands[action] then
        Commands[action](targetPlayer)
        print("Comando "..action.." executado em "..targetPlayer.Name)
    end
end)
