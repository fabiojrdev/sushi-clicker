<script lang="ts">
import { defineComponent, onMounted } from 'vue'

// Exemplo de interface
// interface Sushi {
//   Nome: string
//   Coins: string
// }

export default defineComponent({
  components: {},
  data() {
    return {
      startGame: false,
      menuGlobal: false,
      menuConfig: false,
      menuMagias: false,
      menuResetData: false,
      countTotal: 0,
      countTotalSaved: '',
      clickBonus: 0,
      clickValue: 1,
      isScaled: false,
      hashiSkill: false,
      hashiInterval: 1,
      hashiSkillValue: 1,
      showCountUp: false,
      countUpValue: 0,
      countUpInterval: 127,
      shoyuSkill: false,
      shoyuInterval: 1,
      shoyuSkillValue: 10
    }
  },

  methods: {
    async clickCount() {
      if (!this.startGame) {
        const storedCount = localStorage.getItem('countTotal')
        this.countTotal = parseInt(storedCount || '0', 10)
        this.startGame = true
      }
      this.isScaled = true
      this.countTotal += this.clickValue
      await setTimeout(() => {
        this.isScaled = false
      }, 200)
      await this.countUp(true, 'click')
    },
    async skillHashi() {
      if (this.hashiSkill) {
        this.hashiSkill = false
        this.countUp(false, 'hashi')
        clearInterval(this.hashiInterval)
        return
      }
      this.hashiSkill = true
      this.hashiInterval = setInterval(() => {
        this.countUp(true, 'hashi')
      }, 1000)
    },
    async skillShoyu() {
      if (this.shoyuSkill) {
        this.shoyuSkill = false
        this.countUp(false, 'shoyu')
        clearInterval(this.shoyuInterval)
        return
      }
      this.shoyuSkill = true
      this.shoyuInterval = setInterval(() => {
        this.countUp(true, 'shoyu')
      }, 1000)
    },
    delay(ms: number) {
      return new Promise((resolve) => setTimeout(resolve, ms))
    },
    async countUp(params: boolean, skill: string) {
      if (params === true) {
        this.showCountUp = true

        switch (skill) {
          case 'hashi':
            this.countTotal += this.hashiSkillValue
            this.countUpValue = this.hashiSkillValue
            break
          case 'click':
            this.countUpValue = this.clickValue
            await this.delay(500)
            this.showCountUp = false
            break
          case 'shoyu':
            this.countTotal += this.shoyuSkillValue
            this.countUpValue = this.shoyuSkillValue
            break
        }
      } else {
        this.showCountUp = false
      }
    },
    async resetData() {
      this.countTotal = 0
      localStorage.setItem('countTotal', '0')
      this.menuResetData = false
      return
    }
  },
  watch: {
    countTotal(newValue) {
      console.log(this.countTotal)
      const totalValueNew = newValue.toString()
      console.log('Salvou')
      return localStorage.setItem('countTotal', totalValueNew)
    }
  },
  mounted() {}
})
</script>

