# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gojuon Quiz is a Vue.js 3 application for memorizing Japanese Gojuon (kana characters). It provides an interactive quiz interface with statistics tracking for accuracy and speed. The application supports both hiragana and katakana characters, includes dark mode, and features internationalization support.

## Development Commands

- `yarn serve` - Start development server
- `yarn build` - Build for production  
- `yarn lint` - Run ESLint
- `yarn i18n:report` - Generate i18n translation report

## Technology Stack

- **Framework**: Vue.js 3 with Composition API
- **Build Tool**: Vue CLI Service
- **Internationalization**: Vue I18n v9
- **PWA**: Service Worker support enabled
- **Language**: JavaScript (no TypeScript)

## Key Architecture

### Core Data Structure
- `src/gojuon.js` - Contains all kana character data with properties:
  - `hiragana` and `katakana` characters
  - `sound` pronunciation
  - `type` (清/浊/半浊 - plain/voiced/semi-voiced)
  - `row` (a, ka, sa, ta, na, ha, ma, ya, ra, wa, n)
  - `skip_count` for irregular positions

### Main Application Structure
- `src/App.vue` - Root component with Google Analytics integration and PWA update handling
- `src/components/GojuonQuiz.vue` - Main quiz component with all game logic
- `src/i18n.js` - Vue I18n configuration supporting multiple languages
- `src/mixins/update.js` - PWA update handling mixin

### Settings Management
Settings are stored in `localStorage` and include:
- Input wait timing
- Font customization
- Sound playback (using Speech Synthesis API)
- Character selection (hiragana/katakana)
- Row selection (specific character groups)
- Theme mode (system/light/dark)

### Internationalization
- Language files in `src/locales/` directory
- Auto-detects browser language with English fallback
- Supports both Chinese and English interfaces

### PWA Features
- Service worker registration for offline functionality
- Automatic update detection and refresh prompts
- Progressive Web App manifest configuration

## Development Notes

- Uses Vue 3 Options API pattern in main components
- Implements localStorage for persistent settings
- Speech Synthesis API for audio pronunciation
- Google Analytics integration via script injection
- Responsive design with mobile-first approach
- Dark mode CSS implementation