<template>
  <view class="uni-container">
    <uni-forms ref="form" :model="formData" validate-trigger="submit" err-show-type="toast">
      <uni-forms-item name="name" label="姓名">
        <undefined v-model="formData.name"></undefined>
      </uni-forms-item>
      <uni-forms-item name="age" label="年龄">
        <undefined v-model="formData.age"></undefined>
      </uni-forms-item>
      <uni-forms-item name="gender" label="性别">
        <undefined v-model="formData.gender"></undefined>
      </uni-forms-item>
      <uni-forms-item name="phone" label="电话">
        <undefined v-model="formData.phone"></undefined>
      </uni-forms-item>
      <uni-forms-item name="address" label="地址">
        <undefined v-model="formData.address"></undefined>
      </uni-forms-item>
      <uni-forms-item name="createTime" label="创建时间">
        <undefined v-model="formData.createTime"></undefined>
      </uni-forms-item>
      <uni-forms-item name="updateTime" label="更新时间">
        <undefined v-model="formData.updateTime"></undefined>
      </uni-forms-item>
      <view class="uni-button-group">
        <button type="primary" class="uni-button" @click="submit">提交</button>
      </view>
    </uni-forms>
  </view>
</template>

<script>
  import { validator } from '../../js_sdk/validator/user-dome.js';

  const db = uniCloud.database();
  const dbCollectionName = 'user-dome';

  function getValidator(fields) {
    let result = {}
    for (let key in validator) {
      if (fields.indexOf(key) > -1) {
        result[key] = validator[key]
      }
    }
    return result
  }

  

  export default {
    data() {
      let formData = {
        "name": null,
        "age": null,
        "gender": null,
        "phone": null,
        "address": null,
        "createTime": null,
        "updateTime": null
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
            icon: 'none',
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

<style>
  .uni-container {
    padding: 15px;
  }

  .uni-input-border,
  .uni-textarea-border {
    width: 100%;
    font-size: 14px;
    color: #666;
    border: 1px #e5e5e5 solid;
    border-radius: 5px;
    box-sizing: border-box;
  }

  .uni-input-border {
    padding: 0 10px;
    height: 35px;

  }

  .uni-textarea-border {
    padding: 10px;
    height: 80px;
  }

  .uni-button-group {
    margin-top: 50px;
    /* #ifndef APP-NVUE */
    display: flex;
    /* #endif */
    justify-content: center;
  }

  .uni-button {
    width: 184px;
  }
</style>
