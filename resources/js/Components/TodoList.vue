<template>
  <div class="max-w-lg mx-auto p-8 bg-gradient-to-r from-pink-100 to-pink-200 rounded-2xl shadow-md">
    <h2 class="text-center text-3xl font-poppins font-semibold text-gray-700 mb-6">
      To-do List
    </h2>

    <!-- Form to Add a Task -->
    <form @submit.prevent="addTask" class="flex gap-4 mb-6">
      <input
        v-model="newTask"
        type="text"
        placeholder="Add a new task"
        required
        class="flex-grow px-5 py-3 border-2 border-pink-400 rounded-lg focus:outline-none focus:ring-2 focus:ring-pink-500 text-lg placeholder-pink-300"
      />
      <button
        type="submit"
        class="px-6 py-3 bg-pink-500 text-white rounded-lg hover:bg-pink-600 transform transition-all duration-200"
      >
        Add
      </button>
    </form>

    <!-- Display Tasks -->
    <ul class="space-y-4">
      <li
        v-for="task in tasks"
        :key="task.id"
        class="flex items-center justify-between bg-white shadow-sm rounded-lg py-4 px-5 hover:bg-pink-50 transition-all duration-200"
      >
        <div class="flex items-center">
          <input
            type="checkbox"
            v-model="task.completed"
            @change="updateTask(task)"
            class="w-5 h-5 text-pink-500 border-2 border-pink-400 rounded-full focus:ring-0"
          />
          <span
            :class="{ 'line-through text-gray-400': task.completed }"
            class="ml-4 text-lg text-gray-700 font-medium"
          >
            {{ task.task }}
          </span>
        </div>
        <button
          @click="deleteTask(task.id)"
          class="px-4 py-2 bg-red-500 text-white rounded-full hover:bg-red-600 text-sm focus:outline-none"
        >
          Delete
        </button>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      tasks: [], // List of tasks
      newTask: "", // Holds the new task input
    };
  },
  mounted() {
    this.loadTasksFromLocalStorage(); // Load tasks from localStorage on mount
    this.fetchTasks(); // Fetch existing tasks from the backend (optional)
  },
  methods: {
    // Load tasks from localStorage
    loadTasksFromLocalStorage() {
      const storedTasks = localStorage.getItem("tasks");
      if (storedTasks) {
        this.tasks = JSON.parse(storedTasks); // Parse and set tasks from localStorage
      }
    },

    // Save tasks to localStorage
    saveTasksToLocalStorage() {
      localStorage.setItem("tasks", JSON.stringify(this.tasks));
    },

    // Fetch tasks from the API
    fetchTasks() {
      axios
        .get("/api/tasks")
        .then((response) => {
          this.tasks = response.data;
          this.saveTasksToLocalStorage(); // Save fetched tasks to localStorage
        })
        .catch((error) => {
          console.error("Error fetching tasks:", error);
        });
    },

    // Add a new task
    addTask() {
      if (!this.newTask.trim()) {
        alert("Task cannot be empty!");
        return;
      }

      const newTask = {
        id: Date.now(), // Unique ID based on timestamp (local-only tasks)
        task: this.newTask,
        completed: false,
      };

      this.tasks.push(newTask); // Add the task to the list
      this.saveTasksToLocalStorage(); // Save to localStorage
      this.newTask = ""; // Clear the input field

      // Optional: Save to backend if needed
      axios
        .post("/api/tasks", { task: newTask.task })
        .then((response) => {
          // Sync with backend task ID
          newTask.id = response.data.id;
          this.saveTasksToLocalStorage(); // Save updated tasks
        })
        .catch((error) => {
          console.error("Error adding task to backend:", error);
        });
    },

    // Update task completion status
    updateTask(task) {
      this.saveTasksToLocalStorage(); // Save updated status locally

      // Optional: Save to backend if needed
      axios
        .patch(`/api/tasks/${task.id}`, { completed: task.completed })
        .catch((error) => {
          console.error("Error updating task:", error);
        });
    },

    // Delete a task
    deleteTask(id) {
      this.tasks = this.tasks.filter((task) => task.id !== id); // Remove the task
      this.saveTasksToLocalStorage(); // Save updated tasks to localStorage

      // Optional: Delete from backend if needed
      axios
        .delete(`/api/tasks/${id}`)
        .catch((error) => {
          console.error("Error deleting task:", error);
        });
    },
  },
};
</script>

