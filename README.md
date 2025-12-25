
--[[
    CLIENT-SIDE PAINEL - TRITE SPIRIT
    Local: StarterGui > AdminPanelUI > LocalScript
]]

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- CONEXÃO COM O SERVIDOR
local Remotes = ReplicatedStorage:WaitForChild("TriteRemotes")
local AdminRemote = Remotes:WaitForChild("AdminAction")
local ClientEffectRemote = Remotes:WaitForChild("ClientEffect")

-- CONFIGURAÇÃO VISUAL
local LOADING_IMAGE = "rbxassetid://119608493674167" -- COLOQUE O ID DA SUA IMAGEM AQUI
local LOADING_AUDIO = "rbxassetid://112214814544629"

-- CRIAR A UI (GUI) VIA SCRIPT
local ScreenGui = script.Parent

-- 1. TELA DE CARREGAMENTO
local LoadingFrame = Instance.new("Frame")
LoadingFrame.Size = UDim2.new(1, 0, 1, 0)
LoadingFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
LoadingFrame.Parent = ScreenGui
LoadingFrame.ZIndex = 10

local Img = Instance.new("ImageLabel")
Img.Size = UDim2.new(0, 200, 0, 200)
Img.Position = UDim2.new(0.5, -100, 0.4, -100)
Img.Image = LOADING_IMAGE
Img.BackgroundTransparency = 1
Img.Parent = LoadingFrame

local TipText = Instance.new("TextLabel")
TipText.Size = UDim2.new(1, 0, 0, 50)
TipText.Position = UDim2.new(0, 0, 0.8, 0)
TipText.BackgroundTransparency = 1
TipText.TextColor3 = Color3.fromRGB(255, 255, 255)
TipText.Font = Enum.Font.Gotham
TipText.TextSize = 18
TipText.Text = "Carregando Painel..."
TipText.Parent = LoadingFrame

local Audio = Instance.new("Sound")
Audio.SoundId = LOADING_AUDIO
Audio.Parent = ScreenGui
Audio:Play()

-- Animação de saída do Loading
task.delay(4, function()
    TweenService:Create(LoadingFrame, TweenInfo.new(1), {BackgroundTransparency = 1}):Play()
    TweenService:Create(Img, TweenInfo.new(1), {ImageTransparency = 1}):Play()
    TweenService:Create(TipText, TweenInfo.new(1), {TextTransparency = 1}):Play()
    task.wait(1)
    LoadingFrame.Visible = false
    Audio:Stop()
end)

-- 2. O PAINEL PRINCIPAL
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 300)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.Visible = false
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)

-- Botão para fechar/abrir (Mobile)
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 10, 0.5, -25)
ToggleBtn.Text = "MENU"
ToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 170, 0)
ToggleBtn.Parent = ScreenGui
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local function ToggleUI()
    MainFrame.Visible = not MainFrame.Visible
end
ToggleBtn.MouseButton1Click:Connect(ToggleUI)

UserInputService.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Enum.KeyCode.F then
        ToggleUI()
    end
end)

-- 3. INTERFACE INTERNA (Abas e Botões)
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 120, 1, 0)
Sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
Sidebar.Parent = MainFrame
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 8)

local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -130, 1, 0)
Content.Position = UDim2.new(0, 130, 0, 0)
Content.BackgroundTransparency = 1
Content.Parent = MainFrame

-- Input de Target
local TargetBox = Instance.new("TextBox")
TargetBox.Size = UDim2.new(1, 0, 0, 30)
TargetBox.PlaceholderText = "Nome do Jogador"
TargetBox.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
TargetBox.TextColor3 = Color3.fromRGB(255, 255, 255)
TargetBox.Parent = Content

-- Função para criar botões de Admin
local layout = Instance.new("UIListLayout")
layout.Parent = Content
layout.Padding = UDim.new(0, 5)

local function AddButton(text, command)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Parent = Content
    
    btn.MouseButton1Click:Connect(function()
        if command == "f3x" then
             AdminRemote:FireServer("f3x")
        else
             -- Envia o comando e o nome escrito na caixa para o servidor
             AdminRemote:FireServer(command, TargetBox.Text)
        end
    end)
end

-- ADICIONANDO OS BOTÕES
AddButton("Kick (Expulsar)", "kick")
AddButton("Ban (Banir)", "ban")
AddButton("Punishment (Glitch)", "punishment")
AddButton("Jumpscare (Susto)", "jumpscare")
AddButton("Troll (Jogar Longe)", "troll")
AddButton("Pegar BTools F3X", "f3x")

-- 4. OUVIR EFEITOS DO SERVIDOR (Se eu for a vítima)
ClientEffectRemote.OnClientEvent:Connect(function(effectType)
    if effectType == "Jumpscare" then
        local scare = Instance.new("ImageLabel")
        scare.Size = UDim2.new(1,0,1,0)
        scare.Image = LOADING_IMAGE
        scare.Parent = ScreenGui
        task.wait(2)
        scare:Destroy()
        
    elseif effectType == "Glitch" then
        local cc = Instance.new("ColorCorrectionEffect")
        cc.TintColor = Color3.new(1, 0, 0)
        cc.Contrast = 2
        cc.Parent = game.Lighting
        task.wait(5)
        cc:Destroy()
    end
end)
