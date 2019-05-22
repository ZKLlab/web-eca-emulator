<template>
  <div id="app">
    <table class="table-cell-status" style="margin-bottom: 20px">
      <tbody>
      <tr>
        <th><em>t</em> 时刻状态</th>
        <td v-for="(status, index) in tTickStatus" :key="index">
          <div v-for="(value, index) in status" class="cell big-cell" :value="value" :key="index"></div>
        </td>
      </tr>
      <tr>
        <th><em>t</em> + 1 时刻状态</th>
        <td class="t-1-tick-table-cell" v-for="(value, index) in t1TickStatus" :key="index"
          @click="changeStatus(7 - index)">
          <div class="cell big-cell" :value="value"></div>
        </td>
      </tr>
      </tbody>
    </table>
    <div style="margin-bottom: 40px">
      <p>预设</p>
      <p v-for="item in presets" :key="item.id" class="presets-row">
        <el-button
          v-for="preset in item.presets" :key="preset[0]"
          size="small"
          round
          :type="(ruleNo === preset[0] && initialCellRule === preset[1]) ? item.type : 'default'"
          @click="usePreset(preset)"
        >
          {{ preset[0] }}
        </el-button>
      </p>
      <p>初等元胞自动机编号</p>
      <el-input-number v-model="ruleNo" :min="0" :max="255" size="medium" />
      <p>初始元胞状态</p>
      <el-radio-group v-model="initialCellRule">
        <el-radio label="center">中心为
          <span class="cell big-cell" value="1"></span>，其余为
          <span class="cell big-cell" value="0"></span>
        </el-radio>
        <el-radio label="random">随机，预期密度为
          <el-input-number
            v-model="initialRandomDensity" :min="0" :max="1" :step="0.05" :precision="2" size="mini"
            :disabled="initialCellRule !== 'random'"
          />
        </el-radio>
      </el-radio-group>
      &nbsp;
      <el-button size="mini" @click="init()" :disabled="initialCellRule !== 'random'">刷新</el-button>
    </div>
    <div>
      <h1>{{ ruleNo }}号初等元胞自动机</h1>
      <div class="cell-wrapper">
        <div class="cell-row" v-for="(line, index) in ca" :key="index">
          <div v-for="(value, index) in line" class="cell small-cell" :value="value" :key="index"></div>
        </div>
      </div>
    </div>
    <p>&copy; ZKLlab</p>
  </div>
</template>

<script>
import qs from 'qs'

