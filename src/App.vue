<template>
  <div id="app">
    <!-- Header Links -->
    <header class="header-links">
      <a
        class="iconItem"
        data-tip="GitHub"
        target="_blank"
        href="https://github.com/MrGiovanni/Touchstone"
      >
        <img
          src="@/assets/github.png"
          alt="GitHub Logo"
          class="icon"
          width="22"
          height="22"
        />
      </a>
      <a
        class="iconItem"
        data-tip="Mail"
        target="_blank"
        href="mailto:zhaozeyu9185@gmail.com"
      >
        <img
          src="@/assets/email.png"
          alt="Email Logo"
          class="icon"
          width="22"
          height="22"
        />
      </a>
    </header>

    <!-- Hero Section -->
    <section class="hero-section">
      <!-- Stars (Bubble Effect) -->
      <Stars :numStars="150" />
      <div class="hero-content">
        <h1 class="hero-title">Touchstone Benchmark Leaderboard</h1>
        <div class="hero-description">
          <p>
            Dive into our <strong>Touchstone Benchmark leaderboard</strong>,
            which evaluates the performance of various models across different
            organs.
            <br />
            The leaderboard utilizes two datasets, as shown below.
          </p>
        </div>
        <div class="hero-button-group">
          <a
            class="btn btn-primary"
            href="https://www.sciencedirect.com/science/article/pii/S2211568419301391"
            target="_blank"
          >
            Proprietary JHH dataset (N=5,172)
          </a>
          <a
            class="btn btn-primary"
            href="https://github.com/wasserth/TotalSegmentator"
            target="_blank"
          >
            Public TotalSegmentator V2 dataset (N=1,228)
          </a>
        </div>

        <!-- Marquee with Models -->
        <div class="marquee-container">
          <div class="marquee-title">
            <p>Evaluate <strong>15+ models</strong> across organs.</p>
          </div>
          <div class="marquee-content">
            <div
              v-for="(model, index) in models"
              :key="index"
              class="model-box"
            >
              <span class="model-name">{{ model }}</span>
            </div>
            <div
              v-for="(model, index) in models"
              :key="'dup-' + index"
              class="model-box"
            >
              <span class="model-icon">🤖</span>
              <span class="model-name">{{ model }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Info Boxes -->
    <div class="info-boxes">
      <!-- Box 1: Chart -->
      <div class="info-box box1">
        <div class="chart-wrapper">
          <BarChart :chart-data="chartData" :chart-options="chartOptions" />
        </div>
      </div>

      <!-- Box 2: Additional Info -->
      <div class="info-box box2">
        <p class="text-detail">
          <h2>Leaders</h2>
          <p><strong>MedNeXT</strong> and <strong>MedFormer</strong>, based on our scoring methodology, these models
          scored both <strong>90.0</strong>. We measured performance using <strong>DSC</strong> and <strong>NSD</strong>. Data being non-parametric, we apply <strong>the Wilcoxon paired test</strong>
          . A p-value above 0.05 indicates no significant difference.
          </p>
        </p>
      </div>
    </div>

    <!-- Control Panel with Dropdowns -->
    <div class="control-panel">
      <div class="form-row">
        <label for="dataset-select">Choose Dataset</label>
        <AnimatedDropdown v-model="selectedDataset" :options="datasetOptions" />
      </div>
      <div class="form-row">
        <label for="organ-select">Choose Organs</label>
        <AnimatedDropdown v-model="selectedClass" :options="organOptions" />
      </div>
      <div class="form-row">
        <label for="score-select">Choose Score Type</label>
        <AnimatedDropdown
          v-model="selectedScoreType"
          :options="scoreTypeOptions"
        />
      </div>
    </div>

    <!-- Loading Indicator -->
    <div v-if="isLoading" class="loading">Loading, please wait...</div>

    <!-- Data Table -->
    <HighlightTable v-else :data="tableData" :scoreType="selectedScoreType" />

    <!-- Visitor Count -->
    <!-- <footer class="visitor-counter">Visitors: {{ visitorCount }}</footer> -->
  </div>
</template>

<script setup>
import { ref, watch, onMounted, computed } from "vue";
import Papa from "papaparse";
import { debounce } from "lodash-es";

// Import components
import AnimatedDropdown from "./components/AnimatedDropdown.vue";
import HighlightTable from "./components/HighlightTable.vue";
import BarChart from "./components/BarChart.vue";

// Sample models for the marquee
const models = ref([
  "MedNeXt",
  "MedFormer",
  "STU-Net B",
  "STU-Net H",
  "nnU-Net U-Net",
  "STU-Net L",
  "nnU-Net_ResEncL",
  "UniSeg",
  "Diff-UNet",
  "NexToU",
  "LHU-Net",
  "SegVol",
  "U-Net_CLIP",
  "Swin_UNETR_CLIP",
  "Swin_UNETR",
  "UNEST",
  "UNETR",
  "UCTransNet",
  "SAM-Adapter",
]);

// Raw scores for the chart
const rawScores = [
  { name: "MedNeXt", score: 90.0, color: "#FF6E7F" },
  { name: "MedFormer", score: 90.0, color: "#F76C85" },
  { name: "STU-Net B", score: 89.9, color: "#ffc3ca" },
  { name: "STU-Net H", score: 89.8, color: "#ffc3ca" },
  { name: "nnU-Net U-Net", score: 89.8, color: "#ffc3ca" },
];

// Computed chart data for Chart.js
const chartData = computed(() => ({
  labels: rawScores.map((item) => item.name),
  datasets: [
    {
      label: "DSC",
      data: rawScores.map((item) => item.score),
      backgroundColor: rawScores.map((item) => item.color),
      borderColor: rawScores.map((item) => item.color),
      borderWidth: 1,
    },
  ],
}));

// Chart options
const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    x: {
      grid: {
        display: false,
      },
    },
    y: {
      beginAtZero: true,
      suggestedMax: 100,
      grid: {
        display: false,
      },
    },
  },
};

