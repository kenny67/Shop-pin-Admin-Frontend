<template>
  <div class="time-container">
    <el-alert
      title="可自由设置任意个每日收团的时间点 系统会自动匹配最近的时间点作为拼团截止时间"
      type="warning"
      close-text="知道了"
    />
    <el-col :span="16">

      <el-table
        v-loading="repayArgLoading"
        :data="timePointList"
        stripe
        style="width: 80%"
        @selection-change="handleSelectionChange"
      >>
        <el-table-column
          type="selection"
          width="55"
        />
        <el-table-column label="ID" prop="id" align="center" min-width="80">
          <template slot-scope="scope">
            <span>时间点{{ scope.row.number }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="时间点"
          min-width="100"
          align="center"
        >
          <template slot-scope="scope">
            <i class="el-icon-time" />
            <span style="margin-left: 10px">{{ millisecondsToTimeString(scope.row.time) }}</span>
          </template>
        </el-table-column>
      </el-table>
    </el-col>
    <el-col :span="8">

      <el-form label-position="left">
        <div style="margin-top: 10px">
          <el-button v-if="addingNewTime==false" @click="addingNewTime=true">添加</el-button>
          <el-button v-if="addingNewTime==true" type="success" @click="handleAdd">保存</el-button>
          <el-button v-if="addingNewTime==true" type="danger" @click="addingNewTime=false">取消</el-button>
          <el-button v-if="addingNewTime==false" @click="handleDelete">删除</el-button>
        </div>
        <el-form-item v-if="addingNewTime==true" label="时间点">
          <el-time-picker
            v-model="newTime"
            :picker-options="{
              selectableRange: '00:00:00 - 23:59:59'
            }"
            placeholder="请选择"
          />
        </el-form-item>
      </el-form>
    </el-col>
  </div>
</template>

<script>
import { getTimePoint, addTimePoint, deleteTimePoint } from '../../api/store'

export default {
  name: 'TimeSet',
  data() {
    return {
      timePointList: [],
      repayArgLoading: false,
      multiTimeSet: [],
      addingNewTime: false,
      newTime: '',
      temp: ''
    }
  },

  mounted() {
    this.getList()
    this.newTime = new Date()
  },
  methods: {
    async getList() {
      this.repayArgLoading = true
      await new Promise((resolve, reject) => {
        getTimePoint().then(response => {
          this.repayArgLoading = false
          this.timePointList = response.data.list
          for (let i = 0; i <= this.timePointList.length; i++) {
            this.timePointList[i].number = i + 1
          }
        }).catch(error => {
          reject(error)
        })
      })
    },
    handleSelectionChange(val) {
      this.multiTimeSet = val
    },
    async handleAdd() {
      this.addingNewTime = false
      this.temp = this.timeStringToMilliseconds(this.newTime.toString().split(' ')[4])
      await new Promise((resolve, reject) => {
        addTimePoint(this.temp).then(response => {
          this.$message({
            message: '添加成功',
            type: 'success'
          })
          this.getList()
        }).catch(error => {
          reject(error)
        })
      })
      this.newTime = ''
      this.temp = ''
    },
    async handleDelete() {
      if (this.multiTimeSet.length === 0) {
        this.$message({
          message: '请选择要删除的时间点',
          type: 'warning'
        })
        return
      }
      this.$confirm('确定要删除' + this.multiTimeSet.length + '个时间点吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        await new Promise((resolve, reject) => {
          deleteTimePoint({ closeBatchList: this.multiTimeSet
          }).then(response => {
            this.$message({
              message: '删除成功',
              type: 'success'
            })
            this.getList()
          }).catch(error => {
            reject(error)
          })
        })
      })
    },
    millisecondsToTimeString(milliseconds) {
      milliseconds += 8 * 3600000 // 增加 timestamp offset
      const seconds = Math.floor(milliseconds / 1000)
      let hours = String(Math.floor(seconds / 3600))
      let minutes = String(Math.floor((seconds % 3600) / 60))
      if (hours.length < 2) {
        hours = '0' + hours
      }
      if (minutes.length < 2) {
        minutes = '0' + minutes
      }
      return String(hours + ':' + minutes)
    },
    timeStringToMilliseconds(timeString) {
      const hoursAndMinutes = timeString.split(':')
      const hours = Number(hoursAndMinutes[0])
      const minutes = Number(hoursAndMinutes[1])
      return (1000 * (minutes * 60 + hours * 3600)) - 8 * 3600000 // 减少 timestamp offset
    }
  }
}
</script>

<style scoped>
  .time-container {
    /*background-color: #f0f2f5;*/
    padding: 0px 30px;
    min-height: calc(100vh - 84px);
  }
</style>
