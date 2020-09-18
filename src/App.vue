<template>
  <div id="app">
    <router-view />
  </div>
</template>

<style lang="less">
html,
body {
  height: 100%;
  padding: 0;
  margin: 0;
  background-color: #121212;
}
#app {
  height: 100%;
}

.video-js .vjs-control:focus,
.video-js .vjs-control:focus:before,
.video-js .vjs-control:hover:before {
  // text-shadow: none !important;
  outline: none;
}

.music_container {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  z-index: 1;
  background-color: #000;
  transition: all 0.2s ease-in-out;
  iframe {
    width: 100%;
    height: 100%;
    border: none;
  }
}
</style>
