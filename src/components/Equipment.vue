<template>
  <v-card dark class="pa-6" elevation="12">
    <v-card-title>Blacksmith</v-card-title>
    <v-card-subtitle>Create weapons and equipment that improves gathering and combat</v-card-subtitle>
    <v-flex class="d-flex flex-wrap">
      <v-tooltip color="black" right v-for="item in ITEMS" v-bind:key="item.label">
        <template v-slot:activator="{on, attrs}">
          <v-card
            width="60px"
            height="60px"
            v-bind="attrs"
            v-on="on"
            class="ma-2"
            style="user-select: none;"
            v-bind:color="item.crafted == true ? 'green' : item.color"
            v-bind:style="item.crafted ? 'opacity: 1;' : !INCREMENTABLITY(item.cost) ? 'opacity: 1;' : 'opacity: 0.5'"
            @click="itemClicked(item)"
          >
            <v-img v-if="item.tier == 'normal'" src="../assets/backgrounds/bg_grey.png">
              <v-img v-bind:src="require('../assets/Equipment/' + item.icon)"></v-img>
            </v-img>
            <v-img v-if="item.tier == 'good'" src="../assets/backgrounds/bg_green.png">
              <v-img v-bind:src="require('../assets/Equipment/' + item.icon)"></v-img>
            </v-img>
            <v-img v-if="item.tier == 'rare'" src="../assets/backgrounds/bg_blue.png">
              <v-img v-bind:src="require('../assets/Equipment/' + item.icon)"></v-img>
            </v-img>
          </v-card>
        </template>
        <h3 class="mb-2">{{ item.label }}</h3>
        <div>{{ item.upgradeEffect }} gathering : {{ item.ammount }}x</div>
        <h3 class="mt-2">Costs:</h3>
        <div v-for="(cost, name) in item.cost" v-bind:key="item.label + name" v-bind:class="AFFORDANCE(cost, name) ? '' : 'red--text'">
          <span v-if="cost > 0">{{ name }}: {{ cost }}</span>
        </div>
      </v-tooltip>
    </v-flex>
  </v-card>
</template>

<script>
export default {
  props: ["ITEMS", "AFFORDANCE", "INCREMENTABLITY"],
  methods: {
    itemClicked(item) {
      this.$emit("item-clicked", item);
    }
  }
};
</script>

<style></style>
