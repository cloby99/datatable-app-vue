<template>
  <div>
    <h1>Comments DataTable</h1>
    <input v-model="searchQuery" @input="debouncedSearch" placeholder="Search...">
    <select v-model="rowsPerPage" @change="paginate">
      <option v-for="option in rowsPerPageOptions" :key="option" :value="option">{{ option }}</option>
    </select>
    <table>
      <thead>
        <tr>
          <th @click="sortBy('id')">ID</th>
          <th>Name</th>
          <th @click="sortBy('email')">Email</th>
          <th>Body</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="comment in paginatedData" :key="comment.id">
          <td>{{ comment.id }}</td>
          <td>{{ comment.name }}</td>
          <td>{{ comment.email }}</td>
          <td>{{ comment.body }}</td>
          <td>
            <button @click="editComment(comment)">Edit</button>
            <button @click="removeComment(comment.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
    <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
    <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import debounce from 'lodash.debounce';

export default {
  setup() {
    const comments = ref([]);
    const searchQuery = ref('');
    const rowsPerPage = ref(10);
    const rowsPerPageOptions = [10, 15, 20, 'all'];
    const currentPage = ref(1);
    const totalPages = computed(() => {
      return rowsPerPage.value === 'all' ? 1 : Math.ceil(filteredComments.value.length / rowsPerPage.value);
    });

    const fetchComments = async () => {
      try {
        const response = await axios.get('https://jsonplaceholder.typicode.com/comments');
        comments.value = response.data;
      } catch (error) {
        console.error(error);
      }
    };

    const paginatedData = computed(() => {
      const start = (currentPage.value - 1) * rowsPerPage.value;
      const end = start + parseInt(rowsPerPage.value);
      return rowsPerPage.value === 'all' ? filteredComments.value : filteredComments.value.slice(start, end);
    });

    const filteredComments = computed(() => {
      return comments.value.filter(comment =>
        comment.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        comment.email.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        comment.body.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    });

    const debouncedSearch = debounce(() => {
      paginate();
    }, 300);

    const sortBy = (key) => {
      comments.value.sort((a, b) => {
        return a[key] > b[key] ? 1 : -1;
      });
    };

    const paginate = () => {
      currentPage.value = 1;
    };

    const prevPage = () => {
      if (currentPage.value > 1) currentPage.value--;
    };

    const nextPage = () => {
      if (currentPage.value < totalPages.value) currentPage.value++;
    };

    const editComment = (comment) => {
      alert(`Edit comment ID: ${comment.id}`);
    };

    const removeComment = (id) => {
      comments.value = comments.value.filter(comment => comment.id !== id);
    };

    onMounted(() => {
      fetchComments();
    });

    return {
      searchQuery,
      rowsPerPage,
      rowsPerPageOptions,
      currentPage,
      totalPages,
      paginatedData,
      debouncedSearch,
      sortBy,
      prevPage,
      nextPage,
      editComment,
      removeComment
    };
  }
};
</script>

<style>
table {
  width: 100%;
  border-collapse: collapse;
}
th, td {
  border: 1px solid #ddd;
  padding: 8px;
}
th {
  cursor: pointer;
}
button {
  margin: 5px;
}
</style>