<template>
  <div class="logo-game">
    <img class="skill-button-active" src="@/assets/logo.png" alt="" />
  </div>

  <div v-if="menuGlobal" class="container-modal">
    <div class="menu-modal">
      <div class="title">Menu</div>
      <span class="close-button-menu" @click="menuGlobal = false">X</span>
      <button class="button-menu-secundary" @click="(menuMagias = true), (menuGlobal = false)">
        Magias
      </button>
    </div>
  </div>

  <div v-if="menuResetData" class="container-modal">
    <div class="menu-modal">
      <div class="title">Você tem certeza que gostaria de deletar todos os dados?</div>
      <span class="close-button-menu" @click="menuResetData = false">X</span>
      <div class="confirm-buttons">
        <button class="button-menu-secundary" @click="menuResetData = false">Não</button>
        <button class="button-menu-secundary" @click="resetData()">Sim</button>
      </div>
    </div>
  </div>

  <div v-if="menuConfig" class="container-modal">
    <div class="menu-modal">
      <div class="title">Config</div>
      <span class="close-button-menu" @click="menuConfig = false">X</span>
      <button class="button-menu-secundary" @click="(menuResetData = true), (menuConfig = false)">
        Resetar dados
      </button>
    </div>
  </div>

  <div v-if="menuMagias" class="container-modal">
    <div class="menu-modal">
      <div class="title">Magias</div>
      <span class="close-button-menu" @click="menuMagias = false">X</span>
      <div class="magias">
        <span
          :class="{ 'active-skill': hashiSkill, 'desactive-skill': !hashiSkill }"
          class="skill-container"
          @click="startGame ? skillHashi() : null"
        >
          <img class="skill-button-active" src="@/assets/hashi.png" alt="" />
        </span>
        <span
          :class="{ 'active-skill': shoyuSkill, 'desactive-skill': !shoyuSkill }"
          class="skill-container"
          @click="startGame ? skillShoyu() : null"
        >
          <img class="skill-button-active" src="@/assets/shoyu.png" alt="" />
        </span>
      </div>
      <button class="button-menu-secundary" @click="(menuMagias = false), (menuGlobal = true)">
        Voltar
      </button>
    </div>
  </div>
  <img
    v-if="!startGame"
    :class="{ 'click-effect-normal': isScaled }"
    @click="!startGame ? clickCount() : null"
    class="start-screen"
    src="@/assets/start-screen.jpg"
    alt=""
  />
  <p v-if="!startGame" class="start-text">Clique para iniciar o jogo</p>
  <div v-if="startGame" class="game">
    <div class="notification"></div>
    <div class="score-bar">
      <p class="title-count">Sushi's:</p>
      <span class="value-count">
        <h1 class="score-count">{{ countTotal }}</h1>
      </span>
      <button class="button-menu-config" @click="menuConfig = !menuConfig"></button>
      <button class="button-menu" @click="menuGlobal = !menuGlobal"></button>
    </div>
    <div class="container-app">
      <div class="app">
        <span
          class="sushi-button"
          :class="{ 'click-effect-normal': isScaled }"
          @click="startGame ? clickCount() : null"
        >
          <p v-if="showCountUp" class="add-click">+{{ countUpValue }}</p>
          <img
            v-if="hashiSkill"
            class="rotating-image bounce skill-button-active"
            src="@/assets/hashi.png"
            alt=""
          />
          <img class="sushi-image" src="@/assets/sushi.png" alt="" />
        </span>
      </div>
    </div>
  </div>
  <!-- <RouterView /> -->
</template>

