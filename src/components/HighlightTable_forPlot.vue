<!-- src/components/HighlightTable.vue -->
<template>
  <table class="highlight-table">
    <thead>
      <tr>
        <th>Model Name</th>
        <th>Median</th>
        <!-- 필요시 추가 열들 -->
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="(row, index) in data"
        :key="index"
        @click="selectModel(row.Model_name)"
      >
        <td>{{ row.Model_name }}</td>
        <td>{{ row.Median }}</td>
      </tr>
    </tbody>
  </table>
</template>

<script>
export default {
  name: "HighlightTable",
  props: {
    data: { type: Array, default: () => [] },
    scoreType: { type: String, default: "dsc" },
  },
  methods: {
    selectModel(modelName) {
      this.$emit("model-selected", modelName);
    },
  },
};
</script>

<style scoped>
.highlight-table {
  width: 100%;
  border-collapse: collapse;
}
.highlight-table th,
.highlight-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}
.highlight-table tr:hover {
  background-color: #f1f1f1;
  cursor: pointer;
}
</style>
