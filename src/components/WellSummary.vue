<template>
  <div>
    <b-card bg-variant='dark' style='height:81px;'>
      <b-card-title class='cardTitle'>Name</b-card-title>
      <b-card-sub-title><div style='font-size:16px;color:#f8f9fa;'>{{ name }}</div></b-card-sub-title>
    </b-card>
    <b-card-group>
      <b-card bg-variant='dark'>
        <b-card-title class='cardTitle'>Min</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ min }} °C</div></b-card-sub-title>
        <div class='cardSub'>{{ gmin }}*</div>
      </b-card>
      <b-card bg-variant='dark'>
        <b-card-title class='cardTitle'>Max</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ max }} °C</div></b-card-sub-title>
        <div class='cardSub'>{{ gmax }}*</div>
      </b-card>
      <b-card bg-variant='dark'>
        <b-card-title class='cardTitle'>Mean</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ mean }} °C</div></b-card-sub-title>
        <div class='cardSub'>{{ gmean }}*</div>
      </b-card>
      <b-card bg-variant='dark'>
        <b-card-title class='cardTitle'>Count</b-card-title>
        <b-card-sub-title><div class='cardSub'>{{ n_measurements }}</div></b-card-sub-title>
        <div class='cardSub'>{{ gn_measurements }}*</div>
      </b-card>
    </b-card-group>
  </div>
</template>

<script>
export default {
  name: 'WellSummary',
  props: {
    info: null
  },
  data () {
    return {
      n_measurements: 'Na',
      min: 'Na',
      max: 'Na',
      mean: 'Na',
      gn_measurements: 'Na',
      gmin: 'Na',
      gmax: 'Na',
      gmean: 'Na',
      name: 'Na'
    }
  },
  mounted () {
  },
  watch: {
    info (datum) {
      this.n_measurements = datum.n_measurements
      this.min = datum.min
      this.max = datum.max
      this.mean = datum.mean
      this.gn_measurements = datum.gn_measurements
      this.gmin = datum.gmin
      this.gmax = datum.gmax
      this.gmean = datum.gmean
      this.name = datum.name
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
