# [vue-validator](https://github.com/yesixuan/vue-validate)
[![](https://img.shields.io/badge/Powered%20by-yesixuan%20base-brightgreen.svg)](https://github.com/yesixuan/vue-validate)
[![license](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/yesixuan/vue-validate/blob/master/LICENSE)
[![Coveralls](https://img.shields.io/coveralls/yanhaijing/jslib-base.svg)](https://coveralls.io/github/yesixuan/vue-validator)
[![npm](https://img.shields.io/badge/npm-0.1.0-orange.svg)](https://www.npmjs.com/package/@ignorance/vue-validator)
[![NPM downloads](http://img.shields.io/npm/dm/jslib-base.svg?style=flat-square)](http://www.npmtrends.com/@ignorance/vue-validator)

一个 VUE 表单校验插件.

**The library that based vue-validator can be shared to the [vue-validator](https://github.com/yesixuan) platform**

## 特色

1. 与 UI 组件库解耦，只提供最纯粹的校验功能（使用者可以自己选择使用校验结果来实现自己想要的功能）
2. 配置约定，通过配置来定义表单的校验规则。实现表单校验与业务逻辑解耦
3. 校验规则支持默认规则、正则表达式、校验函数
4. 支持扩展默认的规则。（通过 `extendRegexp` 扩展正则规则，通过 `extendValidator` 扩展校验函数）
5. 支持单个表单元素校验。（校验信息通过调用 `$verify(<name>)` 来获取）
6. 支持提交时，校验不通过则自动拦截提交操作（可配置，通过 `v-validate` 指令的修饰符 `autoCatch` 来自动拦截提交）

## 安装

```bash
npm install @ignorance/vue-validator --save-dev
```

## 使用

```js
// main.js
import validator from '@ignorance/vue-validator'
// ...
Vue.use(validator)
```

```vue
<template>
  <form ref="myForm">
    <input placeholder="姓名" v-model="formData.name" name="name" :class="{ error: $isError('name') }" />
    <input placeholder="电话" v-model="formData.tel" name="tel" :class="{ error: $isError('tel') }" />
    <select name="habit" v-model="formData.habit" :class="{ error: $isError('habit') }">
      <option value="">空</option>
      <option value="1">睡觉</option>
      <option value="2">打豆豆</option>
      <option value="3">错误选项</option>
    </select>
    <OwnerBtn text="保存" v-validate:submit.autoCatch="validateData" />
   
    姓名：{{ JSON.parse(JSON.stringify($verify('name'))) }}
    手机：{{ JSON.parse(JSON.stringify($verify('tel'))) }}
    爱好：{{ JSON.parse(JSON.stringify($verify('habit'))) }}
  </form>
</template>

<script>
export default {
  data() {
    return {
      formData: {
        name: '',
        tel: '',
        habit: ''
      }
    }
  },
  created() {
    // 传给校验插件的配置信息
    this.validateData = {
      ref: 'myForm',
      formData: 'formData',
      fields: [ 'name', 'tel', 'habit' ],
      rules: {
        name: [
          {
            validator: 'required',
            msg: '必填'
          },
          {
            validator: /^[a-zA-Z]+$/,
            msg: '只接受字母'
          },
          {
            validator: 'max:8 min:5',
            msg: '长度在 5 ~ 8 之间'
          }
        ],
        tel: [
          {
            validator: 'mobile',
            msg: '请输入正确的手机号码',
            trigger: 'input' // 可以省略，默认在 blur 时校验
          }
        ],
        habit: [
          {
            validator: 'required',
            msg: '必填'
          },
          {
            validator: val => val === '1' || val === '2' // 自定义校验函数
          }
        ]
      }
    }
  },
  methods: {
    submit() {
      const res = this.$refs.myForm.validator()
      console.log('执行 submit 方法')
    }
  }
}
</script>

<style scoped>
.error {
  color: red;
  border-color: red;
}
</style>
```

## API

### rules.extendRegexp()
> 扩展正则表达式规则

```js
import validator, { rules } from '@ignorance/vue-validator'

rules.extendRegexp({
  ruleName: regexp,
  // ...
})
```  

### rules.extendValidator()
> 扩展自定义校验规则

```js
import validator, { rules } from '@ignorance/vue-validator'

rules.extendValidator({
  ruleName: validator,
  // ...
})
``` 

### $isError(<name>)

原型方法：校验某字段是否校验不通过。  
```vue
<template>
  <input name="mobile" :class="{ error: $isError('mobile') }" />
</template>
```

### $verify(<name>)

原型方法：校验某字段是否校验信息（包含校验是否通过、不通过的提示信息）。
```js
$verify('mobile')
// { valid: false, msg: '请输入正确的手机号码' }
```

### formRef.validator()

表单引用对象上的校验方法（用于在提交时作整体校验）
```js
function submit() {
  const { valid, msg } = this.$refs.myForm.validator()
  if (valid) {
    // do something
  } else {
    alert(msg)
  }
}
```

