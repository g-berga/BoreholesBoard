<template>
  <vue-plotly :data='data' :layout='layout' :options='options' id='tempChart'/>
</template>

<script>
import VuePlotly from '@statnett/vue-plotly'

const d3 = require('d3')

const scaleColor = d3.scaleOrdinal()
  .domain(['0', '1', '2', '3', '4', '5', '6', '7', '8'])
  .range(['#8dc44f', '#1ca45c', '#ffc718', '#ff9e0f', '#da483b', '#4486f4', '#dcf8c6', '#00C6FF', '#6F1BC6'])

export default {
  name: 'TempChart',
  components: {
    VuePlotly
  },
  props: {
    well: null
  },
  data: function () {
    return {
      data: [{
        type: 'scatter',
        x: [],
        y: [],
        name: [],
        mode: 'markers',
        marker: {
          color: '',
          size: 10
        }
      }],
      layout: {
        height: 310,
        margin: { t: 20, b: 35, l: 80, r: 30 },
        plot_bgcolor: '#343a40',
        paper_bgcolor: '#343a40',
        xaxis: {
          title: 'Temperature (°C)',
          color: '#f8f9fa'
        },
        yaxis: {
          title: 'Depth (m)',
          color: '#f8f9fa'
        },
        legend: {
          orientation: 'v',
          font: { color: '#f8f9fa' },
          width: 30
        },
        // title: 'Temperature vs Depth by Method',
        hovermode: 'closest',
        showlegend: true
      },
      options: {
        responsive: true,
        scrollZoom: true,
        displayModeBar: false
      }
    }
  },
  methods: {
    resetScatter () {
      this.data = [{
        type: 'scatter',
        x: [],
        y: [],
        mode: 'markers',
        marker: {
          color: '',
          size: 10
        },
        text: [],
        hovertemplate: []
      }]
    },
    createScatterTrace (Xs, Ys, color, name, texts) {
      var scatter = {
        type: 'scatter',
        x: Xs,
        y: Ys,
        name: 'Method ' + name,
        legendgroup: name,
        marker: {
          color: color,
          size: 10
        },
        text: texts,
        hovertemplate: 'Temperature: %{x} °C<br>' +
            'Depth: %{y} m<br>' +
            'Date: %{text}' +
            '<extra></extra>',
        mode: 'markers'
      }
      return scatter
    },
    refreshScatter (datum) {
      this.data = []
      var traces = {}
      var minTemp = 999
      var maxTemp = -1
      var minDepth = 9999
      var maxDepth = -1
      for (var i = 0; i < datum.depths.length; i++) {
        var method = datum.methods[i]
        var depth = Math.round(datum.depths[i])
        var temp = Math.round(datum.temperatures[i])
        var text = datum.dates[i].toString()
        if (minTemp > temp) {
          minTemp = temp
        }
        if (maxTemp < temp) {
          maxTemp = temp
        }
        if (minDepth > depth) {
          minDepth = depth
        }
        if (maxDepth < depth) {
          maxDepth = depth
        }
        this.layout.xaxis.range = [Math.max(0, minTemp - minTemp * 0.2), maxTemp + maxTemp * 0.2]
        this.layout.yaxis.range = [Math.max(0, minDepth - minDepth * 0.2), maxDepth + maxDepth * 0.2]
        if (!(method in traces)) {
          traces[method] = {
            depths: [],
            temperatures: [],
            texts: [],
            color: scaleColor(method)
          }
        }
        traces[method].depths.push(depth)
        traces[method].temperatures.push(temp)
        traces[method].texts.push(text)
      }
      for (const [key, value] of Object.entries(traces)) {
        this.data.push(this.createScatterTrace(value.temperatures, value.depths, value.color, key, value.texts))
      }
    }
  },
  watch: {
    well (datum) {
      if (datum === null) {
        this.resetScatter()
      } else {
        this.refreshScatter(datum)
      }
    }
  }
}
</script>
