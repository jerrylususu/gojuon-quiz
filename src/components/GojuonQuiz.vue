<template>
<details class="quickstart-section">
    <summary class="quickstart-summary">{{ t('quickstart_title') }}</summary>

<div class="quickstart-content">
<p>{{ t('quickstart_content.0') }}</p>
<p>{{ t('quickstart_content.1') }}</p>
<p>{{ t('quickstart_content.2') }}</p>
<p>{{ t('quickstart_content.3') }}<a href="https://github.com/jerrylususu/gojuon-quiz" target="_blank" >Github</a></p>
<p>{{ t('quickstart_content.4') }}</p>
</div>

</details>
<details class="settings-section">
    <summary class="settings-summary">{{ t('settings_title') }}</summary>
    <div class="settings-grid">
        <div class="setting-item">
            <label class="setting-label">{{ t('settings.input_wait') }}</label>
            <div class="setting-control">
                <input type="number" v-model="this.settings.correct_wait_ms" class="number-input" />
                <span class="unit">ms</span>
            </div>
        </div>
        
        <div class="setting-item">
            <label class="setting-label">Language</label>
            <div class="setting-control">
                <select v-model="ui_language" class="select-input">
                    <option v-for="(lang, i) in langs" :key="`Lang${i}`" :value="lang">
                        {{ lang }}
                    </option>
                </select>
            </div>
        </div>
        
        <div class="setting-item">
            <label class="setting-label">{{ t('settings.font') }}</label>
            <div class="setting-control">
                <input type="text" v-model="this.settings.font_css_str" class="text-input" placeholder="sans-serif, serif"/>
            </div>
        </div>
        
        <div class="setting-item">
            <label class="setting-label">{{ t('settings.play_sound') }}</label>
            <div class="setting-control">
                <label class="switch">
                    <input type="checkbox" v-model="settings.play_sound_enabled">
                    <span class="slider"></span>
                </label>
                <p v-if="!speech_synthesis_supported" class="warning-text">{{ t('settings.play_sound_not_supported') }}</p>
            </div>
        </div>
        
        <div class="setting-item">
            <label class="setting-label">{{ t('settings.test_on') }}</label>
            <div class="setting-control">
                <div class="checkbox-group">
                    <label class="checkbox-label">
                        <input type="checkbox" v-model="hiragana_enabled">
                        <span class="checkbox-text">{{ t('hiragana') }}</span>
                    </label>
                    <label class="checkbox-label">
                        <input type="checkbox" v-model="katakana_enabled">
                        <span class="checkbox-text">{{ t('katakana') }}</span>
                    </label>
                </div>
                <p v-if="!hiragana_enabled && !katakana_enabled" class="error-text">{{ t('settings.nothing_can_be_shown')}}</p>
            </div>
        </div>
        
        <div class="setting-item">
            <label class="setting-label">{{ t('settings.enabled_rows') }}</label>
            <div class="setting-control">
                <div class="checkbox-grid">
                    <label v-for="(row, index) in all_supported_rows" :key="index" class="checkbox-label">
                        <input type="checkbox" v-model="settings.enabled_rows" @change="initGame" :value="row">
                        <span class="checkbox-text">{{ row }}</span>
                    </label>
                </div>
            </div>
        </div>
        
        <div class="setting-item">
            <label class="setting-label">{{ t('settings.dark_mode') }}</label>
            <div class="setting-control">
                <select v-model="settings.theme_mode" class="select-input">
                    <option value="system">{{ t('settings.theme_system') }}</option>
                    <option value="light">{{ t('settings.theme_light') }}</option>
                    <option value="dark">{{ t('settings.theme_dark') }}</option>
                </select>
            </div>
        </div>
    </div>
</details>

