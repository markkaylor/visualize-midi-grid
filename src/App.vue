<template>
  <div id="app">
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(0, 12)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(12, 24)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(24, 36)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(36, 48)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(48, 60)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(60, 72)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(72, 84)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(84, 96)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(96, 108)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(108, 120)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
    </div>
    <div class="container-flex">
      <div
        class="note-item"
        v-for="note in notes.slice(120, 128)"
        :key="note"
        :id="`note-number-${note}`"
      >
        {{ note }}
      </div>
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
    // octave -1- 9
    // note 0 - 119
    notes() {
      let notes = [];
      for (let i = 0; i <= 127; i++) {
        notes.push(i);
      }

      return notes;
    },
  },
  async mounted() {
    const { data } = await axios.get("http://localhost:8080/midi-raw.mid", {
      responseType: "arraybuffer",
    });
    // Initialize player and register event handler
    const Player = new MidiPlayer.Player();

    // Load a MIDI file
    Player.loadArrayBuffer(data);
    console.log(data);
    Player.on("midiEvent", function(event) {
      // Do something when a MIDI event is fired.
      // (this is the same as passing a function to MidiPlayer.Player() when instantiating.

      if (event.noteName || event.velocity) {
        console.log("gogogogog");
        const pitch = event.noteName.replace(/([0-9]|[-])/g, "");

        const rgb = info.noteColors[pitch];
        const alpha = event.velocity;
        const rgba = `${rgb.join(", ")}, 0.${alpha}`;
        const el = document.getElementById(`note-number-${event.noteNumber}`);

        el.style.backgroundColor = `rgba(${rgba})`;

        setTimeout(() => {
          el.style.backgroundColor = "white";
        }, (60000 / 120) * event.delta);

        // eslint-disable-next-line
        console.log(Player);
        // [note on] set color to assigned note color with alpha equal to velocity
      }
    });

    Player.play();
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
  width: 900px;
  justify-content: center;
  align-content: center;
  margin: 5rem auto;
}

.note-item {
  width: 40px;
  height: 40px;
}

.container-flex {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
}
</style>
