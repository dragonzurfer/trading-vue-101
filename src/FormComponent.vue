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
  mounted() {
    const now = new Date();
    const oneWeekAgo = new Date();
    oneWeekAgo.setDate(now.getDate() - 7);

    this.form.ticker = 'MCX:CRUDEOIL24JULFUT';
    this.form.timeframe = '5minute';
    this.form.from = oneWeekAgo.toISOString().slice(0, 16);
    this.form.to = now.toISOString().slice(0, 16);
  },
};
</script>

<style scoped>
form {
  margin-bottom: 20px;
}
</style>
