<template>
  <div class="shipyard">
    <h1>{{ $t("Shipyard") }}</h1>
    <p>
      <i>{{ $t("Ships need skill level 20 to build.") }}</i>
    </p>
    <template v-if="gameUser !== 'null' && planetId != 'null'">
      <table>
        <thead>
          <th @click="sort('longname')">{{ $t("Ship") }}</th>
          <th @click="sort('min_level')">{{ $t("Need") }}</th>
          <th @click="sort('cur_level')">{{ $t("Yard") }}</th>
          <th @click="sort('cur_level_skill')">{{ $t("Skill") }}</th>
          <th @click="sort('coal')">{{ $t("C") }}</th>
          <th @click="sort('ore')">{{ $t("Fe") }}</th>
          <th @click="sort('copper')">{{ $t("Cu") }}</th>
          <th @click="sort('uranium')">{{ $t("U") }}</th>
          <th @click="sort('time')">{{ $t("Needs") }}</th>
          <th @click="sort('attack')">{{ $t("A/D") }}</th>
          <th @click="sort('busy_until')">{{ $t("Busy") }}</th>
          <th v-if="loginUser !== null && loginUser === gameUser">
            {{ $t("Build") }}
          </th>
          <th></th>
        </thead>
        <tbody>
          <tr v-for="ship in sortedShipyard" :key="ship.longname">
            <td>
              <font v-if="ship.activated === true || ship.variant === 0">{{
                $t(ship.longname)
              }}</font
              ><font v-else color="grey">{{ $t(ship.longname) }}</font>
            </td>
            <td>
              {{ ship.min_level }}
            </td>
            <td>
              <font v-if="ship.cur_level < ship.min_level" color="red">{{
                ship.cur_level
              }}</font
              ><font v-else>{{ ship.cur_level }}</font>
            </td>
            <td>
              <font v-if="ship.skill < 20" color="red">{{
                ship.skill === null ? 0 : ship.skill
              }}</font
              ><font v-else>{{ ship.skill === null ? 0 : ship.skill }}</font>
            </td>
            <td>
              <font v-if="ship.cost.coal > coal" color="red">{{
                ship.cost.coal === 0 ? "-" : ship.cost.coal
              }}</font>
              <font v-else>{{
                ship.cost.coal === 0 ? "-" : ship.cost.coal
              }}</font>
            </td>
            <td>
              <font v-if="ship.cost.ore > ore" color="red">{{
                ship.cost.ore === 0 ? "-" : ship.cost.ore
              }}</font
              ><font v-else>{{
                ship.cost.ore === 0 ? "-" : ship.cost.ore
              }}</font>
            </td>
            <td>
              <font v-if="ship.cost.copper > copper" color="red">{{
                ship.cost.copper === 0 ? "-" : ship.cost.copper
              }}</font
              ><font v-else>{{
                ship.cost.copper === 0 ? "-" : ship.cost.copper
              }}</font>
            </td>
            <td>
              <font v-if="ship.cost.uranium > uranium" color="red">{{
                ship.cost.uranium === 0 ? "-" : ship.cost.uranium
              }}</font
              ><font v-else>{{
                ship.cost.uranium === 0 ? "-" : ship.cost.uranium
              }}</font>
            </td>
            <td>
              {{ ship.cost.time | timePretty }}
            </td>
            <td>
              {{ (ship.rocket + ship.bullet + ship.laser) | omitZero }} /
              {{ ship.structure + ship.armor + ship.shield }}
            </td>
            <td>{{ ship.busy_until | busyPretty }}</td>
            <td v-if="loginUser !== null && loginUser === gameUser">
              <button
                :disabled="clicked.includes(ship.longname)"
                v-if="shipPossible(ship)"
                @click="buildShip(ship)"
              >
                <arrow-up-bold-icon :title="$t('Build')" />
              </button>
            </td>
            <td v-if="chainResponse.includes(ship.longname)">
              <timer-sand-icon :title="$t('Transaction sent')" />
            </td>
          </tr>
        </tbody>
      </table>
    </template>
    <template v-else>
      <template v-if="gameUser === 'null'">
        <p>
          {{ $t("Please set the") }}
          <router-link to="/user">{{ $t("user") }}</router-link>
        </p>
      </template>
      <template v-if="planetId === 'null'"
        ><p>
          {{ $t("Please set the") }}
          <router-link :to="'/planets'">{{ $t("planet") }}</router-link>
        </p>
      </template>
    </template>
  </div>
