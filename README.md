# Sprunki Playground

2D-песочница с ragdoll physics и инвентарём предметов. Android-приложение на основе Construct 3 runtime, запускаемого через WebView.

## Возможности

- Ragdoll-персонажи с физикой Box2D
- Инвентарь предметов: оружие, одежда, декорации, блоки, техника
- Стрельба, взрывы, разрушаемые объекты
- Транспорт: танк, вертолёт, мотоцикл
- Звуковые эффекты

## Скачать

APK доступен в [Releases](https://github.com/h33h/sprunki-playground/releases/latest).

## Сборка

Требуется JDK 17.

```bash
./gradlew :app:assembleDebug    # debug
./gradlew :app:assembleRelease  # release
```

APK: `app/build/outputs/apk/release/app-release.apk`

## Архитектура

- `app/` — Android WebView обёртка
- `webview/` — Construct 3 export (runtime + ассеты)

WebView загружает `index.html` через `WebViewAssetLoader` (`https://appassets.androidplatform.net/`) для корректной работы ES-модулей и WASM.
