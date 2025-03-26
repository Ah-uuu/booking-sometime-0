<template>
  <div class="booking-form">
    <h2 class="title">桑time線上預約系統</h2>
    <form @submit.prevent="submitForm">
      <!-- 姓名 -->
      <label for="name" class="label">姓名：</label>
      <input type="text" id="name" v-model="formData.name" required />

      <!-- 電話 -->
      <label for="phone" class="label">電話：</label>
      <input type="tel" id="phone" v-model="formData.phone" required />

      <!-- 預約日期 -->
      <label for="appointmentDate" class="label">預約日期：</label>
      <input type="date" id="appointmentDate" v-model="formData.appointmentDate" @input="updateOptions" required />

      <!-- 預約幾點 -->
      <label for="appointmentHour" class="label">預約幾點：</label>
      <select id="appointmentHour" v-model="formData.appointmentHour" @change="updateOptions" required>
        <option
          v-for="hour in availableHours"
          :key="hour"
          :value="hour"
          :disabled="isHourDisabled(hour)"
        >
          {{ hour }}:00
        </option>
      </select>

      <!-- 預約幾分 -->
      <label for="appointmentMinutes" class="label">預約幾分：</label>
      <select id="appointmentMinutes" v-model="formData.appointmentMinutes" required>
        <option
          v-for="minute in availableMinutes"
          :key="minute"
          :value="minute"
          :disabled="isMinuteDisabled(minute)"
        >
          {{ minute }} 分
        </option>
      </select>

      <!-- 按摩項目（含時長） -->
      <label for="service" class="label">按摩項目：</label>
      <select id="service" v-model="formData.service" required>
        <option
          v-for="option in serviceOptions"
          :key="option.value"
          :value="option.value"
          :disabled="isServiceDisabled(option.duration)"
        >
          {{ option.label }}
        </option>
      </select>

      <!-- 指定師傅 -->
      <label for="master" class="label">指定師傅（可選）：</label>
      <select id="master" v-model="formData.master">
        <option value="">不指定</option>
        <option
          v-for="therapist in therapists"
          :key="therapist.name"
          :value="therapist.name"
          :disabled="isTherapistOff(therapist.offDays)"
        >
          {{ therapist.name }} ({{ therapist.offDaysText }})
        </option>
      </select>

      <!-- 新增「當日可預約時段」按鈕 -->
      <button type="button" @click="checkAvailableTimes" class="available-times-button">
        當日可預約時段
      </button>

      <!-- 提交按鈕 -->
      <button type="submit">送出預約</button>
    </form>

    <!-- 彈出視窗 -->
    <div v-if="showModal" class="modal-overlay">
      <div class="modal-content" :class="{ 'success-modal': message.success, 'error-modal': !message.success }">
        <p>{{ message.text }}</p>
        <button @click="closeModal" class="modal-button">確定</button>
      </div>
    </div>

    <!-- 版權標示 -->
    <p class="footer">Made by 阿u</p>
  </div>
</template>

<script>
import axios from 'axios';
import moment from 'moment-timezone';

