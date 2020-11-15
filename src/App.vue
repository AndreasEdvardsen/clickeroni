<template>
  <v-app class="app" style="background: rgba(255,255,255,0.1);">
    <v-app-bar dense app dark>
      <resources :RESOURCES="resourceItems" @resource-clicked="increment"></resources>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <v-row>
          <v-col :cols="3">
            <v-card dark>
              <v-card-title>Stats</v-card-title>
              <v-card-subtitle>
                <div class="mt-2">Health: {{ hitpoints }}</div>
                <v-card height="25px" color="red" class="mb-2"><v-card :width="healthbarWidth" height="25px" color="green"> </v-card></v-card>
                <div>Damage: {{ dps }} Defense: {{ defense }}</div>
                <div>Lodging: {{ lodging }}</div>
              </v-card-subtitle>
            </v-card>
            <v-card dark class="mt-4">
              <v-card-title>Quick Guide</v-card-title>
              <v-card-subtitle>
                <div>As the new ruler of a new humble village, your goal is to grow and protect your people</div>
              </v-card-subtitle>
              <v-card-subtitle>
                <ul>
                  <li>Gather resources by clicking on them in the top bar.</li>
                  <li class="mt-3">Build buildings to help your expansion by clicking on icons on the left side.</li>
                  <li class="mt-3">Build a smithy to unlock the possebility for new equipment.</li>
                  <li class="mt-3">
                    Defend against attackers by clicking on them in the right sidebar. but be carefull! They will try to attack back! Training
                    soldiers in the barracks will protect you, and make your attacks stronger.
                  </li>
                </ul>
              </v-card-subtitle>
            </v-card>
          </v-col>
          <v-col>
            <equipment :ITEMS="equipmentItems" :AFFORDANCE="canAfford" :INCREMENTABLITY="canIncrement" @item-clicked="craftEquipment"></equipment>
          </v-col>
          <v-col>
            <millitary
              :ITEMS="millitaryItems"
              :AFFORDANCE="canAfford"
              :INCREMENTABLITY="canIncrement"
              @item-clicked="trainMillitary"
              :LODGING="lodging"
            ></millitary>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <v-navigation-drawer width="300px" dark app right>
      <enemies :ITEMS="enemies" :HITPOINTS="hitpoints" @item-clicked="attackEnemy"></enemies>
    </v-navigation-drawer>

    <v-navigation-drawer width="100px" dark app>
      <villageItems :ITEMS="villageItems" :AFFORDANCE="canAfford" @item-clicked="increment"></villageItems>
    </v-navigation-drawer>

    <v-app-bar dense clipped-right clipped-left app bottom dark>
      <v-btn color="green" class="ml-2" @click="updateLocalStorage">Save</v-btn>
      <v-btn color="red" class="ml-2" @click="clearLocalStorage">reset</v-btn>
    </v-app-bar>
  </v-app>
</template>

<script>
import itemsJson from "./items.json";
import equipmentJson from "./equipment.json";
import enemiesJson from "./enemies.json";

