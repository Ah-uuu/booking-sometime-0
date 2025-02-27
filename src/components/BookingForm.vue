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
        <option v-for="hour in availableHours" :key="hour" :value="hour">
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

    <!-- 回應訊息 -->
    <div v-if="message" class="message" :class="{ 'success-message': message.success, 'error-message': !message.success }">
      <p>{{ message.text }}</p>
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
    updateOptions() {
      if (this.availableHours.length > 0 && !this.availableHours.includes(this.formData.appointmentHour)) {
        this.formData.appointmentHour = this.availableHours[0];
      }
      this.updateMinutesOptions(); // 同步更新分鐘選項
      this.$forceUpdate();
    },
    updateMinutesOptions() {
      if (this.formData.appointmentHour === '21' || this.formData.appointmentHour === '22') {
        this.formData.appointmentMinutes = '00';
      }
    },
    isMinuteDisabled(minute) {
      if (!this.formData.appointmentHour) return false;
      const isLateHour = this.formData.appointmentHour === '21' || this.formData.appointmentHour === '22';
      return isLateHour && minute !== '00';
    },
    isServiceDisabled(duration) {
      if (!this.formData.appointmentDate || !this.formData.appointmentHour) return false;
      const selectedDate = new Date(this.formData.appointmentDate);
      const dayOfWeek = selectedDate.getDay();
      const isWeekday = dayOfWeek >= 1 && dayOfWeek <= 4;
      const isLateHour = (isWeekday && this.formData.appointmentHour === '21') || 
                        (!isWeekday && this.formData.appointmentHour === '22');
      
      if (!isLateHour) return false;
      return duration > 70; // 70 分鐘以上的項目禁用（不包含 70 分鐘）
    },
    async checkAvailableTimes() {
      try {
        if (!this.formData.service || !this.formData.appointmentDate) {
          this.message = { success: false, text: '請選擇按摩項目和日期！' };
          setTimeout(() => this.message = null, 5000);
          return;
        }
        const response = await axios.get(`https://booking-k1q8.onrender.com/available-times?service=${this.formData.service}&date=${this.formData.appointmentDate}`);
        if (response.data.success) {
          this.message = {
            success: true,
            text: `當日最快可預約時段：${response.data.nextAvailableTime}`,
          };
          // 自動填充預約時間（可選）
          const [date, time] = response.data.nextAvailableTime.split(' ');
          const [hour, minute] = time.split(':');
          this.formData.appointmentDate = date;
          this.formData.appointmentHour = hour;
          this.formData.appointmentMinutes = minute;
        } else {
          this.message = { success: false, text: response.data.message };
        }
        setTimeout(() => this.message = null, 5000);
      } catch (error) {
        console.error('查詢可用時段時發生錯誤：', error);
        this.message = {
          success: false,
          text: error.response?.data?.message || '查詢可用時段失敗，請稍後再試！',
        };
        setTimeout(() => this.message = null, 5000);
      }
    },
    async submitForm() {
      console.log('預約資料：', this.formData);

      // 提交完整的 service 格式（例如 "半身按摩_30"）
      const service = this.formData.service;

      const appointmentTimeLocal = `${this.formData.appointmentDate}T${this.formData.appointmentHour}:${this.formData.appointmentMinutes.padStart(2, '0')}:00+08:00`;
      const appointmentTime = new Date(appointmentTimeLocal);

      const payload = {
        name: this.formData.name,
        phone: this.formData.phone,
        service: service, // 傳完整的服務名稱_時長
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
            ? [40, duration - 40] // 腳底固定 40 分鐘，其餘分配
            : [duration];
          const timeSegments = segments.map((seg, index) => {
            const segmentStart = new Date(startTime.getTime() + (index === 0 ? 0 : durationPerSegment[0]) * 60000);
            const segmentEnd = new Date(segmentStart.getTime() + durationPerSegment[index] * 60000);
            return `${seg}: ${segmentStart.toLocaleString()} - ${segmentEnd.toLocaleString()}`;
          });

          this.message = {
            success: true,
            text: `預約成功！預約時間：\n${timeSegments.join('\n')}`,
          };

          this.formData.name = '';
          this.formData.phone = '';
          this.formData.service = '';
          this.formData.master = '';
          this.formData.appointmentDate = '';
          this.formData.appointmentHour = '';
          this.formData.appointmentMinutes = '00';

          setTimeout(() => {
            this.message = null;
          }, 5000);
        }
      } catch (error) {
        console.error('提交預約時發生錯誤：', error);
        const errorMessage = error.response?.data?.message || '預約提交失敗，請稍後再試！';

        this.message = {
          success: false,
          text: errorMessage.includes('請點擊「當日可預約時段」')
            ? errorMessage
            : `${errorMessage}\n${error.response?.data?.nextAvailableTime ? `目前最快可預約時段：${error.response.data.nextAvailableTime}` : ''}`,
        };
        setTimeout(() => {
          this.message = null;
        }, 5000);
      }
    },
  },
};
</script>

<style scoped>
.booking-form {
  max-width: 400px;
  margin: auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.1);
}

.title {
  color: black;
}

.label {
  color: black;
  display: block;
  margin-top: 10px;
}

input,
select {
  width: 100%;
  padding: 8px;
  margin-top: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

select option:disabled {
  color: gray;
  background-color: #f0f0f0;
}

button {
  margin-top: 10px;
  padding: 10px;
  width: 100%;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.available-times-button {
  margin-top: 10px;
  padding: 10px;
  width: 100%;
  background: #28a745; /* 綠色按鈕，與提交按鈕區別 */
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover,
.available-times-button:hover {
  opacity: 0.9;
}

.message {
  margin-top: 20px;
  padding: 10px;
  border-radius: 4px;
  text-align: center;
  white-space: pre-line;
}

.success-message {
  background-color: #4caf50;
  color: white;
}

.error-message {
  background-color: #f44336;
  color: white;
}

.footer {
  text-align: center;
  margin-top: 20px;
  font-size: 12px;
  color: gray;
}
</style>
