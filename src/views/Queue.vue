<template>
  <v-container fluid>
    <v-row class="fill-height" align="center" justify="center">
      <!-- Left Side: Join/Leave Queue Buttons -->
      <v-col cols="4">
        <v-btn
          color="primary"
          block
          @click="addPlayerToQueue()"
          :disabled="inQueue"
        >
          <!-- This will switch between showing "Leave Queue" and the timer -->
          {{ inQueue ? elapsedTime : "Join Queue" }}
        </v-btn>
        <v-btn color="error" block @click="leaveQueue()" :disabled="!inQueue">
          Leave Queue
        </v-btn>
        <v-btn color="success" block @click="acceptMatch()" v-if="accept_ready">
          Accept
        </v-btn>
      </v-col>

      <!-- Middle: Logged In Player Details -->
      <v-col cols="4" class="d-flex flex-column align-center">
        <v-avatar size="120" class="mb-2">
          <img :src="user.large_image" :alt="user.name" />
        </v-avatar>
        <div>{{ user.name }}</div>
      </v-col>

      <!-- Right Side: Players In Queue -->
      <v-col cols="4">
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
                  <img :src="player.avatar" :alt="player.username" />
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title>{{ player.username }}</v-list-item-title>
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
      queueJoinTime: null,
      timer: null,
      accept_ready: false
    };
  },
  computed: {
    elapsedTime() {
      if (!this.queueJoinTime) return "";
      const now = Date.now();
      const elapsedMilliseconds = now - new Date(this.queueJoinTime);
      const seconds = Math.floor((elapsedMilliseconds / 1000) % 60);
      const minutes = Math.floor((elapsedMilliseconds / (1000 * 60)) % 60);
      const hours = Math.floor((elapsedMilliseconds / (1000 * 60 * 60)) % 24);
      return `${hours
        .toString()
        .padStart(2, "0")}:${minutes
        .toString()
        .padStart(2, "0")}:${seconds.toString().padStart(2, "0")}`;
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
    async setupPage(message) {
      let playerFound = false;
      let qPlayer = null;
      this.players = message;
      this.players.forEach(player => {
        if (player.steam_id === this.user.steam_id) {
          playerFound = true;
          qPlayer = player;
        }
      });
      if (playerFound) {
        this.inQueue = true;
        this.queueJoinTime = qPlayer.date;
      } else {
        this.inQueue = false;
        this.queueJoinTime = null;
      }
    }
  },
  async mounted() {
    this.setupQueue();
    this.user = await this.IsLoggedIn();
  }
};
</script>
