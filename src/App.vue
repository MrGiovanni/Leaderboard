<template>
  <div id="app">
    <h1>Ranking of each class</h1>
    <HighlightTable
      v-for="(file, index) in csvFiles"
      :key="index"
      :filePath="file.path"
      :title="file.title"
    />
  </div>
</template>

<script>
import HighlightTable from "./components/HighlightTable.vue";

export default {
  components: {
    HighlightTable,
  },
  data() {
    return {
      csvFiles: [
        { path: "/dapatlas_results/aorta_summary.csv", title: "Aorta Summary" },
        { path: "/dapatlas_results/gall_bladder_summary.csv", title: "Gall Bladder Summary" },
        { path: "/dapatlas_results/kidney_left_summary.csv", title: "Kidney Left Summary" },
        { path: "/dapatlas_results/kidney_right_summary.csv", title: "Kidney Right Summary" },
        { path: "/dapatlas_results/liver_summary.csv", title: "Liver Summary" },
        { path: "/dapatlas_results/pancreas_summary.csv", title: "Pancreas Summary" },
        { path: "/dapatlas_results/postcava_summary.csv", title: "Postcava Summary" },
        { path: "/dapatlas_results/spleen_summary.csv", title: "Spleen Summary" },
        { path: "/dapatlas_results/stomach_summary.csv", title: "Stomach Summary" },
      ],
    };
  },
};
</script>

<style>
h1 {
  text-align: center;
  margin-bottom: 20px;
}
h3 {
  margin-top: 30px;
  text-align: center;
  color: #333;
}
</style>
