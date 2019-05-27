<template>
  <div class="home">
    <h1>LL分析法</h1>
    <h5>作者：邱冠霖1607052230</h5>
    <!-- 终结符与非终结符 -->
    <el-row>
      <el-button type="warning"
                 @click="restart">初始化</el-button>
      <el-button type="success"
                 @click="getDemo">DEMO</el-button>
    </el-row>
    <el-row :gutter="20">
      <el-col :xs="24" :sm="12">
        <el-card>
          <div slot="header">
            终结符
          </div>
          <el-input v-model="inputEnd"
                    placeholder="请输入终结符"></el-input>

          <el-tag :key="tag"
                  v-for="tag in end"
                  :disable-transitions="true"
                  style="margin-top:10px">
            {{tag}}
          </el-tag>
        </el-card>
      </el-col>
      <el-col :xs="24" :sm="12">
        <el-card>
          <div slot="header">
            非终结符
          </div>
          <el-input v-model="inputNotEnd"
                    placeholder="请输入非终结符"></el-input>
          <el-tag :key="tag"
                  v-for="tag in notEnd"
                  :disable-transitions="true"
                  style="margin-top:10px">
            {{tag}}
          </el-tag>
        </el-card>
      </el-col>

    </el-row>
    <el-row>
      <el-button type="primary"
                 @click="handleInputEnd">提交终结符</el-button>
    </el-row>
    <!-- 产生式 -->
    <el-row :gutter="20">
      <el-card>
        <div slot="header">
          产生式（空字用‘@’表示)
        </div>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-input type="textarea"
                      :rows="10"
                      placeholder="输入所有产生式（空字用‘@’表示)"
                      v-model="production">
            </el-input>
          </el-col>
          <el-col :span="12">
            <el-tag :key="pro"
                    v-for="pro in proceList"
                    :disable-transitions="true"
                    style="margin-top:10px">
              {{pro}}
            </el-tag>
          </el-col>
        </el-row>

      </el-card>
    </el-row>
    <el-row>
      <el-button type="primary"
                 @click="handlePro">提交产生式</el-button>
    </el-row>
    <!-- first,follow -->
    <el-row :gutter="20">
      <el-col :xs="24" :sm="12">
        <el-card style="text-align: left;">
          <div slot="header">
            <span>FIRST集</span>
          </div>
          <p v-for="i in notEnd"
             :key="i"
             style="margin: 0;">
            FIRST( {{i[0]}} ) :
            <!-- {{ first[i[0]] }} -->
            <el-tag :key="tag"
                    v-for="tag in first[i[0]]"
                    :disable-transitions="true"
                    style="margin-top:5px">{{tag}}</el-tag>
            <el-divider></el-divider>
          </p>
        </el-card>
      </el-col>
      <el-col :xs="24" :sm="12">
        <el-card style="text-align: left;">
          <div slot="header">
            <span>FOLLOW集</span>
          </div>
          <p v-for="i in notEnd"
             :key="i"
             style="margin: 0;">
            FOLLOW( {{i[0]}} ) :
            <!-- {{follow[i[0]]}} -->
            <el-tag :key="tag"
                    v-for="tag in follow[i[0]]"
                    :disable-transitions="true"
                    style="margin-top:5px">{{tag}}</el-tag>
            <el-divider></el-divider>
            </p>
        </el-card>
      </el-col>
    </el-row>
    <el-row>
      <el-card>
        <el-table :data="selectTableData"
                  size="small">
          <!-- <el-table-column prop="proce"
                       label="产生式" /> -->
          <el-table-column prop="proce"
                           label="选择集">
            <template slot-scope="scope">
              <span>
                {{scope.row.proce.split('')[0]}} ->
                <span :key="s"
                      v-for="s in scope.row.proce.split('').slice(1)">
                  {{s}}
                </span>
              </span>
            </template>
          </el-table-column>
          <el-table-column prop="selectList"
                           label="选择集">
            <template slot-scope="scope">
              <el-tag v-for="s in scope.row.selectList"
                      :key="s">{{s}}</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </el-card>
    </el-row>
    <el-row>
      <el-card>
        <div slot="header">
          预测分析表
        </div>
        <el-table :data="predictAnalyzeTableData"
                  border>
          <el-table-column prop="name" />
          <el-table-column v-for="i in predictAnalyzeTableColumn"
                           :key="i"
                           :prop=i
                           :label=i />

        </el-table>
      </el-card>
    </el-row>
    <el-row>
      <el-card>
        <div slot="header">
          输入要分析的句子
        </div>
        <el-input v-model="inputSentence"
                  placeholder="请输入句子" />

        <el-button type="primary"
                   @click="handleAnalyze"
                   style="margin-top:10px">确定</el-button>
      </el-card>
    </el-row>

    <el-row>
      <el-card>
        <div slot="header">
          分析过程
        </div>
        <el-table :data="analyzeInfo">
          <el-table-column prop="stackCur"
                           label="栈"
                           align="left" />
          <el-table-column prop="sentenceCur"
                           label="输入"
                           align="right" />
          <el-table-column prop="msg"
                           label="动作" />
        </el-table>
      </el-card>
    </el-row>
  </div>
