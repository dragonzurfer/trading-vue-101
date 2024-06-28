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
import TradingVue from 'trading-vue-js';
import { DataCube } from 'trading-vue-js';
import BandOverlay from './BandOverlay.vue'; // Import the new overlay

export default {
  components: { TradingVue, BandOverlay }, // Register the new overlay
  name: 'chart',
  props: ['id', 'ticker', 'cdata'],
  data() {
    return {
      tooloptions: {
        selectedticker: this.ticker,
      },
      chart: new DataCube(this.cdata),
      width: window.innerWidth,
      height: window.innerHeight - 50, // Adjust height to account for form
      colors: {
        colorBack: '#fff',
        colorGrid: '#eee',
        colorText: '#000',
      },
      overlays: [
        {
          name: 'BandOverlay',
          type: 'BandOverlay',
          data: this.extractHighLowData(this.cdata), // Extract data for the overlay
          settings: {
            color: 'rgba(135, 206, 235, 0.3)', // Optional: custom color
          },
        },
      ],
    };
  },
  watch: {
    cdata: {
      handler(newVal) {
        this.chart = new DataCube(newVal);
        this.updateOverlays(newVal); // Update overlays when data changes
      },
      deep: true,
      immediate: true,
    },
  },
  methods: {
    extractHighLowData(cdata) {
      return cdata.chart.data.map(item => [item[0], item[2], item[3]]); // Extract [time, high, low] pairs
    },
    updateOverlays(cdata) {
      this.overlays = [
        {
          name: 'BandOverlay',
          type: 'BandOverlay',
          data: this.extractHighLowData(cdata),
          settings: {
            color: 'rgba(135, 206, 235, 0.3)', // Optional: custom color
          },
        },
      ];
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
