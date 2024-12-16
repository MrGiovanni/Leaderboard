<template>
    <div>
      <table>
        <thead>
          <tr>
            <th>Model</th>
            <th>Average Score</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in models"
            :key="index"
            :class="{ highlighted: isHighlighted(item.model) }"
            @mouseover="highlightModel(item.model)"
            @mouseleave="clearHighlight"
          >
            <td>{{ item.model }}</td>
            <td>{{ item.score }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
  
  <script>
  import Papa from "papaparse";
  
  export default {
    data() {
      return {
        models: [], // 存储解析后的CSV数据
        highlightedModel: null, // 当前高亮的模型
      };
    },
    mounted() {
      this.loadCSVData();
    },
    methods: {
      // 加载 CSV 文件并解析数据
      loadCSVData() {
        Papa.parse("/dapatlas_results/aorta_summary.csv", {
          download: true,
          header: true,
          complete: (result) => {
            this.models = result.data.filter((row) => row.Model && row.Average_Score).map((row) => ({
              model: row.Model,
              score: parseFloat(row.Average_Score),
              equivalentModels: row.Equivalent_Models
                ? row.Equivalent_Models.split(",").map((m) => m.trim())
                : [],
            })).sort((a, b) => b.score - a.score);
          },
        });
      },
      // 设置高亮模型
      highlightModel(model) {
        this.highlightedModel = model;
      },
      // 清除高亮状态
      clearHighlight() {
        this.highlightedModel = null;
      },
      // 判断是否高亮
      isHighlighted(model) {
        if (!this.highlightedModel) return false;
        const current = this.models.find(
          (item) => item.model === this.highlightedModel
        );
        return model === this.highlightedModel || current?.equivalentModels.includes(model);
      },
    },
  };
  </script>
  
  <style>
  table {
    border-collapse: collapse;
    width: 100%;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  tr {
    transition: background-color 0.3s ease;
  }
  tr.highlighted {
    background-color: #ffd966; /* 更柔和的浅黄色 */
    color: #333333; /* 深灰色文字，增加对比度 */
  }
  </style>
  