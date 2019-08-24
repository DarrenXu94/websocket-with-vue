<template>
  <span></span>
</template>
<script>
export default {
  name: "connection-manager",
  data: function() {
    return {
      myId: "",
      allPeers: []
    };
  },
  created: function() {
    this.$socket.addEventListener("open", () => {
      console.log("Connection established client side");
      this.spinUp();
    });
    this.$socket.addEventListener("message", event => {
      this.handleMessages(event.data);
    });
  },
  methods: {
    spinUp: function() {
      let sessionId = this.checkSession();
      if (!sessionId) {
        this.createSession();
      } else {
        this.joinSession(sessionId);
      }
    },
    checkSession: function() {
      let sessionId = window.location.hash.substring(
        2,
        window.location.hash.length
      );
      return sessionId && sessionId != "" ? sessionId : false;
    },
    createSession: function() {
      this.$socket.send(JSON.stringify({ type: "create-session" }));
    },
    joinSession: function(sessionId) {
      this.$socket.send(
        JSON.stringify({
          type: "join-session",
          id: sessionId
        })
      );
    },
    handleMessages: function(msg) {
      const data = JSON.parse(msg);
      if (data.type === "session-created") {
        this.setSessionId(data.id);

        window.location.hash = data.id;
      } else if (data.type === "session-broadcast") {
        // console.log(data, "broadcast received");
        this.setMyId(data.peers.you);
        this.setMyPeers(data.peers.clients);
        this.setBroadcastGameState(data.peers);
      } else if (data.type === "state-update") {
        // console.log(data, "state update received");
        this.updateReceivedFromServer(data);
      }
    }
  },
  props: {
    setMyId: Function,
    setMyPeers: Function,
    setSessionId: Function,
    setBroadcastGameState: Function,
    updateReceivedFromServer: Function
  }
};
</script>