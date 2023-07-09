<script setup lang="ts">
import { computed, onBeforeMount, ref } from "vue";

const DISPLAY_SIZE = 96; // vmin / 100 - (padding * 2)
const TARGET_SIZE = 400; // vmin
const MOVE_AMOUNT = 10; // vmin

const REGAL_RANGE_MAX = 0;
const REGAL_RANGE_MIN = -(TARGET_SIZE - DISPLAY_SIZE);

type Direction = "up" | "right" | "down" | "left";

type Props = {
  target: string;
};

const props = defineProps<Props>();

//////////

const guessTarget = ref("");
const positionX = ref(0);
const positionY = ref(0);

//////////

const targetStyle = computed(() => {
  return {
    transform: `translate(${positionX.value}vmin, ${positionY.value}vmin)`,
    "font-size": `${TARGET_SIZE}vmin`,
  };
});

const canMoveDirections = computed(() => {
  const direcitons: Direction[] = [];

  if (positionX.value < REGAL_RANGE_MAX) {
    direcitons.push("left");
  }

  if (positionY.value < REGAL_RANGE_MAX) {
    direcitons.push("up");
  }

  if (positionX.value > REGAL_RANGE_MIN) {
    direcitons.push("right");
  }

  if (positionY.value > REGAL_RANGE_MIN) {
    direcitons.push("down");
  }

  return direcitons;
});

//////////

onBeforeMount(() => {
  const getRandom = (min: number, max: number): number =>
    Math.floor(Math.random() * (max + 1 - min)) + min;
  positionX.value = getRandom(REGAL_RANGE_MIN, REGAL_RANGE_MAX);
  positionY.value = getRandom(REGAL_RANGE_MIN, REGAL_RANGE_MAX);

  guessTarget.value = props.target;
});

//////////

const moveTarget = (direction: Direction) => {
  switch (direction) {
    case "up":
      positionY.value += MOVE_AMOUNT;
      break;
    case "right":
      positionX.value -= MOVE_AMOUNT;
      break;
    case "down":
      positionY.value -= MOVE_AMOUNT;
      break;
    case "left":
      positionX.value += MOVE_AMOUNT;
      break;
    default:
      throw new Error(`unknown direction: ${direction}`);
  }
};
</script>

<template>
  <div class="gameboard-wrapper">
    <div class="gameboard">
      <div class="target" :style="targetStyle">{{ guessTarget }}</div>

      <div class="controllers">
        <button
          v-show="canMoveDirections.includes('up')"
          class="ctrl-btn up"
          @click="moveTarget('up')"
        >
          ↑
        </button>
        <button
          v-show="canMoveDirections.includes('right')"
          class="ctrl-btn right"
          @click="moveTarget('right')"
        >
          →
        </button>
        <button
          v-show="canMoveDirections.includes('down')"
          class="ctrl-btn down"
          @click="moveTarget('down')"
        >
          ↓
        </button>
        <button
          v-show="canMoveDirections.includes('left')"
          class="ctrl-btn left"
          @click="moveTarget('left')"
        >
          ←
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
@use "sass:math";

.gameboard-wrapper {
  box-sizing: border-box;
  margin: 0 auto;
  padding: 2vmin;
  width: 100vmin;
  height: 100vmin;
  user-select: none;

  .gameboard {
    position: relative;
    box-sizing: border-box;
    width: 100%;
    height: 100%;
    border: solid 1px black;
    overflow: hidden;

    .controllers {
      width: 100%;
      height: 100%;

      .ctrl-btn {
        $min-size: 40px;
        $btn-size: 70%;
        $margin-size: math.div(100% - $btn-size, 2);
        $button-border-radius: 4px;
        $background-color: #999;
        $stroke-color: white;

        position: absolute;
        display: flex;
        align-items: center;
        justify-content: center;
        min-width: $min-size;
        min-height: $min-size;
        text-shadow: -1px -1px 0 $stroke-color, -1px 0 0 $stroke-color,
          -1px 1px 0 $stroke-color, 0 -1px 0 $stroke-color,
          0 1px 0 $stroke-color, 1px -1px 0 $stroke-color, 1px 0 0 $stroke-color,
          1px 1px 0 $stroke-color;

        border: none;
        background: rgba($background-color, 0.3);

        &:hover {
          background: rgba($background-color, 0.8);
        }

        &:active {
          background: $background-color;
        }

        &.up {
          top: 0;
          left: 0;
          margin: 0 $margin-size;
          width: $btn-size;
          height: $min-size;
          border-radius: 0 0 $button-border-radius $button-border-radius;
        }

        &.right {
          right: 0;
          bottom: 0;
          margin: $margin-size 0;
          width: $min-size;
          height: $btn-size;
          border-radius: $button-border-radius 0 0 $button-border-radius;
        }

        &.down {
          right: 0;
          bottom: 0;
          margin: 0 $margin-size;
          width: $btn-size;
          height: $min-size;
          border-radius: $button-border-radius $button-border-radius 0 0;
        }

        &.left {
          top: 0;
          left: 0;
          margin: $margin-size 0;
          width: $min-size;
          height: $btn-size;
          border-radius: 0 $button-border-radius $button-border-radius 0;
        }
      }
    }

    .target {
      position: absolute;
      top: 0;
      left: 0;
      transition: all 0.1s 0s ease;
    }
  }
}
</style>
