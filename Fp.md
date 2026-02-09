-- 🥔 FISCH SMART POTATO MODE V2

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Terrain = workspace:FindFirstChildOfClass("Terrain")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()

-- ===== CONFIG =====
local DISTANCIA = 250 -- só otimiza perto do player
local PROCESSO_POR_FRAME = 80 -- evita travar CPU

-- ===== QUALIDADE MINIMA =====
pcall(function()
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
end)

-- ===== ILUMINAÇÃO =====
Lighting.GlobalShadows = false
Lighting.Brightness = 1
Lighting.EnvironmentDiffuseScale = 0
Lighting.EnvironmentSpecularScale = 0
Lighting.FogEnd = 9e9
Lighting.ClockTime = 14

for _,v in pairs(Lighting:GetChildren()) do
    if v:IsA("PostEffect") then
        v.Enabled = false
    end
end

-- ===== ÁGUA OTIMIZADA =====
if Terrain then
    Terrain.WaterWaveSize = 0
    Terrain.WaterWaveSpeed = 0
    Terrain.WaterReflectance = 0
    Terrain.WaterTransparency = 0.5
end

-- ===== FUNÇÃO DE OTIMIZAÇÃO =====
local function otimizar(obj)

    -- Partículas
    if obj:IsA("ParticleEmitter")
    or obj:IsA("Trail")
    or obj:IsA("Smoke")
    or obj:IsA("Fire")
    or obj:IsA("Sparkles") then
        obj.Enabled = false
        return
    end

    -- Luzes
    if obj:IsA("PointLight")
    or obj:IsA("SpotLight")
    or obj:IsA("SurfaceLight") then
        obj.Enabled = false
        return
    end

    -- Texturas
    if obj:IsA("Decal") or obj:IsA("Texture") then
        obj.Transparency = 1
        return
    end

    -- Partes físicas
    if obj:IsA("BasePart") then
        obj.Material = Enum.Material.SmoothPlastic
        obj.Reflectance = 0
        obj.CastShadow = false
    end
end

-- ===== PROCESSAMENTO INTELIGENTE =====
task.spawn(function()

    local objetos = workspace:GetDescendants()
    local index = 1

    while index <= #objetos do
        
        local root = Character:FindFirstChild("HumanoidRootPart")
        if root then
            
            for i = 1, PROCESSO_POR_FRAME do
                local obj = objetos[index]
                if not obj then break end

                if obj:IsA("BasePart") then
                    if (obj.Position - root.Position).Magnitude <= DISTANCIA then
                        otimizar(obj)
                    end
                else
                    otimizar(obj)
                end

                index += 1
            end
        end

        task.wait() -- evita travar celular
    end

end)

-- ===== OTIMIZA NOVOS OBJETOS =====
workspace.DescendantAdded:Connect(function(obj)
    task.defer(function()
        otimizar(obj)
    end)
end)

print("🥔 SMART POTATO MODE V2 ATIVADO")
