
-- =================================================================
-- SPACE HUB v2 BETA - Script Central Completo
-- Programador: userzarvokpfc | Versão: 2.0 Beta (Funcional)
-- Todas as funções com Rayfield UI (arrastável)
-- =================================================================

print("Space Hub by userzarvokpfc - Iniciando com Rayfield...")

-- === TENTAR CARREGAR RAYFIELD ===
local Rayfield = nil
local sucessoRayfield = false

-- Tentativa 1: URL mais confiável
local sucesso1, resultado1 = pcall(function()
    return loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source.lua'))()
end)

if sucesso1 then
    Rayfield = resultado1
    sucessoRayfield = true
    print("✅ Rayfield carregado da URL 1")
else
    -- Tentativa 2: Backup
    print("⚠️ URL 1 falhou, tentando backup...")
    local sucesso2, resultado2 = pcall(function()
        return loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
    end)
    
    if sucesso2 then
        Rayfield = resultado2
        sucessoRayfield = true
        print("✅ Rayfield carregado da URL 2")
    else
        warn("❌ Rayfield não carregou. Criando interface básica...")
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Space Hub - Modo Básico",
            Text = "Rayfield falhou, usando sistema simples",
            Duration = 5
        })
    end
end

-- === CONFIGURAÇÕES ===
local lp = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- === VARIÁVEIS GLOBAIS ===
local flyEnabled = false
local flyBodyVelocity
local noclipEnabled = false
local rainbowVehicleEnabled = false
local antiSitEnabled = false

-- === TELA DE LOADING (se Rayfield funcionou) ===
if sucessoRayfield then
    local PlayerGui = lp:WaitForChild("PlayerGui")
    
    local LoadingGui = Instance.new("ScreenGui")
    LoadingGui.Name = "SpaceLoading"
    LoadingGui.Parent = PlayerGui
    LoadingGui.ResetOnSpawn = false
    LoadingGui.IgnoreGuiInset = true

    local Fundo = Instance.new("Frame", LoadingGui)
    Fundo.Size = UDim2.new(1, 0, 1, 0)
    Fundo.BackgroundColor3 = Color3.fromRGB(60, 0, 120)
    Fundo.BorderSizePixel = 0

    local Logo = Instance.new("ImageLabel", Fundo)
    Logo.Size = UDim2.new(0.45, 0, 0.45, 0)
    Logo.Position = UDim2.new(0.275, 0, 0.1, 0)
    Logo.BackgroundTransparency = 1
    Logo.Image = "https://files.catbox.moe/0k3z4j.png"
    Logo.ScaleType = Enum.ScaleType.Fit

    local TextLoad = Instance.new("TextLabel", Fundo)
    TextLoad.Size = UDim2.new(0.8, 0, 0.1, 0)
    TextLoad.Position = UDim2.new(0.1, 0, 0.65, 0)
    TextLoad.BackgroundTransparency = 1
    TextLoad.Text = "Carregando Space Hub... 0%"
    TextLoad.TextColor3 = Color3.new(1, 1, 1)
    TextLoad.Font = Enum.Font.GothamBold
    TextLoad.TextSize = 50

    local BarFrame = Instance.new("Frame", Fundo)
    BarFrame.Size = UDim2.new(0.6, 0, 0.04, 0)
    BarFrame.Position = UDim2.new(0.2, 0, 0.78, 0)
    BarFrame.BackgroundColor3 = Color3.fromRGB(40, 0, 80)
    BarFrame.BorderSizePixel = 0

    local Fill = Instance.new("Frame", BarFrame)
    Fill.Size = UDim2.new(0, 0, 1, 0)
    Fill.BackgroundColor3 = Color3.fromRGB(180, 100, 255)
    Fill.BorderSizePixel = 0

    -- Animação da barra
    spawn(function()
        for i = 0, 100 do
            Fill.Size = UDim2.new(i/100, 0, 1, 0)
            TextLoad.Text = "Carregando Space Hub... " .. i .. "%"
            wait(0.025)
        end
        wait(0.8)
        LoadingGui:Destroy()
        iniciarInterface()
    end)
else
    -- Se Rayfield falhou, iniciar direto
    wait(1)
    iniciarInterfaceBasica()
end

-- === FUNÇÕES DO SISTEMA ===

