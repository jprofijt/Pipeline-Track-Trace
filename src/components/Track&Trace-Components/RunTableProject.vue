<template>
<b-row>
        <b-col class="text-center text-truncate">{{project}}</b-col>
        <b-col class="text-center"><status-icon :status="status" /></b-col>
        <b-col class="text-center"><project-timer :startTime="startTime" :started="started" :finishTime="finishTime"></project-timer></b-col>
        <b-col class="d-flex align-items-center justify-content-center" >
            <progress-bar class="w-100 mt-1" :variant="variant" :step="steps" :totalSteps="totalSteps" :noWarning="noWarning" :error="false" :animated="true"></progress-bar>
        </b-col>
</b-row>
</template>

<script>
import progressBar from './ProgressBar.vue'
import ProjectTimer from './ProjectTimer.vue'
import StatusIcon from './StatusIcon.vue'

export default {
    name: 'project',
    props: {
        header: {
            type: Boolean,
            required: false,
            default: false
        },

        project: {
            type: String,
            required: true
        },

        jobs: {
            type: Array,
            required: true
        },

        runID: {
            type: String,
            required: true
        },

        projectCount: {
            type: Number,
            required: true
        },

        time: {
            type: Number,
            required: true
        },

        resultCopy: {
            type: String | undefined,
            required: true
        }
    },
    components: {
        progressBar,
        ProjectTimer,
        StatusIcon
    },
    data () {
        return {
            variant: 'warning',
            averageHours: 4,
            noWarning: true
            
        }
    },
    computed: {
        /**
         * Converts average hours to milliseconds for timer
         * @returns Number
         */
        averageHoursInMs: function () {
            return this.averageHours * 3600 * 1000
        },
        /**
         * Filters jobs that are not completed sorted by start date
         * @returns Array
         */
        remainingJobs: function () {
            return this.jobs.filter(function (x) {return x.status !== 'finished'}).sort((a, b) => {
                const Astarted = a.started_date
                const Bstarted = b.started_date

                if (Astarted === '' && Bstarted === '') {
                    return 0
                } else if (Astarted !== '' && Bstarted === '') {
                    return 1
                } else if (Bstarted !== '' && Astarted === '') {
                    return -1
                }

                const DateA = new Date(Astarted).getTime()
                const DateB = new Date(Bstarted).getTime()

                if (DateA > DateB) {
                    return 1
                } else if (DateA < DateB) {
                    return -1
                } else return 0
                
            })
        },
        /**
         * calculates finished step count
         * @returns Number
         */
        steps: function () {
            return this.jobs.filter(function (x) {return x.status === 'finished'}).length
        },
        /**
         * Calculates total step count for completion
         * @returns Number
         */
        totalSteps: function () {
            return this.jobs.length
        },
        /**
         * Finds status of project and sets variant
         * @returns String
         */
        status: function () {
            if (!this.noWarning) {
                this.variant = 'warning'
                return 'warning'
        } else if (this.finished) {
            this.variant = 'success'
            return 'finished'
        }  else if (this.remainingJobs.filter(function (x) {return x.status === 'error'}).length >= 1) {
            this.variant = 'danger'
            return 'error'
        } else if (this.remainingJobs.filter(function (x) {return x.status === 'started'}).length >= 1) {
            this.variant = 'primary'
            return 'running'
        } else {
            this.variant = 'secondary'
            return 'waiting'
        }
        },
        /**
         * Checks if project has been started
         * @returns Boolean
         */
        started: function() {
            const startedJobs = this.jobs.filter(function (x) {return x.status === 'started' }).length
            
            if (this.steps > 0) {
                return true
            } else if (startedJobs > 0) {
                return true
            }
            return false
        },
        /**
         * Checks if all steps are completed
         * @returns Boolean
         */
        finished: function () {
            return this.steps/this.totalSteps === 1 || this.resultCopy === 'finished'
        },
        /**
         * Sums up all job runtimes
         * @returns total runtime
         */
        runtime: function() {
            let sortedJobs = this.jobs.sort(function (x) {return x.job})
            let runtime = 0
            sortedJobs.forEach((job) => {
                if (job.started_date && job.finished_date) {
                    runtime += (new Date(job.finished_date) - new Date(job.started_date))
                }
            })
            return runtime
        },
        /**
         * Gets finished time, if not finished returns now()
         * @returns Number (milliseconds)
         */
        finishTime: function() {
            let finished_date = 0
            if (this.finished) {
                return this.findLastDateTime(this.jobs)
            } else {
                return this.time
            }
        },
        /**
         * Gets started time
         * @returns Number (milliseconds)
         */
        startTime: function() {
            return this.findStartDateTime(this.jobs)
        }

    },
    methods: {
        /**
         * emits finished
         */
        projectFinished() {
            this.$emit('finished', this.runtime)
        },
        /**
         * checks if expected time has been breached
         */
        hoursNegative() {
            if (!this.finished) {
                this.variant = 'warning'
                this.noWarning = false
            }
        },
        findLastDateTime(jobs) {
            let finished_date = 0
            jobs.forEach((job) => {
                    let currentjob = new Date(job.finished_date).getTime()
                    if (finished_date < currentjob && !isNaN(finished_date)) {
                        finished_date = currentjob
                    }
            })

            return finished_date
        },
        findStartDateTime(jobs) {
            let started_date = Infinity
            jobs.forEach((job) => {
                let currentjob = new Date(job.started_date).getTime()
                if (started_date > currentjob && !isNaN(started_date)) {
                    started_date = currentjob
                }
            })
            return started_date
        }
    }
}

</script>

<style scoped>

</style>