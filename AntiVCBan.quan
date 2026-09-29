local VoiceChatService = game:GetService("VoiceChatService")
local VoiceChatInternal = game:GetService("VoiceChatInternal")
local CoreGui = game:GetService("CoreGui")

local MUTED_IMAGE = "rbxasset://textures/ui/VoiceChat/MicLight/Muted.png"
local REJOIN_COUNT = 4
local REJOIN_DELAY = 5
local CurrentlyMuted = true

local TopBarApp = CoreGui:WaitForChild("TopBarApp"):WaitForChild("TopBarApp")
local UnibarMenu = TopBarApp:WaitForChild("UnibarLeftFrame"):WaitForChild("UnibarMenu")
local MicContainer = UnibarMenu:WaitForChild("2"):WaitForChild("3")
local MicPath = MicContainer:FindFirstChild("toggle_mic_mute")

local function get_mic_icon(micButton)
    micButton = micButton or MicPath
    return micButton:WaitForChild("IntegrationIconFrame"):WaitForChild("IntegrationIcon")["1"]
end

local function is_muted(): boolean
    return get_mic_icon().Image == MUTED_IMAGE
end

local function wait_for_mute()
    repeat task.wait(0.1) until is_muted()
end

local function wait_for_unmute()
    repeat task.wait(0.1) until not is_muted()
end

local function create_unmute_prompt(): TextLabel
    local label = Instance.new("TextLabel")
    label.Text = "Please unmute your microphone to continue."
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, 0, 0.03, 0)
    label.AnchorPoint = Vector2.new(0.5, 0.5)
    label.Position = UDim2.new(0.5, 0, 0.5, 0)
    label.TextScaled = true
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Parent = TopBarApp
    return label
end

local function ensure_voice_joined()
    if not MicPath then
        VoiceChatService:joinVoice()
        MicPath = MicContainer:WaitForChild("toggle_mic_mute")
        wait_for_mute()
    end
end

local function cycle_voice_connections()
    local groupId = VoiceChatInternal:GetGroupId()

    VoiceChatInternal:JoinByGroupId(groupId, true)
    VoiceChatService:leaveVoice()
    task.wait()

    for _ = 1, REJOIN_COUNT do
        VoiceChatInternal:JoinByGroupId(groupId, true)
    end

    task.wait(REJOIN_DELAY)
    VoiceChatService:joinVoice()
    VoiceChatInternal:JoinByGroupId(groupId, true)
end

local function replace_mic_button()
    MicPath.Visible = false

    local newMic = MicPath:Clone()
    newMic.Name = "toggle_mic_mute_new"
    newMic.Visible = true
    newMic.Parent = MicPath.Parent

    return newMic
end

local function watch_old_mic(newMic)
    local visibilityConn = MicPath:GetPropertyChangedSignal("Visible"):Connect(function()
        if MicPath.Visible then
            newMic:Destroy()
        end
    end)

    local destroyConn = MicPath.Destroying:Connect(function()
        newMic:Destroy()
    end)

    newMic.Destroying:Connect(function()
        visibilityConn:Disconnect()
        destroyConn:Disconnect()
    end)
end

local function setup_mic_toggle(newMic)
    local newIcon = get_mic_icon(newMic)
    local oldIcon = get_mic_icon(MicPath)
    local hitArea = newMic:WaitForChild("IconHitArea_toggle_mic_mute")
    local highlighter = newMic:WaitForChild("Highlighter")
    local redDot = newMic:WaitForChild("IntegrationIconFrame"):WaitForChild("IntegrationIcon"):WaitForChild("RedVoiceDot")
    highlighter.Visible = false
    redDot.Visible = false
    newIcon.Image = MUTED_IMAGE
    VoiceChatInternal:PublishPause(true)

    hitArea.MouseEnter:Connect(function()
        highlighter.Visible = true
    end)

    hitArea.MouseLeave:Connect(function()
        highlighter.Visible = false
    end)

    hitArea.Activated:Connect(function()
        CurrentlyMuted = not CurrentlyMuted
        VoiceChatInternal:PublishPause(CurrentlyMuted)

        if CurrentlyMuted then
            newIcon.Image = MUTED_IMAGE
            redDot.Visible = false
        else
            newIcon.Image = oldIcon.Image
            redDot.Visible = true
        end
    end)

    oldIcon:GetPropertyChangedSignal("Image"):Connect(function()
        if not CurrentlyMuted then
            newIcon.Image = oldIcon.Image
            redDot.Visible = true
        else
            newIcon.Image = MUTED_IMAGE
            redDot.Visible = false
        end
    end)
end

ensure_voice_joined()

local prompt = create_unmute_prompt()
wait_for_unmute()
prompt:Destroy()

cycle_voice_connections()

local newMic = replace_mic_button()
watch_old_mic(newMic)
setup_mic_toggle(newMic)
