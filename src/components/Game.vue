<template>
  <div id="overlay" v-if="dontStartedYet">
    <button type="button" class="startBtn" @click="startGame">
      PLAY
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
        <path d="M13 2.004c5.046.504 9 4.783 9 9.97 0 1.467-.324 2.856-.892 4.113l1.738 1.005c.732-1.553 1.154-3.284 1.154-5.117 0-6.304-4.842-11.464-11-11.975v2.004zm-10.109 14.083c-.568-1.257-.891-2.646-.891-4.112 0-5.188 3.954-9.466 9-9.97v-2.005c-6.158.511-11 5.671-11 11.975 0 1.833.421 3.563 1.153 5.118l1.738-1.006zm17.213 1.734c-1.817 2.523-4.769 4.174-8.104 4.174s-6.288-1.651-8.105-4.175l-1.746 1.01c2.167 3.123 5.768 5.17 9.851 5.17 4.082 0 7.683-2.047 9.851-5.168l-1.747-1.011zm-8.104-13.863c-4.419 0-8 3.589-8 8.017s3.581 8.017 8 8.017 8-3.589 8-8.017-3.581-8.017-8-8.017zm-2 11.023v-6.013l6 3.152-6 2.861z"/>
      </svg>
    </button>
  </div>
  <div class="board">
    <div data-pad="0" @click="newTry"></div>
    <div data-pad="1" @click="newTry"></div>
    <div data-pad="2" @click="newTry"></div>
    <div data-pad="3" @click="newTry"></div>
  </div>
  <div id="text">
    <p :class="isAlertText ? 'red-text' : ''">{{ text }}</p>
  </div>
</template>

<script>
import {Howl} from 'howler';


export default {
  name: 'Game',
  data() {
    return {
      path: [],
      tries: [],
      step: 0,
      dontStartedYet: true,
      text: '',
      isAlertText: false,
      canTry: true,
    }
  },
  methods: {
    verifyPadId(padId) {
      try {
        return parseInt(padId);
      } catch (e) {
        alert('There is an error with an input, the page will be reloaded');
        window.location.reload();
        return null;
      }
    },
    simulateClick(el) {
      el.classList.add('active');
      setTimeout(() => {
        el.classList.remove('active');
      }, 100);
    },
    getAudioSource(search) {
      let sound;

      switch (search) {
        case 0:
          sound = new Howl({
            src: [require('@/assets/sounds/c.ogg')],
            volume: 0.2,
          });
          break;
        case 1:
          sound = new Howl({
            src: [require('@/assets/sounds/d.ogg')],
            volume: 0.2,
          });
          break;
        case 2:
          sound = new Howl({
            src: [require('@/assets/sounds/e.ogg')],
            volume: 0.2,
          });
          break;
        case 3:
          sound = new Howl({
            src: [require('@/assets/sounds/f.ogg')],
            volume: 0.2,
          });
          break;
        case 'fail':
          sound = new Howl({
            src: [require('../assets/sounds/fail.ogg')],
            volume: 0.2,
          });
          break;
        default:
          sound = null;
          break;
      }

      return sound;
    },
    getRandomInt(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    displayText(text) {
      this.text = text;
      const textEl = document.querySelector('#text');

      textEl.classList.add('show');
      setTimeout(() => {
        textEl.classList.remove('show');
        if (this.isAlertText) {
          this.isAlertText = false;
        }
      }, 2000)
    },
    startGame() {
      this.dontStartedYet = false;
      this.addNextPath();

      setTimeout(() => {
        this.showPath()
      }, 1000);
    },
    addNextPath() {
      const number = this.getRandomInt(0, 3);
      this.path.push(number);

      console.log("Level: " + this.path.length);
    },
    showPath() {
      const divs = document.querySelectorAll('.board div');
      let index = 0;
      this.canTry = false;

      const interval = setInterval(() => {
        const div = divs[this.path[index]];

        // Toggle 'active' class
        this.simulateClick(div);

        // Play sound depending of the pad
        const divId = this.verifyPadId(div.getAttribute('data-pad'));
        const sound = this.getAudioSource(divId);
        sound.play();

        // Clear interval at the end
        if (index >= (this.path.length - 1)) {
          clearInterval(interval);
          this.canTry = true;
        } else {
          index++;
        }

      }, 1000);
    },
    newTry(event) {
      if (!this.canTry) {
        return;
      }

      const pad = this.verifyPadId(event.target.getAttribute('data-pad'));
      const comparison = this.path[this.step];

      if (comparison === pad) {
        // Find correct pad
        this.tries.push(pad);

        // Play Sound
        const sound = this.getAudioSource(pad);
        sound.play();

        this.simulateClick(event.target);
      } else {
        // Fuck** up man
        this.endGameAndResetVariables();
      }

      this.step++;
      if (this.step >= this.path.length) {
        // End of round
        this.addNextPath(); // Add new pad
        this.step = 0;

        setTimeout(() => {
          this.showPath(); // Show path again
        }, 1000);
      }
    },
    endGameAndResetVariables() {
      // Reset variables
      this.tries = [];
      this.step = 0;
      this.path = [];

      // Play Sound
      const sound = this.getAudioSource('fail');
      sound.play();

      // Display text
      this.isAlertText = true;
      this.displayText('You failed! Try again :P');
    },
  }
}
</script>

<style scoped>
.red-text {
  color: #df2424 !important;
}


#text {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 6;

  width: 100vw;
  height: 100vh;

  display: flex;
  justify-content: center;
  align-items: center;

  opacity: 0;
  transform: translateY(-10px);

  pointer-events: none;
}
#text.show {
  animation: animate-text 2s ease-in-out 1 forwards;
}
#text p {
  color: #fff;
  -webkit-text-stroke-width: 1px;
  -webkit-text-stroke-color: black;

  font-size: 4em;

  text-transform: uppercase;
  font-weight: bold;
}
@keyframes animate-text {
  0% {
    transform: translateY(-10px);
  }
  10% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  90% {
    opacity: 0;
  }
  100% {
    transform: translateY(10px);
  }
}


