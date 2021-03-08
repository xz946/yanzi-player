<template>
  <div id="app">
    <yanziPlayer :initVolume="initVolume" :isShowMusic.sync="isShowMusic" :prePath="prePath" :musicList.sync="musicList" :zIndex="zIndex" @uploadLRC="uploadLRC" @uploadMusic="uploadMusic" @uploadMusicIcon="uploadMusicIcon"></yanziPlayer>
  </div>
</template>

<script>
import yanziPlayer from "./components/yanzi_player.vue";

export default {
  name: "App",
  components: {
    yanziPlayer
  },
  data() {
    return {
      zIndex: "1111",
      musicList: [
        // {
        //   author: "CG",
        //   image: "",
        //   lrc: "",
        //   name: "文爱",
        //   src: "http://192.168.1.238:9995/music/CG - 文爱.mp3"
        // },
        // {
        //   author: "爱乐团王超",
        //   image: "",
        //   lrc: "",
        //   name: "死而无憾",
        //   src: "http://192.168.1.238:9995/music/爱乐团王超 - 死而无憾.mp3"
        // },
      ],
      prePath: "",
      isShowMusic: false,
      initVolume: 50
    };
  },
  methods: {
    uploadLRC(music) {
      console.log("uploadLRC", music);
    },
    uploadMusic(music) {
      console.log(music);
    },
    uploadMusicIcon(music) {
      console.log(music);
    }
  }
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
}
</style>