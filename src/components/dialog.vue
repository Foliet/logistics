<template>
  <el-dialog v-model="dialogVisible"
             title="添加订单"
             top="5vh" width="40%">

    <div>
      <div class="space1">发件人地址</div>
      <el-select
          v-model="orderData.senderContactId"
          placeholder="选择我的地址"
          style="width:90%">
        <el-option
            v-for="contact in contacts"
            :key="contact.receiverId"
            :label="contact.receiverName"
            :value="contact.id"
        >
          <span>地址：{{ contact.address }}</span>
          <el-popover
              :width="200"
              placement="right"
              style="height: 10%"
              trigger="hover"
          >
            <div>电话：{{ contact.telephone }}</div>
            <div>省市区：{{ contact.province }},{{ contact.city }},{{ contact.district }}</div>
            <div>地址：{{ contact.address }}</div>
            <template #reference>
              <span style="float: right">...</span>
            </template>
          </el-popover>
        </el-option>
      </el-select>
      <el-button @click="this.$refs.g.dialogVisible=true">
        <el-icon>
          <plus/>
        </el-icon>
      </el-button>
    </div>
    <div>
      <div class="space1">收件人</div>
      <el-select
          v-model="orderData.receiverContactId"
          placeholder="选择收件人"
          style="width:90%">
        <el-option
            v-for="contact in contacts1"
            :key="contact.receiverId"
            :label="contact.receiverName"
            :value="contact.id"
        >
          <span>联系人：{{ contact.receiverName }}</span>
          <el-popover
              :width="200"
              placement="right"
              style="height: 10%"
              trigger="hover"
          >
            <div>电话：{{ contact.telephone }}</div>
            <div>省市区：{{ contact.province }},{{ contact.city }},{{ contact.district }}</div>
            <div>地址：{{ contact.address }}</div>
            <template #reference>
              <span style="float: right">...</span>
            </template>
          </el-popover>
        </el-option>
      </el-select>
      <el-button @click="this.$refs.d.dialogVisible=true">
        <el-icon>
          <plus/>
        </el-icon>
      </el-button>
    </div>
    <div class="space1">货物名称</div>
    <el-input v-model="orderData.title" clearable placeholder="货物名称" type="text"/>
    <div class="space1">货物介绍</div>
    <el-input v-model="orderData.remark" :autosize="{minRows:4,maxRows:7}" clearable placeholder="货物介绍"
              type="textarea"/>
    <div class="space1">
      <span style="padding: 0 30% 0 0">体积</span><span style="padding: 0 28% 0 0">重量</span><span>价值</span>
    </div>
    <div>
      <el-input v-model="orderData.volume" class="space2" min="0" placeholder="体积" type="text">
        <template #append>cm³</template>
      </el-input>
      <el-input v-model="orderData.weight" class="space2" min="0" placeholder="重量" type="text">
        <template #append>KG</template>
      </el-input>
      <el-input v-model="orderData.value" class="space2" min="0" placeholder="价值" type="text">
        <template #append>CNY</template>
      </el-input>
    </div>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addOrder()"
        >确认</el-button>
      </span>
    </template>
  </el-dialog>
  <dia ref="d" type="others" @confirm="getOptions"></dia>
  <dia ref="g" type="mine" @confirm="getOptions"></dia>
</template>

<script>
import {Plus} from '@element-plus/icons'
import dia from '../components/dia'

export default {
  emits: ['confirm'],
  components: {
    Plus,
    dia,
  },
  data() {
    return {
      dialogVisible: "",
      handleClose: "",
      orderData:
          {
            senderContactId: '',
            receiverContactId: '',
            title: "",
            weight: "",
            value: "",
            volume: "",
            remark: "",
          },
      contacts: [],
      contacts1: [],
    }
  },
  created() {
    this.dialogVisible = false
    this.getOptions()
  },
  methods: {
    addOrder: function () {
      if (this.orderData.senderContactId != null && this.orderData.receiverContactId != null && this.orderData.title != null && this.orderData.title.length > 0 && Number(this.orderData.weight) > 0 && Number(this.orderData.volume) > 0 && Number(this.orderData.value > 0)) {
        this.$axios.post('/add-order', this.orderData).then(() => {
          this.$emit("confirm")
        })
        this.dialogVisible = false;
      } else {
        this.$message.error('输入信息不可为空！')
      }
    },
    reset: function () {
      for (const key in this.orderData) {
        this.orderData[key] = null;
      }
    },
    getOptions() {
      this.$axios.get('/get-contacts?type=mine').then(res => {
        this.contacts = res.data.contacts;
      })
      this.$axios.get('/get-contacts?type=others').then(res => {
        this.contacts1 = res.data.contacts;
      })
    },
  }
}
</script>

<style>
.space1 {
  padding: 10px 0;
}

.space2 {
  width: 33%;
}


</style>
