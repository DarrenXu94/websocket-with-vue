<template>
  <div>
    <ConnectionManager
      :setMyId="setMyId"
      :setMyPeers="setMyPeers"
      :setSessionId="setSessionId"
      :setBroadcastGameState="setBroadcastGameState"
      :updateReceivedFromServer="updateReceivedFromServer"
    />
    My Session ID: {{mySession}}
    My ID: {{myId}}
    <CanvasGame :gameState="gameState" :myId="myId" :setMyGameState="setMyGameState" />
    <div id="others"></div>
  </div>
</template>
<script>
import ConnectionManager from "@/components/ConnectionManager.vue";
import CanvasGame from "@/components/CanvasGame.vue";

export default {
  name: "game-manager",
  components: {
    ConnectionManager,
    CanvasGame
  },
  data: function() {
    return {
      myId: "",
      allPeers: [],
      mySession: "",
      gameState: {}
    };
  },
  methods: {
    setSessionId(id) {
      this.mySession = id;
    },
    setMyId(id) {
      this.myId = id;
    },
    setMyPeers(peers) {
      this.allPeers = peers;
    },
    setMyGameState: function(state) {
      this.gameState.myState = state;
    },
    setBroadcastGameState: function(state) {
      let canvas = document.getElementById("others");

      let otherCanvases = state.clients.filter(x => x.id != this.myId);

      otherCanvases.forEach(function(element) {
        let exists = document.getElementById(element.id);
        if (exists) {
          console.log("Exists");
        } else {
          let p = document.createElement("canvas");
          p.setAttribute("id", element.id);
          p.setAttribute("height", 300);
          p.setAttribute("width", 300);
          // let textnode = document.createTextNode(element.id);
          // p.appendChild(textnode);

          canvas.appendChild(p);
        }
      });
    },
    updateReceivedFromServer: function(state) {
      // Draw the new thing
      let docoCanvas = document.getElementById(state.clientId);
      let canvasContext = docoCanvas.getContext("2d");
      canvasContext.scale(100, 100);
      canvasContext.fillStyle = "#000";
      canvasContext.fillRect(0, 0, docoCanvas.width, docoCanvas.height);

      let offset = { x: 0, y: 0 };

      state.state.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            canvasContext.fillStyle = "#FF0D72";
            canvasContext.fillRect(x + offset.x, y + offset.y, 1, 1);
          }
        });
      });
    }
  }
};
</script>