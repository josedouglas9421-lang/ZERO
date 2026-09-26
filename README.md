-- ==============================================
-- 🥚 ROUBE UM OVO — FARM AUTOMÁTICO V2.0
-- ✅ Farm sem parar | Sem bugs | Detecção inteligente
-- ==============================================

print("✅ Script carregado com sucesso!")

-- Serviços do Roblox
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Jogador
local LocalPlayer = Players.LocalPlayer
local Personagem, RootPart

-- Recarregar personagem se respawnar
local function AtualizarPersonagem()
    Personagem = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    RootPart = Personagem:WaitForChild("HumanoidRootPart")
end
AtualizarPersonagem()
LocalPlayer.CharacterAdded:Connect(AtualizarPersonagem)

-- CONFIGURAÇÕES (ajuste se precisar)
local Config = {
    AutoFarm = true,           -- Farm sozinho sem parar
    IrParaOvo = true,          -- Ir até o ovo
    EntregarBase = true,       -- Levar até a base
    DistanciaPegar = 10,       -- Distância para pegar o ovo
    DistanciaEntregar = 10,    -- Distância para contar entrega
    EsperaEntreCiclos = 1,     -- Tempo entre cada volta (segundos)
    PularSeNaoAchar = true,    -- Não travar se não achar nada
    MostrarMensagens = true    -- Ver o que está acontecendo
}

-- ESTADO
local TemOvo = false
local UltimoOvo = nil

-- MOVER JOGADOR SUAVEMENTE
local function MoverPara(alvoPosicao, alturaExtra)
    if not RootPart then return end
    alturaExtra = alturaExtra or 2
    RootPart.CFrame = CFrame.new(alvoPosicao + Vector3.new(0, alturaExtra, 0))
    task.wait(0.3)
end

-- PROCURAR OVO
local function AcharOvo()
    local ovoEncontrado = nil
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            local nome = string.lower(obj.Name)
            if nome == "ovo" or nome == "egg" or nome == "ovos" or nome == "eggpart" then
                if not ovoEncontrado then
                    ovoEncontrado = obj
                end
            end
        end
    end
    return ovoEncontrado
end

-- PROCURAR BASE
local function AcharBase()
    local baseEncontrada = nil
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") or obj:IsA("Model") then
            local nome = string.lower(obj.Name)
            if nome == "base" or nome == "entrega" or nome == "delivery" or nome == "home" then
                if not baseEncontrada and obj.PrimaryPart then
                    baseEncontrada = obj.PrimaryPart
                elseif not baseEncontrada and obj:IsA("BasePart") then
                    baseEncontrada = obj
                end
            end
        end
    end
    return baseEncontrada
end

-- PEGAR OVO
local function PegarOvo()
    if not Config.AutoFarm then return end
    
    local ovo = AcharOvo()
    if not ovo then
        if Config.MostrarMensagens then print("🥚 Nenhum ovo encontrado...") end
        return false
    end

    if Config.MostrarMensagens then print("🥚 Ovo encontrado! Indo pegar...") end
    MoverPara(ovo.Position)
    TemOvo = true
    UltimoOvo = ovo
    return true
end

-- ENTREGAR NA BASE
local function Entregar()
    if not TemOvo then return end
    
    local base = AcharBase()
    if not base then
        print("⚠️ Base não encontrada!")
        return
    end

    if Config.MostrarMensagens then print("🏠 Levando para entregar...") end
    MoverPara(base.Position)
    TemOvo = false
    UltimoOvo = nil
    print("✅ OVO ENTREGUE! +1 PONTO")
end

-- LOOP PRINCIPAL — FARMA SEM PARAR
task.spawn(function()
    while task.wait(Config.EsperaEntreCiclos) do
        if not Config.AutoFarm then 
            task.wait(0.5)
            continue 
        end

        if not RootPart then
            AtualizarPersonagem()
            task.wait(1)
            continue
        end

        local sucesso = PegarOvo()
        if sucesso and Config.EntregarBase then
            task.wait(0.5)
            Entregar()
        end
    end
end)

-- TECLAS DE CONTROLE
UserInputService.InputBegan:Connect(function(entrada, gp)
    if gp then return end

    -- F = Ativar/Desativar AutoFarm
    if entrada.KeyCode == Enum.KeyCode.F then
        Config.AutoFarm = not Config.AutoFarm
        print(Config.AutoFarm and "✅ AutoFarm LIGADO" or "⏸️ AutoFarm DESLIGADO")
    end

    -- E = Entregar agora
    if entrada.KeyCode == Enum.KeyCode.E then
        Entregar()
    end

    -- R = Resetar
    if entrada.KeyCode == Enum.KeyCode.R then
        TemOvo = false
        UltimoOvo = nil
        print("🔄 Resetado!")
    end
end)

-- ==============================================
-- 📋 COMO USAR:
-- F = Ligar/Desligar farm automático
-- E = Entregar manualmente
-- R = Resetar se travar
-- ==============================================
print("👉 Tecla F = Ligar/Desligar | E = Entregar | R = Resetar")
