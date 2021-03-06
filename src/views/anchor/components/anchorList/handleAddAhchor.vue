<!--
  添加/修改banner
-->
<template>
  <div class="bannerDialog">
    <el-dialog
      :title="title"
      :width="width"
      :center="center"
      :top="top"
      :close-on-press-escape="isEscClose"
      :close-on-click-modal="isMaskClose"
      :visible.sync="dialogVisiable"
      @close="handleClose"
    >
      <el-form class="formWrap" :model="formData" :rules="rules" ref="formRef">
        <el-form-item label="ID：" prop="userId" :label-width="labelWidth" size="small">
          <el-input v-model="formData.userId" size="small"></el-input>
        </el-form-item>

        <el-form-item label="姓名：" prop="name" :label-width="labelWidth" size="small">
          <el-input v-model="formData.name" size="mini"></el-input>
        </el-form-item>

        <el-form-item label="手机号：" prop="phone" :label-width="labelWidth" size="small">
          <el-input v-model="formData.phone" type="number" size="mini"></el-input>
        </el-form-item>

        <el-form-item label="证件类型：" prop="type" :label-width="labelWidth" size="small">
          <el-select v-model="formData.type" placeholder="请选择证件类型" size="small" @change="handleSelChange">
            <el-option label="身份证" value="1"></el-option>
            <el-option label="护照" value="2"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="证件号码：" prop="certificateNo" :label-width="labelWidth" size="small">
          <el-input v-model="formData.certificateNo" size="small"></el-input>
        </el-form-item>

        <div class="cerfiticateImgBox">
          <el-form-item label="证件照片：" prop="photoPositive" :label-width="labelWidth" size="small">
            <div class="frontImgBox">
              <img-upload
                v-model="formData.photoPositive"
                ref="frontImgRef"
                :limit="1"
                :size="10240"
                @success="handleFrontImgSuccess"
              ></img-upload>
              <p class="txt">(正面)</p>
            </div>
          </el-form-item>

          <el-form-item label="" prop="photoSide" label-width="20px" size="small">
            <div class="frontImgBox">
              <img-upload
                v-model="formData.photoSide"
                ref="sideImgRef"
                :limit="1"
                :size="10240"
                @success="handleSideImgSuccess"
              ></img-upload>
              <p class="txt">(反面)</p>
            </div>
          </el-form-item>
        </div>

        <el-form-item label="认证视频：" prop="videoUrl" :label-width="labelWidth" size="small">
          <video-upload
            v-model="formData.videoUrl"
            :limit="1"
            :size="102400"
            @success="handleCertifyVideoSuccess"
          ></video-upload>
        </el-form-item>

        <el-form-item label="简介：" prop="introduction" :label-width="labelWidth" size="small">
          <el-input
            v-model="formData.introduction"
            type="textarea"
            :rows="4"
            placeholder="请输入个人简介"
            maxlength="100"
            show-word-limit
          ></el-input>
        </el-form-item>
      </el-form>
      <div class="dialog-footer" slot="footer">
        <!-- <el-button @click="onReset">重置</el-button> -->
        <el-button type="primary" @click="onSubmit">确定提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import imgUpload from '@/components/imgUpload'
