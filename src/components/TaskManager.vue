<template>
  <div class="container my-5">
    <h1 class="text-center mb-4">To-Do List</h1>

    <!-- Success Notification -->
    <div
      v-if="notification"
      class="alert alert-success alert-dismissible fade show"
      role="alert"
    >
      {{ notification }}
      <button
        type="button"
        class="btn-close"
        aria-label="Close"
        @click="clearNotification"
      ></button>
    </div>

    <!-- Filter Options -->
    <div class="d-flex flex-column flex-sm-row justify-content-sm-between mb-4">
      <!-- Button - Create Task -->
      <button
        type="button"
        class="btn btn-primary order-1 order-sm-2 mb-3 mb-sm-0"
        data-bs-toggle="modal"
        data-bs-target="#addTaskModal"
      >
        Create task
      </button>

      <!-- Filter Tasks -->
      <div
        class="btn-group order-2 order-sm-1"
        role="group"
        aria-label="Filter Tasks"
      >
        <input
          type="radio"
          class="btn-check"
          value="all"
          v-model="filterStatus"
          id="filter-all"
          autocomplete="off"
          checked
        />
        <label class="btn btn-outline-secondary" for="filter-all">All</label>

        <input
          type="radio"
          class="btn-check"
          value="1"
          v-model="filterStatus"
          id="filter-completed"
          autocomplete="off"
        />
        <label class="btn btn-outline-success" for="filter-completed">
          Completed
        </label>

        <input
          type="radio"
          class="btn-check"
          value="0"
          v-model="filterStatus"
          id="filter-pending"
          autocomplete="off"
        />
        <label class="btn btn-outline-warning" for="filter-pending">
          Pending
        </label>
      </div>
    </div>

    <!-- Task List -->
    <div class="task-list-container">
      <ul class="list-group">
        <li
          v-for="task in filteredTasks"
          :key="task.id"
          class="list-group-item d-flex flex-column flex-md-row justify-content-md-between align-items-md-center"
        >
          <div class="flex-grow-1">
            <!-- Display Task Title -->
            <span
              :class="{
                'text-decoration-line-through': task.status,
                'text-muted': task.status,
              }"
              class="fw-bold"
            >
              <b>Title:</b> {{ task.title }}
            </span>

            <!-- Display Task Description -->
            <p
              :class="{
                'text-decoration-line-through': task.status,
                'text-muted': task.status,
              }"
              class="mb-1 small"
            >
              <b>Description:</b> {{ task.description }}
            </p>

            <!-- Display Created At and Updated At -->
            <p class="text-muted small mb-0">
              <b>Date Created:</b> {{ formatDate(task.created_at) }}
            </p>
            <p class="text-muted small mb-0">
              <b>Last Updated:</b> {{ formatDate(task.updated_at) }}
            </p>
          </div>

          <!-- Action Buttons -->
          <div>
            <button
              @click="toggleStatus(task)"
              :class="[
                'btn',
                task.status ? 'btn-warning' : 'btn-success',
                'my-2 my-md-0',
              ]"
            >
              {{ task.status ? "Mark as Pending" : "Mark as Completed" }}
            </button>
            <button
              @click="openEditModal(task)"
              class="btn btn-info mx-1"
              data-bs-toggle="modal"
              data-bs-target="#editTaskModal"
            >
              Edit
            </button>
            <button
              @click="confirmDelete(task)"
              class="btn btn-danger"
              data-bs-toggle="modal"
              data-bs-target="#deleteConfirmModal"
            >
              Delete
            </button>
          </div>
        </li>
      </ul>
    </div>

    <!-- Add Task Modal -->
    <div
      class="modal fade"
      id="addTaskModal"
      tabindex="-1"
      aria-labelledby="addModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog">
          <div class="modal-content">
            <form class="mb-4" @submit.prevent="addTask">
                <div class="modal-header">
                <h5 class="modal-title" id="addModalLabel">Add task</h5>
                <button
                    type="button"
                    class="btn-close"
                    data-bs-dismiss="modal"
                    aria-label="Close"
                    ref="addCloseButton"
                ></button>
                </div>
                <div class="modal-body">
                <div class="mb-3">
                    <input
                    v-model="newTask.title"
                    type="text"
                    class="form-control"
                    placeholder="Task title"
                    />
                    <div v-if="errors.title" class="text-danger mt-1">
                    {{ errors.title[0] }}
                    </div>
                </div>
                <div class="mb-3">
                    <textarea
                    v-model="newTask.description"
                    class="form-control"
                    placeholder="Task description"
                    ></textarea>
                    <div v-if="errors.description" class="text-danger mt-1">
                    {{ errors.description[0] }}
                    </div>
                </div>
                </div>
                <div class="modal-footer">
                <button
                    type="button"
                    class="btn btn-secondary"
                    data-bs-dismiss="modal"
                >
                    Close
                </button>
                <button type="submit" class="btn btn-primary">Add</button>
                </div>
            </form>
        </div>
      </div>
    </div>

    <!-- Edit Task Modal -->
    <div
      class="modal fade"
      id="editTaskModal"
      tabindex="-1"
      aria-labelledby="editModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="editModalLabel">Edit Task</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
              ref="editCloseButton"
            ></button>
          </div>
          <div class="modal-body">
            <form class="mb-4">
              <div class="mb-3">
                <input
                  v-model="editedTask.title"
                  type="text"
                  class="form-control"
                  placeholder="Task title"
                  required
                />
              </div>
              <div class="mb-3">
                <textarea
                  v-model="editedTask.description"
                  class="form-control"
                  placeholder="Task description"
                ></textarea>
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              data-bs-dismiss="modal"
            >
              Close
            </button>
            <button @click="updateTask" type="button" class="btn btn-primary">
              Update
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div
      class="modal fade"
      id="deleteConfirmModal"
      tabindex="-1"
      aria-labelledby="deleteModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title text-danger" id="deleteModalLabel">
              Delete Task
            </h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
              ref="deleteCloseButton"
            ></button>
          </div>
          <div class="modal-body">
            Are you sure you want to delete the task:
            <strong>{{ taskToDelete?.title }}</strong
            >?
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              data-bs-dismiss="modal"
            >
              Cancel
            </button>
            <button @click="deleteTask" type="button" class="btn btn-danger">
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
      
  <script>
