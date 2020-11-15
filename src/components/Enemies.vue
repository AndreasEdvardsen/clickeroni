<template>
  <v-card height="100%" elevation="0">
    <v-card-title>War</v-card-title>
    <v-card-subtitle>Protect your village against enemies</v-card-subtitle>
    <v-card class="ma-2" v-for="(item, index) in ITEMS" v-bind:key="item.label + index">
      <v-card class="d-flex" style="user-select: none;" dark v-bind:color="item.color" @click="itemClicked(item)" v-bind:disabled="HITPOINTS < 10">
        <v-card flat width="110px">
          <v-img v-if="item.tier == 'normal'" src="../assets/backgrounds/bg_grey.png">
            <v-img v-bind:src="require('../assets/Enemies/' + item.label + '.png')">{{ item.value }}</v-img>
          </v-img>
          <v-img v-if="item.tier == 'good'" src="../assets/backgrounds/bg_green.png">
            <v-img v-bind:src="require('../assets/Enemies/' + item.label + '.png')">{{ item.value }}</v-img>
          </v-img>
          <v-img v-if="item.tier == 'rare'" src="../assets/backgrounds/bg_blue.png">
            <v-img v-bind:src="require('../assets/Enemies/' + item.label + '.png')">{{ item.value }}</v-img>
          </v-img>
          <v-img v-if="item.tier == 'epic'" src="../assets/backgrounds/bg_red.png">
            <v-img v-bind:src="require('../assets/Enemies/' + item.label + '.png')">{{ item.value }}</v-img>
          </v-img>
          <v-img v-if="item.tier == 'legendary'" src="../assets/backgrounds/bg_purple.png">
            <v-img v-bind:src="require('../assets/Enemies/' + item.label + '.png')">{{ item.value }}</v-img>
          </v-img>
        </v-card>
        <div>
          <v-card-title primary-title> {{ item.label }}</v-card-title>
          <v-card-subtitle>
            <v-card height="10px" width="100px" color="red" class="mb-2"
              ><v-card :width="healthbarWidth(item)" height="10px" color="green"> </v-card
            ></v-card>
            <div>Damage: {{ item.damage }}</div>
          </v-card-subtitle>
        </div>
      </v-card>
    </v-card>
  </v-card>
</template>

<script>
export default {
  props: ["ITEMS", "HITPOINTS"],
  methods: {
    itemClicked(item) {
      this.$emit("item-clicked", item);
    },
    healthbarWidth(enemy) {
      return (enemy.hitpoints / enemy.maxHitpoints) * 100 + "%";
    }
  }
};
</script>

<style></style>
