<template>
  <div id="app">
    <h1>模型分数可视化</h1>

    <!-- 控制面板 -->
    <div class="control-panel">
      <div class="form-row">
        <label>选择数据集：</label>
        <select v-model="selectedDataset" @change="debouncedLoadData">
          <option value="dapatlas_results">Dapatlas Results</option>
          <option value="jhh_results">JHH Results</option>
          <option value="totalsegmentator_results">TotalSegmentator Results</option>
        </select>
      </div>

      <div class="form-row">
        <label>选择类别：</label>
        <select v-model="selectedClass" @change="debouncedLoadData">
          <option v-for="category in categories" :key="category" :value="category">
            {{ category }}
          </option>
        </select>
      </div>

      <div class="form-row">
        <label>选择分数：</label>
        <select v-model="selectedScoreType" @change="debouncedLoadData">
          <option value="dsc">DSC</option>
          <option value="nsd">NSD</option>
        </select>
      </div>
    </div>

    <!-- 加载状态 -->
    <div v-if="isLoading" class="loading">
      正在加载数据，请稍候...
    </div>

    <!-- 表格组件 -->
    <HighlightTable
      v-else
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
      isLoading: false, // 加载状态
    };
  },
  mounted() {
    this.loadData();
  },
  methods: {
    loadData() {
      this.isLoading = true; // 开始加载
      const filePath = `${import.meta.env.BASE_URL}${this.selectedDataset}/${this.selectedScoreType.toLowerCase()}/${this.selectedClass}_summary.csv`;
      console.log("加载文件路径:", filePath);

      Papa.parse(filePath, {
        download: true,
        header: true,
        skipEmptyLines: true,
        transform: (value) => (value === undefined ? "" : value),
        complete: (result) => {
          this.isLoading = false; // 完成加载
          if (result.errors.length > 0) {
            console.error("CSV 文件解析错误:", result.errors);
            alert("无法加载文件，请检查路径是否正确");
            this.tableData = [];
          } else {
            this.tableData = result.data
              .filter((row) => row.Model)
              .sort((a, b) => parseFloat(b.Average_Score) - parseFloat(a.Average_Score)); // 按分数降序排序
            console.log("tableData:", this.tableData);
          }
        },
        error: (error) => {
          this.isLoading = false;
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

<style scoped>
/* 页面整体样式 */
#app {
  font-family: Arial, sans-serif;
  color: #333;
  background-color: #f4f4f9;
  padding: 20px;
  max-width: 1000px;
  margin: 0 auto;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

/* 标题样式 */
h1 {
  text-align: center;
  color: #2a2a2a;
}

/* 控制面板样式 */
.control-panel {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-bottom: 20px;
}

.form-row {
  display: flex;
  align-items: center;
  margin: 10px;
  width: 100%;
}

label {
  width: 120px;
  margin-right: 10px;
  font-size: 16px;
  color: #444;
}

select {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  background-color: #fff;
  color: #333;
}

select:focus {
  border-color: #888;
  outline: none;
}

/* 加载状态 */
.loading {
  text-align: center;
  font-size: 18px;
  color: #555;
  margin: 20px 0;
}
</style>
