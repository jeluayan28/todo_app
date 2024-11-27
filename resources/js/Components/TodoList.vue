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
        <div class="flex gap-2">
          <button
            @click="openEditModal(task)"
            class="px-4 py-2 bg-blue-500 text-white rounded-full hover:bg-blue-600 text-sm focus:outline-none"
          >
            Edit
          </button>
          <button
            @click="deleteTask(task.id)"
            class="px-4 py-2 bg-red-500 text-white rounded-full hover:bg-red-600 text-sm focus:outline-none"
          >
            Delete
          </button>
        </div>
      </li>
    </ul>

    <!-- Modal for Editing a Task -->
    <div v-if="isModalVisible" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-white rounded-lg p-6 w-96 shadow-lg">
        <h3 class="text-xl font-semibold text-gray-700 mb-4">Update Task</h3>
        <input
          v-model="editedTask"
          type="text"
          class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-pink-500"
        />
        <div class="flex justify-end gap-2 mt-4">
          <button
            @click="confirmEditTask"
            class="px-4 py-2 bg-pink-500 text-white rounded-lg hover:bg-pink-600"
          >
            Confirm
          </button>
          <button
            @click="closeModal"
            class="px-4 py-2 bg-gray-300 text-gray-700 rounded-lg hover:bg-gray-400"
          >
            Cancel
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      tasks: [], // List of tasks
      newTask: "", // Holds the new task input
      editedTask: "", // Task being edited
      taskToEdit: null, // Task object to be updated
      isModalVisible: false, // Modal visibility state
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
        this.tasks = JSON.parse(storedTasks);
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
          this.saveTasksToLocalStorage();
        })
        .catch((error) => {
          console.error("Error fetching tasks:", error);
        });
    },

    // Add a new task
    addTask() {
      if (!this.newTask.trim()) {
        return;
      }

      const newTask = {
        id: Date.now(), // Temporary local-only ID
        task: this.newTask,
        completed: false,
      };

      // Add locally for instant UI update
      this.tasks.push(newTask);
      this.saveTasksToLocalStorage();
      this.newTask = "";

      // Save to the database
      axios
        .post("/api/tasks", { task: newTask.task })
        .then((response) => {
          // Update task ID with backend ID
          newTask.id = response.data.id;
          this.saveTasksToLocalStorage();
        })
        .catch((error) => {
          console.error("Error adding task to backend:", error);
        });
    },

    // Update task completion status
    updateTask(task) {
      this.saveTasksToLocalStorage(); // Save locally

      // Update in the database
      axios
        .patch(`/api/tasks/${task.id}`, { completed: task.completed })
        .catch((error) => {
          console.error("Error updating task:", error);
        });
    },

    // Delete a task
    deleteTask(id) {
      this.tasks = this.tasks.filter((task) => task.id !== id);
      this.saveTasksToLocalStorage();

      // Delete from the database
      axios
        .delete(`/api/tasks/${id}`)
        .catch((error) => {
          console.error("Error deleting task:", error);
        });
    },

    // Open the modal for editing a task
    openEditModal(task) {
      this.editedTask = task.task;
      this.taskToEdit = task;
      this.isModalVisible = true;
    },

    // Close the modal
    closeModal() {
      this.isModalVisible = false;
      this.editedTask = "";
      this.taskToEdit = null;
    },

    // Confirm task edit and update in the database
    confirmEditTask() {
      if (this.editedTask.trim() && this.taskToEdit) {
        const updatedTask = {
          ...this.taskToEdit,
          task: this.editedTask.trim(),
        };

        // Update locally for instant UI feedback
        this.taskToEdit.task = updatedTask.task;
        this.saveTasksToLocalStorage();

        // Update in the database
        axios
          .patch(`/api/tasks/${updatedTask.id}`, { task: updatedTask.task })
          .then(() => {
            this.saveTasksToLocalStorage(); // Ensure local sync
          })
          .catch((error) => {
            console.error("Error updating task in backend:", error);
          });

        this.closeModal(); // Close the modal after confirmation
      }
    },
  },
};
</script>
