<template>
  <div>
    <el-dialog v-if="showGatherInfoDialog" top="8vh" :visible.sync="showGatherInfoDialog" width="772px" center class="gather-info-dialog" :before-close="handleClose" :append-to-body="true">
      <div class="gather-info-dialog-title">
        <h4 class="gather-info-dialog-title-text">合作咨询</h4>
        <p class="gather-info-dialog-title-hint">我们的销售代表会尽快联系您</p>
      </div>
      <div class="gather-info-dialog-content">
        <div class="gather-info-dialog-content-reminder">
          <img class="gather-info-dialog-content-reminder-icon" src="@/asset/images/hint.png" alt="hint">
          表单提交成功后我们将立即发送“下载与体验”的资料到您的邮箱，并将于1-3个工作日与您联系。
        </div>
        <el-form :model="infoData" :rules="infoDataRules" ref="infoForm" label-position="top" label-width="100px" class="gather-info-dialog-content-form">
          <el-form-item label="联系人姓名：" prop="realname">
            <el-input placeholder="请输入您的姓名" v-model="infoData.realname"></el-input>
          </el-form-item>
          <el-form-item label="联系手机：" prop="cellphone">
            <el-input placeholder="请输入手机号码" v-model="infoData.cellphone"></el-input>
          </el-form-item>
          <el-form-item label="邮箱：" prop="email">
            <el-input placeholder="请输入邮箱" v-model="infoData.email"></el-input>
          </el-form-item>
          <el-form-item label="所属单位名称：" prop="organization">
            <el-input placeholder="请输入单位或者机构名称" v-model="infoData.organization"></el-input>
          </el-form-item>
          <el-form-item label="所在城市：" prop="city">
            <el-cascader placeholder="" :options="cityName" filterable change-on-select v-model="infoData.city"></el-cascader>
          </el-form-item>
          <el-form-item label="机构规模：" prop="scale">
            <el-input placeholder="请输入单位或机构的规模人数" v-model="infoData.scale"></el-input>
          </el-form-item>
          <el-form-item label="描述：">
            <el-input type="textarea" placeholder="请简单描述一下你想了解的问题" v-model="infoData.description"></el-input>
          </el-form-item>
          <el-form-item class="gather-info-dialog-content-form-submit">
            <el-button class="gather-info-dialog-content-form-submit-btn" type="primary" @click="submitInfo()" :loading="submitSuccessLoading">提交</el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import axios from 'axios'
import cityName from '@/component/data/cityName.js'

export default {
  name: 'GetInfoDialog',
  props: {
    showGatherInfoDialog: Boolean
  },
  data() {
    let validatePhoneNumber = (rule, value, callback) => {
      if (!/^1[34578]\d{9}$/.test(value)) {
        callback(new Error('请输入正确的手机号'))
      } else {
        callback()
      }
    }
    return {
      submitSuccessLoading: false,
      infoData: {
        realname: '',
        cellphone: '',
        email: '',
        organization: '',
        city: [],
        scale: undefined,
        description: ''
      },
      infoDataRules: {
        realname: [
          { required: true, message: '请输入您的姓名', trigger: 'blur' }
        ],
        cellphone: [
          {
            required: true,
            message: '请输入手机号码',
            trigger: 'blur'
          },
          { validator: validatePhoneNumber, trigger: ['change', 'blur'] }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: ['blur', 'change']
          }
        ],
        organization: [
          { required: true, message: '请输入您所在的单位名称', trigger: 'blur' }
        ],
        city: [
          { required: true, message: '请选择您所在的城市', trigger: 'blur' }
        ]
      },
      cityName: cityName
    }
  },
  methods: {
    handleClose(infoData = { isSubmitOperation: 0 }) {
      this.$emit('close', infoData)
    },
    submitInfo() {
      let baseUrl = process.env.KEEPWORK_API_PREFIX
      let { city, scale, ..._infoData } = this.infoData
      this.$refs.infoForm.validate(valid => {
        if (valid) {
          this.submitSuccessLoading = true
          axios
            .post(`${baseUrl}/paracraftVisitors/upsert`, {
              ..._infoData,
              extra: {
                scale,
                city
              }
            })
            .then(res => {
              this.handleClose({ ...this.infoData, isSubmitOperation: 1 })
              this.submitSuccessLoading = false
            })
            .catch(err => {
              console.error('err', err)
            })
        } else {
          return false
        }
      })
    }
  }
}
</script>

<style lang="scss">
.gather-info-dialog {
  .el-dialog {
    .el-dialog__header,
    .el-dialog__body {
      padding: 0;
    }
    .el-dialog__header {
      .el-dialog__headerbtn {
        .el-icon-close:before {
          color: #fff;
        }
      }
    }
  }
  &-title {
    height: 129px;
    background: url(../../asset/images/mianfei.jpg);
    color: #fff;
    text-align: center;
    &-text {
      font-size: 32px;
      margin: 0;
      padding-top: 36px;
    }
    &-hint {
    }
  }
  &-content {
    max-width: 635px;
    margin: 0 auto;
    padding: 0 10px;
    &-reminder {
      background-color: #f1f6fa;
      border: solid 1px #e0edf2;
      margin: 20px 0 25px;
      display: flex;
      align-items: center;
      padding: 8px 15px;
      font-size: 12px;
      &-icon {
        margin-right: 13px;
      }
    }
    &-form {
      .el-form-item {
        margin-bottom: 16px;
        .el-form-item__label {
          line-height: 20px;
        }
        .el-form-item__content {
          line-height: 32px;
          .el-input {
            .el-input__inner {
              height: 32px;
              line-height: 32px;
            }
          }
        }
      }
      &-submit {
        text-align: center;
        padding: 13px 0 47px;
        margin: 0;
        &-btn {
          width: 139px;
        }
      }
    }
  }
}
@media screen and (max-width: 769px) {
  .gather-info-dialog {
    .el-dialog {
      width: 100% !important;
      margin-top: 0 !important;
    }
  }
}
</style>

