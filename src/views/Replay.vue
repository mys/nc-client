<template>
  <div class="replay">
    <h1>{{ $t("Replay") }}</h1>
    <p v-if="report !== null">
      {{ $t("Mission") }}: {{ report[battleIndex].mission_id }}
    </p>
    <p v-if="report !== null">
      {{ $t("Battle Log") }}:
      <router-link
        :to="{ path: '/battle/' + report[battleIndex].mission_id }"
        >{{ $t("Battle Log") }}</router-link
      >
    </p>
    <div
      v-for="fight in report"
      :key="fight.battle_number"
      @click="selectBattle(fight.battle_number - 1)"
    >
      <font v-if="fight.battle_number - 1 === battleIndex" color="green"
        >{{ $t("Battle") }}: {{ fight.battle_number }}</font
      >
      <span v-else>{{ $t("Battle") }}: {{ fight.battle_number }}</span>
    </div>
    <h2>
      {{ $t("Attacker") }} {{ report[battleIndex].attacker }}
      <font
        v-if="currentAttacker === this.slots && currentDefender === this.slots"
        color="yellow"
        >{{ $t("Draw") }}</font
      >
      <span v-else>
        <font v-if="currentDefender === this.slots" color="green">
          {{ $t("Winner") }}
        </font>
        <font v-if="currentAttacker === this.slots" color="red">
          {{ $t("Looser") }}
        </font>
      </span>
    </h2>
    <table>
      <thead>
        <th>{{ $t("Tank") }}</th>
        <th>{{ $t("Shooter") }}</th>
        <th>{{ $t("Ship") }}</th>
        <th>{{ $t("Quantity") }}</th>
        <th>{{ $t("Structure") }}</th>
        <th>{{ $t("Armor") }}</th>
        <th>{{ $t("Shield") }}</th>
        <th>{{ $t("Rocket") }}</th>
        <th>{{ $t("Bullet") }}</th>
        <th>{{ $t("Laser") }}</th>
        <th>{{ $t("Survivor") }}</th>
      </thead>
      <tbody>
        <tr
          v-show="attacker.id !== 'end' && attacker.quantity !== 0"
          v-for="(attacker, index) in attackers"
          :key="attacker.id"
        >
          <td>
            <div v-if="index === currentAttacker">
              <shield-airplane-icon :title="$t('Tank')" />
            </div>
          </td>
          <td>
            <div v-if="turn === 'Attacker' && index === currentAttackerShooter">
              <pistol-icon :title="$t('Shooter')" />
            </div>
            <div v-else></div>
          </td>
          <td>{{ attacker.name }}</td>
          <td>{{ attacker.quantity === 0 ? " " : attacker.quantity }}</td>
          <td>
            <div v-if="attacker.structure > 0">
              <font v-if="attacker.structure === 0" color="red">
                {{ attacker.structure.toFixed(0) }}
              </font>
              <font v-else>{{ attacker.structure.toFixed(0) }}</font>
            </div>
          </td>
          <td>
            <div v-if="attacker.armor > 0">{{ attacker.armor.toFixed(0) }}</div>
          </td>
          <td>
            <div v-if="attacker.shield > 0">
              {{ attacker.shield.toFixed(0) }}
            </div>
          </td>
          <td>
            <div v-if="attacker.rocket > 0">
              <font color="orange">{{ attacker.rocket.toFixed(0) }}</font>
            </div>
          </td>
          <td>
            <div v-if="attacker.bullet > 0">
              <font color="orange">{{ attacker.bullet.toFixed(0) }}</font>
            </div>
          </td>
          <td>
            <div v-if="attacker.laser > 0">
              <font color="orange">{{ attacker.laser.toFixed(0) }}</font>
            </div>
          </td>
          <td v-show="attacker.id !== 'end'">
            <div v-if="attacker.survivor > 0">
              <font v-if="attacker.survivor > 0" color="green">
                {{ attacker.survivor }}
              </font>
              <div v-else>{{ attacker.survivor }}</div>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
    <h2>
      {{ $t("Defender") }} {{ report[battleIndex].defender }}
      <font
        v-if="currentAttacker === this.slots && currentDefender === this.slots"
        color="yellow"
        >{{ $t("Draw") }}</font
      >
      <span v-else>
        <font v-if="currentAttacker === slots" color="green">
          {{ $t("Winner") }}
        </font>
        <font v-if="currentDefender === slots" color="red">
          {{ $t("Looser") }}
        </font>
      </span>
    </h2>
    <br />
    <table>
      <thead>
        <th>{{ $t("Tank") }}</th>
        <th>{{ $t("Shooter") }}</th>
        <th>{{ $t("Ship") }}</th>
        <th>{{ $t("Quantity") }}</th>
        <th>{{ $t("Structure") }}</th>
        <th>{{ $t("Armor") }}</th>
        <th>{{ $t("Shield") }}</th>
        <th>{{ $t("Rocket") }}</th>
        <th>{{ $t("Bullet") }}</th>
        <th>{{ $t("Laser") }}</th>
        <th>{{ $t("Survivor") }}</th>
      </thead>
      <tbody>
        <tr
          v-show="defender.id !== 'end' && defender.quantity !== 0"
          v-for="(defender, index) in defenders"
          :key="defender.id"
        >
          <td>
            <div v-if="index === currentDefender">
              <shield-airplane-icon :title="$t('Tank')" />
            </div>
          </td>
          <td>
            <div v-if="turn === 'Defender' && index === currentDefenderShooter">
              <pistol-icon :title="$t('Shooter')" />
            </div>
            <div v-else></div>
          </td>
          <td>{{ defender.name }}</td>
          <td>{{ defender.quantity === 0 ? " " : defender.quantity }}</td>
          <td>
            <div v-if="defender.structure > 0">
              <font v-if="defender.structure === 0" color="red">
                {{ defender.structure.toFixed(0) }}
              </font>
              <font v-else>{{ defender.structure.toFixed(0) }}</font>
            </div>
          </td>
          <td>
            <div v-if="defender.armor > 0">{{ defender.armor.toFixed(0) }}</div>
          </td>
          <td>
            <div v-if="defender.shield > 0">
              {{ defender.shield.toFixed(0) }}
            </div>
          </td>
          <td>
            <div v-if="defender.rocket > 0">
              <font color="orange">{{ defender.rocket.toFixed(0) }}</font>
            </div>
          </td>
          <td>
            <div v-if="defender.bullet > 0">
              <font color="orange">{{ defender.bullet.toFixed(0) }}</font>
            </div>
          </td>
          <td>
            <div v-if="defender.laser > 0">
              <font color="orange">{{ defender.laser.toFixed(0) }}</font>
            </div>
          </td>
          <td>
            <div v-if="defender.survivor > 0">
              <font v-if="defender.survivor > 0" color="green">
                {{ defender.survivor }}
              </font>
              <div v-else>{{ defender.survivor }}</div>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
    <h2>{{ $t("Battle") }}</h2>
    <button v-on:click="battle()">{{ $t("Play Turn") }}</button>
    <button v-on:click="auto()">{{ $t("Play Battle") }}</button>
    <button v-on:click="reset()">{{ $t("Reset") }}</button>
    <h2>{{ $t("Battle Log") }}</h2>
    <p>{{ result }}</p>
    <p>{{ $t("Next Turn") }}: {{ turn }}</p>
    <p>{{ $t("Round") }}: {{ round }}</p>
  </div>
