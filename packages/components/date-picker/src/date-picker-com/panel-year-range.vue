<template>
  <div
    class="el-picker-panel el-date-range-picker"
    :class="[
      {
        'has-sidebar': $slots.sidebar || hasShortcuts,
      },
    ]"
  >
    <div class="el-picker-panel__body-wrapper">
      <slot name="sidebar" class="el-picker-panel__sidebar" />
      <div v-if="hasShortcuts" class="el-picker-panel__sidebar">
        <button
          v-for="(shortcut, key) in shortcuts"
          :key="key"
          type="button"
          class="el-picker-panel__shortcut"
          @click="handleShortcutClick(shortcut)"
        >
          {{ shortcut.text }}
        </button>
      </div>
      <div class="el-picker-panel__body">
        <div
          class="el-picker-panel__content el-date-range-picker__content is-left"
        >
          <div class="el-date-range-picker__header">
            <button
              type="button"
              class="el-picker-panel__icon-btn el-icon-d-arrow-left"
              @click="leftPrevYear"
            >
              <el-icon :class="['arrow-down']">
                <DArrowLeft />
              </el-icon>
            </button>
            <template v-if="unlinkPanels">
              <button
                type="button"
                :disabled="!enableYearArrow"
                :class="{ 'is-disabled': !enableYearArrow }"
                class="el-picker-panel__icon-btn el-icon-d-arrow-right"
                @click="leftNextYear"
              >
                <el-icon>
                  <DArrowRight />
                </el-icon>
              </button>
            </template>
            <div>{{ leftLabel }}</div>
          </div>
          <year-table
            selection-mode="range"
            :date="leftDate"
            :min-date="minDate"
            :max-date="maxDate"
            :range-state="rangeState"
            :disabled-date="disabledDate"
            :cell-class-name="cellClassName"
            @changerange="handleChangeRange"
            @pick="handleRangePick"
            @select="onSelect"
          />
        </div>
        <div
          class="el-picker-panel__content el-date-range-picker__content is-right"
        >
          <div class="el-date-range-picker__header">
            <template v-if="unlinkPanels">
              <button
                type="button"
                :disabled="!enableYearArrow"
                :class="{ 'is-disabled': !enableYearArrow }"
                class="el-picker-panel__icon-btn el-icon-d-arrow-left"
                @click="rightPrevYear"
              >
                <el-icon>
                  <DArrowLeft />
                </el-icon>
              </button>
            </template>
            <button
              type="button"
              class="el-picker-panel__icon-btn el-icon-d-arrow-right"
              @click="rightNextYear"
            >
              <el-icon>
                <DArrowRight />
              </el-icon>
            </button>
            <div>{{ rightLabel }}</div>
          </div>
          <year-table
            selection-mode="range"
            :date="rightDate"
            :min-date="minDate"
            :max-date="maxDate"
            :range-state="rangeState"
            :disabled-date="disabledDate"
            :cell-class-name="cellClassName"
            @changerange="handleChangeRange"
            @pick="handleRangePick"
            @select="onSelect"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { computed, defineComponent, inject, ref, watch } from 'vue'
import dayjs from 'dayjs'
import { ElIcon } from 'element-plus'
import { useLocale } from '@element-plus/hooks'
import { ClickOutside } from '@element-plus/directives'
import { DArrowLeft, DArrowRight } from '@element-plus/icons-vue'
import YearTable from './basic-year-table.vue'

