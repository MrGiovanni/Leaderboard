<template>
  <div id="app">
    <h1>Visualization of TouchStone Experiment Results</h1>
    <div>
      <a class="iconItem" data-tip="Github" target="_blank" href="https://github.com/MrGiovanni/Touchstone">
        <svg t="1704870335945" class="icon" viewBox="0 0 1024 1024" version="1.1"
            xmlns="http://www.w3.org/2000/svg" p-id="2487" width="22px" height="22px">
            <path
                d="M511.6 76.3C264.3 76.2 64 276.4 64 523.5 64 718.9 189.3 885 363.8 946c23.5 5.9 19.9-10.8 19.9-22.2v-77.5c-135.7 15.9-141.2-73.9-150.3-88.9C215 726 171.5 718 184.5 703c30.9-15.9 62.4 4 98.9 57.9 26.4 39.1 77.9 32.5 104 26 5.7-23.5 17.9-44.5 34.7-60.8-140.6-25.2-199.2-111-199.2-213 0-49.5 16.3-95 48.3-131.7-20.4-60.5 1.9-112.3 4.9-120 58.1-5.2 118.5 41.6 123.2 45.3 33-8.9 70.7-13.6 112.9-13.6 42.4 0 80.2 4.9 113.5 13.9 11.3-8.6 67.3-48.8 121.3-43.9 2.9 7.7 24.7 58.3 5.5 118 32.4 36.8 48.9 82.7 48.9 132.3 0 102.2-59 188.1-200 212.9 23.5 23.2 38.1 55.4 38.1 91v112.5c0.8 9 0 17.9 15 17.9 177.1-59.7 304.6-227 304.6-424.1 0-247.2-200.4-447.3-447.5-447.3z"
                fill="#FFFFFF" p-id="2488"></path>

        </svg>
      </a>
      <a class="iconItem" data-tip="Mail" target="_blank" href="mailto:zhaozeyu9185@gmail.com">
        <svg t="1704870588438" class="icon" viewBox="0 0 1024 1024" version="1.1"
            xmlns="http://www.w3.org/2000/svg" p-id="3174" width="22px" height="22px">
            <path
                d="M926.47619 355.644952V780.190476a73.142857 73.142857 0 0 1-73.142857 73.142857H170.666667a73.142857 73.142857 0 0 1-73.142857-73.142857V355.644952l304.103619 257.828572a170.666667 170.666667 0 0 0 220.745142 0L926.47619 355.644952zM853.333333 170.666667a74.044952 74.044952 0 0 1 26.087619 4.778666 72.704 72.704 0 0 1 30.622477 22.186667 73.508571 73.508571 0 0 1 10.678857 17.67619c3.169524 7.509333 5.12 15.652571 5.607619 24.210286L926.47619 243.809524v24.380952L559.469714 581.241905a73.142857 73.142857 0 0 1-91.306666 2.901333l-3.632762-2.925714L97.52381 268.190476v-24.380952a72.899048 72.899048 0 0 1 40.155428-65.292191A72.97219 72.97219 0 0 1 170.666667 170.666667h682.666666z"
                fill="#FFFFFF" p-id="3175"></path>
        </svg>
      </a>
    </div>

    <br>

    <div style="font-family: Arial, sans-serif; color: #dcdcdc; background-color: #1e1e1e; padding: 20px; border-radius: 8px;">
      <h1>Touchstone Benchmark Leaderboard</h1>
      <p>
        This webpage showcases the <strong>Touchstone Benchmark leaderboard</strong>, which evaluates the performance of various models across different organs. The leaderboard utilizes three datasets, as shown below:
      </p>
      <ul>
        <li>Proprietary <a href="https://www.sciencedirect.com/science/article/pii/S2211568419301391" style="color: #1e90ff;">JHH dataset</a> (<em>N=5,172</em>)</li>
        <li>Public <a href="https://github.com/wasserth/TotalSegmentator" style="color: #1e90ff;">TotalSegmentator V2 dataset</a> (<em>N=1,228</em>)</li>
        <li>Public <a href="https://github.com/alexanderjaus/AtlasDataset" style="color: #1e90ff;">DAP Atlas dataset</a> (<em>N=533</em>)</li>
      </ul>
      <p>
        Two scoring metrics are used for the evaluation:
      </p>
      <ul>
        <li><strong>DSC (Dice Similarity Coefficient)</strong>: A commonly used metric for segmentation accuracy.</li>
        <li><strong>NSD (Normalized Surface Dice)</strong>: A metric that assesses the boundary alignment of the segmentation.</li>
      </ul>
      <p>
        Additionally, the leaderboard highlights models that show <strong>no significant differences</strong> in performance for specific tasks. These models are visually marked with <span style="background-color: #deb334; color: black;">yellow highlights</span> to provide clear comparisons.
      </p>
    </div>
    <br>

    <!-- 控制面板 -->
    <div class="control-panel">
      <div class="form-row">
        <label>Category</label>
        <select v-model="selectedDataset" @change="debouncedLoadData">
          <option value="dapatlas_results">Dapatlas Results</option>
          <option value="jhh_results">JHH Results</option>
          <option value="totalsegmentator_results">TotalSegmentator Results</option>
        </select>
      </div>


      <div class="form-row">
        <label>Choose Class</label>
        <select v-model="selectedClass" @change="debouncedLoadData">
          <option v-for="category in categories" :key="category" :value="category">
            {{ category === 'mean' ? 'Overall' : category }}
          </option>
        </select>
      </div>

      <div class="form-row">
        <label>Choose Score</label>
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
  background-color: transparent;
  padding: 20px; /* 内边距 */
  width: 2500px; /* 横向填满页面 */
  height: 100%; /* 垂直填满页面 */
  min-height: 100vh; /* 确保即使页面内容较少时也填满视口高度 */
  margin: 0; /* 去掉默认外边距 */
  box-sizing: border-box; /* 包括内边距和边框在宽高计算中 */
  display: flex; /* 使用 Flex 布局便于内部元素居中或排列 */
  flex-direction: column; /* 子元素按列排列 */
}