</template>

<script>
import attackerShips from "@/data/attackerShips";
import defenderShips from "@/data/defenderShips";
import attackers from "@/data/attackers";
import defenders from "@/data/defenders";
import BattleService from "@/services/battle";
import PistolIcon from "vue-material-design-icons/Pistol.vue";
import ShieldAirplaneIcon from "vue-material-design-icons/ShieldAirplane.vue";

export default {
  name: "Replay",
  props: ["missionId"],
  components: {
    PistolIcon,
    ShieldAirplaneIcon
  },
  data() {
    return {
      rates: [
        { id: 1, attack: "rocket", defense: "structure", rate: 4 },
        { id: 2, attack: "rocket", defense: "armor", rate: 2 },
        { id: 3, attack: "rocket", defense: "shield", rate: 1 },
        { id: 4, attack: "laser", defense: "structure", rate: 2 },
        { id: 5, attack: "laser", defense: "armor", rate: 1 },
        { id: 6, attack: "laser", defense: "shield", rate: 4 },
        { id: 7, attack: "bullet", defense: "structure", rate: 1 },
        { id: 8, attack: "bullet", defense: "armor", rate: 4 },
        { id: 9, attack: "bullet", defense: "shield", rate: 2 }
      ],
      attackers: attackers,
      defenders: defenders,
      turn: "Attacker",
      currentAttacker: 0,
      currentDefender: 0,
      currentAttackerShooter: 0,
      currentDefenderShooter: 0,
      round: 0,
      result: "Ready",
      attackerShieldRegen: 0,
      defenderShieldRegen: 0,
      shieldregreduce: 0.005,
      attackerArmorRep: 0,
      defenderArmorRep: 0,
      armorregreduce: 0.01,
      piercerateshield: 1,
      pierceratearmor: 1,
      slots: 8,
      prepared: false,
      interval: null,
      report: null,
      attackerShips: attackerShips,
      defenderShips: defenderShips,
      battleIndex: 0
    };
  },
  async mounted() {
    await this.prepareComponent();
  },
  methods: {
    async prepareComponent() {
      await this.getBattle();
    },
    async getBattle() {
      const response = await BattleService.all(this.missionId);
      this.report = response;
      this.prepare();
    },
    prepare: function() {
      this.attackerArmorRep = this.report[this.battleIndex].attacker_armorrep;
      this.defenderArmorRep = this.report[this.battleIndex].defender_armorrep;
      this.attackerShieldRegen = this.report[
        this.battleIndex
      ].attacker_shieldregen;
      this.defenderShieldRegen = this.report[
        this.battleIndex
      ].defender_shieldregen;
      var initialAttackers = this.report[this.battleIndex]
        .initial_attacker_ships;
      this.attackers.forEach(attacker => {
        attacker.name = null;
      });
      initialAttackers.forEach(ina => {
        let i = ina.pos - 1;
        // Fill stats variables (will change)
        attackers[i].name = ina.longname;
        attackers[i].quantity = ina.n;
        attackers[i].structure = ina.structure;
        attackers[i].armor = ina.armor;
        attackers[i].shield = ina.shield;
        attackers[i].rocket = ina.rocket;
        attackers[i].bullet = ina.bullet;
        attackers[i].laser = ina.laser;
        attackers[i].survivor = ina.survivor;
        // Save start values (will stay)
        attackerShips.forEach(ship => {
          if (ship.name === ina.longname) {
            ship.structure = ina.structure / ina.n;
            ship.armor = ina.armor / ina.n;
            ship.shield = ina.shield / ina.n;
            ship.rocket = ina.rocket / ina.n;
            ship.bullet = ina.bullet / ina.n;
            ship.laser = ina.laser / ina.n;
          }
        });
      });
      var initialDefenders = this.report[this.battleIndex]
        .initial_defender_ships;
      this.defenders.forEach(defender => {
        defender.name = null;
      });
      initialDefenders.forEach(ina => {
        let j = ina.pos - 1;
        defenders[j].name = ina.longname;
        defenders[j].quantity = ina.n;
        defenders[j].structure = ina.structure;
        defenders[j].armor = ina.armor;
        defenders[j].shield = ina.shield;
        defenders[j].rocket = ina.rocket;
        defenders[j].bullet = ina.bullet;
        defenders[j].laser = ina.laser;
        defenders[j].survivor = ina.survivor;
        defenderShips.forEach(ship => {
          if (ship.name === ina.longname) {
            ship.structure = ina.structure / ina.n;
            ship.armor = ina.armor / ina.n;
            ship.shield = ina.shield / ina.n;
            ship.rocket = ina.rocket / ina.n;
            ship.bullet = ina.bullet / ina.n;
            ship.laser = ina.laser / ina.n;
          }
        });
      });
      this.turn = "Attacker";
      this.round = 0;
      this.currentAttacker = 0;
      this.currentDefender = 0;
      this.currentAttackerShooter = 0;
      this.currentDefenderShooter = 0;
      this.result = "Ready";
      this.prepared = true;
    },
    reset: function() {
      clearInterval(this.interval);
      this.prepare();
    },
    rate: function(attack, defense) {
      var rate = this.rates.filter(
        r => r.attack === attack && r.defense === defense
      );
      return rate[0].rate;
    },
    auto: function() {
      this.prepare();
      this.interval = setInterval(() => {
        this.battle();
      }, 100);
      this.battle();
    },
    selectBattle(battleIndex) {
      this.battleIndex = battleIndex;
      this.reset();
    },
    battle: function() {
      var attackerIndex = this.currentAttacker;
      var defenderIndex = this.currentDefender;
      var a;
      var d;

      if (!this.prepared) {
        this.prepare();
      }

      attackerIndex = this.currentAttacker;
      defenderIndex = this.currentDefender;

      if (attackerIndex === this.slots || defenderIndex === this.slots) {
        if (attackerIndex === this.slots) {
          this.result = "Defender won battle. Game Over";
          clearInterval(this.interval);
        }
        if (defenderIndex === this.slots) {
          this.result = "Attacker won battle. Game Over";
          clearInterval(this.interval);
        }

        return;
      }

      // Move through Tanks
      if (
        this.attackers[attackerIndex].structure === 0 ||
        this.attackers[attackerIndex].quantity === 0
      ) {
        if (this.attackers[attackerIndex + 1] !== "undefined") {
          attackerIndex = attackerIndex + 1;
          this.currentAttacker = attackerIndex;
          this.currentAttackerShooter = attackerIndex;
        }
      }

      if (
        this.defenders[defenderIndex].structure === 0 ||
        this.defenders[defenderIndex].quantity === 0
      ) {
        if (this.defenders[defenderIndex + 1] !== "undefined") {
          defenderIndex = defenderIndex + 1;
          this.currentDefender = defenderIndex;
          this.currentDefenderShooter = defenderIndex;
        }
      }

      a = this.attackers[attackerIndex];
      d = this.defenders[defenderIndex];
      var as = this.attackers[this.currentAttackerShooter];
      var ds = this.defenders[this.currentDefenderShooter];

      if (
        a.structure < 0 ||
        d.structure < 0 ||
        a.survivor === 0 ||
        d.survivor === 0
      ) {
        this.battle();
      } else {
        var lolaser = 0;
        var lobullet = 0;
        var lorocket = 0;
        var lolaser2 = 0;
        var lobullet2 = 0;
        var lorocket2 = 0;
        if (this.turn === "Attacker") {
          // Initial stats of defender
          let dShip = this.defenderShips.filter(s => {
            return s.name === d.name;
          });
          // Attacker Shoots
          if (as.structure > 0) {
            if (d.shield > 0) {
              if (as.laser * this.rate("laser", "shield") > d.shield) {
                lolaser =
                  Math.max(
                    as.laser * this.rate("laser", "shield") - d.shield,
                    0
                  ) * this.piercerateshield;
              } else {
                lolaser = 0;
              }
              if (as.bullet * this.rate("bullet", "shield") > d.shield) {
                lobullet =
                  Math.max(
                    as.bullet * this.rate("bullet", "shield") - d.shield,
                    0
                  ) * this.piercerateshield;
              } else {
                lobullet = 0;
              }
              if (as.rocket * this.rate("rocket", "shield") > d.shield) {
                lorocket =
                  Math.max(
                    as.rocket * this.rate("rocket", "shield") - d.shield,
                    0
                  ) * this.piercerateshield;
              } else {
                lorocket = 0;
              }
              d.shield = Math.max(
                d.shield - as.laser * this.rate("laser", "shield"),
                0
              );
              d.shield = Math.max(
                d.shield - as.bullet * this.rate("bullet", "shield"),
                0
              );
              d.shield = Math.max(
                d.shield - as.rocket * this.rate("rocket", "shield"),
                0
              );
              // Pierce Armor
              if (d.shield === 0) {
                if (lolaser > d.armor) {
                  lolaser2 =
                    Math.max(lolaser - d.armor, 0) * this.pierceratearmor;
                } else {
                  lolaser2 = 0;
                }
                if (lobullet > d.armor) {
                  lobullet2 =
                    Math.max(lobullet - d.armor, 0) * this.pierceratearmor;
                } else {
                  lobullet2 = 0;
                }
                if (lorocket > d.armor) {
                  lorocket2 =
                    Math.max(lorocket - d.armor, 0) * this.pierceratearmor;
                } else {
                  lorocket2 = 0;
                }
                d.armor = Math.max(d.armor - lolaser, 0);
                d.armor = Math.max(d.armor - lobullet, 0);
                d.armor = Math.max(d.armor - lorocket, 0);
                // Pierce structure
                if (d.armor === 0) {
                  d.structure = Math.max(d.structure - lolaser2, 0);
                  d.structure = Math.max(d.structure - lobullet2, 0);
                  d.structure = Math.max(d.structure - lorocket2, 0);
                }
              }
            } else if (d.armor > 0) {
              if (as.laser * this.rate("laser", "armor") > d.armor) {
                lolaser =
                  Math.max(
                    as.laser * this.rate("laser", "armor") - d.armor,
                    0
                  ) * this.pierceratearmor;
              } else {
                lolaser = 0;
              }
              if (as.bullet * this.rate("bullet", "armor") > d.armor) {
                lobullet =
                  Math.max(
                    as.bullet * this.rate("bullet", "armor") - d.armor,
                    0
                  ) * this.pierceratearmor;
              } else {
                lobullet = 0;
              }
              if (as.rocket * this.rate("rocket", "armor") > d.armor) {
                lorocket =
                  Math.max(
                    as.rocket * this.rate("rocket", "armor") - d.armor,
                    0
                  ) * this.pierceratearmor;
              } else {
                lorocket = 0;
              }
              d.armor = Math.max(
                d.armor - as.laser * this.rate("laser", "armor"),
                0
              );
              d.armor = Math.max(
                d.armor - as.bullet * this.rate("bullet", "armor"),
                0
              );
              d.armor = Math.max(
                d.armor - as.rocket * this.rate("rocket", "armor"),
                0
              );
              // Pierce structure
              if (d.armor === 0) {
                d.structure = Math.max(d.structure - lolaser, 0);
                d.structure = Math.max(d.structure - lobullet, 0);
                d.structure = Math.max(d.structure - lorocket, 0);
              }
            } else if (d.structure > 0) {
              d.structure = Math.max(
                d.structure - as.laser * this.rate("laser", "structure"),
                0
              );
              d.structure = Math.max(
                d.structure - as.bullet * this.rate("bullet", "structure"),
                0
              );
              d.structure = Math.max(
                d.structure - as.rocket * this.rate("rocket", "structure"),
                0
              );
            }
            if (d.structure !== "undefined") {
              this.defenders[defenderIndex].survivor = Math.ceil(
                d.structure / dShip[0].structure
              );
            }
            this.defenders[defenderIndex].laser =
              dShip[0].laser * this.defenders[defenderIndex].survivor;
            this.defenders[defenderIndex].bullet =
              dShip[0].bullet * this.defenders[defenderIndex].survivor;
            this.defenders[defenderIndex].rocket =
              dShip[0].rocket * this.defenders[defenderIndex].survivor;
            this.defenders[defenderIndex].structure = d.structure;
            this.defenders[defenderIndex].armor = d.armor;
            if (this.defenders[defenderIndex].armor !== 0) {
              this.defenders[defenderIndex].armor = Math.min(
                this.defenders[defenderIndex].armor +
                  dShip[0].armor *
                    Math.max(
                      this.defenderArmorRep - this.armorregreduce * this.round,
                      0
                    ) *
                    this.defenders[defenderIndex].survivor,
                dShip[0].armor * this.defenders[defenderIndex].survivor
              );
            }
            this.defenders[defenderIndex].shield = d.shield;
            if (this.defenders[defenderIndex].shield !== 0) {
              this.defenders[defenderIndex].shield = Math.min(
                this.defenders[defenderIndex].shield +
                  dShip[0].shield *
                    Math.max(
                      this.defenderShieldRegen -
                        this.shieldregreduce * this.round,
                      0
                    ) *
                    this.defenders[defenderIndex].survivor,
                dShip[0].shield * this.defenders[defenderIndex].survivor
              );
            }
          }
        }

        if (this.turn === "Defender") {
          // Initial stats of Attacker
          let aShip = this.attackerShips.filter(s => {
            return s.name === a.name;
          });
          // Defender Shoots
          if (ds.structure > 0) {
            if (a.shield > 0) {
              if (ds.laser * this.rate("laser", "shield") > a.shield) {
                lolaser =
                  Math.max(
                    ds.laser * this.rate("laser", "shield") - a.shield,
                    0
                  ) * this.piercerateshield;
              } else {
                lolaser = 0;
              }
              if (ds.bullet * this.rate("bullet", "shield") > a.shield) {
                lobullet =
                  Math.max(
                    ds.bullet * this.rate("bullet", "shield") - a.shield,
                    0
                  ) * this.piercerateshield;
              } else {
                lobullet = 0;
              }
              if (ds.rocket * this.rate("rocket", "shield") > a.shield) {
                lorocket =
                  Math.max(
                    ds.rocket * this.rate("rocket", "shield") - a.shield,
                    0
                  ) * this.piercerateshield;
              } else {
                lorocket = 0;
              }
              a.shield = Math.max(
                a.shield - ds.laser * this.rate("laser", "shield"),
                0
              );
              a.shield = Math.max(
                a.shield - ds.bullet * this.rate("bullet", "shield"),
                0
              );
              a.shield = Math.max(
                a.shield - ds.rocket * this.rate("rocket", "shield"),
                0
              );
              // Pierce Armor
              if (a.shield === 0) {
                if (lolaser > a.armor) {
                  lolaser2 =
                    Math.max(lolaser - a.armor, 0) * this.pierceratearmor;
                } else {
                  lolaser2 = 0;
                }
                if (lobullet > a.armor) {
                  lobullet2 =
                    Math.max(lobullet - a.armor, 0) * this.pierceratearmor;
                } else {
                  lobullet2 = 0;
                }
                if (lorocket > a.armor) {
                  lorocket2 =
                    Math.max(lorocket - a.armor, 0) * this.pierceratearmor;
                } else {
                  lorocket2 = 0;
                }
                a.armor = Math.max(a.armor - lolaser, 0);
                a.armor = Math.max(a.armor - lobullet, 0);
                a.armor = Math.max(a.armor - lorocket, 0);
                // Pierce structure
                if (a.armor === 0) {
                  a.structure = Math.max(a.structure - lolaser2, 0);
                  a.structure = Math.max(a.structure - lobullet2, 0);
                  a.structure = Math.max(a.structure - lorocket2, 0);
                }
              }
            } else if (a.armor > 0) {
              if (ds.laser * this.rate("laser", "armor") > a.armor) {
                lolaser =
                  Math.max(
                    ds.laser * this.rate("laser", "armor") - a.armor,
                    0
                  ) * this.pierceratearmor;
              } else {
                lolaser = 0;
              }
              if (ds.bullet * this.rate("bullet", "armor") > a.armor) {
                lobullet =
                  Math.max(
                    ds.bullet * this.rate("bullet", "armor") - a.armor,
                    0
                  ) * this.pierceratearmor;
              } else {
                lobullet = 0;
              }
              if (ds.rocket * this.rate("rocket", "armor") > a.armor) {
                lorocket =
                  Math.max(
                    ds.rocket * this.rate("rocket", "armor") - a.armor,
                    0
                  ) * this.pierceratearmor;
              } else {
                lorocket = 0;
              }
              a.armor = Math.max(
                a.armor - ds.laser * this.rate("laser", "armor"),
                0
              );
              a.armor = Math.max(
                a.armor - ds.bullet * this.rate("bullet", "armor"),
                0
              );
              a.armor = Math.max(
                a.armor - ds.rocket * this.rate("rocket", "armor"),
                0
              );
              // Pierce structure
              if (d.armor === 0) {
                a.structure = Math.max(a.structure - lolaser, 0);
                a.structure = Math.max(a.structure - lobullet, 0);
                a.structure = Math.max(a.structure - lorocket, 0);
              }
            } else if (a.structure > 0) {
              a.structure = Math.max(
                a.structure - ds.laser * this.rate("laser", "structure"),
                0
              );
              a.structure = Math.max(
                a.structure - ds.bullet * this.rate("bullet", "structure"),
                0
              );
              a.structure = Math.max(
                a.structure - ds.rocket * this.rate("rocket", "structure"),
                0
              );
            }
            if (d.structure !== "undefined") {
              this.attackers[attackerIndex].survivor = Math.ceil(
                a.structure / aShip[0].structure
              );
            }
            this.attackers[attackerIndex].laser =
              aShip[0].laser * this.attackers[attackerIndex].survivor;
            this.attackers[attackerIndex].bullet =
              aShip[0].bullet * this.attackers[attackerIndex].survivor;
            this.attackers[attackerIndex].rocket =
              aShip[0].rocket * this.attackers[attackerIndex].survivor;
            this.attackers[attackerIndex].structure = a.structure;
            this.attackers[attackerIndex].armor = a.armor;
            if (this.attackers[attackerIndex].armor !== 0) {
              this.attackers[attackerIndex].armor = Math.min(
                this.attackers[attackerIndex].armor +
                  aShip[0].armor *
                    Math.max(
                      this.attackerArmorRep - this.armorregreduce * this.round,
                      0
                    ) *
                    this.attackers[attackerIndex].survivor,
                aShip[0].armor * this.attackers[attackerIndex].survivor
              );
            }
            this.attackers[attackerIndex].shield = a.shield;
            if (this.attackers[attackerIndex].shield !== 0) {
              this.attackers[attackerIndex].shield = Math.min(
                this.attackers[attackerIndex].shield +
                  aShip[0].shield *
                    Math.max(
                      this.attackerShieldRegen -
                        this.shieldregreduce * this.round,
                      0
                    ) *
                    this.attackers[attackerIndex].survivor,
                aShip[0].shield * this.attackers[attackerIndex].survivor
              );
            }
          }
        }

        if (this.defenders[defenderIndex].structure === 0) {
          this.result = "Attacker destroyed a ship group";
        }
        if (this.attackers[attackerIndex].structure === 0) {
          this.result = "Defender destroyed a ship group";
        }

        // Move throught Shooters
        if (this.turn === "Attacker") {
          if (this.currentAttackerShooter !== this.slots) {
            this.currentAttackerShooter = this.currentAttackerShooter + 1;
          }

          while (
            this.attackers[this.currentAttackerShooter].survivor === 0 &&
            this.currentAttackerShooter !== this.slots
          ) {
            this.currentAttackerShooter = this.currentAttackerShooter + 1;
          }

          if (
            this.currentAttackerShooter === this.slots &&
            this.currentDefenderShooter === this.slots
          ) {
            this.currentAttackerShooter = this.currentAttacker;
            this.currentDefenderShooter = this.currentDefender;
          }
        }
        if (this.turn === "Defender") {
          if (this.currentDefenderShooter !== this.slots) {
            this.currentDefenderShooter = this.currentDefenderShooter + 1;
          }

          while (
            this.defenders[this.currentDefenderShooter].survivor === 0 &&
            this.currentDefenderShooter !== this.slots
          ) {
            this.currentDefenderShooter = this.currentDefenderShooter + 1;
          }

          if (
            this.currentDefenderShooter === this.slots &&
            this.currentAttackerShooter === this.slots
          ) {
            this.currentDefenderShooter = this.currentDefender;
            this.currentAttackerShooter = this.currentAttacker;
          }
        }

        if (this.turn === "Attacker") {
          this.turn = "Defender";
        } else if (this.turn === "Defender") {
          this.turn = "Attacker";
        }
        //Skip if already through all own shooters
        if (this.currentAttackerShooter === this.slots) {
          this.turn = "Defender";
        }
        if (this.currentDefenderShooter === this.slots) {
          this.turn = "Attacker";
        }
        this.round = this.round + 1;
      }
    }
  },
  beforeDestroy() {
    clearInterval(this.interval);
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
input {
  width: 6ch;
}
select {
  width: 20ch;
}
.replay td {
  height: 24px;
}
</style>
