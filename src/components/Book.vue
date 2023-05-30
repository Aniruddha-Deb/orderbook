<script setup>
import BookRow from './BookRow.vue'

import {computed, reactive} from 'vue'

const props = defineProps({
  bias: Number,
  multiplier: Number,
  tick: Number,
  span: Number
});

const emit = defineEmits(['trade'])

const book = reactive({
  data: [],
  best_bid: -1,
  best_ask: -1,
  net_pos: 0,
  pnl: 0
})

const update_book = computed(() => {
  var new_data = []
  var existing_keys = new Set()
  book.data.forEach((entry) => {
    if (entry.vol == 0 && entry.pos == 0) {
      if (book.best_bid == entry.price) {
        book.best_bid = -1;
      }
      if (book.best_ask == entry.price) {
        book.best_ask = -1;
      }
    }
    else {
      existing_keys.add(entry.price)
      new_data.push(entry)
    }
  })

  for (var i=0; i < props.span; i++) {
    var price = (props.bias + props.tick*(props.span-i-1));
    if (existing_keys.has(price)) continue
    new_data.push({
      'price': price,
      'vol': 0,
      'pos': 0,
      'we_bid': false,
      'we_ask': false
    })
  }

  book.data = new_data.sort((a,b) => a.price<b.price)
  console.log(book.data)
  return book
})

function hit(we_act) {
  for (var i=0; i<book.data.length; i++) {
    if (book.data[i].vol > 0) {
      if (we_act && !book.data[i].we_bid) {
        book.data[i].vol--;
        emit('trade', book.data[i].price)
        book.net_pos--;
        var updated = false;
        for (var j=book.data.length-1; j>0; j--) {
          if (book.data[j].pos > 0) {
            book.data[j].pos--;
            book.pnl += (book.data[i].price-book.data[j].price);
            updated = true;
            break;
          }
        }
        if (!updated) {
          book.data[i].pos--;
        }
      }
      else if (!we_act) {
        book.data[i].vol--;
        emit('trade', book.data[i].price)
        if (!book.data[i].we_bid) break;
        book.net_pos++;
        book.data[i].we_bid = false;
        var updated = false;
        for (var j=0; j<book.data.length; j++) {
          if (book.data[j].pos < 0) {
            book.data[j].pos++;
            book.pnl += (book.data[j].price-book.data[i].price);
            updated = true;
            break;
          }
        }
        if (!updated) {
          book.data[i].pos++;
        }
      }
      break;
    }
  }
}

function lift(we_act) {
  for (var i=book.data.length-1; i>=0; i--) {
    if (book.data[i].vol < 0) {
      if (we_act && !book.data[i].we_ask) {
        emit('trade', book.data[i].price)
        book.data[i].vol++;
        book.net_pos++;
        var updated = false;
        for (var j=0; j<book.data.length; j++) {
          if (book.data[j].pos < 0) {
            book.data[j].pos++;
            book.pnl += (book.data[j].price-book.data[i].price);
            updated = true;
            break;
          }
        }
        if (!updated) {
          book.data[i].pos++;
        }
      }
      else if (!we_act) {
        book.data[i].vol++;
        emit('trade', book.data[i].price)
        if (!book.data[i].we_ask) break;
        book.net_pos--;
        book.data[i].we_ask = false;
        var updated = false;
        for (var j=book.data.length-1; j>0; j--) {
          if (book.data[j].pos > 0) {
            book.data[j].pos--;
            book.pnl += (book.data[i].price-book.data[j].price);
            updated = true;
            break;
          }
        }
        if (!updated) {
          book.data[i].pos--;
        }
      }
      break;
    }
  }
}

defineExpose({
  hit,
  lift
})

function update_our_bid_ask_status(price, upd) {
  price = parseInt(price)
  book.data.some((e) => {
    if (e.price === price) {
      upd(e)
      console.log(e)
    }
    return e.price === price
  })
}

function withdraw_bid(price) {
  update_our_bid_ask_status(price, (e) => {e.we_bid = false; e.vol--})
}

function withdraw_ask(price) {
  update_our_bid_ask_status(price, (e) => {e.we_ask = false; e.vol++})
}

function ask(price) {
  update_our_bid_ask_status(price, (e) => {e.we_ask = true; e.vol--})
}

function bid(price) {
  update_our_bid_ask_status(price, (e) => {e.we_bid = true; e.vol++})
}

</script>

<template>
  <table>
    <tr class='ob-header'>
      <th class="bid_bar"></th>
      <th class="long">L</th>
      <th class="vol_bid">Bid</th>
      <th class="price">Price</th>
      <th class="vol_ask">Ask</th>
      <th class="short">S</th>
      <th class="ask_bar"></th>
    </tr>

    <BookRow 
    v-for="entry in update_book.data" 
    v-model:vol="entry.vol"
    v-model:pos="entry.pos"
    :we_bid="entry.we_bid"
    :we_ask="entry.we_ask"
    :price="entry.price"
    :mult="this.multiplier"
    @we-bid="bid"
    @we-withdraw-bid="withdraw_bid"
    @we-withdraw-ask="withdraw_ask"
    @we-ask="ask"></BookRow>
  </table>
  <div class="summary-stat-bar">
    <label class="summary-stat">PnL = {{book.pnl}}</label>
    <label class="summary-stat">Pos = {{book.net_pos}}</label>
  </div>
</template>

<style scoped>
table {
  border-collapse: collapse;
}

div.summary-stat-bar {
  display: flex;
  padding-top: 10px;
  padding-bottom: 10px;
}

label.summary-stat {
  flex-grow: 1;
  text-align: center;
  font-weight: bold;
}

th {
  font-weight: bold;
  padding-left: 8px;
  padding-right: 8px;
}


</style>
