<script>
import button1Audio from "@/assets/1.mp3";
import button2Audio from "@/assets/2.mp3";
import button3Audio from "@/assets/3.mp3";
import button4Audio from "@/assets/4.mp3";

const timeouts = { easy: 1500, normal: 1000, hard: 400 };
const lightTimeouts = { click: 100, auto: 350 };
const audio1 = new Audio(button1Audio);
const audio2 = new Audio(button2Audio);
const audio3 = new Audio(button3Audio);
const audio4 = new Audio(button4Audio);
const sounds = {
  button1: audio1,
  button2: audio2,
  button3: audio3,
  button4: audio4,
};

export default {
  data() {
    return {
      roundNumber: 1,
      currentRound: 0,
      gameRow: [],
      blockButtons: true,
      blockDifficulty: false,
      difficulty: "normal",
      answer: [],
      isFinite: false,
      button1State: false,
      button2State: false,
      button3State: false,
      button4State: false,
      didLose: false,
      currentAudio: null,
    };
  },
  computed: {
    getLoseRound() {
      return ((this.currentRound - 1) ? (this.currentRound - 1) : this.currentRound);
    },
  },
  methods: {
    generateGameRow() {
      this.gameRow = [];
      for (let i = 0; i < this.roundNumber; i = i + 1) {
        this.gameRow.push(Math.floor(Math.random() * 4) + 1);
      }
    },

    lightOff(eventType) {
      window.setTimeout(
        function () {
          this.button1State = false;
          this.button2State = false;
          this.button3State = false;
          this.button4State = false;
        }.bind(this),
        lightTimeouts[eventType]
      );
    },

    stopAudio(audio) {
      audio.pause();
      audio.currentTime = 0;
    },

    round() {
      if (this.currentRound < this.roundNumber) {
        // this.didLose = false;
        if (!this.currentAudio) {
          this.currentAudio = `button${this.gameRow[this.currentRound]}`;
        }
        this.stopAudio(sounds[this.currentAudio]);
        this.currentAudio = `button${this.gameRow[this.currentRound]}`;
        sounds[this.currentAudio].play();
        this[`${this.currentAudio}State`] = true;
        this.currentRound = this.currentRound + 1;
      } else {
        clearInterval(this.timer);
      }
    },

    start() {
      this.didLose = false;
      this.blockDifficulty = true;
      if (!this.isFinite) {
        this.roundNumber = 1;
      }
      this.isFinite = false;
      this.generateGameRow();
      this.currentRound = 0;
      this.blockButtons = true;
      window.setTimeout(
        function () {
          this.blockButtons = false;
        }.bind(this),
        timeouts[this.difficulty] * this.roundNumber + 50
      );
      this.round();
      this.lightOff("auto");
      this.timer = setInterval(
        function () {
          this.lightOff("auto");
          this.round();
        }.bind(this),
        timeouts[this.difficulty]
      );
    },

    click(event) {
      if (!this.currentAudio) {
        this.currentAudio = `button${event.target.id}`;
      }
      this.stopAudio(sounds[this.currentAudio]);
      this.currentAudio = `button${event.target.id}`;
      sounds[this.currentAudio].play();
      this[`${this.currentAudio}State`] = true;
      this.lightOff("click");
      this.answer.push(event.target.id);
      if (
        Number(this.answer[this.answer.length - 1]) !==
        Number(this.gameRow[this.answer.length - 1])
      ) {
        this.answer = [];
        this.roundNumber = 1;
        this.blockButtons = !this.blockButtons;
        this.isFinite = false;
        this.didLose = true;
        this.blockDifficulty = false;
        return;
      }
      if (this.answer.length === this.gameRow.length) {
        this.answer = [];
        this.roundNumber = this.roundNumber + 1;
        this.blockButtons = !this.blockButtons;
        this.isFinite = true;
      }
    },
  },
};
</script>

