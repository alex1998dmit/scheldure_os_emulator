<template>
    <div class="row">
        <div class="col-md-12">
            <div class="memory-info block text-left">
                <h5>Memory info</h5>
                <h6>{{ availableSystemMemory }} Mb / {{ system.memory }} Mb</h6>
                <div class="progress">
                    <div class="progress-bar" role="progressbar" :style="{ width: availableSystemMemory / (system.memory / 100) + '%' }" v-bind:aria-valuenow="availableSystemMemory / (system.memory / 100)" aria-valuemin="0" aria-valuemax="100">
                        {{ Math.floor(availableSystemMemory / (system.memory / 100)) }} %
                    </div>
                </div>
            </div>
        </div>
        <div class="col-md-12">
            <div class="block executing-process-info text-left">
                <h5>Executing process</h5>
                <h6>Process id: {{ processes.current.id}}</h6>
                <h6>Process name: {{ processes.current.name }}</h6>
                <div class="progress">
                    <div class="progress-bar" role="progressbar" :style="{ width: (utils.executingCounter / Math.floor(processes.current.time / 1000)) * 100 + '%' }" v-bind:aria-valuenow="(utils.executingCounter / Math.floor(processes.current.time / 1000)) * 100" aria-valuemin="0" aria-valuemax="100">
<!--                        {{ Math.floor((utils.executingCounter / Math.floor(processes.current.time / 1000)) * 100) }} %-->
                    </div>
                </div>
            </div>
        </div>
        <div class="col-md-12 text-left">
            <div class="control-buttons block">
                <button class="btn btn-control btn-primary btn-xs" @click="executeProcesses">
                    Start execute
                </button>
                <button class="btn btn-control btn-success" @click="addProcess">
                    Add process
                </button>
                <button class="btn btn-control btn-danger" @click="callInterrupt">
                    Call interrupt
                </button>
            </div>
        </div>
        <div class="col-md-12">
            <table class="table">
                <thead>
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Priority</th>
                    <th>Execute time</th>
                    <th>Memory</th>
                    <th>Execute proccess</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="process in sortedProcesses" :key="process.id" :style="{ color: process.id === processes.current.id ? 'lightblue' : 'black' }">
                    <td>{{ process.id }}</td>
                    <td>{{ process.name }}</td>
                    <td>{{ process.priority }}</td>
                    <td>{{ process.time }}</td>
                    <td>{{ process.memory }}</td>
                    <td>{{ process.status }}</td>
                </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
    import { uniqueNamesGenerator } from 'unique-names-generator';

    export default {
        name: "Schedule",
        data () {
            return {
                processes: {
                    items: [],
                    current: {
                        index: null,
                        id: null,
                        name: null,
                        memory: null,
                        time: null,
                        executing_time: 0,
                    },
                    amount: null,
                },
                utils: {
                    timers: [],
                    currentTimer: {
                        executingProcessTimerId: null,
                        clearExecutingTimerId: null
                    },
                    executingCounter: 0
                },
                system: {
                    memory: 1024
                }
            }
        },
        computed: {
            sortedProcesses () { return this.processes.items.sort((a, b) => { return a.priority > b.priority ? -1 : 1 }) },
            availableSystemMemory () { return this.system.memory - this.processes.current.memory }
        },
        mounted() {
            this.generateRandomProcesses()
        },
        methods: {
            generateRandomProcesses (step = 0, processesAmount = 3) {
                this.generateRandomProcess(step)
                if (step === processesAmount) { return }
                step++
                return this.generateRandomProcesses(step)
            },
            executeProcesses () {
                let delay = 0
                this.sortedProcesses.forEach((process, processIndex) => {
                    if (process.status === 'Await') {
                        this.utils.timers.push(
                            { 'timerId' : setTimeout(() => {
                                // process executing
                                const executingProcessTimerId = setInterval(() => {
                                    console.log('executing', process.name)
                                    this.utils.executingCounter = this.utils.executingCounter + 1
                                }, 1000)
                                this.processes.current = { ...process, index: processIndex}
                                const clearExecutingTimerId = setTimeout(() => {
                                    // process finished
                                    clearInterval(executingProcessTimerId)
                                    process.status = "Finished"
                                    this.utils.executingCounter = 0
                                }, process.time)
                                this.utils.currentTimer = { executingProcessTimerId, clearExecutingTimerId }
                                }, delay), 'processName': process.name }
                        )
                        delay = delay + process.time + 1000
                    }
                })
            },
            generateRandomProcess (step = this.processes.items.length) {
                this.processes.items.push({
                    id: step,
                    name: uniqueNamesGenerator(),
                    type: 'process',
                    priority: Math.floor(Math.random() * 10),
                    time: Math.floor(Math.random() * 10000),
                    status: 'Await',
                    memory: Math.floor(Math.random() * (this.system.memory - 1) + 1)
                })
            },
            generateInterruptProcess() {
                this.processes.items.push({
                    id: this.processes.items.length + 100,
                    name: 'interrupt',
                    type: 'interrupt',
                    priority: Math.floor(Math.random() * 10000),
                    time: Math.floor(Math.random() * 10000),
                    status: 'Await',
                    memory: Math.floor(Math.random() * (this.system.memory - 1) + 1)
                })
            },
            addProcess () {
                this.generateRandomProcess()
                this.updateExecutingProcess()
                this.clearAllTimers()
                this.removeAllProcessesTimers()
                this.executeProcesses()
            },
            callInterrupt () {
                this.generateInterruptProcess()
                this.clearAllTimers()
                this.removeAllProcessesTimers()
                this.executeProcesses()
            },
            clearAllTimers () {
                this.clearAllProcessesTimers()
                this.clearExecutingTimerId()
            },
            clearAllProcessesTimers () {
              this.utils.timers.forEach(timer => {
                  clearInterval(timer.timerId)
              })
            },
            clearExecutingTimerId () {
              clearInterval(this.utils.currentTimer.executingProcessTimerId)
              clearInterval(this.utils.currentTimer.clearExecutingTimerId)
            },
            removeAllProcessesTimers () {
              this.utils.timers.splice(0, this.utils.timers.length)
            },
            // Переименоват, функция для пересборки текущего процесса
             updateExecutingProcess () {
                const process = this.sortedProcesses[this.processes.current.index]
                this.sortedProcesses[this.processes.current.index].time = process.time - this.utils.executingCounter * 1000
                this.sortedProcesses[this.processes.current.index].priority = process.priority * 100
             },
            resetExecutingProcesses() {
                this.utils.timers.forEach((timer) => {
                    if (this.processes.current.name !== timer.processName) {
                        clearInterval(timer.timerId)
                    }
                })
            },
        }
    }
</script>

<style scoped>
    .block {
        padding: 10px
    }
    .btn-control {
        margin: 10px;
    }
</style>


<!--Если пришел новый процесс то ожидаем конец текущего процесса, запускаем таймеры следующих идущих процессов, -->