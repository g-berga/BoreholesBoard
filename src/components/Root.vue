<template>
  <b-container fluid class='container'>
    <b-row class='mapRow' style='height:350px;'>
      <b-col cols='3'>
        <Filters @applyFilters="apply" @resetFilters="reset"/>
      </b-col>
      <b-col cols='5' style='background-color:#343a40;height:350px;'>
        <Map :wells='mapWells' v-on:clicked='refreshCharts'/>
        <b-form-group style='color:#f8f9fa;margin-top:1%;font-size:14px;text-align:right;'>
          Displayed Temperature: {{ selType }}
          <b-button-group>
            <b-button variant='secondary' size='sm' v-on:click='colorByMin'>Min</b-button>
            <b-button variant='secondary' size='sm' v-on:click='colorByMax'>Max</b-button>
            <b-button variant='secondary' size='sm' v-on:click='colorByMean'>Mean</b-button>
          </b-button-group>
        </b-form-group>
      </b-col>
      <b-col cols='4' style='height:350px;'>
        <Summary :info='summaryInfo'/>
        <WellSummary :info='wellSummaryInfo' style='margin-top:0px;' />
      </b-col>
    </b-row>
    <b-row class='chartsrow'>
      <b-col cols='6'>
        <LithoChart :well='lithoWell' class='chart'/>
      </b-col>
      <b-col cols='6'>
        <TempChart :well='tempWell' class='chart'/>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import Filters from '@/components/Filters.vue'
import Summary from '@/components/Summary.vue'
import Map from '@/components/Map.vue'
import WellSummary from '@/components/WellSummary.vue'
import TempChart from '@/components/TempChart.vue'
import LithoChart from '@/components/LithoChart.vue'

const d3 = require('d3')

