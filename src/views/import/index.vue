<template>
  <div>
    <upload-excel :on-success="success" />
    <div class="import-button"><el-button type="primary" @click="exportData">导出</el-button></div>
  </div>
</template>

<script>
import { importEmployee } from '@/api/employees'
// import { formatDate } from '@/filters'

export default {
  data() {
    return {
      type: this.$route.query.type,
      importData: [],
      performanceMonth: 0
    }
  },
  methods: {
    async  success({ header, results }) {
      console.log('resultsSecond', results)
      const userRelations = {
        'Development Task Type( 开发任务类型 )': '任务类型',
        'Name': '任务名称',
        'TaskNumber(任务编号)': 'use case/task 对应号码',
        'Completed At': '约定完成时间',
        'Due Date': '实际完成时间',
        'Task Score(项目评分)': '主管评任务质量',
        'Created At': 'Created At'
      }
      var newArr = results.map(item => {
        var userInfo = {}
        Object.keys(item).forEach(key => {
          userInfo[userRelations[key]] = item[key]
        })
        return userInfo
      })
      console.log('test')
      console.log('formatArr', newArr)
      this.importData = newArr
      await importEmployee(newArr) // 接收一个数组
      this.$message.success('导入excel成功')
      this.$router.back() // 回到上一个页面
      // }
    },
    // 转化excel的日期格式
    formatDate(numb, format) {
      const time = new Date((numb - 1) * 24 * 3600000 + 1)
      time.setYear(time.getFullYear() - 70)
      const year = time.getFullYear() + ''
      const month = time.getMonth() + 1 + ''
      const date = time.getDate() - 1 + ''
      if (format && format.length === 1) {
        return year + format + month + format + date
      }
      return year + (month < 10 ? '0' + month : month) + (date < 10 ? '0' + date : date)
    },
    exportData() {
      this.performanceMonth = prompt('请输入绩效月份')
      const headers = {
        '任务类型': '任务类型',
        '编号': '编号',
        '任务名称': '任务名称',
        'use case/task 对应号码': 'use case/task 对应号码',
        '约定完成时间': '约定完成时间',
        '实际完成时间': '实际完成时间',
        '准时交付': '准时交付',
        '自评任务质量': '自评任务质量',
        '自评符合规范': '自评符合规范',
        '自评文档质量': '自评文档质量',
        '主管评任务质量': '主管评任务质量',
        '主管评符合规范': '主管评符合规范',
        '主管评文档质量': '主管评文档质量',
        '第三方': '第三方',
        '第三方评分': '第三方评分',
        '第三方Note': '第三方Note',
        '最终评分': '最终评分'
      }
      // 导出excel
      import('@/vendor/Export2Excel').then(async excel => {
        /**
         * 导出的时间，月份
         */
        const filterData = this.importData.filter((item) => {
          const time = new Date((item['实际完成时间'] - 1) * 24 * 3600000 + 1)
          const month = time.getMonth() + 1 + ''
          return month === this.performanceMonth
        })
        //  excel是引入文件的导出对象
        // 导出  header从哪里来
        // data从哪里来
        // 现在没有一个接口获取所有的数据
        // 获取员工的接口 页码 每页条数    100   1 10000
        // const { rows } = await getEmployeeList({ page: 1, size: this.page.total })
        console.log('rows', filterData)
        const data = this.formatJson(headers, filterData) // 返回的data就是 要导出的结构
        console.log('data', data)
        const multiHeader = [['任务类型', '任务名称', '', '', '', '', '主管评任务质量']]
        const merges = ['A1:A2', 'B1:F1', 'G1:G2'] // 合并单元格
        excel.export_json_to_excel({
          header: Object.keys(headers),
          data,
          filename: '任务完成绩效',
          multiHeader, // 复杂表头
          merges // 合并选项
        })
      })
    },
    // 将表头数据和数据进行对应
    // [{}]  =>   [[]]
    formatJson(headers, rows) {
      /**
       * 筛选用户所选中的数据
       */
      /**
       * 格式化数据
       */
      return rows.map(item => {
        // item是一个对象  { mobile: 132111,username: '张三'  }
        // ["手机号", "姓名", "入职日期" 。。]
        return Object.keys(headers).map(key => {
          // 需要判断 字段
          if (headers[key] === '约定完成时间' || headers[key] === '实际完成时间') {
            // 格式化日期
            return this.formatDate(item[headers[key]])
          }
          return item[headers[key]]
        })
      })
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.import-button{
  display:flex;
  justify-content: center;
  margin-top: 10rem;
}
</style>
