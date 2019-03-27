<template>
  <div class="betting-interface">
    <div v-if="!value.placed">
      <h4>Place your bets</h4>
      <message-box v-if="message" :message="message"></message-box>
      <p>
        <label for="betSize">
        Place your bet to start a game. The minimum bet size is €10 and the maximum bet size is &euro; {{ cash }}.
        </label>
      </p>
      <input type="number" v-model="betSize" id="betSize" placeholder="Bet size...">
      <button class="btn btn-warning" @click="placeBet">Place bet</button>
    </div>
    <div v-if="value.placed">
      <h4>Your current bet: €{{ value.size }}</h4>
    </div>
  </div>
</template>

<script>
import MessageBox from "@/components/MessageBox.vue";

export default {
  name: 'BettingInterface',
  components: {
    MessageBox
  },
  props: {
    cash: Number,
    value: {
      type: Object,
    }
  },
  data() {
    return {
      betSize: this.value.size,
      message: ''
    }
  },
  methods: {
    placeBet() {
      this.message = '';
      if( this.betSize < 10 ) {
        this.message = 'Minimale inzet is €10 !';
        return;
      }
      if( this.betSize > this.cash ) {
        this.message = `Maximale inzet is €${this.cash} !`;
        return;
      }
      this.value.size = this.betSize;
      this.value.placed = true;
      this.$emit('input', this.value);
    }
  }
}
</script>

<style scoped>
  h4 {
    color: black;
  }

  .betting-interface {
    background-color: lightgrey;
    padding: 1rem;
  }
</style>


