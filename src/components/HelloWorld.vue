<template>
  <div>
    <van-cell-group inset>
      <van-field
        :model-value="formModel.leaveStartTime"
        name="leaveStartTime"
        label="开始时间"
        is-link
        readonly
        placeholder="请选择开始时间"
        @click="showStartTimePicker = true"
      />
      <van-popup round :show="showStartTimePicker" position="bottom">
        <van-picker-group
          next-step-text="下一步"
          title="开始时间"
          :tabs="['选择日期', '选择时间']"
          @confirm="onStartTimeConfirm"
          @cancel="showStartTimePicker = false"
        >
          <van-date-picker v-model="formModel.startDate" :max-date="maxDate" />
          <van-time-picker v-model="formModel.startTime" />
        </van-picker-group>
      </van-popup>

      <van-field
        :model-value="formModel.leaveEndTime"
        name="leaveEndTime"
        label="结束时间"
        is-link
        readonly
        placeholder="请选择结束时间"
        @click="showEndTimePicker = true"
      />
      <van-popup round :show="showEndTimePicker" position="bottom">
        <van-picker-group
          next-step-text="下一步"
          title="结束时间"
          :tabs="['选择日期', '选择时间']"
          @confirm="onEndTimeConfirm"
          @cancel="showEndTimePicker = false"
        >
          <van-date-picker v-model="formModel.endDate" :filter="endDateFilter" @change="onEndDateChange" />
          <van-time-picker v-model="formModel.endTime" :filter="endTimeFilter" />
        </van-picker-group>
      </van-popup>
    </van-cell-group>
  </div>
</template>

<script setup lang="ts">
import dayjs from 'dayjs';
import { ref } from 'vue'
const maxDate = new Date(new Date().getFullYear() + 1, 11, 31)
const formModel = defineModel<any>()
const showStartTimePicker = ref(false)
const showEndTimePicker = ref(false)
function onStartTimeConfirm() {
  formModel.value.leaveStartTime = dayjs(`${formModel.value.startDate.join('-')} ${formModel.value.startTime.join(':')}`).format('YYYY-MM-DD HH:mm:ss')
  showStartTimePicker.value = false
}
function onEndTimeConfirm() {
  formModel.value.leaveEndTime = dayjs(`${formModel.value.endDate.join('-')} ${formModel.value.endTime.join(':')}`).format('YYYY-MM-DD HH:mm:ss')
  showEndTimePicker.value = false
}
function endDateFilter(type: any, options: any) {
  if (!formModel.value.leaveStartTime || !formModel.value.endDate?.length) {
    return options
  }

  const startDate = new Date(formModel.value.leaveStartTime)
  if (type === 'year') {
    return options.filter((option: any) => option.value >= startDate.getFullYear())
  }
  else if (type === 'month') {
    if (formModel.value.endDate[0] === String(startDate.getFullYear())) {
      return options.filter((option: any) => option.value > startDate.getMonth())
    }
  }
  else if (type === 'day') {
    if (formModel.value.endDate[0] === String(startDate.getFullYear())
      && formModel.value.endDate[1] === String(startDate.getMonth() + 1).padStart(2, '0')) {
      return options.filter((option: any) => option.value >= startDate.getDate())
    }
  }
  return options
}
function endTimeFilter(type:any, options: any, values: any) {
  if (!formModel.value.leaveStartTime || !formModel.value.endTime?.length) {
    return options
  }

  const startTime = new Date(dayjs(formModel.value.leaveStartTime).format('YYYY-MM-DD HH:mm:ss'))
  const endDate = new Date(dayjs(formModel.value.endDate.join('-')).format('YYYY-MM-DD HH:mm:ss'))
  if (type === 'hour') {
    if (endDate.getDate() === startTime.getDate()) {
      return options.filter((option: any) => Number(option.value) >= startTime.getHours())
    }
    else {
      return options
    }
  }
  else if (type === 'minute') {
    if (dayjs(startTime).isSame(dayjs(endDate), 'day') && values[0] === formModel.value.startTime[0]) {
      return options.filter((option: any) => Number(option.value) > startTime.getMinutes())
    }
    else {
      return options
    }
  }
}
function onEndDateChange({ selectedOptions }: any) {
  const endDate = selectedOptions.map((option: any) => option.value).join('-')
  if (dayjs(endDate).isAfter(dayjs(formModel.value.leaveStartTime))) {
    // not working
    formModel.value.endTime = ['00', '00']
  }
}
</script>

<style lang="less" scoped></style>