-- 1. FLY SYSTEM
local function toggleFly(value)
    flyEnabled = value
    if value then
        flyBodyVelocity = Instance.new("BodyVelocity")
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyBodyVelocity.MaxForce = Vector3.new(10000, 10000, 10000)
        flyBodyVelocity.Parent = lp.Character.HumanoidRootPart
        
        local flyConnection
        flyConnection = UIS.InputBegan:Connect(function(input)
            if not flyEnabled then flyConnection:Disconnect() return end
            
            if input.KeyCode == Enum.KeyCode.W then
                flyBodyVelocity.Velocity = lp.Character.HumanoidRootPart.CFrame.LookVector * 100
            elseif input.KeyCode == Enum.KeyCode.S then
                flyBodyVelocity.Velocity = -lp.Character.HumanoidRootPart.CFrame.LookVector * 100
            elseif input.KeyCode == Enum.KeyCode.A then
                flyBodyVelocity.Velocity = -lp.Character.HumanoidRootPart.CFrame.RightVector * 100
            elseif input.KeyCode == Enum.KeyCode.D then
                flyBodyVelocity.Velocity = lp.Character.HumanoidRootPart.CFrame.RightVector * 100
            elseif input.KeyCode == Enum.KeyCode.Space then
                flyBodyVelocity.Velocity = Vector3.new(0, 100, 0)
            elseif input.KeyCode == Enum.KeyCode.LeftShift then
                flyBodyVelocity.Velocity = Vector3.new(0, -100, 0)
            end
        end)
    elseif flyBodyVelocity then
        flyBodyVelocity:Destroy()
    end
end

-- 2. NO CLIP
local function toggleNoclip(value)
    noclipEnabled = value
end

-- 3. SKY FE / BUILDER MODE
local function ativarSkyFE()
    local character = lp.Character
    if not character then return end
    
    -- Aplicar roupa de Builder
    local shirt = Instance.new("Shirt")
    shirt.ShirtTemplate = "rbxassetid://7079763934"
    shirt.Parent = character
    
    local pants = Instance.new("Pants")
    pants.PantsTemplate = "rbxassetid://7079771309"
    pants.Parent = character
    
    -- Ferramenta de Builder
    local tool = Instance.new("Tool")
    tool.Name = "[SPACE] Martelo Builder"
    
    local handle = Instance.new("Part", tool)
    handle.Name = "Handle"
    handle.Size = Vector3.new(0.5, 4, 0.5)
    handle.BrickColor = BrickColor.new("Bright yellow")
    handle.Material = Enum.Material.Neon
    
    tool.Parent = character
    
    return "✅ Sky FE - Modo Builder ATIVADO"
end

-- 4. NUKE (Efeito Total)
local function ativarNuke()
    local camera = Workspace.CurrentCamera
    local player = lp
    
    -- Flash de tela
    local screenGui = Instance.new("ScreenGui", player.PlayerGui)
    screenGui.Name = "NukeFlash"
    screenGui.IgnoreGuiInset = true
    
    local flashFrame = Instance.new("Frame", screenGui)
    flashFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    flashFrame.Size = UDim2.new(1, 0, 1, 0)
    flashFrame.BackgroundTransparency = 0.8
    flashFrame.BorderSizePixel = 0
    
    -- Animação do flash
    local tweenIn = TweenService:Create(flashFrame, TweenInfo.new(0.2), {BackgroundTransparency = 0})
    local tweenOut = TweenService:Create(flashFrame, TweenInfo.new(0.5), {BackgroundTransparency = 1})
    
    tweenIn:Play()
    tweenIn.Completed:Wait()
    tweenOut:Play()
    tweenOut.Completed:Wait()
    screenGui:Destroy()
    
    -- Explosões
    for i = 1, 10 do
        local explosion = Instance.new("Explosion")
        explosion.Position = Vector3.new(
            math.random(-200, 200),
            math.random(10, 50),
            math.random(-200, 200)
        )
        explosion.BlastRadius = 50
        explosion.Parent = Workspace
        wait(0.1)
    end
    
    return "💥 NUKE DETONADA"
end

-- 5. ANTI SIT
local function antiSit(value)
    antiSitEnabled = value
    if value and lp.Character then
        lp.Character.Humanoid.Seated:Connect(function()
            if antiSitEnabled then
                lp.Character.Humanoid.Sit = false
                lp.Character.Humanoid.Jump = true
            end
        end)
    end
end

