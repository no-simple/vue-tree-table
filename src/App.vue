<template>
  <div id="app">
              <a href="https://github.com/no-simple/vue-tree-table"  class="app_title">基于vue element-ui中table的 tree table</a>
                <el-table 
                  :data="tableListData"
                  :row-style="toggleDisplayTr"
                  border stripe
                  class="init_table">
                      <el-table-column
                      align="center"
                      width="55"
                      type="index"
                      label="序号">
                      </el-table-column>
                    <el-table-column
                      label="权限名称"
                      min-width="200"
                      show-overflow-tooltip
                      align="left">
                          <template slot-scope="scope">
                            <p :style="`margin-left: ${scope.row.__level * 20}px;margin-top:0;margin-bottom:0`"><i  @click="toggleFoldingStatus(scope.row)" class="permission_toggleFold" :class="toggleFoldingClass(scope.row)"></i>{{scope.row.Name}}</p>
                          </template>
                    </el-table-column>
                    <el-table-column
                      align="center"
                      width="90"
                      prop="__level"
                      label="层级">
                    </el-table-column>
                    <el-table-column
                      align="left"
                      prop="__identity"
                      label="节点标识">
                    </el-table-column>
                    <el-table-column
                      align="center"
                      width="80"
                      label="图标">
                      <template slot-scope="scope">
                        <i :class="scope.row.Class"></i>
                      </template>
                    </el-table-column>
                    <el-table-column
                      align="center"
                      prop="PId"
                      width="100"
                      label="Action">
                    </el-table-column>

                    <el-table-column
                      align="center"
                      width="100"
                      label="操作">
                          <template slot-scope="scope">
                            <el-button type="text">编辑</el-button>
                            <el-button type="text">删除</el-button>
                          </template>
                    </el-table-column>
                </el-table>
  </div>
</template>

<script>
  import data from "./data"
  import Vue from 'vue'
export default {
  data: () => ({
      tableListData: [], // tableListData 展示数据的data
      foldList: [] // 该数组中的值 都会在列表中进行隐藏  死亡名单    
  }),
  methods: {
    /*********************************
      ** Fn: toggleFoldingStatus
      ** Intro: 切换展开 还是折叠
      ** @params: params 当前点击行的数据
      ** Author: zyx
    *********************************/
    toggleFoldingStatus (params) {
      this.foldList.includes(params.__identity) ? this.foldList.splice(this.foldList.indexOf(params.__identity), 1) : this.foldList.push(params.__identity)
    },
    /*********************************
      ** Fn: toggleDisplayTr
      ** Intro: 该方法会对每一行数据都做判断 如果foldList 列表中的元素 也存在与当前行的 __family列表中  则该行不展示
      ** @params: 
      ** Author: zyx
    *********************************/
    toggleDisplayTr ({row, index}) {
      for (let i = 0; i < this.foldList.length; i++) {
        let item = this.foldList[i]
        // 如果foldList中元素存在于 row.__family中，则该行隐藏。  如果该行的自身标识等于隐藏元素，则代表该元素就是折叠点
        if (row.__family.includes(item) && row.__identity !== item) return 'display:none;'
      }
      return ''
    },
    /*********************************
      ** Fn: toggleFoldingClass
      ** Intro: 获取展开的class
      ** @params: 
      ** Author: zyx
    *********************************/
    toggleFoldingClass (params) {
      return params.Children.length === 0 ? 'permission_placeholder' : (this.foldList.indexOf(params.__identity) === -1 ? 'iconfont icon-minus-square-o' : 'iconfont icon-plussquareo')
    },
    /*********************************
      ** Fn: formatConversion
      ** Intro: 将树形接口数据扁平化
      ** @params: parent 为当前累计的数组  也是最后返回的数组
      ** @params: children 为当前节点仍需继续扁平子节点的数据
      ** @params: index 默认等于0， 用于在递归中进行累计叠加 用于层级标识
      ** @params: family 装有当前包含元素自身的所有父级 身份标识
      ** @params: elderIdentity 父级的  唯一身份标识
      ** Author: zyx
    *********************************/
    formatConversion (parent, children, index = 0, family = [], elderIdentity = 'x') {
      // children如果长度等于0，则代表已经到了最低层
      // let page = (this.startPage - 1) * 10
      if (children.length > 0) {
        children.map((x, i) => {
          // 设置 __level 标志位 用于展示区分层级
          Vue.set(x, '__level', index)
          // 设置 __family 为家族关系 为所有父级，包含本身在内
          Vue.set(x, '__family', [...family, elderIdentity + '_' + i])
          // 本身的唯一标识  可以理解为个人的身份证咯 一定唯一。
          Vue.set(x, '__identity', elderIdentity + '_' + i)
          parent.push(x)
          // 如果仍有子集，则进行递归
          if (x.Children.length > 0) this.formatConversion(parent, x.Children, index + 1, [...family, elderIdentity + '_' + i], elderIdentity + '_' + i)
        })
      } return parent
    }
  },
  created () {
    // 测试数据 data
    this.tableListData = this.formatConversion([], data)
  }
}
</script>

<style lang='stylus' rel='stylesheet/stylus'>
  .app_title
      display block
      width 100%
      font-size 24px
      line-height 60px
      color #41dae4
      text-align center
  .permission_toggleFold
      vertical-align middle
      padding-right 5px
      font-size 16px
      cursor pointer
  .permission_placeholder
      content ' '   
      display inline-block 
      width 16px
      font-size 16px
  .init_table
      width 66% !important
      max-width 900px !important
      margin 0 auto !important
      th  
          background-color: #edf6ff
          text-align: center !important
          color #066cd4
          font-weight bold
          .cell 
              padding 0 !important
      td, th    
          font-family: '宋体'
          font-size 12px
          padding 0 !important    
          height 40px !important   
      .el-table--border, .el-table--group
          border: 1px solid #dde2ef
      td, th.is-leaf
          border-bottom: 1px solid #dde2ef
      .el-table--border td, .el-table--border th, .el-table__body-wrapper .el-table--border.is-scrolling-left~.el-table__fixed
          border-right: 1px solid #dde2ef    
      .el-table--striped .el-table__body tr.el-table__row--striped td
          background-color #f7f9fa  
</style>