export default {
  data() {
    return {
      formData: {
        name: '',
        phone: '',
        service: '',
        master: '',
        appointmentDate: '',
        appointmentHour: '',
        appointmentMinutes: '00',
      },
      message: null,
      showModal: false,
      serviceDurations: {
        '半身按摩_30': 30,
        '半身按摩_60': 60,
        '全身按摩_60': 60,
        '全身按摩_90': 90,
        '全身按摩_120': 120,
        '全身按摩_150': 150,
        '腳底按摩_40': 40,
        '腳底按摩_70': 70,
        '腳底+半身_70': 70,
        '腳底+全身_100': 100,
        '腳底+全身_130': 130,
      },
      therapists: [
        { name: '阿U 1號', offDays: [3, 4], offDaysText: '休三四' },
        { name: '小周 2號', offDays: [6, 0], offDaysText: '休六日' },
        { name: 'Alan 7號', offDays: [1, 2, 3, 4, 5], offDaysText: '休一到五' },
        { name: 'Vincent 8號', offDays: [6, 0], offDaysText: '休六日' },
        { name: '魚丸 12號', offDays: [5], offDaysText: '休五' },
        { name: '小力 30號', offDays: [1, 2], offDaysText: '休一二' },
      ],
    };
  },
  computed: {
    availableHours() {
      if (!this.formData.appointmentDate) return [];
      const selectedDate = new Date(this.formData.appointmentDate);
      const dayOfWeek = selectedDate.getDay();
      const isWeekday = dayOfWeek >= 1 && dayOfWeek <= 4;
      const startHour = isWeekday ? 12 : 13;
      const endHour = isWeekday ? 21 : 22;
      const hours = [];
      for (let hour = startHour; hour <= endHour; hour++) {
        hours.push(hour < 10 ? `0${hour}` : `${hour}`);
      }
      return hours;
    },
    availableMinutes() {
      return ['00', '10', '20', '30', '40', '50'];
    },
    serviceOptions() {
      return [
        { value: '半身按摩_30', label: '半身按摩 (30 分鐘)', duration: 30 },
        { value: '半身按摩_60', label: '半身按摩 (60 分鐘)', duration: 60 },
        { value: '全身按摩_60', label: '全身按摩 (60 分鐘)', duration: 60 },
        { value: '全身按摩_90', label: '全身按摩 (90 分鐘)', duration: 90 },
        { value: '全身按摩_120', label: '全身按摩 (120 分鐘)', duration: 120 },
        { value: '全身按摩_150', label: '全身按摩 (150 分鐘)', duration: 150 },
        { value: '腳底按摩_40', label: '腳底按摩 (40 分鐘)', duration: 40 },
        { value: '腳底按摩_70', label: '腳底按摩 (70 分鐘)', duration: 70 },
        { value: '腳底+半身_70', label: '腳底+半身 (70 分鐘)', duration: 70 },
        { value: '腳底+全身_100', label: '腳底+全身 (100 分鐘)', duration: 100 },
        { value: '腳底+全身_130', label: '腳底+全身 (130 分鐘)', duration: 130 },
      ];
    },
  },
  methods: {
    isTherapistOff(offDays) {
      if (!this.formData.appointmentDate) return false;
      const selectedDate = new Date(this.formData.appointmentDate);
      const dayOfWeek = selectedDate.getDay();
      return offDays.includes(dayOfWeek);
    },
    isHourDisabled(hour) {
      if (!this.formData.appointmentDate) return false;
      const selectedDate = new Date(this.formData.appointmentDate);
      const isToday = selectedDate.toDateString() === new Date().toDateString();
      if (!isToday) return false;

      const now = new Date();
      const currentHour = now.getHours();
      const currentMinute = now.getMinutes();
      const hourNum = parseInt(hour);

      return hourNum < currentHour || (hourNum === currentHour && currentMinute >= 50);
    },
    updateOptions() {
      if (this.availableHours.length > 0 && !this.availableHours.includes(this.formData.appointmentHour)) {
        this.formData.appointmentHour = this.availableHours[0];
      }
      this.updateMinutesOptions();
      this.$forceUpdate();
    },
    updateMinutesOptions() {
      if (!this.formData.appointmentHour) return;

      const selectedDate = new Date(this.formData.appointmentDate);
      const isToday = selectedDate.toDateString() === new Date().toDateString();
      const currentHour = new Date().getHours();
      const currentMinute = new Date().getMinutes();

      // 根據日期判斷是否為週一到週四或週五到週日
      const dayOfWeek = selectedDate.getDay();
      const isWeekday = dayOfWeek >= 1 && dayOfWeek <= 4;
      const lastHour = isWeekday ? '21' : '22';

      if (this.formData.appointmentHour === lastHour) {
        this.formData.appointmentMinutes = '00';
      } else if (isToday && parseInt(this.formData.appointmentHour) === currentHour) {
        const nextAvailableMinute = this.availableMinutes.find(minute => parseInt(minute) > currentMinute);
        this.formData.appointmentMinutes = nextAvailableMinute || this.availableMinutes[this.availableMinutes.length - 1];
      }
    },
    isMinuteDisabled(minute) {
      if (!this.formData.appointmentHour) return false;
      const selectedDate = new Date(this.formData.appointmentDate);
      const isToday = selectedDate.toDateString() === new Date().toDateString();
      const currentHour = new Date().getHours();
      const currentMinute = new Date().getMinutes();

      // 根據日期判斷是否為週一到週四或週五到週日
      const dayOfWeek = selectedDate.getDay();
      const isWeekday = dayOfWeek >= 1 && dayOfWeek <= 4;
      const lastHour = isWeekday ? '21' : '22'; // 週一到週四的最後一小時是 21:00，週五到週日是 22:00

      // 如果選擇的是最後一小時（週一到週四的 21:00 或週五到週日的 22:00），只能選 00 分
      const isLastHour = this.formData.appointmentHour === lastHour;

      if (isLastHour) return minute !== '00';
      if (isToday && parseInt(this.formData.appointmentHour) === currentHour) {
        return parseInt(minute) <= currentMinute;
      }
      return false;
    },
    isServiceDisabled(duration) {
      if (!this.formData.appointmentDate || !this.formData.appointmentHour) return false;
      const selectedDate = new Date(this.formData.appointmentDate);
      const dayOfWeek = selectedDate.getDay();
      const isWeekday = dayOfWeek >= 1 && dayOfWeek <= 4;
      const isLateHour = (isWeekday && this.formData.appointmentHour === '21') || 
                        (!isWeekday && this.formData.appointmentHour === '22');
      
      if (!isLateHour) return false;
      return duration > 70;
    },
    async checkAvailableTimes() {
      try {
        if (!this.formData.service || !this.formData.appointmentDate) {
          this.message = { success: false, text: '請選擇按摩項目和日期！' };
          this.showModal = true;
          return;
        }
        this.message = { success: false, text: '排查進行中 過程需要數秒 請稍等片刻...' };
        this.showModal = true;

        const response = await axios.get(`https://booking-k1q8.onrender.com/available-times?service=${this.formData.service}&date=${this.formData.appointmentDate}`);
        if (response.data.success) {
          const nextAvailableTime = response.data.nextAvailableTime;
          const [date, time] = nextAvailableTime.split(' ');
          const [hour, minute] = time.split(':');
          this.message = {
            success: true,
            text: `當日最快可預約時段：${nextAvailableTime}`,
          };
          this.formData.appointmentDate = date;
          this.formData.appointmentHour = hour;
          this.formData.appointmentMinutes = minute;
        } else {
          this.message = { success: false, text: response.data.message };
        }
      } catch (error) {
        console.error('查詢可用時段時發生錯誤：', error);
        this.message = {
          success: false,
          text: error.response?.data?.message || '查詢可用時段失敗，請稍後再試！',
        };
      }
    },
    async submitForm() {
      console.log('預約資料：', this.formData);

      const service = this.formData.service;
      const appointmentTimeLocal = `${this.formData.appointmentDate}T${this.formData.appointmentHour}:${this.formData.appointmentMinutes.padStart(2, '0')}:00+08:00`;
      const appointmentTime = new Date(appointmentTimeLocal);

      const payload = {
        name: this.formData.name,
        phone: this.formData.phone,
        service: service,
        appointmentTime: appointmentTime.toISOString(),
        master: this.formData.master || undefined,
      };

      try {
        const response = await axios.post('https://booking-k1q8.onrender.com/booking', payload);

        if (response.data.success) {
          let startTime = new Date(payload.appointmentTime);
          const serviceParts = this.formData.service.split('_');
          const serviceName = serviceParts[0];
          const duration = parseInt(serviceParts[1], 10);
          const isComposite = serviceName.includes('+');
          const segments = isComposite
            ? serviceName.split('+').map(s => s.trim())
            : [serviceName];
          const durationPerSegment = isComposite
            ? [40, duration - 40]
            : [duration];
          const timeSegments = segments.map((seg, index) => {
            const segmentStart = new Date(startTime.getTime() + (index === 0 ? 0 : durationPerSegment[0]) * 60000);
            const segmentEnd = new Date(segmentStart.getTime() + durationPerSegment[index] * 60000);
            return `${seg}: ${segmentStart.toLocaleString()} - ${segmentEnd.toLocaleString()}`;
          });

          this.message = {
            success: true,
            text: `預約成功！預約時間：\n${timeSegments.join('\n')}\n(提醒您!!可截圖此畫面以免忘記預約的時段)`,
          };
          this.showModal = true;

          this.formData.name = '';
          this.formData.phone = '';
          this.formData.service = '';
          this.formData.master = '';
          this.formData.appointmentDate = '';
          this.formData.appointmentHour = '';
          this.formData.appointmentMinutes = '00';
        }
      } catch (error) {
        console.error('提交預約時發生錯誤：', error);
        const errorMessage = error.response?.data?.message || '預約提交失敗，請稍後再試！';

        this.message = {
          success: false,
          text: errorMessage,
        };
        this.showModal = true;
      }
    },
    closeModal() {
      this.showModal = false;
      this.message = null;
    },
  },
};
</script>

