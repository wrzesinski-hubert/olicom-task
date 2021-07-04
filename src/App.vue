<template>
  <div id="app">
    <Table
      endpoint="https://jsonplaceholder.typicode.com/users"
      :sorting="true"
      :pagination="true"
      :searching="true"
      :columns="['name', 'email', 'company.name', 'website', 'address.city']"
    />
  </div>
</template>

<script>
import Table from "./components/Table.vue";

export default {
  name: "App",
  components: {
    Table,
  },
};
</script>

<style>
body {
  background-color: #212529;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: white;
  margin-top: 30px;
  height: calc(100vh - 30px);
  display: flex;
  flex-direction: column;
}
</style>