// State variables for dropdowns and table
const selectedDataset = ref("jhh_results");
const selectedClass = ref("mean");
const selectedScoreType = ref("dsc");
const visitorCount = ref(0);
const tableData = ref([]);
const isLoading = ref(false);

// Dropdown options
const datasetOptions = [
  { value: "jhh_results", label: "JHH Results" },
  { value: "totalsegmentator_results", label: "TotalSegmentator Results" },
];

const organOptions = [
  { value: "mean", label: "Overall" },
  { value: "aorta", label: "Aorta" },
  { value: "gall_bladder", label: "Gall Bladder" },
  { value: "kidney_left", label: "Kidney Left" },
  { value: "kidney_right", label: "Kidney Right" },
  { value: "liver", label: "Liver" },
  { value: "pancreas", label: "Pancreas" },
  { value: "postcava", label: "Postcava" },
  { value: "spleen", label: "Spleen" },
  { value: "stomach", label: "Stomach" },
];

const scoreTypeOptions = [
  { value: "dsc", label: "DSC" },
  { value: "nsd", label: "NSD" },
];

// CSV loading function using Papa Parse
const loadData = () => {
  isLoading.value = true;
  const filePath = `${import.meta.env.BASE_URL}${
    selectedDataset.value
  }/${selectedScoreType.value.toLowerCase()}/${selectedClass.value}_table.csv`;
  console.log("Loading file:", filePath);
  Papa.parse(filePath, {
    download: true,
    header: true,
    skipEmptyLines: true,
    transform: (value) => (value === undefined ? "" : value),
    complete: (result) => {
      isLoading.value = false;
      if (result.errors.length > 0) {
        console.error("CSV parsing errors:", result.errors);
        alert("Unable to load file. Please check the file path.");
        tableData.value = [];
      } else {
        tableData.value = result.data
          .filter((row) => row.Model_name)
          .map((row) => {
            row.Model_name = row.Model_name.replace(/^_+/, "");
            return row;
          })
          .sort((a, b) => parseFloat(b.Median) - parseFloat(a.Median));
      }
    },
    error: (error) => {
      isLoading.value = false;
      console.error("Error loading file:", error.message);
      alert("Failed to load file. Please check the file path.");
      tableData.value = [];
    },
  });
};

