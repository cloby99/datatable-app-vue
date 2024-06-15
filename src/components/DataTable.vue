<template>
  <div class="datatable-page">
    <header>
      <h2>Task 01 - Vue</h2>
    </header>
    <div class="content">
      <div class="search-bar">
        <input
          v-model="searchQuery"
          @input="debouncedSearch"
          placeholder="Search..."
        />

        <div class="sorting-buttons">
          <button @click="sortBy('id')">Sort by Id</button>
          <button @click="sortBy('email')">Sort by Email</button>
        </div>
      </div>

      <table id="comments">
        <thead>
          <tr>
            <th>ID</th>
            <th>Post ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Body</th>
            <th class="actions">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="comment in paginatedData" :key="comment.id">
            <td>{{ comment.id }}</td>
            <td>{{ comment.postId }}</td>
            <td>{{ comment.name }}</td>
            <td>{{ comment.email }}</td>
            <td>{{ comment.body }}</td>
            <td class="action-buttons">
              <button @click="editComment(comment)">Edit</button>
              <button @click="removeComment(comment.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
      <div>
        <button @click="prevPage" :disabled="currentPage === 1">
          Previous
        </button>
        <button @click="nextPage" :disabled="currentPage === totalPages">
          Next
        </button>
      </div>
      <div>
        <label for="">
          Items per page:
          <select v-model="rowsPerPage" @change="paginate">
            <option
              v-for="option in rowsPerPageOptions"
              :key="option"
              :value="option"
            >
              {{ option }}
            </option>
          </select>
        </label>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import debounce from "lodash.debounce";

export default {
  setup() {
    const comments = ref([]);
    const searchQuery = ref("");
    const rowsPerPage = ref(10);
    const rowsPerPageOptions = [10, 15, 20, "all"];
    const currentPage = ref(1);
    const totalPages = computed(() => {
      return rowsPerPage.value === "all"
        ? 1
        : Math.ceil(filteredComments.value.length / rowsPerPage.value);
    });

    const fetchComments = async () => {
      try {
        const response = await axios.get(
          "https://jsonplaceholder.typicode.com/comments"
        );
        comments.value = response.data;
      } catch (error) {
        console.error(error);
      }
    };

    // Function for implement Pagination

    const paginatedData = computed(() => {
      const start = (currentPage.value - 1) * rowsPerPage.value;
      const end = start + parseInt(rowsPerPage.value);
      return rowsPerPage.value === "all"
        ? filteredComments.value
        : filteredComments.value.slice(start, end);
    });

    const paginate = () => {
      currentPage.value = 1;
    };

    const prevPage = () => {
      if (currentPage.value > 1) currentPage.value--;
    };

    const nextPage = () => {
      if (currentPage.value < totalPages.value) currentPage.value++;
    };

    // Sorting Function

    const filteredComments = computed(() => {
      return comments.value.filter(
        (comment) =>
          comment.name
            .toLowerCase()
            .includes(searchQuery.value.toLowerCase()) ||
          comment.email
            .toLowerCase()
            .includes(searchQuery.value.toLowerCase()) ||
          comment.body.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    });

    // Function for Search data from the Comments table

    const debouncedSearch = debounce(() => {
      paginate();
    }, 300);

    const sortBy = (key) => {
      comments.value.sort((a, b) => {
        return a[key] > b[key] ? 1 : -1;
      });
    };

    // Edit Table Data - ToDo

    const editComment = (comment) => {
      alert(`Edit comment ID: ${comment.id}`);
    };

    // Remove Table Data

    const removeComment = (id) => {
      comments.value = comments.value.filter((comment) => comment.id !== id);
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
      removeComment,
    };
  },
};
</script>

<style>
.datatable-page {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  margin: 0;
}

header {
  background-color: #293241;
  color: white;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0.6rem;
}

.content {
  margin: 0.5rem 5rem 1rem 5rem;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  gap: 1rem;
}

.search-bar {
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

.search-bar > input {
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  width: 25%;
}

.sorting-buttons {
  display: flex;
  flex-direction: row;
  gap: 1rem;
}

.sorting-buttons > button {
  padding: 0.5rem 1rem;
  background-color: #ee6c4d;
  font-size: medium;
  font-weight: 700;
  border: #ee6c4d;
  border-radius: 0.5rem;
  opacity: 0.8;
}

.sorting-buttons > button:hover {
  opacity: 1;
}

#comments td,
#comments th {
  padding: 0.2rem 0.8rem;
}

#comments tr:nth-child(even) {
  background-color: #f2f2f2;
}

#comments tr:hover {
  background-color: #e0fbfc;
}

#comments th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #3d5a80;
  color: white;
}

.action-buttons {
  display: flex;
  flex-direction: row;
  gap: 0.2rem;
}

.action-buttons {
  padding: 0.2rem 0.5rem;
}

.edit-comments {
  padding: 0.5rem 1rem;
}
</style>
