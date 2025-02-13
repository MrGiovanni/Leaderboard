<template>
  <div class="table-container">
    <table v-if="data && data.length">
      <thead>
        <tr>
          <th>Rank</th>
          <th @click="handleSort('Model_name')">
            Model
            <span v-if="sortKey === 'Model_name'">{{
              sortOrder === "asc" ? "▲" : "▼"
            }}</span>
          </th>
          <th @click="handleSort('Median')">
            {{ scoreType.toUpperCase() }} Median Score
            <span v-if="sortKey === 'Median'">{{
              sortOrder === "asc" ? "▲" : "▼"
            }}</span>
          </th>
          <th>95% CI</th>
          <th @click="handleSort('organization')">
            Organization
            <span v-if="sortKey === 'organization'">{{
              sortOrder === "asc" ? "▲" : "▼"
            }}</span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(item, index) in sortedData"
          :key="index"
          :class="{ highlighted: isHighlighted(item.Model_name) }"
          @mouseover="highlightModel(item.Model_name, item.Likely_models)"
          @mouseleave="clearHighlight"
        >
          <td>{{ index + 1 }}</td>
          <td>{{ item.Model_name }}</td>
          <td>{{ Number(item.Median * 100).toFixed(1) }}</td>
          <td>{{ item["95% CI"] }}</td>
          <td>{{ item.organization }}</td>
        </tr>
      </tbody>
    </table>
    <p v-else>No Data Available</p>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Array,
      required: true,
    },
    scoreType: {
      type: String,
      default: "dsc",
    },
  },
  data() {
    return {
      highlightedModel: null,
      relatedModels: [],
      sortKey: "",
      sortOrder: "asc",
    };
  },
  computed: {
    sortedData() {
      if (!this.sortKey) return this.data;
      return this.data.slice().sort((a, b) => {
        let aVal = a[this.sortKey];
        let bVal = b[this.sortKey];
        const numA = parseFloat(aVal);
        const numB = parseFloat(bVal);
        if (!isNaN(numA) && !isNaN(numB)) {
          aVal = numA;
          bVal = numB;
        } else {
          aVal = aVal ? aVal.toString().toLowerCase() : "";
          bVal = bVal ? bVal.toString().toLowerCase() : "";
        }
        if (aVal < bVal) return this.sortOrder === "asc" ? -1 : 1;
        if (aVal > bVal) return this.sortOrder === "asc" ? 1 : -1;
        return 0;
      });
    },
  },
  methods: {
    handleSort(key) {
      if (this.sortKey === key) {
        this.sortOrder = this.sortOrder === "asc" ? "desc" : "asc";
      } else {
        this.sortKey = key;
        this.sortOrder = "asc";
      }
    },
    highlightModel(modelName, likelyModels) {
      this.highlightedModel = modelName;
      try {
        this.relatedModels = likelyModels
          ? JSON.parse(likelyModels.replace(/'/g, '"'))
          : [];
      } catch (error) {
        console.error("Failed to parse likelyModels:", error);
        this.relatedModels = [];
      }
    },
    clearHighlight() {
      this.highlightedModel = null;
      this.relatedModels = [];
    },
    isHighlighted(modelName) {
      return (
        modelName === this.highlightedModel ||
        this.relatedModels.includes(modelName)
      );
    },
  },
};
</script>

<style scoped>
.table-container {
  background-color: var(--color-white);
  border: 1px solid var(--color-divider);
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
  margin-bottom: 30px;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 12px 8px;
  border-bottom: 1px solid var(--color-divider);
  text-align: left;
  font-size: 14px;
  color: var(--color-dark-gray);
}

th {
  background-color: var(--color-light-gray);
  cursor: pointer;
  user-select: none;
}

tbody tr:hover {
  background-color: var(--color-yellow);
}

tr.highlighted {
  background-color: var(--color-yellow);
  color: var(--color-white);
}

p {
  text-align: center;
  font-size: 16px;
  color: var(--color-yellow);
  margin-top: 20px;
}
</style>
