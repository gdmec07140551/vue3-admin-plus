<template>
  <div class="scroll-y">
    <!--操作-->
    <div class="mr-3 rowSS">
      <el-button type="primary" @click="addBtnClick">
        <el-icon style="vertical-align: middle">
          <FolderAdd />
        </el-icon>
        <span style="vertical-align: middle">增加</span>
      </el-button>
      <el-button type="primary" @click="multiDelBtnClick">
        <el-icon style="vertical-align: middle">
          <Delete />
        </el-icon>
        <span style="vertical-align: middle">删除</span>
      </el-button>
      <!--条件搜索-->
      <el-form ref="refsearchFormMixin" :inline="true" class="demo-searchFormMixin ml-2">
        <el-form-item label-width="0px" label="" prop="username" label-position="left">
          <!--  --c -->
          <el-input v-model="searchFormMixin.name" class="widthPx-150" placeholder="品牌名字" />
        </el-form-item>
        <el-form-item label-width="0px" label="" prop="createTime" label-position="left">
          <el-date-picker
            v-model="startEndArrMixin"
            type="datetimerange"
            format="YYYY-MM-DD"
            value-format="YYYY-MM-DD HH:mm:ss"
            class="widthPx-250"
            range-separator="-"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            @change="dateTimePacking"
          />
        </el-form-item>
      </el-form>
      <!--查询按钮-->
      <el-button type="primary" @click="searchBtnClick">查询</el-button>
    </div>
    <!--表格和分页-->
    <el-table
      id="resetElementDialog"
      ref="refuserTable"
      :height="`calc(100vh - ${settings.delWindowHeight})`"
      size="mini"
      border
      :data="usertableData"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" align="center" width="50" />
      <el-table-column align="center" prop="name" label="品牌名称" min-width="100" />
      <el-table-column align="center" prop="image" label="品牌图片地址" min-width="100">
        <template #default="{ row }">
          <img :src="row.image" class="widthPx-120 heightPx-120" style="border-radius: 10px" />
        </template>
      </el-table-column>
      <el-table-column align="center" prop="letter" label="首字母" width="80" />
      <el-table-column align="center" prop="seq" label="排序" width="80" />
      <el-table-column align="center" prop="createTime" label="创建时间" width="140" />
      <el-table-column align="center" prop="updateTime" label="更新时间" width="140" />
      <!--点击操作-->
      <el-table-column fixed="right" align="center" label="操作" width="180">
        <template #default="{ row }">
          <el-button type="text" size="small" @click="tableEditClick(row)">编辑</el-button>
          <el-button type="text" size="small" @click="tableDetailClick(row)">详情</el-button>
          <el-button type="text" size="small" @click="tableDelClick(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--分页-->
    <div class="block columnCC mt-2">
      <el-pagination
        :current-page="pageNum"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="pageTotalMixin"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
    <!--详情-->
    <el-dialog v-model="detailDialogMixin" :title="dialogTitleMixin" width="500px" :close-on-click-modal="false">
      <div class="detail-container rowBC">
        <div class="detail-container-item">品牌名称：{{ detailData.name }}</div>
      </div>
      <div class="detail-container rowBC">
        <div class="detail-container-item">品牌首字母：{{ detailData.letter }}</div>
      </div>
      <div class="detail-container rowBC">
        <div class="detail-container-item">排序：{{ detailData.seq }}</div>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button type="primary" @click="detailDialogMixin = false">确 定</el-button>
        </span>
      </template>
    </el-dialog>
    <CRUDForm v-if="showFrom" ref="refCRUDForm" @hideComp="hideComp" @selectPageReq="selectPageReq" />
  </div>
