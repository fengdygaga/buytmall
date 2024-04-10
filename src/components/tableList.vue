<!--
 * @Description: Description
 * @version: 1.0.0
 * @Author: FJW
 * @Date: 2024-01-11 13:22:57
 * @LastEditors: FJW
 * @LastEditTime: 2024-01-22 13:56:23
-->
<template>
  <div class="box">
    <!-- 表头 -->
    <div class="flex-bet">
      <div style='display:flex'>
        <el-input placeholder="请输入店铺名称" prefix-icon="el-icon-search" v-model="searchData" style="margin-right:20px"></el-input>
        <el-button type="primary" @click.native="openAdd">新增记录</el-button>
        <el-button type="primary" @click.native="reflash">刷新</el-button>
      </div>
      <div>
        <el-date-picker v-model="selectMonth" type="month" placeholder="选择月" value-format='yyyy-MM'>
        </el-date-picker>
      </div>
    </div>

    <!-- 表格 -->
    <div style="margin-top:20px">
      <el-table :data='filerData(tableData)' border show-summary :summary-method='summaryMethod'>
        <!-- <el-table-column prop="id" label="id" align='center'></el-table-column> -->
        <el-table-column prop="date" label="日期" align='center'></el-table-column>
        <el-table-column prop="shopName" label="店铺名称" align='center'></el-table-column>
        <el-table-column prop="totalPrice" label="金额" align='center'></el-table-column>
        <el-table-column prop="remark" label="备注" align='center'></el-table-column>
        <el-table-column label="操作" align='center'>
          <template slot-scope="scope">
            <span class="cur del_btn" @click="openEdit(scope.row)">编辑</span>
            <span class="cur del_btn" style="margin-left:10px" @click="del(scope.row)">删除</span>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 日历 -->
    <div style="width:650px;border:1px solid #EBEEF5;margin-top:10px">
      <el-calendar v-model="calendarValue" :first-day-of-week='7' @change='changeMonth'>
        <template slot="dateCell" slot-scope="{data}">
          <div :class="data.isSelected ? 'is-selected' : ''">
            <p style="text-align:center">{{ data.day.split('-').slice(1).join('-') }} {{ data.isSelected ? '✔️' : ''}}</p>
            <p style="text-align:center;color:blue">{{getPriceForDay(data)}}</p>
          </div>
        </template>
      </el-calendar>
    </div>

    <!-- 弹框 -->
    <el-dialog :title="isEdit?'修改记录':'新增记录'" :visible.sync="dialogVisible" width="600px" v-if="dialogVisible" :close-on-click-modal='false'>
      <div>
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
          <el-form-item label="日期" prop="date">
            <el-date-picker type="date" value-format='yyyy-MM-dd' placeholder="选择日期" v-model="ruleForm.date"></el-date-picker>
          </el-form-item>
          <el-form-item label="店铺名称" prop="shopName">
            <el-select v-model="ruleForm.shopName" placeholder='请选择店铺名称'>
              <el-option value='天猫超市' label='天猫超市'></el-option>
              <el-option value='雨露空间' label='雨露空间'></el-option>
              <el-option value='其他' label='其他'></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="金额" prop="totalPrice">
            <el-input v-model="ruleForm.totalPrice" placeholder='请输入金额'></el-input>
          </el-form-item>
          <el-form-item label="备注">
            <el-input v-model="ruleForm.remark" placeholder="请输入备注"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm('ruleForm')">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
const setLocalStorage = (name, content) => {
  if (!name) return
  if (typeof content !== 'string') {
    content = JSON.stringify(content)
  }
  window.localStorage.setItem(name, content)
}

const getLocalStorage = name => {
  if (!name) return
  return JSON.parse(window.localStorage.getItem(name))
}

function computeNumber(a, type, b) {
  a === null ? a = '' : a
  b === null ? b = '' : b
  if (a === '' && b === '') {
    return ''
  }
  function getDecimalLength(n) {
    const decimal = n.toString().split('.')[1]
    return decimal ? decimal.length : 0
  }
  const amend = (n, precision = 15) => parseFloat(Number(n).toPrecision(precision))
  const power = Math.pow(10, Math.max(getDecimalLength(a), getDecimalLength(b)))
  let result = 0
  a = amend(a * power)
  b = amend(b * power)
  switch (type) {
    case '+':
      result = (a + b) / power
      break
    case '-':
      result = (a - b) / power
      break
    case '*':
      result = (a * b) / (power * power)
      break
    case '/':
      result = a / b
      break
  }
  result = amend(result)
  return result
}

