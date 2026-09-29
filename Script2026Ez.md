local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Lighting Controller",
   LoadingTitle = "Memuat GUI...",
   LoadingSubtitle = "Dark Theme",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false
})

local MainTab = Window:CreateTab("Utama", 4483362458)
local Section = MainTab:CreateSection("Pengaturan Waktu & Lingkungan")

local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local timeLoop = nil
local fogLoop = nil
local skyLoop = nil

-- Fungsi universal untuk ngunci waktu
local function setTimeLock(targetTime, state)
   if timeLoop then
      timeLoop:Disconnect()
      timeLoop = nil
   end
   
   if state then
      timeLoop = RunService.Heartbeat:Connect(function()
         Lighting.ClockTime = targetTime
      end)
   end
end

-- Toggle 1: Kunci Siang
MainTab:CreateToggle({
   Name = "Kunci Siang",
   CurrentValue = false,
   Flag = "LockDay",
   Callback = function(Value)
      if Value then
         setTimeLock(12, true)
         Rayfield:Notify({ Title = "Mode Siang", Content = "Waktu dikunci jadi Siang.", Duration = 2 })
      else
         setTimeLock(12, false)
         Rayfield:Notify({ Title = "Mode Siang", Content = "Kunci waktu dimatikan.", Duration = 2 })
      end
   end,
})

-- Toggle 2: Kunci Malam
MainTab:CreateToggle({
   Name = "Kunci Malam",
   CurrentValue = false,
   Flag = "LockNight",
   Callback = function(Value)
      if Value then
         setTimeLock(0, true)
         Rayfield:Notify({ Title = "Mode Malam", Content = "Waktu dikunci jadi Malam.", Duration = 2 })
      else
         setTimeLock(0, false)
         Rayfield:Notify({ Title = "Mode Malam", Content = "Kunci waktu dimatikan.", Duration = 2 })
      end
   end,
})

-- Toggle 3: No Fog (Anti Kabut)
MainTab:CreateToggle({
   Name = "No Fog (Anti Kabut)",
   CurrentValue = false,
   Flag = "NoFog",
   Callback = function(Value)
      if Value then
         fogLoop = RunService.Heartbeat:Connect(function()
            Lighting.FogEnd = 1000000
            for _, child in pairs(Lighting:GetChildren()) do
               if child:IsA("Atmosphere") then
                  child.Density = 0
               end
            end
         end)
         Rayfield:Notify({ Title = "No Fog", Content = "Kabut dibersihkan & dikunci.", Duration = 2 })
      else
         if fogLoop then
            fogLoop:Disconnect()
            fogLoop = nil
         end
         Lighting.FogEnd = 10000
         Rayfield:Notify({ Title = "No Fog", Content = "Efek No Fog dimatikan.", Duration = 2 })
      end
   end,
})

-- Tambahkan variabel ini di bagian atas skrip lu (di dekat variabel Lighting & RunService)
local shadowLoop = nil

-- Toggle: No Shadows / Full Bright (Anti Gelap & Bayangan)
MainTab:CreateToggle({
   Name = "Anti Gelap & Bayangan (Full Bright)",
   CurrentValue = false,
   Flag = "NoShadows",
   Callback = function(Value)
      if Value then
         -- Nyalakan kunci anti gelap & bayangan
         shadowLoop = RunService.Heartbeat:Connect(function()
            Lighting.GlobalShadows = false -- Matikan semua bayangan global game
            Lighting.Brightness = 3 -- Bikin pencahayaan jadi sangat terang
            Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255) -- Ubah warna cahaya luar jadi putih terang
            Lighting.Ambient = Color3.fromRGB(255, 255, 255) -- Ubah bayangan dasar jadi terang
            
            -- Hapus efek post-processing yang bikin gelap (seperti kontras berlebih atau efek malam)
            for _, child in pairs(Lighting:GetChildren()) do
               if child:IsA("PostEffect") or child:IsA("BlurEffect") or child:IsA("SunRaysEffect") then
                  child.Enabled = false
               end
            end
         end)
         Rayfield:Notify({ Title = "Anti Gelap Aktif", Content = "Bayangan dihapus dan map dipaksa terang.", Duration = 2 })
      else
         -- Matikan kunci / Off-kan
         if shadowLoop then
            shadowLoop:Disconnect()
            shadowLoop = nil
         end
         Lighting.GlobalShadows = true -- Kembalikan bayangan standar game (opsional)
         Lighting.Brightness = 1 -- Kembalikan standar
         Rayfield:Notify({ Title = "Anti Gelap Dimatikan", Content = "Pengaturan cahaya kembali normal.", Duration = 2 })
      end
   end,
})

-- Toggle 4: Kunci Hapus Skybox (Anti-Spawn Langit Custom)
MainTab:CreateToggle({
   Name = "Kunci Hapus Skybox",
   CurrentValue = false,
   Flag = "LockNoSky",
   Callback = function(Value)
      if Value then
         -- Terus-menerus mendeteksi dan menghapus objek Sky agar game tidak bisa memunculkannya lagi
         skyLoop = RunService.Heartbeat:Connect(function()
            for _, child in pairs(Lighting:GetChildren()) do
               if child:IsA("Sky") then
                  child:Destroy()
               end
            end
         end)
         Rayfield:Notify({ Title = "Skybox Dikunci", Content = "Skybox dibersihkan dan dicegah muncul kembali.", Duration = 2 })
      else
         -- Matikan kunci skybox
         if skyLoop then
            skyLoop:Disconnect()
            skyLoop = nil
         end
         Rayfield:Notify({ Title = "Skybox Dilepas", Content = "Kunci Hapus Skybox dimatikan.", Duration = 2 })
      end
   end,
})

-- Ambil layanan pemain lokal Roblox
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer

-- Toggle: Zoom Max (Jarak Kamera Tanpa Batas)
MainTab:CreateToggle({
   Name = "Zoom Jauh Tanpa Batas (Max Zoom)",
   CurrentValue = false,
   Flag = "MaxZoom",
   Callback = function(Value)
      if Value then
         -- Nyalakan: Set max zoom distance jadi sangat jauh (misal 999999)
         localPlayer.CameraMaxZoomDistance = 999999
         Rayfield:Notify({ Title = "Zoom Aktif", Content = "Jarak kamera diperpanjang maksimal.", Duration = 2 })
      else
         -- Matikan: Kembalikan ke jarak zoom standar game (biasanya 128 atau 400)
         localPlayer.CameraMaxZoomDistance = 400
         Rayfield:Notify({ Title = "Zoom Dimatikan", Content = "Jarak kamera kembali normal.", Duration = 2 })
      end
   end,
})
