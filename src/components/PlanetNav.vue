<template>
  <span class="planetnav">
    <span v-if="gameUser != null"
      ><router-link to="/user">{{ gameUser }}</router-link></span
    >
    <span v-else
      ><router-link to="/user"><account-icon :title="$t('User')"/></router-link
    ></span>
    |
    <router-link :to="'/planets'">
      <earth-icon :title="$t('Planets')" />
    </router-link>
    <span v-for="planet in this.planets" :key="planet.id">
      |
      <span v-if="planet.id === planetId"
        ><font color="green"
          ><span @click="setPlanet(planet)">
            {{ planet.name | shorten }}
          </span></font
        >
      </span>
      <span class="pointer" v-else @click="setPlanet(planet)">
        {{ planet.name | shorten }}
      </span>
    </span>
  </span>
</template>

<script>
import PlanetsService from "@/services/planets";
import { mapState } from "vuex";
import EarthIcon from "vue-material-design-icons/Earth.vue";
import AccountIcon from "vue-material-design-icons/Account.vue";
import * as types from "@/store/mutation-types";

export default {
  name: "planetnav",
  components: {
    EarthIcon,
    AccountIcon
  },
  data: function() {
    return {
      planets: null
    };
  },
  async mounted() {
    this.$store.subscribe(mutation => {
      switch (mutation.type) {
        case "game/" + types.SET_GAME_USER:
          this.prepareComponent();
      }
    });
    await this.prepareComponent();
  },
  filters: {
    shorten(name) {
      return name.substring(0, 2);
    }
  },
  computed: {
    ...mapState({
      loginUser: state => state.game.loginUser,
      accessToken: state => state.game.accessToken,
      gameUser: state => state.game.user,
      planetId: state => state.planet.id
    })
  },
  methods: {
    async prepareComponent() {
      await this.getPlanets();
    },
    async getPlanets() {
      const response = await PlanetsService.byUser(this.gameUser);
      this.planets = response.planets;
      this.$store.dispatch("planet/setList", response.planets);
    },
    setPlanet(planet) {
      if (planet.id !== this.planetId) {
        this.$store.dispatch("planet/setId", planet.id);
        this.$store.dispatch("planet/setName", planet.name);
        this.$store.dispatch("planet/setPosX", planet.posx);
        this.$store.dispatch("planet/setPosY", planet.posy);
      }
    },
    resetPlanet() {
      this.$store.dispatch("planet/setId", null);
      this.$store.dispatch("planet/setName", null);
      this.$store.dispatch("planet/setPosX", null);
      this.$store.dispatch("planet/setPosY", null);
    }
  }
};
</script>
<style>
.pointer {
  cursor: pointer;
}
</style>
