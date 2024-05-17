<template>
  <div
    class="home"
    @mousemove="handleCardDrag($event)"
    @mouseup="handleCardRelease($event)"
  >
    <div ref="container" class="home__container">
      <a href="https://x.com/kolapo_" target="_blank" class="home__title">
        Stacked Cards — Made with love by Kolapo ❤️
      </a>
      <div
        ref="cards"
        class="home__cards"
        @mousedown="handleCardGrab($event)"
        @mouseover="handleCardHover($event)"
        @touchstart="handleCardTouchStart($event)"
        @touchend="handleCardTouchEnd($event)"
      >
        <div v-for="i in 6" :key="i">
          <img
            :src="require(`~/assets/images/building-${i}.jpg`)"
            alt="An image of a building"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cardSize: '',
      mouseDown: false,
      activeCardIndex: '',
      cardsTouchStartX: null,
    }
  },
  computed: {
    activeCard() {
      return this.$refs.cards.children[this.activeCardIndex]
    },
  },
  mounted() {
    this.activeCardIndex = this.$refs.cards.children.length - 1
    setTimeout(() => {
      this.revealCards()
    }, 300)
  },
  methods: {
    revealCards() {
      this.$refs.cards.style.setProperty('--scale', 1)
    },
    handleCardHover(event) {
      event.target.style.cursor = 'grab'
    },
    handleCardGrab(event) {
      this.mouseDown = true
      event.target.style.cursor = 'grabbing'
      this.activeCard = event.target.lastChild
      this.cardSize = this.activeCard.getBoundingClientRect().width
    },
    handleCardDrag(event) {
      if (this.mouseDown) {
        this.activeCard.style.transform = `translate3d(${
          event.clientX - (this.cardSize * 1.8 * window.innerWidth) / 1680
        }px, 0, 0)`
        this.activeCard.style.transition = 'transform 0.3s ease-out'
      }
    },
    handleCardRelease(event) {
      if (!this.mouseDown) return
      this.mouseDown = false
      event.target.style.cursor = 'auto'
      const offset = this.activeCard.getBoundingClientRect().left
      let translateX = 0

      if (
        offset + this.cardSize / 2 <
        window.innerWidth / 2 - this.cardSize / 2
      ) {
        translateX = -100
      } else if (
        offset + this.cardSize / 2 >
        window.innerWidth / 2 + this.cardSize / 2
      ) {
        translateX = 100
      }

      this.translateCard(translateX)
    },
    translateCard(translateValue) {
      this.activeCard.style.transform = `translate3d(${translateValue}vw, 0, 0)`

      if (translateValue !== 0) {
        const cards = Array.from(this.$refs.cards.children).reverse()
        let rotation = 0
        for (let index = 0; index < cards.length; index++) {
          const cardOffset = cards[index].getBoundingClientRect().left
          if (this.activeCard === cards[index]) continue
          if (cardOffset > 0 && cardOffset < window.innerWidth) {
            cards[index].style.transform = `rotate(${rotation}deg)`
            rotation += 4
          }
        }

        this.activeCardIndex--
      }

      if (this.activeCardIndex < 0) {
        this.activeCardIndex = this.$refs.cards.children.length - 1
        setTimeout(() => {
          Array.from(this.$refs.cards.children).forEach((card) => {
            card.removeAttribute('style')
          })
          this.$refs.container.style.cursor = 'auto'
        }, 300)
      }
    },
    // handle touch start event on mobile
    handleCardTouchStart(event) {
      this.activeCard.style.transition = 'transform 0.3s ease-out'
      this.$refs.cardsTouchStartX = event.touches[0].clientX
    },
    // handle touch end event on mobile
    handleCardTouchEnd(event) {
      const deltaX =
        event.changedTouches[0].clientX - this.$refs.cardsTouchStartX

      let translateX = 0
      if (deltaX > 20) {
        translateX = 100
      } else if (deltaX < -20) {
        translateX = -100
      }

      this.translateCard(translateX)
    },
  },
}
</script>

<style lang="scss">
@import '~/assets/scss/pages/home.scss';
</style>
