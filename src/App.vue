<template>
  <div id="app">
    <div v-for="(row, i) in notes" :key="i" class="container-flex">
      <div
        class="note-item"
        v-for="note in row"
        :key="note"
        :id="`note-number-${note}`"
      ></div>
    </div>
  </div>
</template>

<script>
import MidiPlayer from "midi-player-js";
import axios from "axios";
import info from "@/info.json";

export default {
  name: "App",
  computed: {
    notes() {
      let notes = [];
      for (let i = 0; i <= 127; i++) {
        notes.push(i);
      }
      return this.chunkArray(notes, 12).reverse();
    },
  },
  async mounted() {
    const { data } = await axios.get("http://localhost:8080/midi-raw.mid", {
      responseType: "arraybuffer",
    });
    const Player = new MidiPlayer.Player();
    // eslint-disable-next-line
    Player.loadArrayBuffer(data);
    Player.play();
    // this.drawPainting(Player);
    Player.on("midiEvent", function(event) {
      if (event.noteName || event.velocity) {
        console.log("go");
        const pitch = event.noteName.replace(/([0-9]|[-])/g, "");

        const rgb = info.noteColors[pitch];
        const alpha = event.velocity;
        const rgba = `${rgb.join(", ")}, 0.${alpha}`;
        const el = document.getElementById(`note-number-${event.noteNumber}`);

        if (event.name === "Note on") {
          el.style.backgroundColor = `rgba(${rgba})`;
        }

        if (event.name === "Note off") {
          el.style.backgroundColor = "white";
        }
      }
    });
  },
  methods: {
    drawPainting(Player) {
      const events = Player.getEvents();
      const eventList = events
        .flatMap((e) => e)
        .filter((e) => e.name === "Note on");
      for (const event of eventList) {
        if (event.name === "Note on") {
          const pitch = event.noteName.replace(/([0-9]|[-])/g, "");
          const rgb = info.noteColors[pitch];
          const alpha = event.velocity;
          const rgba = `${rgb.join(", ")}, 0.${alpha}`;
          const el = document.getElementById(`note-number-${event.noteNumber}`);

          el.style.backgroundColor = `rgba(${rgba})`;
        }
      }
    },
    chunkArray(arr, chunkCount) {
      const chunks = [];
      while (arr.length) {
        const chunk = arr.slice(0, chunkCount);
        chunks.push(chunk);
        arr = arr.slice(chunkCount);
      }
      return chunks;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: column;
  width: 500px;
  margin: 10rem auto;
  height: 100vh;
}

.note-item {
  width: 40px;
  height: 40px;
}

.container-flex {
  display: flex;
}
</style>
