 -- instalador_online.lua
-- Cole isto em um Pastebin/Gist (raw) e use loadstring(game:HttpGet("RAW_URL"))()

local ss = game:GetService("ServerScriptService")
local sp = game:GetService("StarterPlayer")
local repl = game:GetService("ReplicatedStorage")

local serverSrc = [[
-- AdminServer (gerado)
local Players = game:GetService("Players")
local Replicated = game:GetService("ReplicatedStorage")
local Admins = { [12345678] = true } -- substitua pelos seus UserIds
local REQUIRE_CONSENT = true
local RE = Replicated:FindFirstChild("RemoteEvents")
if not RE then RE = Instance.new("Folder"); RE.Name="RemoteEvents"; RE.Parent=Replicated end
local function E(n) local ev=RE:FindFirstChild(n) if not ev then ev=Instance.new("RemoteEvent"); ev.Name=n; ev.Parent=RE end return ev end
local REQ_TRANSFER = E("RequestTransfer"); local SET_CONSENT = E("SetConsent"); local SPAWN_TOOL = E("SpawnBrainrot")
local TELEPORT_REQ = E("TeleportToBase"); local NOTIFY_CLIENT = E("NotifyClient")
if not Replicated:FindFirstChild("BrainrotTemplate") then
    local t=Instance.new("Tool"); t.Name="BrainrotTemplate"; t.CanBeDropped=true
    local h=Instance.new("Part"); h.Name="Handle"; h.Size=Vector3.new(1,1,1); h.Parent=t
    local m=Instance.new("SpecialMesh",h); m.MeshType=Enum.MeshType.Sphere; m.Scale=Vector3.new(1.4,1.4,1.4)
    t.Parent=Replicated
end
local function isAdmin(pl) return pl and Admins[pl.UserId] end
local consent = {}
SET_CONSENT.OnServerEvent:Connect(function(pl,allow) if type(allow)~="boolean" then return end consent[pl.UserId]=allow; NOTIFY_CLIENT:FireClient(pl,"Consent:"..tostring(allow)) end)
REQ_TRANSFER.OnServerEvent:Connect(function(invoker,targetId)
    if not isAdmin(invoker) then REQ_TRANSFER:FireClient(invoker,false,"Só admins") return end
    if type(targetId)~="number" then REQ_TRANSFER:FireClient(invoker,false,"ID inválido") return end
    local target = Players:GetPlayerByUserId(targetId)
    if not target then REQ_TRANSFER:FireClient(invoker,false,"Alvo offline") return end
    if REQUIRE_CONSENT and not consent[targetId] then REQ_TRANSFER:FireClient(invoker,false,"Consent necessário") return end
    local function findItem(p)
        local bp = p:FindFirstChildOfClass("Backpack")
        if bp then local it = bp:FindFirstChild("Brainrot") if it then return it end end
        if p.Character then local it2 = p.Character:FindFirstChild("Brainrot") if it2 then return it2 end end
        return nil
    end
    local item = findItem(target)
    if not item then REQ_TRANSFER:FireClient(invoker,false,"Brainrot não encontrado") return end
    local invBP = invoker:FindFirstChildOfClass("Backpack")
    if not invBP then REQ_TRANSFER:FireClient(invoker,false,"Backpack admin ausente") return end
    item.Parent = invBP
    REQ_TRANSFER:FireClient(invoker,true,"Transferido")
    NOTIFY_CLIENT:FireClient(target,"Seu Brainrot foi transferido por admin.")
end)
SPAWN_TOOL.OnServerEvent:Connect(function(pl,pos)
    if not isAdmin(pl) then return end
    local tpl = Replicated:FindFirstChild("BrainrotTemplate")
    if not tpl then return end
    local c = tpl:Clone(); c.Name="Brainrot"; c.Parent = workspace
    local p = (typeof(pos)=="Vector3") and pos or (pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") and pl.Character.HumanoidRootPart.Position+Vector3.new(0,5,0) or Vector3.new(0,5,0))
    if c:IsA("Tool") then local h=c:FindFirstChild("Handle") if h then h.CFrame=CFrame.new(p) end
    elseif c.PrimaryPart then c:SetPrimaryPartCFrame(CFrame.new(p))
    else local pr=Instance.new("Part"); pr.Name="BrainrotPart"; pr.Size=Vector3.new(1,1,1); pr.Position=p; pr.Parent=workspace end
end)
local BASE = CFrame.new(0,10,0)
TELEPORT_REQ.OnServerEvent:Connect(function(pl) if not isAdmin(pl) then return end if pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then pl.Character:PivotTo(BASE) end end)
print("AdminServer criado (via instalador online). Ajuste Admins[] se necessário.")
]]

local clientSrc = [[
-- AdminClientInit (gerado)
local Players=game:GetService("Players")
local Replicated=game:GetService("ReplicatedStorage")
local UIS=game:GetService("UserInputService")
local player=Players.LocalPlayer
local RE=Replicated:WaitForChild("RemoteEvents")
local REQ_TRANSFER=RE:WaitForChild("RequestTransfer")
local SET_CONSENT=RE:WaitForChild("SetConsent")
local SPAWN_TOOL=RE:WaitForChild("SpawnBrainrot")
local TELEPORT_REQ=RE:WaitForChild("TeleportToBase")
local NOTIFY_CLIENT=RE:WaitForChild("NotifyClient")
local AdminsClient={ [12345678]=true } -- opcional: ids para visual
local function isAdmin() return AdminsClient[player.UserId] end
local gui=Instance.new("ScreenGui"); gui.Name="AdminMenu"; gui.Parent=player:WaitForChild("PlayerGui")
local frame=Instance.new("Frame"); frame.Size=UDim2.new(0,220,0,380); frame.Position=UDim2.new(1,-230,0.5,-190); frame.AnchorPoint=Vector2.new(1,0.5); frame.BackgroundTransparency=0.15; frame.Parent=gui
frame.Active=true; frame.Draggable=true
local function mk(n,y,t) local b=Instance.new("TextButton"); b.Name=n; b.Size=UDim2.new(1,-16,0,34); b.Position=UDim2.new(0,8,0,y); b.Font=Enum.Font.SourceSans; b.TextSize=16; b.Text=t; b.Parent=frame; return b end
local tp=mk("TeleportBtn",16,"Teleport: Base"); local sp=mk("SpawnBtn",64,"Spawn Brainrot"); local tf=mk("TransferBtn",112,"Transferir")
local idBox=Instance.new("TextBox"); idBox.Size=UDim2.new(1,-16,0,28); idBox.Position=UDim2.new(0,8,0,160); idBox.PlaceholderText="UserId do alvo"; idBox.Parent=frame
local consentBtn=mk("ConsentBtn",204,"Permitir transferências (off)"); local flyBtn=mk("FlyBtn",252,"Toggle Fly (local)")
local invisBtn=mk("InvisBtn",300,"Toggle Invis (local)"); local noclipBtn=mk("NoclipBtn",348,"Toggle Noclip (local)"); local jumpBtn=mk("JumpBtn",396,"Toggle InfJump (local)")
NOTIFY_CLIENT.OnClientEvent:Connect(function(m) print("[SERVER] "..tostring(m)) end)
tp.MouseButton1Click:Connect(function() TELEPORT_REQ:FireServer() end)
sp.MouseButton1Click:Connect(function() local pos=nil if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then pos=player.Character.HumanoidRootPart.Position+Vector3.new(0,5,0) end SPAWN_TOOL:FireServer(pos) end)
tf.MouseButton1Click:Connect(function() local id=tonumber(idBox.Text) if id then REQ_TRANSFER:FireServer(id) else NOTIFY_CLIENT:FireServer("UserId inválido") end end)
local consent=false consentBtn.MouseButton1Click:Connect(function() consent=not consent consentBtn.Text="Permitir transferências ("..(consent and "on" or "off")..")" SET_CONSENT:FireServer(consent) end)
do local flying=false local bv flyBtn.MouseButton1Click:Connect(function() flying=not flying local root=player.Character and player.Character:FindFirstChild("HumanoidRootPart") if not root then return end if flying then bv=Instance.new("BodyVelocity"); bv.MaxForce=Vector3.new(1e5,1e5,1e5); bv.Parent=root; spawn(function() while flying and bv.Parent do local v=Vector3.new() if UIS:IsKeyDown(Enum.KeyCode.W) then v=v+workspace.CurrentCamera.CFrame.LookVector end if UIS:IsKeyDown(Enum.KeyCode.S) then v=v-workspace.CurrentCamera.CFrame.LookVector end if UIS:IsKeyDown(Enum.KeyCode.A) then v=v-workspace.CurrentCamera.CFrame.RightVector end if UIS:IsKeyDown(Enum.KeyCode.D) then v=v+workspace.CurrentCamera.CFrame.RightVector end if UIS:IsKeyDown(Enum.KeyCode.Space) then v=v+Vector3.new(0,1,0) end if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then v=v-Vector3.new(0,1,0) end bv.Velocity=v*35 task.wait(0.03) end if bv and bv.Parent then bv:Destroy() end end) else if bv and bv.Parent then bv:Destroy() end end end) end
invisBtn.MouseButton1Click:Connect(function() local c=player.Character if not c then return end local cur=false for _,part in ipairs(c:GetChildren()) do if part:IsA("BasePart") then cur=(part.Transparency==1) break end end local n=not cur for _,part in ipairs(c:GetDescendants()) do if part:IsA("BasePart") then part.Transparency=n and 1 or 0 elseif part:IsA("Decal") or part:IsA("Texture") then part.Transparency=n and 1 or 0 elseif part:IsA("Accessory") and part:FindFirstChild("Handle") then part.Handle.Transparency=n and 1 or 0 end end pcall(function() NOTIFY_CLIENT:FireServer("Invis toggled") end) end)
noclipBtn.MouseButton1Click:Connect(function() local c=player.Character if not c then return end for _,p in ipairs(c:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = not p.CanCollide end end pcall(function() NOTIFY_CLIENT:FireServer("Noclip toggled") end) end)
do local on=false jumpBtn.MouseButton1Click:Connect(function() on=not on pcall(function() NOTIFY_CLIENT:FireServer("InfJump "..tostring(on)) end) end) local function hook(ch) local h=ch:WaitForChild("Humanoid") h.Jumping:Connect(function(a) if on and a then h:ChangeState(Enum.HumanoidStateType.Jumping) end end) end if player.Character then hook(player.Character) end player.CharacterAdded:Connect(hook) end
]]

local function createOrReplaceServer()
    local existing = ss:FindFirstChild("AdminServer")
    if existing then existing:Destroy() end
    local s = Instance.new("Script")
    s.Name = "AdminServer"
    s.Source = serverSrc
    s.Parent = ss
    print("AdminServer criado em ServerScriptService.")
end

local function createOrReplaceClient()
    local sps = sp:FindFirstChild("StarterPlayerScripts")
    if not sps then sps = Instance.new("StarterPlayerScripts"); sps.Parent = sp end
    local existing = sps:FindFirstChild("AdminClientInit")
    if existing then existing:Destroy() end
    local ls = Instance.new("LocalScript")
    ls.Name = "AdminClientInit"
    ls.Source = clientSrc
    ls.Parent = sps
    print("AdminClientInit criado em StarterPlayerScripts.")
end

createOrReplaceServer()
createOrReplaceClient()
print("Instalação concluída. Edite ServerScriptService/AdminServer para ajustar Admins[] se quiser.")
