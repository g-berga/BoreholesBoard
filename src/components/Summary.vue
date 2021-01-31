<template>
  <div>
    <b-card bg-variant='dark' style='text-align:center;'>
      <b-card-title class='cardTitle'>Boreholes</b-card-title>
      <b-card-sub-title><div style='font-size:18px;color:#f8f9fa;'>{{ n_records }}</div></b-card-sub-title>
    </b-card>
    <b-card-group>
      <b-card bg-variant='dark' style='text-align:center;'>
        <b-card-title class='cardTitle'>Min</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ min }} °C</div></b-card-sub-title>
      </b-card>
      <b-card bg-variant='dark' style='text-align:center;'>
        <b-card-title class='cardTitle'>Max</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ max }} °C</div></b-card-sub-title>
      </b-card>
      <b-card bg-variant='dark' style='text-align:center;'>
        <b-card-title class='cardTitle'>Mean</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ mean }} °C</div></b-card-sub-title>
      </b-card>
      <b-card bg-variant='dark' style='text-align:center;'>
        <b-card-title class='cardTitle'>Count</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ n_measurements }}</div></b-card-sub-title>
      </b-card>
    </b-card-group>
  </div>
</template>

<script>
export default {
  name: 'Summary',
  props: {
    info: null
  },
  data () {
    return {
      n_records: null,
      n_measurements: null,
      min: null,
      max: null,
      mean: null
    }
  },
  watch: {
    info (data) {
      this.n_records = data.n_records
      this.n_measurements = data.n_measurements
      this.min = data.min
      this.max = data.max
      this.mean = data.mean
    }
  }
}
</script>

<style>
  .cardTitle {
    font-size:14px;
    color:#ced4da;
  }
  .cardSub {
    font-size:14px;
    color:#f8f9fa;
  }
</style>
