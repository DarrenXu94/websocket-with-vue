<template>
  <div>
    <canvas :id="myId" width="300" height="300" ref="canvasref" />
  </div>
</template>
<script>
export default {
  name: "canvas-game",
  data: function() {
    return {
      canvas: null,
      context: null,
      randArray: []
    };
  },
  created: function() {
    // this.drawCanvas();
    for (let i = 0; i < 3; i++) {
      let row = [];
      for (let j = 0; j < 3; j++) {
        row.push(this.genRandom());
      }
      this.randArray.push(row);
    }

    this.setMyGameState(this.randArray);
  },
  methods: {
    genRandom: function() {
      return Math.floor(Math.random() * Math.floor(2));
    },
    drawCanvas: function() {
      this.context.fillStyle = "#000";
      this.context.fillRect(0, 0, this.canvas.width, this.canvas.height);

      this.drawMatrix(this.randArray, { x: 0, y: 0 });
    },
    run: function() {
      let lastTime = 0;
      this._update = (time = 0) => {
        const deltaTime = time - lastTime;
        lastTime = time;

        this.drawCanvas();
        requestAnimationFrame(this._update);
      };
    },
    drawMatrix(matrix, offset) {
      matrix.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            this.context.fillStyle = "#FF0D72";
            this.context.fillRect(x + offset.x, y + offset.y, 1, 1);
          }
        });
      });
    }
    // update: function(){

    // }
  },
  watch: {
    myId: function(val) {
      this.canvas = this.$refs.canvasref;
      this.context = this.canvas.getContext("2d");
      this.context.scale(100, 100);

      this.run();

      this.$socket.send(
        JSON.stringify({
          type: "state-update",
          state: this.randArray
        })
      );
    }
  },
  props: {
    myId: String,
    gameState: Object,
    setMyGameState: Function
  }
};
</script>