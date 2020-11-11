<template>
  <div class="alert-wrapper">
    <img style="display: block;width: 100%;" draggable="false" :src="src" />
    <div class="alert-tips">
      练习时长两年半的个人练习生提醒您
      <br/>
      {{tips}}
    </div>
  </div>
</template>

<script>
  // import dayjs from 'dayjs';
  import { ipcRenderer } from 'electron';
  import gif from '../../assets/img/cxk.gif';
  import jntm from '../../assets/audio/jntm.mp3';
  export default {
    name: 'alert-page',
    data() {
      return {
        src: Boolean(this.$route.query.reach === 'true') ? gif : gif,
        time: null,
        tips: '',
      };
    },
    created() {
      ipcRenderer.on('alert-message', (event, time) => {
        this.time = new Date(time);
      });
      this.$nextTick(() => {
        setInterval(() => {
          // const setTime = dayjs(this.time);
          // const now = dayjs();
          // const seconds = setTime.diff(now, 'second') % 60;
          // const minutes = setTime.diff(now, 'minute') % 60;
          // const hours = setTime.diff(now, 'hour');
          this.tips = '打工人，该休息了！！！';
        }, 1000);
      });
    },
    mounted() {
      const audio = new Audio(jntm);
      audio.play();
    },
  };
</script>

<style>
  .alert-wrapper {
    position: relative;
    text-align: center;
    overflow: hidden;
  }
  .alert-tips {
    position: absolute;
    width: 300px;
    height: 50px;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
    color: blue;
  }
</style>