</template>

<script>
export default {
  name: 'home',
  data () {
    return {
      inputEnd: '+ * ( ) a',
      inputNotEnd: 'E T F A B',
      production: 'E->TA\nA->+TA|@\nT->FB\nB->*FB|@\nF->(E)|a', // 产生式
      inputSentence: 'a*a+a$', // 输入句子
      proceList: [], // 产生式的列表
      // 以下是处理
      end: [], // 终结符
      notEnd: [], // 终结符
      predictAnalyzeTable: [], // 预测分析表
      proce: [], // 预测表数组
      first: [], // first集
      follow: [], // follow集
      selectTableData: [], // 选择集 表格数据
      predictAnalyzeTableData: [], // 预测分析表
      predictAnalyzeTableColumn: [], // 预测分析表的列
      analyzeInfo: [], // 预测分析表的动作表
    }
  },
  created () {
  },
  methods: {
    handleInputEnd () { // 处理终结符和非终结符
      // 处理终结符
      let arr = []
      arr.push('$')
      // 拆封然后依次赋值
      arr.push(...this.inputEnd.split(' '))
      arr.push('@')
      this.end = arr
      // 处理非终结符
      arr = []
      // 拆封然后依次赋值
      arr.push(...this.inputNotEnd.split(' '))
      this.notEnd = arr
      // 完整性验证
      this.end = this.end.filter(i => {
        if (i.length > 1) {
          this.$message.warning('终结符：' + i + ' 错误，原因：只支持单个字符')
          return false
        } else {
          return true
        }
      })
      this.notEnd = this.notEnd.filter(i => {
        if (i.length > 1) {
          this.$message.warning('非终结符：' + i + ' 错误，原因：只支持单个字符')
          return false
        } else {
          return true
        }
      })

      // 把终结符和非终结符存到first和follow的属性中
      this.first = {} // 先设置为空
      this.follow = {} // 先设置为空
      arr.map(i => {
        this.first[i] = []
        this.follow[i] = []
      })
      console.log('终结符end', this.end)
      console.log('非终结符notEnd', this.notEnd)
    },
    handlePro () { // 提交产生式
      let proce = []
      let list = this.production.split('\n') // 按行分成数组
      list.map(one => { // 依次遍历每个数组
        let s = ''
        s += one[0]
        for (let j = 3; j < one.length; j++) { // 从3开始可以跳过-》的字符
          if (one[j] === '|') { // 处理“或“的情况
            proce.push(s)
            s = ''
            s += one[0]
          } else {
            s += one[j]
          }
        }
        proce.push(s)
      })
      this.proce = proce
      // 获取一份可视化的产生式
      let proceList = []
      proce.map(pro => { // 把-》 补全
        let arr = pro.split('')
        let str = arr[0]
        str += '->'
        arr.slice(1).map(s => { str += s })
        proceList.push(str)
      })
      this.proceList = proceList

      console.info('产生式proce', this.proce)

      this.getFirst()
      this.getFollow()
      this.getSelect()
      this.getPredictAnalyzeTable()
    },
    getFirst () {
      this.proce.map(pro => { // 依次遍历产生式，求first
        // console.info('map', pro)
        this.first[pro[0]].push(...this.getAFirst(pro[0]))
      })
      // 去重
      this.notEnd.map(i => {
        this.first[i] = [...new Set(this.first[i])] // 去重
      })
      console.info('first', this.first)
    },
    getAFirst (letter) { // 求 给定单个字母的first集 数组
      let res = []
      for (let i = 0; i < this.proce.length; i++) {
        let pro = this.proce[i]
        if (pro[0] === letter) {
          if (this.isEndLetter(pro[1])) { // 判断当前字符是否为终结符，是则直接返回终结符
            res.push(pro[1])
          } else if (this.isNotEndLetter(pro[1])) { // 判断当前字符是否为非终结符，是则 继续求当前字符的下一个字母的first 递归
            res.push(...this.getAFirst(pro[1]))
          }
        }
      }
      return res // 返回结果
    },
    getFollow () {
      // 对于文法的开始符号S,置$于FOLLOW(S)中
      this.follow[this.proce[0][0]].push('$')

      this.notEnd.forEach(letter => { // 选择一个非终结符
        this.proce.forEach(pro => {
          for (let i = 1; i < pro.length; i++) {
            if (letter === pro[i]) { // 如果读到了当前选择的非终结符
              if (i === pro.length - 1) { // 如果i是最后一位的话 后面没有了则 把左部follow添加到右部follow
                this.follow[pro[i]].push(...this.follow[pro[0]])
              } else {
                if (this.isEndLetter(pro[i + 1])) { // 如果是终结符，则加入follow集
                  this.follow[letter].push(pro[i + 1])
                } else if (this.isNotEndLetter(pro[i + 1])) { // 如果是非终结符，则把非终结符的first加入follow集
                  this.follow[letter].push(...this.first[pro[i + 1]])
                  if (this.first[pro[i + 1]].indexOf('@')) { // 如果存在空集 则找当前字母的下一个
                    this.follow[pro[i]].push(...this.follow[pro[0]])
                  }
                }
              }
            }
          }
        })
      })
      // 去重
      this.notEnd.forEach(i => {
        this.follow[i] = [...new Set(this.follow[i])] // 去重
        let arr = []
        this.follow[i].map(s => { // 过滤@空串
          if (s !== '@') {
            arr.push(s)
          }
        })
        this.follow[i] = arr
      })
      console.info('follow', this.follow)
      // 对于文法的开始符号S,置$于FOLLOW(S)中
      // 若A->aBb是一个产生式,则把FIRST(b){ε}加至FOLLOW(B)中;
      // 若A->aB是一个产生式,或A->aBb是一个产生式而b=>ε(即ε∈FIRST(b))则把FOLLOW(A)加至FOLLOW(B)中
    },
    getSelect () {
      let select = []
      this.proce.forEach(pro => { // 依次便利
        let selectObj = { // 构造对象数组
          proce: pro,
          selectList: []
        }
        for (let i = 1; i < pro.length; i++) {
          if (pro[i] === '@') { // 如果是空串，则将 “$” 加入select,，并且把该符号的follow加入select
            selectObj.selectList.push('$')
            selectObj.selectList.push(...this.follow[pro[0]])
          } else if (this.isEndLetter(pro[i])) { // 如果是终结符，则把终结符加入select
            selectObj.selectList.push(pro[i])
          } else if (this.isNotEndLetter(pro[i])) { // 如果是非终结符，则把非终结符的first加入select
            selectObj.selectList.push(...this.first[pro[i]])
            if (selectObj.selectList.indexOf('@') > -1) { // 如果非终结符可以推出空串，则继续查看下一位
              continue // 满足条件才continue 查看下一位用continue
            }
          }
          break // 只看循环的第一次
        }
        select.push(selectObj)
      })
      // selectList的去重
      select.map(obj => {
        obj.selectList = [...new Set(obj.selectList)]
      })
      this.select = [...new Set(this.select)] // 去重
      this.selectTableData = select // 赋值给表格
      console.info('select集合', select)
    },
    getPredictAnalyzeTable () { // 获取预测分析表
      let table = []
      // 构造二维数组
      this.notEnd.map(i => { // 每个非终结符对象 都有 全部的终结符子对象
        table[i] = {}
        // console.info(table)
        this.end.map(j => {
          table[i][j] = ''
        })
      })
      this.selectTableData.map(select => { // 依次便利 构造可视化的产生式
        let pro = select.proce
        let list = select.selectList
        list.map(i => {
          let arr = pro.split('')
          let str = arr[0]
          str += '->'
          arr.slice(1).map(s => { str += s })
          table[pro[0]][i] = str
        })
      })
      this.predictAnalyzeTable = table
      console.info('PredictAnalyzeTable预测分析表', this.predictAnalyzeTable)

      // 可视化
      let tableData = [] // 构造表格数据
      for (let a in table) { // 依次便利表格的非终结符 赋值给对象数组的name
        let one = {
          name: a
        }
        for (let b in table[a]) { // 依次遍历指定非终结符中的 终结符
          if (b === '@') { // 如果是空串，则跳过
            continue
          }
          one[b] = (table[a][b]) // 把产生式数据 赋值上去
        }
        tableData.push(one)
      }
      console.info('tableData预测分析表表格数据', tableData)
      this.predictAnalyzeTableData = tableData
      // 生成一个不含空串的列属性，构造列属性
      let column = []
      this.end.map(i => {
        if (i !== '@') {
          column.push(i)
        }
      })
      this.predictAnalyzeTableColumn = column
    },
    handleAnalyze () { // 点击解析
      this.getAnalyzeTable()
    },
    getAnalyzeTable () { // 生成分析过程
      this.analyzeInfo = [] // 初始化
      let sentence = this.inputSentence.split('') // 分成数组，方便操作
      let stack = []
      stack[0] = '$'
      stack[1] = this.notEnd[0]
      let i = 0; // 表明当前字符是句子中的第几个
      // let top; // 栈顶元素
      this.analyzeLog(stack.slice(0), sentence.slice(i), '') // 生成动作表
      while (stack.length) {
        // 获取输入的当前需要的字符和栈顶字符
        let curLetter = sentence[i]
        if (this.isEndLetter(curLetter) === false && this.isNotEndLetter(curLetter) === false) {
          this.analyzeLog(stack.slice(0), sentence.slice(i), '出错')
          this.$message('未知字符')
          return 'err'
        }
        let top = stack.pop()
        if (curLetter === top && curLetter === '$') { // 结束
          this.analyzeLog(['$'], sentence.slice(i), '匹配成功')
          console.info('success', this.analyzeInfo)
          return
        } else if (curLetter === top && this.isEndLetter(curLetter)) {
          i++
          this.analyzeLog(stack.slice(0), sentence.slice(i), '匹配' + curLetter)
        } else if (this.predictAnalyzeTable[top][curLetter] !== '') {
          // 查找table[栈顶字符][输入的当前字符] 是否存在值 
          // 找出table中对应的值(以数组的形式放着)
          let tmp = this.predictAnalyzeTable[top][curLetter] // 获取产生式的动作
          let tmpArr = tmp.split('').slice(3).reverse()// 取产生式右部，逆序入栈
          if (tmpArr[0] === '@') { // 读取到空串，跳过
            continue
          }
          stack.push(...tmpArr) // 入栈
          this.analyzeLog(stack.slice(0), sentence.slice(i), '输出' + tmp)
        } else {
          this.analyzeLog(stack.slice(0), sentence.slice(i), '出错')
          console.info('出错')
        }
        // this.analyzeLog(stack, sentence.slice(i), '')
        console.info('stack1', stack)
      }
    },
    analyzeLog (stackCur, sentenceCur, msg) {
      let obj = {
        'stackCur': stackCur,
        'sentenceCur': sentenceCur,
        'msg': msg
      }
      obj.stackCur = stackCur
      this.analyzeInfo.push(obj)
    },
    isEndLetter (letter) { // 判断是否是终结符
      if (this.end.indexOf(letter) > -1) {
        return true
      } else {
        return false
      }
    },
    isNotEndLetter (letter) { // 判断是否是非终结符
      if (this.notEnd.indexOf(letter) > -1) {
        return true
      } else {
        return false
      }
    },
    restart () { // 初始化 全部重新
      this.inputEnd = ''
      this.inputNotEnd = ''
      this.production = ''
      this.inputSentence = ''
      this.proceList = []
      this.end = [],
        this.notEnd = []
      this.redictAnalyzeTable = []
      this.proce = []
      this.first = []
      this.follow = []
      this.selectTableData = []
      this.predictAnalyzeTableData = []
      this.predictAnalyzeTableColumn = []
      this.analyzeInfo = []
    },
    getDemo () { // 课本案例
      this.inputEnd = '+ * ( ) a'
      this.inputNotEnd = 'E T F A B'
      this.production = 'E->TA\nA->+TA|@\nT->FB\nB->*FB|@\nF->(E)|a'
      this.inputSentence = 'a*a+a$'
      this.proceList = []
      this.end = [],
        this.notEnd = []
      this.redictAnalyzeTable = []
      this.proce = []
      this.first = []
      this.follow = []
      this.selectTableData = []
      this.predictAnalyzeTableData = []
      this.predictAnalyzeTableColumn = []
      this.analyzeInfo = []
    }
  }
}
</script>

<style lang="scss">
.home {
  max-width: 800px;
  margin: 0 auto;
  .el-row {
    margin-bottom: 20px;
    text-align: center;
  }
}
.el-card__header {
  text-align: center;
}
.el-tag + .el-tag {
  margin-left: 10px;
}
.el-divider--horizontal {
  margin: 10px 0 !important;
}
</style>
