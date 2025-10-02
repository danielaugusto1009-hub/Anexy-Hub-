--// Serviços (mantido)
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local playerGui = LocalPlayer:WaitForChild("PlayerGui")

--// ScreenGui (visual preservado) (mantido)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "Anexy_CustomUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = playerGui

---------------------------------------------------------------------------
--// Estados globais (mantido)
---------------------------------------------------------------------------
local espOn = false
local superOn = false
local savedPosition = nil -- posição salva pelo Collision
local isTeleporting = false
local canCollideOriginal = {} --[[ Tabela para armazenar valores originais de CanCollide ]]

-- Variáveis "falsas" para confundir análise
local fakeVariable1 = math.random(1, 100)
local fakeVariable2 = "Inofensivo"

-- Função de execução condicional "isca"
local function maybePrintMessage()
    if math.random() < 0.05 then -- 5% de chance
        print("Script executando uma ação inofensiva...")
    end
end

---------------------------------------------------------------------------
--- [ANTI-CHEAT ADDITION] - Variáveis e Funções de Obfuscação
---------------------------------------------------------------------------
-- Guarda o código fonte original para checagem de anti-tamper
local scriptSource = script.Source

-- Função para "ofuscar" valores numéricos sem alterar seu resultado
local function obscure(value)
    local key = math.random(1, 100)
    return value + key - key
end

-- Tabelas para variação de strings
local espText = {"🔴 ESP [OFF]", "🔴 ESP [ON]"}
local superHumanText = {"⚡ Super Human [OFF]", "⚡ Super Human [ON]"}
local collisionText = {
    original = "📍 Collision [Set]",
    saved = "📍 Collision [Saved]",
}
local stealText = {
    original = "💎 Instant Steal (Ultra Safe)",
    saveFirst = "💎 Salve a posição primeiro!"
}
local keyValidationText = {
    incorrect = "❌ Key incorreta!",
    copied = "📋 Link copiado!"
}

-- Variáveis para monitoramento de performance (com isca)
local lastMemoryUsage = collectgarbage("count")
local lastTime = os.clock()

---------------------------------------------------------------------------
--- [ANTI-CHEAT ADDITION] - Variáveis para Correção de Posição
---------------------------------------------------------------------------
local velocidadeMaxima = 20 -- studs por segundo (pode ajustar)
local distanciaMaximaTeleporte = 50 -- studs (distância máxima para teleporte "válido", pode ajustar)
local posicaoAnteriorLocalPlayer = nil -- Para o LocalPlayer

---------------------------------------------------------------------------
--- [ANTI-CHEAT ADDITION] - Funções para Correção de Posição (Client-side)
---------------------------------------------------------------------------
local function detectarAtravessamento(jogador, posicaoInicio, posicaoFim)
    local personagem = jogador.Character
    if not personagem then return false end

    -- Ignora o próprio personagem do jogador e seus descendentes no raycast
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    local raycastParams.FilterDescendantsInstances = {personagem}
    raycastParams.IgnoreWater = true -- Geralmente ignoramos água

    local direction = (posicaoFim - posicaoInicio).Unit
    local distance = (posicaoFim - posicaoInicio).Magnitude

    -- Se a distância for muito pequena, não há atravessamento significativo
    if distance < 0.1 then return false end

    local raycastResult = Workspace:Raycast(posicaoInicio, direction * distance, raycastParams)

    if raycastResult then
        -- Se o raycast atingiu algo que não seja o próprio personagem, pode ser atravessamento
        -- Uma verificação mais robusta incluiria verificar se é um "mapa" e não um objeto pequeno
        return true
    else
        return false
    end
end

local function limitarVelocidade(posicaoInicio, posicaoFim, deltaTime)
    local distancia = (posicaoFim - posicaoInicio).Magnitude
    local maxDistancia = velocidadeMaxima * deltaTime

    if distancia > maxDistancia then
        local direction = (posicaoFim - posicaoInicio).Unit
        return posicaoInicio + direction * maxDistancia
    else
        return posicaoFim
    end
end

local function detectarTeletransporte(posicaoInicio, posicaoFim)
    local distancia = (posicaoFim - posicaoInicio).Magnitude
    if distancia > distanciaMaximaTeleporte then
        return true -- Detectou um teletransporte suspeito
    else
        return false
    end
end

---------------------------------------------------------------------------
--// Funções ESP (mantive igual) (mantido)
---------------------------------------------------------------------------
local function createHighlightForCharacter(char)
    if not char or not char.Parent then
        return
    end
    if char:FindFirstChild("ESP_Highlight") then
        return
    end
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.FillColor = Color3.fromRGB(255,0,0)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = char
    maybePrintMessage() -- Execução condicional
end

local function removeHighlightFromCharacter(char)
    if not char then
        return
    end
    local h = char:FindFirstChild("ESP_Highlight")
    if h then
        h:Destroy()
    end
    maybePrintMessage() -- Execução condicional
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        if espOn and plr ~= LocalPlayer then
            char:WaitForChild("HumanoidRootPart", 5)
            createHighlightForCharacter(char)
        end
    end)
    plr.CharacterRemoving:Connect(function(char)
        removeHighlightFromCharacter(char)
    end)
end)
# Anexy-Hub-
Script para roblox
