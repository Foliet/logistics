<template>
  <el-container>
    <el-container>
      <el-main>
        <div style="height: 96%">
          <el-table id="table1"
                    :data="orders.slice((currentPage-1)*pageSize,currentPage*pageSize)" height="94%" style="width: 100%;">
            <el-table-column label="Id" prop="id" sortable/>
            <el-table-column label="寄件人名字" prop="senderName" sortable/>
            <el-table-column label="取件人名字" prop="receiverName" sortable/>
            <el-table-column label="状态" prop="statusName" sortable/>
            <el-table-column label="操作">
              <template #default="scope">
                <el-button :disabled="scope.row.status!==0" @click="showTable=true;selectedOrder=scope.row">处理订单
                </el-button>
              </template>
            </el-table-column>
            <el-table-column>
              <template #header>
                <el-input v-model="search" placeholder="Type to search" size="mini"/>
              </template>
              <template #default="scope">
                <el-button :disabled="scope.row.status!==0&&scope.row.status!==3" @click="scope.row.visible = true">删除
                </el-button>
                <el-dialog v-model="scope.row.visible">确定要删除吗？
                  <template #footer>
      <span class="dialog-footer">
        <el-button @click="scope.row.visible = false">取消</el-button>
        <el-button type="primary" @click="deleted(scope.row)"
        >确认</el-button>
      </span>
                  </template>
                </el-dialog>
              </template>
            </el-table-column>
            <el-table-column></el-table-column>
          </el-table>
          <el-drawer
              v-model="showTable"
              direction="rtl"
              size="50%"
              title="处理订单："
          >
            <div class="space1">选择车辆：</div>
            <el-select v-model="selectedOrder.chunk" :disabled="selectedOrder.status!==0" placeholder="Select chunk">
              <el-option
                  v-for="item in chunks"
                  :key="item.id"
                  :label="item.model"
                  :value="item.id"
              >
                {{ item.model }}
              </el-option>
            </el-select>
            <div class="space1">选择配货员：</div>
            <el-select v-model="selectedOrder.staff" :disabled="selectedOrder.status!==0" placeholder="Select staff">
              <el-option
                  v-for="item in staffs"
                  :key="item.status"
                  :label="item.name"
                  :value="item.id"
              >
                {{ item.name }}
              </el-option>
            </el-select>
            <div class="space1"></div>
            <el-button :disabled="selectedOrder.status!==0" style="float: right"
                       @click="matching(selectedOrder);showTable=false">确定
            </el-button>
          </el-drawer>
        </div>
        <div style="text-align: center">
          <el-pagination :current-page="currentPage" :page-size="pageSize" :total="orders.length"
                         background layout="prev, pager, next, jumper" style="float: left" @current-change="currentChange">
          </el-pagination>
          <a>选择每页展示数：</a>
          <el-select v-model="pageSize" size="mini" style="width: 10%;">
            <el-option
                v-for="typer in typeses"
                :key="typer"
                :value="typer.value">
              {{ typer.value }}
            </el-option>
          </el-select>
        </div>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  data() {
    return {
      search: null,
      currentPage: 1,
      pageSize: 9,
      orders: [],
      chunks: [],
      staffs: [],
      selectedOrder: {},
      allOrders: [],
      showTable: false,
      typeses: [{value: 1,},
        {value: 2,},
        {value: 3,},
        {value: 4,},
        {value: 5,},
        {value: 6,},
        {value: 7,},
        {value: 8,},
        {value: 9,},
        {value: 10,},
        {value: 11,},
        {value: 12,},
        {value: 13,},
      ]
    }
  },
  created() {
    this.getOrders()
    this.getChunks()
    this.getStaffs()
  },
  watch: {
    search() {
      this.filter()
    }
  },
  mounted() {
    document.title = "订单管理"
  },
  methods: {

    currentChange(index) {
      this.currentPage = index
    },
    getOrders() {
      this.$axios.get('/admin/get-orders').then(res => {
        const currentPage = this.currentPage
        this.allOrders.length = 0
        for (const order of res.data.orders) {
          order.visible = false
          if (order.status === 0) {
            order.statusName = '待出仓'
          } else if (order.status === 1) {
            order.statusName = '运输中'
          } else if (order.status === 2) {
            order.statusName = '待收货'
          } else {
            order.statusName = '已送达'
          }
          this.allOrders.push(order)
        }
        this.allOrders.sort((a, b) => {
          return b.id - a.id
        })
        this.filter()
        this.currentPage = currentPage
      })
    },
    getChunks() {
      this.$axios.get('/admin/get-chunks').then(res => {
        this.chunks.length = 0
        this.chunks = res.data.chunks.filter(chunk => chunk.status === 0)
      })
    },
    getStaffs() {
      this.$axios.get('/admin/get-staffs').then(res => {
        this.staffs.length = 0
        this.staffs = res.data.staffs.filter(staff => staff.status === 0)
      })
    },
    filter() {
      this.orders.length = 0;
      this.orders = this.allOrders.filter(
          (data) =>
              !this.search || data.senderName.toLowerCase().includes(this.search.toLowerCase())
              || data.receiverName.toLowerCase().includes(this.search.toLowerCase())
      )
    },
    matching(row) {
      this.$axios.post('/admin/match', {
        orderId: row.id,
        chunkId: row.chunk,
        staffId: row.staff
      }).then(() => {
        this.getOrders()
        this.getChunks()
        this.getStaffs()
        setTimeout(() => {
          this.$axios.post("/admin/arrival", {orderId: row.id}).then(() => {
            this.getOrders()
            this.getChunks()
            this.getStaffs()
          })
        }, 10000)
      })
    },
    deleted(row) {
      this.$axios.post('/admin/delete-order', {orderId: row.id}).then(() => {
        this.getOrders()
        this.getChunks()
        this.getStaffs()
      })
    }
  }
}
</script>

<style scoped>
.space1 {
  padding: 10px 0;
}

#table1 {
  width: 100%;
  border: #E5E5E5 2px solid;
  border-radius: 15px;
}

</style>