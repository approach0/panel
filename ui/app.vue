<template>
<v-app>

  <v-app-bar color="deep-purple" dark app>
    <template>
    <v-app-bar-nav-icon>
      <v-icon @click="drawer = !drawer">featured_play_list</v-icon>
    </v-app-bar-nav-icon>

    <v-toolbar-title>Approach Zero Panel</v-toolbar-title>

    <v-spacer></v-spacer>

    <v-btn href="http://ait-question-repo-ait.dev.dm-ai.cn/#/question-bank" target="_blank" color="purple darken-1" dark>
      <v-icon>storefront</v-icon> &nbsp; Main Site
    </v-btn>

    </template>
  </v-app-bar>

  <v-navigation-drawer width="600" v-model="drawer" color="purple darken-4 white--text" dark app>
    <v-container>

      <v-text-field v-model="input" label="Run job" v-on:keyup.enter="clickRun" :error-messages="input_err_msg" v-on:keyup="input_err_msg = null"
      append-icon="clear" append-outer-icon="sports" @click:append="input = ''" @click:append-outer="clickRun">
      </v-text-field>

      <v-row>
        <v-chip class="ma-2" :color="(jb == console_outsel) ? 'purple' : 'blue-grey'"
                v-for="jb in console_starjob" :key="jb" @click="clickStar(jb)">
          <v-icon>laptop_mac</v-icon> &nbsp; {{jb}}
        </v-chip>
      </v-row>

      <v-row justify="center" v-if="Object.keys(job_description).length > 0">
        <v-card v-if="job_description" class="d-inline-block" light>
          <v-card-text>
          <p class="text-h5"> {{job_description['name']}} </p>
          <p v-for="(val, key) in job_description" v-if="key != 'name'" class="text-subtitle-2 text--primary">
            {{key}}: {{val}}
          </p>
          </v-card-text>
        </v-card>
      </v-row>

      <v-card id="console" class="grey darken-4 console"
       v-bind:loading="console_loading">{{console_content}}</v-card>

      <v-row justify="space-around" class="flex-wrap">
        <v-switch v-model="console_refresh" label="Console refresh"></v-switch>
        <v-switch v-model="console_stickbt" label="Stick to bottom"></v-switch>
        <v-switch v-model="dry_run" label="Dry run"></v-switch>
        <v-switch v-model="single_job" label="Single job"></v-switch>
      </v-row>

    </v-container>
  </v-navigation-drawer>

  <v-main app>
    <v-container>
      <v-parallax dark src="https://cdn.vuetifyjs.com/images/parallax/material2.jpg">
      <v-row align="center" justify="center">

        <v-col class="text-center" cols="4">
          <v-card>
            <v-img class="white--text align-end" height="200px" :src="img_mounted"></v-img>
            <v-card-title>
              Disk Volume

              <v-spacer></v-spacer>
              <v-btn color="red" @click="changeStatus('mounted', 'mount:vdisk-remove')">
                Delete
              </v-btn>
            </v-card-title>

            <v-card-subtitle v-if="status.mounted === null">Please wait ...</v-card-subtitle>
            <v-card-actions v-else>
              <v-btn color="red" text :disabled="!status.mounted" @click="changeStatus('mounted', 'mount:vdisk-unmount')">
                {{status.mounted ? 'Unmount' : 'Unmounted'}}
              </v-btn>
              <v-btn color="green" text :disabled="status.mounted" @click="changeStatus('mounted', 'mount:vdisk-mount')">
                {{status.mounted ? 'Mounted' : 'Mount'}}
              </v-btn>
            </v-card-actions>

          </v-card>
        </v-col>

        <v-col class="text-center" cols="4">
          <v-card>
            <v-img class="white--text align-end" height="200px" :src="img_indexer"></v-img>
            <v-card-title>Indexer</v-card-title>

            <v-card-subtitle v-if="status.indexer === null">Please wait ...</v-card-subtitle>
            <v-card-actions v-else>
              <v-btn color="red" text :disabled="!status.indexer" @click="changeStatus('indexer', 'indexer:killed')">
                {{status.indexer ? 'Stop' : 'Stopped'}}
              </v-btn>
              <v-btn color="green" text :disabled="status.indexer" @click="changeStatus('indexer', 'indexer:spawn')">
                {{status.indexer ? 'Running' : 'Run'}}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>

        <v-col class="text-center" cols="4">
          <v-card>
            <v-img class="white--text align-end" height="200px" :src="img_searchd"></v-img>
            <v-card-title>Search Daemon</v-card-title>

            <v-card-subtitle v-if="status.searchd === null">Please wait ...</v-card-subtitle>
            <v-card-actions v-else>
              <v-btn color="red" text :disabled="!status.searchd" @click="changeStatus('searchd', 'searchd:killed')">
                {{status.searchd ? 'Stop' : 'Stopped'}}
              </v-btn>
              <v-btn color="green" text :disabled="status.searchd" @click="changeStatus('searchd', 'searchd:spawn')">
                {{status.searchd ? 'Running' : 'Run'}}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>

      </v-row>
  </v-parallax>

      <v-divider inset style="margin: 15px"></v-divider>

      <v-card class="mx-auto" tile>
        <v-list shaped flat>
          <v-subheader>{{tasks.length == 0 ? 'No tasks' : 'Tasks'}}</v-subheader>

          <v-list-item-group color="primary">

            <v-list-item v-for="task in tasks" :key="task.taskid">

              <v-list-item-avatar>
                <v-icon>ballot</v-icon>
              </v-list-item-avatar>

              <v-list-item-content>
                <v-list-item-title> Task#{{task.taskid}} </v-list-item-title>
                <v-list-item-subtitle>

                  <v-chip class="ma-2" v-for="(job, index) in task.runList" :key="index"
                          @click="clickJob(job)" :color="chip_color(job)" label>
                   <v-avatar left> <v-icon>{{ chip_icon(job) }}</v-icon> </v-avatar>
                   {{job.jobname}}
                  </v-chip>
                </v-list-item-subtitle>
              </v-list-item-content>

            </v-list-item>

          </v-list-item-group>
        </v-list>
      </v-card>
    </v-container>
  </v-main>

  <v-footer color="deep-purple" dark app>
    <v-row justify="center">
      <strong>Approach0</strong>&nbsp; - &nbsp; {{ new Date().getFullYear() }}
    </v-row>
  </v-footer>

