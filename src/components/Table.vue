<template>
  <div class="wrap-all">
    <span>
      <div class="input-wrapper" v-if="searching">
        <input
          class="form-control mr-sm-2"
          type="search"
          placeholder="Search"
          aria-label="Search"
          v-model="searchQuery"
        />
      </div>
      <div class="table-responsive">
        <table class="table table-dark">
          <tr>
            <th
              class="table-header"
              v-for="title in columns"
              :key="title"
              @click="sortBy(columns.indexOf(title))"
            >
              {{ title }}
              <span
                v-if="
                  currentSort === columns.indexOf(title) &&
                    currentSortDir === 'asc'
                "
                >⇓</span
              >
              <span
                v-else-if="
                  currentSort === columns.indexOf(title) &&
                    currentSortDir === 'desc'
                "
                >⇑</span
              >
            </th>
          </tr>
          <tr v-for="userObject in paginatedUsers" :key="userObject.name">
            <td v-for="user in userObject" :key="user.name">
              <div v-if="user.includes(`@`)">
                <a :href="`mailto:` + user">{{ user }}</a>
              </div>
              <div v-else>{{ user }}</div>
            </td>
          </tr>
        </table>
      </div>
    </span>
    <span v-if="pagination">
      <button type="button" class="btn btn-primary" @click="prevPage()">
        Prev
      </button>
      <span class="page-number">
        {{ currentPage }}
      </span>
      <button type="button" class="btn btn-primary" @click="nextPage()">
        Next
      </button>
    </span>
  </div>
</template>

<script>
import get from "lodash.get";
export default {
  name: "Table",
  props: {
    endpoint: String,
    sorting: Boolean,
    pagination: Boolean,
    searching: Boolean,
    columns: Array,
  },
  data() {
    return {
      users: [],
      currentSort: "",
      currentSortDir: "asc",
      searchQuery: "",
      pageSize: "3",
      currentPage: "1",
    };
  },
  mounted() {
    fetch(this.endpoint)
      .then((res) => res.json())
      .then((data) => (this.users = data));
  },
  methods: {
    sortBy(prop) {
      if (this.sorting) {
        if (prop === this.currentSort) {
          this.currentSortDir = this.currentSortDir === "asc" ? "desc" : "asc";
        } else {
          this.currentSort = prop;
          this.currentSortDir = "asc";
        }
      }
    },
    nextPage() {
      if (this.currentPage * this.pageSize < this.usersInTable.length)
        this.currentPage++;
    },
    prevPage() {
      if (this.currentPage > 1) this.currentPage--;
    },
    prepareUser(user) {
      return this.columns.map((column) => get(user, column));
    },
    isMatchingSearchQuery(user) {
      if (this.searchQuery && this.searching) {
        const normalizedUser = user.map((property) =>
          property.toString().toLowerCase()
        );
        return normalizedUser.some((r) =>
          r.includes(this.searchQuery.toLowerCase())
        );
      } else {
        return true;
      }
    },
    sortingAlgorithm(a, b) {
      let modifier = 1;
      if (this.currentSortDir === "desc") modifier = -1;
      if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
      if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
      return 0;
    },
  },
  watch: {
    searchQuery() {
      this.currentPage = 1;
    },
  },
  computed: {
    usersInTable() {
      return this.users
        .slice(0)
        .map(this.prepareUser)
        .filter(this.isMatchingSearchQuery)
        .sort(this.sortingAlgorithm);
    },
    paginatedUsers() {
      const paginated = this.usersInTable.filter((row, index) => {
        let start = (this.currentPage - 1) * this.pageSize;
        let end = this.currentPage * this.pageSize;
        if (index >= start && index < end) return true;
      });
      if (this.pagination) {
        return paginated;
      } else {
        return this.usersInTable;
      }
    },
  },
};
</script>

<style scoped>
.wrap-all {
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding-bottom: 10px;
}
.input-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 10px;
}
.table-header {
  cursor: pointer;
}
.page-number {
  color: white;
  margin: 10px;
}
a {
  color: #42b983;
}
</style>
