<script setup lang="ts">
import { computed, onBeforeMount, onBeforeUnmount, onMounted, ref } from "vue";
import { Star } from "@element-plus/icons-vue";

import type { CSSProperties } from "vue";

const DISPLAY_SIZE = 96; // vmin / 100 - (padding * 2)
const TARGET_SIZE = 300; // vmin
const MOVE_AMOUNT = 10; // vmin

const REGAL_RANGE_MAX = 0;
const REGAL_RANGE_MIN = -(TARGET_SIZE - DISPLAY_SIZE);

type Direction = "up" | "right" | "down" | "left";

type GuessEventArg = { moveCount: number; answer: string };

type Props = {
  target: string;
};

type Emits = {
  (e: "guess", value: GuessEventArg): void;
};

const props = defineProps<Props>();

const emit = defineEmits<Emits>();

//////////

const guessTarget = ref<string>("");
const positionX = ref<number>(0);
const positionY = ref<number>(0);
const moveCount = ref<number>(0);
const guessDialogVisible = ref<boolean>(false);
const answer = ref<string>("");

//////////

const targetStyle = computed<CSSProperties>(() => {
  return {
    transform: `translate(${positionX.value}vmin, ${positionY.value}vmin)`,
    "font-size": `${TARGET_SIZE}vmin`,
  };
});

const canMoveDirections = computed<Direction[]>(() => {
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

onBeforeMount((): void => {
  const getRandom = (min: number, max: number): number =>
    Math.floor(Math.random() * (max + 1 - min)) + min;
  positionX.value = getRandom(REGAL_RANGE_MIN, REGAL_RANGE_MAX);
  positionY.value = getRandom(REGAL_RANGE_MIN, REGAL_RANGE_MAX);

  guessTarget.value = props.target;
});

const keyDownHandler = (ev: KeyboardEvent) => {
  if (ev.key === "w" || ev.key === "ArrowUp") {
    moveTarget("up");
  } else if (ev.key === "d" || ev.key === "ArrowRight") {
    moveTarget("right");
  } else if (ev.key === "s" || ev.key === "ArrowDown") {
    moveTarget("down");
  } else if (ev.key === "a" || ev.key === "ArrowLeft") {
    moveTarget("left");
  }
};

onMounted(() => {
  document.addEventListener("keydown", keyDownHandler);
});

onBeforeMount(() => {
  document.removeEventListener("keydown", keyDownHandler);
});

//////////

const moveTarget = (direction: Direction): void => {
  if (direction === "up" && canMoveDirections.value.includes("up")) {
    positionY.value += MOVE_AMOUNT;
  } else if (direction === "right" && canMoveDirections.value.includes("right")) {
    positionX.value -= MOVE_AMOUNT;
  } else if (direction === "down" && canMoveDirections.value.includes("down")) {
    positionY.value -= MOVE_AMOUNT;
  } else if (direction === "left" && canMoveDirections.value.includes("left")) {
    positionX.value += MOVE_AMOUNT;
  } else {
    return;
  }

  moveCount.value++;
};

const showGuessDialog = (): void => {
  guessDialogVisible.value = true;
};

const onClickGuess = (): void => {
  emit("guess", { moveCount: moveCount.value, answer: answer.value });
};
</script>

<template>
  <div
    class="gameboard-wrapper"
    @keydown.w="moveTarget('up')"
    @keydown.d="moveTarget('right')"
    @keydown.s="moveTarget('down')"
    @keydown.a="moveTarget('left')"
  >
    <div class="gameboard">
      <div class="target" :style="targetStyle">
        {{ guessTarget }}
      </div>

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

        <el-button type="warning" :icon="Star" circle class="guess-btn" @click="showGuessDialog" />
      </div>

      <el-dialog v-model="guessDialogVisible" title="guess!" width="30%">
        <p>回答権は1回だけだよ！</p>
        <p>あなたの回答</p>
        <el-input v-model="answer" placeholder="範囲は常用漢字だよ" />
        <template #footer>
          <span class="dialog-footer">
            <el-button type="primary" @click="onClickGuess">guess!</el-button>
          </span>
        </template>
      </el-dialog>
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
        text-shadow: -1px -1px 0 $stroke-color, -1px 0 0 $stroke-color, -1px 1px 0 $stroke-color,
          0 -1px 0 $stroke-color, 0 1px 0 $stroke-color, 1px -1px 0 $stroke-color,
          1px 0 0 $stroke-color, 1px 1px 0 $stroke-color;

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

      .guess-btn {
        position: absolute;
        bottom: 8px;
        right: 8px;
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
