<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query">
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <el-table :data="userList" style="width: 100%" stripe border>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column prop="userName" label="姓名"> </el-table-column>
        <el-table-column prop="email" label="email"> </el-table-column>
        <el-table-column prop="profession" label="职业"> </el-table-column>
        <el-table-column prop="type" label="账户类型"> </el-table-column>
        <el-table-column prop="type" label="状态">
          <template>
            <el-switch></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="120px">
          <template slot-scope="scope">
            <el-button-group>
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="mini"
                round
                @click="editStatus(scope.row.id)"
              ></el-button>
              <el-button
                type="primary"
                icon="el-icon-delete"
                size="mini"
                @click="removeUserById(scope.row.id)"
                round
              ></el-button>
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页部分  -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-size="queryInfo.pagesize"
        :page-sizes="[1, 5, 10, 20]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <el-dialog
      title="添加用户"
      :visible.sync="dialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 主体区域 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="90px"
      >
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="addForm.userName"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" show-password></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" type="email"></el-input>
        </el-form-item>
        <el-form-item label="职业" prop="profession">
          <el-input v-model="addForm.profession"></el-input>
        </el-form-item>
        <el-form-item label="账户类型" prop="type">
          <el-select v-model="addForm.type" placeholder="请选择账户类型">
            <el-option label="管理员" value="0"></el-option>
            <el-option label="企业用户" value="1"></el-option>
            <el-option label="普通用户" value="2"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <!-- 按钮区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户的对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="addFormRules"
        ref="editFormRef"
        label-width="90px"
      >
        <el-form-item label="用户名">
          <el-input v-model="editForm.userName" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" type="email"></el-input>
        </el-form-item>
        <el-form-item label="职业" prop="profession">
          <el-input v-model="editForm.profession"></el-input>
        </el-form-item>
        <el-form-item label="账户类型" prop="type">
          <el-select v-model="editForm.type">
            <el-option label="管理员" value="0"></el-option>
            <el-option label="企业用户" value="1"></el-option>
            <el-option label="普通用户" value="2"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="commitEditInfo()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 1
      },
      userList: [],
      total: 0,
      dialogVisible: false,
      editDialogVisible: false,
      editForm: [],
      addForm: {
        userName: '',
        password: '',
        email: '',
        profession: '',
        type: ''
      },
      addFormRules: {
        userName: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 15, message: '用户名长度在3~15之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '密码长度在6~15之间', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' },
          {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: 'blur,change'
          }
        ],
        profession: [
          { required: true, message: '请输入职业', trigger: 'blur' }
        ],
        type: [{ required: true, message: '请输入账号类型', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('user/get', {
        params: this.queryInfo
      })
      this.userList = res.list
      this.total = res.num
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 添加用户请求
        const { data: res } = await this.$http.post('user/add', this.addForm)
        if (res.code !== 2000) {
          return this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        this.getUserList()
        this.dialogVisible = false
      })
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    async editStatus(id) {
      const { data: res } = await this.$http.get('user/get/' + id)
      console.log(res)
      if (res.code !== 2000) return this.$message.error('查询用户信息失败')
      this.editForm = res.object
      this.editDialogVisible = true
    },
    // 提交修改用户信息
    async commitEditInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'user/edit/' + this.editForm.id,
          {
            email: this.editForm.email,
            profession: this.editForm.profession,
            type: this.editForm.type
          }
        )
        if (res.code !== 2000) return this.$message.error('修改信息失败')
        this.$message.success('修改信息成功')
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    removeUserById(id) {
      console.log(id)
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.get('user/remove/' + id)
          if (res.code !== 2000) return this.$message.error('删除失败')
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          this.getUserList()
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    }
  }
}
</script>

<style lang="less" scoped></style>
