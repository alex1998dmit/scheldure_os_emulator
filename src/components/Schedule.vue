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
                        {{ Math.floor((utils.executingCounter / Math.floor(processes.current.time / 1000)) * 100) }} %
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
                        id: null,
                        name: null,
                        memory: null,
                        time: null,
                    },
                    amount: null,
                },
                utils: {
                    timers: [],
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
                            setTimeout(() => {
                                // process executing
                                const timerId = setInterval(() => {
                                    console.log('executing', process.name)
                                    this.utils.executingCounter = this.utils.executingCounter + 1
                                }, 1000)
                                this.processes.current = process

                                setTimeout(() => {
                                    // process finished
                                    clearInterval(timerId)
                                    process.status = "Finished"
                                    this.utils.executingCounter = 0
                                    if (process.type === 'interrupt') {
                                        this.processes.items.splice(processIndex, 1)
                                    }
                                }, process.time)
                            }, delay))
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
            addProcess () {
                this.generateRandomProcess()
                this.resetExecutingProcesses()
                this.executeProcesses()
            },
            resetExecutingProcesses() {
                this.utils.timers.forEach(timer => { clearInterval(timer) })
            },
            generateInterruptProcess() {
                this.processes.items.push({
                    id: 1001,
                    name: 'interrupt',
                    type: 'interrupt',
                    priority: 10000,
                    time: Math.floor(Math.random() * 10000),
                    status: 'Await',
                    memory: Math.floor(Math.random() * (this.system.memory - 1) + 1)
                })
            },
            callInterrupt () {
                this.generateInterruptProcess()
                this.resetExecutingProcesses()
                this.executeProcesses()
            }
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