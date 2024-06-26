<script>
import { defineProps, ref, reactive, computed, provide, getCurrentInstance, watch, nextTick, onMounted } from 'vue'
import CanvesPanel from '@ER/formEditor/components/Panels/Canves'
import hooks from '@ER/hooks'
import utils from '@ER/utils'
import _ from 'lodash-es'
import defaultProps from './defaultProps'
import { ElMessage } from 'element-plus'
import { showNotify } from 'vant'
export default {
  name: 'Everright-form-preview'
}
</script>
<script setup>
const emit = defineEmits(['listener'])
const props = defineProps(defaultProps)
const layout = {
  pc: [],
  mobile: []
}
const form = ref('')
const state = reactive({
  store: [],
  selection: {},
  platform: utils.isPc() ? 'pc' : 'mobile',
  mode: 'preview',
  config: {},
  Namespace: 'formEditor',
  validateStates: [],
  data: {},
  fields: [],
  logic: {},
  fieldsLogicState: new Map(),
  remoteValues: new Map(),
  othersFiles: {}
})
const ns = hooks.useNamespace('Main', state.Namespace)
hooks.useLogic(state)
const getData = () => {
  const result = {}
  state.fields.forEach(e => {
    if (e.type === 'subform') {
      result[e.key] = utils.getSubFormValues(e)
    } else {
      try {
        if (!utils.checkIsInSubform(e)) {
          result[e.key] = e.options.defaultValue
        }
      } catch (e) {
      }
    }
  })
  return _.cloneDeep(_.merge(result, state.othersFiles))
}
const fireEvent = (type, data) => {
  emit('listener', {
    type,
    data
  })
}
const setValue = (field, value) => {
  if (field.type === 'time' && !field.options.valueFormat) {
    field.options.valueFormat = 'HH:mm:ss'
  }
  field.options.defaultValue = value
}
provide('Everright', {
  state,
  getData,
  props,
  fireEvent,
  setValue,
  form
})
const setOhters = (value) => {
  _.toPairs(value).filter(([key]) => key.indexOf('_other') !== -1).forEach(([key, val]) => {
    state.othersFiles[key] = val
  })
}
const setData2 = (data, value) => {
  const newData = _.cloneDeep(data)
  layout.pc = newData.layout.pc
  layout.mobile = newData.layout.mobile
  state.store = newData.list
  state.fields = newData.fields
  const curLayout = _.cloneDeep(newData.layout[state.platform])
  utils.combinationData2(curLayout, state.fields)
  state.store = curLayout
  state.config = newData.config
  state.data = newData.data
  state.logic = newData.logic
  state.store.forEach((e) => {
    utils.addContext(e, state.store, false)
  })
  if (!_.isEmpty(value)) {
    setOhters(value)
    state.fields.forEach((e) => {
      if (e.type === 'time' && !e.options.valueFormat) {
        e.options.valueFormat = 'HH:mm:ss'
      }
      if (value[e.key]) {
        e.options.defaultValue = value[e.key]
      }
    })
  }
}
const setData1 = async (data, value) => {
  if (_.isEmpty(data)) return false
  const newData = utils.combinationData1(_.cloneDeep(data))
  state.store = newData.list
  state.config = newData.config
  state.data = newData.data
  state.fields = newData.fields
  state.logic = newData.logic
  state.store.forEach((e) => {
    utils.addContext(e, state.store)
  })
  const subforms = _.cloneDeep(state.fields.filter(e => e.type === 'subform'))
  if (!_.isEmpty(value)) {
    setOhters(value)
    for (const key in value) {
      state.remoteValues.set(key, value[key])
    }
    state.fields.forEach((field) => {
      if (field.type !== 'subform') {
        try {
          if (!utils.checkIsInSubform(field)) {
            setValue(field, value[field.key])
          }
        } catch (e) {
        }
      }
    })
  }
}
const setData = props.layoutType === 1 ? setData1 : setData2
defineExpose({
  switchPlatform (platform) {
    state.platform = platform
  },
  setData,
  getData,
  form
})
</script>
<template>
  <CanvesPanel v-if="state.store.length"></CanvesPanel>
</template>