/* 标题样式 */
h1 {
  text-align: left;
  color: #f5f5f5; /* 更亮的灰白色 */
  margin-bottom: 20px; /* 标题和其他元素之间的间距 */
}

/* 控制面板样式 */
.control-panel {
  display: flex;
  flex-direction: row; /* 横向排列 */
  gap: 15px; /* 子元素间距 */
  padding: 15px;
  background-color: #1e1e1e; /* 深灰色背景 */
  border: 1px solid #333; /* 边框颜色 */
  border-radius: 8px; /* 圆角边框 */
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.8); /* 浅阴影 */
  width: 100%; /* 宽度自适应页面 */
  margin: 0 auto; /* 居中显示 */
}

.form-row {
  display: flex;
  flex-direction: column; /* 标签和下拉框垂直排列 */
  justify-content: space-between;
  padding: 15px; /* 内边距，给框内容留空间 */
  border: 1px solid #333; /* 边框颜色 */
  border-radius: 8px; /* 圆角边框 */
  background-color: #2a2a2a; /* 深灰色背景 */
  color: #ffffff; /* 白色字体 */
  width: 200px; /* 固定宽度 */
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5); /* 添加阴影 */
}

.form-row label {
  color: #ffffff;
  font-size: 14px;
  margin-bottom: 0; /* 避免垂直对齐问题 */
  text-align: left; /* 左对齐 */
}

.form-row select {
  background-color: #4a4a4a;
  color: #ffffff;
  border: none;
  border-radius: 5px;
  padding: 8px;
  font-size: 14px;
  appearance: none;
  outline: none;
  cursor: pointer;
  width: 80%; /* 可调整宽度，确保样式整齐 */
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

</style>
