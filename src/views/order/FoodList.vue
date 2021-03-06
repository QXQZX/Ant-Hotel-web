<template>
  <a-card :bordered="false">
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="48">
          <a-col :md="8" :sm="24">
            <a-form-item label="用户编号">
              <a-input v-model="queryParam.userId"  />
            </a-form-item>
          </a-col>
          <a-col :md="8" :sm="24">
            <a-form-item label="用户名">
              <a-input v-model="queryParam.name"  />
            </a-form-item>
          </a-col>
          <a-col :md="!advanced && 8 || 24" :sm="24">
            <span
              class="table-page-search-submitButtons"
              :style="advanced && { float: 'right', overflow: 'hidden' } || {} "
            >
              <a-button type="primary" @click="handleSumbit(queryParam)">查询</a-button>
              <a-button style="margin-left: 8px" @click="() => queryParam = {}">重置</a-button>
              <a @click="toggleAdvanced" style="margin-left: 8px">
                {{ advanced ? '收起' : '展开' }}
                <a-icon :type="advanced ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <a-table
      :loading="loading"
      :columns="columns"
      :pagination="pagination"
      rowKey="orderId"
      :dataSource="loadData"
    >
      <span slot="action" slot-scope="text, record">
        <template>
          <a @click="handleEdit(record)">详情</a>
        </template>
      </span>
    </a-table>
  </a-card>
</template>

<script>
import moment from 'moment'
import { STable, Ellipsis } from '@/components'
import StepByStepModal from '@/views/list/modules/StepByStepModal'
import CreateForm from '@/views/list/modules/CreateForm'
import { getRoleList, getFoodList } from '@/api/manage'
import { foodSearch } from '../../api/manage'

const statusMap = {
  0: {
    status: 'default',
    text: '支付中'
  },
  1: {
    status: 'processing',
    text: '待支付'
  },
  2: {
    status: 'success',
    text: '已支付'
  },
  3: {
    status: 'error',
    text: '异常'
  }
}

export default {
  name: 'TableList',
  components: {
    STable,
    Ellipsis,
    CreateForm,
    StepByStepModal
  },
  data() {
    return {
      mdl: {},
      // 高级搜索 展开/关闭
      advanced: false,
      // 查询参数
      queryParam: {},
      // 表头
      loading: true,
      // 表头
      columns: [
        {
          title: '订单编号',
          dataIndex: 'orderId'
        },
        // {
        //   title: '用户ID',
        //   dataIndex: 'userId'
        // },
        {
          title: '用户姓名',
          dataIndex: 'name'
        },
        {
          title: '消费金额',
          dataIndex: 'cost',
          sorter: true,
          needTotal: true,
          customRender: text => text + ' RMB'
        },
        {
          title: '用餐券',
          dataIndex: 'coupon',
          sorter: true,
          needTotal: true,
          customRender: text => text + ' RMB'
        },
        {
          title: '生成时间',
          dataIndex: 'onTime',
          sorter: true,
          customRender: (val) => {return moment(val).format('YYYY-MM-DD HH:mm:ss')}
        },
        {
          title: '食物',
          dataIndex: 'food',
          scopedSlots: { customRender: 'status' }
        },
        {
          title: '操作',
          dataIndex: 'action',
          width: '150px',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      pagination: {
        pageSize: 10,
        defaultPageSize: 10,
        defaultCurrent: 1,
        showSizeChanger: true,
        showQuickJumper: true,
        pageSizeOptions: ['10', '20', '30', '40']
      },
      loadData: [],
      selectedRowKeys: [],
      selectedRows: [],

      // custom table alert & rowSelection
      options: {
        alert: {
          show: true,
          clear: () => {
            this.selectedRowKeys = []
          }
        },
        rowSelection: {
          selectedRowKeys: this.selectedRowKeys,
          onChange: this.onSelectChange
        }
      },
      optionAlertShow: false
    }
  },
  filters: {
    statusFilter(type) {
      return statusMap[type].text
    },
    statusTypeFilter(type) {
      return statusMap[type].status
    }
  },
  created() {
    this.vueTable()
    // this.tableOption()
    // getRoleList({ t: new Date() })
  },
  methods: {
    handleSumbit(queryParam){
      this.queryParam=queryParam
      foodSearch({userId: queryParam.userId,name:queryParam.name}).then(res => {
        console.log(res)
        this.loadData = res.data
        this.loading = false
      })
        .catch(e => {
          this.loading = false
        })
    },
    tableOption() {
      if (!this.optionAlertShow) {
        this.options = {
          alert: {
            show: true,
            clear: () => {
              this.selectedRowKeys = []
            }
          },
          rowSelection: {
            selectedRowKeys: this.selectedRowKeys,
            onChange: this.onSelectChange,
            getCheckboxProps: record => ({
              props: {
                disabled: record.no === 'No 2', // Column configuration not to be checked
                name: record.no
              }
            })
          }
        }
        this.optionAlertShow = true
      } else {
        this.options = {
          alert: false,
          rowSelection: null
        }
        this.optionAlertShow = false
      }
    },
    vueTable() {
      getFoodList()
        .then(res => {
          console.log(res)
          this.loadData = res.data
          this.loading = false
        })
        .catch(e => {
          this.loading = false
        })
    },
    handleEdit(record) {
      console.log(record.orderId)
      this.$router.push({
        path: '/order/foodDetail',
        query: {
          orderId: record.orderId
        }
      })
      // this.$refs.modal.edit(record)
    },
    handleSub(record) {
      if (record.status !== 0) {
        this.$message.info(`${record.no} 订阅成功`)
      } else {
        this.$message.error(`${record.no} 订阅失败，规则已关闭`)
      }
    },
    handleOk() {
      this.$refs.table.refresh()
    },
    onSelectChange(selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    toggleAdvanced() {
      this.advanced = !this.advanced
    },
    resetSearchForm() {
      this.queryParam = {
        date: moment(new Date())
      }
    }
  }
}
</script>
