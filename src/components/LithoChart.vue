<template>
  <vue-plotly :data='data' :layout='layout' :options='options' id='chart'/>
</template>

<script>
import VuePlotly from '@statnett/vue-plotly'
const d3 = require('d3')
const scaleColor = d3.scaleLinear().domain([0, 20]).range(['orange', 'purple'])
export default {
  name: 'LithoChart',
  components: {
    VuePlotly
  },
  props: {
    well: null
  },
  data: function () {
    return {
      data: [{
        type: 'bar',
        x: [], // altezze livelli
        y: [], // range in stringa di profondità
        text: [], // composition
        // orientation: 'h',
        hoverinfo: 'text'
      }],
      layout: {
        plot_bgcolor: '#343a40',
        paper_bgcolor: '#343a40',
        margin: { t: 20, b: 90, l: 60, r: 60 },
        height: 310,
        xaxis: {
          title: 'Depth range (m)',
          color: '#f8f9fa'
        },
        yaxis: {
          title: 'Thickness (m)',
          color: '#f8f9fa'
        },
        hoverlabel: {
          font: {
            size: 12
          }
        }
      },
      options: {
        responsive: true,
        scrollZoom: false,
        displayModeBar: false
      }
    }
  },
  methods: {
    refreshBar (datum) {
      for (var i = 0; i < datum.compositions.length; i++) {
        this.data[0].y.push(Math.round(Math.abs(datum.endDepths[i] - datum.startDepths[i])))
        this.data[0].x.push('[' + Math.round(datum.startDepths[i]) + ', ' + Math.round(datum.endDepths[i]) + ']')
        this.data[0].text.push(datum.compositions[i])
        this.data[0].marker.color.push(scaleColor(i))
      }
    },
    resetBar () {
      this.data = [{
        type: 'bar',
        x: [],
        y: [],
        text: [],
        hoverinfo: 'text',
        // orientation: 'h',
        marker: {
          color: []
        }
      }]
    }
  },
  watch: {
    well (datum) {
      this.resetBar()
      if (datum !== null) {
        this.refreshBar(datum)
      }
    }
  }
}
</script>