import videoUpload from '@/components/videoUpload'
import { validIdCard, validPassport } from '@/utils/validate'
import { addAnchor, editAnchor } from '@/api/anchor'
import { mapState } from 'vuex'
export default {
  name: '',
  data() {
    const validIC = (rule, value, callback) => {
      console.log(rule, value)
      if (!value) {
        return callback(new Error('证件号不能为空'))
      } else if (!validIdCard(value)) {
        return callback(new Error('请输入正确的身份证信息'))
      } else {
        callback()
      }
    }
    const validCerfiticateNo = (rule, value, callback) => {
      console.log(rule, value)
      if (!value) {
        return callback(new Error('证件号不能为空'))
      } else if (!validPassport(value)) {
        return callback(new Error('请输入正确的护照信息'))
      } else {
        callback()
      }
    }
    return {
      initData: {
        userId: '',
        name: '',
        phone: '',
        type: '1',
        certificateNo: '',
        photoPositive: [],
        photoSide: [],
        videoUrl: [],
        introduction: ''
      },
      idCardRules: {
        certificateNo: [{ validator: validIC, trigger: 'blur' }]
      },
      passportRules: {
        cerfiticateNo: [{ validator: validCerfiticateNo, trigger: 'blur' }]
      }
    }
  },
  props: {
    dialogVis: {
      type: Boolean,
      default: false
    },
    // 标题
    title: {
      type: String,
      default: ''
    },
    width: {
      type: String,
      default: '600px'
    },
    center: {
      type: String,
      default: ''
    },
    top: {
      type: String,
      default: '8vh'
    },
    // 是否按esc关闭dialog,默认为true
    isEscClose: {
      type: Boolean,
      default: true
    },
    // 是否点空白处遮罩关闭dialog，默认为true
    isMaskClose: {
      type: Boolean,
      default: true
    },
    labelWidth: {
      type: String,
      default: '140px'
    },
    // 添加/修改，默认为添加 add
    type: {
      type: String,
      default: 'add'
    },
    // 初始formData
    initFormData: {
      type: Object,
      default: () => {
        return {}
      }
    },
    infoData: {
      type: Object,
      required: true,
      default: () => {
        return {}
      }
    }
  },
  computed: {
    ...mapState('user', ['guildId']),
    dialogVisiable: {
      get() {
        return this.dialogVis
      },
      set(val) {
        this.$emit('update:dialogVis', val)
      }
    },

    formData: {
      get() {
        return this.infoData
      },
      set(val) {
        this.$emit('update:infoData', val)
      }
    },
    rules() {
      return {
        userId: [{ required: true, message: '请填写用户名', trigger: 'blur' }],
        name: [{ required: true, message: '请填写姓名', trigger: 'blur' }],
        phone: [{ required: true, message: '请填写手机号码', trigger: 'blur' }],
        type: [{ required: true, message: '请选择证件类型', trigger: 'change' }],
        certificateNo: [{ required: true, message: '请填写证件号码', trigger: 'blur' }],
        photoPositive: [{ required: true, message: '请上传证件照片(正面)', trigger: 'blur' }],
        photoSide: [{ required: true, message: '请上传证件照片(反面)', trigger: 'blur' }],
        videoUrl: [{ required: true, message: '请上传认证视频', trigger: 'blur' }],
        introduction: [{ required: true, message: '请填写个人简介', trigger: 'blur' }]
      }
    }
  },
  watch: {
    infoData: {
      handler(newVal, oldVal) {
        // console.log('-watch-formData-', newVal)
      },
      deep: true
    }
  },
  mounted() {
    this.$nextTick(() => {
      // this.setRulesData('idCardRules')
    })
  },
  methods: {
    // 设置验证规则
    setRulesData(type) {
      // console.log(this[type])
      this.rules = { ...this.baseRules, ...this[type] }
      // this.$refs.formRef && this.$refs.formRef.resetFields()
      console.log(this.rules)
    },
    // 关闭dialog
    handleClose() {
      // console.log('-handle-close-')
      // console.log('-type-', this.type)
      this.formData = this.initData
      this.onReset()
    },
    handleSelChange(data) {
      // 1 身份证 2 护照
      console.log(data)
      // let type = data == 1 ? 'idCardRules' : 'passportRules'
      // this.setRulesData(type)
    },
    onSubmit() {
      console.log('-type-', this.type, '-init-formdata-', this.initFormData, '-change-formdata-', this.formData)
      this.setEditAnchorParams()
      this.validateFunc(() => {
        this.submitReq()
      })
    },
    onReset() {
      // console.log('-reset-')
      this.$refs['formRef'].resetFields()
    },
    validateFunc(callback) {
      this.$refs['formRef'].validate(valid => {
        if (valid) {
          callback()
          return
        }
        console.log('error submit!!!')
        return false
      })
    },
    // 上传正面图片成功回调
    handleFrontImgSuccess(imgList) {
      this.$refs.formRef.validateField('photoPositive') // 单独验证图片上传
    },
    // 上传反面图片成功回调
    handleSideImgSuccess(imgList) {
      this.$refs.formRef.validateField('photoSide') // 单独验证图片上传
    },
    // 视频上传成功回调
    handleCertifyVideoSuccess(videoList) {
      this.$refs.formRef.validateField('videoUrl') // 单独验证视频上传
    },
    submitReq() {
      this.type === 'add' ? this.addAnchorReq() : this.editAnchorReq()
    },
    setSubmitPramas() {
      // console.log('-guildId-', this.guildId)
      let params = { guildId: this.guildId, ...this.formData }
      params.photoPositive = params.photoPositive[0]
      params.photoSide = params.photoSide[0]
      params.videoUrl = params.videoUrl[0]
      return params
    },
    addAnchorReq() {
      // console.log('-add-')
      let params = JSON.stringify(this.setSubmitPramas())
      // console.log('-params-', params)
      addAnchor(params).then(res => {
        // console.log('-add-res-', res)
        this.$emit('success', this.type)
        this.$message({
          message: '恭喜你，添加成功',
          type: 'success',
          duration: 2 * 1000,
          onClose: () => {
            this.dialogVisiable = false
          }
        })
      })
    },
    editAnchorReq() {
      // console.log('-edit-')
      // TODO 传递改变后的对象值
      let params = JSON.stringify(this.setEditAnchorParams())
      if (params === '{}') {
        this.$message({
          message: '您还没有修改信息呢！',
          type: 'warning'
        })
        return
      }
      // console.log('-params-', params)
      editAnchor(params).then(res => {
        // console.log('-edit-res-', res)
        this.$emit('success', this.type)
        this.$message({
          message: '恭喜你，重新提交审核成功！',
          type: 'success',
          duration: 2 * 1000,
          onClose: () => {
            this.dialogVisiable = false
          }
        })
      })
    },
    // 设置修改后需要传给后端的参数(只传递修改过的值，因此需要将初始对象和改变后的对象做比较)
    setEditAnchorParams() {
      let result = {}
      let midObj = { ...this.setSubmitPramas() }
      // console.log(this.initFormData, midObj)
      Object.keys(this.initFormData).map((key, index, array) => {
        // console.log(key, index, array)
        if (this.initFormData[key] !== midObj[key]) {
          result[key] = midObj[key]
        }
      })
      if (JSON.stringify(result) === '{}') {
        return result
      }
      if (result.type) {
        // type值存在的话需要将字符串格式的转化成数字格式
        result.type = +result.type
      }
      result.id = midObj.id
      return result
    }
  },
  components: { imgUpload, videoUpload }
}
</script>
<style lang="less" scoped>
//@import url(); 引入公共css类
.formWrap {
  .cerfiticateImgBox {
    display: flex;

    .frontImgBox {
      display: flex;
      align-items: flex-end;
      .txt {
        margin-left: 6px;
      }
    }
  }
}
/deep/ .el-dialog--center {
  .el-dialog__body {
    padding: 25px 90px 10px 30px;
  }
}
.el-form {
  /deep/ .el-form-item__label {
    padding-right: 12px;
  }
  /deep/ .el-input {
    width: 260px;
  }
}
</style>
