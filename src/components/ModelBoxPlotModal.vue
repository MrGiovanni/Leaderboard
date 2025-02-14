<!-- src/components/ModelBoxPlotModal.vue -->
<template>
  <div class="modal-overlay" @click.self="closeModal">
    <div class="modal-content">
      <button class="close-button" @click="closeModal">X</button>
      <!-- 선택한 모델의 박스플롯 분석 컴포넌트 -->
      <BoxPlotDisplay
        :dataset="dataset"
        :modelName="modelName"
        :scoreType="scoreType"
        :organ="organ"
      />
    </div>
  </div>
</template>

<script>
import BoxPlotDisplay from "./BoxPlotDisplay.vue";

export default {
  name: "ModelBoxPlotModal",
  components: { BoxPlotDisplay },
  props: {
    dataset: { type: String, required: true },
    modelName: { type: String, required: true },
    scoreType: { type: String, required: true },
    organ: { type: String, default: "mean" },
  },
  methods: {
    closeModal() {
      this.$emit("close");
    },
  },
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
.modal-content {
  background-color: #fff;
  padding: 20px;
  max-width: 90%;
  max-height: 90%;
  overflow-y: auto;
  border-radius: 8px;
  position: relative;
}
.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
}
</style>
