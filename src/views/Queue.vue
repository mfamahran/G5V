<template>
  <v-container fluid>
    <v-row class="fill-height" justify="center">
      <v-col cols="12" md="8">
        <!-- Left Side: Match Room, Join/Leave Queue Buttons -->
        <v-row>
          <v-col cols="12" md="6">
            <!-- Join/Leave Queue Buttons -->
            <v-btn
              color="primary"
              block
              @click="addPlayerToQueue()"
              :disabled="inQueue"
              >Join Queue</v-btn
            >
            <v-btn
              color="error"
              block
              @click="leaveQueue()"
              :disabled="!inQueue || matchFound"
              >Leave Queue</v-btn
            >
            <v-btn
              color="success"
              block
              @click="acceptMatch()"
              v-if="accept_ready"
              >Accept</v-btn
            >
          </v-col>

          <!-- Middle: Logged In Player Details -->
          <v-col cols="12" md="6" class="d-flex flex-column align-center">
            <v-avatar size="120" class="mb-2">
              <img :src="user.large_image" :alt="user.name" />
            </v-avatar>
            <div>{{ user.name }}</div>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="12" v-if="matchFound">
            <!-- Match Room -->
            <v-card elevation="2" class="pa-3 mb-3">
              <v-card-title class="justify-center">Match Room</v-card-title>
              <v-row>
                <v-col cols="26">
                  <div class="text-center" v-if="team1.captain">Team 1</div>
                  <v-avatar size="100" class="mb-2">
                    <img
                      :src="team1.captain.medium_image"
                      :alt="team1.captain.name"
                    />
                  </v-avatar>
                  <v-list dense v-if="team1.players">
                    <v-list-item
                      v-for="player in team1.players"
                      :key="player.steam_id"
                    >
                      <v-list-item-avatar>
                        <img :src="player.medium_image" :alt="player.name" />
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ player.name }}</v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list>
                </v-col>
                <v-col cols="6">
                  <div class="text-center" v-if="team2.captain">Team 2</div>
                  <v-avatar size="100" class="mb-2">
                    <img
                      :src="team2.captain.large_image"
                      :alt="team1.captain.name"
                    />
                  </v-avatar>
                  <v-list dense v-if="team2.players">
                    <v-list-item
                      v-for="player in team2.players"
                      :key="player.steam_id"
                    >
                      <v-list-item-avatar>
                        <img :src="player.medium_image" :alt="player.name" />
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ player.name }}</v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list>
                </v-col>
              </v-row>
              <v-row justify="center" v-if="turn == 'team1' || turn == 'team2'">
                <v-btn
                  small
                  color="info"
                  @click="pickPlayers(player)"
                  v-for="player in matchedComputed"
                  :key="player.steam_id"
                  :disabled="
                    checkPlayer(player) || checkCaptain() || checkTurn()
                  "
                >
                  Pick {{ player.name }}
                </v-btn>
              </v-row>
              <v-row
                justify="center"
                v-if="turn == 'map_pick_team1' || turn == 'map_pick_team2'"
              >
                <v-btn
                  small
                  color="info"
                  @click="banMap(map)"
                  v-for="map in maps"
                  :key="map"
                  :disabled="checkCaptain() || checkTurn()"
                >
                  Ban {{ map }}
                </v-btn>
              </v-row>
            </v-card>
          </v-col>
        </v-row>
      </v-col>

      <!-- Right Side: Players In Queue -->
      <v-col cols="12" md="4">
        <v-card elevation="2" class="pa-3">
          <v-card-title class="justify-center">Players In Queue</v-card-title>
          <v-card-text>
            <v-list>
              <v-list-item
                v-for="player in players"
                :key="player.steam_id"
                class="my-2"
              >
                <v-list-item-avatar>
                  <img :src="player.medium_image" :alt="player.name" />
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title>{{ player.name }}</v-list-item-title>
                  <v-list-item-subtitle>{{ player.rank }}</v-list-item-subtitle>
                </v-list-item-content>
                <v-list-item-action>
                  <v-icon>mdi-trophy</v-icon> {{ player.score }}
                </v-list-item-action>
              </v-list-item>
            </v-list>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "Queue",
  data() {
    return {
      user: {
        admin: false,
        steam_id: "",
        id: -1,
        super_admin: false,
        name: "",
        small_image: "",
        medium_image: "",
        large_image: ""
      }, // should be object from JSON response
      inQueue: false,
      players: [],
      timer: null,
      accept_ready: false,
      team1: {
        captain: null,
        players: []
      },
      team2: {
        captain: null,
        players: []
      },
      queue: null,
      matchFound: false,
      turn: "team1",
      playersMatch: [],
      maps: []
    };
  },
  computed: {
    matchedComputed() {
      const captains = [
        this.team1.captain.steam_id,
        this.team2.captain.steam_id
      ];
      // remove captains from playersMatch array
      const filterd = this.playersMatch.filter(
        player => captains.indexOf(player.steam_id) === -1
      );
      return filterd;
    }
  },
  // Example Vue component method
  methods: {
    async leaveQueue() {
      try {
        const message = await this.LeaveQueue();
        console.log(message);
      } catch (err) {
        console.log(err);
      }
    },
    async addPlayerToQueue() {
      try {
        const message = await this.JoinQueue();
        console.log(message);
      } catch (err) {
        console.log(err);
      }
    },
    async pickPlayers(player) {
      try {
        const message = await this.PickPlayer({
          player: player
        });
        console.log(message);
      } catch (err) {
        console.log(err);
      }
    },
    async banMap(map) {
      try {
        this.queue.maps = this.maps;
        const message = await this.BanMap({
          map: map
        });
        console.log(message);
      } catch (err) {
        console.log(err);
      }
    },
    async setupQueue() {
      try {
        let sseClient = await this.GetQueue();
        await sseClient.connect();
        await sseClient.on("queue", async message => {
          try {
            await this.setupPage(message);
          } catch (error) {
            console.error(
              "Error retrieving information from matches event stream. ",
              error
            );
          }
        });
        return;
      } catch (ignored) {
        return;
      }
    },
    checkInTeam(player) {
      console.log("checkInTeam",  Object.values(this.team1.players).indexOf(player) !== -1 ||
        Object.values(this.team2.players).indexOf(player) !== -1);
      return (
        Object.values(this.team1.players).indexOf(player) !== -1 ||
        Object.values(this.team2.players).indexOf(player) !== -1
      );
    },
    checkCaptain() {
      console.log(
        "checkCaptain",
        this.user.steam_id !== this.team1.captain.steam_id &&
          this.user.steam_id !== this.team2.captain.steam_id
      );
      return (
        this.user.steam_id !== this.team1.captain.steam_id &&
        this.user.steam_id !== this.team2.captain.steam_id
      );
    },
    checkPlayer(player) {
      let playerFound = false;
      this.team1.players.forEach(element => {
        if (element.steam_id === player.steam_id) {
          console.log("checkPlayer", true);
          playerFound = true;
        }
      });

      this.team2.players.forEach(element => {
        if (element.steam_id === player.steam_id) {
          console.log("checkPlayer", true);
          playerFound = true;
        }
      });

      console.log("checkPlayer", false);
      return playerFound;
    },
    checkTurn() {
      if (this.user.steam_id === this.team1.captain.steam_id) {
        if (this.turn === "team1" || this.turn === "map_pick_team1") {
          console.log("checkTurn", false);
          return false;
        }
      } else if (this.user.steam_id === this.team2.captain.steam_id) {
        if (this.turn === "team2" || this.turn === "map_pick_team2") {
          console.log("checkTurn", false);
          return false;
        }
      }
      console.log("checkTurn", true);
      return true;
    },
    async setupPage(message) {
      let playerFound = false;
      let qPlayer = null;
      this.players = message.players;
      this.playersMatch = message.playersMatch;
      this.team1.captain = message.team1Captain;
      this.team2.captain = message.team2Captain;
      this.queue = message;
      if (message.turn === undefined) {
        this.turn = "team1";
      } else {
        this.turn = message.turn;
      }
      if (message.maps === undefined) {
        this.maps = [];
      } else {
        this.maps = message.maps;
      }
      if (message.team1 === undefined) {
        this.team1.players = [];
      } else {
        this.team1.players = message.team1;
      }
      if (message.team2 === undefined) {
        this.team2.players = [];
      } else {
        this.team2.players = message.team2;
      }
      if (message.turn === "done") {
        this.$router.push({ name: `Match`, params: { id: message.matchId } });
      }
      this.matchFound = message.matchFound;
      console.log(message);
      if (!this.matchFound) {
        this.players.forEach(player => {
          if (player.steam_id === this.user.steam_id) {
            playerFound = true;
            qPlayer = player;
          }
        });
      } else {
        this.playersMatch.forEach(player => {
          if (player.steam_id === this.user.steam_id) {
            playerFound = true;
            qPlayer = player;
          }
        });
      }

      if (playerFound) {
        this.inQueue = true;
        this.queueJoinTime = qPlayer.date;
      } else {
        this.inQueue = false;
        this.queueJoinTime = null;
      }
      this.$forceUpdate();
    }
  },
  async mounted() {
    this.setupQueue();
    this.user = await this.IsLoggedIn();
  }
};
</script>
