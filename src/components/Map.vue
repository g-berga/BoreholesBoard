<template>
  <vue-plotly :data='data' :layout='layout' :options='options' id='map'/>
</template>
<script>

import VuePlotly from '@statnett/vue-plotly'
import Plotly from 'plotly.js'

let plotlyMap
var selWell = {
  prevIndex: null,
  prevSize: null,
  prevOpacity: null
}

const d3 = require('d3')
const scaleOpacity = d3.scaleLinear().domain([0, 500]).range([0.6, 1])
const scaleSize = d3.scaleLinear().domain([0, 500]).range([6, 12])

export default {
  name: 'Map',
  components: {
    VuePlotly
  },
  props: {
    wells: NaN
  },
  data: function () {
    return {
      data: [{
        type: 'scattermapbox',
        lon: [],
        lat: [],
        text: [],
        mode: 'markers',
        hoverinfo: 'text',
        marker: {
          color: [],
          size: [],
          opacity: [],
          scale: 'YlOrRd',
          showscale: true,
          cmin: 0,
          cmax: 500,
          colorbar: {
            bgcolor: '#343a40',
            thickness: 15,
            side: 'bottom',
            tickcolor: '#f8f9fa',
            tickfont: {
              color: '#f8f9fa'
            }
          }
        }
      }],
      layout: {
        mapbox: { style: 'satellite', zoom: 3, center: { lon: 13, lat: 43 } },
        margin: { t: 0, b: 0, r: 0, l: 0 },
        // width: 560,
        height: 310,
        width: 460,
        hovermode: 'closest',
        paper_bgcolor: '#343a40'
      },
      options: {
        mapboxAccessToken: 'pk.eyJ1IjoiZy1iZXJnYW50aW5vIiwiYSI6ImNrajh5eDh2dTF0dXQyem5xMm94NzV5OWQifQ.jI_yN7vp8VflxrA0Mp5ZsQ',
        responsive: true,
        displayModeBar: false
      }
    }
  },
  mounted () {
    plotlyMap = document.getElementById('map')

    plotlyMap.on('plotly_click', d => {
      const index = d.points[0].pointIndex
      const prevIdx = selWell.prevIndex
      if (prevIdx !== null) {
        this.data[0].marker.opacity[prevIdx] = selWell.prevOpacity
        this.data[0].marker.size[prevIdx] = selWell.prevSize
      }
      selWell.prevIndex = index
      selWell.prevOpacity = this.data[0].marker.opacity[index]
      selWell.prevSize = this.data[0].marker.size[index]

      var colors = this.data[0].marker.color
      var opacities = this.data[0].marker.opacity
      var sizes = this.data[0].marker.size

      // colors[index] = 'green'
      sizes[index] = 24
      opacities[index] = 1

      var update = { marker: { color: colors, size: sizes, opacity: opacities, scale: 'YlOrRd', showscale: true, cmin: 0, cmax: 500, colorbar: { bgcolor: '343a40', side: 'right', thickness: 15, tickcolor: '#f8f9fa', tickfont: { color: '#f8f9fa' } } } }

      Plotly.restyle('map', update)

      this.$emit('clicked', index)
      // this.$emit('clicked_pt', this.data[0])
    })
  },
  watch: {
    wells (datum) {
      if (datum === null) {
        this.data[0].lon = []
        this.data[0].lat = []
        this.data[0].text = []
        this.data[0].marker.color = []
        this.data[0].marker.size = []
        this.data[0].marker.opacity = []
      } else {
        this.data[0].lon = datum.map(d => d.lon)
        this.data[0].lat = datum.map(d => d.lat)
        this.data[0].text = datum.map(d => '<b>' + d.name + '</b><br>' +
          'Temperature (°C): ' + d.texttemp + '<br>' +
          'Coordinates: (' + d.lat + ', ' + d.lon + ')<br>' +
          'Depth (m): ' + d.depth + ', ' + '<br>' +
          'Altitude (m): ' + d.altitude + '<br>' +
          'Region: ' + d.region + '<br>' +
          'Province: ' + d.province + '<br>')
        this.data[0].marker.color = datum.map(d => d.temp)
        this.data[0].marker.size = datum.map(d => scaleSize(d.temp))
        this.data[0].marker.opacity = datum.map(d => scaleOpacity(d.temp))
        selWell.index = null
        selWell.opacity = null
        selWell.size = null
      }
    }
  }
}
</script>
