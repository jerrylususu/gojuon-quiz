<template>
<details>
    <summary>{{ t('quickstart_title') }}</summary>

<p>{{ t('quickstart_content.0') }}</p>
<p>{{ t('quickstart_content.1') }}</p>
<p>{{ t('quickstart_content.2') }}</p>
<p>{{ t('quickstart_content.3') }}<a href="https://github.com/jerrylususu/gojuon-quiz" target="_blank" >Github</a></p>
<p>{{ t('quickstart_content.4') }}</p>

</details>
<details>
    <summary>{{ t('settings_title') }}</summary>
    {{ t('settings.input_wait') }} <input type="number" v-model="this.settings.correct_wait_ms" /> ms
    <br />
    Language:  <select v-model="$i18n.locale">
      <option v-for="(lang, i) in langs" :key="`Lang${i}`" :value="lang">
        {{ lang }}
      </option>
    </select>
    <br />
    {{ t('settings.font') }}: <input type="text" v-model="this.settings.font_css_str"/>
    <br />
    {{ t('settings.test_on')}}: 
    {{ t('hiragana')}} <input type="checkbox" v-model="hiragana_enabled">
    {{ t('katakana')}} <input type="checkbox" v-model="katakana_enabled">
    <p v-if="!hiragana_enabled && !katakana_enabled" style="color: red">{{ t('settings.nothing_can_be_shown')}}</p>
</details>

<details>
    <summary>{{ t('history_title') }}</summary>
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
123
<details>
    <summary>{{ t('speed_stat') }} (ms)</summary>
    <table>
        <tr v-for="(row, hidx) in this.history" :key="hidx">
            <td v-for="(item, vidx) in row" :key="vidx" :style="this.speed_style(item)">
            <span v-if="item.hiragana !== '' ">{{item.hiragana}}{{item.avg_time_ms.toFixed(0)}}</span>
                
            </td>
        </tr>
    </table>
</details>

<details>
    <summary>{{ t('accuracy_stat') }}</summary>
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
                correct_wait_ms: 1000,
                font_css_str: 'serif, sans-serif'
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
            // console.log("Reveal")
            this.show_hiragana = true;
            this.show_katakana = true;
            this.show_sound = true;
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
                next_item = gujuon_data[Math.floor(Math.random() * gujuon_data.length)];
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
            this.pool = [...gujuon_data];
        }
    },
    mounted() {
        this.fillPool()
        this.next()
        this.buildHistoryMatrix()
    },
    setup() {
        const { t } = useI18n({
            inheritLocale: true,
            missingWarn: false,
            fallbackWarn: false,
            // useScope: 'local',
        })

        return { t }
  }
}
</script>

<style>

.large {
    height: 200px;
    width: 200px;
    font-size: 100px;
    vertical-align: middle;
    text-align: center;
}

.sound-input {
    width: 30%;
    text-align: center;
    font-size: 30px;
}

.centered {
    margin: 0 auto;
}

.text-centered {
    text-align: center;
}

.correct {
    background-color: #81c784;
}

.incorrect {
    background-color: #e57373;
}
</style>