<template>
    <div>
      <h3>{{ title }}</h3>
      <!-- 数据表格 -->
      <table v-if="data.length > 0">
        <thead>
          <tr>
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
  table {
    border-collapse: collapse;
    width: 100%;
  }
  th,
  td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  tr {
    transition: background-color 0.3s ease;
  }
  tr.highlighted {
    background-color: #ffd966; /* 柔和高亮背景色 */
    color: #333333; /* 字体颜色 */
  }
  p {
    text-align: center;
    font-size: 16px;
    margin-top: 20px;
  }
  </style>
  