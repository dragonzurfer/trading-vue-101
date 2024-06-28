<template>
  <form @submit.prevent="$emit('fetch-data')">
    <label for="ticker">Ticker:</label>
    <input type="text" id="ticker" v-model="form.ticker" required />

    <label for="timeframe">Time Frame:</label>
    <input type="text" id="timeframe" v-model="form.timeframe" required />

    <label for="from">From:</label>
    <input type="datetime-local" id="from" v-model="form.from" required />

    <label for="to">To:</label>
    <input type="datetime-local" id="to" v-model="form.to" required />

    <button type="submit">Fetch Data</button>
  </form>
</template>

<script>
export default {
  props: ["form"],
};
</script>

<style scoped>
form {
  margin-bottom: 20px;
}
</style>
