<template>
  <v-card height="100%">
    <v-flex class="d-flex flex-wrap">
      <v-tooltip color="black" right v-for="item in ITEMS" v-bind:key="item.label">
        <template v-slot:activator="{on, attrs}">
          <v-card v-bind="attrs" v-on="on" class="ma-2" style="user-select: none;" v-bind:color="item.color" @click="itemClicked(item)">
            <v-card flat width="80px">
              <v-img v-if="item.tier == 'normal'" src="../assets/backgrounds/bg_grey.png">
                <v-img v-bind:src="require('../assets/Village/' + item.label + '.png')">
                  <H2 style="background: rgba(0,0,0,0.4);">{{ item.value }}</H2></v-img
                >
              </v-img>
              <v-img v-if="item.tier == 'good'" src="../assets/backgrounds/bg_green.png">
                <v-img v-bind:src="require('../assets/Village/' + item.label + '.png')"
                  ><H2 style="background: rgba(0,0,0,0.4);">{{ item.value }}</H2></v-img
                >
              </v-img>
              <v-img v-if="item.tier == 'rare'" src="../assets/backgrounds/bg_blue.png">
                <v-img v-bind:src="require('../assets/Village/' + item.label + '.png')"
                  ><H2 style="background: rgba(0,0,0,0.4);">{{ item.value }}</H2></v-img
                >
              </v-img>
              <v-img v-if="item.tier == 'epic'" src="../assets/backgrounds/bg_red.png">
                <v-img v-bind:src="require('../assets/Village/' + item.label + '.png')"
                  ><H2 style="background: rgba(0,0,0,0.4);">{{ item.value }}</H2></v-img
                >
              </v-img>
              <v-img v-if="item.tier == 'legendary'" src="../assets/backgrounds/bg_purple.png">
                <v-img v-bind:src="require('../assets/Village/' + item.label + '.png')"
                  ><H2 style="background: rgba(0,0,0,0.4);">{{ item.value }}</H2></v-img
                >
              </v-img>
            </v-card>
          </v-card>
        </template>
        <h3 class="mb-2">{{ item.label }}</h3>
        <div>{{ item.description }}</div>
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
  props: ["ITEMS", "AFFORDANCE"],
  methods: {
    itemClicked(item) {
      this.$emit("item-clicked", item);
    }
  }
};
</script>

<style></style>
