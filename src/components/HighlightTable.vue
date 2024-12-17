<template>
    <div>
      <h3>{{ title }}</h3>
      <!-- 数据表格 -->
      <table v-if="data.length > 0">
        <thead>
          <tr>
            <th>Rank</th>
            <th>Model</th>
            <th>{{ scoreType }} Score</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in data"
            :key="index"
            :class="{ highlighted: isHighlighted(item.Model, item.Equivalent_Models) }"
            @mouseover="highlightModel(item.Model)"
            @mouseleave="clearHighlight"
          >
            <td>{{ index + 1 }}</td>
            <td>{{ item.Model }}</td>
            <td>{{ item.Average_Score }}</td>
          </tr>
        </tbody>
      </table>
      <!-- 数据为空时的提示 -->
      <p v-else>暂无数据，请检查输入内容。</p>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      data: {
        type: Array,
        required: true, // 父组件传递的数据
      },
      scoreType: {
        type: String,
        default: "DSC", // 分数类型
      },
      title: {
        type: String,
        default: "模型分数表格",
      },
    },
    data() {
      return {
        highlightedModel: null, // 当前高亮的模型
      };
    },
    methods: {
      // 设置当前高亮的模型
      highlightModel(model) {
        this.highlightedModel = model;
      },
      // 清除高亮
      clearHighlight() {
        this.highlightedModel = null;
      },
      // 判断是否需要高亮
      isHighlighted(model, equivalentModels) {
        if (!this.highlightedModel) return false;
  
        // 解析 Equivalent_Models 为数组（如果存在）
        const equivalents = equivalentModels
          ? equivalentModels.split(",").map((m) => m.trim())
          : [];
  
        // 判断当前模型是否为高亮的模型或等效模型
        return (
          model === this.highlightedModel ||
          equivalents.includes(this.highlightedModel)
        );
      },
    },
  };
  </script>
  
  <style scoped>
  /* 整体页面样式 */
  div {
    font-family: Arial, sans-serif;
    color: #f0f0f0; /* 浅灰文字颜色 */
    background-color: #1e1e1e; /* 深色背景 */
    padding: 20px;
    min-height: 100vh;
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
    width: 90%;
    margin: 0 auto;
    background-color: #2a2a2a; /* 表格背景颜色 */
    border: 1px solid #444;
    border-radius: 8px; /* 圆角边框 */
    overflow: hidden; /* 隐藏溢出边角 */
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.5); /* 阴影效果 */
  }
  
  th, td {
    border: 1px solid #444;
    padding: 12px 16px;
    text-align: center;
    font-size: 14px;
    color: #f0f0f0;
  }
  
  th {
    background-color: #3a3a3a; /* 表头背景颜色 */
    color: #f0f0f0;
    text-transform: uppercase;
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
  
  