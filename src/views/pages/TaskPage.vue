<script setup>
    import { ref, inject, watchEffect, computed } from 'vue'
    import { useRoute, useRouter } from 'vue-router'
    import { VUE_APP_BACKEND_URL } from '../../../env'
    import ErrorText from '../components/ErrorText.vue'
    import axios from 'axios'

    const store = inject('store')
    const route = useRoute()
    const router = useRouter()
    // list selected employees for manager
    const employees = ref([])
    const projects = ref([])
    const errorMessage = ref('')
    const token = localStorage.getItem('accessToken') === null ? store.state.accessToken : localStorage.getItem('accessToken')
    const options = {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    }

    const task = ref()
    const subtask = ref(store.state.subtask)

    const title = ref('')
    const parentId = ref(null)
    const description = ref('')
    const priority = ref('')
    const status = ref('')
    const employeeId = ref('')
    const estimateHours = ref('')
    const completion = ref()
    const startDate = ref(new Date().toISOString().substring(0, 10))
    const endDate = ref(new Date().toISOString().substring(0, 10))
    const employeeName = ref('')
    const projectId = ref('')
    const projectName = ref('')
    const employeeReview = ref('')
    const managerReview = ref('')


    const resetTask = () => {
        title.value = null
        description.value = null
        priority.value = 'LOW'
        status.value = 'NEW'
        employeeId.value = null
        estimateHours.value = null
        completion.value = 0           
        startDate.value = new Date().toISOString().substring(0, 10)
        endDate.value = new Date().toISOString().substring(0, 10)
        if (!route.path.includes('/add-subtask')) {
            parentId.value = null
            projectId.value = null
        }
    }

    const isValidTitle = ref(true)
    const isValidParentId = ref(true)
    const isValidPriority = ref(true)
    const isValidStatus = ref(true)
    const isValidEmployeeName = ref(true)
    const isValidEstimateHours = ref(true)
    const isValidCompletion = ref(true)
    const isValidEndDate = ref(true)
    const isValidProjectId = ref(true)

    const resetValid = () => {
        isValidTitle.value = true
        isValidParentId.value = true
        isValidPriority.value = true
        isValidStatus.value = true
        isValidEmployeeName.value = true
        isValidEstimateHours.value = true
        isValidCompletion.value = true
        isValidEndDate.value = true
        isValidProjectId.value = true
    }

    const isValidInput = () => {
        if (projectId.value === null || projectId.value === '') {
            errorMessage.value = 'Project is required'
            isValidProjectId.value = false
        }
        else if (title.value === null || title.value === '') {
            errorMessage.value = 'Title is required'
            isValidTitle.value = false
        }
        else if (isNaN(parentId.value)) {
            errorMessage.value = 'Parent task must be a number'
            isValidParentId.value = false
        }
        else if (priority.value === null || priority.value === '') {
            errorMessage.value = 'Priority is required'
            isValidPriority.value = false
        }
        else if (status.value === null || status.value === '') {
            errorMessage.value = 'Status is required'
            isValidStatus.value = false
        }
        else if (employeeId.value === null || employeeId.value === '') {
            errorMessage.value = 'Employee is required'
            isValidEmployeeName.value = false
        }
        else if (estimateHours.value === null || estimateHours.value === '') {
            errorMessage.value = 'Estimate hours is required'
            isValidEstimateHours.value = false
        }
        else if (completion.value === null || completion.value === '') {
            errorMessage.value = 'Completion is required'
            isValidCompletion.value = false
        }
        else if (isNaN(estimateHours.value)) {
            errorMessage.value = 'Estimate hours must be a number'
            isValidEstimateHours.value = false
        }
        else if (Date.parse(endDate.value) - Date.parse(startDate.value) < 0) {
            errorMessage.value = 'End date must be greater than start date'
            isValidEndDate.value = false
        } 
        setTimeout(() => {
            errorMessage.value = ''
            resetValid()
        }, 1500)
    }

    const isDisabled = computed(() => {
        if (store.state.user.role === 'EMPLOYEE' && route.path.includes('/task-details')) {
            return true
        }

        return false
    })

    if (window.location.pathname === '/new-task' || window.location.pathname.includes('add-subtask')) {
        resetTask()
    }   

    if (subtask.value !== null) {
        parentId.value = subtask.value.parentId
        projectName.value = subtask.value.projectName
        projectId.value = subtask.value.projectId
    }

    watchEffect( async () => {
        store.state.isLoading = true

        if (store.state.user.role === 'EMPLOYEE' && (route.path.includes('/new-task') || route.path.includes('/add-subtask'))) {
            // get all projects for employee in TaskPage
            const allProjectsResponse = await axios.get(`${VUE_APP_BACKEND_URL}/api/employee/get-all-project-names`, options)
            projects.value = allProjectsResponse.data.data

            employeeName.value = store.state.user.firstName + ' ' + store.state.user.lastName
            employeeId.value = store.state.user.id
        }
        
        if (store.state.user.role === 'MANAGER') {
            // get all employees for manager in TaskPage
            const allEmployeesResponse = await axios.get(`${VUE_APP_BACKEND_URL}/api/manager/get-all-employees`, options)
            employees.value = allEmployeesResponse.data.data
    
            // get all projects for manager in TaskPage
            const allProjectsResponse = await axios.get(`${VUE_APP_BACKEND_URL}/api/manager/get-all-project-names`, options)
            projects.value = allProjectsResponse.data.data
        }

        if (route.path.includes('/task-details')) {
            if (store.state.user.role === 'MANAGER') {
                // get task by id for manager in TaskPage
                const taskByIdResponse = await axios.get(`${VUE_APP_BACKEND_URL}/api/manager/get-task/${route.params.id}`, options)
                task.value = taskByIdResponse.data.data
            }

            if (store.state.user.role === 'EMPLOYEE') {
                const taskByIdResponse = await axios.get(`${VUE_APP_BACKEND_URL}/api/employee/get-task/${route.params.id}`, options)
                task.value = taskByIdResponse.data.data 
            }

            title.value = task.value.title
            parentId.value = task.value.parentId
            description.value = task.value.description
            priority.value = task.value.priority
            status.value = task.value.status
            employeeId.value = task.value.employeeId
            estimateHours.value = task.value.estimateHours
            completion.value = task.value.completion
            startDate.value = task.value.startDate
            endDate.value = task.value.endDate
            employeeName.value = task.value.employeeName
            projectId.value = task.value.projectId
            projectName.value = task.value.projectName
            managerReview.value = task.value.managerReview
            employeeReview.value = task.value.employeeReview
        }

        store.state.isLoading = false
    })

    const handleClick = async () => {
        isValidInput()
        if (errorMessage.value !== '') {
            return
        }

        store.state.isLoading= true;
        try {
            if (store.state.user.role === 'EMPLOYEE') {
                let response = null;
                if (route.path.includes('task-details')) {
                        const task = {
                        status: status.value,
                        completion: completion.value,
                        employeeReview: employeeReview.value,
                    }
                    response = await axios.put(`${VUE_APP_BACKEND_URL}/api/employee/update-task/${route.params.id}`, task , options)
                
                
                    if (response.data.status === 200) {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000, true)
                    } else {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000)
                    }
                } else {
                    const task = {
                        title: title.value,
                        description: description.value,
                        status: status.value,
                        completion: completion.value,
                        priority: priority.value,
                        startDate: startDate.value,
                        endDate: endDate.value,
                        employeeId: employeeId.value,
                        estimateHours: estimateHours.value,
                        parentId: parentId.value,
                        projectId: projectId.value,
                        employeeReview: employeeReview.value,
                    }

                    isValidInput()
                    if (errorMessage.value !== '') {
                        return
                    }

                    const response = await axios.post(`${VUE_APP_BACKEND_URL}/api/employee/create-task`, task, options)
                    if (response.data.status === 200) {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000, true)
                        // reset task if create new task successfully
                        resetTask()
                    } else {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000)
                    }

                }
            } else {             
                // for manager
                const task = {
                    title: title.value,
                    description: description.value,
                    status: status.value,
                    completion: completion.value,
                    priority: priority.value,
                    startDate: startDate.value,
                    endDate: endDate.value,
                    employeeId: employeeId.value,
                    estimateHours: estimateHours.value,
                    parentId: parentId.value,
                    projectId: projectId.value,
                    managerReview: managerReview.value
                }

                isValidInput()
                if (errorMessage.value !== '') {
                    return
                }

                if (route.path.includes('/task-details')) {
                    const response = await axios.put(`${VUE_APP_BACKEND_URL}/api/manager/update-task/${route.params.id}`, task, {
                        headers: {
                            "Authorization": `Bearer ${token}`,
                            'Content-Type': 'application/json'
                        }
                    })
                    
                    if (response.data.status === 200) {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000, true)
                    } else {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000)
                    }
                } else if (route.path.includes('/new-task') || route.path.includes('/add-subtask')) {
                    
                    const response = await axios.post(`${VUE_APP_BACKEND_URL}/api/manager/create-task`, task, options)
                    if (response.data.status === 200) {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000, true)
                        // reset task if create new task successfully
                        resetTask()
                    } else {
                        store.state.popup.displayForMilliSecond(response.data.message, 2000)
                    }
                }
            }
        } catch (error) {
            store.state.popup.displayForMilliSecond("Action failed", 2000)
        }
        store.state.isLoading= false
    }

    const hideTask = async () => {
        const id = route.params.id
        store.state.isLoading = true
        try {
            const response = await axios.delete(`${VUE_APP_BACKEND_URL}/api/manager/hide-task/${id}`, options)
            if (response.data.status === 200) {
                store.state.popup.displayForMilliSecond(response.data.message, 2000, true)
                router.push('/dashboard')
            } else {
                store.state.popup.displayForMilliSecond(response.data.message, 2000)
            }
        } catch (error) {
            store.state.popup.displayForMilliSecond("Action failed", 2000)
        }
        store.state.isLoading = false
    }
    
