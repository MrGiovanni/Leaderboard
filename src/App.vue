<template>
  <div id="app">
    <h1>模型分数可视化</h1>

    <label>选择数据集：</label>
    <select v-model="selectedDataset" @change="debouncedLoadData">
      <option value="dapatlas_results">Dapatlas Results</option>
      <option value="jhh_results">JHH Results</option>
      <option value="totalsegmentator_results">TotalSegmentator Results</option>
    </select>

    <label>选择类别：</label>
    <select v-model="selectedClass" @change="debouncedLoadData">
      <option v-for="category in categories" :key="category" :value="category">
        {{ category }}
      </option>
    </select>

    <label>选择分数：</label>
    <select v-model="selectedScoreType" @change="debouncedLoadData">
      <option value="dsc">DSC</option>
      <option value="nsd">NSD</option>
    </select>

    <!-- 表格组件 -->
    <HighlightTable
      :data="tableData"
      :scoreType="selectedScoreType"
    />
  </div>
</template>

<script>
import Papa from "papaparse";
import HighlightTable from "./components/HighlightTable.vue";
import _ from "lodash";

export default {
  components: {
    HighlightTable,
  },
  data() {
    return {
      selectedDataset: "dapatlas_results", // 默认数据集
      selectedScoreType: "dsc", // 默认分数类型
      selectedClass: "aorta", // 默认类别
      categories: [
        "aorta",
        "gall_bladder",
        "kidney_left",
        "kidney_right",
        "liver",
        "pancreas",
        "postcava",
        "spleen",
        "stomach",
      ],
      tableData: [], // 存储表格数据
    };
  },
  mounted() {
    this.loadData();
  },
  methods: {
    loadData() {
      const filePath = `/${this.selectedDataset}/${this.selectedScoreType.toLowerCase()}/${this.selectedClass}_summary.csv`;
      console.log("加载文件路径:", filePath); // 调试信息
      Papa.parse(filePath, {
        download: true,
        header: true,
        skipEmptyLines: true, // 跳过空行
        transform: (value) => (value === undefined ? "" : value), 
        complete: (result) => {
          if (result.errors.length > 0) {
            console.error("CSV 文件解析错误:", result.errors);
            alert("无法加载文件，请检查路径是否正确");
            this.tableData = [];
          } else {
            this.tableData = result.data.filter((row) => row.Model).sort((a, b) => b.Average_Score - a.Average_Score);;
            console.log("tableData:", this.tableData);
          }
        },
        error: (error) => {
          console.error("加载文件失败:", error.message);
          alert("加载文件失败，请检查文件路径！");
          this.tableData = [];
        },
      });
    },
    debouncedLoadData: _.debounce(function () {
      this.loadData();
    }, 300),
  },
};
</script>

<style>
h1 {
  text-align: center;
  margin: 20px 0;
}
label {
  margin-right: 10px;
}
select {
  margin-right: 20px;
  padding: 5px;
  font-size: 14px;
}
</style>