<details class="history-section">
    <summary class="history-summary">{{ t('history_title') }}</summary>
    <table>
        <tr>
            <th>{{ t('history.overall_accuracy') }}</th>
            <th>{{ t('history.overall_time') }}</th>
            <th>{{ t('history.finished_pool') }}</th>
            <th>{{ t('history.combo') }}</th>
            <th>{{ t('history.last_10') }} </th>
        </tr>
        <tr>
            <td class="text-centered">{{this.total_accruacy_str}} ({{this.stats.total_correct}}/{{this.stats.total_tested}})</td>
            <td class="text-centered">{{this.average_time_str}}</td>
            <td class="text-centered">{{this.stats.pool_finish_count}}</td>
            <td class="text-centered">{{this.stats.combo}} ({{ t('history.max_combo') }} {{this.stats.max_combo}})</td>
            <td>    
                <span v-for="item in this.stats.last_10.slice().reverse()" :key="item.enrolled_time" :class="{correct: item.isCorrect, incorrect: !item.isCorrect}">  
                    {{item.hiragana}}{{item.katakana}}
                </span>
            </td>
        </tr>
    </table>
</details>


<div class="text-centered">
<table class="centered" :style="this.custom_css">
  <tr>
    <th>{{ t('hiragana') }}</th>
    <th>{{ t('katakana') }}</th>
    <th>{{ t('sound') }}</th>
  </tr>
  <tr>
    <td class="large">{{this.display_hiragana}}</td>
    <td class="large">{{this.display_katakana}}</td>
    <td class="large">{{this.display_sound}}</td>
  </tr>
</table>

<div class="sound-input centered" :class="inputBackgroundClass">&nbsp;{{this.inputValue}}&nbsp;</div>
<input class="sound-input centered"  @input="onInput" v-on:keyup.enter="onEnter" v-model="this.inputValue" type="password"
    :disabled="!hiragana_enabled && !katakana_enabled" autocomplete="off" style="color:transparent"/>
<div class="centered" v-show="initial_input">{{ t('click_above_to_start') }}</div>
</div>

<details class="speed-section">
    <summary class="speed-summary">{{ t('speed_stat') }} (ms)</summary>
    <table>
        <tr v-for="(row, hidx) in this.history" :key="hidx">
            <td v-for="(item, vidx) in row" :key="vidx" :style="this.speed_style(item)">
            <span v-if="item.hiragana !== '' ">{{item.hiragana}}{{item.avg_time_ms.toFixed(0)}}</span>
                
            </td>
        </tr>
    </table>
</details>

<details class="accuracy-section">
    <summary class="accuracy-summary">{{ t('accuracy_stat') }}</summary>
    <table>
        <tr v-for="(row, hidx) in this.history" :key="hidx">
            <td v-for="(item, vidx) in row" :key="vidx" :style="this.accuracy_style(item)">
                <span v-if="item.hiragana !== '' ">{{item.hiragana}}{{item.total === 0 ? 0: (item.correct / item.total * 100).toFixed(0)}}%</span>
            </td>
        </tr>
    </table>
</details>

<!-- <table>
    <tr v-for="(row, hidx) in this.history" :key="hidx">
        <td v-for="(item, vidx) in row" :key="vidx">
            {{item.hiragana}}- {{item.relative_speed}}
        </td>
    </tr>
</table> -->


</template>

<script>

import { gujuon_data } from "@/gojuon.js";
import { useI18n } from 'vue-i18n'


