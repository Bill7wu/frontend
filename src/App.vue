<template>
  <div style="padding: 240px;">
    <el-row :gutter="80">
      <el-col :span="12">
        <div class="grid-content ep-bg-purple"/>
        <div class='calculator'>
          <div class='showPanel'>
            <span class='expression'>{{ expression }}</span>
          </div>
          <div class='caculator-button'>

            <el-button @click="getResult('(')">(</el-button>
            <el-button @click="getResult(')')">)</el-button>
            <el-button @click="getResult('lg(')">lg</el-button>
            <el-button @click="getResult('ans')" type='primary' class="ans" :loading="historyLoading">ans</el-button>

            <el-button @click="getResult('c')">c</el-button>
            <el-button @click="getResult('del')">del</el-button>
            <el-button @click="getResult('/')">/</el-button>
            <el-button @click="getResult('*')">*</el-button>

            <el-button @click="getResult('7')">7</el-button>
            <el-button @click="getResult('8')">8</el-button>
            <el-button @click="getResult('9')">9</el-button>
            <el-button @click="getResult('-')">-</el-button>

            <el-button @click="getResult('4')">4</el-button>
            <el-button @click="getResult('5')">5</el-button>
            <el-button @click="getResult('6')">6</el-button>
            <el-button @click="getResult('+')">+</el-button>

            <el-button @click="getResult('1')">1</el-button>
            <el-button @click="getResult('2')">2</el-button>
            <el-button @click="getResult('3')">3</el-button>
            <el-button @click="getResult('%')">%</el-button>

            <el-button @click="getResult('0')" class="zero">0</el-button>
            <el-button @click="getResult('.')">.</el-button>
            <el-button @click="getResult('=')" type='danger' :loading="calcLoading">=</el-button>

          </div>
        </div>
      </el-col>
      <el-col :span="12">
        <div class="grid-content ep-bg-purple"/>
        <el-card class="box-card">
          <template #header>
            <div class="card-header">
              <span>历史记录(最近10条)</span>
            </div>
          </template>
          <el-table v-if="tableData?.length > 0" border :data="tableData" style="width: 100%">
            <el-table-column prop="expression" label="表达式"/>
            <el-table-column prop="result" label="结果"/>
          </el-table>
          <div v-else>空</div>
        </el-card>
      </el-col>
    </el-row>

  </div>
</template>

<script setup>
import {ref} from 'vue'
import axios from 'axios'
import {useAsyncState} from '@vueuse/core'
import {ElMessage} from 'element-plus'


const expression = ref('')

const {isLoading: historyLoading, state: tableData, execute: getHistory} = useAsyncState(
  () => axios.get('http://localhost:5000/history').then(res => res.data.result),
  [],
  {immediate: false}
)


const {isLoading: calcLoading, execute: calc} = useAsyncState(
  () => axios.post('http://localhost:5000/calculate', {
    expression: expression.value
  }).then(res => {
    if (res.data.errMsg) {
      ElMessage.error(res.data.errMsg)
    } else {
      expression.value = res.data.result
    }
  }),
  [],
  {immediate: false}
)

async function getResult(e) {
  switch (e) {
    case 'del':
      expression.value = expression.value.slice(0, expression.value.length - 1)
      break
    case 'ans':
      getHistory(0)
      break
    case '=':
      calc(0)
      // expression.value = eval(expression.value)
      break
    case 'c':
      expression.value = ''
      break
    default:
      expression.value += e
  }
}

</script>

<style lang='scss' scoped>
.calculator {
  border: solid 1px #dcdcdc;
  padding: 10px;
  background-color: #ffffff;
  border-radius: 4px;
  box-shadow: 0 0 2px #dcdcdc;
  width: 100%;

  .showPanel {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    padding: 2px 20px;
    height: 80px;
    border: 1px #dcdcdc solid;
    width: 100%;
    border-radius: 4px;
    box-sizing: border-box;
    margin-bottom: 8px;
    justify-content: space-evenly;

    .expression {
      font-size: 24px;
      font-weight: 700;
    }
  }
}

.el-button {
  margin: 0 !important;
  padding: 10px;
  font-weight: 600;
  height: 50px;
  width: 100%;
  font-size: 18px;
}

.caculator-button {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  gap: 8px;
  border-radius: 4px;
  background-color: #fffffff1;
}

.zero {
  grid-column: 1/3;
}
.ans {
  //grid-column: 3/5;
}
</style>