const debouncedLoadData = debounce(loadData, 300);

watch([selectedDataset, selectedClass, selectedScoreType], () => {
  debouncedLoadData();
});

// Increment visitor count using countapi
const incrementVisitorCount = () => {
  fetch(
    "https://api.countapi.xyz/hit/mrgiovanni.github.io/Leaderboard/visitors"
  )
    .then((res) => res.json())
    .then((data) => {
      visitorCount.value = data.value;
    })
    .catch((error) => console.error("Error updating visitor count:", error));
};

onMounted(() => {
  loadData();
  incrementVisitorCount();
});
</script>

<style scoped>
/* Global App Styles */
#app {
  background-color: #ffffff;
  color: #333333;
  margin: 0 auto;
  max-width: 1200px;
  padding: 5px;
}

/* Header Links */
.header-links {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
}
.iconItem {
  text-decoration: none;
}
.icon {
  display: block;
}

/* Hero Section */
.hero-section {
  position: relative;
  width: 100vw;
  min-height: 550px;
  margin-left: calc(50% - 50vw);
  padding: 4rem 1rem;
  box-sizing: border-box;
  background: linear-gradient(45deg, #ff6e7f 0%, #c54ffb 50%, #5ac8fa 100%);
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 30px;
}
.hero-content {
  position: relative;
  z-index: 1;
  max-width: 1000px;
  width: 100%;
  text-align: center;
}
.hero-title {
  font-size: 3rem;
  font-weight: 600;
  margin-bottom: 1rem;
  color: #ffffff;
}
.hero-description {
  width: 100%;
  line-height: 1.6;
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
  color: #ffffff;
}
.hero-button-group {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 1rem;
}

/* Button Styles */
.btn {
  padding: 0.7rem 1.5rem;
  font-size: 1rem;
  border-radius: 40px;
  border: none;
  cursor: pointer;
  transition: all 0.3s ease;
  text-decoration: none;
  margin: 0.3rem;
}
.btn-primary {
  background: linear-gradient(to bottom, #217aff 0%, #1c64d1 100%);
  color: #fff;
  font-weight: 500;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
.btn-primary:hover {
  transform: scale(1.05);
}
.btn-secondary {
  background-color: #ffffff;
  color: #000000;
  font-weight: 500;
  border: 1px solid #e0e0e0;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}
.btn-secondary:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 10px rgba(0, 0, 0, 0.1);
}

/* Marquee Styles */
.marquee-container {
  width: 100%;
  overflow: hidden;
  position: relative;
  padding: 40px;
}
.marquee-title {
  font-size: 1.25rem;
  margin-bottom: 10px;
  color: #ffffff;
}
.marquee-content {
  display: flex;
  gap: 16px;
  animation: scroll 30s linear infinite;
  white-space: nowrap;
}
.model-box {
  display: flex;
  align-items: center;
  gap: 8px;
  background: #f8f9fa;
  border-radius: 50px;
  padding: 6px 20px;
  font-size: 16px;
  font-weight: 500;
  color: #1c1c1e;
  flex-shrink: 0;
  box-shadow: 0 4px 14px rgba(255, 110, 127, 0.45),
    0 6px 16px rgba(197, 79, 251, 0.35), 0 8px 18px rgba(90, 200, 250, 0.3);
}
.model-box:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 10px rgba(0, 0, 0, 0.1);
}
.model-icon {
  font-size: 20px;
}

.info-boxes {
  display: flex;
  gap: 20px;
  margin-bottom: 30px;
  justify-content: center;
  flex-wrap: nowrap;
}

.info-box.box1 {
  background-color: #ffffff;
  border-radius: 20px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  box-shadow: 0 4px 14px rgba(255, 110, 127, 0.45),
    0 6px 16px rgba(197, 79, 251, 0.35), 0 8px 18px rgba(90, 200, 250, 0.3);
}

.info-box.box2 {
  background-color: #ffffff; /* White background */
  border-radius: 16px; /* Rounded corners */
  padding: 24px; /* Padding for spacing */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  width: 100%;
  max-width: 450px; /* Restrict max width */
  text-align: left; /* Left-aligned text */
}

/* Title Styling */
.info-box.box2 h2 {
  font-size: 30px;
  font-weight: 600;
  color: #333;
  margin-bottom: 10px;
}

/* Paragraph/Text Styling */
.info-box.box2 .text-detail {
  font-size: 16px;
  line-height: 1.6;
  color: #555;
}

/* Responsive Styles */
@media (max-width: 768px) {
  .info-box.box2 {
    max-width: 100%;
    text-align: center;
    align-items: center;
  }
}

@media (min-width: 768px) {
  .info-box.box1 {
    width: 65%;
    height: 400px;
    overflow: hidden;
    position: relative;
  }
  .chart-wrapper {
    width: 100%;
    height: 100%;
  }

  .info-box.box1 canvas {
    width: 100% !important;
    height: 100% !important;
  }
  .info-box.box2 {
    width: 30%;
    height: 400px;
  }
}

@media (max-width: 767px) {
  .info-boxes {
    flex-direction: column;
    align-items: center;
  }
  .info-box {
    width: 90%;
    height: auto;
  }
}

.info-box.box1:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 16px rgba(255, 110, 127, 0.45),
    0 8px 18px rgba(197, 79, 251, 0.35), 0 10px 20px rgba(90, 200, 250, 0.3);
}

/* Control Panel */
.control-panel {
  display: flex;
  gap: 20px;
  padding: 20px;
  background-color: #f8f8f8;
  border: 1px solid #e0e0e0;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
  margin-bottom: 30px;
  transition: transform 0.3s ease;
}
.form-row {
  display: flex;
  flex-direction: column;
}
.form-row label {
  margin-bottom: 5px;
  font-size: 14px;
}

/* Loading Indicator */
.loading {
  text-align: center;
  font-size: 18px;
  color: #007aff;
  margin: 20px 0;
}

/* Visitor Counter */
.visitor-counter {
  margin-top: 40px;
  text-align: center;
  font-size: 1.2rem;
  color: #007aff;
}

/* Data Table Styles */
::v-deep table {
  width: 100%;
  border-collapse: collapse;
  margin: 0;
  padding: 0;
  text-align: center;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: none;
}
::v-deep tbody tr {
  transition: transform 0.2s ease-in-out;
}
::v-deep tbody tr:hover {
  transform: scale(1.02);
  position: relative;
  z-index: 1;
}
::v-deep thead th {
  background-color: #f8485c;
  color: #fff;
  padding: 12px 16px;
  font-weight: 600;
  border-bottom: 1px solid #ddd;
}
::v-deep tbody td {
  padding: 12px 16px;
  border-bottom: 1px solid #eee;
  text-align: center;
}
::v-deep table th:nth-child(1),
::v-deep table td:nth-child(1),
::v-deep table th:nth-child(2),
::v-deep table td:nth-child(2),
::v-deep table th:nth-child(3),
::v-deep table td:nth-child(3),
::v-deep table th:nth-child(4),
::v-deep table td:nth-child(4),
::v-deep table th:nth-child(5) {
  text-align: center;
}

@keyframes slideUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
#app > * {
  animation: slideUp 0.5s ease-out;
}

/* Marquee Animation */
@keyframes scroll {
  from {
    transform: translateX(0%);
  }
  to {
    transform: translateX(-100%);
  }
}
</style>

<style>
@import url("https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap");
body {
  font-family: "Inter", sans-serif;
}
</style>
