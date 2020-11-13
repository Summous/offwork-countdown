<template>
  <div style="text-align: center;position:relative; top: 50px;">
    <el-time-picker
      v-model="time"
      :disabled="!!timer"
      placeholder="请选择结束时间"
      value-format="HH:mm"
       style="width: 240px"
    >
    </el-time-picker>
    <div style="margin-top: 10px;">
      <el-input :disabled="!!timer" v-model="input"  placeholder="请选择间隔（分钟，默认整点）" style="width: 240px" :min="1" type="number"></el-input>  
    </div>
    <div style="margin-top: 10px;">
      <el-button
        type="primary"
        size="small"
        @click="handleClick"
        @change="handleChange"
      >
        {{timer ? '取消' : '设定'}}
      </el-button>
    </div>
    <div style="margin-top: 10px;" v-if="!!timer">
        下次提醒时间 {{ showNextTime }}
    </div>
    <div v-if="showTips" style="font-size: 16px;margin-top: 10px;">{{showTips === 1 ? '请选择结束时间' : '下次提醒时间不能小于结束时间'}}</div>
  </div>
</template>

<script>
  import { ipcRenderer } from 'electron';
//   import dayjs from 'dayjs';

  export default {
    name: 'set-time-page',
    data() {
      return {
        time: null,
        showTips: 0,
        timer: null,
        input: null,
        nextTime: null,
      };
    },
    computed: {
      showNextTime() {
        const setTime = new Date(this.nextTime * 1000);
        const hours = setTime.getHours();
        const minutes = setTime.getMinutes();
        const seconds = setTime.getSeconds();
        const nextTime = new Date().getHours() + 1;
        return this.nextTime ?
          `${hours < 10 ? '0' : ''}${hours}:${minutes < 10 ? '0' : ''}${minutes}:${seconds < 10 ? '0' : '' && '0'}${seconds}` :
          `${nextTime < 10 ? '0' : ''}${nextTime}:00:00`;
      },
    },
    methods: {
      handleClick() {
        // 未设定结束时间
        if (!this.time) {
          this.showTips = 1;
          return;
        }
        this.showTips = 0;
        // 已计时，点击停止
        if (this.timer) {
          clearInterval(this.timer);
          this.timer = null;
          this.nextTime = null;
          return;
        }
        const [hour, minute] = this.time.split(':');
        let setTime = new Date();
        setTime.setHours(hour);
        setTime.setMinutes(minute);
        setTime.setSeconds(0);
        setTime = parseInt(setTime.getTime() / 1000, 10);
        // 设定时间间隔
        if (this.input) {
          this.nextTime = parseInt(new Date().getTime() / 1000, 10) + (this.input * 60);
          // 下次提醒时间超过结束时间
          if (this.nextTime > setTime) {
            this.showTips = 2;
            return;
          }
        // 未设定时间间隔，结束时间小于当前时间
        } else if (parseInt(new Date().getTime() / 1000, 10) > setTime) {
          this.showTips = 2;
          return;
        }
        this.counter(setTime);
      },
      /**
       * @description: 计时器
       * @param {Date} setTime 结束时间
       * @return {Null}
       */
      counter(setTime) {
        this.timer = setInterval(() => {
          console.log(parseInt(new Date().getTime() / 1000, 10), this.nextTime);
          const check = this.input ? this.checkTime(this.nextTime < setTime ?
            this.nextTime : setTime, this.nextTime !== setTime) : this.checkTime(setTime);
          if (check) {
            if (this.input) {
              this.nextTime = this.nextTime + (this.input * 60);
              // 下次提醒时间大于技术时间
              if (this.nextTime > setTime) {
                this.nextTime = setTime;
              }
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
      /**
       * @description: 判断当前时间是否为整点或设置的间隔时间, 以及是否以及到达time, 精确到秒
       * @param {Date} time 结束时间
       * @param {Bollean} isInput 输入时间间隔
       * @return {Object} 结束标志 { reach: true }、 { reach: false }、 null
       */
      checkTime(time, isInput = false) {
        const now = new Date();
        if (parseInt(now.getTime() / 1000, 10) === time) {
          return {
            reach: !isInput,
          };
        }
        return !isInput && now.getMinutes() === 0 && now.getSeconds() === 0 ? {
          reach: false,
        } : null;
      },
      handleChange() {
        // console.log(1);
        // if (!this.showTips) {
        //   this.showTips = true;
        // }
      },
    },
  };
</script>