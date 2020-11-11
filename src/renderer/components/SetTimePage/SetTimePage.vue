<template>
  <div style="text-align: center;position:relative; top: 50px;">
    <el-time-picker
      v-model="time"
      placeholder="请选择结束时间"
      value-format="HH:mm"
       style="width: 240px"
    >
    </el-time-picker>
    <div style="margin-top: 10px;">
      <el-input v-model="input"  placeholder="请选择间隔（分钟，默认整点）" style="width: 240px" :min="1" type="number"></el-input>  
    </div>
    <div style="margin-top: 10px;">
      <el-button
        type="primary"
        size="small"
        @click="handleClick"
        @change="handleChange"
      >
        设定
      </el-button>
    </div>
    <span v-if="showTips">请选择时间</span>
  </div>
</template>

<script>
  import { ipcRenderer } from 'electron';
  
  export default {
    name: 'set-time-page',
    data() {
      return {
        time: null,
        showTips: false,
        timer: null,
        input: null,
        nextTime: null,
      };
    },
    methods: {
      handleClick() {
        if (!this.time) {
          this.showTips = true;
          return;
        }
        if (this.timer) {
          return;
        }
        const [hour, minute] = this.time.split(':');
        let setTime = new Date();
        setTime.setHours(hour);
        setTime.setMinutes(minute);
        setTime.setSeconds(0);
        // 判断当前时间是否为整点或设置的间隔时间, 以及是否以及到达time, 精确到秒
        const checkTime = (time, isInput = true) => {
          const now = new Date();
          if (parseInt(now.getTime() / 1000, 10) === time) {
            return {
              reach: isInput,
            };
          }
          return isInput && now.getMinutes() === 0 && now.getSeconds() === 0 ? {
            reach: false,
          } : null;
        };
        if (this.input) {
          this.nextTime = parseInt(new Date().getTime() / 1000, 10) + (this.input * 60);
        }
        setTime = parseInt(setTime.getTime() / 1000, 10);
        this.timer = setInterval(() => {
          const check = this.input ? checkTime(this.nextTime < setTime ?
            this.nextTime : setTime, this.nextTime > setTime) : checkTime(setTime);
          if (check) {
            if (this.input) {
              this.nextTime = this.nextTime + (this.input * 60);
            }
            ipcRenderer.send('open-alert-dialog', check.reach, setTime);
            // 到达设定的时间则清除定时器
            if (check.reach) {
              clearInterval(this.timer);
              this.timer = null;
            }
          }
        }, 1000);
      },
      handleChange() {
        if (!this.showTips) {
          this.showTips = true;
        }
      },
    },
  };
</script>