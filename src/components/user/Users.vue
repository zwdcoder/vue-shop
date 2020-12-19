<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片试图区域-->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input clearable placeholder="请输入内容" v-model="queryInfo.query" @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!--用户列表区域-->
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!--修改按钮-->
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)"></el-button>
            <!--删除按钮-->
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeUserByid(scope.row.id)"></el-button>
            <!--分配角色按钮-->
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button size="mini" type="warning" icon="el-icon-setting" @click="setRole(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1,2,5,10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!--添加用户的对话框-->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
      >
      <el-form
      :model="addForm"
      :rules="addFormRules"
      ref="addFormRef"
      label-width="70px"
      class="demo-ruleForm">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!--修改的用户的对话框-->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      >
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="setRoleDialog"
    @close="setRolesDialogClosed" width="50%">
      <div>
        <p>当前的用户:{{userInfo.username}}</p>
        <p>当前的角色:{{userInfo.role_name}}</p>
        <p>
          分配新角色：
          <el-select v-model="selectedRoleId" placeholder="请选择角色">
            <el-option
              v-for="item in roleList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            ></el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer">
        <el-button @click=" setRoleDialog= false">取 消</el-button>
        <el-button type="primary" @click="saveRolesInfo">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default{
  data() {
    //验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/
        if (regEmail.test(value)) return cb()
        cb(new Error("请输入合法的邮箱"))
    };
    //验证手机号的规则
    var checkMobile = (rule, value, cb) => {
        const regMobile = /^1[3456789]\d{9}$/
        if (regMobile.test(value)) return cb()
        cb(new Error("请输入正确的手机号"))
    };

    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        //当前每页显示的数据
        pagesize: 2
      },
      userList: [],
      total: 0,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      }, //添加用户的表单数据
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名的长度在3~10为之间', trigger: 'blur'}
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '用户名的长度在6~15为之间', trigger: 'blur'}
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur'}
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur'}
        ]
      }, //添加表单的验证规则
      addDialogVisible: false,
      editDialogVisible: false,
      //用户查询
      editForm: {},
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur'}
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur'}
        ]
      },
      setRoleDialog: false,
      userInfo: [],
      roleList: [],
      selectedRoleId: ""
    }
  },
  created() {
    this.getUserList();
  },
  methods: {
    getUserList() {
      this.$axios.get("users", { params: this.queryInfo }).then(res => {
        this.userList = res.data.data.users;
        this.total = res.data.data.total;
      });
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    //监听switch状态的改变
    userStateChanged(userinfo){
      console.log(userinfo)
      this.$axios
        .put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
        .then(res => {
          let resp = res.data;
          if (resp.meta.status !== 200) {
            userinfo.mg_state = !userinfo.mg_state;
            return this.$message.error("更新用户状态失败!")
          }
          this.$message.success("更新用户状态成功!")
      })
    },
    //监听对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    addUser() {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return;
        this.$axios.post("users", this.addForm).then(res => {
          let resp = res.data
          if (resp.meta.status !== 201) {
              return this.$message({
                  message: "添加用户失败！",
                  type: "error"
              })
          }
          this.$message({
              message: "添加用户成功!",
              type: "success"
          });
          this.addDialogVisible = false
          this.getUserList()
        })
      })
    },
    //展示编辑用户的对话框
    showEditDialog(id) {
      this.editDialogVisible = true;
      this.$axios.get(`users/${id}`).then(response => {
        let res = response.data;
        if (res.meta.status !== 200)
            return this.$message({
                message: "查询用户信息失败",
                type: "error",
                showClose: true
            });
        this.editForm = res.data;
      });
    },
    //修改用户信息
    editUserInfo() {
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return
        this.$axios
          .put(`users/${this.editForm.id}`, this.editForm)
          .then(response => {
              let res = response.data;
              if (res.meta.status !== 200)
                  return this.$message({
                      message: "更新用户信息失败",
                      type: "error",
                      showClose: true
                  });
              this.$message({
                  message: "更新用户信息成功",
                  type: "success",
                  showClose: true
              });
              this.editDialogVisible = false
              this.getUserList()
          })
      })
    },
    removeUserByid(id){
      this.$confirm("是否确认删除该用户?", "删除用户", {
      	confirmButtonText: "确定",
      	cancelButtonText: "取消",
      	type: "warning",
      	callback: action => {
      		if (action !== "confirm") return;
      		this.$axios.delete(`users/${id}`).then(response => {
      			let res = response.data;
      			if (res.meta.status !== 200)
      				return this.$message({
      					message: "删除失败!",
      					type: "error",
      					showClose: true
      				});
      			this.$message({
      				message: "删除成功",
      				type: "success",
      				showClose: true
      			});
      			this.getUserList();
      		});
      	}
      });
    },
    setRole(userInfo) {
      this.userInfo = userInfo
      this.$axios.get("roles").then(response => {
        let res = response.data;
        if (res.meta.status !== 200)
          return this.$message({
              message: "获取角色列表失败！",
              type: "error",
              showClose: true
          });
        this.roleList = res.data;
      });
      this.setRoleDialog = true
    },
    // 点击按钮，分配角色
    async saveRolesInfo (){
      if (!this.selectedRoleId) {
        return this.$message.error('请选择角色！')
      }
      this.$axios
        .put(`users/${this.userInfo.id}/role`, {
          rid: this.selectedRoleId
        })
        .then(response => {
          let res = response.data;
          if (res.meta.status !== 200)
            return this.$message({
              message: "分配角色失败!",
              type: "error",
              showClose: true
            });
          this.$message({
            message: '更新角色成功！',
            type: 'success',
            showClose: true,
          });
          this.getUserList();
          this.setRoleDialog=false
        });
    },
    // 分配角色对话框的关闭事件
    setRolesDialogClosed() {
      this.selectedRoleId=''
      this.userInfo=[]
    }
  }
}
</script>

<style>
</style>
