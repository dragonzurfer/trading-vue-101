<template>
  <trading-vue
    :title-txt="tooloptions.selectedticker"
    :toolbar="true"
    :overlays="overlays"
    :data="chart"
    :width="width"
    :height="height"
    :color-back="colors.colorBack"
    :color-grid="colors.colorGrid"
    :color-text="colors.colorText"
    :color-title="colors.tvTitle"
    :color-cross="colors.cross"
    :color-candle-dw="colors.candle_dw"
    :color-wick-dw="colors.wick_dw"
    :index-based="true"
    :id="id"
    :ref="id"
  >
  </trading-vue>
</template>

<script>
import TradingVue from "trading-vue-js";
import { DataCube } from "trading-vue-js";

export default {
  components: { TradingVue },
  name: "chart",
  props: ["id", "ticker", "cdata"],
  data() {
    return {
      tooloptions: {
        selectedticker: this.ticker,
      },
      chart: new DataCube(this.cdata),
      width: window.innerWidth,
      height: window.innerHeight - 50, // Adjust height to account for form
      colors: {
        colorBack: "#fff",
        colorGrid: "#eee",
        colorText: "#000",
      },
      overlays: [],
    };
  },
  watch: {
    cdata: {
      handler(newVal) {
        this.chart = new DataCube(newVal);
      },
      deep: true,
      immediate: true,
    },
  },
};
</script>

<style scoped>
trading-vue {
  width: 100%;
  height: 100%;
}
</style>
