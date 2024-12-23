<template>
    <div>
      <h3>{{ title }}</h3>
      <!-- 数据表格 -->
      <table v-if="data.length > 0">
        <thead>
          <tr>
            <th>Rank</th>
            <th>Model</th>
            <th>{{ scoreType }}Median Score</th>
            <th>95% CI</th>
            <th>Organization</th>
            <!-- <th>{{ scoreType }} Score Mean</th> -->
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in data"
            :key="index"
            :class="{ highlighted: isHighlighted(item.Model_name) }"
            @mouseover="highlightModel(item.Model_name, item.Likely_models)"
            @mouseleave="clearHighlight"
          >
            <td>{{ index + 1 }}</td>
            <td>{{ item.Model_name }}</td>
            <td>{{ Number(item.Median*100).toFixed(1) }}</td>
            <td>  {{item["95% CI"]}}</td>
            <td>{{ item.organization }}</td>
            <!-- <td>{{ item.Mean }}</td> -->
          </tr>
        </tbody>
      </table>
      <!-- 数据为空时的提示 -->
      <p v-else>No Data</p>
    </div>
  </template>
  
  <script>export default {
    props: {
      data: {
        type: Array,
        required: true,
      },
      scoreType: {
        type: String,
        default: "DSC",
      },
      title: {
        type: String,
        default: "Model Score Table",
      },
    },
    data() {
      return {
        highlightedModel: null, // 当前高亮的模型
        relatedModels: [], // 相关的模型数组
      };
    },
    methods: {
      // 鼠标悬浮事件
      highlightModel(modelName, likelyModels) {
    this.highlightedModel = modelName;

    try {
      // 替换单引号为双引号并解析为数组
      this.relatedModels = likelyModels
        ? JSON.parse(likelyModels.replace(/'/g, '"'))
        : [];
    } catch (error) {
      console.error("解析 likelyModels 失败:", error);
      // 如果解析失败，设置为空数组
      this.relatedModels = [];
    }
  },

      parseArrayString(rawString) {
        try {
          // 替换单引号为双引号
          const jsonString = rawString.replace(/'/g, '"');
          // 解析为数组
          return JSON.parse(jsonString);
        } catch (error) {
          console.error("解析失败:", error);
          return []; // 返回空数组作为默认值
        }
      },
  
      // 鼠标移出事件
      clearHighlight() {
        this.highlightedModel = null;
        this.relatedModels = [];
      },
  
      // 判断是否高亮
      isHighlighted(modelName) {
        // 条件 1：当前模型是高亮模型
        const isCurrentModel = modelName === this.highlightedModel;
  
        // 条件 2：当前模型在相关模型数组中
        const isRelatedModel = this.relatedModels.includes(modelName);
  
        // 返回是否高亮
        return isCurrentModel || isRelatedModel;
      },
    },
  };
  
  </script>
  
  <style scoped>
/* 整体页面样式 */
div {
  font-family: Arial, sans-serif;
  color: #060303; /* 浅灰文字颜色 */
  background-color: #1e1e1e; /* 深色背景 */
  padding: 20px;
  min-height: 100vh;
  max-width: 100%;
  box-sizing: border-box;
}

h3 {
  text-align: center;
  color: #ffffff;
  font-size: 24px;
  margin-bottom: 20px;
}

/* 表格样式 */
table {
  border-collapse: collapse;
  width: auto;
  margin: 0 auto;
  background-color: transparent; /* 表格背景颜色 */
  border: 1px solid #444;
  border-radius: 8px; /* 圆角边框 */
  overflow: hidden; /* 隐藏溢出边角 */
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.5); /* 阴影效果 */
}

th, td {
  border: 1px solid #444;
  padding: 4px 8px;
  text-align: center;
  font-size: 14px;
  color: #f0f0f0;
}

th {
  background-color: #3a3a3a; /* 表头背景颜色 */
  color: #f0f0f0;
  text-transform: uppercase;
}

/* 特定列宽度调整 */
th:nth-child(4), 
td:nth-child(4) {
  width: 25% !important; /* 扩展 95% CI 列宽 */
  word-wrap: break-word; /* 自动换行 */
  white-space: normal; /* 确保多行显示 */
}

/* 其他列宽度调整（根据需求适配） */
th:nth-child(1), 
td:nth-child(1) {
  width: 10%; /* Rank 列 */
}

th:nth-child(2), 
td:nth-child(2) {
  width: 30%; /* Model 列 */
}

th:nth-child(3), 
td:nth-child(3) {
  width: 15%; /* Median Score 列 */
}

th:nth-child(5), 
td:nth-child(5) {
  width: 20%; /* Organization 列 */
}

tbody tr:nth-child(even) {
  background-color: #313131; /* 奇偶行背景色 */
}

tbody tr:hover {
  background-color: #555555; /* 悬停时背景颜色 */
  color: #ffffff;
  transition: background-color 0.3s ease;
}

tr.highlighted {
  background-color: #deb334 !important; /* 柔和高亮背景 */
  color: #333333 !important;
  font-weight: bold; /* 加粗文字 */
  transition: background-color 0.3s ease, color 0.3s ease;
}

select, label {
  display: block;
  margin: 10px auto;
  font-size: 14px;
  color: #f0f0f0;
}

select {
  width: 50%;
  padding: 8px 12px;
  background-color: #2a2a2a;
  color: #ffffff;
  border: 1px solid #444;
  border-radius: 4px;
  outline: none;
  cursor: pointer;
}

select:hover {
  border-color: #888;
}

p {
  text-align: center;
  font-size: 16px;
  margin-top: 20px;
  color: #ff8c00; /* 提示文字颜色 */
}

  </style>
  
  