#overlay{
  position: absolute;
  top: 0;
  left: 0;
  z-index: 3;

  width: 100vw;
  height: 100vh;

  background: rgba(0, 0, 0, 0.4);
}
.startBtn {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);

  z-index: 4;

  width: 30vw;

  padding: 0.3em 0.2em;

  display: flex;
  justify-content: center;
  align-items: center;

  background: #4f46e5;
  border: none;
  border-radius: 50px / 50%;
  outline: none;
  box-shadow: 0 8px 17px 2px rgba(0, 0, 0, 0.14), 0 3px 14px 2px rgba(0, 0, 0, 0.12), 0 5px 5px -3px rgba(0, 0, 0, 0.2);

  font-size: 3em;
  color: #fff;

  cursor: pointer;

  transition: background 0.15s ease;

  animation: animate-startbtn 1s ease-in-out infinite alternate forwards;
}
.startBtn:hover {
  background: #4338ca;
}
.startBtn > svg {
  height: 1.4em;
  width: 1.4em;

  fill: #fff;

  margin-left: 0.3em;

  animation: animate-startbtn-svg 3s cubic-bezier(.46,-0.4,.21,.98) infinite forwards;
}
@keyframes animate-startbtn {
  from {
    transform: translate(-50%, -50%) scale(1);
  }
  to {
    transform: translate(-50%, -50%) scale(1.1);
  }
}
@keyframes animate-startbtn-svg {
  0% {
    transform: rotate(0);
  }
  60% {
    transform: rotate(360deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.board {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;

  z-index: 2;

  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  gap: 0 0;
  grid-template-areas:
    ". ."
    ". .";
}

.board div {
  cursor: pointer;
  transition: background 0.15s ease;
}

.board div:nth-child(1) {
  background: #ef4444;
}
.board div:nth-child(1).active {
  background: #fca5a5;
}

.board div:nth-child(2) {
  background: #3b82f6;
}
.board div:nth-child(2).active {
  background: #93c5fd;
}

.board div:nth-child(3) {
  background: #eab308;
}
.board div:nth-child(3).active {
  background: #fde047;
}

.board div:nth-child(4) {
  background: #22c55e;
}
.board div:nth-child(4).active {
  background: #86efac;
}
</style>
