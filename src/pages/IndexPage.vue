<template>
  <q-page padding>
    <div class="q-mb-md">
      <q-input v-model="taskName" label="Task Name" filled />
    </div>

    <q-btn @click="addOrEditTask" :label="currentTask ? 'Update Task' : 'Add Task'" color="primary" />

    <q-list bordered v-if="tasks.length">
      <q-item v-for="task in tasks" :key="task.id">
        <q-item-section>
          <q-item-label>{{ task.title }}</q-item-label>
        </q-item-section>
        <q-item-section>
          <q-btn @click="editTask(task)" icon="edit" flat />
          <q-btn @click="deleteTask(task.id)" icon="delete" flat />
        </q-item-section>
      </q-item>
    </q-list>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import axios from "axios";

export default defineComponent({
  name: "IndexPage",
  setup() {
    const taskName = ref("");
    const tasks = ref([]);
    const currentTask = ref(null);  // To store the task being edited

    const getTasks = async () => {
      try {
        const response = await axios.get("https://jsonplaceholder.typicode.com/todos");
        tasks.value = response.data;
      } catch (error) {
        console.error(error);
      }
    };

    // Function to add or edit a task
    const addOrEditTask = async () => {
      if (!taskName.value) return;

      const taskData = {
        title: taskName.value,
        completed: false,
      };

      if (currentTask.value) {
        // Editing an existing task
        try {
          const updatedTask = { ...currentTask.value, title: taskName.value };
          await axios.put(`https://jsonplaceholder.typicode.com/todos/${currentTask.value.id}`, updatedTask);
          const index = tasks.value.findIndex(t => t.id === currentTask.value.id);
          tasks.value[index] = updatedTask;
          currentTask.value = null;  // Clear the current task after editing
          taskName.value = "";  // Clear the input
        } catch (error) {
          console.error(error);
        }
      } else {
        // Adding a new task
        try {
          const response = await axios.post("https://jsonplaceholder.typicode.com/todos", taskData);
          tasks.value.push(response.data);
          taskName.value = "";  // Clear the input
        } catch (error) {
          console.error(error);
        }
      }
    };

    const editTask = (task) => {
      currentTask.value = task;
      taskName.value = task.title;  // Set the input field to the task title for editing
    };

    const deleteTask = async (taskId) => {
      try {
        await axios.delete(`https://jsonplaceholder.typicode.com/todos/${taskId}`);
        tasks.value = tasks.value.filter(task => task.id !== taskId);
      } catch (error) {
        console.error(error);
      }
    };

    // Fetch tasks on component mount
    getTasks();

    return {
      taskName,
      tasks,
      addOrEditTask,
      editTask,
      deleteTask,
      currentTask,
    };
  },
});
</script>

<style scoped>
.q-btn {
  margin-top: 20px;
}
</style>
