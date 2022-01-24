<template>
  <div
    class="inner d-flex flex-column h-100"
  >
    <h2 class="p-2">
      <p v-if="!turn.game_over" class="fs-3 mb-0">{{dino.name}}{{turn.attacking === dino.id ? '\'s turn!' : ''}}</p>
      <p v-else-if="turn.game_over" class="fs-3 mb-0">{{dino.name}}{{turn.attacker === dino.id ? ' won!' : ''}}</p>
      <p class="fs-4 mb-0" :class="{ 'invisible': turn.attacking === dino.id }">Moves left: {{turn.moves_left}}</p>
    </h2>
    <img :alt="`${dino.name}`" :src="`./assets/${dino.type}.jpeg`" >
    <div
      class="health-attack-container h-100 mb-2 d-flex flex-column"
    >
      <Health :type="dino.type" :health="dino.health" :confidence="dino.confidence" />
      <Attacks @attack-btn-click="handleAttack" :type="dino.type" :dino_id="dino.id" :turn="turn.attacking === dino.id" :game_over="turn.game_over" />
    </div>
  </div>
</template>

<script>
import Health from './Health'
import Attacks from './Attacks'

export default {
  name: "Dinosaur",
  components: {
    Health,
    Attacks
  },
  props: {
    turn: Object,
    dino: Object
  },
  methods: {
    handleAttack: async function(obj) {
      const attack = await this.getAttack(obj.attackId)
      const attacking = await this.getAttacking(this.turn.attacking)
      const newHealth = attack.type !== 'Roar' ? (attacking.health - (attack.damage + Math.floor(attacking.confidence * .1)) - Math.floor((50 - attacking.confidence) * .1)) : attacking.health
      const newConfidence = (attacking.confidence - attack.intimidation)
      let updateTurn = {}
      if (newHealth <= 0) {
        await this.gameOver()
        return
      }
      updateTurn = {
        ...this.turn,
        moves_left: this.turn.moves_left - 1
      }
      const attackingUpdate = {
        ...attacking,
        health: newHealth,
        confidence: newConfidence >= 0 ? newConfidence : 0
      }
      const updatedTurn = await this.updateTurn(updateTurn)
      const updatedAttacking = await this.updateAttacking(attackingUpdate)
      this.$emit('update', { attacking: updatedAttacking, turn: updatedTurn })
    },
    getAttack: async function(attackId) {
      const res = await fetch(`http://localhost:3000/attacks/${attackId}`)
      return await res.json()
    },
    getAttacking: async function(id) {
      const res = await fetch(`http://localhost:3000/dinosaurs/${id}`)
      return await res.json()
    },
    updateAttacker: async function(newData) {
      const res = await fetch(`http://localhost:3000/dinosaurs/${newData.id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(newData)
      })
      return await res.json()
    },
    updateAttacking: async function(newData) {
      const res = await fetch(`http://localhost:3000/dinosaurs/${newData.id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(newData)
      })
      return await res.json()
    },
    updateTurn: async function(newData) {
      const res = await fetch(`http://localhost:3000/turn`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(newData)
      })
      return await res.json()
    },
    gameOver: async function() {
      const res = await fetch(`http://localhost:3000/turn`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify({ ...this.turn, game_over: true })
      })
      const data = await res.json()
      this.$emit('game-over', data)
    }
  }
}
</script>