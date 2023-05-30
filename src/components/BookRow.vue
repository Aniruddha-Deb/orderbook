<script setup>
import {ref, computed, reactive} from 'vue'

const props = defineProps({
  price: Number,
  vol: Number,
  pos: Number,
  mult: Number,
  we_bid: Boolean,
  we_ask: Boolean,
});

const emit = defineEmits(['weBid', 'weWithdrawBid', 'weAsk', 'weWithdrawAsk'])

const our_status = computed(() => {
  console.log(props.we_bid, props.we_ask)
  if (props.we_bid) return "green"
  else if (props.we_ask) return "yellow"
  else return "default"
})

function bid() {
  if (props.we_bid) {
    emit('weWithdrawBid', props.price)
  }
  else {
    if (props.we_ask) {
      emit('weWithdrawAsk', props.price)
    }
    emit('weBid', props.price)
  }
}

function ask() {
  if (props.we_ask) {
    emit('weWithdrawAsk', props.price)
  }
  else {
    if (props.we_bid) {
      emit('weWithdrawBid', props.price)
    }
    emit('weAsk', props.price)
  }
}

function abs_bid_vol() {
  return Math.max(props.vol,0)
}

function abs_ask_vol() {
  return Math.max(-props.vol,0)
}

function abs_long_pos() {
  return Math.max(props.pos,0)
}

function abs_short_pos() {
  return Math.max(-props.pos,0)
}
</script>

<template>
  <tr>
    <td class="bar-graph bid-bar-cell">
      <progress 
      :value="abs_bid_vol()" 
      max="10" 
      class="bid_bar">
      </progress>
    </td>

    <td 
    class="long"
    @click="$emit('update:pos', pos+1); $emit('update:vol', vol+1)"
    @contextmenu.prevent="$emit('update:pos', pos-1)">
      {{abs_long_pos()||''}}
    </td>

    <td 
    class="vol_bid" 
    @click="$emit('update:vol', vol+1)" 
    @contextmenu.prevent="$emit('update:vol', vol-1)">
      {{abs_bid_vol()||''}}
    </td>

    <td 
    class="price" 
    :class="our_status" 
    @click="bid"
    @contextmenu.prevent="ask">
      {{price/mult}}
    </td>

    <td 
    class="vol_ask" 
    @click="$emit('update:vol', vol-1)" 
    @contextmenu.prevent="$emit('update:vol', vol+1)">
      {{abs_ask_vol()||''}}
    </td>

    <td 
    class="short" 
    @click="$emit('update:pos', pos-1); $emit('update:vol', vol-1)"
    @contextmenu.prevent="$emit('update:pos', pos+1)">
      {{abs_short_pos()||''}}
    </td>

    <td class="bar-graph">
      <progress 
      :value="abs_ask_vol()" 
      max="10" 
      class="ask_bar">
      </progress>
    </td>
  </tr>
</template>

<style scoped>
td {
  border: solid 1px var(--vt-c-divider-dark-1);
  padding: 2px;
  padding-right: 8px;
  padding-left: 8px;
  color: var(--vt-c-white);
  user-select: none;
}

td.vol_ask {
  text-align: left;
}

td.vol_bid {
  text-align: right;
}

td.price {
  text-align: center;
}

td.price.yellow {
  background-color: GoldenRod;
}

td.price.green {
  background-color: ForestGreen;
}

td.bar-graph {
  padding: 0px;
  border: none;
  vertical-align: middle;
}

td.bid-bar-cell {
  text-align: right;
}

tr {
  font-family: monospace;
  font-size: 14px;
}

progress{
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  border: none;
  background-color: rgba(255,255,255,0);
}

.bid_bar {
  transform: rotate(180deg);
  color: green;
}

progress.bid_bar::-moz-progress-bar {
  background-color: green;
  border: none;
}
progress.ask_bar::-moz-progress-bar {
  background-color: red;
  border: none;
}

</style>
