<template>
	<wd-select-picker
			:label-width="labelWidth"
			:label="label"
			v-model="selected"
			filterable
			clearable
			:columns="options"
			:disabled="disabled"
			placeholder="请选择"
			@change="handleChange"
	></wd-select-picker>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import { isArray, isString } from 'lodash';
import {http} from "@/utils/http"; // 假设使用 lodash 来判断类型

// 定义 props
const props = defineProps({
	dict: {
		type: [Array, String],
		default: () => [],
		required: true,
	},
	label: {
		type: String,
		default: '',
		required: false,
	},
	labelWidth: {
		type: String,
		default: '80px',
		required: false,
	},
	name: {
		type: String,
		default: '',
		required: false,
	},
	dictStr: {
		type: String,
		default: '',
		required: false,
	},
	type: {
		type: String,
		default: '',
		required: false,
	},
  modelValue: {
		type: [Array, String],
		required: false,
	},
	disabled: {
		type: Boolean,
		default: false,
		required: false,
	},
})

// 定义 emits
const emit = defineEmits(['change', 'update:modelValue'])

// 定义响应式数据
const selected = ref([]);
const options = ref([]);

// 初始化选项
const initSelections = async () => {
	options.value = []
	if (props.type === 'sel_search' && props.dictStr) {
		let temp = props.dictStr
		if (temp.indexOf(' ') > 0) {
			temp = encodeURI(props.dictStr)
		}
		try {
			const res = await http.get('/sys/dict/getDictItems/' + temp)
			if (res.success) {
				options.value = res.result
			}
		} catch (error) {
			console.error('请求数据出错:', error)
		}
	}
	else {
		if (!props.dict || props.dict.length === 0) {
			return
		}
		if (isString(props.dict)) {
			try {
        let code = props.dict;
        if (code.indexOf(',') > 0 &&  code.indexOf(' ') > 0) {
          // 编码后类似sys_user%20where%20username%20like%20xxx' 是不包含空格的,这里判断如果有空格和逗号说明需要编码处理
          code = encodeURI(code);
        }
				const res = await http.get('/sys/dict/getDictItems/' + code)
				if (res.success) {
					options.value = res.result
				}
			} catch (error) {
				console.error('请求数据出错:', error)
			}
		} else {
			props.dict.forEach((item) => {
				options.value.push(item)
			})
		}
	}
}

// 选择器改变事件处理函数
const handleChange = ({value}) => {
  let valueStr='';
	if (value && isArray(value)) {
    valueStr = value.join(',')
	}
	emit('update:modelValue', valueStr);
  emit('change', valueStr);
}

// 监听 dict 和 value 的变化
watch(() => props.dict, () => {
	initSelections();
});
// 监听value 的变化
watchEffect(()=>{
  props.modelValue && initVal()
})
//初始化数值
function initVal(){
  selected.value = isString(props.modelValue)?  props.modelValue.split(',') :[];
}
// 组件挂载时初始化选项
onMounted(() => {
	initSelections()
})
</script>

<style lang="scss" scoped>
  :deep(.wd-checkbox__shape){
    border-radius:0 !important;
  }
</style>
