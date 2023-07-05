<script setup lang="ts">
import { ref, computed, } from 'vue'

const scene = ref('title')

//////////

const welcomeDialogVisible = computed(() => {
  return scene.value === 'title'
})

const gameSceneVisible = computed(() => {
  return scene.value === 'game'
})

//////////

const onClickStart = () => {
  scene.value = 'game'
}

</script>

<template>
  <el-dialog
    v-model="welcomeDialogVisible"
    title="welcome"
    width="30%"
  >
    <h1>MOJIGUESSER</h1>
    <p>拡大された漢字がなんという文字か、スクロールしながら探ろう！</p>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="onClickStart">
          start!
        </el-button>
      </span>
    </template>
  </el-dialog>
  <div
    v-if="gameSceneVisible"
  >
    <p>this is game scene!</p>
  </div>
</template>

<style scoped lang="scss">
.wrapper {
  color: red;

  .inner {
    color: blue;
  }
}
</style>
