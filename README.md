# Fish_hook
-- AUTO FIX REQUEST
request = request or http_request or syn.request

local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- ⚠️ FIX WEBHOOK (HTTPS)
local webhook = "‎https://discord.com/api/webhooks/1462962958299693141/rJtPJoLjpWSOHQ8ilzencmYJcQ_vfRqQsM5fzTVWvQ4A7gZwPiSo-KWI915D4kQc_G8-"

local cache = {}

-- SCREENSHOT
local function takeScreenshot()
    if getscreenshot then
        return getscreenshot()
    elseif screenshot then
        return screenshot()
    else
        warn("Executor tidak support screenshot")
        return nil
    end
end

-- SEND WEBHOOK
local function sendWebhookWithImage(text)
    if cache[text] then return end
    cache[text] = true
    task.delay(5, function() cache[text] = nil end)

    local image = takeScreenshot()

    if not image then
        request({
            Url = webhook,
            Method = "POST",
            Headers = {["Content-Type"] = "application/json"},
            Body = HttpService:JSONEncode({
                content = "📸 "..text.." (no screenshot support)"
            })
        })
        return
    end

    local boundary = "----WebhookBoundary"..tostring(math.random())

    local body = ""
    body = body.."--"..boundary.."\r\n"
    body = body.."Content-Disposition: form-data; name=\"content\"\r\n\r\n"
    body = body.."📸 ULTRA RARE DETECTED\n"..text.."\r\n"

    body = body.."--"..boundary.."\r\n"
    body = body.."Content-Disposition: form-data; name=\"file\"; filename=\"fish.png\"\r\n"
    body = body.."Content-Type: image/png\r\n\r\n"
    body = body..image.."\r\n"
    body = body.."--"..boundary.."--"

    request({
        Url = webhook,
        Method = "POST",
        Headers = {
            ["Content-Type"] = "multipart/form-data; boundary="..boundary
        },
        Body = body
    })
end

-- FILTER ULTRA
local function isUltra(text)
    text = tostring(text):lower()
    return text:find("secret") or text:find("forgotten")
end

-- UI DETECT
local function scanUI(gui)
    for _,v in pairs(gui:GetDescendants()) do
        if v:IsA("TextLabel") or v:IsA("TextButton") then
            if isUltra(v.Text) then
                sendWebhookWithImage(v.Text)
            end
        end
    end
end

PlayerGui.ChildAdded:Connect(function(gui)
    task.wait(0.2)
    scanUI(gui)
end)

-- BACKPACK
local backpack = player:WaitForChild("Backpack")

backpack.ChildAdded:Connect(function(item)
    if isUltra(item.Name) then
        sendWebhookWithImage(item.Name)
    end
end)

print("📸 Loader Script aktif!")
