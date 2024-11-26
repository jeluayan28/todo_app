<template>
    <div class="todo-container">
      <h2 class="title">Todo List</h2>
  
      <!-- Form to Add a Task -->
      <form @submit.prevent="addTask" class="task-form">
        <input
          v-model="newTask"
          type="text"
          placeholder="Add a new task"
          required
          class="task-input"
        />
        <button type="submit" class="add-task-button">Add Task</button>
      </form>
  
      <!-- Display Tasks -->
      <ul class="task-list">
        <li v-for="task in tasks" :key="task.id" class="task-item">
          <input
            type="checkbox"
            v-model="task.completed"
            @change="updateTask(task)"
            class="task-checkbox"
          />
          <span :class="{ completed: task.completed }" class="task-text">
            {{ task.task }}
          </span>
          <button @click="deleteTask(task.id)" class="delete-button">Delete</button>
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

  
<style scoped>
.todo-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
  font-family: 'Arial', sans-serif;
}

.title {
  text-align: center;
  font-size: 2rem;
  color: #4a4a4a;
  margin-bottom: 20px;
  text-transform: uppercase;
  letter-spacing: 1.5px;
}

.task-form {
  display: flex;
  margin-bottom: 20px;
  gap: 12px;
}

.task-input {
  flex-grow: 1;
  padding: 12px;
  font-size: 1rem;
  border: 1px solid #dcdcdc;
  border-radius: 6px;
  background-color: #f5f5f5;
  transition: border-color 0.3s ease;
}

.task-input:focus {
  outline: none;
  border-color: #eed42b;
}

.add-task-button {
  padding: 12px 18px;
  background-color: #38a169;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: bold;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.add-task-button:hover {
  background-color: #2f855a;
  transform: translateY(-2px);
}

.task-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.task-item:last-child {
  border-bottom: none;
}

.task-checkbox {
  margin-right: 14px;
  width: 18px;
  height: 18px;
}

.task-text {
  flex-grow: 1;
  font-size: 1rem;
  color: #4a4a4a;
  word-break: break-word;
  transition: color 0.3s ease, text-decoration 0.3s ease;
}

.completed {
  text-decoration: line-through;
  color: #a0aec0;
}

.delete-button {
  background-color: #e53e3e;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.875rem;
  font-weight: bold;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.delete-button:hover {
  background-color: #c53030;
  transform: translateY(-2px);
}
</style>

  