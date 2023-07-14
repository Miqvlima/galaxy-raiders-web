<template>
  <div id="canvas">
    <div id="deep-space" />
    <div id="leaderboard" v-show="leaderboardOn">
      <p>First Place</p>
      <ul>
        <li>Score: {{ scoreTable[0].score }}</li>
        <li>Asteroids Destroyed: {{ scoreTable[0].numberAsteroidsDestroyed }}</li>
      </ul>
      <p>Second Place</p>
      <ul>
        <li>Score: {{ scoreTable[1].score }}</li>
        <li>Asteroids Destroyed: {{ scoreTable[1].numberAsteroidsDestroyed }}</li>
      </ul>
      <p>Third Place</p>
      <ul>
        <li>Score: {{ scoreTable[2].score }}</li>
        <li>Asteroids Destroyed: {{ scoreTable[2].numberAsteroidsDestroyed }}</li>
      </ul>
    </div>

    <div id="menu" v-show="openMenu">
      <button v-on:click="startGame">Start</button>
      <button v-on:click="showLeaderboard">Leaderboard</button>
      <button v-on:click="quitGame">Quit</button>
    </div>
      
    <div id="space-field">
      <SpaceObject id="spaceship" class="spaceship" :data="spaceField.ship" resolution="2" />

      <SpaceObject class="asteroid" :data="asteroid" resolution="2"
                  :key="asteroid.center"
                  v-for="asteroid in spaceField.asteroids" />

      <SpaceObject class="missile" :data="missile" resolution="2"
                  :key="missile.center"
                  v-for="missile in spaceField.missiles" />

      <SpaceObject class="explosion" :data="explosion" resolution="2"
                  :key="explosion.center"
                  v-for="explosion in spaceField.explosions" />
    </div>
  </div>
</template>


<script setup>

import Leaderboard from "../assets/Leaderboard.json"

let leaderboardOn = false;
let openMenu = true;
let playingGame = false;

export default {
  data: () => ({
    scoreTable: Leaderboard
  })
}

const startGame = () => {
  playingGame = true;
  openMenu = false;
  leaderboardOn = false;
}

const showLeaderboard = () => {
  leaderboardOn = true;
}

const quitGame = () => {
  playingGame = false;
  openMenu = false;
  leaderboardOn = false;
  $post("/quit")
}

const {
  data: spaceField,
  refresh: updateSpaceField
} = await $get("/space-field");

onMounted(() => {
  window.addEventListener("keydown", async (event) => {
    const keyToCommand = {
      "ArrowUp": "MOVE_SHIP_UP",
      "ArrowDown": "MOVE_SHIP_DOWN",
      "ArrowRight": "MOVE_SHIP_RIGHT",
      "ArrowLeft": "MOVE_SHIP_LEFT",
      "Space": "LAUNCH_MISSILE",
      "Escape": "PAUSE_GAME",
    };

    const command = keyToCommand[event.code];

    // Ignore if invalid key was pressed
    if (command === undefined) return;

    console.log(`Triggering command: ${command}`);
    await $post("/ship/commands", { command })
  });

  window.setInterval(updateSpaceField, 1000);
})
</script>

<style>

#leaderboard {
  height: 80px;
  width: 45px;
  flex-direction: column;
  align-items: center;
  left: 30px;
  position: absolute;
}

#menu {
  height: 100px;
  width: 60px;
  flex-direction: column;
  align-items: center;
  position: absolute;
}

#canvas {
  height: calc(100vh - 4rem);
  width: calc(100vw - 4rem);

  padding: 2rem;

  background-color: #36bbf5;
  overflow: hidden;

  position: relative;

  display: flex;
  justify-content: center;
  align-items: center;
}

@keyframes slide {
  0% {
    transform: translate(1px);
  }
  50% {
    transform: translate(-1px);
  }
  100% {
    transform: translate(1px);
  }
}

#deep-space {
  height: calc(100% - 4rem);
  width: calc(100% - 4rem);

  background-image: url("~/assets/space.png");
  background-origin: content-box;
  animation: slide 3s linear infinite;

  position: absolute;
  z-index: 0;
}

#space-field {
  height: calc(100% - 4rem);
  width: calc(100% - 4rem);

  position: relative;
}

.spaceship {
  background-image: url("~/assets/spaceship.png");
}

.asteroid {
  background-image: url("~/assets/asteroid.png");
}

.missile {
  background-image: url("~/assets/missile.png");
}

.explosion {
  background-image: url("~/assets/explosion.png");
}
</style>