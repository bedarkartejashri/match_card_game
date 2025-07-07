<!--
  ==========================================================================
  Component: Card.vue
  Description: Single card component. Handles flip animation and emits 
               events on user selection.
  Author: Tejashri Bedarkar
  Created: 2025-07-05
  Last Updated: 2025-07-06
  ==========================================================================
-->
<script>
import { computed } from 'vue'

export default {
    props: {
        position: {
            type: Number,
            required: true
        },
        value: {
            type: String,
            required: true
        },
        visible: {
            type: Boolean,
            default: false
        },
        matched: {
            type: Boolean,
            default: false
        },
        disabled: {
            type: Boolean,
            default: false
        }
    },
    setup(props, context) {
        if (props.disabled) return; // To disable card click event after a match or if player wins.

        /**
         * Computes the CSS class for flip animation
         * Adds 'is-flipped' class if the card's visible flag is true
         */
        const flipAnimation = computed(() => {
            if(props.visible) {
                return 'is-flipped'
            }
        })

        /**
         * Emits an event when the card is clicked
         * Passes its position and value to the parent
         */
        const chooseCard = () => {
            context.emit('choose-card', {
                position: props.position,
                sideValue: props.value
            })
        }

        // expose variables and methods to the template
        return {
            flipAnimation,
            chooseCard
        }
    }
}
</script>

<template>
    <div class="card" :class="flipAnimation" @click="chooseCard">
        <!-- front side of the card, shows the flag image -->
        <div class="card-side is-front">
            <img :src="`/images/${value}.png`" :alt="value" />
        </div>
        <!-- back side of the card -->
        <div class="card-side is-back"></div>
    </div>
</template>

<style>
/* Base card styles */
.card {
    position: relative;
    transition: 0.5s transform ease-in;
    transform-style: preserve-3d;
}

/* Flip animation: rotate the card on the Y axis */
.card.is-flipped {
    transform: rotateY(180deg);
}

/* Shared side styles for both front and back */
.card-side {
    width: 100%;
    height: 100%;
    position: absolute;
    border: 2px solid cornsilk;
    border-radius: 5px;
    display: flex;
    align-items: center;
    justify-content: center;
    backface-visibility: hidden;
}

/* front side of the card: shows the image */
.card-side.is-front {
    background-color: lightsteelblue;
    color: white;
    transform: rotateY(180deg);
}

/* back side of the card */
.card-side.is-back {
    background-color: blueviolet;
    color: black;
}
</style>