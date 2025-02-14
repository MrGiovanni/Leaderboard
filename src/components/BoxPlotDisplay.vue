<!-- src/components/BoxPlotDisplay.vue -->
<template>
  <div class="box-plot-display">
    <h2>
      Box Plot Analysis for {{ modelName }} ({{ scoreType.toUpperCase() }})
    </h2>
    <div v-if="loading">
      Loading precomputed analysis and generating charts...
    </div>
    <div v-else>
      <!-- 각 그룹 변수별 차트를 생성 -->
      <div
        class="chart-container"
        v-for="(groupKey, index) in groupKeys"
        :key="index"
      >
        <div :id="getChartId(groupKey)" class="plotly-chart"></div>
      </div>
    </div>
  </div>
</template>

<script>
import Plotly from "plotly.js-dist-min";

export default {
  name: "BoxPlotDisplay",
  props: {
    dataset: { type: String, required: true },
    modelName: { type: String, required: true },
    scoreType: { type: String, required: true },
    // organ: "mean" 또는 특정 장기 이름
    organ: { type: String, default: "mean" },
  },
  data() {
    return {
      loading: true,
      precomputedData: null,
    };
  },
  computed: {
    // 그룹 변수 목록 – 여기서는 고정: age, pathology, institute, gender, manufacturer
    groupKeys() {
      return ["age", "pathology", "institute", "gender", "manufacturer"];
    },
  },
  methods: {
    getChartId(groupKey) {
      // 예: "chart-age", "chart-pathology", etc.
      return `chart-${groupKey}`;
    },
    loadPrecomputedData() {
      // 모델 이름에 포함된 공백 등 특수 문자를 URL 인코딩합니다.
      const encodedModelName = encodeURIComponent(this.modelName);
      // precomputed JSON 파일 경로 (예: "jhh_results_raw/precomputed/nnU-Net%20U-Net_dsc_analysis.json")
      const url = `${import.meta.env.BASE_URL}${
        this.dataset
      }_raw/precomputed/${encodedModelName}_${this.scoreType.toLowerCase()}_analysis.json`;
      console.log("Loading JSON from:", url);
      fetch(url)
        .then((response) => response.json())
        .then((data) => {
          console.log("Precomputed data loaded:", data);
          this.precomputedData = data;
          this.loading = false;
          this.$nextTick(() => {
            // 그룹별로 차트를 순차적으로 생성 (1초 간격)
            this.createAllPlots();
          });
        })
        .catch((error) => {
          console.error("Error loading precomputed data:", error);
          this.loading = false;
        });
    },
    createChartForGroup(groupKey, containerId) {
      const groupInfo = this.precomputedData[groupKey];
      if (!groupInfo) return;
      const summary = groupInfo.summary; // 각 그룹의 요약 통계량
      const significance = groupInfo.significance;
      const groupNames = Object.keys(summary).sort(); // (예: 나이 순 정렬)

      let shapes = [];
      let outlierX = [];
      let outlierY = [];
      let tickvals = [];
      let ticktext = [];
      const halfHeight = 0.3; // 각 박스의 높이의 절반 (y축 좌표)

      groupNames.forEach((groupLabel, index) => {
        const y = index + 1;
        tickvals.push(y);
        ticktext.push(groupLabel);
        const s = summary[groupLabel];
        // 박스 (사각형): Q1 ~ Q3
        shapes.push({
          type: "rect",
          xref: "x",
          yref: "y",
          x0: s.q1,
          x1: s.q3,
          y0: y - halfHeight,
          y1: y + halfHeight,
          line: { color: "black" },
          fillcolor: "lightblue",
          opacity: 0.7,
        });
        // 중앙값 선
        shapes.push({
          type: "line",
          xref: "x",
          yref: "y",
          x0: s.median,
          x1: s.median,
          y0: y - halfHeight,
          y1: y + halfHeight,
          line: { color: "red", width: 2 },
        });
        // 왼쪽 whisker: lowerWhisker ~ Q1
        shapes.push({
          type: "line",
          xref: "x",
          yref: "y",
          x0: s.lowerWhisker,
          x1: s.q1,
          y0: y,
          y1: y,
          line: { color: "black", width: 1 },
        });
        // 오른쪽 whisker: Q3 ~ upperWhisker
        shapes.push({
          type: "line",
          xref: "x",
          yref: "y",
          x0: s.q3,
          x1: s.upperWhisker,
          y0: y,
          y1: y,
          line: { color: "black", width: 1 },
        });
        // 왼쪽 캡 (작은 수직선)
        shapes.push({
          type: "line",
          xref: "x",
          yref: "y",
          x0: s.lowerWhisker,
          x1: s.lowerWhisker,
          y0: y - halfHeight / 2,
          y1: y + halfHeight / 2,
          line: { color: "black", width: 1 },
        });
        // 오른쪽 캡
        shapes.push({
          type: "line",
          xref: "x",
          yref: "y",
          x0: s.upperWhisker,
          x1: s.upperWhisker,
          y0: y - halfHeight / 2,
          y1: y + halfHeight / 2,
          line: { color: "black", width: 1 },
        });
        // 이상치(outliers)
        if (s.outliers && s.outliers.length > 0) {
          s.outliers.forEach((val) => {
            outlierX.push(val);
            outlierY.push(y);
          });
        }
      });

      const outlierTrace = {
        x: outlierX,
        y: outlierY,
        mode: "markers",
        type: "scatter",
        marker: { color: "black", size: 6, symbol: "circle-open" },
        name: "Outliers",
      };

      // 항상 최소한 하나의 trace를 전달하도록 함 (빈 trace 전달)
      const dataTraces =
        outlierX.length > 0
          ? [outlierTrace]
          : [{ x: [], y: [], type: "scatter" }];

      const layout = {
        title: `${
          groupKey.charAt(0).toUpperCase() + groupKey.slice(1)
        } (Significance: ${significance})`,
        xaxis: { title: "DSC", range: [0, 1] },
        yaxis: {
          title: groupKey.charAt(0).toUpperCase() + groupKey.slice(1),
          tickmode: "array",
          tickvals: tickvals,
          ticktext: ticktext,
          autorange: "reversed", // 상위 그룹이 위쪽에 오도록
        },
        shapes: shapes,
        margin: { l: 100, r: 20, t: 50, b: 50 },
      };

      Plotly.newPlot(containerId, dataTraces, layout);
    },
    // 그룹별 차트를 순차적으로 처리 (1초 간격)
    createAllPlots() {
      const keys = this.groupKeys;
      let index = 0;
      const processNext = () => {
        if (index >= keys.length) return;
        this.createChartForGroup(keys[index], this.getChartId(keys[index]));
        index++;
        setTimeout(processNext, 1000);
      };
      processNext();
    },
  },
  mounted() {
    this.loadPrecomputedData();
  },
};
</script>

<style scoped>
.box-plot-display {
  padding: 20px;
}
.chart-container {
  margin-bottom: 40px;
}
/* Plotly 차트 컨테이너에 고정 크기 지정 */
.plotly-chart {
  width: 100%;
  height: 400px;
}
</style>