<style scoped>
.container-modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  /* background-color: rgba(0, 0, 0, 0.5); */
  z-index: 999;
  /* pointer-events: none; */
}
.button-menu-config {
  right: 20%;
  position: absolute;
  background-image: url('@/assets/config-menu.png');
  background-size: contain;
  background-color: transparent;
  width: 45px;
  height: 45px;
  font-size: 22px;
  text-transform: uppercase;
  color: white;
  border: none;
  background-repeat: no-repeat;
}
.button-menu-secundary {
  background-image: url('@/assets/button-menu.png');
  background-size: contain;
  background-color: transparent;
  padding: 5px;
  width: 137px;
  height: 50px;
  font-size: 18px;
  font-weight: bolder;
  text-transform: uppercase;
  color: rgb(121, 60, 14);
  border: none;
  background-repeat: no-repeat;
}
.button-menu {
  right: 5%;
  position: absolute;
  background-image: url('@/assets/menu-global.png');
  background-size: contain;
  background-color: transparent;
  width: 45px;
  height: 45px;
  font-size: 22px;
  text-transform: uppercase;
  color: white;
  background-repeat: no-repeat;
  border: none;
}
.menu-modal {
  flex-direction: column;
  display: flex;
  /* justify-content: center; */
  align-items: center;
  max-width: 500px;
  min-width: 325px;
  max-height: 500px;
  min-height: 300px;
  padding: 20px;
  border-radius: 15px;
  background-image: url('@/assets/menu-background.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-color: transparent;
}
.menu-modal .title {
  font-size: 22px;
  color: white;
  text-transform: uppercase;
  padding-bottom: 20px;
}
.menu-modal .close-button-menu {
  position: absolute;
  right: 3%;
  top: 3%;
  font-size: 22px;
  color: rgb(255, 0, 0);
}
.magias {
  display: flex;
  padding-top: 20px;
  gap: 10px;
}
.logo-game {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  top: 10%;
}
.logo-game img {
  max-width: 600px;
  width: 100%;
  height: auto;
  object-fit: contain;
  left: 0%;
}
.start-screen {
  position: absolute;
  top: 0%;
  left: 0%;
  justify-content: center;
  align-items: center;
  max-width: 100vw;
  min-width: auto;
  min-height: auto;
  z-index: 99999;
  display: flex;
  overflow: hidden;
}
.start-text {
  top: 80%;
  position: absolute;
  justify-content: center;
  align-items: center;
  font-size: 22px;
  color: white;
  text-transform: uppercase;
  z-index: 999999;
  pointer-events: none;
}
.add-click {
  color: gold;
  font-size: 12px;
  animation: bounce-out-down 1s ease infinite;
}
.rotating-image {
  position: absolute;
  width: 60px;
  height: 60px;
  top: 48%;
  left: 50%;
  transform: translate(-50%, -50%);
  animation: rotate 10s linear infinite;
}
@keyframes rotate {
  from {
    transform: rotate(0deg) translateX(250px) rotate(0deg);
  }
  to {
    transform: rotate(360deg) translateX(600px) rotate(-360deg);
  }
}
.bounce {
  animation: bounce 1s ease infinite;
}
@keyframes bounce {
  70% {
    transform: translateY(0%);
  }
  80% {
    transform: translateY(-15%);
  }
  90% {
    transform: translateY(0%);
  }
  95% {
    transform: translateY(-7%);
  }
  97% {
    transform: translateY(0%);
  }
  99% {
    transform: translateY(-3%);
  }
  100% {
    transform: translateY(0);
  }
}
.score-bar {
  position: absolute;
  display: grid;
  align-items: center;
  width: 100vw;
  padding: 20px;
  background-color: rgba(0, 0, 0, 0.593);
  z-index: 999;
  bottom: 0%;
  left: 0%;
  grid-template-columns: 1fr 1fr 1fr;
}
.title-count {
  top: 5%;
  left: 5%;
  position: absolute;
  font-size: 10px;
  color: white;
}
.value-count {
  width: 100%;
  height: 30px;
  background-image: url('@/assets/count-value.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-color: transparent;
}
.score-count {
  padding-left: 10px;
  color: white;
  font-size: 16px;
}

.container-app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0 auto;
}
.sushi-button {
  transition: transform 0.3s ease-in-out;
  background-color: transparent;
}
.sushi-image {
  width: 150px;
  height: auto;
}
.click-effect-normal .sushi-image {
  transform: scale(1.25);
}
.skill-container {
  width: 30px;
  height: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 5px;
  border-radius: 5px;
}
.active-skill {
  background-color: rgb(58, 183, 49);
  border: 2px solid rgb(58, 183, 49);
}
.desactive-skill {
  background-color: rgb(255, 187, 0);
  border: 2px solid rgb(162, 129, 35);
}
.skill-button-active {
  width: 30px;
}

@keyframes bounce-out-down {
  0% {
    transform: translateY(0);
  }
  20% {
    opacity: 1;
    transform: translateY(-20px);
  }
  100% {
    opacity: 0;
    transform: translateY(20px);
  }
}

.rotate-in-down-left {
  animation: rotate-in-down-left 2s ease infinite;
}
</style>
