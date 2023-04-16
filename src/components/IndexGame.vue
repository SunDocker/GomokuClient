<template>
  <div>
    <table>
      <tr>
        <template v-if="status === 0 || status === 1 || status === -1">
          <td colspan="10">
            <input
              type="text"
              v-model="username"
              placeholder="Type your name ..."
              :disabled="status === 1 || status === -1"
              maxlength="5"
              @keydown.enter="connect"
            />
            <button
              :disabled="status === 1 || status === -1 || !username"
              @click="connect"
            >
              Match opponent
            </button>
          </td>
        </template>
        <template v-if="status === 2 || status === 3">
          <th colspan="3">
            <u v-if="role === -1">Black Player</u>
            <span v-if="role === 1">Black Player</span>
          </th>
          <td colspan="2">
            <u v-if="role === -1">{{ blackName }}</u>
            <span v-if="role === 1">{{ blackName }}</span>
          </td>
          <th colspan="3">
            <u v-if="role === 1">White Player</u>
            <span v-if="role === -1">White Player</span>
          </th>
          <td colspan="2">
            <u v-if="role === 1">{{ whiteName }}</u>
            <span v-if="role === -1">{{ whiteName }}</span>
          </td>
        </template>
      </tr>
      <tr>
        <td v-if="status === -1" colspan="10">
          Your browser doesn't support WebSocket. Try another one.
        </td>
        <td v-if="status === 0" colspan="10">No game</td>
        <td v-if="status === 1" colspan="10">Matching</td>
        <td v-if="status === 2" colspan="10">
          <span v-if="turn === -1">{{ blackName }}</span>
          <span v-if="turn === 1">{{ whiteName }}</span>
          is thinking
        </td>
        <td v-if="status === 3" colspan="10">
          <span v-if="turn === -2">{{ blackName }}</span>
          <span v-if="turn === 2">{{ whiteName }}</span>
          won
          <button @click="restart">Restart</button>
        </td>
      </tr>
      <tr v-for="(boardRow, i) in gameBoard" :key="i">
        <td v-for="(grid, j) in boardRow" :key="j + 10 * i">
          <img
            @click="play(i, j)"
            v-if="grid === 0"
            src="../assets/background.gif"
          />
          <img v-if="grid === -1" src="../assets/blackStone.gif" />
          <img v-if="grid === 1" src="../assets/whiteStone.gif" />
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: "IndexGame",
  data() {
    return {
      websock: null,
      gameBoard: [
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      ],
      blackName: "",
      whiteName: "",

      /**
       * status:
       * -1 - unsupported websocket
       *  0 - no game
       *  1 - matching
       *  2 - playing
       *  3 - game over
       */
      status: 0,

      /**
       * turn:
       * -2 - black won
       * -1 - black turn
       *  0 - no game
       *  1 - white turn
       *  2 - white won
       */
      turn: 0,

      username: "",

      /**
       * role:
       * -1 - black turn
       * 0 - no game
       * 1 - white turn
       */
      role: 0,
    };
  },
  created() {
    if (typeof WebSocket === "undefined") {
      this.status = -1;
    }
  },
  destroyed() {
    if (this.websock) {
      this.websock.close();
    }
  },
  methods: {
    connect() {
      const url = `ws://sundocker.online:9999/gomoku/${this.username}`;
      this.status = 1;
      if (this.websock) {
        this.websock.close();
      }
      this.websock = new WebSocket(url);
      this.websock.onmessage = this.webSocketOnMessage;
      this.websock.onclose = this.webSocketOnClose;
    },
    play(i, j) {
      if (this.status !== 2 && this.role !== this.turn) {
        return;
      }
      this.websock.send(
        JSON.stringify({
          type: "try",
          x: i,
          y: j,
        })
      );
    },
    restart() {
      location.reload();
    },
    webSocketOnMessage(msg) {
      const message = JSON.parse(msg.data);
      switch (message.type) {
        case "connect":
          this.role = message.role;
          if (this.role === -1) {
            this.blackName = this.username;
            this.whiteName = message.username;
          } else if (this.role === 1) {
            this.blackName = message.username;
            this.whiteName = this.username;
          }
          this.status = 2;
          this.turn = -1;
          break;
        case "play":
          this.turn = message.nextTurn;
          this.gameBoard[message.x].splice(message.y, 1, message.role);
          if (Math.abs(this.turn) > 1) {
            this.status = 3;
          }
          break;
        default:
          break;
      }
    },
    webSocketOnClose() {
      window.alert("Your opponent is missing. Try matching again");
      this.websock = null;
      this.whiteName = "";
      this.blackName = "";
      this.status = 0;
      this.turn = 0;
      this.role = 0;
      for (let i = 0; i < 10; i++) {
        for (let j = 0; j < 10; j++) {
          this.gameBoard[i][j] = 0;
        }
      }
    },
  },
};
</script>

<style>
table {
  border-top: 2px solid black;
  border-left: 2px solid black;
  border-spacing: 0;
}
td,
th {
  border-right: 2px solid black;
  border-bottom: 2px solid black;
  padding: 0;
  text-align: center;
}
img {
  vertical-align: top;
}
input {
  border: 0;
  margin-right: 20px;
}
</style>