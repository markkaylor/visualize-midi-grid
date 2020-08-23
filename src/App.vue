<template>
  <div id="app">
    <div class="container-flex">
      <div
        class="note-item"
        v-for="(note, index) in notes.sort(() => Math.random() - 0.5)"
        :key="`${note}-${index}`"
        :id="`note-number-${index}`"
      />
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
      return new Array(127);
    },
  },
  async mounted() {
    const { data } = await axios.get("http://localhost:8080/midi-raw.mid", {
      responseType: "arraybuffer",
    });
    const Player = new MidiPlayer.Player();
    Player.loadArrayBuffer(data);
    Player.play();
    Player.on("midiEvent", function(event) {
      if (event.noteName || event.velocity) {
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
  width: 900px;
  justify-content: center;
  align-content: center;
  margin: 0 auto;
  height: 100vh;
}

.note-item {
  width: 40px;
  height: 40px;
}

.container-flex {
  display: flex;
  flex-wrap: wrap;
}
</style>
