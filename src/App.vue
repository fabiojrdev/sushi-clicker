<script lang="ts">
import { defineComponent, toRaw } from 'vue'

interface Skill {
  name: string // Nome da habilidade
  interval: number // Intervalo de tempo para aplicar a habilidade
  value: number // Valor que a habilidade adiciona ao countTotal
  cost: number // Custo da habilidade
  status: boolean // Status se a habilidade está ativa ou não
  intervalId: number // ID do intervalo para controle (opcional)
}

interface CountUpMessage {
  skillName: string
  value: number
  id: string // Adicione o id como uma string
}

export default defineComponent({
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
      showCountUp: false,
      countUpValue: 0,
      countUpInterval: 127,
      resetTotal: 0,
      resetMax: 1000,
      resetModal: false,
      sushiReset: 10,
      purchasedSkills: [] as string[],
      skills: [
        { name: 'Hashi', interval: 1, value: 1, cost: 50, status: false, intervalId: 1 },
        { name: 'Shoyu', interval: 1, value: 10, cost: 150, status: false, intervalId: 2 },
        { name: 'Ramen', interval: 1, value: 100, cost: 500, status: false, intervalId: 3 },
        { name: 'Salada', interval: 1, value: 500, cost: 5000, status: false, intervalId: 4 },
        { name: 'Temaki', interval: 1, value: 1000, cost: 100000, status: false, intervalId: 5 },
        { name: 'Shimeji', interval: 1, value: 2000, cost: 500000, status: false, intervalId: 6 }
      ] as Skill[],
      countUpMessages: [] as { skillName: string; value: number; id: number }[]
    }
  },

  methods: {
    async resetSushis() {
      if (this.countTotal < this.sushiReset) {
        alert(
          `Você não tem sushi's suficientes para resetar, você ainda precisa de ${
            this.sushiReset - this.countTotal
          } sushi's para renascer como um verdadeiro Sushi Man`
        )
        return
      }
      this.resetTotal++
      const resetAddedTotal = this.resetTotal
      const resetValue = this.resetTotal.toString()
      localStorage.setItem('resetTotal', resetValue)
      this.countTotal = 0
    },

    async clickCount() {
      if (!this.startGame) {
        const storedCount = localStorage.getItem('countTotal')
        const resetCount = localStorage.getItem('resetTotal')
        const storedSkills = JSON.parse(localStorage.getItem('purchasedSkills') || '[]')
        this.countTotal = parseInt(storedCount || '0', 10)
        this.resetTotal = parseInt(resetCount || '0', 10)
        this.purchasedSkills = storedSkills
        this.startGame = true
        return
      }
      this.isScaled = true
      this.countTotal += this.clickValue
      await new Promise((resolve) => setTimeout(resolve, 200))
      this.isScaled = false
      await this.countUp(true, 'click')
    },

    handleSkill(skill: Skill) {
      if (!this.purchasedSkills.includes(skill.name)) {
        if (this.countTotal >= skill.cost) {
          this.countTotal -= skill.cost
          this.purchasedSkills.push(skill.name)
          this.activateSkill(skill)
        } else {
          alert(
            `Você precisa de mais ${skill.cost - this.countTotal} sushi(s) para adquirir ${skill.name}!`
          )
        }
      } else {
        this.toggleSkill(skill)
      }
    },

    activateSkill(skill: Skill) {
      skill.intervalId = window.setInterval(() => {
        this.countTotal += skill.value
        this.countUpMessages.push({
          skillName: skill.name,
          value: skill.value,
          id: skill.intervalId
        })
      }, 1000)
      skill.status = true
    },

    toggleSkill(skill: Skill) {
      if (skill.status) {
        clearInterval(skill.intervalId)
        skill.status = false
        this.countUpMessages = this.countUpMessages.filter((msg) => msg.skillName !== skill.name)
      } else {
        this.activateSkill(skill)
      }
    },

    async countUp(active: boolean, skillName: string) {
      const arrayNull = [] as any[]
      this.countUpMessages = arrayNull
      if (active) {
        this.showCountUp = true

        if (skillName === 'click') {
          this.countUpValue = this.clickValue
          await new Promise((resolve) => setTimeout(resolve, 500))
          this.showCountUp = false
        } else {
          const skill = this.skills.find((s) => s.name === skillName)
          if (skill) {
            this.countTotal += skill.value
            this.countUpValue = skill.value
            const message = {
              skillName: skill.name,
              value: skill.value,
              id: skill.intervalId
            }
            this.countUpMessages.push(message)

            setTimeout(() => {
              this.countUpMessages = this.countUpMessages.filter((msg) => msg.id)
            }, 2000)
          }
        }
      } else {
        this.showCountUp = false
      }
    },

    delay(ms: number) {
      return new Promise((resolve) => setTimeout(resolve, ms))
    },

    async resetData() {
      this.countTotal = 0
      localStorage.setItem('countTotal', '0')
      this.menuResetData = false
    }
  },

  watch: {
    countTotal(newValue) {
      const totalValueNew = newValue.toString()
      localStorage.setItem('countTotal', totalValueNew)
    },
    purchasedSkills(newSkills) {
      console.log('purchasedSkills mudou para:', newSkills)
      const rawSkills = toRaw(newSkills)
      const skillData = JSON.stringify(rawSkills)
      localStorage.setItem('purchasedSkills', skillData)
    }
  }
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
      <button class="button-menu-secundary" @click="resetSushis(), (menuConfig = false)">
        Renascer
      </button>
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
        <button
          v-for="skill in skills"
          :key="skill.name"
          :class="{ 'active-skill': skill.status, 'desactive-skill': !skill.status }"
          class="skill-container"
          @click="startGame ? handleSkill(skill) : null"
        >
          <p class="title-skill">{{ skill.name }}<br />(+{{ skill.value }})</p>
          <img
            :src="`@/assets/${skill.name.toLowerCase()}.png`"
            class="skill-button-active"
            alt=""
          />
          <p v-if="!skill.status && !purchasedSkills.includes(skill.name)" class="cost-skill">
            Custo: {{ skill.cost }}
          </p>
        </button>
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
        <img
          style="
            height: 20px;
            width: 20px;
            display: flex;
            position: absolute;
            margin-top: 1px;
            margin-left: 8px;
          "
          src="@/assets/sushi.png"
          alt=""
        />
        <h1 class="score-count">{{ countTotal }}</h1>
      </span>
      <p class="title-reset">Você renasceu:</p>
      <div v-for="index in resetTotal" :key="index.valueOf" class="resets">
        <img class="reset-star" src="@/assets/reset-start-full.png" alt="" />
      </div>
      <button class="button-menu-config" @click="menuConfig = !menuConfig"></button>
      <button class="button-menu" @click="menuGlobal = !menuGlobal"></button>
    </div>
    <div class="container-app">
      <div class="app">
        <!-- Mostrar mensagens de contagem -->
        <p class="add-click">+{{ countUpValue }}</p>

        <!-- Área para exibir as habilidades ativas -->
        <div class="skills-container">
          <!-- <img
            v-for="skill in skills"
            :key="skill.name"
            class="rotating-image"
            :class="{ bounce: skill.name === 'Hashi' }"
            :src="`@/assets/${skill.name.toLowerCase()}.png`"
            :alt="skill.name"
          /> -->
        </div>

        <!-- O <span> principal do jogo -->
        <span
          class="sushi-button"
          :class="{ 'click-effect-normal': isScaled }"
          @click="startGame ? clickCount() : null"
        >
          <img class="sushi-image" src="@/assets/sushi.png" alt="" />
        </span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.resets {
  top: 0%;
  display: inline-flex;
  justify-content: center;
  align-items: center;
}
.resets img {
  width: 40px;
  height: 40px;
}
.title-reset {
  top: 2%;
  font-size: 10px;
  color: white;
  position: absolute;
}
.title-skill {
  margin-bottom: 70px;
  font-size: 10px;
  color: white;
  position: absolute;
}
.cost-skill {
  margin-top: 70px;
  font-size: 10px;
  color: white;
  position: absolute;
}
.container-modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 999;
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
  margin-top: -80px;
  padding-bottom: 75px;
  background-size: contain;
  background-repeat: no-repeat;
  background-color: transparent;
}
.menu-modal .close-button-menu {
  position: absolute;
  right: 3%;
  top: 3%;
  font-size: 22px;
  color: rgb(255, 0, 0);
}
.magias {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  padding-top: 0px;
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
  width: 40px;
  height: 40px;
  top: 48%;
  left: 50%;
  transform: translate(-50%, -50%);
  animation: rotate 10s linear infinite;
}
@keyframes rotate {
  from {
    transform: rotate(0deg) translateX(90px) rotate(0deg);
  }
  to {
    transform: rotate(360deg) translateX(90px) rotate(-360deg);
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
  padding-left: 30px;
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
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 5px;
  border-radius: 5px;
  margin-bottom: 25px;
  border: none;
}
.active-skill {
  background-image: url('@/assets/skill-on.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-color: transparent;
}
.desactive-skill {
  background-image: url('@/assets/skill-off.png');
  background-size: cover;
  background-repeat: no-repeat;
  background-color: transparent;
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
