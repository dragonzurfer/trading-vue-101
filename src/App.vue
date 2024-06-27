<template>
  <!-- TradingVueJs 101 (example from 'Getting Started' ) -->
  <trading-vue :data="chart" :width="this.width" :height="this.height"
      :color-back="colors.colorBack"
      :color-grid="colors.colorGrid"
      :color-text="colors.colorText">
  </trading-vue>
</template>

<script>
import TradingVue from 'trading-vue-js'
import axios from 'axios'
import { SERVER_URL } from './config'

export default {
    name: 'app',
    components: { TradingVue },
    methods: {
        onResize(event) {
            this.width = window.innerWidth
            this.height = window.innerHeight
        },
        async fetchCandleData() {
            try {
                const response = await axios.post(`${SERVER_URL}/app/no-auth/candles`, 
                {
                    "Ticker": "MCX:CRUDEOIL24JULFUT",
                    "TimeFrame": "5minute",
                    "from": "2024-06-20 00:00:00",
                    "to": "2024-06-27 23:59:00"
                },
                {
                    headers: {
                        'Content-Type': 'application/json'
                    }
                })
                if (response.data.status === 'ok') {
                    console.log('Raw data:', response.data.data);
                    this.chart = this.formatCandleData(response.data.data)
                    console.log('Formatted data:', this.chart);
                } else {
                    console.error('Failed to fetch data:', response.data.message)
                }
            } catch (error) {
                console.error('Error fetching data:', error)
            }
        },
        formatCandleData(data) {
            return {
                ohlcv: data.map(candle => [
                    new Date(candle.Timestamp).getTime() / 1000,
                    candle.Open,
                    candle.High,
                    candle.Low,
                    candle.Close,
                    candle.Volume || 0 // Ensure there is a volume field
                ])
            }
        }
    },
    mounted() {
        window.addEventListener('resize', this.onResize)
        this.fetchCandleData()
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.onResize)
    },
    data() {
        return {
            chart: {
                ohlcv: [] // Initialize with an empty array
            },
            width: window.innerWidth,
            height: window.innerHeight,
            colors: {
                colorBack: '#fff',
                colorGrid: '#eee',
                colorText: '#333',
            }
        }
    }
}
</script>

<style>
/* Add your styles here */
</style>