<style scoped>
/* 引入 Google Fonts 的 Noto Serif TC 字體 */
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@400;500;700&display=swap');

/* 整體容器 */
.booking-form {
  max-width: 400px;
  margin: 40px auto;
  padding: 30px;
  background-color: #FFFFFF; /* 白色背景，作為備用 */
  background-image: url('/paper-texture.jpg'); /* 紙張紋理背景 */
  background-repeat: repeat; /* 平鋪背景 */
  background-size: auto; /* 保持紋理圖片的原始大小，適當平鋪 */
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  font-family: 'Noto Serif TC', serif; /* 使用文青風格字體 */
}

/* 標題 */
.title {
  color: #305450; /* 深青綠色 */
  text-align: center;
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 20px;
}

/* 標籤 */
.label {
  color: #305450; /* 深青綠色 */
  display: block;
  margin-top: 15px;
  font-size: 16px;
  font-weight: 500;
}

/* 輸入框和下拉選單 */
input,
select {
  width: 100%;
  padding: 10px;
  margin-top: 5px;
  border: 2px solid #678772; /* 柔和青綠色邊框 */
  border-radius: 8px;
  font-size: 16px;
  background-color: #F5F5F5; /* 淺灰色背景 */
  transition: border-color 0.3s ease;
  font-family: 'Noto Serif TC', serif; /* 應用字體 */
  color: #333333; /* 深灰色文字 */
}

