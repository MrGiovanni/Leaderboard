<template>
    <div>
      <table>
        <thead>
          <tr>
            <th>Model</th>
            <th>Score</th>
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
  export default {
    data() {
      return {
        models: [
        { model: "Model A", score: 95, similarModels: ["Model B", "Model C"] },
        { model: "Model B", score: 88, similarModels: ["Model A"] },
        { model: "Model C", score: 92, similarModels: ["Model A"] },
        ],
        highlightedModel: null,
      };
    },
    methods: {
      highlightModel(model) {
        this.highlightedModel = model;
      },
      clearHighlight() {
        this.highlightedModel = null;
      },
      isHighlighted(model) {
        if (!this.highlightedModel) return false;
        const similarModels = this.models.find(
          (item) => item.model === this.highlightedModel
        )?.similarModels;
        return model === this.highlightedModel || similarModels?.includes(model);
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
    background-color: #6f670c;
  }
  </style>
  