<img align="right" width="200" height="200" src="src/assets/logo.png">

# Gojuon Quiz

[中文](README_zh-cn.md)

A memorization tool for Gojuon, including statistics for accuracy and speed.

[http://nekonull.me/50](http://nekonull.me/50)

![screenshot](screenshot.png)

## Features

- **Interactive Quiz System**: Practice hiragana and katakana characters with immediate feedback
- **Character Selection**: Choose specific character types (hiragana/katakana) and rows (a, ka, sa, ta, na, ha, ma, ya, ra, wa, n)
- **Audio Pronunciation**: Built-in speech synthesis for character pronunciation
- **Performance Tracking**: Real-time statistics for accuracy and typing speed
- **Customizable Settings**: Adjustable input wait timing and font customization
- **Dark Mode**: Support for system/light/dark themes
- **Progressive Web App**: Offline functionality with service worker
- **Internationalization**: Support for English and Chinese interfaces
- **Responsive Design**: Mobile-friendly interface

## Project Structure

```
src/
├── components/
│   └── GojuonQuiz.vue          # Main quiz component with game logic
├── locales/
│   ├── en.json                 # English translations
│   └── zh.json                 # Chinese translations
├── assets/
│   └── logo.png                # Application logo
├── mixins/
│   └── update.js               # PWA update handling mixin
├── App.vue                     # Root component with analytics and PWA
├── main.js                     # Application entry point
├── i18n.js                     # Vue I18n configuration
├── gojuon.js                   # Complete kana character data
└── registerServiceWorker.js    # Service worker registration
```

## Data Structure

The application uses a comprehensive dataset (`src/gojuon.js`) containing all 46 basic Japanese kana characters with properties:
- `hiragana` and `katakana` character representations
- `sound` pronunciation in romaji
- `type` (清/浊/半浊 - plain/voiced/semi-voiced)
- `row` classification (a, ka, sa, ta, na, ha, ma, ya, ra, wa, n)
- `skip_count` for irregular positioning

## Settings Management

User preferences are stored in localStorage and include:
- Input wait timing for response intervals
- Font customization for character display
- Sound playback toggle using Speech Synthesis API
- Character type selection (hiragana/katakana)
- Row selection for focused practice
- Theme mode (system/light/dark)

## Technology Stack

- **Frontend**: Vue.js 3 with Composition API
- **Build Tool**: Vue CLI Service
- **Internationalization**: Vue I18n v9
- **PWA**: Service Worker support enabled
- **Language**: JavaScript
- **Audio**: Web Speech Synthesis API
- **Storage**: localStorage for settings persistence

## For Developers
- build: `yarn build`
- serve: `yarn serve`
- lint: `yarn lint`
- i18n report: `yarn i18n:report`

## Node.js Compatibility

This project uses Vue CLI 5.x, which has a dependency (`@achrinza/node-ipc`) that doesn't officially support Node.js 22+. If you encounter the following error:

```
error @achrinza/node-ipc@9.2.8: The engine "node" is incompatible with this module.
Expected version "8 || 9 || 10 || 11 || 12 || 13 || 14 || 15 || 16 || 17 || 18 || 19 || 20 || 21". Got "24.3.0"
```

Run the install command with the `--ignore-engines` flag:

```bash
yarn install --ignore-engines
```

This will bypass the engine compatibility check and allow the installation to proceed.
