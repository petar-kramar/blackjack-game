<template>
  <div v-if="roundOver" class="busted">
    <h1 v-if="isBusted">BUSTED</h1>
    <h1 v-text="playerWinner ? 'Player wins' : 'Computer wins'" />
  </div>
  <div class="gamearea">
    <div class="dealer">
      <h2>Dealer</h2>
      <h2 v-if="!isBusted && !playerTurn">Dealer score: {{ totalScore.dealerScore }}</h2>
      <div class="dealer__Cards" v-if="dealerCards">
        <div v-for="item in dealerCards" :class="{ dealer__CardHidden: !dealerCardShown }">
          <img :src="item.image" alt="" />
        </div>
      </div>
    </div>
    <div class="dealer">
      <h2>Player</h2>
      <h2>Player score: {{ totalScore.playerScore }}</h2>
      <div class="options">
        <button @click="drawPlayerCards" :disabled="!playerTurn || totalScore.playerScore > 21">Hit</button>
        <button @click="stand" :disabled="!playerTurn || totalScore.playerScore > 21">Stand</button>
        <button v-if="!playerTurn" @click="computerPlay">Computer</button>
      </div>
      <div class="dealer__Cards" v-if="playerCards">
        <div v-for="item in playerCards">
          <img :src="item.image" alt="" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, onBeforeMount, computed } from 'vue'
  const deckUrl = 'https://deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1'
  const deck = ref(null)
  const cards = ref(null)
  const dealerCards = ref([])
  const playerCards = ref([])
  const numberOfCards = ref(4)
  const dealerCardShown = ref(false)
  const playerTurn = ref(true)
  const roundOver = ref(false)
  const fetchDeck = async () => {
    try {
      const response = await fetch(deckUrl)
      if (!response.ok) {
        throw new Error(`Failed to fetch first data: ${response.statusText}`)
      }
      deck.value = await response.json()
    } catch (error) {
      console.error('Error fetching data:', error)
    }
  }

  const fetchCards = async () => {
    try {
      const newCards = await fetch(
        `https://deckofcardsapi.com/api/deck/${deck?.value.deck_id}/draw/?count=${numberOfCards.value}`
      )
      if (!newCards.ok) {
        throw new Error(`Failed to fetch first data: ${newCards.statusText}`)
      }
      const allCards = await newCards.json()
      cards.value = allCards
    } catch (error) {
      console.error('Error fetching data:', error)
    }
  }

  onBeforeMount(async () => {
    await fetchDeck()
    await fetchCards()
    dealerCards.value = cards.value.cards.splice(0, 2)
    playerCards.value = cards.value.cards.splice(0, 2)
  })

  async function drawPlayerCards() {
    numberOfCards.value = 1
    await fetchCards()
    let newCard = cards.value.cards.splice(0, 1)
    playerCards.value.push(newCard[0])
  }

  const totalScore = computed(() => {
    let playerScore = []
    let dealerScore = []
    playerCards.value.forEach(element => {
      if (element.value === 'QUEEN' || element.value === 'KING' || element.value === 'JACK') {
        playerScore.push(10)
      } else if (element.value === 'ACE') {
        playerScore.push(11)
      } else {
        playerScore.push(element.value)
      }
    })

    dealerCards.value.forEach(element => {
      if (element.value === 'QUEEN' || element.value === 'KING' || element.value === 'JACK') {
        dealerScore.push(10)
      } else if (element.value === 'ACE') {
        dealerScore.push(11)
      } else {
        dealerScore.push(element.value)
      }
    })
    let totalDealerScore = dealerScore.reduce((a, b) => parseInt(a) + parseInt(b), 0)
    let totalPlayerScore = playerScore.reduce((a, b) => parseInt(a) + parseInt(b), 0)
    if (totalPlayerScore > 21) {
      playerTurn.value = false
    }
    return {
      dealerScore: totalDealerScore,
      playerScore: totalPlayerScore
    }
  })

  const isBusted = computed(() => {
    if (totalScore.value.playerScore > 21) {
      roundOver.value = true
      return true
    }
    return false
  })

  const playerWinner = computed(() => {
    if (roundOver.value) {
      if (
        totalScore.value.playerScore < 22 &&
        (totalScore.value.playerScore > totalScore.value.dealerScore || totalScore.value.dealerScore > 21)
      ) {
        return true
      } else if (totalScore.value.dealerScore < 22 && totalScore.value.dealerScore >= totalScore.value.playerScore) {
        return false
      }
    }
    return false
  })
  function stand() {
    dealerCardShown.value = true
    playerTurn.value = false
  }

  async function computerPlay() {
    console.log(totalScore.value.playerScore)
    console.log(totalScore.value.dealerScore)
    if (isBusted.value == false) {
      if (totalScore.value.dealerScore < 17) {
        playerTurn.value = false
        numberOfCards.value = 1
        await fetchCards()
        let newCard = cards.value.cards.splice(0, 1)
        dealerCards.value.push(newCard[0])
      }
      if (totalScore.value.dealerScore >= 17) {
        roundOver.value = true
      }
    }
  }
</script>
