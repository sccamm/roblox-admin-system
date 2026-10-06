<div align="center">

# 🎮 Roblox Admin System

**Полноценная админ-система для Roblox на Lua**

![Lua](https://img.shields.io/badge/Lua-2C2D72?style=for-the-badge&logo=lua&logoColor=white)
![Roblox](https://img.shields.io/badge/Roblox-000000?style=for-the-badge&logo=roblox&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)

</div>

---

## 📖 О проекте

Админ-система для Roblox-игр с ранговой моделью, логами и интеграцией с Discord.

### Возможности

- ⚡ **30+ команд**: `kick`, `ban`, `warn`, `freeze`, `speed`, `god`, `tp`
- 🏅 **Ранговая система**: owner → admin → mod → trial
- 📝 **Асинхронное логирование** в Discord-вебхук через `HttpService`
- 🛡 **Защита от спама**: кулдауны на команды
- 🔧 **Конфиг** через `Config.lua` — не нужно лезть в код

---

## 🗂 Структура

```
roblox-admin-system/
├── ServerScriptService/
│   ├── AdminSystem.server.lua      — инициализация и роутинг команд
│   ├── CommandHandler.lua          — парсер и вызов команд
│   └── Logger.lua                  — вебхук-логи
├── ReplicatedStorage/
│   ├── Config.lua                  — ранги, кулдауны, вебхук-URL
│   └── Remotes.lua                 — RemoteEvent'ы
└── README.md
```

---

## 🚀 Установка

1. Открой Roblox Studio → **View → Command Bar**
2. Скопируй папку `ServerScriptService` в свой проект
3. В `ReplicatedStorage/Config.lua` укажи:
   ```lua
   Config.WebhookURL = "https://discord.com/api/webhooks/..."
   Config.OwnerIds = { 123456789 }
   ```
4. Опубликуй — система готова

---

## 💻 Пример кода

**Config.lua**

```lua
local Config = {}

Config.WebhookURL = "https://discord.com/api/webhooks/xxx/yyy"
Config.OwnerIds = { 123456789, 987654321 }
Config.Cooldown = 1.5

Config.Ranks = {
    ["owner"] = 4,
    ["admin"] = 3,
    ["mod"]   = 2,
    ["trial"] = 1,
}

return Config
```

**Logger.lua**

```lua
local HttpService = game:GetService("HttpService")
local Config = require(game.ReplicatedStorage.Config)

local Logger = {}

function Logger.log(action, executor, target, reason)
    local payload = {
        embeds = {{
            title = "🛡 " .. action,
            color = 0x3B82F6,
            fields = {
                { name = "Выполнил", value = executor.Name, inline = true },
                { name = "Цель",     value = target and target.Name or "—", inline = true },
                { name = "Причина",  value = reason or "не указана", inline = false },
            },
            timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ"),
        }}
    }

    pcall(function()
        HttpService:PostAsync(
            Config.WebhookURL,
            HttpService:JSONEncode(payload),
            Enum.HttpContentType.ApplicationJson
        )
    end)
end

return Logger
```

**CommandHandler.lua**

```lua
local Players = game:GetService("Players")
local Config = require(game.ReplicatedStorage.Config)
local Logger = require(script.Parent.Logger)

local Commands = {}

local function getRank(player)
    for rank, level in pairs(Config.Ranks) do
        if player:GetRankInGroup(0) >= level then
            return rank, level
        end
    end
    return "guest", 0
end

Commands.kick = {
    rank = "mod",
    run = function(executor, args)
        local target = Players:FindFirstChild(args[1])
        if not target then return "❌ Игрок не найден" end

        Logger.log("Kick", executor, target, args[2])
        target:Kick(args[2] or "Кик от администрации")
        return "✅ " .. target.Name .. " кикнут"
    end,
}

Commands.ban = {
    rank = "admin",
    run = function(executor, args)
        local target = Players:FindFirstChild(args[1])
        if not target then return "❌ Игрок не найден" end

        Logger.log("Ban", executor, target, args[2])
        target:Kick("Бан: " .. (args[2] or "без причины"))
        return "✅ " .. target.Name .. " забанен"
    end,
}

function Commands.run(executor, message)
    local args = message:split(" ")
    local cmd = args[1]
    table.remove(args, 1)

    local command = Commands[cmd]
    if not command then return end

    local _, level = getRank(executor)
    if level < Config.Ranks[command.rank] then
        return "❌ Недостаточно прав"
    end

    return command.run(executor, args)
end

return Commands
```

---

## 📜 Лицензия

MIT
