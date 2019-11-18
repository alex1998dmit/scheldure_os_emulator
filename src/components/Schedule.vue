<template>
    <!--    Сбросить счетчик команд, отсюда и баг что счетчие команд не обнулился -->
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
        <hr style="width: 100%;">        
        <div class="col-md-12">
            <div class="block executing-process-info text-left">
                <div class="row">
                    <div class="col-md-6">
                        <h5>Executing process (Multi)</h5>
                        <h6>Process id: {{ processes.current.id}}</h6>
                        <h6>Process name: {{ processes.current.name }}</h6>
                    </div>
                    <div class="col-md-6" v-if="display.enable_no_multi">
                        <h5>Executing process (No Multi)</h5>
                        <h6>Process id: {{ processes.current_multi.id}}</h6>
                        <h6>Process name: {{ processes.current_multi.name }}</h6>
                    </div>
                </div>

                <div class="progress">
                    <div class="progress-bar" role="progressbar" :style="{ width: (utils.executingCounter / Math.floor(processes.current.time / this.system.processor_clock)) * 100 + '%' }" v-bind:aria-valuenow="(utils.executingCounter / Math.floor(processes.current.time / this.system.processor_clock)) * 100" aria-valuemin="0" aria-valuemax="100">
<!--                        {{ Math.floor((utils.executingCounter / Math.floor(processes.current.time / 1000)) * 100) }} %-->
                    </div>
                </div>
                <br>
                <div class="progress" v-if="display.enable_no_multi">
                    <div class="progress-bar" role="progressbar" :style="{ width: (utils.executing_timer_multi / Math.floor(processes.current_multi.time / this.system.processor_clock)) * 100 + '%' }" v-bind:aria-valuenow="(utils.executing_timer_multi / Math.floor(processes.current_multi.time / this.system.processor_clock)) * 100" aria-valuemin="0" aria-valuemax="100">
                    </div>
                </div>
            </div>
        </div>
        <hr style="width: 100%;">        
        <div class="col-md-12">
            <div class="show-command-status text-left">
                <h5>Current Command (Multi): {{ commands.current.description }}</h5>
            </div>
             <div class="show-command-status text-left" v-if="display.enable_no_multi">
                <h5>Current Command (Not Multi): {{ commands.current_multi.description }}</h5>
            </div>
        </div>
        <div class="col-md-12 text-left" v-if="display.enable_no_multi">
        <hr style="width: 100%;">        
            <div class="row">
                <div class="col-md-12">
                    <h5>
                        Multi time executing: {{ execute_time }}
                    </h5>
                    <h5>
                        No multi time executing: {{ execute_time_not_multi}}
                    </h5>
                    <h5>                        
                        Time difference: {{ execute_time_not_multi - execute_time }}
                    </h5>                    
                </div>
            </div>
        </div>
        <hr style="width: 100%;">        
        <div class="col-md-12" v-if="system.status === 'executing'">
            <div class="row">
                <div class="col-md-3 text-left">
                    <label for="">Speed (processor clock) [ms]</label>
                    <input type="number" class="form-control" v-model="system.processor_clock" min="1000" max="10000" step="1000" @change="changeProcessClock">
                </div>
                <div class="col-md-3 text-center">
                    <label for="">Shut down after execute </label>
                    <input type="checkbox" class="forn-control" v-model="system.shut_down_after_execute">
                </div>
                <div class="col-md-3 text-center">
                    <label for="">Show not multi mode </label>
                    <input type="checkbox" class="forn-control" @click="display.enable_no_multi = !display.enable_no_multi" v-model="display.enable_no_multi">
                </div>
                <div class="col-md-3 text-left">
                    <label for="">Processes amount</label>
                    <input type="number" min="1" max="15" step="1" class="form-control" v-model="system.processes_amount" @change="changeProcessesAmount()">
                </div>
            </div>
        </div>
        <div class="col-md-12 text-left">
            <div class="control-buttons block">
                <button class="btn btn-control btn-primary btn-xs" @click="executeProcesses(); executeMultiProcesses()" v-if="system.status === 'waiting'">
                    Start execute
                </button>
                <button class="btn btn-control btn-danger btn-xs" @click="shutDown();" v-if="this.system.status !== 'shuted down'">
                    Shut down
                </button>
                <!-- <button class="btn btn-control btn-primary btn-xs" @click="generateRandomProcesses(system.processes_amount)" v-if="this.system.status === 'shuted down'">
                    Restart
                </button> -->
                <button class="btn btn-control btn-danger btn-xs" @click="stopExecute();" v-if="system.status === 'executing'">
                    Stop execute
                </button>
                <button class="btn btn-control btn-primary btn-xs" @click="restartExecute(); updateExecutingProcess();" v-if="system.status === 'execute_stoped'">
                    Continue execute
                </button>
                <button class="btn btn-control btn-success" @click="addProcess" v-if="system.status === 'executing'">
                    Add random process
                </button>
                <button class="btn btn-control btn-success" data-toggle="modal" data-target="#addCustomModal" @click="stopExecute(); setCustomProcessName()" v-if="system.status === 'executing'">
                    Add custom process
                </button>
                <button class="btn btn-control btn-danger" @click="callInterrupt" v-if="system.status === 'executing'">
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
                    <th>Process Type</th>
                    <th>Memory</th>
                    <th>Execute proccess</th>
                    <th colspan="2">Control</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="(process, index) in sortedProcesses" :key="index" :style="{ color: process.id === processes.current.id ? 'lightblue' : 'black' }">
                    <td>{{ process.id }}</td>
                    <td>
                        {{ process.name }}
                        <input class="form-control" v-model="process.name" v-if="display.edit_process && process.status !== 'Finished'">
                    </td>
                    <td>
                        {{ process.priority }}
                         <input class="form-control" type="number" v-model="process.priority" v-if="display.edit_process && process.status !== 'Finished'">
                    </td>
                    <td> {{ process.time }}</td>
                    <td>
                        {{ process.type }}
                    </td>
                    <td>
                        {{ process.memory }}
                         <input class="form-control" type="number" v-model="process.memory" v-if="display.edit_process && process.status !== 'Finished'">
                    </td>
                    <td>{{ process.status }}</td>
                    <td>
                        <button class="btn-success" @click="stopExecute(); display.edit_process = true" v-if="!display.edit_process && process.status !== 'Finished'">
                            o
                        </button>
                        <button class="btn-success" @click="restartExecute(); display.edit_process = false" v-if="display.edit_process && process.status !== 'Finished'">
                            S
                        </button>
                    </td>
                    <td>
                        <button class="btn-xs btn-danger" @click="stopExecute(); removeProcess(index); restartExecute()">
                            x
                        </button>
                    </td>
                </tr>                
                </tbody>
            </table>
        </div>
        <div class="col-md-12 text-left" v-if="display.enable_no_multi">
            <div class="show-statics">
                <table class="table">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Name</th>
                            <th>Priority</th>
                            <th>Execute time</th>
                            <th>Process Type</th>
                            <th>Memory</th>
                            <th>Execute proccess</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="process in sortedMulitProcesses" :key="process.id" :style="{ color: process.id === processes.current_multi.id ? 'lightblue' : 'black' }">
                            <td>{{ process.id }}</td>
                            <td>{{ process.name }}</td>
                            <td>{{ process.priority }}</td>
                            <td>{{ process.time }}</td>
                            <td>{{ process.type }}</td>
                            <td>{{ process.memory }}</td>
                            <td>{{ process.status }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
        <div class="modal fade" id="addCustomModal" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLongTitle">Добавить новый процесс</h5>
                    </div>
                    <div class="modal-body">
                        <form action="">
                            <div class="form-group">
                                <label for="">Название</label>
                                <input type="text" class="form-control" v-model="processes.new_process.name">
                            </div>
                            <div class="form-group">
                                <label for="">Приоритет</label>
                                <input type="number" class="form-control" min="1" max="10" step="1" v-model="processes.new_process.priority">
                            </div>                            
                            <div class="form-group">
                                <label for="">Память</label>
                                <input type="number" class="form-control" min="1" :max="system.memory" step="100" v-model="processes.new_process.memory">
                            </div>
                            <div class="form-group">
                                <label for="">Количество команд</label>
                                <input type="number" class="form-control" min="1" max="10" step="1" v-model="processes.new_process.commands_amount">
                            </div>
                            <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="restartExecute()">Отменить</button>
                            <button type="button" class="btn btn-primary" data-dismiss="modal" @click="addProcess();">Добавить</button>
                        </form>
                    </div>                    
                </div>
            </div>
        </div>        
    </div>
</template>

<script>
    import { uniqueNamesGenerator } from 'unique-names-generator';

    export default {
        name: "Schedule",
        data () {
            return {
                commands: {
                    items: [
                        {
                            id: 0,
                            code: 'ReadFromMemory',
                            description: 'Чтение из памяти',
                            processStatus: 'Инициализация IO'
                        },
                        {
                            id: 1,
                            code: 'IncrementIP',
                            description: 'Увеличение счетчика команд',
                            processStatus: 'Активен'
                        },
                        {
                            id: 2,
                            code: 'RecognizeCodOperation',
                            description: 'Распознавание команды',
                            processStatus: 'Активен'
                        },
                        {
                            id: 3,
                            code: 'ReadOperands',
                            description: 'Чтение операндов из аппаратной памяти',
                            processStatus: 'Инициализация IO'
                        },
                        {
                            id: 4,
                            code: 'CodOрeration',
                            description: 'Выполнение операции',
                            processStatus: 'Активен'
                        },
                        {
                            id: 5,
                            code: 'WriteCodeResult',
                            description: 'Запись результата операции в аппаратную память',
                            processStatus: 'Инициализация IO'
                        },
                        {
                            id: 6,
                            code: 'Wait',
                            description: 'Ожидание на переключение',
                            processStatus: 'Ожидание'
                        }
                    ],
                    current: {
                        id: null,
                        code: null,
                        description: null
                    },
                    current_multi: {
                        id: null,
                        code: null,
                        description: null
                    }
                },
                processes: {
                    items: [],
                    itemsMulti: [],
                    current: {
                        index: null,
                        id: null,
                        name: null,
                        memory: null,
                        time: null,
                        executing_time: 0,
                        commands: [],
                    },
                    current_multi: {
                        index: null,
                        id: null,
                        name: null,
                        memory: null,
                        time: null,
                        executing_time: 0,
                        commands: []
                    },
                    new_process: {
                        name: null,
                        memory: null,
                        commands_amount: null,
                        priority: null
                    },
                    edit_process: {
                        name: null,
                        memory: null,
                        commands_amount: null,
                        priority: null
                    },
                    amount: null,
                },
                utils: {
                    isStart: false,
                    timers: [],
                    multiTimers: [],
                    executingProcessesTimers: [],
                    mutliExecutingTimers: [],
                    currentTimer: {
                        executingProcessTimerId: null,
                        clearExecutingTimerId: null
                    },
                    current_multiTimer: {
                        executingProcessTimerId: null,
                        clearExecutingTimerId: null                        
                    },
                    executingCounter: 0,
                    executing_timer_multi: 0
                },
                system: {
                    memory: null,
                    speed: null,
                    processes_amount: null,                    
                    processor_clock: null,
                    status: 'waiting',
                    shut_down_after_execute: true
                },
                display: {
                    enable_no_multi: false,
                    edit_process: false,
                }                
            }
        },
        computed: {
            sortedProcesses () { return this.processes.items.sort((a, b) => { return a.priority > b.priority ? -1 : 1 }) },
            sortedMulitProcesses () { return this.processes.itemsMulti.sort((a, b) => { return a.priority > b.priority ? -1 : 1 }) },
            availableSystemMemory () { return this.system.memory - this.processes.current.memory },
            execute_time () { return this.sortedProcesses.reduce((acc, item) => acc + item.time, 0) },
            execute_time_not_multi () { return this.sortedMulitProcesses.reduce((acc, item) => acc + item.time, 0)}
        },
        mounted() {
            this.system.memory = process.env.VUE_APP_MEMORY || 1024
            this.system.speed = process.env.VUE_APP_SPEED || 1.0,
            this.system.processes_amount = process.env.VUE_APP_PROCESSES_AMOUNT || 5
            this.system.processor_clock = process.env.VUE_APP_COMMAND_DURATION || 1000
            
            // this.processes.new_process.memory = process.env.VUE_APP_NEW_PROCESS_MEMORY || null
            // this.processes.new_process.priority = process.env.VUE_APP_NEW_PROCESS_PRIORITY || null
            // this.processes.new_process.commands_amount = process.env.VUE_APP_NEW_PROCESS_COMMANDS_AMOUNT || null
            this.generateRandomProcesses(this.system.processes_amount)
        },
        methods: {
            generateRandomProcesses (processesAmount) {
                for (let i = 0; i < processesAmount; i++) {
                    this.generateRandomProcess(i)
                }
            },
            executeProcesses () {
                this.system.status = 'executing'
                let delay = 0
                this.sortedProcesses.forEach((process, processIndex) => {
                    if (process.status !== 'Finished') {
                        this.utils.timers.push(
                            { 'timerId' : setTimeout(() => {
                                this.utils.executing_timer_multi = 0
                                // process executing
                                const executingProcessTimerId = setInterval(() => {
                                    this.utils.executingCounter = this.utils.executingCounter + 1
                                    this.commands.current = process.commands[this.utils.executingCounter - 1]
                                    process.status = process.commands[this.utils.executingCounter - 1].processStatus
                                    // shut down if last command at last command
                                    if (processIndex === this.sortedProcesses.length - 1 && this.system.shut_down_after_execute) {
                                        if (this.utils.executingCounter - 1 === process.commands.length - 1) {
                                            this.shutDown()
                                        }
                                    }                                    
                                }, this.system.processor_clock)
                                this.processes.current = { ...process, index: processIndex }
                                const clearExecutingTimerId = setTimeout(() => {
                                    // process finished
                                    clearInterval(executingProcessTimerId)
                                    process.status = "Finished"
                                    this.utils.executingCounter = 0
                                }, process.time)
                                this.utils.currentTimer = { executingProcessTimerId, clearExecutingTimerId }
                                }, delay), 'processName': process.name }
                        )
                        delay = delay + process.time
                    }
                })
            },
            executeMultiProcesses () {
                let delay = 0
                this.sortedMulitProcesses.forEach((process, processIndex) => {
                    if (process.status !== 'Finished') {
                        this.utils.multiTimers.push(
                            { 'timerId' : setTimeout(() => {
                                this.utils.executingCounter = 0
                                // process executing
                                const executingProcessTimerId = setInterval(() => {
                                    this.utils.executing_timer_multi = this.utils.executing_timer_multi + 1
                                    this.commands.current_multi = process.commands[this.utils.executing_timer_multi - 1]
                                    process.status = process.commands[this.utils.executing_timer_multi - 1].processStatus
                                }, this.system.processor_clock)              
                                this.processes.current_multi = { ...process, index: processIndex }
                                const clearExecutingTimerId = setTimeout(() => {
                                    // process finished
                                    clearInterval(executingProcessTimerId)
                                    process.status = "Finished"
                                    this.utils.executing_timer_multi = 0
                                }, process.time)
                                this.utils.current_multiTimer = { executingProcessTimerId, clearExecutingTimerId }
                                }, delay), 'processName': process.name 
                            }
                        )
                        delay = delay + process.time
                    }
                })     
            },
            generateRandomProcess (step = this.processes.items.length) {
                const commands = this.generateRandomCommandsForProcess()
                const commands_multi = [...commands, this.commands.items.find(command => command.code === 'Wait')]
                const time = commands.length * this.system.processor_clock
                const time_multu = commands_multi.length *  this.system.processor_clock
                const process = {
                    id: step,
                    name: this.processes.new_process.name || uniqueNamesGenerator(),
                    type: 'process',
                    priority: this.processes.new_process.priority || Math.floor(Math.random() * 10),
                    commands,
                    time,
                    status: 'Await',
                    memory: this.processes.new_process.memory || Math.floor(Math.random() * (this.system.memory - 1) + 1)
                }
                const multi_process = {
                    ...process, time: time_multu, commands: commands_multi
                }
                this.processes.items.push(process)
                this.processes.itemsMulti.push(multi_process)           
            },
            generateRandomCommandsForProcess () {
                const min = 0
                const max = this.commands.items.length - 1
                let commands = []
                const commands_amount = this.processes.new_process.commands_amount || Math.floor((Math.random() * 10) + 3)
                for (let i = 0; i < commands_amount; i++) {
                    const id = Math.floor(Math.random() * (max - min)) + min
                    commands.push(this.commands.items.find(command => command.id === id))
                }
                return commands
            },
            generateInterruptProcess() {
                const commands = this.generateRandomCommandsForProcess()
                const commands_multi = [...commands, this.commands.items.find(command => command.code === 'Wait')]
                const time = (commands.length) *  this.system.processor_clock
                const time_multi = (commands_multi.length) * this.system.processor_clock
                const interrupt_process = {
                    id: this.processes.items.length + 100,
                    name: 'interrupt',
                    type: 'interrupt',
                    priority: Math.floor(Math.random() * 1000000),
                    time,
                    commands,
                    status: 'Await',
                    memory: Math.floor(Math.random() * (this.system.memory - 1) + 1)
                }
                this.processes.items.push(interrupt_process)
                this.processes.itemsMulti.push({
                    ...interrupt_process, time: time_multi, commands: commands_multi
                })
            },
            addProcess () {
                this.resetExecutingCount()
                this.generateRandomProcess()
                this.updateExecutingProcess()                
                this.clearAllTimers()
                this.executeProcesses()
                this.executeMultiProcesses()                
            },
            removeProcess (index) {                
                if (confirm("Вы уверены, что хотите удалить процесс ?")) {
                    this.resetExecutingCount()
                    this.sortedProcesses.splice(index, 1)
                }
            },
            callInterrupt () {
                this.resetExecutingCount()
                this.updateExecutingProcess()
                this.updateExecutingProcessStatus()
                this.generateInterruptProcess()                
                this.clearAllTimers()
                this.executeProcesses()
                this.executeMultiProcesses()
            },
            // timers
            clearAllTimers () {
                // Not good
                for (let i = 0; i < 1000; i ++) {
                    clearInterval(i)
                }
            },
            clearExecutingTimerId () {
              clearInterval(this.utils.currentTimer.executingProcessTimerId)
              clearInterval(this.utils.currentTimer.clearExecutingTimerId)
            },
            removeAllProcessesTimers () {
              this.utils.timers.splice(0, this.utils.timers.length)
            },
            resetExecutingCount() {
              this.utils.executingCounter = 0
            },
            //------------
             updateExecutingProcess () {
                const process = this.sortedProcesses[this.processes.current.index]
                const process_multi = this.sortedMulitProcesses[this.processes.current_multi.index]

                this.sortedProcesses[this.processes.current.index].time = process.time - this.utils.executingCounter * 1000
                this.sortedMulitProcesses[this.processes.current_multi.index].time = process_multi.time - this.utils.executing_timer_multi * 1000

                this.sortedProcesses[this.processes.current.index].priority = process.priority * 10
                this.sortedMulitProcesses[this.processes.current_multi.index].priority = process.priority * 10
             },
            updateExecutingProcessStatus () {
                this.sortedProcesses[this.processes.current.index].commands.splice(0, this.utils.executingCounter)
                this.sortedMulitProcesses[this.processes.current_multi.index].commands.splice(0, this.utils.executing_timer_multi)

                this.sortedProcesses[this.processes.current.index].status = 'Blocked'
                this.sortedMulitProcesses[this.processes.current_multi.index].status = 'Blocked'
            },
            stopExecute () {
                this.system.status = 'execute_stoped'
                this.clearAllTimers()
            },
            restartExecute () {
                this.system.status = 'executing'
                this.executeProcesses()
                this.executeMultiProcesses()                
            },
            changeProcessClock () {
                this.stopExecute()
                this.sortedProcesses.forEach(process => { process.time = process.commands.length * this.system.processor_clock })
                this.sortedMulitProcesses.forEach(process => { process.time = process.commands.length * this.system.processor_clock })
                this.restartExecute()
            },
            changeProcessesAmount () {
                this.stopExecute()
                const new_processes_amount = this.system.processes_amount
                const processes_amount = this.sortedProcesses.length
                if (new_processes_amount > processes_amount) {
                    for (let i = processes_amount + 1; i <= Number(new_processes_amount); i++) {
                        this.generateRandomProcess(i)
                    }
                } else if (new_processes_amount < processes_amount) {
                    this.sortedProcesses.splice(0, this.sortedProcesses.length - Number(this.system.processes_amount))
                    this.sortedMulitProcesses.splice(0, this.sortedMulitProcesses.length - this.system.processes_amount)
                }
                this.restartExecute()
            },
            shutDown () {
                if (this.system.status === 'executing') {
                    this.stopExecute()
                }    
                this.system.status = 'shuted down'            
                this.sortedProcesses.splice(0, this.sortedProcesses.length)
                this.sortedMulitProcesses.splice(0, this.sortedMulitProcesses.length)
                this.restartExecute()
            },
            setCustomProcessName () {
                this.processes.new_process.name = process.env.VUE_APP_NEW_PROCESS_NAME
            }
        },
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