export default {
  computed: {},
  components: {},
  props: {},
  data() {
    return {
      tableData: [],
      selectMonth: new Date(),
      dialogVisible: false,
      ruleForm: {
        date: new Date(),
        shopName: '天猫超市',
        totalPrice: '',
        remark: ''
      },
      rules: {
        date: [
          { required: true, message: '请选择日期', trigger: 'change' }
        ],
        shopName: [
          { required: true, message: '请选择店铺名称', trigger: 'change' }
        ],
        totalPrice: [
          { required: true, message: '请输入金额', trigger: 'blur' }
        ]
      },
      isEdit: false,
      id: '',
      searchData: '',
      calendarValue: new Date()
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    changeMonth(value){
      console.log(value)
    },

    getPriceForDay(data) {
      let resData = 0
      this.tableData.forEach(v => {
        if (data.day === v.date) {
          resData = v.totalPrice
        }
      })

      if (resData && data.type === 'current-month') {
        return `￥${resData}`
      }
    },

    reflash() {
      this.selectMonth = new Date()
    },

    openEdit(row) {
      this.ruleForm.date = row.date
      this.ruleForm.shopName = row.shopName
      this.ruleForm.totalPrice = row.totalPrice
      this.ruleForm.remark = row.remark
      this.isEdit = true
      this.id = row.id
      this.dialogVisible = true
    },

    openAdd() {
      this.isEdit = false
      this.dialogVisible = true
    },

    del(row) {
      const newArray = this.tableData.filter(obj => obj.id !== row.id)
      setLocalStorage('shopData', newArray)
      this.init()
    },

    summaryMethod(param) {
      const { data } = param
      const sums = []
      sums[0] = '合计'
      const tmpSum = data.reduce((prev, cur) => {
        return computeNumber(Number(prev), '+', Number(cur.totalPrice))
      }, 0)
      sums[2] = tmpSum + '元'
      return sums
    },

    checkDateFn(obj, data) {
      if (!data) {
        return true
      } else {
        return this.getDateMonth(obj.date) === this.getDateMonth(data)
      }
    },

    checkFn(obj, data) {
      if (!data) {
        return true
      } else {
        const tmpBool1 = obj.shopName && obj.shopName.indexOf(data) !== -1
        const tmpBool2 = obj.shopName && obj.shopName.indexOf(data.toUpperCase()) !== -1
        return tmpBool1 || tmpBool2
      }
    },

    filerData(data) {
      const filterData = data.filter(v => {
        return this.checkDateFn(v, this.selectMonth)
      }).filter(v => {
        return this.checkFn(v, this.searchData)
      })
      return filterData
    },

    getDateMonth(date) {
      return `${new Date(date).getFullYear()}-${(new Date(date).getMonth() + 1).toString().padStart(2, '0')}`
    },

    getDate(date) {
      return `${new Date(date).getFullYear()}-${(new Date(date).getMonth() + 1).toString().padStart(2, '0')}-${new Date(date).getDate().toString().padStart(2, '0')}`
    },

    init() {
      this.tableData = getLocalStorage('shopData') || []
      this.tableData.sort((a, b) => {
        return a.date >= b.date ? 1 : -1
      })
    },

    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          const tmpArr = getLocalStorage('shopData') || []
          if (this.isEdit) {
            tmpArr.forEach(v => {
              if (v.id === this.id) {
                v.date = this.ruleForm.date
                v.shopName = this.ruleForm.shopName
                v.totalPrice = this.ruleForm.totalPrice
                v.remark = this.ruleForm.remark
              }
            })
          } else {
            tmpArr.push({
              date: this.getDate(this.ruleForm.date),
              shopName: this.ruleForm.shopName,
              totalPrice: this.ruleForm.totalPrice,
              remark: this.ruleForm.remark,
              id: tmpArr.length > 0 ? tmpArr[tmpArr.length - 1].id + 1 : 1
            })
          }

          setLocalStorage('shopData', tmpArr)
          this.init()
          this.ruleForm.totalPrice = ''
          this.ruleForm.remark = ''
          this.dialogVisible = false
        }
      })
    }
  }
}

</script>
<style scoped>
.box {
  margin: 20px 50px 0;
}
.flex-bet {
  display: flex;
  justify-content: space-between;
}
.del_btn {
  color: #409eff;
  cursor: pointer;
}
.del_btn:hover {
  color: #66b1ff;
}
</style>