export default {
    data() {
        return {
            inputValue: "",
            show_hiragana: false,
            show_katakana: false,
            show_sound: false,
            existing_timeout: null,
            last_show_time: null,
            initial_input: true,
            settings: {
                ui_language: 'zh',
                correct_wait_ms: 1000,
                font_css_str: 'sans-serif, serif',
                enabled_rows: ["a", "ka", "sa", "ta", "na", "ha", "ma", "ya", "ra", "wa", "(...)"],
                play_sound_enabled: false,
                theme_mode: 'system',
            },
            current: {
                hiragana: "",
                katakana: "",
                sound: "",
            },
            stats: {
                last_10: [],
                total_tested: 0,
                total_correct: 0,
                total_response_time: 0,
                pool_finish_count: 0,
                combo: 0,
                max_combo: 0,
            },
            history: [],
            hiragana_to_history: {},
            pool: [],
            langs: ['zh', 'en'],
            hiragana_enabled: true,
            katakana_enabled: true,
            all_supported_rows: ["a", "ka", "sa", "ta", "na", "ha", "ma", "ya", "ra", "wa", "(...)"],
            speech_synthesis_supported: false,
            speech_synthesis_instance: null,
        }
    },
    computed: {
        correct() {
            return this.current.sound.split(",").includes(this.inputValue.toLowerCase());
        },
        maxLenInAnswer() {
            return this.current.sound.split(",").map(x => x.length).reduce((a, b) => Math.max(a, b))
        },    
        inputBackgroundClass() {
            return {
                correct: this.correct,
                incorrect: !this.correct && this.inRevealState,
            }
        },
        display_hiragana() {
            return this.show_hiragana ? this.current.hiragana: " "
        },
        display_katakana() {
            return this.show_katakana ? this.current.katakana: " "
        },
        display_sound() {
            return this.show_sound ? this.current.sound: " "
        },
        inRevealState() {
            return this.show_hiragana && this.show_katakana && this.show_sound
        },
        total_accruacy_str() {
            return `${(this.stats.total_correct / this.stats.total_tested * 100).toFixed(2)}%`
        },
        average_time_str() {
            return `${(this.stats.total_response_time / this.stats.total_tested).toFixed(2)}ms`
        },
        custom_css() {
            return `font-family: ${this.settings.font_css_str};`
        },
        ui_language: {
            get() {
                return this.$i18n.locale;
            },
            set(value) {
                this.settings.ui_language = value;
                this.$i18n.locale = value;
            }
        }
    },
    methods: {
        onInput() {
            if (this.correct) {
                this.reveal()
                this.existing_timeout = setTimeout(() => {
                    this.next()
                }, this.settings.correct_wait_ms);
            } else if (this.inputValue.length == this.maxLenInAnswer) {
                this.reveal()
            }
        },
        onEnter() {
            if (this.existing_timeout !== null) {
                window.clearTimeout(this.existing_timeout);
            }
            if (this.correct) {
                this.next()
                return
            }
            if (this.inRevealState && !this.correct) {
                this.next()
                return
            }
            if (!this.correct) {
                this.reveal()
                return
            }
        },
        updateHistory() {
            if (this.initial_input) {
                this.initial_input = false;
                return;
            }
            this.hiragana_to_history[this.current.hiragana].total += 1;
            this.stats.total_tested += 1;
            let response_time = Date.now() - this.last_show_time;
            this.stats.last_10.push({
                hiragana: this.current.hiragana,
                katakana: this.current.katakana,
                isCorrect: this.correct,
                enrolled_time: this.last_show_time,
            });
            if (this.stats.last_10.length > 10) {
                this.stats.last_10.shift();
            }

            if(!this.correct) {
                this.stats.combo = 0;
            }

            if(this.correct) {
                this.stats.combo += 1;
                this.stats.max_combo = Math.max(this.stats.max_combo, this.stats.combo);
                this.stats.total_response_time += response_time;
                this.stats.total_correct += 1;
                let history_item = this.hiragana_to_history[this.current.hiragana]
                // only update response time when success
                history_item.avg_time_ms = (history_item.avg_time_ms * history_item.correct + response_time) / (history_item.correct + 1);
                history_item.correct += 1;
            }

            // update relative speed
            let max_time = 0
            let min_time = 1000000000
            
            for (const row of this.history) {
                for (const cell of row) {
                    if (cell.avg_time_ms > 0) {
                        max_time = Math.max(max_time, cell.avg_time_ms)
                        min_time = Math.min(min_time, cell.avg_time_ms)
                    }
                }
            }

            for (let row of this.history) {
                for (let cell of row) {
                    if (cell.avg_time_ms > 0) {
                        cell.relative_speed = (cell.avg_time_ms - min_time + 0.001) / (max_time - min_time)
                    }
                }
            }
        },
        reveal() {
            this.updateHistory();
            this.show_hiragana = true;
            this.show_katakana = true;
            this.show_sound = true;
            if (this.settings.play_sound_enabled && this.speech_synthesis_supported) {
                this.speech_synthesis_instance.text = this.current.katakana;  // a random decision, both should be fine
                window.speechSynthesis.speak(this.speech_synthesis_instance);
            }
        },
        next() {
            this.inputValue = ""
            // try to balance between true random and iterate all items in pull
            let next_item = null
            if (this.pool.length == 0) {
                this.fillPool()
                this.stats.pool_finish_count += 1;
            }
            if (Math.random() < 0.8 && !this.initial_input) {
                let random_index = Math.floor(Math.random() * this.pool.length)
                next_item = this.pool[random_index]
                this.pool.splice(random_index, 1)
            } else {
                next_item = this.randomInRange();
            }

            this.current.sound = next_item.sound;
            this.current.hiragana = next_item.hiragana;
            this.current.katakana = next_item.katakana;

            // random hide hiragana or katakana
            if (this.hiragana_enabled && this.katakana_enabled) {
                if (Math.random() > 0.5) {
                    this.show_hiragana = false;
                    this.show_katakana = true;
                } else {
                    this.show_katakana = false;
                    this.show_hiragana = true;
                }
            } else {
                if (this.hiragana_enabled && !this.katakana_enabled) {
                    this.show_hiragana = true;
                    this.show_katakana = false;
                } else if (!this.hiragana_enabled && this.katakana_enabled) {
                    this.show_hiragana = false;
                    this.show_katakana = true;
                }
            }
            
            this.show_sound = false;

            // add history
            this.last_show_time = Date.now();
        },
        buildHistoryMatrix() {
            let matrix = [[],[],[],[],[]]
            let row = -1;
            let empty_item = {
                hiragana: "",
                katakana: "",
                sound: "",
                total: 0,
                correct: 0,
                avg_time_ms: 0,
                relative_speed: 0,
            }
            let hiragana_to_history = {};
            for (let item of gujuon_data) {
                hiragana_to_history[item.hiragana] = item;

                item["total"] = 0;
                item["correct"] = 0;
                item["avg_time_ms"] = 0;
                item["relative_speed"] = 0;

                row = (row + 1) % 5;
                matrix[row].push(item);
                 
                if (item.skip_count !== "") {
                    item.skip_count = parseInt(item.skip_count);
                    for (let i = 0; i < item.skip_count; i++) {
                        row = (row + 1) % 5;
                        matrix[row].push(empty_item); 
                    }
                }
            }
            this.history = matrix;
            this.hiragana_to_history = hiragana_to_history;
        },
        lerp_color(a, b, amount) {
            // https://gist.github.com/nikolas/b0cce2261f1382159b507dd492e1ceef
            const ar = a >> 16,
            ag = a >> 8 & 0xff,
            ab = a & 0xff,

            br = b >> 16,
            bg = b >> 8 & 0xff,
            bb = b & 0xff,

            rr = ar + amount * (br - ar),
            rg = ag + amount * (bg - ag),
            rb = ab + amount * (bb - ab);

            return '#' + ((1 << 24) + (rr << 16) + (rg << 8) + rb | 0).toString(16).slice(1);
        },
        speed_style(item) {
            if (item.relative_speed == 0) {
                return {}
            }

            let speed = item.relative_speed;
            let color = this.lerp_color(0x00ff00, 0xff0000, speed);
            return {
                backgroundColor: color,
            }
        },
        accuracy_style(item) {
            if (item.total == 0) {
                return {}
            }

            let color = this.lerp_color(0xff0000, 0x00ff00, item.correct / item.total);
            return {
                backgroundColor: color,
            }
        },
        fillPool() {
            this.pool = [...gujuon_data].filter(x =>
                this.settings.enabled_rows.includes(x.row)
                || (this.settings.enabled_rows.includes("(...)") && !this.all_supported_rows.includes(x.row)));
        },
        randomInRange() {
            const selected = [...gujuon_data].filter(x =>
                this.settings.enabled_rows.includes(x.row)
                || (this.settings.enabled_rows.includes("(...)") && !this.all_supported_rows.includes(x.row)));
            return selected[Math.floor(Math.random() * selected.length)];
        },
        initGame() {
            this.fillPool();
            this.next();
            this.buildHistoryMatrix();
        },
        setSpeechSynthesis() {
            if ('speechSynthesis' in window) {
                this.speech_synthesis_supported = true;
                this.speech_synthesis_instance = new SpeechSynthesisUtterance();
                this.speech_synthesis_instance.lang = 'ja-JP';
                this.speech_synthesis_instance.rate = 0.5;
            }
        },
        loadSettings() {
            let settings = localStorage.getItem('settings');
            if (settings !== null) {
                this.settings = JSON.parse(settings);
            }
        },
        applyTheme() {
            const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
            const isDark = this.settings.theme_mode === 'dark' || 
                          (this.settings.theme_mode === 'system' && prefersDark);
            
            document.documentElement.classList.toggle('dark-mode', isDark);
        },
        initTheme() {
            // 监听系统主题变化
            window.matchMedia('(prefers-color-scheme: dark)')
                .addEventListener('change', () => this.applyTheme());
            this.applyTheme();
        }
    },
    mounted() {
        this.initGame();
        this.setSpeechSynthesis();
        this.loadSettings();
        this.initTheme();
    },
    setup() {
        const { t } = useI18n({
            inheritLocale: true,
            missingWarn: false,
            fallbackWarn: false,
            // useScope: 'local',
        })

        return { t }
    },
    watch: {
        settings: {
            handler(newVal) {
                localStorage.setItem('settings', JSON.stringify(newVal));
            },
            deep: true
        },
        'settings.theme_mode'() {
            this.applyTheme();
        }
    }
}
</script>