</script>

<template>
    <div class="wrapper">
        <header class="project">
            <div class="project_field">
                <h1>Project</h1>
                <select v-model="projectId" :disabled="isDisabled || route.path.includes('/add-subtask')">
                    <option disabled value="">Please select one</option>
                    <option v-if="store.state.user.role==='EMPLOYEE'" :value="projectId">{{ projectName }}</option>
                    <option v-for="project of projects" :value="project.id">{{ project.name }}</option>
                </select>
            </div>
            <ErrorText v-if="!isValidProjectId" :errorMessage="errorMessage" class="project_error-text"/>
        </header>
        <main>
            <div class="row-1">
                <div class="name-field">
                    <label for="title">Title</label>
                    <input type="text" id="title" v-model="title" :disabled="isDisabled">
                    <ErrorText v-if="!isValidTitle" :errorMessage="errorMessage"/>
                </div>

                <div class="parent_task-Field">
                    <label for="parent-task">Parent Task</label>
                    <input type="text" id="parent-task" v-model="parentId" :disabled="isDisabled || route.path.includes('/add-subtask')">
                    <ErrorText v-if="!isValidParentId" :errorMessage="errorMessage"/>
                </div>
            </div>

            <div class="row-2">
                <div class="description-field">
                    <label for="description">Description</label>
                    <textarea name="description" id="description" v-model="description" :disabled="isDisabled"></textarea>
                </div>
            </div>


            <div class="row-3">
                <!-- TODO: change the state -->
                <div class="review-field">
                    <label for="review">Employeee Review</label>
                    <textarea name="review" id="review" v-model="employeeReview" :disabled="store.state.user.role === 'MANAGER'"></textarea>
                </div>
            </div>
            

            <div class="row-3">
                <div class="review-field">
                    <label for="review">Manger Review</label>
                    <textarea name="review" id="review" v-model="managerReview" :disabled="store.state.user.role === 'EMPLOYEE'"></textarea>
                </div>
            </div>
            
            

            <div class="footer-field">
                <div class="selected-field">
                    <div class="priority">
                        <div>Priority</div>
                        <select v-model="priority" :disabled="isDisabled">
                            <option disabled value="">Please select one</option>
                            <option>LOW</option>
                            <option>MEDIUM</option>
                            <option>HIGH</option>
                        </select>
                        <ErrorText v-if="!isValidPriority" :errorMessage="errorMessage"/>
                    </div>
                    <div class="status">
                        <div>Status</div>
                        <select v-model="status" :disabled="isDisabled && employeeId !== store.state.user.id && parentId !== ''">
                            <option disabled value="">Please select one</option>
                            <option>NEW</option>
                            <option value="IN_PROGRESS">IN-PROGRESS</option>
                            <option value="READY_FOR_REVIEW">READY FOR REVIEW</option>
                            <option>DONE</option>
                        </select>
                        <ErrorText v-if="!isValidStatus" :errorMessage="errorMessage"/>
                    </div>
                    <div class="in-charge">
                        <div>In-Charge</div>
                        <select v-model="employeeId" :disabled="isDisabled">
                            <option disabled value="">Please select one</option>
                            <option v-if="store.state.user.role==='EMPLOYEE'" :value="employeeId">{{ employeeName }}</option>
                            <option v-for="employee of employees" :value="employee.id">{{ employee.firstName + " " + employee.lastName }}</option>
                        </select>
                        <ErrorText v-if="!isValidEmployeeName" :errorMessage="errorMessage"/>
                    </div>
                </div>

                <div class="additional-field">
                    <div class="done">
                        <div>Completion(%)</div>
                        <select v-model="completion" :disabled="isDisabled && employeeId !== store.state.user.id && parentId !== ''">
                            <option disabled value="">Please select one</option>
                            <option>0</option>
                            <option>10</option>
                            <option>20</option>
                            <option>30</option>
                            <option>40</option>
                            <option>50</option>
                            <option>60</option>
                            <option>70</option>
                            <option>80</option>
                            <option>90</option>
                            <option>100</option>
                        </select>
                        <ErrorText v-if="!isValidCompletion" :errorMessage="errorMessage"/>
                    </div>
                    <div class="estimate-field">
                        <label for="estimate">Estimate Time</label>
                        <input type="number" id="estimate" v-model="estimateHours" :disabled="isDisabled">
                        <ErrorText v-if="!isValidEstimateHours" :errorMessage="errorMessage"/>
                    </div>
                    <div class="start_date-field">
                        <label for="start_date">Start date</label>
                        <input type="date" id="start_date" v-model="startDate" :disabled="isDisabled">
                    </div>
                    <div class="end_date-field">
                        <label for="end_date">Due date</label>
                        <input type="date" id="end_date" v-model="endDate" :disabled="isDisabled">
                        <ErrorText v-if="!isValidEndDate" :errorMessage="errorMessage"/>
                    </div>
                </div>
            </div>
            <footer>
                <button @click="hideTask" v-show="route.path.includes('/task-details') && store.state.user.role === 'MANAGER'">Hide it and its subtasks</button>
                <button @click="handleClick" :disabled="isDisabled && employeeId !== store.state.user.id && parentId !== ''">Save</button>
            </footer>
        </main>
    </div>
