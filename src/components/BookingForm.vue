<template>
  <div class="booking-form">
    <!-- 加入 LOGO -->
    <div class="logo-container">
      <img src="/logo.png" alt="桑time LOGO" class="logo" />
    </div>
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

    <!-- 回應訊息 -->
    <div v-if="message" class="message" :class="{ 'success-message': message.success, 'error-message': !message.success }">
      <p>{{ message.text }}</p>
    </div>

    <!-- 版權標示 -->
    <p class="footer">Made by 阿u</p>
  </div>
</template>