<style>

.large {
    height: min(200px, 30vh);
    width: min(200px, 30vh);
    font-size: min(100px, 15vh);
    vertical-align: middle;
    text-align: center;
}

.sound-input {
    width: min(30%, 300px);
    text-align: center;
    font-size: clamp(20px, 4vw, 30px);
    margin: 0.5rem auto;
}

.sound-input {
    border: none;
    background: transparent;
    padding: 0.25rem;
    min-height: 1.5em;
}

input.sound-input {
    padding: 0.5rem;
    border-radius: 8px;
    border: 1px solid #ccc;
}

.centered {
    margin: 0 auto;
}

.text-centered {
    text-align: center;
}

.correct {
    background-color: #81c784;
    transition: background-color 0.3s ease;
}

.incorrect {
    background-color: #e57373;
    transition: background-color 0.3s ease;
}

details {
    max-width: 800px;
    margin: 0.5rem auto;
    padding: 0.5rem 1rem;
    border-radius: 8px;
    background-color: #f5f5f5;
}

summary {
    cursor: pointer;
    padding: 0.5rem 0.75rem;
    font-weight: 500;
    font-size: 1rem;
    border-radius: 6px;
    transition: background-color 0.2s ease;
}

summary:hover {
    background-color: var(--bg-color);
    opacity: 0.9;
}

