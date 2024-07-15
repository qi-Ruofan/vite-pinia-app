<template>
  <div class="apply-title">
    <el-button type="primary" @click="handleAdd">添加审批</el-button>
    <el-space>
      <el-input placeholder="请输入搜索关键词" v-model="searchWord"></el-input>
      <el-button type="primary" icon="search">搜索</el-button>
      <el-divider direction="vertical"></el-divider>
      <el-radio-group v-model="approverType">
        <el-radio-button label="全部" />
        <el-radio-button label="待审批" />
        <el-radio-button label="已通过" />
        <el-radio-button label="未通过" />
      </el-radio-group>
    </el-space>
  </div>
  <div class="apply-table">
    <el-table :data="pageApplyList" border>
      <el-table-column prop="applicantname" label="申请人" width="180" />
      <el-table-column prop="reason" label="审批事由" width="180" />
      <el-table-column prop="time" label="时间">
        <template #default="scope">
          {{ scope.row.time.join(' - ') }}
        </template>
      </el-table-column>
      <el-table-column prop="note" label="备注" />
      <el-table-column prop="approvername" label="审批人" width="180" />
      <el-table-column prop="state" label="状态" width="180" />
    </el-table>
    <el-pagination
      small
      background
      layout="prev, pager, next"
      :total="applyList.length"
      :page-size="pageSize"
      @current-change="handleChange"
    />
  </div>
  <el-dialog v-model="dialogVisible" title="添加审批" width="600" :before-close="handleClose">
    <el-form
      ref="ruleFormRef"
      :model="formData"
      label-width="auto"
      style="max-width: 600px"
      :rules="rules"
    >
      <el-form-item label="审批人" prop="approvername">
        <el-select v-model="formData.approvername" placeholder="请选择审批人">
          <el-option
            v-for="item in approver"
            :key="item._id"
            :label="item.name"
            :value="item.name"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="审批事由" prop="reason">
        <el-select v-model="formData.reason" placeholder="请选择审批事由">
          <el-option label="年假" value="年假" />
          <el-option label="事假" value="事假" />
          <el-option label="病假" value="病假" />
          <el-option label="外出" value="外出" />
          <el-option label="补签卡" value="补签卡" />
        </el-select>
      </el-form-item>
      <el-form-item label="时间" prop="time" required>
        <div class="block">
          <el-date-picker
            v-model="formData.time"
            type="datetimerange"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            format="YYYY-MM-DD HH:mm:ss"
            date-format="YYYY/MM/DD ddd"
            time-format="A hh:mm:ss"
          />
        </div>
      </el-form-item>
      <el-form-item label="备注" prop="note">
        <el-input v-model="formData.note" type="textarea" />
      </el-form-item>
    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="resetHandle(ruleFormRef)">重置</el-button>
        <el-button type="primary" @click="submitHandle(ruleFormRef)"> 提交 </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, computed, reactive } from 'vue'
import { useUsersStore } from '@/stores/users'
import { useChecksStore } from '@/stores/checks'
import { useNewsStore } from '@/stores/news'
import { storeToRefs } from 'pinia'
import type { PostApply as ApplyList } from '@/stores/checks'
import { type FormRules, type FormInstance, type DateModelType, ElMessage } from 'element-plus'
import moment from 'moment'

const usersStore = useUsersStore()
const checksStore = useChecksStore()
const newsStore = useNewsStore()
const { infos: usersInfos } = storeToRefs(usersStore)
const { applyList: checksApplyList } = storeToRefs(checksStore)

const defaultType = '全部'
const approverType = ref(defaultType)
const searchWord = ref('')
const dialogVisible = ref(false)
const ruleFormRef = ref<FormInstance>()

interface formDataType {
  applicantid: string
  applicantname: string
  approverid: string
  approvername: string
  note: string
  reason: string
  time: [DateModelType, DateModelType]
}
const formData = reactive<formDataType>({
  applicantid: '',
  applicantname: '',
  approverid: '',
  approvername: '',
  note: '',
  reason: '',
  time: ['', '']
})

const validatorTime = (
  rule: unknown,
  value: [DateModelType, DateModelType],
  callback: (arg?: Error) => void
) => {
  if (!value[0] && !value[1]) {
    callback(new Error('请添加审批'))
  } else {
    callback()
  }
}
const rules = reactive<FormRules<formDataType>>({
  approvername: [{ required: true, message: '请填写审批人姓名', trigger: 'blur' }],
  note: [{ required: true, message: '请填写审批人姓名', trigger: 'blur' }],
  reason: [{ required: true, message: '请填写审批人姓名', trigger: 'blur' }],
  time: [{ validator: validatorTime, trigger: 'blur' }]
})

const applyList = computed(() =>
  checksApplyList.value.filter(
    (item) =>
      (item.state === approverType.value || defaultType === approverType.value) &&
      (item.note as string).includes(searchWord.value)
  )
)
const pageSize = ref(5)
const pageCurren = ref(1)
const pageApplyList = computed(() =>
  applyList.value.slice((pageCurren.value - 1) * pageSize.value, pageCurren.value * pageSize.value)
)
const approver = computed(() => usersInfos.value.approver as { [index: string]: unknown }[])

const handleChange = (value: number) => {
  pageCurren.value = value
}

const handleClose = () => {
  dialogVisible.value = false
}
const handleAdd = () => {
  dialogVisible.value = true
}
const resetHandle = (formEl: FormInstance | undefined) => {
  if (!formEl) return
  formEl.resetFields()
}
const submitHandle = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  await formEl.validate((valid, fields) => {
    if (valid) {
      formData.applicantid = usersInfos.value._id as string
      formData.applicantname = usersInfos.value.name as string
      formData.approverid = (
        approver.value.find((v) => v.name === formData.approvername) as { [index: string]: unknown }
      )._id as string
      formData.time[0] = moment(formData.time[0]).format('YYYY-MM-DD hh:mm:ss')
      formData.time[1] = moment(formData.time[1]).format('YYYY-MM-DD hh:mm:ss')
      console.log(formData)
      checksStore.postApplyAction(formData).then((res) => {
        if (res.data.errcode === 0) {
          checksStore.getApplyAction({ applicantid: usersInfos.value._id }).then((res) => {
            if (res.data.errcode === 0) {
              checksStore.updateApplyList(res.data.rets)
            }
          })
          newsStore.putRemindAction({ userid: formData.approverid, approver: true })
          ElMessage.success('添加审批成功')
          resetHandle(ruleFormRef.value)
          dialogVisible.value = false
        }
      })
    } else {
      console.log('error submit!', fields)
    }
  })
}
</script>

<style scoped lang="scss">
.apply-title {
  margin: 20px;
  display: flex;
  justify-content: space-between;
}
.apply-table {
  margin: 10px;
}
.el-pagination {
  float: right;
  margin: 10px;
}
.el-select {
  width: 200px;
}
</style>
