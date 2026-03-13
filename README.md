# AI Configs

Конфигурации для **Claude Code** и **Cursor AI** — агенты, правила и навыки для разных стеков.

```bash
git clone https://github.com/ENG1N3X/ai-configs.git
```

## Структура

```
ai-configs/
├── claude/          # Конфигурации для Claude Code
│   └── nuxt/
│       ├── agents/  # Агенты (architect, developer, reviewer)
│       ├── commands/ # Команды (/new-feature и др.)
│       └── skills/  # Навыки (nuxt, tailwind, swiper, yandex-metrika)
└── cursor/          # Конфигурации для Cursor AI
    └── swift/
        ├── agents/  # Агенты (ios-developer)
        ├── rules/   # Правила (структура папок проекта)
        └── skills/  # Навыки (ios-developer)
```

## Claude Code

1. Склонируйте репозиторий
2. Скопируйте нужные папки из `claude/<стек>/` в папку `.claude` вашего проекта
3. Папки `agents/`, `commands/` и `skills/` будут доступны в Claude Code

## Cursor AI

1. Склонируйте репозиторий
2. Скопируйте нужные папки из `cursor/<стек>/` в папку `.cursor` вашего проекта
3. Папки `rules/`, `agents/` и `skills/` будут доступны в Cursor
