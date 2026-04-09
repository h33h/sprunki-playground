# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sprunki Playground — 2D-песочница (ragdoll physics + инвентарь предметов). Два варианта реализации:

1. **kotlin-impl** — нативная реализация на Kotlin/libGDX + Box2D (desktop + Android)
2. **webview** — оригинальный Construct 3 runtime, запускается через WebView (desktop-webview, android-webview)

## Build & Run

Требуется JDK 17 (`/opt/homebrew/opt/openjdk@17`).

```bash
# Desktop (libGDX)
cd kotlin-impl && ./gradlew :desktop:run

# Desktop WebView (запускает локальный HTTP-сервер + Chrome в app-режиме)
cd kotlin-impl && ./gradlew :desktop-webview:run

# Android (libGDX)
cd kotlin-impl && ./gradlew :android:assembleDebug

# Android WebView
cd kotlin-impl && ./gradlew :android-webview:assembleDebug
```

Рабочая директория desktop-модуля — `kotlin-impl/core/assets/`, desktop-webview — `webview/`.

## Architecture (kotlin-impl)

### Модули Gradle

- **core** — вся игровая логика, не зависит от платформы
- **desktop** — LWJGL3-лаунчер
- **android** — Android-лаунчер (libGDX backend)
- **desktop-webview** — встроенный HTTP-сервер, открывает `webview/` в Chrome
- **android-webview** — Activity с WebView, загружает `webview/index.html` из assets

### Игровой цикл

`SprunkirGame` → `PlayScreen` — единственный экран. PlayScreen владеет всеми подсистемами:

- **PhysicsWorld** — обёртка над Box2D (PPM=100, fixed timestep 1/60). Создаёт ground и dynamic-тела.
- **EntityManager** — хранит Entity по id, фильтрует по компонентам/тегам
- **RagdollFactory** — собирает персонажа из 6 частей (body, head, 2 руки, 2 ноги) с RevoluteJoint. Каждая часть — dual-layer (zombie-спрайт + normal-спрайт через `visualChildIds`)
- **GameRenderer** — SpriteBatch, рисует фон, землю, все entity отсортированные по zOrder
- **InventoryPanel** — Scene2D UI, категории предметов с табами и тулбаром (delete/mirror)
- **DragDropHandler** — клик вне панели → спавн выбранного предмета
- **CameraController** — drag-перемещение + scroll-зум

### ECS (упрощённая)

Entity хранит nullable-компоненты напрямую (без component-map):
- `TransformComponent` — позиция, угол, масштаб
- `SpriteComponent` — TextureRegion, размеры, zOrder, mirrored, visible
- `PhysicsComponent` — ссылка на Box2D Body
- `PinComponent` — привязка к другому entity (одежда/шлемы)
- `HealthComponent`

Связи между entity: `parentId`, `visualChildIds`, `pinnedToId`.

### Система спрайтов

Все спрайты — sprite-sheet PNG в `core/assets/sprites/`. Координаты кадров захардкожены в `SpriteData` (FrameRect). 8 вариантов персонажей (frame 0–7), у каждой части тела свой sheet.

`ItemSprites` резолвит спрайт предмета по его id-префиксу (weapon_, bomb_, decor_, clothes_). Для блоков и техники используется иконка из buttonitem-sheet.

### Предметы

`ItemDef` описывает предмет: category, spawnType, iconIndex, физические параметры.
`ItemRegistry.load()` генерирует все предметы программно, маппя их на индексы иконок из SpriteData.

SpawnType определяет поведение:
- `RAGDOLL` → RagdollFactory
- `PHYSICS_BOX` → одиночное динамическое тело
- `PINNED` → привязка к ближайшей части рэгдолла (одежда/шлемы)
- `VEHICLE`, `STATIC_DECOR` → аналогично PHYSICS_BOX

## Webview

`webview/` — самодостаточное веб-приложение (Construct 3 export). `index.html` содержит заглушку `window.bridge` (InstantGamesBridge SDK mock), убирающую рекламу и платформенные зависимости. Скрипты в `scripts/` — минифицированный runtime, не предназначен для ручного редактирования.
