<template>
  <div
    class="attacks d-flex justify-content-around pt-3"
  >
    <button 
      :disabled="turn || game_over" 
      class="btn bite"
      @click="$emit('attack-btn-click', { attackId: 1 })"
    >
      Bite
    </button>
    <button 
      :disabled="turn || game_over" 
      class="btn roar"
      @click="$emit('attack-btn-click', { attackId: 2 })"
    >
      Roar
    </button>
    <button 
      :disabled="turn || game_over"
      class="btn stomp"
      @click="$emit('attack-btn-click', { attackId: 3 })"
    >
      Stomp
    </button>
  </div>
</template>

<script> 

export default {
  name: "Attacks",
  props: {
    type: String,
    turn: Boolean,
    dino_id: Number,
    game_over: Boolean
  }
}
</script>