<template>
  <q-page padding>
    <q-card>
      <!-- Header Section -->
      <q-card-section>
        <div class="text-h6">To-Do Application</div>
      </q-card-section>

      <!-- Task Form -->
      <q-card-section>
        <q-form @submit.prevent="isEditing ? updateTask() : addTask()">
          <div class="row q-col-gutter-sm">
            <div class="col">
              <q-input
                v-model="task.title"
                label="Task Name"
                filled
                dense
                required
              />
            </div>
            <div class="col-auto">
              <q-btn
                type="submit"
                :label="isEditing ? 'Update' : 'Add'"
                color="primary"
                icon="save"
              />
              <q-btn
                v-if="isEditing"
                label="Cancel"
                color="secondary"
                flat
                @click="cancelEdit"
              />
            </div>
          </div>
        </q-form>
      </q-card-section>

      <!-- Task List Section -->
      <q-card-section>
        <q-list bordered>
          <q-item
            v-for="todo in paginatedTodos"
            :key="todo.id"
            clickable
            v-ripple
          >
            <q-item-section>
              <div>{{ todo.title }}</div>
            </q-item-section>
            <q-item-section side>
              <div class="row q-col-gutter-sm">
                <div class="col-auto">
                  <q-btn
                    icon="edit"
                    color="primary"
                    flat
                    @click="editTask(todo)"
                  />
                </div>
                <div class="col-auto">
                  <q-btn
                    icon="delete"
                    color="negative"
                    flat
                    @click="deleteTask(todo.id)"
                  />
                </div>
              </div>
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>

      <!-- Pagination Section -->
      <q-card-section>
        <div class="row justify-center">
          <q-btn
            label="Previous"
            :disabled="currentPage === 1"
            color="primary"
            @click="previousPage"
          />
          <q-btn
            label="Next"
            :disabled="currentPage * itemsPerPage >= todos.length"
            color="primary"
            @click="nextPage"
          />
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import { ref, computed, onMounted } from 'vue'  // Make sure computed is imported here
import axios from 'axios'

const API_URL = 'https://jsonplaceholder.typicode.com/todos'

export default {
  name: 'IndexPage',
  setup() {
    const todos = ref([])
    const task = ref({ title: '', id: null })
    const isEditing = ref(false)

    const currentPage = ref(1)  // Current page number
    const itemsPerPage = 10     // Number of tasks to display per page

    // Fetch tasks (READ)
    const fetchTasks = async () => {
      try {
        const res = await axios.get(API_URL, { params: { _limit: 2000 } })
        todos.value = res.data
      } catch (error) {
        console.error('Fetch error:', error)
      }
    }

    let localId = 1000 

    // Add a new task (CREATE)
    const addTask = async () => {
      if (!task.value.title) return

      try {
        const newTask = {
          id: localId++, 
          title: task.value.title,
          completed: false,
        }

        todos.value.unshift(newTask)

        task.value = { title: '', id: null }
      } catch (error) {
        console.error('Add error:', error)
      }
    } 

    // Edit an existing task (SETUP FOR UPDATE)
    const editTask = (todo) => {
      task.value = { ...todo }
      isEditing.value = true
    }

    // Update a task (UPDATE)
    const updateTask = async () => {
      if (!task.value.title || !task.value.id) return

      try {
        let updatedTask = { ...task.value, completed: false }
        if (task.value.id < 1000) {
          const res = await axios.put(`${API_URL}/${task.value.id}`, {
            title: task.value.title,
            completed: false,
          })
          updatedTask = res.data
        }
        const index = todos.value.findIndex((t) => t.id === task.value.id)
        if (index !== -1) {
          todos.value[index] = updatedTask
        }
        task.value = { title: '', id: null }
        isEditing.value = false
      } catch (error) {
        console.error('Update error:', error)
      }
    }

    // Delete a task (DELETE)
    const deleteTask = async (id) => {
      try {
        if (id < 1000) {
          await axios.delete(`${API_URL}/${id}`)
        }

        todos.value = todos.value.filter((t) => t.id !== id)
      } catch (error) {
        console.error('Delete error:', error)
      }
    }

    // Pagination: Get the tasks for the current page
    const paginatedTodos = computed(() => {
      const start = (currentPage.value - 1) * itemsPerPage
      return todos.value.slice(start, start + itemsPerPage)
    })

    // Pagination: Go to the next page
    const nextPage = () => {
      if (currentPage.value * itemsPerPage < todos.value.length) {
        currentPage.value++
      }
    }

    // Pagination: Go to the previous page
    const previousPage = () => {
      if (currentPage.value > 1) {
        currentPage.value--
      }
    }

    onMounted(fetchTasks)

    return {
      todos,
      task,
      isEditing,
      addTask,
      editTask,
      updateTask,
      deleteTask,
      paginatedTodos,
      currentPage,
      nextPage,
      previousPage,
    }
  },
}
</script>


<style scoped>
.q-mt-sm {
  margin-top: 1rem;
}

.q-ml-sm {
  margin-left: 1rem;
}
</style>
