<template>
  <iframe
    v-if="shouldFlatbedBeVisible"
    width="560"
    height="315"
    id="flatbed"
    src="https://www.youtube.com/embed/aYYWW7V25Ok?rel=0&controls=0&showinfo=0&autoplay=1&mute=1"
    allow="autoplay; encrypted-media"
  ></iframe>
  <NavBar siteName="MonoGuns Revamped"></NavBar>
  <transition name="fade">
    <Menu v-show="isMenuOpen"></Menu>
  </transition>
  <div
    class="menuClickaway"
    v-if="isMenuOpen"
    @click="this.$store.state.menuOpen = false"
    style="width: 100vw; height: 100vh; position: fixed; z-index: 2"
  ></div>
  <div id="parent">
    <ItemSelector></ItemSelector>
    <ItemCalculator></ItemCalculator>
  </div>
  <footer v-if="shouldFooterBeVisible">
    <p style="text-align: center">#addtheflatbed</p>
    <p class="smallText" style="text-align: center">
      Don't wanna see this? Turn it off in the settings
    </p>
  </footer>
</template>

<script>
import NavBar from "./components/NavBar.vue";
import ItemSelector from "./components/ItemSelector.vue";
import ItemCalculator from "./components/ItemCalculator.vue";
import axios from "axios";
import Menu from "./components/Menu.vue";
import swal from "sweetalert2";
export default {
  name: "App",
  components: {
    NavBar,
    ItemSelector,
    ItemCalculator,
    Menu,
  },
  methods: {
    async loadSettings() {
      let options = JSON.parse(localStorage.getItem("options")) || {};
      for (let i in options) {
        this.$store.commit("setOption", {
          option: i,
          value: options[i],
        });
      }
    },
    async fetchItems() {
      let materials;
      let items;
      let categories = {};
      if (this.$store.getters.getOption("materialsInsteadOfItems")) {
        items = await axios.get("./materials.json");
        materials = await axios.get("./items.json");
      } else {
        items = await axios.get("./items.json");
        materials = await axios.get("./materials.json");
      }
      for (let i in items.data) {
        let item = items.data[i];
        if (!categories[item.category]) categories[item.category] = {};
        categories[item.category][i] = item;
      }
      this.$store.commit("setItems", items.data);
      this.$store.commit("setMaterials", materials.data);
      this.$store.commit("setCategories", categories);
    },
  },
  created() {
    this.loadSettings();
    this.fetchItems();
    let konamiProgress = 0;
    document.addEventListener("keyup", async (ev) => {
      if (!ev.code) return;
      if (
        !ev.code.startsWith("Arrow") &&
        ev.code != "KeyA" &&
        ev.code != "KeyB"
      )
        return;
      if (ev.code == "ArrowUp" && konamiProgress < 2) {
        konamiProgress++;
      }
      if (ev.code == "ArrowDown" && konamiProgress >= 2 && konamiProgress < 4) {
        konamiProgress++;
      }
      if (
        ev.code == "ArrowLeft" &&
        (konamiProgress == 4 || konamiProgress == 6)
      ) {
        konamiProgress++;
      }
      if (
        ev.code == "ArrowRight" &&
        (konamiProgress == 5 || konamiProgress == 7)
      ) {
        konamiProgress++;
      }
      if (ev.code == "KeyB" && konamiProgress == 8) {
        konamiProgress++;
      }
      if (ev.code == "KeyA" && konamiProgress == 9) {
        konamiProgress++;
      }
      if (konamiProgress == 10) {
        konamiProgress = 69420;
        this.$store.state.optionsObj.push({
          name: "Show materials instead of items in the itemSelector.",
          id: "materialsInsteadOfItems",
          default: false,
          onChange: () => {
            setTimeout(() => {
              this.fetchItems();
            }, 250);
          },
        });
        swal.fire(
          "Woah, a smart one!",
          "You have unlocked a secret setting!",
          "success"
        );
      }
    });
  },
  computed: {
    isMenuOpen() {
      return this.$store.state.menuOpen;
    },
    shouldFlatbedBeVisible() {
      return this.$store.getters.getOption("flatbed");
    },
    shouldFooterBeVisible() {
      return this.$store.getters.getOption("footer");
    },
  },
};
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.itemDiv {
  display: flex;
  align-items: center;
  margin-top: 0.25vw;
  width: 100%;
  height: 6vw;
  background: #333;
  cursor: pointer;
}
.itemInfo {
  position: relative;
  display: flex;
  flex-direction: column;
  margin: 0.2vw 0.2vw 0.2vw 0;
  width: calc(100% - 0.25vw);
  height: calc(100% - 0.5vw);
  background: #222;
  border-radius: 0.1vw;
}
.itemAmount {
  display: inline;
  font-size: 1vw;
  margin-left: 0.3vw;
  margin-bottom: 0.1vw;
}
.itemStacksize {
  position: absolute;
  font-size: 0.9vw;
  right: 0.2vw;
  bottom: 0.2vw;
}
.itemName {
  display: inline;
  font-size: 1vw;
  margin-bottom: 0.2vw;
  margin-top: 0.5vw;
  margin-left: 0.3vw;
}
.flexCenter {
  display: flex;
  justify-content: center;
}
::-webkit-scrollbar {
  width: 1vw;
}