import resources from "./components/Resources";
import villageItems from "./components/Village";
import equipment from "./components/Equipment";
import millitary from "./components/Millitary.vue";
import enemies from "./components/Enemies.vue";
export default {
  components: {
    resources,
    villageItems,
    equipment,
    millitary,
    enemies
  },
  data() {
    return {
      items: itemsJson.items,
      equipments: equipmentJson.equipment,
      enemies: [],
      lodging: 10,
      hitpoints: 100,
      maxEnemies: 5
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
    dps() {
      var dps = 0;
      this.millitaryItems.forEach(item => {
        dps += item.dps * item.value;
      });
      return dps;
    },
    defense() {
      var defense = 0;
      this.millitaryItems.forEach(item => {
        defense += item.defense * item.value;
      });
      return defense;
    },
    healthbarWidth() {
      return (this.hitpoints / 100) * 100 + "%";
    },
    resourceItems() {
      return this.items.filter(i => {
        if (i.type == "resource" && this.meetItemRequirements(i)) {
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
    },
    millitaryItems() {
      return this.items.filter(i => {
        if (i.type == "millitary" && this.meetEquipmentRequirements(i)) {
          return i;
        }
      });
    },
    equipmentItems() {
      return this.equipments
        .filter(i => {
          if (this.meetEquipmentRequirements(i)) {
            return i;
          }
        })
        .sort(function(a, b) {
          return b.crafted - a.crafted;
        })
        .sort((a, b) => {
          if (a.tier == "normal") return -1;
          if (b.tier == "normal") return 1;
          if (a.tier == "good") return -1;
          if (b.tier == "good") return 1;
          if (a.tier == "rare") return -1;
          if (b.tier == "rare") return 1;
        });
    }
  },
  methods: {
    gameLoop() {
      var dubloonMultiplier = 1;
      dubloonMultiplier += Math.floor(this.items.find(i => i.label == "Coins").value * 0.1);
      this.villageItems.forEach(item => {
        if (item.value > 0) {
          if (item.label == "Farm") {
            var food = this.items.find(i => i.label.toLowerCase() == "food");
            food.value += 1 * item.value * dubloonMultiplier;
          }
          if (item.label == "Sawmill") {
            var wood = this.items.find(i => i.label.toLowerCase() == "wood");
            wood.value += 1 * item.value * dubloonMultiplier;
          }
          if (item.label == "Mines") {
            var stone = this.items.find(i => i.label.toLowerCase() == "stone");
            var coal = this.items.find(i => i.label.toLowerCase() == "coal");
            var iron = this.items.find(i => i.label.toLowerCase() == "iron");
            var gold = this.items.find(i => i.label.toLowerCase() == "gold");
            stone.value += 1 * item.value * dubloonMultiplier;
            if (Math.random() > 0.5) coal.value += 1 * item.value * dubloonMultiplier;
            if (Math.random() > 0.7) iron.value += 1 * item.value * dubloonMultiplier;
            if (Math.random() > 0.95) gold.value += 1 * item.value * dubloonMultiplier;
          }
        }
      });
      this.heal(1);
      this.handleEnemyStates();
      if (this.enemies.length < this.maxEnemies) this.spawnEnemy();
    },
    handleEnemyStates() {
      this.enemies.forEach(e => {
        if (e.hitpoints <= 0) {
          e.loot.forEach(l => {
            if (l.label == "Royal Dubloons") this.items.find(item => item.label == l.label).value += l.value;
          });
          this.enemies.splice(this.enemies.indexOf(e), 1);
        }
      });
    },
    spawnEnemy() {
      var rand = Math.floor(Math.random() * enemiesJson.enemies.length);
      var enemy = enemiesJson.enemies[rand];
      this.enemies.push(JSON.parse(JSON.stringify(enemy)));
    },
    heal(ammount) {
      if (this.hitpoints >= 100) this.hitpoints == 100;
      else this.hitpoints += ammount;
    },
    takeDamage(ammount) {
      if (ammount > this.defense) this.hitpoints -= ammount - this.defense;
      else this.hitpoints -= 1;
      if (this.hitpoints <= 0) this.hitpoints == 0;
    },
    attackEnemy(enemy) {
      if (this.hitpoints >= enemy.damage) {
        enemy.hitpoints -= this.dps;
        this.takeDamage(enemy.damage);
      }
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

        //handle additional effects
        if (item.label == "Smeltery") this.items.find(i => i.label == "steel").multiplier++;
        if (item.label == "Barracks") this.lodging += 10;
        if (item.label == "Fort") this.lodging += 50;
        if (item.label == "Castle") this.lodging += 100;
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
        if (equipment.upgradeEffect == "food") {
          this.items.find(i => i.label == "food").multiplier += equipment.ammount;
          this.$set(equipment, "crafted", true);
          this.applyCosts(equipment.cost);
        }
        if (equipment.upgradeEffect == "mineral") {
          this.items.find(i => i.label == "stone").multiplier += equipment.ammount;
          this.items.find(i => i.label == "coal").multiplier += equipment.ammount;
          this.items.find(i => i.label == "iron").multiplier += equipment.ammount;
          this.items.find(i => i.label == "gold").multiplier += equipment.ammount;
          this.$set(equipment, "crafted", true);
          this.applyCosts(equipment.cost);
        }
      }
    },
    trainMillitary(millitary) {
      if (millitary.cost && this.canIncrement(millitary.cost)) {
        this.applyCosts(millitary.cost);
        millitary.value += 1 * millitary.multiplier;
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
      var item = this.items.find(c => {
        if (c.label.toLowerCase() == name.toLowerCase()) {
          return c;
        }
      });
      if (name == "lodging" && this.lodging >= cost) return true;
      if (item && item.value >= cost) return true;
      else return false;
    },
    applyCosts(costs) {
      Object.keys(costs).filter(name => {
        this.applyCost(costs[name], name);
      });
    },
    applyCost(cost, name) {
      if (name == "lodging") {
        this.lodging -= cost;
      } else {
        this.items.find(c => {
          if (c.label.toLowerCase() == name.toLowerCase()) {
            c.value -= cost;
          }
        });
      }
    }
  }
};
</script>

<style lang="scss">
body {
  background-image: url("./assets/bg.jpeg");
}
</style>
