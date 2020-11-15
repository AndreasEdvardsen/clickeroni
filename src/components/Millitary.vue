<template>
  <v-card dark class="pa-6" elevation="12">
    <v-card-title>Barracks</v-card-title>
    <v-card-subtitle>
      <div>Hire and train soldiers to protect your village from attacks</div>
    </v-card-subtitle>
    <v-flex class="d-flex">
      <v-tooltip color="black" right v-for="item in ITEMS" v-bind:key="item.label">
        <template v-slot:activator="{on, attrs}">
          <v-card height="60px" width="60px" v-bind="attrs" v-on="on" class="ma-2" style="user-select: none;" @click="itemClicked(item)">
            <v-img v-if="item.tier == 'normal'" src="../assets/backgrounds/bg_grey.png">
              <v-img v-bind:src="require('../assets/Millitary/' + item.label + '.png')">{{ item.value }}</v-img>
            </v-img>
            <v-img v-if="item.tier == 'good'" src="../assets/backgrounds/bg_green.png">
              <v-img v-bind:src="require('../assets/Millitary/' + item.label + '.png')">{{ item.value }}</v-img>
            </v-img>
            <v-img v-if="item.tier == 'rare'" src="../assets/backgrounds/bg_blue.png">
              <v-img v-bind:src="require('../assets/Millitary/' + item.label + '.png')">{{ item.value }}</v-img>
            </v-img>
            <v-img v-if="item.tier == 'epic'" src="../assets/backgrounds/bg_red.png">
              <v-img v-bind:src="require('../assets/Millitary/' + item.label + '.png')">{{ item.value }}</v-img>
            </v-img>
            <v-img v-if="item.tier == 'legendary'" src="../assets/backgrounds/bg_purple.png">
              <v-img v-bind:src="require('../assets/Millitary/' + item.label + '.png')">{{ item.value }}</v-img>
            </v-img>
          </v-card>
        </template>
        <h3 class="mb-2">{{ item.label }}</h3>
        <div>Damage: {{ item.dps }} Defense: {{ item.defense }}</div>
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
  props: ["ITEMS", "AFFORDANCE", "LODGING"],
  methods: {
    itemClicked(item) {
      this.$emit("item-clicked", item);
    }
  }
};
</script>

<style></style>
