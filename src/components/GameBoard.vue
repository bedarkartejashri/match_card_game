<!--
  ==========================================================================
  Component: GameBoard.vue
  Description: A memory matching game board with card flipping logic.
  Author: Tejashri Bedarkar
  Created: 2025-07-05
  Last Updated: 2025-07-06
  ==========================================================================
-->
<script>
import { ref, watch, computed, onMounted } from 'vue';
import Card from './Card.vue'

export default {
  name: 'GameBoard',
  components: {
    Card
  },
  setup() {
    /**
     * onMounted lifecycle hook
     * - Resets the game on initial load
     */
    onMounted(() => {
      resetGame(); 
    })

    // To disable card click event
    const disableCards = ref(false) 

    // Cards Object
    const cards = ref([])

    // Tracks current card selection(max two cards at a time)
    const userSelection = ref([])

    /**
     * Computes the number of pairs left
     * - Filters out matched cards
     * - Returns half the unmatched count
     */
    const remainingCardPairs = computed (() => {
      const remainingCards = cards.value.filter(card => card.matched === false).length

      return remainingCards / 2
    })

    /**
     * Provides a status message based on the game progress
     * - "You Win!" if no pairs left
     * - otherwise shows pairs remaining
     */
    const status = computed(() => {
      if(remainingCardPairs.value === 0 ) {
        return 'You Win!'
      } else {
        return `Remaining Card Pairs: ${remainingCardPairs.value}`
      }
    })

    /**
     * Shuffles the cards randomly
     */
    const shuffleCards = () => {
      let currentIndex = cards.value.length - 1
      let randomIndex

      while(currentIndex !== 0) {
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex--

        [cards.value[currentIndex], cards.value[randomIndex]] = [cards.value[randomIndex], cards.value[currentIndex]]
      }

      return cards;
    }

    /**
     * Resets the entire game:
     * - enables card clicks
     * - shuffles the cards
     * - resets their matched/visible states
     */
    const resetGame = () => {
      disableCards.value = false;
      shuffleCards()

      cards.value = cards.value.map((card, index) => {
        return {
          ...card,
          matched: false,
          position: index,
          visible: false
        }
      })
    }

    // Card items
    const flags = [
      'au', 'br', 'ca', 'in', 'it', 'jp', 'uk', 'us'
    ]

    /**
     * Dynamically generates the cards:
     * - one card for each flag
     * - and a duplicate of each
     */
    flags.forEach(flag => {
      cards.value.push({
        value: flag,
        visible: false,
        position: null,
        matched: false
      })

      // Add duplicates
      cards.value.push({
        value: flag,
        visible: false,
        position: null,
        matched: false
      })
    })

    // Update the position of the card
    cards.value = cards.value.map((card, index) => {
      return {
        ...card,
        position: index,
      }
    })

    /**
     * Handles Flip Card Event.
     * @param payload - information of clicked card.
     */
    const flipCard = (payload) => {
      if (disableCards.value) return; // Do nothing if user wins
      if(cards.value[payload.position].matched) return // Do nothing if card already matched  

      // Allow flipping
      cards.value[payload.position].visible = true

      if (userSelection.value[0]) {
        if(userSelection.value[0].position === payload.position && userSelection.value[0].sideValue === payload.sideValue) {
          // Do nothing. Same card clicked again
          return
        } else {
          userSelection.value[1] = payload // Second card selection
        }
      } else {
        userSelection.value[0] = payload // First card selection
      }
    }

     /**
     * Watch for remaining pairs:
     * - disables further clicks once there are no pairs left
     */
    watch(remainingCardPairs, currentValue => {
      if(currentValue === 0) {
        disableCards.value = true
      }
    })

    /**
     * Watch for user card selections:
     * - if two cards selected:
     *    - check if they match
     *    - if matched, mark them as matched
     *    - if not, hide both after 1 second
     */
    watch(userSelection, currentValue => {
      if(currentValue.length === 2) {
        const cardOne = currentValue[0]
        const cardTwo = currentValue[1]

        if(cardOne.sideValue === cardTwo.sideValue) {
          cards.value[cardOne.position].matched = true
          cards.value[cardTwo.position].matched = true
        } else {
          setTimeout(() => {
            cards.value[cardOne.position].visible = false
            cards.value[cardTwo.position].visible = false
          }, 1000) 
          
        }

        userSelection.value.length = 0 // resets user selection for next turn
      }
    }, {deep: true})

    // expose data and methods to the template
    return {
      cards,
      userSelection,
      status,
      disableCards,
      flipCard,
      shuffleCards,
      resetGame
    }
  }
}
</script>

<template>
  <div class="game-board">
    <!-- Dynamically create cards -->
    <Card v-for="(card, index) in cards" :key="`card-${index}`" :value="card.value" :visible="card.visible" :position="card.position" :matched="card.matched" :disabled="disableCards"  @choose-card="flipCard"/>
  </div>
  <!-- Show the game status and a reset button -->
  <div class="status-bar">
    <h3>{{ status }}</h3>
    <button id="reset_button" @click="resetGame">Reset Game</button>
  </div>
</template>

<style scoped>
/* The game grid layout with 4 rows and 4 columns */
.game-board {
  display: grid;
  grid-template-rows: repeat(4, 90px);
  grid-template-columns: repeat(4, 130px);
  grid-row-gap: 20px;
  grid-column-gap: 20px;
  justify-content: center;
}

/* Reset Button */
#reset_button {
    background-color: crimson;
}

/* To show status text and reset button side by side */
.status-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 10px auto;
}
</style>