<template>
<b-container id="dashboard" class="h-100" fluid>
  <b-row  no-gutters class="h-50">
    <b-col class="h-100">
      <track-and-trace
        :headers="headers"
        :APIvOne="APIv1Url"
        :url="APIv2Url"
        @add-statistic="addStatistics"
        @token-expired="setToken"
        :thresholdOnco="thresholdOnco"
        :thresholdPcs="thresholdPcs"
        :thresholdExoom="thresholdExoom"
        :thresholdSvp="thresholdSvp"
        class="h-100 mt-1"/>
    </b-col>
  </b-row>
  <b-row no-gutters class="h-50">
    <b-col cols="12" lg="6" class="h-100">
      <run-time-statistics
        :run-times="runTimeArray"
        @new-threshold-onco="setOncoMax"
        @new-threshold-pcs="setPcsMax"
        @new-threshold-exoom="setExoomMax"
        @new-threshold-svp="setSvpMax"
      />
    </b-col>
    <b-col cols="12" lg="6" class="h-100">
      <sample-statistics :API="APIv2Url" :headers="headers" ></sample-statistics>
    </b-col>
  </b-row>
</b-container>
</template>

<script lang="ts">
import Vue from 'vue'
import TrackAndTrace from '@/components/TrackAndTrace.vue'
import RunTimeStatistics from '@/components/RunTimeStatistics.vue'
import SampleStatistics from '@/components/SampleStatistics.vue'
import { responseJSON, RunTimeStatistic } from '@/types/dataTypes'

declare module 'vue/types/vue' {
  interface Vue {
    username: string
    password: string
    token: string
    APIv1Url: string
    APIv2Url: string
    runTimeArray: RunTimeStatistic[]
    threshold: number
    headers: object
    getToken(username: string, password: string): Promise<string>
    setToken(): Promise<void>
    addStatistics(run: RunTimeStatistic): void
    setThreshold(threshold: number): void
  }
}

export default Vue.extend({
  name: 'app',
  components: {
    TrackAndTrace,
    RunTimeStatistics,
    SampleStatistics
  },
  data () {
    return {
      username: 'admin',
      password: 'admin',
      token: 'admin-test-token',
      APIv1Url: 'http://localhost:8081/api/v1/',
      APIv2Url: 'http://localhost:8081/api/v2/',
      runTimeArray: [],
      threshold: 20,
      thresholdOnco: 20,
      thresholdPcs: 20,
      thresholdExoom: 20,
      thresholdSvp: 20
    }
  },
  computed: {
    /**
     * Creates the api headers object
     * 
     * @returns {Object}
     */
    headers () {
      const header = {
        'x-molgenis-token': this.token,
        'Content-Type': 'application/json'
      }
      return header
    }
  },
  methods: {
    /**
     * Gets the new login token
     * @param {String} username - login name
     * @param {String} password - login password
     * 
     * @returns {String}
     */
    async getToken (username: string, password: string) {
      const response = await fetch('http://localhost:8081/api/v1/login')

      let json: responseJSON = await response.json()

      const token: string = json.token
      return token
    },

    /**
     * Sets the access token
     * 
     * @returns {void}
     */
    async setToken () {
      this.token = await this.getToken(this.username, this.password)
    },
    /**
     * Adds new runtime statistics to graph with a max lenght of 10
     * @param {RunTimeStatistic} run - new statistic to add
     * 
     * @returns {void}
     */
    addStatistics (run: RunTimeStatistic) {
      let timeArray = this.runTimeArray
      if (timeArray.length >= 10) {
        timeArray.shift()
      }
      let currentRun = run
      timeArray.push(run)

      this.runTimeArray = timeArray
    },
    /**
     * Sets maximum threshold for ONCO pipeline types
     * @param {Number} threshold - ONCO threshold hours
     * 
     * @returns {void}
     */
    setOncoMax (threshold: number): void {
      this.thresholdOnco = threshold
    },
    /**
     * Sets maximum threshold for PCS pipeline types
     * @param {Number} threshold - PCS threshold hours
     * 
     * @returns {void}
     */
    setPcsMax (threshold: number): void {
      this.thresholdPcs = threshold
    },
    /**
     * Sets maximum threshold for Exoom pipeline types
     * @param {Number} threshold - Exoom threshold hours
     * 
     * @returns {void}
     */
    setExoomMax (threshold: number):void {
      this.thresholdExoom = threshold
    },
    /**
     * Sets maximum threshold for SVP pipeline types
     * @param {Number} threshold - SVP threshold hours
     * 
     * @returns {void}
     */
    setSvpMax (threshold: number): void {
      this.thresholdSvp = threshold
    }
  }
})

</script>


<style lang="scss" scoped>

.h-45 {
height: 45%;
}
.h-55 {
  height: 55%
}
.fill {

  height: 100%;
}
</style>