table {
    border-collapse: collapse;
    width: 100%;
    max-width: 800px;
    margin: 1rem auto;
}

td, th {
    padding: 0.5rem;
    border: 1px solid #ddd;
}

@media (max-width: 600px) {
    .large {
        height: min(150px, 25vh);
        width: min(150px, 25vh);
        font-size: min(75px, 12vh);
    }
    
    table {
        font-size: 0.9rem;
    }
}

/* 暗色模式样式 */
:root {
    --bg-color: #ffffff;
    --text-color: #000000;
    --details-bg: #f5f5f5;
    --border-color: #ddd;
    --correct-color: #81c784;
    --incorrect-color: #e57373;
}

.dark-mode {
    --bg-color: #121212;
    --text-color: #ffffff;
    --details-bg: #1e1e1e;
    --border-color: #333;
    --correct-color: #2e7d32;
    --incorrect-color: #c62828;
}

/* 应用变量 */
body {
    background-color: var(--bg-color);
    color: var(--text-color);
}

details {
    background-color: var(--details-bg);
}

td, th {
    border-color: var(--border-color);
}

.correct {
    background-color: var(--correct-color);
}

.incorrect {
    background-color: var(--incorrect-color);
}

input.sound-input {
    border-color: var(--border-color);
    background-color: var(--bg-color);
    color: var(--text-color);
}

