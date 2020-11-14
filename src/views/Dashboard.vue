<template>
  <v-container fluid class="d-flex">
    <v-card v-if="items" class="d-flex">
      <v-card>
        <v-card-title>Resources</v-card-title>
        <v-card-subtitle>Used in advanced production</v-card-subtitle>
        <v-card class="ma-2" v-for="clickable in resourceItems" v-bind:key="clickable.label">
          <v-card v-if="meetItemRequirements(clickable)" style="user-select: none;" dark v-bind:color="clickable.color" @click="increment(clickable)">
            <v-card-title primary-title> {{ clickable.label }} : {{ clickable.value }} </v-card-title>
          </v-card>
        </v-card>
      </v-card>
      <v-card>
        <v-card-title>Your Village</v-card-title>
        <v-card-subtitle>Automates resource production</v-card-subtitle>
        <v-tooltip color="black" right v-for="villageItem in villageItems" v-bind:key="villageItem.label">
          <template v-slot:activator="{ on, attrs }">
            <v-card v-bind="attrs" v-on="on" class="ma-2" style="user-select: none;" v-bind:color="villageItem.color" @click="increment(villageItem)">
              <v-card-title primary-title> {{ villageItem.label }} : {{ villageItem.value }} </v-card-title>
            </v-card>
          </template>
          <h3>Costs:</h3>
          <h4 v-for="(cost, name) in villageItem.cost" v-bind:key="villageItem.label + name" v-bind:class="canAfford(cost, name) ? '' : 'red--text'">
            <span v-if="cost > 0">{{ name }}: {{ cost }}</span>
          </h4>
        </v-tooltip>
      </v-card>
    </v-card>

    <v-card v-if="equipments">
      <v-card-title>Your Equipment</v-card-title>
      <v-card-subtitle>Improves manual gathering</v-card-subtitle>
      <v-card class="ma-2" v-for="equipment in equipments" v-bind:key="equipment.label">
        <v-tooltip v-if="meetEquipmentRequirements(equipment)" color="black" right>
          <template v-slot:activator="{ on, attrs }">
            <v-card
              dark
              v-bind="attrs"
              v-on="on"
              style="user-select: none;"
              width="200"
              v-bind:color="equipment.crafted == true ? 'green' : canIncrement(equipment.cost) ? equipment.color : 'grey lighten-2'"
              v-bind:disabled="equipment.crafted"
              @click="craftEquipment(equipment)"
            >
              <v-card-title primary-title> {{ equipment.label }}</v-card-title>
            </v-card>
          </template>
          <h3>Costs:</h3>
          <h4 v-for="(cost, name) in equipment.cost" v-bind:key="equipment.label + name" v-bind:class="canAfford(cost, name) ? '' : 'red--text'">
            <span v-if="cost > 0">{{ name }}: {{ cost }}</span>
          </h4>
        </v-tooltip>
      </v-card>
    </v-card>

    <v-btn class="ml-2" @click="updateLocalStorage">Save</v-btn>
    <v-btn class="ml-2" @click="clearLocalStorage">reset</v-btn>
  </v-container>
</template>

<script>
import itemsJson from "../items.json";
import equipmentJson from "../equipment.json";
export default {
  data() {
    return {
      items: itemsJson.items,
      equipments: equipmentJson.equipment
    };
  },
  mounted() {
    //fetch local storage
    if (localStorage.items) this.items = JSON.parse(localStorage.items);
    if (localStorage.equipments) this.equipments = JSON.parse(localStorage.equipments);

    //Starting Gameloop
    setInterval(this.gameLoop, 1000);
  },
  computed: {
    resourceItems() {
      return this.items.filter(i => {
        if (i.type == "resource") {
          return i;
        }
      });
    },
    villageItems() {
      return this.items.filter(i => {
        if (i.type == "village") {
          return i;
        }
      });
    }
  },
  methods: {
    gameLoop() {
      this.villageItems.forEach(item => {
        if (item.value > 0) {
          if (item.label == "Farm") {
            var food = this.items.find(i => i.label.toLowerCase() == "food");
            food.value += 1 * item.value;
          }
          if (item.label == "Sawmill") {
            var wood = this.items.find(i => i.label.toLowerCase() == "wood");
            wood.value += 1 * item.value;
          }
          if (item.label == "Mines") {
            var stone = this.items.find(i => i.label.toLowerCase() == "stone");
            var coal = this.items.find(i => i.label.toLowerCase() == "coal");
            var iron = this.items.find(i => i.label.toLowerCase() == "iron");
            var gold = this.items.find(i => i.label.toLowerCase() == "gold");
            stone.value += 1 * item.value;
            if (Math.random() > 0.5) coal.value += 1 * item.value;
            if (Math.random() > 0.7) iron.value += 1 * item.value;
            if (Math.random() > 0.95) gold.value += 1 * item.value;
          }
        }
      });
    },
    updateLocalStorage() {
      localStorage.items = JSON.stringify(this.items);
      localStorage.equipments = JSON.stringify(this.equipments);
    },
    clearLocalStorage() {
      localStorage.removeItem("items");
      localStorage.removeItem("equipments");

      location.reload();
    },
    increment(item) {
      if (item.cost && this.canIncrement(item.cost)) {
        this.applyCosts(item.cost);
        item.value += item.incrementAmmount * item.multiplier;
      } else if (!item.cost) {
        item.value += item.incrementAmmount * item.multiplier;
      }
    },
    craftEquipment(equipment) {
      console.log(this.canIncrement(equipment.cost));
      if (equipment.cost && this.canIncrement(equipment.cost) && equipment.crafted == false) {
        if (equipment.upgradeEffect == "wood") {
          this.items.find(i => i.label == "wood").multiplier += equipment.ammount;
          this.$set(equipment, "crafted", true);
          this.applyCosts(equipment.cost);
        }
        if (equipment.upgradeEffect == "mining") {
          this.items.find(i => i.label == "stone").multiplier += equipment.ammount;
          this.items.find(i => i.label == "coal").multiplier += equipment.ammount;
          this.items.find(i => i.label == "iron").multiplier += equipment.ammount;
          this.items.find(i => i.label == "gold").multiplier += equipment.ammount;
          this.$set(equipment, "crafted", true);
          this.applyCosts(equipment.cost);
        }
      }
    },
    meetEquipmentRequirements(equipment) {
      if (equipment.requirements == null) return true;
      else if (equipment.requirements.some(r => r == "smith") && this.items.find(i => i.label == "Smith").value == 0) return false;
      else if (equipment.requirements.some(r => r == "smeltery") && this.items.find(i => i.label == "Smeltery").value == 0) return false;
      else return true;
    },
    meetItemRequirements(item) {
      if (item.requirement == null) return true;
      else if (item.requirement == "Smeltery" && this.items.find(i => i.label == "Smeltery").value > 0) return true;
      else return false;
    },
    canIncrement(costs) {
      return (
        Object.keys(costs).filter(name => {
          if (!this.canAfford(costs[name], name)) {
            return name;
          }
        }).length == 0
      );
    },
    canAfford(cost, name) {
      var affordance =
        this.items.find(c => {
          if (c.label.toLowerCase() == name.toLowerCase()) {
            return c;
          }
        }).value >= cost;
      return affordance;
    },
    applyCosts(costs) {
      Object.keys(costs).filter(name => {
        this.applyCost(costs[name], name);
      });
    },
    applyCost(cost, name) {
      this.items.find(c => {
        if (c.label.toLowerCase() == name.toLowerCase()) {
          c.value -= cost;
        }
      });
    }
  }
};
</script>

<style></style>