export default defineComponent({
  directives: { clickoutside: ClickOutside },

  components: {
    YearTable,
    ElIcon,
    DArrowLeft,
    DArrowRight,
  },

  props: {
    unlinkPanels: Boolean,
    parsedValue: {
      type: Array,
    },
  },

  emits: ['pick', 'set-picker-option'],

  setup(props, ctx) {
    const { t } = useLocale()

    const leftDate = ref(dayjs())
    const rightDate = ref(dayjs().add(10, 'year'))
    const minDate = ref(null)
    const maxDate = ref(null)

    const leftLabel = computed(() => {
      return `${Math.floor(leftDate.value.year() / 10) * 10} ${t(
        'el.datepicker.year'
      )}`
    })

    const rightLabel = computed(() => {
      return `${Math.floor(rightDate.value.year() / 10) * 10 + 9} ${t(
        'el.datepicker.year'
      )}`
    })

    const pickerBase = inject('EP_PICKER_BASE')
    const { shortcuts, disabledDate, cellClassName, format, defaultValue } =
      pickerBase.props

    const hasShortcuts = computed(() => !!shortcuts.length)

    const leftPrevYear = () => {
      leftDate.value = leftDate.value.subtract(10, 'year')
      if (!props.unlinkPanels) {
        rightDate.value = leftDate.value.add(10, 'year')
      }
    }

    const rightNextYear = () => {
      if (!props.unlinkPanels) {
        leftDate.value = leftDate.value.add(10, 'year')
        rightDate.value = leftDate.value.add(10, 'year')
      } else {
        rightDate.value = rightDate.value.add(10, 'year')
      }
    }

    const leftNextYear = () => {
      leftDate.value = leftDate.value.add(10, 'year')
    }

    const rightPrevYear = () => {
      rightDate.value = rightDate.value.subtract(10, 'year')
    }

    const isValidValue = (value) => {
      return (
        Array.isArray(value) &&
        value &&
        value[0] &&
        value[1] &&
        value[0].valueOf() <= value[1].valueOf()
      )
    }

    const rangeState = ref({
      endDate: null,
      selecting: false,
    })

    const handleChangeRange = (val) => {
      rangeState.value = val
    }

    const onSelect = (selecting) => {
      rangeState.value.selecting = selecting
      if (!selecting) {
        rangeState.value.endDate = null
      }
    }

    const handleConfirm = (visible = false) => {
      if (isValidValue([minDate.value, maxDate.value])) {
        ctx.emit('pick', [minDate.value, maxDate.value], visible)
      }
    }

    const handleRangePick = (val, close = true) => {
      const minDate_ = val.minDate
      const maxDate_ = val.maxDate
      if (maxDate.value === maxDate_ && minDate.value === minDate_) {
        return
      }
      maxDate.value = maxDate_
      minDate.value = minDate_

      if (!close) return
      handleConfirm()
    }

    const handleShortcutClick = (shortcut) => {
      if (shortcut.value) {
        ctx.emit('pick', [dayjs(shortcut.value[0]), dayjs(shortcut.value[1])])
        return
      }
      if (shortcut.onClick) {
        shortcut.onClick(ctx)
      }
    }

    const getDefaultValue = () => {
      let start
      if (Array.isArray(defaultValue)) {
        const left = dayjs(defaultValue[0])
        let right = dayjs(defaultValue[1])
        if (!props.unlinkPanels) {
          right = left.add(10, 'year')
        }
        return [left, right]
      } else if (defaultValue) {
        start = dayjs(defaultValue)
      } else {
        start = dayjs()
      }
      return [start, start.add(10, 'year')]
    }

    const handleClear = () => {
      minDate.value = null
      maxDate.value = null
      leftDate.value = getDefaultValue()[0]
      rightDate.value = leftDate.value.add(10, 'year')
      ctx.emit('pick', null)
    }

    const formatToString = (value) => {
      return Array.isArray(value)
        ? value.map((_) => _.format(format))
        : value.format(format)
    }

    const parseUserInput = (value) => {
      return Array.isArray(value)
        ? value.map((_) => dayjs(_, format))
        : dayjs(value, format)
    }

    ctx.emit('set-picker-option', ['isValidValue', isValidValue])
    ctx.emit('set-picker-option', ['parseUserInput', parseUserInput])
    ctx.emit('set-picker-option', ['formatToString', formatToString])
    ctx.emit('set-picker-option', ['handleClear', handleClear])

    watch(
      () => props.parsedValue,
      (newVal) => {
        if (newVal && newVal.length === 2) {
          minDate.value = newVal[0]
          maxDate.value = newVal[1]

          leftDate.value = minDate.value
          if (props.unlinkPanels && maxDate.value) {
            const minDateYear = minDate.value.year()
            const maxDateYear = maxDate.value.year()
            rightDate.value =
              minDateYear === maxDateYear
                ? maxDate.value.add(10, 'year')
                : maxDate.value
          } else {
            rightDate.value = leftDate.value.add(10, 'year')
          }
        } else {
          const defaultArr = getDefaultValue()
          leftDate.value = defaultArr[0]
          rightDate.value = defaultArr[1]
        }
      },
      { immediate: true }
    )

    return {
      shortcuts,
      disabledDate,
      cellClassName,
      handleShortcutClick,
      rangeState,
      minDate,
      maxDate,
      handleRangePick,
      onSelect,
      handleChangeRange,
      rightPrevYear,
      rightNextYear,
      leftPrevYear,
      leftNextYear,
      hasShortcuts,
      leftLabel,
      rightLabel,
      leftDate,
      rightDate,
      t,
    }
  },
})
</script>
