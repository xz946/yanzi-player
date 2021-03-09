<template>
  <div id="app">
    <yanziPlayer :initVolume="initVolume" :openBtnDrag="openBtnDrag" :showOpenBtn="showOpenBtn"
      :isShowMusic.sync="isShowMusic" :btnStyle="btnStyle" :prePath="prePath" :musicList.sync="musicList"
      :zIndex="zIndex" @uploadLRC="uploadLRC" @uploadMusic="uploadMusic" @uploadMusicIcon="uploadMusicIcon">
    </yanziPlayer>
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
        zIndex: "1111", // 层级
        musicList: [], // 歌曲列表
        prePath: "", // 域名前缀
        isShowMusic: false, // 是否隐藏播放器
        initVolume: 50, // 初始化音量
        showOpenBtn: true, // 打开播放器的按钮
        openBtnDrag: true, //打开播放器的按钮是否可拖拽
        btnStyle: { //打开播放器的按钮的样式
          color: "white", // 字体颜色
          backgroundColor: "#ccc", // 背景颜色
          width: "50px", // 宽度
          height: "50px", // 高度
          left: "30px", // 左边距离
          bottom: "80px", // 下边距离
          shadowColor: "#666", // 阴影颜色
        }
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

</style>