</template>
<script>
export default {
  name: 'Brand'
}
</script>
<script setup>
import { Delete, FolderAdd } from '@element-plus/icons-vue'
/*1.初始化引入和实例化*/
import settings from '@/settings'
import CRUDForm from './CRUDForm.vue'
import { onMounted, getCurrentInstance, ref, reactive, onActivated, onDeactivated } from 'vue'
let { proxy } = getCurrentInstance()
const refCRUDForm = ref(null)
onActivated(() => {
  console.log('onActivated')
})
onDeactivated(() => {
  console.log('onDeactivated')
})
/*2.表格操作和查询*/
let multipleSelection = ref([])
const handleSelectionChange = (val) => {
  multipleSelection.value = val
}
let usertableData = ref([])
let searchFormMixin = reactive({
  id: '',
  name: '',
  image: '',
  letter: '',
  seq: ''
})
onMounted(() => {
  selectPageReq()
})
let selectPageReq = () => {
  const data = Object.assign(searchFormMixin, {
    pageNum: pageNum.value,
    pageSize: pageSize.value
  })
  Object.keys(data).forEach((fItem) => {
    if (data[fItem] === '' || data[fItem] === null || data[fItem] === undefined) delete data[fItem]
  })
  let reqConfig = {
    url: '/ty-user/brand/selectPage',
    method: 'get',
    data,
    isParams: true
  }
  proxy.$axiosReq(reqConfig).then((resData) => {
    usertableData.value = resData.data?.records
    proxy.pageTotalMixin = resData.data?.total
  })
}
import tablePageHook from '@/hooks/useTablePage'
let { pageNum, pageSize, handleCurrentChange, handleSizeChange } = tablePageHook(selectPageReq)
const dateTimePacking = (timeArr) => {
  if (timeArr && timeArr.length === 2) {
    searchFormMixin.startTime = timeArr[0]
    searchFormMixin.endTime = timeArr[1]
  } else {
    searchFormMixin.startTime = ''
    searchFormMixin.endTime = ''
  }
}
const searchBtnClick = () => {
  //此处要重置页数，也是常出的bug
  pageNum.value = 1
  selectPageReq()
}
//删除相关
const refuserTable = ref(null)
const multiDelBtnClick = () => {
  let rowDeleteIdArrMixin = []
  // let selectionArr = proxy.$refs.refuserTable //--c
  let deleteNameTitle = ''
  rowDeleteIdArrMixin = multipleSelection.value.map((mItem) => {
    deleteNameTitle = deleteNameTitle + mItem.name + ','
    return mItem.id
  })
  if (rowDeleteIdArrMixin.length === 0) {
    proxy.elMessageMixin('表格选项不能为空', 'warning')
    return
  }
  let stringLength = deleteNameTitle.length - 1
  proxy
    .elConfirmMixin('删除', `您确定要删除【${deleteNameTitle.slice(0, stringLength)}】吗`)
    .then(() => {
      const data = rowDeleteIdArrMixin
      proxy
        .$axiosReq({
          url: `/ty-user/brand/deleteBatchIds`,
          data,
          method: 'DELETE',
          bfLoading: true
        })
        .then((res) => {
          proxy.elMessageMixin('删除成功')
          selectPageReq()
        })
    })
    .catch(() => {})
}
let deleteByIdReq = (id) => {
  return proxy.$axiosReq({
    url: '/ty-user/brand/deleteById',
    data: { id: id },
    isParams: true,
    method: 'delete',
    bfLoading: true
  })
}
let tableDelClick = (row) => {
  proxy
    .elConfirmMixin('确定', `您确定要删除【${row.name}】吗？`)
    .then(() => {
      deleteByIdReq(row.id).then(() => {
        selectPageReq()
        proxy.elMessageMixin(`【${row.name}】删除成功`)
      })
    })
    .catch(() => {})
}
//添加和修改
let showFrom = ref(false)
let addBtnClick = () => {
  showFrom.value = true
  proxy.$nextTick(() => {
    refCRUDForm.value.showModal()
  })
}
onMounted(() => {
  console.log('import', import.meta.env.VITE_APP_IMAGE_URL)
})
const hideComp = () => {
  showFrom.value = false
}
let tableEditClick = (row) => {
  getDetailByIdReq(row.id).then((resData) => {
    showFrom.value = true
    proxy.$nextTick(() => {
      refCRUDForm.value.showModal(true, resData.data)
    })
  })
}
/*3.详情modal*/
let detailData = ref({})
let tableDetailClick = (row) => {
  proxy.dialogTitleMixin = `详情【${row.name}】`
  getDetailByIdReq(row.id).then((resData) => {
    detailData.value = resData.data
    proxy.detailDialogMixin = true
  })
}
let getDetailByIdReq = (id) => {
  return proxy.$axiosReq({
    url: '/ty-user/brand/selectById',
    data: { id }, //--c
    isParams: true,
    method: 'get'
  })
}

/*4.文件上传和时间选择*/
// let downloadReq = (row) => {
//   let reqConfig = {
//     url: '/ty-user/brand/download',
//     method: 'get',
//     data: {
//       id: row.id
//     },
//     isDownLoadFile: true,
//     isParams: true,
//     isAlertErrorMsg: false
//   }
//   proxy.$axiosReq(reqConfig).then((resData) => {
//     const url = window.URL.createObjectURL(new Blob([resData]))
//     const link = document.createElement('a')
//     link.href = url
//     link.setAttribute('download', `${row.username}.xls`)
//     document.body.appendChild(link)
//     link.click()
//   })
// }
// let tableDownloadClick = (row) => {
//   downloadReq(row)
// }
</script>

<style scoped lang="scss">
/*详情*/
.detail-container {
  flex-wrap: wrap;
}

.detail-container-item {
  min-width: 40%;
  margin-bottom: 20px;
}

.detailDialog-title {
  margin-bottom: 14px;
  font-weight: bold;
  font-size: 16px;
}
</style>
