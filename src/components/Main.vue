<template>
  <div
    class="main mt-1 w-100 h-100 d-flex flex-column"
  >
    <div class="desktop-only-warning container"><h3>This demo is for desktop only</h3></div>
    <div
      class="container content w-100 h-100 row mx-auto"
    >
      <h1 class="fs-3 mx-auto text-center" :class="{ 'invisible': turn && !turn.game_over }">Game over!</h1>
      <div
        class="dinosaur col shadow"
        :class="{ 'type': dino.type, 'border border-4 border-success': (turn.attacker === dino.id && turn.game_over) }"
        v-for="dino in this.dinosaurs"
        :key="dino.id"
      >
        <Dinosaur @update="updateProps" @game-over="setGameOver" :turn="turn" :dino="dino" />
      </div>
    </div>
    <div
      class="d-flex justify-content-center"
    >
      <span class="py-5 px-3">Bite | Attack: 3 | Intimidation: 5</span>
      <span class="py-5 px-3">Roar | Attack: 2 | Intimidation: 8</span>
      <span class="py-5 px-3">Stomp | Attack: 1 | Intimidation: 12</span>
    </div>
    <button class="btn mx-auto my-5 bg-success" @click="reset">Reset</button>
  </div>
</template>

<script>
import Dinosaur from './Dinosaur.vue'

export default {
  async created() {
    this.turn = await this.getTurn()
    this.dinosaurs = await this.getDinosaurs()
  },
  name: 'Main',
  components: {
    Dinosaur
  },
  data: function() {
    return {
      turn: null,
      dinosaurs: null
    }
  },
  methods: {
    getTurn: async function() {
      const res = await fetch('http://localhost:3000/turn')
      return await res.json()
    },
    getDinosaurs: async function() {
      const res = await fetch('http://localhost:3000/dinosaurs')
      return await res.json()
    },
    restoreAttacker: async function(attacker) {
      let restore = {
        ...attacker,
        health: attacker.health <= 48 ? attacker.health + 2 : attacker.health,
        confidence: attacker.confidence <= 45 ? attacker.confidence + 3 : attacker.confidence
      }

      const res = await fetch(`http://localhost:3000/dinosaurs/${attacker.id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(restore)
      })
      return await res.json()
    },
    getAttacker: async function(id) {
      const res = await fetch(`http://localhost:3000/dinosaurs/${id}`)
      return await res.json()
    },
    setGameOver: function(turn) {
      this.turn = turn
    },
    updateProps: async function(update) {
      const attacker = await this.getAttacker(this.turn.attacker)
      const attacking = update?.attacking
      const turn = update?.turn
      this.dinosaurs.forEach((d, i) => {
        if (attacking && attacking.id === d.id) {
          this.dinosaurs.splice(i, 1, attacking)
        }
      })

      if (turn) {
        if (turn.moves_left) {
          this.turn = turn
        } else {
          const res = await fetch(`http://localhost:3000/turn`, {
            method: 'PUT',
            headers: {
              'Content-type': 'application/json'
            },
            body: JSON.stringify({ attacking: turn.attacker, attacker: turn.attacking, moves_left: 2, game_over: turn.game_over })
          })
          const data = await res.json()
          const restoredAttacker = await this.restoreAttacker(attacker)
          this.dinosaurs.forEach((d, i) => {
            if (restoredAttacker.id === d.id) {
              this.dinosaurs.splice(i, 1, restoredAttacker)
            }
          })
          this.turn = data
        }
      }
    },
    reset: async function() {     
      this.dinosaurs.forEach(async (d) => {
        d.health = 50
        d.confidence = 50
        let newObj = { ...d, health: 50, confidence: 50 }
        await fetch(`http://localhost:3000/dinosaurs/${d.id}`, {
          method: 'PUT',
          headers: {
            'Content-type': 'application/json'
          },
          body: JSON.stringify(newObj)
        })
      })

      const res = await fetch('http://localhost:3000/turn', {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json'
        },
        body: JSON.stringify({ attacker: 1, attacking: 2, moves_left: 2, game_over: false })
      })
      const data2 = await res.json()
      this.turn = data2
    }
  },
  computed: {
    isTurn: (turn, dinoType) => {
      return turn.type === dinoType
    }
  }
}
</script>