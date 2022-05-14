<template>
  <view class="uni-container">
    <uni-forms ref="form" :value="formData" validateTrigger="bind">
      <uni-forms-item name="date" label="预约表中的日期">
        <uni-datetime-picker return-type="date" v-model="formData.date"></uni-datetime-picker>
      </uni-forms-item>
      <uni-forms-item name="time" label="预约时间段">
        <uni-easyinput placeholder="用户名，不允许重复" v-model="formData.time" trim="both"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="status" label="预约状态">
        <uni-easyinput placeholder="可预约或已被预约" v-model="formData.status" trim="both"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="tablePerson" label="预约表所属人">
        <uni-easyinput v-model="formData.tablePerson"></uni-easyinput>
      </uni-forms-item>
      <view class="uni-button-group">
        <button type="primary" class="uni-button" style="width: 100px;" @click="submit">提交</button>
        <navigator open-type="navigateBack" style="margin-left: 15px;">
          <button class="uni-button" style="width: 100px;">返回</button>
        </navigator>
      </view>
    </uni-forms>
  </view>
</template>

<script>
  import { validator } from '../../js_sdk/validator/timeTable_xiawei.js';

  const db = uniCloud.database();
  const dbCmd = db.command;
  const dbCollectionName = 'timeTable_xiawei';

  function getValidator(fields) {
    let result = {}
    for (let key in validator) {
      if (fields.includes(key)) {
        result[key] = validator[key]
      }
    }
    return result
  }

  export default {
    data() {
      let formData = {
        "date": null,
        "time": "",
        "status": "",
        "tablePerson": ""
      }
      return {
        formData,
        formOptions: {},
        rules: {
          ...getValidator(Object.keys(formData))
        }
      }
    },
    onReady() {
      this.$refs.form.setRules(this.rules)
    },
    methods: {
      /**
       * 验证表单并提交
       */
      submit() {
        uni.showLoading({
          mask: true
        })
        this.$refs.form.validate().then((res) => {
          return this.submitForm(res)
        }).catch(() => {
        }).finally(() => {
          uni.hideLoading()
        })
      },

      /**
       * 提交表单
       */
      submitForm(value) {
        // 使用 clientDB 提交数据
        return db.collection(dbCollectionName).add(value).then((res) => {
          uni.showToast({
            title: '新增成功'
          })
          this.getOpenerEventChannel().emit('refreshData')
          setTimeout(() => uni.navigateBack(), 500)
        }).catch((err) => {
          uni.showModal({
            content: err.message || '请求服务失败',
            showCancel: false
          })
        })
      }
    }
  }
</script>