import apiClient from "../services/api";
import moment from "moment";

export default {
  data() {
    return {
      tasks: [],
      newTask: { title: "", description: "", status: 0 },
      editedTask: { id: null, title: "", description: "", status: 0 }, // Task to edit
      filterStatus: "all",
      notification: "", // To store the success message
      taskToDelete: null,
      errors: {}, // Object to store validation errors
    };
  },
  computed: {
    filteredTasks() {
      if (this.filterStatus === "all") return this.tasks;
      return this.tasks.filter(
        (task) => task.status.toString() === this.filterStatus
      );
    },
  },
  methods: {
    async fetchTasks() {
      const response = await apiClient.get("/tasks");
      this.tasks = response.data;
    },
    async addTask() {
      // Reset errors before validation
      this.errors = {};

      // Frontend validation: Check if required fields are empty
      if (!this.newTask.title) {
        this.errors.title = ["The title field is required."];
        return; // Prevent submission
      }

      try {
        const response = await apiClient.post("/tasks", this.newTask);

        // If task creation is successful
        this.tasks.push(response.data);
        this.newTask = { title: "", description: "", status: 0 };

        // Close the modal programmatically
        this.$refs.addCloseButton.click();

        this.notification = "Task created successfully!";

        // Automatically clear notification after 3 seconds
        setTimeout(() => {
          this.clearNotification();
        }, 3000);
      } catch (error) {
        if (error.response && error.response.status === 422) {
          // If validation error occurred on the backend, capture the validation errors
          this.errors = error.response.data.errors;
        } else {
          console.error("Error adding task:", error);
          // Handle other errors (e.g., server error)
        }
      }
    },
    async updateTask() {
      try {
        const response = await apiClient.put(
          `/tasks/${this.editedTask.id}`,
          this.editedTask
        );
        const index = this.tasks.findIndex(
          (task) => task.id === this.editedTask.id
        );
        if (index !== -1) {
          this.tasks[index] = response.data;
        }
        this.editedTask = { id: null, title: "", description: "", status: 0 };
        this.$refs.editCloseButton.click();

        this.notification = "Task updated successfully!";

        // Automatically clear notification after 3 seconds
        setTimeout(() => {
          this.clearNotification();
        }, 3000);
      } catch (error) {
        console.error("Error updating task:", error);
      }
    },
    async deleteTask() {
      if (this.taskToDelete) {
        await apiClient.delete(`/tasks/${this.taskToDelete.id}`);
        this.tasks = this.tasks.filter(
          (task) => task.id !== this.taskToDelete.id
        );
        this.taskToDelete = null;
        this.$refs.deleteCloseButton.click();
        this.notification = "Task deleted successfully!";

        // Automatically clear notification after 3 seconds
        setTimeout(() => {
          this.clearNotification();
        }, 3000);
      }
    },
    openEditModal(task) {
      this.editedTask = { ...task };
    },
    confirmDelete(task) {
      this.taskToDelete = task;
    },
    async toggleStatus(task) {
      task.status = task.status ? 0 : 1;
      await apiClient.put(`/tasks/${task.id}`, task);
    },
    clearNotification() {
      this.notification = "";
    },
    formatDate(date) {
      return moment(date).format("MMM D YYYY, h:mm a"); // e.g., February 10th 2024, 3:30:00 pm
    },
  },
  async mounted() {
    await this.fetchTasks();
  },
};
</script>
      
  <style>
.text-decoration-line-through {
  text-decoration: line-through;
}
.task-list-container {
  max-height: 500px; /* Adjust the height as needed */
  overflow-y: auto;
}
</style>
      