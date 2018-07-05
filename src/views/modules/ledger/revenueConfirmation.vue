<template>
  <div class="mod-menu">
    <el-table :data="dataList" border max-height="500" style="width: 100%;">
      <el-table-column prop="projectName" fixed="left" show-overflow-tooltip header-align="center" align="center" width="100" label="项目名称"></el-table-column>
      <el-table-column prop="customer" fixed="left" show-overflow-tooltip header-align="center" align="center" treeKey="menuId" width="150" label="客户名称"></el-table-column>
      <el-table-column prop="propertyType" fixed="left" show-overflow-tooltip header-align="center" align="center" width="120" label="产品类型"></el-table-column>
      <el-table-column prop="reservedNo"  show-overflow-tooltip header-align="center" align="center" width="120" label="认购单号"></el-table-column>
      <el-table-column prop="reservedDate" show-overflow-tooltip header-align="center" align="center" width="120" label="认购日期"></el-table-column>
      <el-table-column prop="waiterNo" show-overflow-tooltip header-align="center" align="center" width="120" label="员工编号"></el-table-column>
      <el-table-column prop="waiterName" show-overflow-tooltip header-align="center" align="center" width="120" label="业务员"></el-table-column>
      <el-table-column prop="signNo" show-overflow-tooltip header-align="center" align="center" width="120" label="签约单号"></el-table-column>
      <el-table-column prop="connections" show-overflow-tooltip header-align="center" align="center" width="120" label="原客户性质"></el-table-column>
      <el-table-column prop="changedConnections" show-overflow-tooltip header-align="center" align="center" width="120" label="变更后客户性质"></el-table-column>
      <el-table-column prop="roomName" show-overflow-tooltip header-align="center" align="center" width="120" label="房源"></el-table-column>
      <el-table-column prop="customerPhone" show-overflow-tooltip header-align="center" align="center" width="120" label="联系电话"></el-table-column>
      <el-table-column prop="signDate" show-overflow-tooltip header-align="center" align="center" width="120" label="签约日期"></el-table-column>
      <el-table-column prop="signBackDate" show-overflow-tooltip header-align="center" align="center" width="120" label="退签日期"></el-table-column>
      <el-table-column prop="buildingArea" show-overflow-tooltip header-align="center" align="center" width="120" label="面积（㎡）"></el-table-column>
      <el-table-column prop="signPrice" show-overflow-tooltip header-align="center" align="center" width="120" label="签约合同单价（元）"></el-table-column>
      <el-table-column prop="signAmount" show-overflow-tooltip header-align="center" align="center" width="120" label="签约合同总价（元）"></el-table-column>
      <el-table-column prop="baseRightsIncome" show-overflow-tooltip header-align="center" align="center" width="120" label="基础权责收入"></el-table-column>
      <el-table-column prop="agencyCommissionAmount" show-overflow-tooltip header-align="center" align="center" width="120" label="基础代理费金额"></el-table-column>
      <el-table-column prop="agencyCommissionRate" show-overflow-tooltip header-align="center" align="center" width="120" label="代理费率%"></el-table-column>
      <el-table-column prop="agencyCommissionAuditStatus" show-overflow-tooltip header-align="center" align="center" width="120" label="基础权责收入审核状态"></el-table-column>
      <el-table-column prop="agencyCommissionAuditDate" show-overflow-tooltip header-align="center" align="center" width="120" label="基础权责收入确认日期"></el-table-column>
      <el-table-column prop="rightsIncome" show-overflow-tooltip header-align="center" align="center" width="120" label="权责收入总额"></el-table-column>
      <el-table-column prop="accrualCommissionRate" show-overflow-tooltip header-align="center" align="center" width="120" label="计提佣金比率"></el-table-column>
      <el-table-column prop="accrualCommissionAmount" show-overflow-tooltip header-align="center" align="center" width="120" label="计提佣金金额"></el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
          userName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/ledger/revenueConfirmation/getData'),
          method: 'get',
          params: this.$http.adornParams({
            'current': this.pageIndex,
            'size': this.pageSize
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var userIds = id ? [id] : this.dataListSelections.map(item => {
          return item.userId
        })
        this.$confirm(`确定对[id=${userIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/sys/user/delete'),
            method: 'post',
            data: this.$http.adornData(userIds, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      }
    }
  }
</script>
