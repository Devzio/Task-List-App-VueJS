<template>
  <v-app id="inspire">

    <!-- Top App Bar -->
    <v-app-bar absolute color="#6A96AB" dark src="https://picsum.photos/1920/1080?random" fade-img-on-scroll>
      <template v-slot:img="{ props }">
        <v-img
          v-bind="props"
          gradient="to top right, rgba(100,115,201,.7), rgba(28, 218, 5, 0.5)"
        ></v-img>
      </template>
      <v-spacer class="d-none d-md-flex d-lg-flex d-xl-flex"></v-spacer>
      <v-app-bar-title class="flex text-center"><h2>Task List App</h2></v-app-bar-title>
    </v-app-bar>

    <h2 class="pt-0 pt-lg-12 light-green-bg">-</h2>
    <v-main class="pt-16 pt-md-16 light-green-bg">

      <!-- Input fields - Task Name Input, Date Picker Input and Add Task Button -->
      <v-layout align-center justify-center >
        <div width="900px">
          <v-form ref="form" v-model="valid" lazy-validation>
            <v-card class="px-3 mb-8 mx-4">
              <v-row>

                <v-col cols="12" md="5">
                  <!-- Task Name Text field -->
                  <v-text-field
                    v-model="newTaskTitle"
                    label="Task Name"
                    color="green darken-1"
                    hide-details
                    clearable
                    outlined
                    required
                    @keyup.enter="addTask"
                  ></v-text-field>
                </v-col>

                <v-col cols="12" md="5">
                  <!-- Task Due Date Picker -->
                  <v-dialog ref="dialog" v-model="modal" :return-value.sync="newDueDate" persistent required width="290px">
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="newDueDate"
                        color="green darken-1"
                        label="Due Date"
                        append-icon="mdi-calendar"
                        readonly
                        outlined
                        hide-details
                        v-bind="attrs"
                        v-on="on"
                        required
                        @keyup.enter="addTask"
                      ></v-text-field>
                    </template>
                    <v-date-picker v-model="date" scrollable color="green darken-1" show-adjacent-months>
                      <v-spacer></v-spacer>
                      <v-btn text color="green darken-1" @click="modal = false">Cancel</v-btn>
                      <v-btn text color="green darken-1" @click="$refs.dialog.save(date)">OK</v-btn>
                    </v-date-picker>
                  </v-dialog>
                </v-col>

                <v-col cols="12" md="2" >
                  <!-- Add Task Button -->
                  <v-layout align-center justify-center>
                    <v-btn :disabled="!valid" color="success" height="55px" width="100%" @click="addTask" >
                      <v-icon>mdi-plus</v-icon>
                      <span class="d-flex d-md-none">Add Task</span>
                    </v-btn>
                  </v-layout>
                </v-col>

              </v-row>
            </v-card>
          </v-form>
        </div>
      </v-layout>
      
      <!-- Task List Card -->
      <v-layout align-center justify-center class="mx-4 mx-lg-0 my-lg-10">
        <v-card height="560px" width="900px" rounded="lg" class="white border mb-4">
          <v-list flat subheader three-line v-if="tasks.length">
            <v-subheader>Tasks List</v-subheader>
            <v-divider></v-divider>
            <!-- Task List -->
            <v-list-item-group class="list-item-group">
              <div v-for="task in visibleTasks" :key="task.id">
                <v-list-item @click="doneTask(task.id)" :class="{ 'grey lighten-3' : task.done }">
                  <template v-slot:default>
                    
                    <v-list-item-action>
                      <v-checkbox :input-value="task.done" color="success" hide-details></v-checkbox>
                    </v-list-item-action>

                    <v-list-item-content>
                      <v-list-item-title class="text-lg-h6 font-weight-regular" :class="{ 'text-decoration-line-through' : task.done }">{{ task.title }}</v-list-item-title>
                      <v-list-item-subtitle><v-icon v-if="task.dueDate" small class="calendar-icon">mdi-calendar</v-icon>{{ task.dueDate }}</v-list-item-subtitle>
                      <!-- To use niceDate use this (buggy, needs debug): {{ task.dueDate | niceDate }} -->
                    </v-list-item-content>

                    <v-list-item-action>
                      <v-btn class="trash-icon mt-3 " icon @click.stop="deleteTask(task.id)">
                        <v-icon color="red lighten-1" size="35px">mdi-trash-can-outline</v-icon>
                      </v-btn>
                    </v-list-item-action>

                  </template>
                </v-list-item>
                <v-divider></v-divider>
              </div>
            </v-list-item-group>
            <div class="text-center mt-4">
              <v-pagination 
                class="pagination"
                color="green darken-1"
                circle
                v-model="page" 
                :length="pages" 
                @input="updatePage">
              </v-pagination>
            </div>
            
          </v-list>
          <!-- No Tasks Page -->
          <div v-else class="no-tasks">
            <v-icon color="green" size="100px">mdi-check</v-icon>
            <div class="text-h5 green--text">No Tasks</div>
          </div>
        </v-card>
      </v-layout>
    </v-main>
  </v-app>
