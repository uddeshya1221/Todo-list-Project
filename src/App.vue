<template>
  <div id="app" class="container mt-5">
    <!-- Vue.js Logo -->
    <div class="vue-logo">
      <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSTHgnQcyAt960fDGsqDp0UqvLKMt1oRYifGA&usqp=CAU" height="100" alt="Vue.js Logo">
    </div>

    <!-- Title -->
    <h1 class="mb-4 text-center text-primary"><u>Vue.js To-Do List</u></h1>

    <!-- Open Tasks -->
    <div class="card mb-4">
      <div class="card-body">
        <h2 class="card-title">Open Tasks</h2>
        <ul class="list-group" type="none">
          <li v-for="task in filteredOpenTasks" :key="task.id" class="list-group-item d-flex justify-content-between align-items-center" transition="fade">
            {{ task.text }}
            <div>
              <span class="badge bg-primary rounded-pill">{{ formatDate(task.createdAt) }}</span>
            </div>
            <div class="button-group">
              <button @click="completeTask(task)" class="btn btn-success btn-sm mr-1">Complete</button>
              <button @click="confirmDeleteTask(task)" class="btn btn-danger btn-sm">Delete</button>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <!-- Completed Tasks -->
    <div class="card mb-4">
      <div class="card-body">
        <h2 class="card-title">Completed Tasks</h2>
        <ul class="list-group" type="none">
          <li v-for="task in filteredCompletedTasks" :key="task.id" class="list-group-item d-flex justify-content-between align-items-center" transition="fade">
            {{ task.text }}
            <span class="badge bg-secondary rounded-pill">{{ formatDate(task.completedAt) }}</span>
          </li>
        </ul>
      </div>
    </div>

    <!-- Add New Task -->
    <div class="card">
      <div class="card-body">
        <h2 class="card-title">Add New Task</h2>
        <div class="input-group">
          <input v-model="newTask" type="text" class="form-control" placeholder="New Task" aria-label="New Task" aria-describedby="button-addon2">
          <div class="input-group-append">
            <button @click="addTask" class="btn btn-success" type="button">Add Task</button>
            <button @click="resetTaskInput" class="btn btn-outline-secondary ml-1" type="button">Clear</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";

export default {
  data() {
    return {
      tasks: [],
      newTask: "",
    };
  },
  computed: {
    filteredOpenTasks() {
      return this.tasks.filter((task) => !task.completed && !task.deleted);
    },
    filteredCompletedTasks() {
      return this.tasks.filter((task) => task.completed && !task.deleted);
    },
  },
  methods: {
    async deleteTask(task) {
      try {
        await axios.delete(`http://localhost:3000/tasks/${task.id}`);
        task.deleted = true;
      } catch (error) {
        console.error("Error deleting task", error);
      }
    },
    async fetchTasks() {
      try {
        const response = await axios.get("http://localhost:3000/tasks");
        this.tasks = response.data.map(task => ({
          ...task,
          deleting: false,
          deleted: false
        }));
      } catch (error) {
        console.error("Error fetching tasks", error);
      }
    },
    async addTask() {
      if (this.newTask.trim() === "") return;

      try {
        const response = await axios.post("http://localhost:3000/tasks", {
          text: this.newTask,
          createdAt: moment().format(),
          completed: false,
        });
        this.tasks.push({
          ...response.data,
          deleting: false,
          deleted: false
        });
        this.newTask = "";
      } catch (error) {
        console.error("Error adding task", error);
      }
    },
    async completeTask(task) {
      try {
        const response = await axios.patch(
          `http://localhost:3000/tasks/${task.id}`,
          {
            completed: true,
            completedAt: moment().format(),
          }
        );

        const updatedTask = response.data;
        const index = this.tasks.findIndex((t) => t.id === updatedTask.id);

        // Directly update the array using assignment
        this.tasks[index] = {
          ...updatedTask,
          deleting: false,
          deleted: false
        };
      } catch (error) {
        console.error("Error completing task", error);
      }
    },
    formatDate(dateString) {
      return moment(dateString).format("MMMM D, YYYY [at] h:mm A");
    },
    async confirmDeleteTask(task) {
      const confirmDelete = confirm("Are you sure you want to delete this task?");
      if (confirmDelete) {
        task.deleting = true;
        await this.deleteTask(task);
      }
    },
    resetTaskInput() {
      this.newTask = ""; // Clear the input field
    },
  },
  created() {
    this.fetchTasks();
  },
};
</script>

<style>
#app {
  text-align: center;
  padding: 20px; /* Add some padding for better readability */
}

.card {
  margin-bottom: 20px;
  border: 1px solid #ccc; /* Add border */
  border-radius: 10px; /* Add some border radius for a softer look */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* Add shadow */
}

.list-group-item {
  border: none;
}

.btn {
  cursor: pointer;
  margin-right: 5px;
  margin-left: 5px;
  margin-top: 10px;
  background-color: #2196F3;
  color: #FFF;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
.completed-tasks {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.completed-task {
  width: calc(33.33% - 20px);
  margin-bottom: 20px;
}

.completed-task .card-body {
  padding: 10px;
}

.vue-logo {
  margin-bottom: 20px;
}

.input-group {
  margin-bottom: 20px; /* Add margin below input group */
}

.button-group {
  margin-top: 10px; /* Add margin above button group */
}

/* Add margin between buttons */
.button-group button {
  margin-right: 5px;
  margin-left: 5px;
}
</style>
