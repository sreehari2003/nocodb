<script lang="ts" setup>
import dayjs from 'dayjs'

interface Props {
  selectedDate?: dayjs.Dayjs | null
  pageDate?: dayjs.Dayjs
  isYearPicker?: boolean
  hideCalendar?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  selectedDate: null,
  pageDate: dayjs(),
  isYearPicker: false,
  hideCalendar: false,
})
const emit = defineEmits(['update:selectedDate', 'update:pageDate'])

const pageDate = useVModel(props, 'pageDate', emit)

const selectedDate = useVModel(props, 'selectedDate', emit)

const years = computed(() => {
  const date = pageDate.value
  const startOfYear = date.startOf('year')
  const years: dayjs.Dayjs[] = []
  for (let i = 0; i < 12; i++) {
    years.push(dayjs(startOfYear).add(i, 'year'))
  }
  return years
})

const months = computed(() => {
  const months: dayjs.Dayjs[] = []
  for (let i = 0; i < 12; i++) {
    months.push(pageDate.value.set('month', i))
  }
  return months
})

const compareDates = (date1: dayjs.Dayjs, date2: dayjs.Dayjs) => {
  if (!date1 || !date2) return false
  return date1.isSame(date2, 'month') && date1.isSame(date2, 'year')
}

const isMonthSelected = (date: dayjs.Dayjs) => {
  if (!dayjs(selectedDate.value).isValid()) return false
  return compareDates(date, selectedDate.value)
}

const paginateMonth = (action: 'next' | 'prev') => {
  let date = pageDate.value
  if (action === 'next') {
    date = date.add(1, 'year')
  } else {
    date = date.subtract(1, 'year')
  }
  pageDate.value = date
  emit('update:pageDate', date)
}

const paginateYear = (action: 'next' | 'prev') => {
  let date = dayjs(pageDate.value)
  if (action === 'next') {
    date = date.add(12, 'year')
  } else {
    date = date.subtract(12, 'year')
  }
  pageDate.value = date
  emit('update:pageDate', date)
}

const paginate = (action: 'next' | 'prev') => {
  if (props.isYearPicker) {
    paginateYear(action)
  } else {
    paginateMonth(action)
  }
}

const compareYear = (date1: dayjs.Dayjs, date2: dayjs.Dayjs) => {
  if (!date1 || !date2) return false
  return date1.isSame(date2, 'year')
}
</script>

<template>
  <div class="flex flex-col">
    <div class="flex px-2 border-b-1 py-0.5 justify-between items-center">
      <div class="flex">
        <NcTooltip hide-on-click>
          <NcButton class="!border-0" size="small" type="secondary" @click="paginate('prev')">
            <component :is="iconMap.arrowLeft" class="h-4 w-4" />
          </NcButton>
          <template #title>
            <span>{{ $t('labels.next') }}</span>
          </template>
        </NcTooltip>
      </div>

      <span class="text-gray-700 font-semibold">{{
        isYearPicker ? dayjs(selectedDate).year() : dayjs(pageDate).format('YYYY')
      }}</span>
      <div class="flex">
        <NcTooltip hide-on-click>
          <NcButton class="!border-0" size="small" type="secondary" @click="paginate('next')">
            <component :is="iconMap.arrowRight" class="h-4 w-4" />
          </NcButton>
          <template #title>
            <span>{{ $t('labels.next') }}</span>
          </template>
        </NcTooltip>
      </div>
    </div>
    <div v-if="!hideCalendar" class="rounded-y-xl px-2.5 py-1 max-w-[350px]">
      <div class="grid grid-cols-4 gap-2">
        <template v-if="!isYearPicker">
          <span
            v-for="(month, id) in months"
            :key="id"
            :class="{
              '!bg-gray-200 !text-brand-900 !font-bold ': isMonthSelected(month),
              '!text-brand-500': dayjs().isSame(month, 'month'),
            }"
            class="h-8 rounded-lg flex items-center transition-all font-medium justify-center hover:(border-1 border-gray-200 bg-gray-100) text-gray-700 cursor-pointer"
            @click="selectedDate = month"
          >
            {{ month.format('MMM') }}
          </span>
        </template>
        <template v-else>
          <span
            v-for="(year, id) in years"
            :key="id"
            :class="{
              '!bg-gray-200 !text-brand-500 !font-bold ': compareYear(year, selectedDate),
              '!text-brand-500': dayjs().isSame(year, 'year'),
            }"
            class="h-8 rounded-lg flex items-center transition-all font-medium justify-center hover:(border-1 border-gray-200 bg-gray-100) text-gray-900 cursor-pointer"
            @click="selectedDate = year"
          >
            {{ year.format('YYYY') }}
          </span>
        </template>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>
