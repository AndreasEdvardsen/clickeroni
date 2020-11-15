<template>
  <v-flex class="d-inline-flex flex-shrink-1">
    <v-btn
      text
      class="ma-2"
      v-for="resource in RESOURCES"
      v-bind:key="resource.label"
      style="user-select: none;"
      dark
      v-bind:color="resource.color"
      @click="resourceClicked(resource)"
      :disabled="resource.disabled"
    >
      <v-img height="40px" width="40px" contain v-bind:src="require('../assets/Resources/' + resource.label + '.png')"></v-img>
      {{ resource.label }} : {{ resource.value }}
    </v-btn>
  </v-flex>
</template>

<script>
export default {
  props: ["RESOURCES"],
  methods: {
    resourceClicked(resource) {
      this.$emit("resource-clicked", resource);
    }
  }
};
</script>

<style></style>