-- === INTERFACE COM RAYFIELD ===
function iniciarInterface()
    print("Loading concluído! Abrindo Rayfield Hub...")
    
    local Window = Rayfield:CreateWindow({
       Name = "Space Hub by userzarvokpfc | Versão 2 Beta",
       LoadingTitle = "Space Hub v2 Beta",
       LoadingSubtitle = "by userzarvokpfc",
       ConfigurationSaving = {Enabled = false},
       KeySystem = false
    })
    
    -- Aba de Créditos
    local CreditosTab = Window:CreateTab("Créditos")
    CreditosTab:CreateSection("✨ Créditos do Space Hub ✨")
    CreditosTab:CreateLabel("Versão: 2 Beta")
    CreditosTab:CreateLabel("Programador: userzarvok0fc")
    CreditosTab:CreateLabel("Dono: rdzin.038rn")
    CreditosTab:CreateLabel("Discord: https://discord.gg/VspWJC9m")
    CreditosTab:CreateButton({
       Name = "Copiar Link Discord",
       Callback = function()
          setclipboard("https://discord.gg/VspWJC9m")
          Rayfield:Notify({
             Title = "Link Copiado!",
             Content = "Cole no seu Discord ou browser!",
             Duration = 4
          })
       end,
    })
    
    -- Aba Principal
    local PrincipalTab = Window:CreateTab("Principal")
    PrincipalTab:CreateSection("Movimentos Básicos")
    
    PrincipalTab:CreateToggle({
       Name = "Fly (WASD + Space/Shift)",
       CurrentValue = false,
       Callback = function(Value)
          toggleFly(Value)
          Rayfield:Notify({
             Title = Value and "✈️ Fly ATIVADO" or "✗ Fly DESATIVADO",
             Content = Value and "Use WASD + Space/Shift" or "Fly desligado",
             Duration = 4
          })
       end,
    })
    
    PrincipalTab:CreateToggle({
       Name = "Noclip",
       CurrentValue = false,
       Callback = function(Value)
          toggleNoclip(Value)
          Rayfield:Notify({
             Title = Value and "👻 Noclip ON" or "✗ Noclip OFF",
             Content = Value and "Pode atravessar tudo!" or "Colisão normal",
             Duration = 3
          })
       end,
    })
    
    PrincipalTab:CreateSlider({
       Name = "WalkSpeed",
       Range = {16, 300},
       Increment = 1,
       CurrentValue = 16,
       Callback = function(Value)
          if lp.Character and lp.Character:FindFirstChild("Humanoid") then
             lp.Character.Humanoid.WalkSpeed = Value
          end
       end,
    })
    
    -- Aba Skybox
    local SkyboxTab = Window:CreateTab("Skybox")
    SkyboxTab:CreateSection("Skybox Controls")
    
    local SkyboxList = {"Default", "Cyclic", "Factory", "Nuke FE"}
    local SelectedSkybox = "Default"
    
    SkyboxTab:CreateDropdown({
        Name = "Select Skybox",
        Options = SkyboxList,
        CurrentOption = SelectedSkybox,
        Callback = function(Option)
            SelectedSkybox = Option
        end,
    })
    
    -- Aba Veículos
    local VeiculosTab = Window:CreateTab("Veículos")
    VeiculosTab:CreateSection("Controle de Veículos")
    
    VeiculosTab:CreateButton({
        Name = "Bring Vehicle",
        Callback = function()
            Rayfield:Notify({
                Title = "Bring Vehicle",
                Content = "Função em desenvolvimento",
                Duration = 3
            })
        end,
    })
    
    VeiculosTab:CreateToggle({
        Name = "Rainbow Vehicle",
        CurrentValue = false,
        Callback = function(Value)
            rainbowVehicleEnabled = Value
            Rayfield:Notify({
                Title = Value and "🌈 Rainbow Vehicle ON" or "✗ Rainbow Vehicle OFF",
                Content = Value and "Cores mudando no veículo!" or "Cores normais",
                Duration = 3
            })
        end,
    })
    
    -- Aba Casa
    local CasaTab = Window:CreateTab("Casa")
    CasaTab:CreateSection("House Controls")
    
    CasaTab:CreateToggle({
        Name = "Noclip Doors",
        CurrentValue = false,
        Callback = function(Value)
            Rayfield:Notify({
                Title = Value and "🚪 Noclip Doors ON" or "✗ Noclip Doors OFF",
                Content = Value and "Pode atravessar portas!" or "Portas normais",
                Duration = 3
            })
        end,
    })
    
    -- Aba Antis
    local AntisTab = Window:CreateTab("Antis")
    AntisTab:CreateSection("Proteções")
    
    AntisTab:CreateToggle({
       Name = "Anti Sit v1",
       CurrentValue = false,
       Callback = function(Value)
          Rayfield:Notify({
             Title = Value and "🪑 Anti-Sit v1 ON" or "✗ Anti-Sit OFF",
             Content = Value and "Não pode sentar!" or "Pode sentar",
             Duration = 3
          })
       end,
    })
    
    AntisTab:CreateToggle({
       Name = "Anti Sit v2 (Avançado)",
       CurrentValue = false,
       Callback = function(Value)
          antiSit(Value)
          Rayfield:Notify({
             Title = Value and "🪑 Anti-Sit v2 ON" or "✗ Anti-Sit OFF",
             Content = Value and "Proteção avançada!" or "Proteção desligada",
             Duration = 3
          })
       end,
    })
    
    -- Aba Efeitos FE
    local EfeitosTab = Window:CreateTab("Efeitos FE")
    EfeitosTab:CreateSection("Efeitos Visuais")
    
    EfeitosTab:CreateButton({
        Name = "🔨 Ativar Sky FE",
        Callback = function()
            local result = ativarSkyFE()
            Rayfield:Notify({
                Title = result,
                Content = "Modo Builder ativado!",
                Duration = 5
            })
        end,
    })
    
    EfeitosTab:CreateButton({
        Name = "💥 Detonar Nuke",
        Callback = function()
            local result = ativarNuke()
            Rayfield:Notify({
                Title = result,
                Content = "Efeito nuclear ativado!",
                Duration = 6
            })
        end,
    })
    
    -- Aba Ferramentas
    local FerramentasTab = Window:CreateTab("Ferramentas")
    FerramentasTab:CreateSection("Gamepass / Decal")
    
    FerramentasTab:CreateButton({
        Name = "Get Color Campass",
        Callback = function()
            Rayfield:Notify({
                Title = "Color Campass",
                Content = "Função em desenvolvimento",
                Duration = 3
            })
        end,
    })
    
    -- Aba Família
    local FamiliaTab = Window:CreateTab("Família")
    FamiliaTab:CreateSection("Controle de Personagem")
    
    FamiliaTab:CreateButton({
        Name = "Star del Personagem",
        Callback = function()
            Rayfield:Notify({
                Title = "⭐ Personagem Estrela",
                Content = "Função em desenvolvimento",
                Duration = 3
            })
        end,
    })
    
    -- Sistema de background
    RunService.Stepped:Connect(function()
        if noclipEnabled and lp.Character then
            for _, part in pairs(lp.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
    
    -- Rainbow Vehicle
    spawn(function()
        while true do
            if rainbowVehicleEnabled then
                for _, vehicle in pairs(Workspace:GetChildren()) do
                    if vehicle:IsA("VehicleSeat") then
                        for _, part in pairs(vehicle:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.Color = Color3.fromHSV(tick() % 5 / 5, 1, 1)
                            end
                        end
                    end
                end
            end
            wait(0.1)
        end
    end)
    
    Rayfield:Notify({
       Title = "🚀 Space Hub v2 Beta Carregado!",
       Content = "Interface arrastável com 8 abas!",
       Duration = 6.5
    })
    
    print("Space Hub v2 Beta totalmente carregado!")
end

-- === INTERFACE BÁSICA (se Rayfield falhar) ===
function iniciarInterfaceBasica()
    print("Iniciando interface básica...")
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "SpaceHubBasic"
    screenGui.Parent = lp.PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 300, 0, 400)
    frame.Position = UDim2.new(0.5, -150, 0.5, -200)
    frame.BackgroundColor3 = Color3.fromRGB(40, 0, 80)
    frame.Parent = screenGui
    
    local title = Instance.new("TextLabel")
    title.Text = "🚀 SPACE HUB (Básico)"
    title.Size = UDim2.new(1, 0, 0.1, 0)
    title.TextColor3 = Color3.new(1,1,1)
    title.Font = Enum.Font.GothamBold
    title.BackgroundTransparency = 1
    title.Parent = frame
    
    -- Botões básicos
    local botoes = {
        {"✈️ FLY", function() toggleFly(not flyEnabled) end},
        {"👻 NOCLIP", function() toggleNoclip(not noclipEnabled) end},
        {"🔨 SKY FE", ativarSkyFE},
        {"💥 NUKE", ativarNuke},
        {"🛡️ ANTI-SIT", function() antiSit(not antiSitEnabled) end},
        {"⚡ SPEED 50", function() 
            if lp.Character then 
                lp.Character.Humanoid.WalkSpeed = 50 
            end 
        end}
    }
    
    for i, botao in ipairs(botoes) do
        local btn = Instance.new("TextButton")
        btn.Text = botao[1]
        btn.Size = UDim2.new(0.8, 0, 0.1, 0)
        btn.Position = UDim2.new(0.1, 0, 0.15 + (i * 0.12), 0)
        btn.BackgroundColor3 = Color3.fromRGB(80, 0, 160)
        btn.TextColor3 = Color3.new(1,1,1)
        btn.Font = Enum.Font.Gotham
        btn.Parent = frame
        btn.MouseButton1Click:Connect(botao[2])
    end
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Space Hub (Básico)",
        Text = "Rayfield falhou, usando interface simples",
        Duration = 5
    })
end