::-webkit-scrollbar-thumb {
  background-color: rgb(102, 102, 102);
  border-radius: 0.15vw;
}
::-webkit-scrollbar-track {
  background-color: rgb(51, 51, 51);
}
.itemContainer {
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  height: 80vh;
  /* height: 10vw; */
  display: flex;
  flex-direction: column;
  padding-right: 0.3vw;
}
.itemMarketprice,
.itemIllegal,
.itemLegal,
.itemPrice {
  margin-left: 0.3vw;
}

.itemImg {
  display: inline;
  margin: 0.2vw 0.2vw 0.2vw 0.2vw;
  height: calc(100% - 0.5vw);
  border-radius: 0.1vw;
}

.itemIllegal {
  color: red;
}

.itemLegal {
  color: green;
}

#flatbed {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}

.assaultrifle {
  background: rgb(255, 154, 0);
  background: linear-gradient(
    135deg,
    rgba(255, 154, 0, 0.4) 0%,
    rgba(255, 154, 0, 1) 100%
  );
}

.pistol {
  background: rgb(0, 99, 255);
  background: linear-gradient(
    135deg,
    rgba(0, 99, 255, 0.4) 0%,
    rgba(0, 99, 255, 1) 100%
  );
}

.shotgun {
  background: rgb(214, 0, 255);
  background: linear-gradient(
    135deg,
    rgba(214, 0, 255, 0.4) 0%,
    rgba(214, 0, 255, 1) 100%
  );
}

.melee {
  background: rgb(255, 0, 0);
  background: linear-gradient(
    135deg,
    rgba(255, 0, 0, 0.4) 0%,
    rgba(255, 0, 0, 1) 100%
  );
}

.exotic {
  background: rgb(238, 255, 0);
  background: linear-gradient(
    135deg,
    rgba(238, 255, 0, 0.4) 0%,
    rgba(238, 255, 0, 1) 100%
  );
}

.greenItem {
  background: rgb(0, 255, 0);
  background: linear-gradient(
    135deg,
    rgba(0, 255, 0, 0.4) 0%,
    rgba(0, 255, 0, 1) 100%
  );
}

.blueprint {
  background: url("/img/blueprint.png") no-repeat center center fixed;
  background-size: 100% 100%;
}

.itemMarketprice,
.itemPrice,
.itemLegality {
  font-size: 1vw;
}

.craftingItem {
  background: rgb(255, 255, 255);
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.4) 0%,
    rgba(255, 255, 255, 1) 100%
  );
}
* {
  margin: 0;
  padding: 0;
}
#parent {
  display: flex;
  flex-direction: row;
  justify-content: space-evenly;
  width: 100%;
  height: 100%;
}
* {
  font-family: "Montserrat", sans-serif;
}
#app {
  color: white;
}
body {
  background-color: #111;
}
footer {
  padding: 1vw;
  border-radius: 0.3vw;
  position: fixed;
  right: 50%;
  transform: translate(50%, -50%);
  top: 90%;
  font-size: 2vw;
  color: orange;
  background-color: black;
  opacity: 0.5;
}
footer > .smallText {
  font-size: 0.6vw;
}
</style>