export default {
  name: 'app',
  data() {
    return {
      tTickStatus: [[1, 1, 1], [1, 1, 0], [1, 0, 1], [1, 0, 0], [0, 1, 1], [0, 1, 0], [0, 0, 1], [0, 0, 0]],
      ca: [],
      ruleNo: 90,
      length: 100,
      initialCellRule: 'center',
      initialRandomDensity: 0.25,
      presets: [
        {
          id: 0,
          type: 'primary',
          presets: [
            [28, 'center', null],
            [50, 'center', null],
            [54, 'center', null],
            [60, 'center', null],
            [90, 'center', null],
            [94, 'center', null],
            [102, 'center', null],
            [110, 'center', null],
            [150, 'center', null],
            [158, 'center', null],
            [188, 'center', null],
            [190, 'center', null],
            [220, 'center', null],
            [222, 'center', null],
          ]
        },
        {
          id: 1,
          type: 'warning',
          presets: [
            [0, 'random', 0.5],
            [32, 'random', 0.5],
            [160, 'random', 0.5],
            [250, 'random', 0.5],
          ]
        },
        {
          id: 2,
          type: 'warning',
          presets: [
            [4, 'random', 0.5],
            [108, 'random', 0.5],
            [218, 'random', 0.5],
          ]
        },
        {
          id: 3,
          type: 'warning',
          presets: [
            [22, 'random', 0.5],
            [30, 'random', 0.5],
            [126, 'random', 0.5],
            [150, 'random', 0.5],
            [182, 'random', 0.5],
          ]
        },
        {
          id: 4,
          type: 'warning',
          presets: [
            [110, 'random', 0.5],
          ]
        },
        {
          id: 5,
          type: 'danger',
          presets: [
            [184, 'random', () => {
              return Math.random()
            }],
          ]
        },
      ],
    }
  },
  watch: {
    ruleNo() {
      this.init()
    },
    initialCellRule() {
      this.init()
    },
    initialRandomDensity() {
      this.init()
    },
  },
  computed: {
    t1TickStatus() {
      let ans = Array(8)
      for (let i = 0; i < 8; i++) {
        ans[7 - i] = (this.ruleNo >> i) % 2
      }
      return ans
    },
  },
  created() {
    let options = qs.parse(location.search.substr(1))
    let no = parseFloat(options['n']), cr = options['c'], rds = options['r'], rdn = parseFloat(options['r'])
    if (no >= 0 && no <= 255 && Math.floor(no) === no) {
      this.ruleNo = no
    }
    if (cr === 'c') {
      this.initialCellRule = 'center'
    } else if (cr === 'r') {
      this.initialCellRule = 'random'
    }
    if (rds === 'rnd') {
      this.initialRandomDensity = Math.random()
    } else if (rdn >= 0 && rdn <= 1) {
      this.initialRandomDensity = rdn
    }
    this.init()
  },
  methods: {
    init() {
      this.ca.splice(0)
      let new_status = Array(this.length)
      new_status.fill(0)
      if (this.initialCellRule === 'center') {
        new_status[Math.floor((this.length - 1) / 2)] = 1
      } else if (this.initialCellRule === 'random') {
        for (let i = 0; i < this.length; i++) {
          new_status[i] = Math.random() < this.initialRandomDensity ? 1 : 0
        }
      }
      this.ca.push(new_status)
      for (let i = 0; i < this.length - 1; i++) {
        let new_status = Array(this.length)
        for (let i = 0; i < this.length; i++) {
          let last_line = this.ca[this.ca.length - 1]
          new_status[i] = this.t1TickStatus[7 - last_line[(i + this.length - 1) % this.length] * 4 - last_line[(i + this.length) % this.length] * 2 - last_line[(i + this.length + 1) % this.length] * 1]
        }
        this.ca.push(new_status)
      }
    },
    changeStatus(index) {
      let patch = (1 - (this.ruleNo >> index) % 2) << index
      this.ruleNo &= ~(1 << index)
      this.ruleNo += patch
    },
    usePreset(preset) {
      this.ruleNo = preset[0]
      this.initialCellRule = preset[1]
      if (typeof preset[2] === 'number') {
        this.initialRandomDensity = preset[2]
      } else if (typeof preset[2] === 'function') {
        this.initialRandomDensity = preset[2]()
      }
      this.init()
    },
  }
}
</script>

<style>
  body {
    margin: 0;
    padding: 0;
  }

  #app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    min-width: 640px;
    padding: 10px;
  }

  .presets-row {
    line-height: 36px;
  }

  .table-cell-status {
    border-collapse: collapse;
    width: 100%;
  }

  .table-cell-status th, .table-cell-status td {
    padding: 12px;
    border: 1px solid black;
    line-height: 14px;
  }

  .cell {
    box-sizing: border-box;
    display: inline-block;
  }

  .cell.big-cell {
    border: 1px solid black;
    width: 14px;
    height: 14px;
    margin-left: -1px;
  }

  .cell[value="0"] {
    background: white;
  }

  .cell[value="1"] {
    background: black;
  }

  .cell-wrapper {
    display: inline-block;
    margin: 0;
    padding: 0;
    outline: 1px solid black;
  }

  .cell-row {
    height: 4px;
    margin: 0;
    padding: 0;
  }

  .cell.small-cell {
    width: 4px;
    height: 4px;
    vertical-align: top;
  }

  .t-1-tick-table-cell {
    cursor: pointer;
  }

  .t-1-tick-table-cell:hover {
    background: rgba(0, 0, 0, 0.05);
  }
</style>
