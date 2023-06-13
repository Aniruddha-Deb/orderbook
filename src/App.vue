<script setup>
import Book from './components/book.vue'

import {ref, shallowRef, triggerRef, reactive, computed} from 'vue'

const state = reactive({
  bias: 4000,
  multiplier: 100,
  tick: 100,
  span: 20
})

const ltp_data = ref([
  {
    name: "series-1",
    data: [],
  },
])

const chart_options = ref({
  chart: {
    id: "vuechart-example",
  },
  xaxis: {
    type: 'number'
  },
})

function update_plot(trade_price) {
  var new_ltp_data = ltp_data.value
  new_ltp_data[0].data.push({
    'x': new_ltp_data[0].data.length,
    'y': trade_price
  })
  ltp_data.value = new_ltp_data
}
</script>

<template>
  <div class="book-container">
    <form>
      <div class="field_pad">
        <label for="bias">Multiplier:</label>
        <input type="number" v-model.number="state.multiplier" placeholder="multiplier">
      </div>
      <div class="field_pad">
        <label for="bias">Bias:</label>
        <input type="number" id="bias" v-model.number="state.bias" placeholder="bias">
      </div>
      <div class="field_pad">
        <label for="bias">Tick:</label>
        <input type="number" v-model.number="state.tick" placeholder="ticks">
      </div>
      <div class="field_pad">
        <label for="bias">Span:</label>
        <input type="number" v-model.number="state.span" placeholder="span">
      </div>
    </form>
    <Book 
    ref="book" 
    :bias="state.bias" 
    :multiplier="state.multiplier" 
    :tick="state.tick" 
    :span="state.span"
    @trade="update_plot"></Book>
    <div class="button_bar">
      <div class="button-enclose"><button class='button-hit' @click="$refs.book.hit(true)">Hit</button></div>
      <div class="button-enclose"><button class='button-lift' @click="$refs.book.lift(true)">Lift</button></div>
      <div class="button-enclose"><button class='button-reset' @click="reset">Reset</button></div>
      <div class="button-enclose"><button class='button-mkt-hit' @click="$refs.book.hit(false)">MktHit</button></div>
      <div class="button-enclose"><button class='button-mkt-lift' @click="$refs.book.lift(false)">MktLift</button></div>
    </div>
  </div>
  <apexchart
    id="last-traded-price"
    :options="chart_options"
    :series="ltp_data"></apexchart>
</template>

<style scoped>
form {
  display: flex;
  width: 100%;
  padding-bottom: 20px;
}

div.button_bar {
  display: flex;
  width: 100%;
}

div.book-container {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

div.button-enclose {
  flex-grow: 1;
  padding-right: 10px;
  padding-left: 10px;
}

button.button-hit {
  background-color: ForestGreen;
}

button.button-mkt-hit {
  background-color: DarkGreen;
}

button.button-lift {
  background-color: GoldenRod;
}

button.button-mkt-lift {
  background-color: DarkGoldenRod;
}

button.button-reset {
  background-color: Crimson;
}

button:hover {
  filter: brightness(90%);
}

button:active {
  filter: brightness(80%);
}

apexchart {
  flex-grow: 1;
}

button {
  width: 100%;
  height: 30px;
  border: none;
  font-weight: bold;
  background-color: green;
  color: var(--vt-c-white);
  border-radius: 4px;
}

div.field_pad {
  width: 100%;
  align: center;
  text-align: center;
}

label {
  padding-right: 4px;
}

input[type=number] {
  width: 60px;
  padding: 4px;
  background-color: var(--vt-c-black);
  border: none;
  text-align: center;
  border-bottom: solid 2px var(--vt-c-divider-dark-1);
  color: var(--vt-c-white);
  -moz-appearance: textfield;
}

</style>
