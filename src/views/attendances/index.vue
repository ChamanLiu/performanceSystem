<template>
  <div v-loading="loading" class="dashboard-container">
    <div class="app-container">
      <div>12122</div>
      <AttendanceSet />
    </div>
  </div>
</template>

<script>
import attendanceApi from '@/api/constant/attendance'
import { getAttendancesList, updateAttendance } from '@/api/attendances'
import AttendanceSet from './components/attendance-set'
import { getDepartments } from '@/api/departments'
export default {
  name: 'Attendances',
  components: { AttendanceSet },
  data() {
    return {
      list: [],
      selectData: [],
      stateData: attendanceApi,
      departments: [],
      total: 100,
      attendanceRecord: '',
      monthOfReport: '',
      centerDialogVisible: false,
      tipsDialogVisible: false,
      month: '',
      yearMonth: '',
      loading: false,
      attendInfo: {
        month: '',
        getDate: '',
        getInfo: '',
        name: '',
        counts: '',
        tobeTaskCount: ''
      },
      formData: {
        page: 1,
        pagesize: 10,
        keyword: this.keyword,
        deptID: [], // 性别
        stateID: ''
      },
      page: {
        page: 1,
        pagesize: 10,
        total: 0
      },
      modifyData: {
        userId: '',
        day: '',
        adtStatu: ''
      }
    }
  },
  // 创建完毕状态
  created() {
    this.getAttendancesList() // 获取考勤列表
    this.getDepartments() // 获取考勤列表
  },
  methods: {
    // 暂时不处理
    handleSub() {
      this.tipsDialogVisible = false
      this.$message.success('提醒成功')
    },
    handleTip() {
      this.tipsDialogVisible = true
    },
    // 设置
    handleSet() {
      this.$refs.set.dialogFormV()
    },
    // 弹框关闭
    handleCloseModal() {
      this.$refs.set.dialogFormH()
    },
    // 获取组织列表
    async getDepartments() {
      const { depts } = await getDepartments()
      this.departments = depts
    },

    // 初始化数据
    async getAttendancesList() {
      this.loading = true
      const { data, monthOfReport, tobeTaskCount } = await getAttendancesList({ ...this.page })
      this.list = data.rows // 当前记录
      this.page.total = data.total // 总条数
      this.attendInfo.counts = data.total
      this.attendInfo.month = monthOfReport
      this.attendInfo.tobeTaskCount = tobeTaskCount

      var date = new Date()
      var year = date.getFullYear()
      const month = monthOfReport
      var d = new Date(year, month, 0) // 获取月份
      this.monthOfReport = d.getDate() // 获取日期
      this.yearMonth = year + ('' + month < 10 ? '0' + month : month)
      this.month = monthOfReport
      this.loading = false
    },
    // 确定修改
    async  btnOK() {
      await updateAttendance(this.modifyData)
      this.centerDialogVisible = false
      this.getAttendancesList() // 成功之后 重新拉取数据
    },
    // 页码改变
    pageChange(page) {
      this.page.page = page
      this.getAttendancesList() // 获取数据
    },
    showChangeDialog(item, id, it) {
      this.modifyData.userId = item.id
      this.modifyData.day = it.day
      this.modifyData.departmentId = item.departmentId
      this.modifyData.adtStatu = it.adtStatu + '' // 数字转成字符串

      if (it.adtStatu !== '') {
        this.attendInfo.getDate = parseInt(id + 1)
        this.attendInfo.getInfo = it.adtStatu
        this.attendInfo.name = item.name
        this.centerDialogVisible = true
      }
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .tableInfo {
    line-height: 36px;
    border: solid 1px #ebeef5;
    border-right: 0 none;
    border-bottom: 0 none;
    tr {
      th {
        height: 50px;
        text-align: center;
        border-right: solid 1px #ebeef5;
        border-bottom: solid 1px #ebeef5;
        border-bottom: 2px solid #e8e8e8;
        background: #fafafa;
        min-width:  100px;
      }
      td {
        height: 36px;
        text-align: center;
        border-right: solid 1px #ebeef5;
        border-bottom: solid 1px #ebeef5;
      }
    }
  }

.attenInfo {
  p {
    &.check {
      padding: 20px 0 0;
    }
    .el-radio {
        display: inline-block;
        width: 60px;
        padding: 5px 0;

    }
  }
}
</style>
