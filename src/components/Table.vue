<template>
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
            class="header"
            v-for="title in getTitles"
            :key="title"
            @click="sortBy(title)"
          >
            {{ title }}
            <span v-if="currentSort === title && currentSortDir === 'asc'"
              >⇓</span
            >
            <span v-else-if="currentSort === title && currentSortDir === 'desc'"
              >⇑</span
            >
          </th>
        </tr>
        <tr v-for="userObject in sortedUsers" :key="userObject.name">
          <td v-for="user in userObject" :key="user.name">
            <div v-if="user.includes(`@`)">
              <a :href="`mailto:` + user">{{ user }}</a>
            </div>
            <div v-else>{{ user }}</div>
          </td>
        </tr>
      </table>
    </div>

    <span v-if="pagination">
      <button type="button" class="btn btn-primary" @click="prevPage()">
        Prev
      </button>
      <span class="page">
        {{ currentPage }}
      </span>
      <button type="button" class="btn btn-primary" @click="nextPage()">
        Next
      </button>
    </span>
  </span>
</template>

<script>
export default {
  name: "Table",
  props: {
    endpoint: String,
    sorting: Boolean,
    pagination: Boolean,
    searching: Boolean,
  },
  data() {
    return {
      users: [],
      users2: [],
      titles: [],
      currentSort: "name",
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
        }
        this.currentSort = prop;
      }
    },
    nextPage() {
      if (this.currentPage * this.pageSize < this.filteredResults.length)
        this.currentPage++;
    },
    prevPage() {
      if (this.currentPage > 1) this.currentPage--;
    },
  },
  watch: {
    searchQuery() {
      this.currentPage = 1;
    },
  },
  computed: {
    userToTable() {
      this.users.map(
        ({
          name,
          email,
          company: { name: companyName },
          address: { city: addressCity },
          website,
        }) =>
          this.users2.push({ name, email, companyName, addressCity, website })
      );
      return this.users2;
    },
    getTitles() {
      this.users2.map((item) => {
        this.titles.push(Object.keys(item));
      });
      return this.titles[0];
    },

    filteredResults() {
      if (this.searchQuery && this.searching) {
        return this.userToTable.filter((item) => {
          var array = Object.keys(item).map((key) => {
            return item[key].toString().toLowerCase();
          });
          return array.some((r) => r.includes(this.searchQuery.toLowerCase()));
        });
      } else {
        return this.userToTable;
      }
    },
    sortedUsers() {
      const sorted = this.filteredResults.slice(0).sort((a, b) => {
        let modifier = 1;
        if (this.currentSortDir === "desc") modifier = -1;
        if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
        if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
        return 0;
      });

      const paginated = sorted.filter((row, index) => {
        let start = (this.currentPage - 1) * this.pageSize;
        let end = this.currentPage * this.pageSize;
        if (index >= start && index < end) return true;
      });
      if (this.pagination) {
        return paginated;
      } else {
        return sorted;
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.input-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 10px;
}
.header {
  cursor: pointer;
}
.page {
  color: white;
  margin: 10px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
