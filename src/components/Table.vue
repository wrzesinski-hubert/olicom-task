<template>
  <div class="hello">
    <table style="border-collapse: collapse" border="1">
      <tr>
        <th @click="sortBy('name')">Name</th>
        <th @click="sortBy('email')">Email</th>
        <th @click="sortBy('company.name')">Company.name</th>
        <th @click="sortBy('address.city')">Address.city</th>
        <th @click="sortBy('website')">Website</th>
      </tr>
      <tr v-for="c in sortedUsers" v-bind:key="c.id">
        <td>{{ c.name }}</td>
        <td>
          <a :href="`mailto:` + c.email">{{ c.email }}</a>
        </td>
        <td>{{ c.company.name }}</td>
        <td>{{ c.address.city }}</td>
        <td>{{ c.website }}</td>
      </tr>
    </table>

    <button @click="prevPage()">xdd</button>
    <button @click="nextPage()">xd</button>

    {{ sorting }}
    <button
      @click="
        () => {
          sorting = !sorting;
        }
      "
    >
      xd
    </button>
    <br />
    <input type="text" v-model="searchQuery" placeholder="Search" />
    {{ searchQuery }}
  </div>
</template>

<script>
export default {
  name: "Table",
  props: {
    endpoint: String,
    sorting: Boolean,
  },
  data() {
    return {
      users: [],
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
    filteredResults() {
      if (this.searchQuery) {
        return this.users.filter((item) => {
          const array = [
            item.name.toString().toLowerCase(),
            item.email.toString().toLowerCase(),
            item.company.name.toString().toLowerCase(),
            item.address.city.toString().toLowerCase(),
            item.website.toString().toLowerCase(),
          ];
          return array.some((r) => r.includes(this.searchQuery.toLowerCase()));
        });
      } else {
        return this.users;
      }
    },
    sortedUsers() {
      return this.filteredResults
        .slice(0)
        .sort((a, b) => {
          let modifier = 1;
          if (this.currentSortDir === "desc") modifier = -1;
          if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
          if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
          return 0;
        })
        .filter((row, index) => {
          let start = (this.currentPage - 1) * this.pageSize;
          let end = this.currentPage * this.pageSize;
          if (index >= start && index < end) return true;
        });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
