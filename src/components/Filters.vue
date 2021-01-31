<template>
  <div>
    <b-card bg-variant='dark' style='height:350px;color:white;'>
      <b-card-title style='font-size:16px;color:#ced4da;'>Filters</b-card-title>
      <b-form-group style='margin-top:20px;'>
        <b-input-group size='sm' id="depthFilter" prepend='Depth'>
          <b-form-input id="iMinDepth" v-model="fmindepth" :number="true" :formatter="formatDepth" placeholder="Min" type="number" size="sm" min="0" max="9999"></b-form-input>
          <b-form-input id="iMaxDepth" v-model="fmaxdepth" :number="true" :formatter="formatDepth" placeholder="Max" type="number" size="sm" min="0" max="9999"></b-form-input>
        </b-input-group>
      <div class='selected'>Selected: [{{ mindepth }}, {{ maxdepth }}]</div>
      </b-form-group>
      <b-form-group>
        <b-input-group size="sm" prepend="Temperature" id="tempFilter">
          <b-form-input id="iMinTemp" v-model="fmintemp" :number="true" :formatter="formatTemp" placeholder="Min" type="number" size="sm" min="0" max="999"></b-form-input>
          <b-form-input id="iMaxTemp" v-model="fmaxtemp" :number="true" :formatter="formatTemp" placeholder="Max" type="number" size="sm" min="0" max="999"></b-form-input>
        </b-input-group>
      <div class='selected'>Selected: [{{ mintemp }}, {{ maxtemp }}]</div>
      </b-form-group>
      <b-form-group>
        <b-form-checkbox id="checkboxLitho" v-model="flitho" name="checkboxLitho" value="Required" unchecked-value="Not required">
          <div style='font-size:14px'>Thickness</div>
        </b-form-checkbox>
        <div class='selected'>{{ litho }}</div>
      </b-form-group>
      <b-form-group>
        <b-button-group>
          <b-button variant='secondary' size='sm' v-on:click="apply()">Apply</b-button>
          <b-button variant='secondary' size='sm' v-on:click="cancel()">Cancel</b-button>
          <b-button variant='secondary' size='sm' v-on:click="reset()">Reset</b-button>
        </b-button-group>
      </b-form-group>
    </b-card>
  </div>
</template>

<script>
export default {
  name: 'Filters',
  data () {
    return {
      fmintemp: 0,
      fmaxtemp: 999,
      fmindepth: 0,
      fmaxdepth: 9999,
      mintemp: 0,
      maxtemp: 999,
      mindepth: 0,
      maxdepth: 9999,
      flitho: 'Not required',
      litho: 'Not required'
    }
  },
  methods: {
    apply () {
      this.mintemp = this.fmintemp
      this.maxtemp = this.fmaxtemp
      this.mindepth = this.fmindepth
      this.maxdepth = this.fmaxdepth
      this.litho = this.flitho
      const f = {
        mintemp: this.mintemp,
        maxtemp: this.maxtemp,
        mindepth: this.mindepth,
        maxdepth: this.maxdepth,
        litho: this.litho
      }
      this.$emit('applyFilters', f)
    },
    cancel () {
      this.fmintemp = 0
      this.fmaxtemp = 999
      this.fmindepth = 0
      this.fmaxdepth = 9999
      this.flitho = 'Not required'
      document.getElementById('checkboxLitho').state = null
    },
    reset () {
      this.mintemp = 0
      this.maxtemp = 999
      this.mindepth = 0
      this.maxdepth = 9999
      this.litho = 'Not required'
      this.$emit('resetFilters')
    },
    formatDepth (val) {
      return Number(String(val).substring(0, 4))
    },
    formatTemp (val) {
      return Number(String(val).substring(0, 3))
    }
  },
  watch: {
    fmindepth: function (val) {
      if (val < 0) {
        this.fmindepth = 0
        val = 0
      }
      if (val > this.fmaxdepth) {
        this.fmaxdepth = val
      }
    },
    fmaxdepth: function (val) {
      if (val < 0) {
        val = 0
        this.fmaxdepth = 0
      }
      if (val < this.fmindepth) {
        this.fmindepth = val
      }
    },
    fmintemp: function (val) {
      if (val < 0) {
        this.fmintemp = 0
        val = 0
      }
      if (val > this.fmaxtemp) {
        this.fmaxtemp = val
      }
    },
    fmaxtemp: function (val) {
      if (val < 0) {
        val = 0
        this.fmaxtemp = 0
      }
      if (val < this.fmintemp) {
        this.fmintemp = val
      }
    }
  }
}
</script>

<style>
  .selected {
    font-size:14px;
    color:#f8f9fa;
  }
</style>
