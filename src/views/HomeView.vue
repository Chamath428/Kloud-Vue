<script setup>
import { ref, computed, onMounted } from 'vue'

const comments = ref([])
const currentPage = ref(1)
const itemsPerPage = 10
const searchQuery = ref('')
const sortKey = ref('id')
const sortOrder = ref('asc')

const fetchComments = async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/comments')
  comments.value = await response.json()
}

onMounted(fetchComments)

const sortedComments = computed(() => {
  return [...comments.value].sort((a, b) => {
    const result = a[sortKey.value] > b[sortKey.value] ? 1 : -1
    return sortOrder.value === 'asc' ? result : -result
  })
})

const filteredComments = computed(() => {
  return sortedComments.value.filter(comment =>
    comment.name.includes(searchQuery.value) ||
    comment.email.includes(searchQuery.value) ||
    comment.body.includes(searchQuery.value)
  )
})

const paginatedComments = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return filteredComments.value.slice(start, end)
})

const totalPages = computed(() => {
  return Math.ceil(filteredComments.value.length / itemsPerPage)
})

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--
  }
}

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++
  }
}

const sortTable = (key) => {
  if (sortKey.value === key) {
    sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortKey.value = key
    sortOrder.value = 'asc'
  }
}
</script>

<template>
  <div class="main-div">
    <h1>Data Area</h1>

    <div class="input-area">
      <input type="text" class="search-bar" placeholder="Search..." v-model="searchQuery">
    </div>

    <div class="table-area">
      <table class="data-table">
        <thead>
          <tr>
            <th style="width: 5%;">Post Id</th>
            <th style="width: 15%;">ID <span style="margin-left: 15px;"><button @click="sortTable('id')" >Sort</button></span></th>
            <th style="width: 20%;">Name</th>
            <th style="width: 20%;">Email <span style="margin-left: 15px;"><button @click="sortTable('email')" >Sort</button></span></th>
            <th style="width: 40%;">Body</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="comment in paginatedComments" :key="comment.id">
            <td>{{ comment.postId }}</td>
            <td>{{ comment.id }}</td>
            <td>{{ comment.name }}</td>
            <td>{{ comment.email }}</td>
            <td>{{ comment.body }}</td>
          </tr>
        </tbody>
      </table>
      <div v-if="totalPages > 1" class="pagination-area">
        <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
        <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
      </div>
    </div>

  </div>
</template>


<style scoped>
.main-div {
  width: 100%;
  min-height: 100vh;
  padding: 2rem;
}

.input-area {
  margin-top: 2rem;
}

.search-bar {
  padding: 8px;
  border-radius: 4px;
}

.table-area {
  margin-top: 2rem;
  border: 1px solid white;
}

.data-table {
  width: 100%;
}

.data-table thead tr th {
  border-right: 1px solid white;
  border-bottom: 1px solid white;
}

.data-table tbody tr {
  background-color: #c7f2d2;
}

td {
  background-color: #04AA6D;
  color: white;
  text-align: center
}

.pagination-area{
  float: right;
}
</style>