</template>

<script>
import ShipyardService from "@/services/shipyard";
import QuantityService from "@/services/quantity";
import SteemConnectService from "@/services/steemconnect";
import moment from "moment";
import { mapState } from "vuex";
import TimerSandIcon from "vue-material-design-icons/TimerSand.vue";
import ArrowUpBoldIcon from "vue-material-design-icons/ArrowUpBold.vue";
import * as types from "@/store/mutation-types";

export default {
  name: "shipyard",
  components: {
    TimerSandIcon,
    ArrowUpBoldIcon
  },
  data: function() {
    return {
      shipyard: null,
      quantity: null,
      interval: null,
      coal: null,
      ore: null,
      copper: null,
      uranium: null,
      clicked: [],
      chainResponse: [],
      currentSort: "longname",
      currentSortDir: "asc"
    };
  },
  async mounted() {
    this.clicked = [];
    this.chainResponse = [];
    await this.prepareComponent();
    this.interval = setInterval(() => {
      this.calculateCoal();
      this.calculateOre();
      this.calculateCopper();
      this.calculateUranium();
    }, 1000);
    this.$store.subscribe(mutation => {
      switch (mutation.type) {
        case "planet/" + types.SET_PLANET_ID:
          this.clicked = [];
          this.chainResponse = [];
          this.prepareComponent();
      }
    });
  },
  filters: {
    busyPretty(busy) {
      var busyUntil = moment(new Date(busy * 1000));
      var now = moment.utc();
      if (busy === 0) {
        return "-";
      } else {
        if (now.isAfter(busyUntil)) {
          return "-";
        } else {
          return moment.duration(now.diff(busyUntil)).humanize();
        }
      }
    },
    timePretty(time) {
      return moment.duration(parseInt(time), "seconds").humanize();
    },
    omitZero(number) {
      if (number == 0) {
        return "-";
      }
      return number;
    }
  },
  computed: {
    ...mapState({
      loginUser: state => state.game.loginUser,
      accessToken: state => state.game.accessToken,
      gameUser: state => state.game.user,
      planetId: state => state.planet.id
    }),
    sortedShipyard() {
      var sortedShipyard = this.shipyard;
      if (sortedShipyard !== null) {
        return sortedShipyard.sort((a, b) => {
          let modifier = 1;
          if (this.currentSortDir === "desc") modifier = -1;
          if (a[this.currentSort] === null) return -1 * modifier;
          if (b[this.currentSort] === null) return 1 * modifier;
          // cost
          if (
            this.currentSort === "coal" ||
            this.currentSort === "ore" ||
            this.currentSort === "copper" ||
            this.currentSort === "uranium" ||
            this.currentSort === "time"
          ) {
            if (a.cost[this.currentSort] < b.cost[this.currentSort])
              return -1 * modifier;
            if (a.cost[this.currentSort] > b.cost[this.currentSort])
              return 1 * modifier;
            // attack
          } else if (this.currentSort === "attack") {
            if (a.rocket + a.bullet + a.laser < b.rocket + b.bullet + b.laser)
              return -1 * modifier;
            if (a.rocket + a.bullet + a.laser > b.rocket + b.bullet + b.laser)
              return 1 * modifier;
            // defense
          } else if (this.currentSort === "defense") {
            if (
              a.structure + a.armor + a.shield <
              b.structure + b.armor + b.shield
            )
              return -1 * modifier;
            if (
              a.structure + a.armor + a.shield >
              b.structure + b.armor + b.shield
            )
              return 1 * modifier;
            // all the others
          } else {
            if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
            if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
          }
          return 0;
        });
      } else {
        return sortedShipyard;
      }
    }
  },
  methods: {
    async prepareComponent() {
      await this.getShipyard();
      await this.getQuantity();
    },
    async getShipyard() {
      const response = await ShipyardService.all(this.planetId);
      this.shipyard = response;
    },
    isBusy(busy) {
      var busyUntil = moment(new Date(busy * 1000));
      var now = moment.utc();
      if (busyUntil === 0 || busy === null) {
        return false;
      } else {
        if (now.isAfter(busyUntil)) {
          return false;
        } else {
          return true;
        }
      }
    },
    buildShip(ship) {
      this.clicked.push(ship.longname);
      SteemConnectService.setAccessToken(this.accessToken);
      SteemConnectService.buildShip(
        this.loginUser,
        this.planetId,
        ship.type,
        (error, result) => {
          if (error === null && result.success) {
            this.chainResponse.push(ship.longname);
          }
        }
      );
    },
    shipPossible(ship) {
      if (this.isBusy(ship.busy_until)) {
        return false;
      }
      if (this.coal < ship.cost.coal) {
        return false;
      }
      if (this.ore < ship.cost.ore) {
        return false;
      }
      if (this.copper < ship.cost.copper) {
        return false;
      }
      if (this.uranium < ship.cost.uranium) {
        return false;
      }
      if (ship.cur_level < ship.min_level) {
        return false;
      }
      if (ship.activated === false && ship.variant !== 0) {
        return false;
      }
      if (ship.skill < 20) {
        return false;
      }
      return true;
    },
    async getQuantity() {
      const response = await QuantityService.get(this.planetId);
      this.quantity = response;
      this.calculateCoal();
      this.calculateOre();
      this.calculateCopper();
      this.calculateUranium();
    },
    calculateQuantity(quantity, depot, rate, lastUpdate) {
      var startTime = moment.unix(parseInt(lastUpdate));
      var endTime = moment.utc();
      var duration = moment.duration(endTime.diff(startTime));
      var diff = duration.asDays();

      return Math.max(
        Math.min(
          parseFloat(quantity) + parseFloat(rate) * diff, // accumulation
          parseFloat(depot) // below depot
        ),
        parseFloat(quantity) // or overflow above depot
      ).toFixed(1);
    },
    calculateCoal() {
      if (this.quantity !== null) {
        this.coal = this.calculateQuantity(
          this.quantity.coal,
          this.quantity.coaldepot,
          this.quantity.coalrate,
          this.quantity.lastUpdate
        );
      } else {
        this.coal = 0;
      }
    },
    calculateOre() {
      if (this.quantity !== null) {
        this.ore = this.calculateQuantity(
          this.quantity.ore,
          this.quantity.oredepot,
          this.quantity.orerate,
          this.quantity.lastUpdate
        );
      } else {
        this.ore = 0;
      }
    },
    calculateCopper() {
      if (this.quantity !== null) {
        this.copper = this.calculateQuantity(
          this.quantity.copper,
          this.quantity.copperdepot,
          this.quantity.copperrate,
          this.quantity.lastUpdate
        );
      } else {
        this.copper = 0;
      }
    },
    calculateUranium() {
      if (this.quantity !== null) {
        this.uranium = this.calculateQuantity(
          this.quantity.uranium,
          this.quantity.uraniumdepot,
          this.quantity.uraniumrate,
          this.quantity.lastUpdate
        );
      } else {
        this.uranium = 0;
      }
    },
    sort(s) {
      //if s == current sort, reverse
      if (s === this.currentSort) {
        this.currentSortDir = this.currentSortDir === "asc" ? "desc" : "asc";
      }
      this.currentSort = s;
    }
  },
  beforeDestroy() {
    clearInterval(this.interval);
  }
};
</script>
