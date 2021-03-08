<template>
  <div id="yanzi-player-box">

    <div v-if="showOpenBtn" @click="openPlayer" :class="{ openButton: isPlay }" id="openButton" class="iconfont">
      &#xe66c;</div>

    <div id="yanzi-player" :class="{ shoyanziist: isShoyanziist, showMusic: isShowMusic }" class="yanzi-white"
      :style="'z-index:' + zIndex">

      <audio ref="yanziAudio111" id="yanzi-audio" @canplay="getDuration" crossorigin="anonymous"></audio>

      <div class="listAndLrc">
        <!-- 歌曲列表 -->
        <div v-show="isShoyanziistAndLrc" class="yanzi-playerList">
          <!-- 滚动到顶部 -->
          <span class="focusMusicItem goTop iconfont" @click="goTop">&#xe624;</span>
          <!-- 当前播放的歌曲，滚动到到可见 -->
          <span class="focusMusicItem iconfont" @click="focusMusic">&#xe6d4;</span>
          <div class="listTools">
            <div class="searchBox">
              <input @change="startSearchMusic" id="searchMusic" type="text" v-model="searchMusic"
                placeholder="搜索歌名、歌手" />
              <span class="iconfont" @click="startSearchMusic">&#xe65d;</span>
            </div>

            <div @click="choiceMusicFile" class="yanzi-btn">上传歌曲</div>
            <input id="uploadMusicFile" ref="uploadMusicFile" accept=".mp3" type="file" style="display:none;"
              @change="selectMusicFile" />
          </div>
          <div class="musicListBox">
            <div @click="selectedListMusic(index)" class="musicItem" v-for="(item, index) in filterList" :key="index">
              <div class="musicName" :class="{ activeColor: musicIndex == index }"
                v-html="item.author + ' - ' + item.name"></div>
            </div>
          </div>
        </div>

        <div v-show="isShowYesOrNo" class="yanzi-btn-group" style="position:absolute;top:10px;right:30px;z-index:1003;">
          <div @click="uploadLRC" class="yanzi-btn yanzi-btn-sm">保存</div>
          <div @click="clearLrc" class="yanzi-btn yanzi-btn-sm yanzi-bg-default">
            取消
          </div>
        </div>

        <!-- 歌词 -->
        <div v-if="!isShoyanziistAndLrc" class="yanzi-lrc">
          <div class="tispNull" v-show="!filterList[musicIndex].lrc">
            此歌曲没有歌词哦~
            <div @click="choiceImg" class="yanzi-btn yanzi-btn-mini">
              上传歌词
            </div>
            <input id="lrcFileInput" type="file" ref="filElem" accept=".lrc" style="display:none;"
              @change="selectLrcFile" />
          </div>
          <div style="height:170px;" v-show="filterList[musicIndex].lrc"></div>
          <div class="lrcItems" :class="{ lrcActiveStyle: lrcColorIndex == index }"
            :lrc-active="currentTimeNumber == index ? 'Y' : ''" v-for="(item, index) in lrcArray" :key="index">
            {{ item }}
          </div>
          <div style="height:200px;" v-show="filterList[musicIndex].lrc"></div>
        </div>
      </div>

      <div class="yanziAudio-main">
        <!-- 关闭歌曲列表 -->
        <span id="colseMusic" @click="closeList" class="iconfont">&#xe623;</span>

        <div class="mainHead">
          <!-- 歌手头像 -->
          <div class="yanziAudio-icon" :class="{ pausedLoop: !isPlay }">
            <!-- <img src="../assets/image/xz.jpg" /> -->
            <img v-if="filterList && filterList.length && filterList[musicIndex].image"
              :src="filterList[musicIndex].image" />
            <img v-else src="../assets/music.png" />
          </div>
        </div>

        <div class="tools">
          <span class="mainButton">
            <!-- 停止 -->
            <span @click="stop" class="iconfont">&#xe612;</span>
            <!-- 上一曲 -->
            <span @click="pre" class="iconfont">&#xe800;</span>

            <!-- 播放暂停 -->

            <span @click="playAndPause" v-if="!isPlay" class="iconfont"
              style="font-size:30px !important;">&#xe60a;</span>
            <span @click="playAndPause" v-if="isPlay" class="iconfont"
              style="font-size:30px !important;color:#1989fa;">&#xe60b;</span>

            <!-- 下一曲 -->
            <span @click="next()" class="iconfont">&#xe7ff;</span>

            <!-- 重新播放本曲目 -->
            <span @click="reset" class="iconfont">&#xe67d;</span>
          </span>

          <span class="btn2Block">
            <!-- 随机播放 -->
            <span @click="openAndCloseRandom" v-show="isRandom" class="iconfont"
              style="font-size:16px !important;">&#xe6d8;</span>

            <!-- 顺序播放 -->
            <span @click="openAndCloseRandom" v-show="!isRandom" class="iconfont">&#xe67b;</span>

            <!-- 单曲循环 -->
            <span @click="isSingleLoop = !isSingleLoop" v-show="isSingleLoop" class="iconfont">&#xe6ae;</span>

            <!-- 全部循环 -->
            <span @click="isSingleLoop = !isSingleLoop" v-show="!isSingleLoop" class="iconfont"
              style="font-size:16px !important;vertical-align: text-top;">&#xe61f;</span>

            <span class="iconfont volumeIcon">&#xe606;
              <div class="volumeBlock">
                <!-- 音量 -->
                <input @change="changeVolume" id="volume" type="range" orient="vertical" v-model="volumeValue" /><span
                  id="volumeValueNumber" style="">{{ volumeValue }}</span>
              </div>
            </span>
          </span>
        </div>

        <!-- 画布 -->
        <div id="canvasBlock">
          <canvas id="canvas" width="480" height="80"></canvas>
        </div>
        <div class="tools2">
          <!-- 当前播放的歌名 -->
          <div class="musicName" :class="{ loopMusicName: isPlay }">
            {{ returnMusicName() }}
          </div>
          <div class="listAndClose">
            <span class="speed">
              <span>{{ currentTime }}</span>
              <input @change="setCurrentTime" type="range" :max="allTimeNumber" min="0" step="1"
                v-model="currentTimeNumber" />
              <span>{{ allTime }}</span>
            </span>

            <!-- 显示歌词 -->
            <span style="float:right;" @click="shoyanziistAndLrc(false)" class="iconfont" :style="
              !isShoyanziistAndLrc && isShoyanziist
                ? 'color: #1989fa !important;'
                : ''
            ">&#xe727;</span>

            <!-- 显示歌曲列表 -->
            <span style="float:right;" @click="shoyanziistAndLrc(true)" class="iconfont" :style="
              isShoyanziistAndLrc && isShoyanziist
                ? 'color: #1989fa !important;'
                : ''
            ">&#xe657;</span>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
  import $ from "jquery"

  export default {
    name: "yanzi-player",
    data() {
      return {
        currentMusicName: "未播放...",
        searchMusic: "", //搜索的歌名
        yanziAudio: {}, //音频组件
        musicIndex: 0, //第几个 mp3
        isPlay: false, //是否在播放
        isMuted: false, //是否静音
        volumeValue: 50, //音量
        isShoyanziist: false, //是否显示歌曲和歌词列表
        isShoyanziistAndLrc: true, //显示歌曲列表还是歌词
        filterList: [], // 实时使用的歌曲集合
        isBuildCanvas: false, //是否调用画布
        isRandom: false, //是否随机播放
        isSingleLoop: false, //是否单曲循环
        currentTime: "00:00", //当前时长
        allTime: "00:00", //总时长
        currentTimeNumber: 0, //当前时长
        allTimeNumber: 0, //总时长
        lrcColorIndex: 0, //歌词变颜色
        lrcArray: null, //当前歌曲的歌词数组
        indexList: [], //下标数组
        lrcFile: {}, //可上传的 lrc 文件
        isShowYesOrNo: false // 是否显示上传歌词的 确定、取消按钮
      };
    },
    watch: {
      musicList: {
        handler: function (newVal) {
          if (!newVal || !newVal.length) {
            this.getMusicList()
          } else {
            this.filterList = newVal;
          }
        },
        deep: true, //true 深度监听,
        immediate: true //侦听开始之后被立即调用
      }
    },
    props: {
      zIndex: {
        type: String,
        default: "1111"
      },
      musicList: {
        type: Array,
        default: function () {
          return [];
        }
      },
      prePath: {
        type: String,
        default: ""
      },
      isShowMusic: {
        //是否显示播放器
        type: Boolean,
        default: false
      },
      initVolume: {
        //初始化音量
        type: Number,
        default: 50
      },
      showOpenBtn: {
        type: Boolean,
        default: true
      }
    },
    created() {
      this.filterList = this.musicList;
    },
    mounted() {



      this.initMethods();

      this.yanziAudio = document.getElementById("yanzi-audio");
      this.volumeValue = this.initVolume ? this.initVolume : 50;
      this.yanziAudio.volume = this.volumeValue / 100;

      if (this.filterList.length)
        this.yanziAudio.src = this.prePath + this.filterList[0].src;

      // 对audio的一系列监听
      this.watchAudio();
    },

    methods: {
      // 获取默认歌曲列表
      getMusicList() {
        let that = this;

        $.ajax({
          type: "get",
          url: "https://xzlovecyy.com/server/music/getMusicList",
          dataType: "JSON",
          success: function (res) {
            console.log("默认歌曲列表获取成功")
            that.$emit('update:musicList', res.data);
            that.filterList = res.data;
          },
          error: function (err) {
            alert(err)
          }
        })
      },
      // 监听audio
      watchAudio() {
        let that = this;

        // 监听 audio 时间变化
        this.yanziAudio.addEventListener("timeupdate", function () {
          var time = Math.floor(this.currentTime);
          that.currentTime = that.timeFormat(time);
          that.currentTimeNumber = time;

          if (that.currentTimeNumber == that.allTimeNumber) {
            console.log("播放结束，跳转下一曲");
            that.autoNext();
            return false;
          }
          try {
            let obj = that.lrcArray[time];
            if (obj != undefined) {
              that.g = obj;
              // 当前歌词下标
              that.lrcColorIndex = time;
              // 滚动条滚动到当前歌词
              that.focusLRC();
            }
          } catch (e) {
            console.log("计算歌词出错");
          }
        });

        this.yanziAudio.addEventListener("playing", function () { //播放状态Doing
          that.isPlay = true;
        });
        this.yanziAudio.addEventListener("pause", function () { //暂停状态Doing
          that.isPlay = false;
        });

      },
      // 点击，开始选择本地文件
      choiceMusicFile() {
        this.$refs.uploadMusicFile.dispatchEvent(new MouseEvent("click"));
      },
      // 选中本地文件之后，开始调用父组件上传歌曲
      selectMusicFile() {
        const inputFile = this.$refs.uploadMusicFile.files[0];
        $("#uploadMusicFile").val(""); //置空文件域
        var data = {
          file: inputFile
        };
        this.$emit("uploadMusic", data);
      },
      //隐藏确定取消按钮
      clearLrc() {
        this.isShowYesOrNo = false;
        this.filterList[this.musicIndex].lrc = "";
        this.lrcArray = [];
      },
      // 点击触发选择文件
      choiceImg() {
        this.$refs.filElem.dispatchEvent(new MouseEvent("click"));
      },
      // 选择 LRC 文件
      selectLrcFile() {
        var that = this;
        const inputFile = this.$refs.filElem.files[0];
        $("#lrcFileInput").val(""); //置空文件域

        if (inputFile) {
          that.isShowYesOrNo = true;

          // 放在页面变量中，用于上传
          that.lrcFile = inputFile;

          // 开始读取 LRC 文件的内容
          var reader = new FileReader();
          // 读取纯文本文件,且编码格式为 utf-8
          reader.readAsText(inputFile, "UTF-8");
          reader.onload = function (e) {
            let lrcContent = e.target.result;
            that.filterList[that.musicIndex].lrc = lrcContent;
            that.lrcArray = that.getLRC();
            that.focusLRC();
          };
        } else {
          return;
        }
      },
      // 上传 LRC文件
      uploadLRC() {
        var mp3 = this.filterList[this.musicIndex].src;
        var name = mp3.substring(mp3.lastIndexOf("/") + 1, mp3.lastIndexOf("."));
        var fileName = name + ".lrc";
        var content = this.filterList[this.musicIndex].lrc;
        var data = {
          fileName: fileName,
          content: content
        };
        this.$emit("uploadLRC", data);
        this.isShowYesOrNo = false;
      },
      // 获取当前歌词
      getLRC() {
        var lyrics = this.filterList[this.musicIndex].lrc.split("\n");
        var lrcArray = {};
        for (var i = 0; i < lyrics.length; i++) {
          var lyric = decodeURIComponent(lyrics[i]);
          /* eslint-disable */
          var timeReg = /\[\d*:\d*((\.|\:)\d*)*\]/g;
          /* eslint-disable */
          var timeRegExpArr = lyric.match(timeReg);
          if (!timeRegExpArr) continue;
          var clause = lyric.replace(timeReg, "");
          for (var k = 0, h = timeRegExpArr.length; k < h; k++) {
            var t = timeRegExpArr[k];
            /* eslint-disable */
            var min = Number(String(t.match(/\[\d*/i)).slice(1));
            var sec = Number(String(t.match(/\:\d*/i)).slice(1));
            /* eslint-disable */
            var time = min * 60 + sec;
            lrcArray[time] = clause;
          }
        }
        return lrcArray;
      },
      initMethods() {
        // 按元素删除数组中对应的一个元素，添加到Array原型链
        Array.prototype.removeOneByElement = function (element) {
          var index = this.indexOf(element);
          this.splice(index, 1);
        };
      },

      //生成从minNum到maxNum的随机数
      randomNum(minNum, maxNum) {
        switch (arguments.length) {
          case 1:
            return parseInt(Math.random() * minNum + 1, 10);
          case 2:
            return parseInt(Math.random() * (maxNum - minNum + 1) + minNum, 10);
          default:
            return 0;
        }
      },
      /**
       * start:开始数
       *	end:结束数
       */
      createRandomList(start, end) {
        var arr = [];
        for (var j = start; j < end; j++) {
          arr.push(j);
        }

        // var len = end;
        //首先从最大的数开始遍历，之后递减
        for (var i = arr.length - 1; i >= 0; i--) {
          //随机索引值randomIndex是从0-arr.length中随机抽取的
          var randomIndex = Math.floor(Math.random() * (i + 1));
          //下面三句相当于把从数组中随机抽取到的值与当前遍历的值互换位置
          var itemIndex = arr[randomIndex];
          arr[randomIndex] = arr[i];
          arr[i] = itemIndex;
        }
        //每一次的遍历都相当于把从数组中随机抽取（不重复）的一个元素放到数组的最后面（索引顺序为：len-1,len-2,len-3......0）
        return arr;
      },

      // 显示歌曲列表和歌词
      shoyanziistAndLrc(flag) {
        //isShoyanziist
        //isShoyanziistAndLrc
        if (this.isShoyanziist) {
          //列表打开了
          this.isShoyanziist = this.isShoyanziistAndLrc != flag;
          if ($("[lrc-active='Y']")[0]) {
            this.focusLRC();
          }
        } else {
          this.isShoyanziist = true;
        }
        this.isShoyanziistAndLrc = flag;
      },
      // 拖动播放进度
      setCurrentTime() {
        this.yanziAudio.currentTime = this.currentTimeNumber;
      },
      // 获取 音乐总时长
      getDuration() {
        this.allTimeNumber = Math.floor(this.yanziAudio.duration);
        this.allTime = this.allTime = this.timeFormat(
          Math.floor(this.yanziAudio.duration)
        );
      },
      //js将秒数转换成HH:MM:SS格式
      timeFormat(value) {
        let theTime = value; //秒
        let middle = 0; //分
        // let hour = 0; //小时
        if (theTime > 59) {
          middle = parseInt(theTime / 60);
          theTime = parseInt(theTime % 60);
        }
        if (middle > 59) {
          // hour = parseInt(middle / 60);
          middle = parseInt(middle % 60);
        }
        var newTime = (middle < 10 ? "0" + middle : middle) + ":" + (theTime < 10 ? "0" + theTime : theTime);
        return newTime;
      },

      // 修改音量
      changeVolume() {
        this.yanziAudio.volume = this.volumeValue / 100; // audio的取值范围是0~1，而range的值默认是0~100，所以要除以100
      },
      // 开始搜索关键字
      startSearchMusic() {
        this.outStyle(this.searchMusic);
      },
      // 搜索后突出显示关键字
      outStyle(val) {
        var that = this;
        if (val) {
          that.filterList = [];
          that.musicList.forEach(function (item) {
            item.name = item.name.replace(/<b>/g, "").replace(/<\/b>/g, "");
            if (item.name.indexOf(val) != -1 || item.author.indexOf(val) != -1) {
              that.filterList.push(item);
            }
          });
          //   $(".musicItem").remove();

          var newArr = [];
          that.filterList.forEach(function (item) {
            // item.name = item.name.replace(/<b>/g, "").replace(/<\/b>/g, "");
            if (item.name.indexOf(val) != -1) {
              item.name = item.name.replace(
                new RegExp(val, "g"),
                "<b>" + val + "</b>"
              );
              newArr.push(item);
            }
          });
          that.filterList = newArr;
        } else {
          that.musicList.forEach(function (item) {
            item.name = item.name.replace(/<b>/g, "").replace(/<\/b>/g, "");
          });
          console.log("没有");
          that.filterList = that.musicList;
          for (var i = 0; i < that.musicList.length; i++) {
            if (that.musicList[i].name == that.currentMusicName) {
              that.musicIndex = i;
              break;
            }
          }
          console.log(that.musicIndex);
          this.$nextTick(function () {
            this.focusMusic();
          });
        }
      },
      // 滚动到顶部
      goTop() {
        var panel = $(".musicListBox");
        panel.animate({
            scrollTop: 0
          },
          500
        );
      },
      //【聚焦】歌词滚动条
      focusLRC() {
        var panel = $(".yanzi-lrc");
        // var list = $(".lrcItems");
        var div = $("[lrc-active='Y']")[0];
        if (div) {
          var top = div.offsetTop;
          panel.animate({
              scrollTop: top - panel.height() / 2
            },
            200
          );
        }
      },
      //【聚焦】当前播放的歌曲在可见范围
      focusMusic() {
        var panel = $(".musicListBox");
        var div = $(".musicItem")[this.musicIndex];
        $(".musicItem:eq(" + this.musicList + ")").css(
          "background",
          "red !important"
        );
        panel.animate({
            scrollTop: div.offsetTop - panel.height() / 2
          },
          500
        );
      },
      // 关闭歌曲列表
      closeList() {
        if (!this.isShoyanziist) {
          // this.$parent.hideMusic();
          this.$emit("update:isShowMusic", false);
        } else {
          this.isShoyanziist = false;
          setTimeout(() => {
            // this.$parent.hideMusic();
            this.$emit("update:isShowMusic", false);
          }, 300);
        }
      },
      openPlayer() {
        this.$emit("update:isShowMusic", true);
      },
      // 打开或者关闭 随机模式
      openAndCloseRandom() {
        if (this.isRandom) {
          this.isRandom = false;
        } else {
          this.isRandom = true;
          this.buildIndexList();
        }
      },
      // 生成当前播放列表
      buildIndexList() {
        this.indexList = [];
        var arr = [];
        for (var i = 0; i < this.filterList.length - 1; i++) {
          arr.push(i);
        }
        this.indexList = arr;
      },
      // 返回当前播放的歌曲名
      returnMusicName() {
        if (!this.filterList.length) {
          return "列表为空";
        }
        var name = this.filterList[this.musicIndex].name
          .replace(/<b>/g, "")
          .replace(/<\/b>/g, "");
        this.currentMusicName = name;
        return name ? name : "未播放..";
      },
      // 随机下一首歌
      randomNext() {
        while (this.indexList.indexOf(random) == -1) {
          var random = this.randomNum(0, this.filterList.length - 1);
          console.log(random);
          if (this.indexList.indexOf(random) != -1) {
            this.indexList.removeOneByElement(random);
            var src = this.filterList[random].src;
            this.musicIndex = random;
            this.play(src);
            return false;
          }
        }
      },
      //停止
      stop() {
        this.currentTimeNumber = 0;
        this.yanziAudio.currentTime = 0;
        this.yanziAudio.pause();
        this.isPlay = false;
      },
      // 重新播放
      reset() {
        this.currentTimeNumber = 0;
        this.yanziAudio.currentTime = 0;
      },
      // 自动下一曲
      autoNext() {
        // 如果开启了单曲循环模式
        if (this.isSingleLoop) {
          this.currentTimeNumber = 0;
          this.yanziAudio.currentTime = 0;
          this.yanziAudio.play();
          return false;
        }
        var src = "";
        // 如果开启了随机播放模式
        if (this.isRandom) {
          this.randomNext();
          return false;
          //
        } else {
          // 正常全部顺序循环
          if (this.musicIndex + 1 < this.filterList.length) {
            this.musicIndex++;
          } else {
            this.musicIndex = 0;
          }

          src = this.filterList[this.musicIndex].src;
        }
        this.play(src);
      },
      // 点击列表的歌曲
      selectedListMusic(index) {
        if (
          this.filterList[index] &&
          this.filterList[index].src != this.yanziAudio.src
        ) {
          this.musicIndex = index;
          var src = this.filterList[index].src;
          this.play(src);
        }
      },
      // 播放或者暂停
      playAndPause() {
        if (this.isPlay) {
          this.isPlay = false;
          this.yanziAudio.pause();
        } else {
          if (this.isBuildCanvas) {
            this.isPlay = true;
            this.yanziAudio.play();
          } else {
            this.play();
          }
        }
      },
      // 播放
      play(src) {
        var audio = document.getElementById("yanzi-audio");
        this.isShowYesOrNo = false;
        if (src) {
          audio.src = this.prePath + src;
        } else {
          audio.src = this.prePath + this.filterList[this.musicIndex].src;
        }
        this.checkStatus_play(audio);
      },
      // 检查音频状态在播放
      checkStatus_play(audio) {
        // 获取单曲 歌词
        this.lrcArray = this.getLRC();
        this.isPlay = true;
        try {
          audio.play();
          this.focusMusic(); //聚焦到当前播放的歌曲
          this.focusLRC(); //聚焦到当前一句歌词
        } catch (e) {
          console.log("播放出错");
          this.isPlay = false;
        }

        if (!this.isBuildCanvas) {
          this.buildCanvas(audio);
        }
      },
      // 构建画布音频
      buildCanvas(audio) {
        this.isBuildCanvas = true;
        var ctx = new AudioContext();
        var analyser = ctx.createAnalyser();
        var audioSrc = ctx.createMediaElementSource(audio);
        audioSrc.connect(analyser);
        analyser.connect(ctx.destination);

        var canvas = document.getElementById("canvas"),
          cwidth = canvas.width,
          cheight = canvas.height - 2,
          meterWidth = 10,
          capHeight = 2,
          capStyle = "#fff",
          meterNum = 850 / (10 + 2),
          capYPositionArray = [];
        ctx = canvas.getContext("2d");
        var gradient = ctx.createLinearGradient(0, 0, 0, 110);
        gradient.addColorStop(1, "#0f0");
        gradient.addColorStop(0.5, "#ff0");
        gradient.addColorStop(0, "#f00");
        // loop
        function renderFrame() {
          try {
            var array = new Uint8Array(analyser.frequencyBinCount);
            analyser.getByteFrequencyData(array);
            var step = Math.round(array.length / meterNum);
            ctx.clearRect(0, 0, cwidth, cheight);
            for (var i = 0; i < meterNum; i++) {
              var value = array[i * step];
              if (capYPositionArray.length < Math.round(meterNum)) {
                capYPositionArray.push(value);
              }
              ctx.fillStyle = capStyle;
              if (value < capYPositionArray[i]) {
                ctx.fillRect(
                  i * 12,
                  cheight - --capYPositionArray[i] / 3,
                  meterWidth,
                  capHeight
                );
              } else {
                ctx.fillRect(i * 12, cheight - value / 3, meterWidth, capHeight);
                capYPositionArray[i] = value;
              }
              ctx.fillStyle = gradient;
              ctx.fillRect(
                i * 12,
                cheight - value / 3 + capHeight,
                meterWidth,
                cheight
              ); //the meter
            }
          } catch (e) {
            console.log("画布失败");
          }
          requestAnimationFrame(renderFrame);
        }
        renderFrame();
      },
      //   下一曲
      next() {
        if (this.musicIndex + 1 < this.filterList.length) {
          this.musicIndex++;
        } else {
          this.musicIndex = 0;
        }
        this.play();
      },
      //上一曲
      pre() {
        if (this.musicIndex - 1 >= 0) {
          this.musicIndex--;
        } else {
          this.musicIndex = this.filterList.length - 1;
        }
        this.play();
      }
    }
  };
</script>

<style scoped>
  @import "//at.alicdn.com/t/font_2273881_qfilmac4o29.css";
</style>
<style>
  /* 播放器盒子 */
  #yanzi-player {
    /* min-height: 100px; */
    position: fixed !important;
    bottom: -2px;
    left: 0;
    z-index: 1001;
    position: relative;
    transition: all 0.3s;
    transform: translateX(-100%);
    border-radius: 0 10px 0 0;
  }

  .yanzi-btn {
    display: inline-block;
    height: 38px;
    line-height: 38px;
    padding: 0 18px;
    background-color: #2dbe60;
    color: #fff;
    white-space: nowrap;
    text-align: center;
    font-size: 14px;
    border: none;
    border-radius: 2px;
    cursor: pointer;
  }

  .yanzi-btn-mini {
    height: 22px;
    line-height: 22px;
    padding: 0 5px;
    font-size: 12px;
  }

  .yanzi-btn-sm {
    height: 30px;
    line-height: 30px;
    padding: 0 10px;
    font-size: 12px;
  }

  .yanzi-btn-group {
    display: inline-block;
    vertical-align: middle;
    font-size: 0;
  }

  .yanzi-btn-group .yanzi-btn {
    margin-left: 0 !important;
    margin-right: 0 !important;
    border-left: 1px solid rgba(255, 255, 255, 0.5);
    border-radius: 0;
  }

  .yanzi-btn-group .yanzi-btn:first-child {
    border-left: none;
    border-radius: 2px 0 0 2px;
  }

  .yanzi-btn-group .yanzi-btn:last-child {
    border-radius: 0 2px 2px 0;
  }

  .yanzi-bg-default {
    border: 1px solid #c9c9c9;
    background-color: #fff;
    color: #555;
    height: 28px;
    line-height: 28px;
  }

  .yanzi-bg-green {
    background-color: #009688;
    color: #fff;
  }

  .yanzi-bg-blue {
    background-color: #1e9fff;
    color: #fff;
  }

  .yanzi-bg-orange {
    background-color: #ffb800;
    color: #fff;
  }

  .yanzi-bg-red {
    background-color: #ff5722;
    color: #fff;
  }

  .showMusic {
    transform: translateX(0) !important;
  }

  .yanziAudio-main {
    border-radius: 0 10px 0 0;
    /* min-width: 400px; */
  }

  #canvasBlock {
    background: #333;
  }

  .listAndLrc {
    left: 0px;
    width: 100%;
    max-height: 400px;
    height: 400px;
    position: absolute;
    top: 100%;
    z-index: 1001;
  }

  .yanzi-playerList {
    height: 100%;
    position: relative;
    box-shadow: 0 0 10px #ccc;
  }

  .focusMusicItem {
    position: absolute;
    bottom: 30px;
    right: 20px;
    font-size: 30px !important;
    color: #ccc !important;
  }

  .goTop {
    bottom: 70px;
    font-size: 28px !important;
  }

  .musicListBox {
    width: calc(100% - 60px);
    margin: 0 auto;
    height: calc(100% - 70px);
    overflow-y: scroll;
    overflow-x: hidden;
  }

  .shoyanziist {
    bottom: 400px !important;
  }

  /* 歌手头像 */
  .yanziAudio-icon {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    position: absolute;
    top: -30px;
    left: 10px;
    z-index: 1001;
    /* 使用动画 */
    animation: remindMessage 10s linear infinite;
  }

  .pausedLoop {
    animation-play-state: paused;
  }

  .yanziAudio-ico:hover {
    animation-play-state: paused;
    /*暂停动画*/
    /*animation-play-state:running;继续播放动画*/
  }

  /* 定义动画 - 一直旋转*/
  @keyframes remindMessage {
    0% {
      -webkit-transform: rotate(0deg);
    }

    100% {
      -webkit-transform: rotate(360deg);
    }
  }

  .mainButton>.iconfont {
    margin: 5px;
  }

  .yanziAudio-icon img {
    width: calc(100% - 20px);
    height: calc(100% - 20px);
    margin: 10px auto;
    border-radius: 50%;
    display: block;
  }

  #yanzi-player .right {
    width: 100%;
    overflow: hidden;

    z-index: 1001;
  }

  .btn2Block {
    vertical-align: bottom;
    margin-left: 30px;
    height: 20px;
    display: inline-block;
  }

  .btn2Block .iconfont {
    /* margin: 0 5px; */
    float: left;
    margin-left: 15px;
  }

  .btn2Block .iconfont:nth-of-type(2) {
    margin-top: -1px;
  }

  /* 歌曲名 */
  .tools2 .musicName {
    line-height: 40px;
    text-align: left;
    font-family: "STHeiti Light";
    color: #444;
    text-indent: 1em;
    white-space: nowrap;
    font-size: 14px;
  }

  .loopMusicName {
    /* 使用动画 */
    /* animation: loopMusicName 5s linear infinite; */
  }

  /* 定义动画 - 滚动歌名*/
  @keyframes loopMusicName {
    0% {
      -webkit-transform: translateX(100%);
    }

    100% {
      -webkit-transform: translateX(-100%);
    }
  }

  #yanzi-player .tools {
    /* padding-left: 100px; */
    background: #eee;
    border-radius: 0 10px 0 0;
    padding: 10px 50px 10px 130px;
  }

  .tools .iconfont {
    font-size: 18px !important;
  }

  .tools2 {
    background: #eee;
    display: flex;
    padding-right: 10px;
  }

  .tools2>div:nth-of-type(1) {
    width: 50%;
  }

  .tools2>div:nth-of-type(2) {
    width: 50%;
  }

  .speed {
    display: flex;
  }

  .speed span {
    font-size: 12px;
    margin: 0 5px;
  }

  .speed input {
    width: 100px;
  }

  /* 列表和关闭 */
  .listAndClose {
    /* position: absolute;
  top: 10px;
  right: 10px; */
    padding: 10px 0;
    display: flex;
    justify-content: space-between;
  }

  .listAndClose .iconfont {
    margin: 0 5px;
    font-size: 18px !important;
  }

  .listTools {
    display: flex;
    justify-content: space-between;
    padding: 0 40px;
  }

  /* 搜索框box */
  .searchBox {
    height: 40px;
    line-height: 40px;
    border-radius: 20px;
    background: #e2e2e2;
    position: relative;
    width: calc(80% - 60px);
    margin-top: 10px;
  }

  .listTools .yanzi-btn {
    margin-top: 10px;
  }

  /* 搜索文本框 */
  #searchMusic {
    width: calc(100% - 60px);
    border: none;
    outline: none;
    background: transparent;
    height: 40px;
    line-height: 40px;
    float: left;
    margin-left: 20px;
  }

  .yanzi-lrc {
    position: relative;
    height: 100%;
    overflow-x: hidden;
    overflow-y: scroll;
  }

  .tispNull {
    height: 400px;
    line-height: 400px;
    text-align: center;
  }

  .lrcItems {
    text-align: center;
    line-height: 40px;
    transition: all 0.3s;
  }

  .lrcActiveStyle {
    transform: scale(1.5);
    color: orangered;
  }

  #colseMusic {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 14px;
    border-radius: 50%;
    padding: 3px;
  }

  #colseMusic:hover {
    background: red;
    color: white;
  }

  b {
    color: red;
  }

  .volumeIcon {
    position: relative;
  }

  /* 音量滑块样式 */
  .volumeBlock {
    position: absolute;
    background: white;
    box-shadow: 0 0 10px #666;
    transform: rotate(90deg);
    width: 110px;
    left: -55px;
    top: -70px;
    border-radius: 10px;
    padding: 3px 10px;
    transition: all 0.3s;
    opacity: 0;
  }

  .volumeIcon:hover .volumeBlock {
    opacity: 1;
  }

  #volume {
    vertical-align: middle;
    height: 3px;
    width: 80px;
    transform: rotate(180deg);
  }

  #volumeValueNumber {
    font-size: 12px;
    position: relative;
    transform: rotate(-90deg);
    line-height: 30px;
    margin-left: 5px;
    display: inline-block;
  }

  #volume::-webkit-slider-thumb {
    /* -webkit-appearance: none; */
  }

  /* ************************************************颜色样式【yanzi-white】********************************************** */
  /* 图标颜色 */
  .yanzi-white .iconfont {
    font-size: 24px;
    color: #444;
    cursor: pointer;
  }

  /* 图标鼠标悬停颜色 */
  .yanzi-white .iconfont:active {
    color: #1989fa !important;
  }

  /* 界面背景色 */
  .yanzi-white,
  .yanziAudio-icon,
  .listAndLrc {
    background: white;
  }

  .yanzi-white .yanziAudio-main {
    box-shadow: 0 0 15px black;
  }

  .activeColor {
    color: #1989fa !important;
  }

  .yanzi-white .focusMusicItem:hover {
    color: #1989fa !important;
  }

  .musicItem .musicName {
    padding: 0;
    font-size: 13px;
    cursor: pointer;
    height: 30px;
    line-height: 30px;
    padding: 0 30px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .musicItem .musicName:hover {
    background: #e2e2e2;
  }

  .musicListBox::-webkit-scrollbar {
    width: 10px;
    height: 10px;
    background-color: #fff;
  }

  .musicListBox::-webkit-scrollbar-thumb {
    /* 鼠标停留之前，滚动条看不见 */
    background-color: #fff;
    height: 10px;
  }

  .musicListBox:hover::-webkit-scrollbar-thumb {
    /* 鼠标悬停，滚动条显示 */
    background-color: #e2e2e2;
  }

  #openButton {
    position: fixed;
    bottom: 20px;
    left: 20px;
    width: 40px;
    height: 40px;
    background: white;
    color: orange;
    font-size: 14px;
    text-align: center;
    line-height: 40px;
    /* border-radius: 0 50% 50% 0; */
    cursor: pointer;
    border-radius: 50%;
    box-shadow: 0 0 10px #ccc;

  }

  .openButton {
    animation: remindMessage 5s linear infinite;
  }

  #openButton:hover {
    animation: rubberBand 1s;
  }


  @keyframes rubberBand {
    from {
      transform: scale3d(1, 1, 1);
    }

    30% {
      transform: scale3d(1.25, 0.75, 1);
    }

    40% {
      transform: scale3d(0.75, 1.25, 1);
    }

    50% {
      transform: scale3d(1.15, 0.85, 1);
    }

    65% {
      transform: scale3d(.95, 1.05, 1);
    }

    75% {
      transform: scale3d(1.05, .95, 1);
    }

    to {
      transform: scale3d(1, 1, 1);
    }
  }

  /* ************************************************颜色样式【yanzi-white】********************************************** */
</style>