/* 確保下拉選單的選項文字顏色為深灰色 */
select option {
  color: #333333; /* 深灰色文字 */
}

/* 禁用選項 */
select option:disabled {
  color: #999999; /* 禁用選項使用較淺的灰色 */
  background-color: #E0E0E0;
}

/* 確保聚焦時的樣式 */
input:focus,
select:focus {
  outline: none;
  border-color: #305450; /* 聚焦時邊框變為深青綠色 */
  background-color: #FFFFFF;
}

/* 按鈕 */
button {
  margin-top: 20px;
  padding: 12px;
  width: 100%;
  background: #305450; /* 深青綠色 */
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s ease;
  font-family: 'Noto Serif TC', serif; /* 應用字體 */
}

button:hover {
  background: #3E6A66; /* 稍微變亮 */
}

/* 當日可預約時段按鈕 */
.available-times-button {
  background: #678772; /* 柔和青綠色 */
}

.available-times-button:hover {
  background: #74917C; /* 稍微變亮 */
}

/* 彈出視窗樣式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5); /* 半透明背景 */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  max-width: 90%;
  width: 400px;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  font-family: 'Noto Serif TC', serif;
  font-size: 16px;
  white-space: pre-line;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.success-modal {
  background-color: #678772; /* 柔和青綠色 */
  color: #FFFFFF;
}

.error-modal {
  background-color: #F44336; /* 柔和紅色 */
  color: #FFFFFF;
}

.modal-button {
  margin-top: 20px;
  padding: 10px 20px;
  background: #305450; /* 深青綠色 */
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s ease;
  font-family: 'Noto Serif TC', serif;
}

.modal-button:hover {
  background: #3E6A66; /* 稍微變亮 */
}

/* 版權標示 */
.footer {
  text-align: center;
  margin-top: 30px;
  font-size: 12px;
  color: #678772; /* 柔和青綠色 */
  font-family: 'Noto Serif TC', serif; /* 應用字體 */
}
</style>