</template>

<script>
import { format } from 'date-fns'

  export default {
    // Date formatting
    // filters: {
    //   niceDate(date){
    //     return format(new Date(date), 'dd MMM yyyy')
    //   }
    // },
    data() {
      return {
        newTaskTitle: "",
        newDueDate: "",
        tasks: [
          {
            id: 1,
            title: 'Add a New Task by typing in the Task Name, then click the plus button ⬆⬆⬆',
            dueDate: '2023-03-03',
            done: false
          },
          {
            id: 2,
            title: 'You may add a new task with / without a due date',
            dueDate: '2023-03-03',
            done: false
          },
          {
            id: 3,
            title: 'Delete a Task using the Trashcan icon ➡ ➡ ➡',
            dueDate: '2023-03-03',
            done: false
          },
          {
            id: 4,
            title: '⬅ ⬅ ⬅ Check a Task off when Done by clicking the checkbox',
            dueDate: '2023-03-03',
            done: false
          },
          {
            id: 5,
            title: 'You can use the pagination below to see more tasks ⬇ ⬇ ⬇',
            dueDate: '2023-03-03',
            done: false
          },
          {
            id: 6,
            title: 'There are more tasks here ⬇ ⬇ ⬇',
            dueDate: '2023-03-03',
            done: false
          },
          {
            id: 7,
            title: 'Delete all tasks to view the no tasks page',
            dueDate: '2023-03-03',
            done: false
          },
          {
            id: 8,
            title: 'This is also a mobile responsive webpage',
            dueDate: '',
            done: false
          },
        ],
        page: 1,
        pageSize: 5,
        tasksCount: 0,
			  visibleTasks: []

      }
    },
    created() {
      let _this = this;
      _this.initPage();
      _this.updatePage(_this.page);
    },
    methods: {
      // Done Task
      doneTask(id) {
        let task = this.tasks.filter(task => task.id === id)[0]
        task.done = !task.done
      },
      // Delete Task
      deleteTask(id) {
        this.tasks = this.tasks.filter(task => task.id !== id)
        let _this = this;
        _this.initPage();
        _this.updatePage(_this.page);
      },
      // Add Task
      addTask() {
        if(this.newTaskTitle || this.newTaskTitle && this.newDueDate){
          let newTask = {
            id: Date.now(),
            title: this.newTaskTitle,
            dueDate: this.newDueDate,
            done: false
          }
          this.tasks.push(newTask)
          this.newTaskTitle = ""
          this.newDueDate = ""
          let _this = this;
          _this.initPage();
          _this.updatePage(_this.page);
        }
      },
      
      // Pagination Handling
      initPage() {
        let _this = this;
        _this.tasksCount = _this.tasks.length;
        if (_this.tasksCount < _this.pageSize) {
          _this.visibleTasks = _this.tasks;
        } else {
          _this.visibleTasks = _this.tasks.slice(0, _this.pageSize);
        }
      },
      updatePage(pageIndex) {
        let _this = this;
        let _start = (pageIndex - 1) * _this.pageSize;
        let _end = pageIndex * _this.pageSize;
        _this.visibleTasks = _this.tasks.slice(_start, _end);
        _this.page = pageIndex;

        // if there are 0 visibleTasks then go back 1 page
        if(_this.visibleTasks.length == 0 && _this.page > 0) {
          this.initPage(_this.page - 1);
        }
      }

    },
    computed: {
      // Calculates number of pages required for pagination handling
      pages() {
        let _this = this;
        if (_this.pageSize == null || _this.tasksCount == null) return 0;
        return Math.ceil(_this.tasksCount / _this.pageSize);
      }
    }
  }
</script>

<style lang="sass">
.v-input--checkbox .theme--light.v-icon
  color: rgb(255, 208, 0)
  font-size: 35px
  margin-top: 20px
  margin-left: 4px

.v-input--selection-controls__ripple
  height: 40px
  width: 40px
  left: -11px
  top: calc(50% - 18px)

.greentick
  color: rgb(8, 248, 0)

.trash-icon .v-icon
  opacity: 50%

.trash-icon .v-icon:hover
  opacity: 1

.no-tasks
  position: absolute
  left: 50%
  top: 50%
  transform: translate(-50%, -50%)
  opacity: 0.8

.calendar-icon
  margin-bottom: 3px
  margin-right: 5px

.light-green-bg
  background-color: #f5fff5

.list-item-group
  height: 440px

.v-list-item:hover
  background-color: #f5f5f7
</style>