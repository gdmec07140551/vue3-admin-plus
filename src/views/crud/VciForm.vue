<template>
  <el-dialog
    v-model="dialogVisibleMixin"
    :title="dialogTitleMixin"
    width="600px"
    :close-on-click-modal="false"
    :before-close="closeFormModal"
  >
    <el-form ref="refForm" label-width="150px" :inline="false" :model="subForm" :rules="formRulesMixin" class="pr-5">
      <el-form-item label="设备号" prop="sn" :rules="formRulesMixin.isNotNull" label-position="left">
        <el-input v-model="subForm.sn" class="widthPx-150" placeholder="设备号" />
      </el-form-item>
      <el-form-item label="硬件版本" prop="hardVersion" :rules="formRulesMixin.isNotNull" label-position="left">
        <el-input v-model="subForm.hardVersion" class="widthPx-150" placeholder="硬件版本" />
      </el-form-item>
      <!--      <el-form-item label="软件(固件)版本" prop="softVersion" :rules="formRulesMixin.isNotNull" label-position="left">-->
      <!--        <el-input v-model="subForm.softVersion" class="widthPx-150" placeholder="软件(固件)版本" />-->
      <!--      </el-form-item>-->
      <el-form-item label="状态：" prop="status" :rules="formRulesMixin.isNotNull" label-position="left">
        <el-select v-model="subForm.status" clearable placeholder="请选择" class="widthPx-120">
          <el-option label="未出库" :value="0" />
          <el-option label="已出库" :value="1" />
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="closeFormModal">取 消</el-button>
        <el-button type="primary" @click="confirmBtnClick">确 定</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
/*1.初始化引入和实例化*/
import { getCurrentInstance, watch, ref, toRefs, reactive, computed } from 'vue'

let { proxy } = getCurrentInstance()
const emit = defineEmits(['selectPageReq', 'hideComp'])

/*2.modal新增和修改*/
//新增
const refForm = ref(null)
let subForm = reactive({
  id: '',
  sn: '',
  hardVersion: '',
  softVersion: '',
  createTime: '',
  status: '',
  remark: ''
})
let confirmBtnClick = () => {
  refForm.value.validate((valid) => {
    if (valid) {
      if (subForm.id) {
        updateReq()
      } else {
        insertReq()
      }
    } else {
      return false
    }
  })
}
const insertReq = () => {
  const data = JSON.parse(JSON.stringify(subForm))
  delete data.id
  proxy
    .$axiosReq({
      url: '/ty-user/vci/insert',
      data: data,
      method: 'post',
      bfLoading: true
    })
    .then((res) => {
      proxy.elMessageMixin('保存成功')
      emit('selectPageReq')
      emit('hideComp')
    })
}
//修改
const reshowData = (row) => {
  Object.keys(row).forEach((fItem) => {
    Object.keys(subForm).forEach((sItem) => {
      if (fItem === sItem) {
        subForm[sItem] = row[sItem]
      }
    })
  })
}
let updateReq = () => {
  return proxy
    .$axiosReq({
      url: '/ty-user/vci/updateById',
      data: subForm,
      method: 'put',
      bfLoading: true
    })
    .then(() => {
      proxy.elMessageMixin('更新成功')
      emit('selectPageReq')
      emit('hideComp')
    })
}

/*3.弹框相关*/
let showModal = (isEdit, detailData) => {
  if (isEdit) {
    proxy.dialogTitleMixin = `编辑【${detailData.sn}】`
    proxy.dialogVisibleMixin = true
    reshowData(detailData)
  } else {
    proxy.dialogTitleMixin = '添加【VCI设备表】'
    proxy.dialogVisibleMixin = true
  }
}

//关闭弹框
let closeFormModal = () => {
  emit('hideComp')
}
/*4.文件上传和时间选择*/
const tableImportReq = () => {
  const formData = new FormData()
  formData.append('file', proxy.$refs.refSettingFile.files[0])
  proxy
    .$axiosReq({
      url: '/ty-user/ty-upload/uploadFile',
      data: formData,
      method: 'post',
      bfLoading: true,
      isUploadFile: true
    })
    .then((resData) => {
      let { ori, shot, name } = resData.data
      proxy.$refs.refSettingFile.value = ''
    })
    .catch((err) => {
      proxy.$refs.refSettingFile.value = ''
    })
}

const fileUploadSave = () => {
  const formData = new FormData()
  formData.append('file', proxy.$refs.refSettingFile.files[0])
  proxy
    .$axiosReq({
      url: '/ty-user/ty-upload/upload/image',
      data: formData,
      method: 'post',
      bfLoading: true,
      isUploadFile: true
    })
    .then((resData) => {
      let { shortPath } = resData.data
      proxy.chooseFileNameMixin = shortPath
      // 存储文件名称
      const filename = proxy.$refs.refSettingFile.value
      proxy.chooseFileNameMixin = filename.slice(filename.lastIndexOf('\\') + 1)

      subForm.image = import.meta.env.VITE_APP_IMAGE_URL + shortPath
      proxy.$refs.refSettingFile.value = ''
    })
    .catch(() => {
      proxy.chooseFileNameMixin = ''
      subForm.image = ''
      proxy.$refs.refSettingFile.value = ''
    })
}
//导出内部组件属性
defineExpose({
  showModal
})
//导出属性到页面中使用
// let {levelList} = toRefs(state);
</script>

<style scoped lang="scss"></style>