export default {
  name: 'MapWithCharts',
  components: {
    Filters,
    Summary,
    Map,
    WellSummary,
    TempChart,
    LithoChart
  },
  props: {
    summaryInfo: null,
    wellSummaryInfo: null,
    lithoWell: null,
    tempWell: null
  },
  data () {
    return {
      currentWells: [],
      selType: 'Max',
      mapWells: []
    }
  },
  mounted () {
    fetch('/static/wells.json')
      .then(res => res.json())
      .then((data) => {
        this.currentWells = data.wells.map(d => d)
        this.refreshMap()
        this.refreshSummary(this.currentWells)
        this.refreshCharts(null)
      })
  },
  methods: {
    // it applies filters to data
    apply: function (filters) {
      fetch('/static/wells.json')
        .then(res => res.json())
        .then((data) => {
          this.currentWells = data.wells.map(d => d)
          this.refreshCurrentWells(filters)
          this.refreshSummary(this.currentWells)
          this.refreshMap()
          this.refreshCharts(null)
        })
    },
    // it resets currentwells and the components
    reset () {
      fetch('/static/wells.json')
        .then(res => res.json())
        .then((data) => {
          this.currentWells = data.wells.map(d => d)
          this.refreshSummary(this.currentWells)
          this.refreshMap()
          this.refreshCharts(null)
        })
    },
    // it refreshes the current set of wells displayed in the map and calculates
    // min, mean, max temperature and count of temperature measurements satisfying Filters input parameters
    refreshCurrentWells (filters) {
      var res = []
      let well
      if (filters.litho === 'Required') {
        for (var i = 0; i < this.currentWells.length; i++) {
          well = this.currentWells[i]
          if (well.compositions.length !== 0 && well.n_measurements !== 0) {
            var count = 0
            var sum = 0
            var mintemp = 999
            var maxtemp = -1
            for (var j = 0; j < well.n_measurements; j++) {
              const t = well.temperatures[j]
              const d = well.tempDepths[j]
              if ((filters.mintemp <= t && t <= filters.maxtemp) && (filters.mindepth <= d && d <= filters.maxdepth)) {
                count = count + 1
                sum = sum + t
                if (t < mintemp) {
                  mintemp = t
                }
                if (t > maxtemp) {
                  maxtemp = t
                }
              }
            }
            if (count !== 0) {
              well.meanTemperature = Math.round(sum / count)
              well.minTemperature = Math.round(mintemp)
              well.maxTemperature = Math.round(maxtemp)
              well.n_measurements = count
              res.push(well)
            }
          }
        }
      } else {
        for (var k = 0; k < this.currentWells.length; k++) {
          well = this.currentWells[k]
          if (well.n_measurements !== 0) {
            count = 0
            sum = 0
            mintemp = 1000
            maxtemp = -1
            for (var l = 0; l < well.n_measurements; l++) {
              const t = well.temperatures[l]
              const d = well.tempDepths[l]
              if ((filters.mintemp <= t && t <= filters.maxtemp) && (filters.mindepth <= d && d <= filters.maxdepth)) {
                count = count + 1
                sum = sum + t
                if (t < mintemp) {
                  mintemp = t
                }
                if (t > maxtemp) {
                  maxtemp = t
                }
              }
            }
            if (count !== 0) {
              well.meanTemperature = Math.round(sum / count)
              well.minTemperature = Math.round(mintemp)
              well.maxTemperature = Math.round(maxtemp)
              well.n_measurements = count
              res.push(well)
            }
          }
        }
      }
      this.currentWells = res
    },
    // it refreshes the map and wells style according to the selected metric
    refreshMap () {
      if (this.currentWells.length === 0) {
        this.mapWells = null
      } else {
        if (this.selType === 'Max') {
          this.currentWells.sort((a, b) => a.maxTemperature - b.maxTemperature)
          this.mapWells = this.currentWells.map((d) => {
            var wells = {
              lon: d.lon_wgs84,
              lat: d.lat_wgs84,
              key: d.key,
              name: d.name,
              region: d.region,
              province: d.province,
              altitude: d.altitude,
              depth: d.depth,
              temp: d.maxTemperature
            }
            if (d.maxTemperature === -1) {
              wells.texttemp = 'None'
            } else {
              wells.texttemp = d.maxTemperature
            }
            return wells
          })
        } else {
          if (this.selType === 'Mean') {
            this.currentWells.sort((a, b) => a.meanTemperature - b.meanTemperature)
            this.mapWells = this.currentWells.map((d) => {
              var wells = {
                lon: d.lon_wgs84,
                lat: d.lat_wgs84,
                key: d.key,
                name: d.name,
                region: d.region,
                province: d.province,
                altitude: d.altitude,
                depth: d.depth,
                temp: d.meanTemperature
              }
              if (d.meanTemperature === -1) {
                wells.texttemp = 'None'
              } else {
                wells.texttemp = d.meanTemperature
              }
              return wells
            })
          } else {
            this.currentWells.sort((a, b) => a.minTemperature - b.minTemperature)
            this.mapWells = this.currentWells.map((d) => {
              var wells = {
                lon: d.lon_wgs84,
                lat: d.lat_wgs84,
                key: d.key,
                name: d.name,
                region: d.region,
                province: d.province,
                altitude: d.altitude,
                depth: d.depth,
                temp: d.minTemperature
              }
              if (d.minTemperature === -1) {
                wells.texttemp = 'None'
              } else {
                wells.texttemp = d.minTemperature
              }
              return wells
            })
          }
        }
      }
    },
    // The next three methosds restyle by min, max or mean temperature boreholes on the map
    // (it makes refreshes also components related to the selected well)
    colorByMean () {
      if (this.selType !== 'Mean') {
        this.selType = 'Mean'
        this.refreshMap()
        this.refreshCharts(null)
        this.refreshWellSummary(null)
      }
    },
    colorByMin () {
      if (this.selType !== 'Min') {
        this.selType = 'Min'
        this.refreshMap()
        this.refreshCharts(null)
        this.refreshWellSummary(null)
      }
    },
    colorByMax () {
      if (this.selType !== 'Max') {
        this.selType = 'Max'
        this.refreshMap()
        this.refreshCharts(null)
        this.refreshWellSummary(null)
      }
    },
    // it refreshes charts and WellSummary components with the new well object retrieved on currentWells array at its input index
    refreshCharts (index) {
      if (index === null) {
        this.refreshWellSummary(null)
        this.lithoWell = null
        this.tempWell = null
      } else {
        var well = this.currentWells[index]
        if (well.temperatures.length < 1) {
          this.refreshWellSummary(null)
        } else {
          this.refreshWellSummary(well)
          this.tempWell = {
            temperatures: well.temperatures,
            depths: well.tempDepths,
            methods: well.methods,
            dates: well.dates
          }
        }
        if (well.compositions.length > 0) {
          this.lithoWell = {
            startDepths: well.startDepths,
            endDepths: well.endDepths,
            compositions: well.compositions
          }
        } else {
          this.lithoWell = null
        }
      }
    },
    // it refreshes Summary component calculating the updated metrics on the new input data
    refreshSummary (data) {
      var wcount = data.length
      if (wcount === 0) {
        this.summaryInfo = {
          min: 'Na',
          max: 'Na',
          mean: 'Na',
          n_measurements: 0,
          n_records: 0
        }
      } else {
        var min = 999
        var max = 0
        var sum = 0
        var tcount = 0
        for (var i = 0; i < wcount; i++) {
          if (data[i].n_measurements !== 0) {
            sum = sum + data[i].meanTemperature * data[i].n_measurements
            tcount = tcount + data[i].n_measurements
            if (data[i].minTemperature < min) {
              min = data[i].minTemperature
            }
            if (data[i].maxTemperature > max) {
              max = data[i].maxTemperature
            }
          }
        }
        this.summaryInfo = {
          min: min,
          max: max,
          mean: Math.round(sum / tcount),
          n_measurements: tcount,
          n_records: wcount
        }
      }
    },
    // it refreshes WellSummary by recalculating the new metrics over well temperature measurements
    // both on filtered and unfiltered measurments of the well
    refreshWellSummary (well) {
      if (well === null) {
        this.wellSummaryInfo = {
          n_measurements: 'Na',
          min: 'Na',
          max: 'Na',
          mean: 'Na',
          gmin: 'Na',
          gmax: 'Na',
          gmean: 'Na',
          gn_measurements: 'Na',
          name: 'Na'
        }
      } else {
        this.wellSummaryInfo = {
          n_measurements: well.n_measurements,
          min: well.minTemperature,
          max: well.maxTemperature,
          mean: well.meanTemperature,
          gmin: Math.round(d3.min(well.temperatures)),
          gmax: Math.round(d3.max(well.temperatures)),
          gmean: Math.round(d3.mean(well.temperatures)),
          gn_measurements: well.temperatures.length,
          name: well.name
        }
      }
    }
  }
}
</script>

<style>
  .mapRow {
    margin-top: 2%;
  }
  .chartsrow {
    margin-top: 1.2%;
    margin-bottom: 10%;
  }
  .scalebutton {
    margin-top: 5%;
    text-align: right;
  }
</style>
