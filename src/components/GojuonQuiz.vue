<template>
<details>
    <summary>使用说明</summary>

<p>随机显示一个平假名或片假名，需要输入其对应的发音。</p>
<p>输入正确后输入框背景会变为绿色，可以按下回车进入下一题，但稍等一会也会自动进入下一题。输入错误后输入框背景变为红色，需要按下回车手动进入下一题。</p>
<p>仅正确输入会计入时间和准确率。首次输入不计入时间和准确率。数值颜色越绿表示越好（时间更短、准确率越高）。</p>
<p>点个 Star 吧：<a href="https://github.com/jerrylususu/gojuon-quiz" target="_blank" >Github</a></p>

</details>
<details>
    <summary>设置</summary>
    输入正确后等待时间（ms）：<input type="number" v-model="this.settings.correct_wait_ms" />
</details>

<details>
    <summary>历史数据</summary>
    总体正确率：{{this.total_accruacy_str}} ({{this.stats.total_correct}}/{{this.stats.total_tested}}) 平均用时: {{this.average_time_str}}
    最近10个 
    <span v-for="item in this.stats.last_10.slice().reverse()" :key="item.enrolled_time" :class="{correct: item.isCorrect, incorrect: !item.isCorrect}">  
        {{item.hiragana}}{{item.katakana}}
    </span>
</details>


<div class="wrapper">
<table class="centered">
  <tr>
    <th>平假名</th>
    <th>片假名</th>
    <th>读音</th>
  </tr>
  <tr>
    <td class="large">{{this.display_hiragana}}</td>
    <td class="large">{{this.display_katakana}}</td>
    <td class="large">{{this.display_sound}}</td>
  </tr>
</table>

<input class="sound-input centered" :class="inputBackgroundClass" @input="onInput" v-on:keyup.enter="onEnter" v-model="this.inputValue" type="text"/>
</div>

<details>
    <summary>速度统计（ms）</summary>
    <table>
        <tr v-for="(row, hidx) in this.history" :key="hidx">
            <td v-for="(item, vidx) in row" :key="vidx" :style="this.speed_style(item)">
            <span v-if="item.hiragana !== '' ">{{item.hiragana}}{{item.avg_time_ms.toFixed(0)}}</span>
                
            </td>
        </tr>
    </table>
</details>

<details>
    <summary>正确率统计</summary>
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
            },
            history: [],
            hiragana_to_history: {},
        }
    },
    computed: {
        correct() {
            return this.current.sound.split(",").includes(this.inputValue)
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
            if(this.correct) {
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
                    console.log(cell)
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
            // console.log("next")
            // choose one random item from gujuon_data
            const random_index = Math.floor(Math.random() * gujuon_data.length);
            const next_item = gujuon_data[random_index];
            this.current.sound = next_item.sound;
            this.current.hiragana = next_item.hiragana;
            this.current.katakana = next_item.katakana;

            // random hide hiragana or katakana
            if (Math.random() > 0.5) {
                this.show_hiragana = false;
                this.show_katakana = true;
            } else {
                this.show_katakana = false;
                this.show_hiragana = true;
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
        }
    },
    mounted() {
        this.next()
        this.buildHistoryMatrix()
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

.wrapper {
    text-align: center;
}

.centered {
    margin: 0 auto;
}

.correct {
    background-color: #81c784;
}

.incorrect {
    background-color: #e57373;
}
</style>