<template>
  <v-app>
    <Dashboard></Dashboard>
  </v-app>
</template>

<script>
import Dashboard from './views/Dashboard'
export default {
  name: 'App',

  components: {
    Dashboard
  },

  data: () => ({
    //
  }),
};
</script>
