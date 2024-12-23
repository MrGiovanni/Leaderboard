<template>
  <div id="app">
    <h1>Visualization of TouchStone Experiment Results</h1>

    <!-- 控制面板 -->
    <div class="control-panel">
      <div class="form-row">
        <label>Choose Dataset：</label>
        <select v-model="selectedDataset" @change="debouncedLoadData">
          <option value="dapatlas_results">Dapatlas Results</option>
          <option value="jhh_results">JHH Results</option>
          <option value="totalsegmentator_results">TotalSegmentator Results</option>
        </select>
      </div>

      <div class="form-row">
        <label>Choose Class：</label>
        <select v-model="selectedClass" @change="debouncedLoadData">
          <option v-for="category in categories" :key="category" :value="category">
            {{ category === 'mean' ? 'Overall' : category }}
          </option>
        </select>
      </div>

      <div class="form-row">
        <label>Choose Score：</label>
        <select v-model="selectedScoreType" @change="debouncedLoadData">
          <option value="dsc">DSC</option>
          <option value="nsd">NSD</option>
        </select>
      </div>
    </div>

    <!-- 加载状态 -->
    <div v-if="isLoading" class="loading">
      Loading, waiting please...
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
      selectedClass: "mean", // 默认类别
      categories: [
        "mean",
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
      const filePath = `${import.meta.env.BASE_URL}${this.selectedDataset}/${this.selectedScoreType.toLowerCase()}/${this.selectedClass}_table.csv`;
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
              .filter((row) => row.Model_name)
              // .sort((a, b) => parseFloat(b.Mean) - parseFloat(a.Mean));
              .sort((a, b) => parseFloat(b.Median) - parseFloat(a.Median)); // 按分数降序排序
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
  color: #dcdcdc; /* 浅灰色文字 */
  background-color: #000; /* 全页面黑色背景 */
  padding: 20px;
  width: 100%; /* 让内容横向铺满页面 */
  height: 100%; /* 让内容垂直铺满页面 */
  min-height: 100vh;
  margin: 0;
  box-sizing: border-box;
}

/* 表格样式调整 */
table {
  width: 100%; /* 表格铺满页面宽度 */
  border-collapse: collapse;
  margin-top: 20px;
  background-color: #1a1a1a; /* 深黑色表格背景 */
  color: #fff; /* 白色文字 */
  border: 1px solid #333; /* 表格边框 */
  border-radius: 8px;
  overflow: hidden;
}

/* 标题样式 */
h1 {
  text-align: center;
  color: #f5f5f5; /* 更亮的灰白色 */
  margin-bottom: 20px; /* 标题和其他元素之间的间距 */
}

/* 控制面板样式 */
.control-panel {
  display: flex;
  flex-direction: column; /* 纵向排列 */
  gap: 15px; /* 元素间距 */
  padding: 15px;
  background-color: #1e1e1e; /* 深灰色背景 */
  border: 1px solid #333; /* 边框颜色 */
  border-radius: 8px; /* 圆角边框 */
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.8); /* 浅阴影 */
}

.form-row {
  display: flex;
  align-items: center;
  justify-content: space-between; /* 左右对齐 */
  width: 100%;
}

label {
  color: #f5f5f5; /* 白色文字 */
  font-size: 16px;
}

select {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #444; /* 深灰边框 */
  border-radius: 6px;
  font-size: 14px;
  background-color: #2b2b2b; /* 黑灰色背景 */
  color: #fff; /* 白色文字 */
}

select:focus {
  border-color: #666; /* 更亮的灰色边框 */
  outline: none;
  background-color: #3b3b3b; /* 聚焦时的背景色 */
}

/* 加载状态 */
.loading {
  text-align: center;
  font-size: 18px;
  color: #ffcc00; /* 黄色提示文字 */
  margin: 20px 0;
}

/* 表格组件样式 */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  background-color: #1a1a1a; /* 表格深黑背景 */
  color: #fff; /* 白色文字 */
  border: 1px solid #333; /* 表格边框 */
  border-radius: 8px;
  overflow: hidden;
}

th, td {
  padding: 12px 15px;
  text-align: left;
  border: 1px solid #333; /* 单元格边框 */
}

th {
  background-color: #2b2b2b; /* 表头深灰背景 */
  color: #ffcc00; /* 黄色文字 */
}

tbody tr:nth-child(even) {
  background-color: #1e1e1e; /* 偶数行深灰色 */
}

tbody tr:hover {
  background-color: #333333; /* 悬停时颜色 */
}

</style>