<template>
  <div class="wrapper">
    <h1>Simon Says</h1>
    <div class="inner">
      <div class="buttonsField">
        <div>
          <button
            class="button1"
            v-bind:class="{ active: button1State }"
            v-bind:disabled="blockButtons"
            v-on:click="click($event)"
            id="1"
          />
          <button
            class="button2"
            v-bind:class="{ active: button2State }"
            v-bind:disabled="blockButtons"
            v-on:click="click($event)"
            id="2"
          />
        </div>
        <div>
          <button
            class="button3"
            v-bind:class="{ active: button3State }"
            v-bind:disabled="blockButtons"
            v-on:click="click($event)"
            id="3"
          />
          <button
            class="button4"
            v-bind:class="{ active: button4State }"
            v-bind:disabled="blockButtons"
            v-on:click="click($event)"
            id="4"
          />
        </div>
      </div>
      <div class="roundField">
        <h2>Раунд: {{ roundNumber }}</h2>
        <button v-on:click="start">Новый раунд</button>
        <p v-bind:class="{ hidden: !didLose }">Вы проиграли после {{ getLoseRound }} раунда</p>
        <fieldset v-bind:disabled="blockDifficulty">
          <legend>Сложность:</legend>
          <div>
            <input
              type="radio"
              id="easy"
              name="difficulty"
              value="easy"
              v-model="difficulty"
            />
            <label for="easy">легко</label>
          </div>
          <div>
            <input
              type="radio"
              id="normal"
              name="difficulty"
              value="normal"
              checked
              v-model="difficulty"
            />
            <label for="normal">средне</label>
          </div>
          <div>
            <input
              type="radio"
              id="hard"
              name="difficulty"
              value="hard"
              v-model="difficulty"
            />
            <label for="hard">сложно</label>
          </div>
        </fieldset>
      </div>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  width: 75vw;
  border-radius: 50px;
  padding-top: 50px;
  margin-top: 20px;
  margin-bottom: 20px;
  padding-bottom: 20px;
  background: #1f1f1f;
  text-align: center;
  color: #fd850a;
}

.wrapper h1 {
  margin-bottom: 40px;
}

.wrapper h2 {
  margin-bottom: 20px;
}

.inner {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  margin: auto;
}

.buttonsField {
  display: flex;
  flex-wrap: nowrap;
  justify-content: center;
  flex-direction: column;
  padding: auto;
}

fieldset {
  text-align: center;
  width: 150px;
  padding: 5px 0px;
  font-size: 18px;
}

fieldset div {
  text-align: left;
}

fieldset input {
  margin-left: 28%;
  margin-right: 2px;
}

.roundField {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 40%;
  padding: auto;
}

.roundField button {
  width: 150px;
  height: 50px;
  font-size: 18px;
  border-radius: 12px;
  margin-bottom: 20px;
  background: #5b2915;
  color: #f7efed;
  border: 2px solid #db6b3d;
  cursor: pointer;
}

.roundField p {
  margin-bottom: 20px;
}

.buttonsField button {
  margin-left: 1px;
  margin-right: 1px;
}

.button1 {
  width: 200px;
  height: 200px;
  background: #137eea;
  border-radius: 95% 0px 0px 0px;
  cursor: pointer;
}

.button1:disabled {
  background: #08396a;
  cursor: default;
}

.button2 {
  width: 200px;
  height: 200px;
  background: #e6ba2a;
  border-radius: 0px 95% 0px 0px;
  cursor: pointer;
}

.button2:disabled {
  background: #a9891f;
  cursor: default;
}

.button3 {
  width: 200px;
  height: 200px;
  background: #3def3a;
  border-radius: 0px 0px 0px 95%;
  cursor: pointer;
}

.button3:disabled {
  background: #1e791c;
  cursor: default;
}

.button4 {
  width: 200px;
  height: 200px;
  background: #e33aab;
  border-radius: 0px 0px 95% 0px;
  cursor: pointer;
}

.button4:disabled {
  background: #8b2369;
  cursor: default;
}

.hidden {
  display: none;
}

.active {
  background: red;
  transition: transform 500ms ease;
  transform: scale(1.02);
}

.active:disabled {
  background: red;
}
</style>