</v-app>
</template>

<script>
import axios from 'axios'
import moment from 'moment'

const merge = require('utils-merge')
const port = 8964

export default {
  data () {
    return {
      img_mounted: require('./assets/disk.jpg'),
      img_indexer: require('./assets/index.jpg'),
      img_searchd: require('./assets/search.jpg'),
      drawer: false,
      input: '',
      input_err_msg: null,
      single_job: false,
      dry_run: false,
      job_description: {},
      console_outsel: null,
      console_refresh: true,
      console_stickbt: true,
      console_loading: false,
      console_starjob: ['_master_'],
      console_content: '',
      status: {
        "mounted": null,
        "indexer": null,
        "searchd": null
      },
      updateStatus_enable: true,
      tasks: []
    }
  },

  computed: {
  },

  watch: {
    input: function (newVal, oldVal) {
      const val = newVal.trim()
      this.getJobDescription(val)
    }
  },

  mounted: function () {
    const vm = this
    setInterval(function () {
      const recent_job = vm.console_outsel || vm.console_starjob[0]
      if (vm.console_refresh) {
        vm.fetch_log(recent_job)
      }

      vm.update_tasks_list()

      if (vm.updateStatus_enable) {
        vm.updateStatus()
      }
    }, 1000)

    vm.run('check-alive:all', false, true)
    setInterval(() => {
      vm.run('check-alive:all', false, true)
    }, 15 * 1000 /* for command to run */)
  },

  methods: {
    chip_color(job) {
      if (job.alive)
        return 'blue'
      else if (job.exitcode == 0)
        return 'green'
      else if (job.pid < 0)
        return 'grey'
      else
        return 'red'
    },

    chip_icon(job) {
      if (job.alive)
        return 'toys'
      else if (job.exitcode == 0)
        return 'done'
      else if (job.pid < 0)
        return 'timer'
      else
        return 'error'
    },

    fetch_log(jobname) {
      const vm = this
      vm.console_loading = true

      if (this.console_stickbt) {
        var element = document.getElementById("console")
        element.scrollTop = element.scrollHeight
      }

      axios.get(`/get/log/${jobname}`)
      .then(function (res) {
        const data = res.data
        vm.console_content = data['logdata']
        vm.console_loading = false

        var index = vm.console_starjob.indexOf(jobname)
        if (index === -1) {
          vm.console_starjob.push(jobname)
        }
      })
      .catch(function (err) {
        console.error(err)
      })
    },

    run(jobname, manual, status) {
      const vm = this
      return new Promise((resolve, reject) => {
        axios.post(`/runjob`, {
          goal: jobname,
          dry_run: manual && vm.dry_run,
          single_job: manual && vm.single_job,
          status_task: status
        })
        .then(function (res) {
          const data = res.data

          if (manual) {
            if ('error' in data) {
              vm.input_err_msg = "Job is not defined."
              reject(vm.input_err_msg)
              return
            }

            vm.fetch_log(jobname)
          }

          resolve(data['taskID'])
        })
        .catch(function (err) {
          console.error(err)
          reject(err)
        })
      })
    },

    clickRun() {
      let input = this.input.trim()
      this.run(input, true, false)
    },

    clickStar(jobname) {
      if (jobname !== '_master_')
        this.input = jobname
      this.console_outsel = jobname
    },

    getJobDescription(jobname) {
      const vm = this
      let job = {
        'name': jobname
      }
      axios.get(`/get/job/${jobname}`)
      .then(function (res) {
        const data = res.data
        if ('error' in data) {
          vm.job_description = {}
          return
        }
        job = merge(data.props, job)
        vm.job_description = merge(job, vm.job_description)
      })
      .catch(function (err) {
        vm.job_description = {}
        console.error(err)
      })
    },

    clickJob(job) {
      this.drawer = true
      this.input = job.jobname

      this.job_description = {}
      this.getJobDescription(job.jobname)
      this.$set(this.job_description, 'pid', '' + job.pid + ' (' + (job.alive ? 'alive' : 'dead') + ')')
      //this.$set(this.job_description, 'alive', job.alive)
      this.$set(this.job_description, 'exitcode', job.exitcode)

      //this.$set(this.job_description, 'spawn_time', job.spawn_time)
      //this.$set(this.job_description, 'exit_time', job.exit_time)
      const start_moment = moment(job.spawn_time)
      const time_cost = moment(job.exit_time).from(start_moment, true)
      this.$set(this.job_description, 'start time', start_moment.fromNow())
      this.$set(this.job_description, 'time cost', time_cost)
    },

    update_tasks_list() {
      const vm = this
      axios.get(`/get/tasks`)
      .then(function (res) {
        const data = res.data
        const all_tasks = data['all_tasks']
        vm.tasks = all_tasks.reverse()
      })
      .catch(function (err) {
        console.error(err)
      })
    },

    async runAndWait() {
      const vm = this
      const arg = arguments
      const taskID = await vm.run(arg[0], arg[1], arg[2])

      return new Promise((resolve, reject) => {
        const timer = setInterval(() => {
          if (vm.isTaskFinished(taskID)) {
            resolve(taskID)
          }
        }, 1000)
      })
    },

    async changeStatus(key, goal) {
      this.status[key] = null
      this.updateStatus_enable = false

      await this.runAndWait(goal, false, false)
      console.log(`${goal} finished.`)

      await this.runAndWait('check-alive:all', false, true)
      console.log(`check-alive finished.`)

      this.updateStatus_enable = true
    },

    queryTaskID_runList(taskID) {
      const queryTasks = this.tasks.filter(t => t.taskid == taskID)
      if (queryTasks.length == 0)
        return []

      return queryTasks[0].runList
    },

    isTaskFinished(taskID) {
      const runList = this.queryTaskID_runList(taskID)
      if (runList.length == 0)
        return 0

      const last_job = runList[runList.length - 1]
      const err_jobs = runList.filter(j => j.exitcode > 0)

      if (err_jobs.length > 0 || last_job.exitcode >= 0) {
        return 1
      }

      return 0
    },

    updateStatus() {
      const vm = this
      const runList = this.queryTaskID_runList(0)
      runList.forEach(job => {
        const exitcode = job.exitcode
        switch (job.jobname) {

        case 'check-alive:mounted':
          if (exitcode >= 0) vm.status.mounted = (exitcode == 0)
          break

        case 'check-alive:indexer':
          if (exitcode >= 0) vm.status.indexer = (exitcode == 0)
          break

        case 'check-alive:searchd':
          if (exitcode >= 0) vm.status.searchd = (exitcode == 0)
          break
        }
      })
    }
  }
}
</script>

<style>
.console {
  margin-top: 20px;
  white-space: pre-wrap;
  overflow: auto;
  font-size: 14px;
  height: 60vh;
  padding: 8px;
}
</style>
