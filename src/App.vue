<template>
  <div id="app">
    <form-component :form="form" @fetch-data="fetchData"></form-component>
    <chart v-if="cData" :id="'customChart'" :ticker="form.ticker" :cdata="cData"></chart>
  </div>
</template>

<script>
import FormComponent from "./FormComponent.vue";
import "./styles.css";

export default {
  components: {
    FormComponent,
    chart: () => import("./Chart.vue"),
  },
  data() {
    return {
      form: {
        ticker: '',
        timeframe: '',
        from: '',
        to: '',
      },
      cData: null,
      ws: null,
      intervalId: null,
      lastCandleCreationTime: null,
    };
  },
  created() {
    this.setupWebSocket();
    this.setupPeriodicRefresh();
  },
  methods: {
    setupWebSocket() {
      this.ws = new WebSocket('wss://tradingfno.com/ws');
      this.ws.onopen = () => {
        this.ws.send('subscribe:NSE:NIFTY50-INDEX');
      };
      this.ws.onmessage = (message) => {
        const [ticker, priceStr] = message.data.match(/\[([^\]]+)]/g).map(str => str.slice(1, -1));
        if (ticker === 'NSE:NIFTY50-INDEX' && this.cData && this.cData.chart.data.length > 0) {
          const price = parseFloat(priceStr);
          const lastCandle = this.cData.chart.data[this.cData.chart.data.length - 1];
          const now = new Date();

          // Update the last candle's high, low, close prices
          lastCandle[2] = Math.max(lastCandle[2], price); // High
          lastCandle[3] = Math.min(lastCandle[3], price); // Low
          lastCandle[4] = price; // Close

          // Check if a new candle should be created
          const intervalMs = this.getIntervalMs(this.form.timeframe);
          const nowMs = now.getTime();
          if (this.lastCandleCreationTime === null || nowMs - this.lastCandleCreationTime >= intervalMs) {
            this.cData.chart.data.push([nowMs, price, price, price, price, 0]); // Open, High, Low, Close, Volume
            this.lastCandleCreationTime = nowMs;
          }

          this.cData = { ...this.cData }; // Trigger reactivity
        }
        console.log('WebSocket message:', message.data);
      };
      this.ws.onerror = (error) => {
        console.error('WebSocket error:', error);
      };
      this.ws.onclose = () => {
        console.log('WebSocket connection closed');
      };
    },
    getIntervalMs(timeframe) {
      const timeUnits = {
        'minute': 60 * 1000,
        'hour': 60 * 60 * 1000,
        'day': 24 * 60 * 60 * 1000,
        'week': 7 * 24 * 60 * 60 * 1000,
      };

      const match = timeframe.match(/^(\d+)([a-zA-Z]+)$/);
      if (match) {
        const value = parseInt(match[1], 10);
        const unit = match[2].toLowerCase();
        return value * (timeUnits[unit] || 0);
      }
      return 5 * 60 * 1000; // Default to 5 minutes if parsing fails
    },
    setupPeriodicRefresh() {
      setInterval(this.fetchData, 300000); // 5 minutes = 300000 milliseconds
    },
    async fetchData() {
      const formatDateTime = (datetime) => {
        const date = new Date(datetime);
        const yyyy = date.getFullYear();
        const mm = String(date.getMonth() + 1).padStart(2, '0');
        const dd = String(date.getDate()).padStart(2, '0');
        const hh = String(date.getHours()).padStart(2, '0');
        const min = String(date.getMinutes()).padStart(2, '0');
        const ss = String(date.getSeconds()).padStart(2, '0');
        return `${yyyy}-${mm}-${dd} ${hh}:${min}:${ss}`;
      };

      try {
        const response = await fetch('http://localhost:9994/app/no-auth/candles', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            Ticker: this.form.ticker,
            TimeFrame: this.form.timeframe,
            from: formatDateTime(this.form.from),
            to: formatDateTime(this.form.to),
          }),
        });

        const result = await response.json();
        if (result.status === 'ok') {
          this.cData = {
            chart: {
              type: "Candles",
              tf: this.form.timeframe,
              data: result.data.map(item => [
                new Date(item.Timestamp).getTime(),
                item.Open,
                item.High,
                item.Low,
                item.Close,
                0 // Volume, if available
              ]),
            },
            onchart: [],
            offchart: [],
          };
          this.lastCandleCreationTime = this.cData.chart.data.length > 0
            ? new Date(this.cData.chart.data[this.cData.chart.data.length - 1][0]).getTime()
            : null;
        } else {
          alert(result.message);
        }
      } catch (error) {
        alert('Failed to fetch data');
        console.error(error);
      }
    }
  },
  beforeDestroy() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  },
};
</script>
