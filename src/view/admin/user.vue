<template>
  <el-container style="margin: 0 0 0 0">
    <el-container class="nameness">
      <el-main>
        <div style="padding:10px 0 0 0">
          <el-icon style="font-size: 20px;margin-right: 1em">
            <Search/>
          </el-icon>
          <el-input v-model="search" placeholder="输入关键字搜索" style="width: 95%" type="text"></el-input>
        </div>
        <br/>
        <div style="height: 79%">
          <el-table id="table1"
                    :data="users.slice((currentPage-1)*pageSize,currentPage*pageSize)" height="100%">
            <el-table-column label="Id" prop="id" sortable/>
            <el-table-column label="用户名" prop="username" sortable/>
            <el-table-column label="Email" prop="email" sortable/>
            <el-table-column label="用户组" prop="groupName" sortable/>
            <el-table-column>
              <template #header>
                <el-button style="width: 50%;float: right;margin-right: 25%" @click="addUser">
                  新增
                </el-button>
              </template>
              <template #default="scope">
                <el-button round style="color: #FFB500; border: 1px #FFB500 solid" @click="edit(scope.row)">
                  修改
                </el-button>
                |
                <el-button round style="color: #FF3D00;border: 1px #FF3D00 solid" @click="scope.row.visible=true">删除
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
          </el-table>
        </div>
        <dialog3 ref="d" @confirm="getUsers"></dialog3>
      </el-main>
    </el-container>
    <el-footer>
      <div style="padding-left: 35%">
        <el-pagination :current-page="currentPage" :page-size="pageSize" :total="users.length" background
                       layout="prev, pager, next, jumper" style="width: 40%;float: left"
                       @current-change="currentChange">
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
    </el-footer>
  </el-container>
</template>

<script>
import dialog3 from '/src/components/dialog3'
import {Search} from "@element-plus/icons";

export default {
  components: {
    dialog3,
    Search
  },
  data() {
    return {
      currentPage: 1,
      pageSize: 7,
      users: [],
      allUsers: [],
      search: null,
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
  watch: {
    search() {
      this.filter()
    }
  },
  mounted() {
    document.title = "用户管理"
  },
  created() {
    this.getUsers()
  },
  methods: {
    currentChange(index) {
      this.currentPage = index
    },
    getUsers() {
      this.$axios.get('/admin/get-users').then(res => {
        const currentPage = this.currentPage
        this.allUsers.length = 0
        for (const user of res.data.users) {
          user.visible = false
          if (user.groupId === 0) {
            user.groupName = '用户'
          } else {
            user.groupName = '管理员'
          }
          this.allUsers.push(user)
        }
        this.filter()
        this.currentPage = currentPage
      })
    },
    filter() {
      this.users.length = 0;
      this.users = this.allUsers.filter(
          (data) =>
              !this.search || data.username.toLowerCase().includes(this.search.toLowerCase())
              || data.email.toLowerCase().includes(this.search.toLowerCase())
      )
    },
    addUser() {
      this.$refs.d.reset()
      this.$refs.d.dialogVisible = true;
    },
    edit(row) {
      this.$refs.d.tableData.username = row.username;
      this.$refs.d.tableData.email = row.email;
      this.$refs.d.tableData.password = row.password;
      this.$refs.d.tableData.id = row.id;
      this.$refs.d.tableData.groupId = row.groupId;
      this.$refs.d.dialogVisible = true;
    },
    deleted(row) {
      this.$axios.post('/admin/delete-user', {id: row.id}).then(() => {
        this.getUsers()
      })
    }
  }
}
</script>

<style scoped>
.nameness {
  margin: 0;
  display: flex;
  vertical-align: center;
}

#table1 {
  width: 100%;
  border: #E5E5E5 2px solid;
  border-radius: 15px;
}
</style>