</template>

<style scoped lang="scss">
    input, textarea, select {
        font-size: small;
        width: 100%;
        height: 40px;
        border: 1px solid #ccc;
        border-radius: 4px;
        margin-top: 10px;
    }

    textarea {
        width: 100%;
        height: 100px;
        resize: none;
    }

    .wrapper {
        display: flex;
        flex-direction: column;
        gap:34px;
        padding: 20px;

        .project {
            display: flex;
            flex-direction: column;

            .project_field {
                width: 95%;
                display: flex;
                gap: 10px;
                align-items: center;
            }

            .project_error-text {
                margin-left: 115px;
            }
        }
    }
    
    main {
        display: flex;
        flex-direction: column;
        padding: 0 80px 0;
        gap: 34px;
    }

    .row-1 {
        display: grid;
        justify-content: space-between;
        gap: 15px;
        grid-template-columns: 1fr 1fr;

        input {
            padding: 0 0 0 5px;
        }
    }

    .footer-field {
        display: grid;
        gap: 20px;
        grid-template-columns: 1fr 1fr;
        justify-content: space-between;
    }

    .selected-field {
        display: grid;
        justify-content: space-between;
        grid-template-columns: 150px 150px;
        grid-template-rows: 1fr 1fr;
        gap: 15px;
    }

    .in-charge {
        grid-column-start: 1;
        grid-column-end: 3;
    }

    .additional-field {
        padding-left: 5px;
        display: grid;
        justify-content: space-between;
        gap: 15px;
        grid-template-columns: 1fr 1fr;

        input {
            padding: 0 0 0 5px;
        }
    }

    footer {
        display: flex;
        gap: 20px;
        justify-content: flex-end;
    }

    button {
        background-color: var(--primary);
        color: #fff;
        cursor: pointer;
        border-radius: 5px;
        padding: 10px;
        min-width: 200px;
        height: 44px;
    }

    button:disabled {
        background-color: #ccc;
        cursor: not-allowed;
    }
</style>