/* 暗色模式下的过渡效果 */
body, details, td, th, input {
    transition: background-color 0.3s ease, color 0.3s ease, border-color 0.3s ease;
}

/* Section Styles */
.quickstart-section, .settings-section, .history-section, .speed-section, .accuracy-section {
    margin: 0.5rem auto;
    padding: 0.75rem;
    border-radius: 8px;
    background-color: var(--details-bg);
    border: 1px solid var(--border-color);
}

.quickstart-content {
    padding: 0.5rem 0;
}

.settings-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1.5rem;
    padding: 1rem 0;
}

.setting-item {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.setting-label {
    font-weight: 500;
    color: var(--text-color);
    font-size: 0.95rem;
}

.setting-control {
    display: flex;
    align-items: center;
    gap: 0.75rem;
}

.number-input, .text-input, .select-input {
    padding: 0.5rem 0.75rem;
    border: 2px solid var(--border-color);
    border-radius: 6px;
    background-color: var(--bg-color);
    color: var(--text-color);
    font-size: 0.95rem;
    transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.number-input:focus, .text-input:focus, .select-input:focus {
    outline: none;
    border-color: #4CAF50;
    box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.1);
}

.number-input {
    width: 80px;
}

.text-input {
    width: 200px;
}

.select-input {
    min-width: 120px;
    cursor: pointer;
}

.unit {
    font-size: 0.9rem;
    color: var(--text-color);
    opacity: 0.7;
}

/* Switch Toggle */
.switch {
    position: relative;
    display: inline-block;
    width: 50px;
    height: 24px;
}

.switch input {
    opacity: 0;
    width: 0;
    height: 0;
}

.slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    transition: 0.3s;
    border-radius: 24px;
}

.slider:before {
    position: absolute;
    content: "";
    height: 18px;
    width: 18px;
    left: 3px;
    bottom: 3px;
    background-color: white;
    transition: 0.3s;
    border-radius: 50%;
}

input:checked + .slider {
    background-color: #4CAF50;
}

input:checked + .slider:before {
    transform: translateX(26px);
}

/* Checkbox Groups */
.checkbox-group {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
}

.checkbox-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 0.75rem;
    width: 100%;
    max-width: 500px;
}

.checkbox-label {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    cursor: pointer;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    transition: background-color 0.2s ease;
}

.checkbox-label:hover {
    background-color: var(--bg-color);
    opacity: 0.8;
}

.checkbox-label input[type="checkbox"] {
    width: 16px;
    height: 16px;
    accent-color: #4CAF50;
}

.checkbox-text {
    font-size: 0.9rem;
    color: var(--text-color);
}

/* Status Messages */
.warning-text {
    color: #ff9800;
    font-size: 0.85rem;
    margin: 0.25rem 0;
    font-weight: 500;
}

.error-text {
    color: #f44336;
    font-size: 0.85rem;
    margin: 0.25rem 0;
    font-weight: 500;
}

/* Responsive Settings */
@media (max-width: 768px) {
    .settings-grid {
        grid-template-columns: 1fr;
        gap: 1rem;
    }
    
    .setting-control {
        flex-direction: column;
        align-items: flex-start;
        gap: 0.5rem;
    }
    
    .checkbox-grid {
        grid-template-columns: repeat(3, 1fr);
        max-width: 350px;
    }
}

@media (max-width: 480px) {
    .settings-section {
        padding: 0.75rem;
        margin: 0.25rem auto;
    }
    
    .settings-grid {
        padding: 0.5rem 0;
        gap: 0.75rem;
    }
    
    .checkbox-group {
        flex-direction: column;
        gap: 0.5rem;
    }
}
</style>