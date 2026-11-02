-- == Launcher leve para aparelho fraco (2GB) ==
-- Mantém todas as funções do original; só reduz o custo de rodar.

local URL   = "https://api.luarmor.net/files/v4/loaders/271f96ca25f29ae395be1c0f9e6f4ead.lua"
local CACHE = "luarmor_271f96c.lua"
local TTL   = 3600 -- segundos de validade do cache (use 86400 se quiser mais)

script_key = "SUA_KEY_AQUI"

-- 1) Modo performance: derruba o que mais pesa em GPU fraca
local function perf()
    pcall(function()
        local s = settings().Rendering
        s.QualityLevel    = Enum.QualityLevel.Level01
        s.TextureQuality  = Enum.TextureQuality.Low
        s.MeshPartDetail  = Enum.MeshPartDetail.Level01
    end)
    pcall(function()
        local L = game:GetService("Lighting")
        L.GlobalShadows  = false
        L.FogEnd         = 250
        L.ShadowSoftness = 0
        for _, v in ipairs(L:GetChildren()) do
            if v:IsA("PostEffect") then v.Enabled = false end
        end
    end)
    pcall(function()
        local T = workspace.Terrain
        T.WaterWaveSize, T.WaterWaveSpeed = 0, 0
    end)
end

-- 2) Cache local com validade: evita rebaixar o loader toda vez
local function lerCache()
    if not (isfile and readfile) then return nil end
    local ok, data = pcall(readfile, CACHE)
    if not ok or type(data) ~= "string" then return nil end
    local ts, body = data:match("^(%d+)\n(.*)$")
    if not ts or #body < 300 then return nil end
    if os.time() - tonumber(ts) > TTL then return nil end
    return body
end

local src = lerCache()
if not src then
    local ok, res = pcall(game.HttpGet, game, URL)
    if ok and type(res) == "string" then
        src = res
        if writefile then pcall(writefile, CACHE, os.time() .. "\n" .. res) end
    end
end

-- 3) Executa fora da thread principal
perf()
local chunk, err = loadstring(src or "")
if not chunk then
    src = game:HttpGet(URL)          -- cache corrompido/expirado: vai direto na rede
    chunk, err = loadstring(src)
end
assert(chunk, "Falha ao compilar o loader: " .. tostring(err))
task.spawn(chunk)# Op
