<img align="right" width="200" height="200" src="src/assets/logo.png">

# Gojuon Quiz

一个五十音记忆辅助工具，包含准确率和速度统计。

[http://nekonull.me/50](http://nekonull.me/50)

![screenshot](screenshot.png)

## 功能特性

- **互动测验系统**: 练习平假名和片假名字符，即时反馈
- **字符选择**: 可选择特定字符类型（平假名/片假名）和行（あ、か、さ、た、な、は、ま、や、ら、わ、ん）
- **音频发音**: 内置语音合成功能，播放字符发音
- **性能追踪**: 实时统计准确率和打字速度
- **自定义设置**: 可调节输入等待时间和字体自定义
- **深色模式**: 支持系统/明亮/深色主题
- **渐进式Web应用**: 支持离线功能的Service Worker
- **国际化**: 支持英文和中文界面
- **响应式设计**: 移动设备友好界面

## 项目结构

```
src/
├── components/
│   └── GojuonQuiz.vue          # 主要测验组件，包含游戏逻辑
├── locales/
│   ├── en.json                 # 英文翻译
│   └── zh.json                 # 中文翻译
├── assets/
│   └── logo.png                # 应用程序图标
├── mixins/
│   └── update.js               # PWA更新处理混入
├── App.vue                     # 根组件，包含分析和PWA功能
├── main.js                     # 应用程序入口点
├── i18n.js                     # Vue I18n配置
├── gojuon.js                   # 完整的假名字符数据
└── registerServiceWorker.js    # Service Worker注册
```

## 数据结构

应用程序使用综合数据集（`src/gojuon.js`），包含所有46个基本日语假名字符的属性：
- `hiragana` 和 `katakana` 字符表示
- `sound` 罗马音发音
- `type`（清/浊/半浊 - 普通/浊音/半浊音）
- `row` 分类（あ、か、さ、た、な、は、ま、や、ら、わ、ん）
- `skip_count` 用于不规则位置

## 设置管理

用户偏好存储在localStorage中，包括：
- 输入等待时间，用于响应间隔
- 字体自定义，用于字符显示
- 声音播放开关，使用语音合成API
- 字符类型选择（平假名/片假名）
- 行选择，用于专注练习
- 主题模式（系统/明亮/深色）

## 技术栈

- **前端**: Vue.js 3 与组合式API
- **构建工具**: Vue CLI Service
- **国际化**: Vue I18n v9
- **PWA**: Service Worker支持已启用
- **语言**: JavaScript
- **音频**: Web语音合成API
- **存储**: localStorage用于设置持久化

## 开发者相关
- 构建: `yarn build`
- 预览: `yarn serve`
- 代码检查: `yarn lint`
- 国际化报告: `yarn i18n:report`

## Node.js 兼容性

本项目使用 Vue CLI 5.x，其依赖项（`@achrinza/node-ipc`）不正式支持 Node.js 22+。如果遇到以下错误：

```
error @achrinza/node-ipc@9.2.8: The engine "node" is incompatible with this module.
Expected version "8 || 9 || 10 || 11 || 12 || 13 || 14 || 15 || 16 || 17 || 18 || 19 || 20 || 21". Got "24.3.0"
```

请使用 `--ignore-engines` 标志运行安装命令：

```bash
yarn install --ignore-engines
```

这将绕过引擎兼容性检查，允许安装继续进行。
