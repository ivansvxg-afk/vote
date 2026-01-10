# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Команди за разработка

```bash
npm install          # Инсталиране на зависимости
npm run dev          # Стартиране на dev сървър с hot-reload
npm run build        # Билд за продукция
npm run preview      # Преглед на production билда
npm run lint         # ESLint с автоматични поправки
npm run format       # Prettier форматиране на src/
```

**Изисквания:** Node.js ^20.19.0 или >=22.12.0

## Архитектура

Проектът е Vue 3 SPA приложение с Vite, представляващо интерактивна страница за насърчаване на гласуване с драматични визуални ефекти.

### Структура

- `src/App.vue` - Основен компонент с цялата логика на приложението
- `src/components/CountdownTimer.vue` - Обратен брояч до датата на изборите
- `src/main.js` - Входна точка, монтира App към `#app`
- `@` alias сочи към `src/` директорията (конфигурирано в `vite.config.js`)

### Фиксирана дата на изборите

Датата на изборите е хардкодната: **3 март 2026** (`2026-03-03T00:00:00`). Приложението автоматично изчислява колко дни остават и показва обратен брояч.

### App.vue състояния

Екранът има няколко състояния (`screenState`):
- `neutral` - начално състояние (повече от 7 дни до изборите)
- `urgent` - 7 или по-малко дни до изборите
- `election` - изборният ден е настъпил
- `success` - след натискане на "Гласувах"
- `infected` - "nightmare mode" след натискане на "Няма да гласувам"

### Nightmare Mode

При натискане на "Няма да гласувам" се активират:
- Визуални ефекти: static overlay, blood drips, floating eyes, glitch clones
- Аудио: сирена (`/siren.mp3`), TTS фрази на български
- Вибрация (ако се поддържа)
- Ghost popups с escalating съобщения
- Doom countdown таймер

## Стил

- ESLint с Vue essential правила + Prettier интеграция
- Flat ESLint config (`eslint.config.js`)
- Игнорирани: `dist/`, `dist-ssr/`, `coverage/`
