<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图-->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addFormDialog=true">添加角色</el-button>
        </el-col>
      </el-row>
      <!--角色列表区域-->
      <el-table :data="rolelist" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!--一级权限-->
              <el-col :span="5">
                <el-tag>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--二级权限-->
              <el-col :span="19">
                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById()">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRoles(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addFormDialog" width="50%">
      <el-form :model="addForm" ref="addFormRef" :rules="addFormRules" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addFormDialog = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户" :visible.sync="editFormDialog" width="50%">
      <el-form :model="editForm" ref="editFormRef" :rules="editFormRules" label-width="80px">
        <el-form-item label="ID">
          <el-input v-model="editForm.roleId" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editFormDialog = false">取 消</el-button>
        <el-button type="primary" @click="editRoles">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        //列表数据
        rolelist: [],
        SetRightDialog: false,
        rightList: [],
        treeProps: {
          label: "authName",
          children: "children"
        },
        addForm: {
          roleName: "",
          roleDesc: ""
        },
        editForm: {},
        addFormRules: {
          roleName: [{
            required: true,
            message: "请输入角色名称",
            trigger: "blur"
          }]
        },
        editFormRules: {
          roleName: [{
            required: true,
            message: "请输入角色名称",
            trigger: "blur"
          }]
        },
        addFormDialog: false,
        editFormDialog: false,
        defKeys: [],
        roleId: ""
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      getRolesList() {
        this.$axios.get("roles").then(response => {
          let res = response.data
          if (res.meta.status !== 200)
            return this.$message({
              message: "获取角色列表失败",
              type: "error",
              showClose: true
            })
          this.rolelist = res.data;
        })
      },
      addUser() {
        this.$refs.addFormRef.validate(valid => {
          if (!valid) return;
          this.$axios.post("roles", this.addForm).then(response => {
            let res = response.data;
            if (res.meta.status !== 201)
              return this.$message({
                message: "添加角色失败！",
                type: "error",
                showClose: true
              });
            this.$message({
              message: "添加用户成功！",
              type: "success",
              showClose: true
            });
            this.addFormDialog = false;
            this.getRolesList();
          });
        });
      },
      showEditRoles(id) {
        this.$axios.get(`roles/${id}`).then(response => {
          let res = response.data;
          if (res.meta.status !== 200)
            return this.$message({
              message: "获取角色信息失败！",
              type: "error",
              showClose: true
            });
          this.editForm = res.data;
          this.editFormDialog = true;
        });
      },
      editRoles() {
        this.$refs.editFormRef.validate(valid => {
          if (!valid) return;
          this.$axios
            .put(`roles/${this.editForm.roleId}`, this.editForm)
            .then(response => {
              let res = response.data;
              console.log(res);
              if (res.meta.status !== 200)
                return this.$message({
                  message: "更新角色信息失败！",
                  type: "error",
                  showClose: true
                });
              this.$message({
                message: "更新角色信息成功！",
                type: "success",
                showClose: true
              });
              this.editFormDialog = false;
              this.getRolesList();
            });
        });
      },
      deleteRoleById(id) {
        this.$confirm("是否确认删除该角色？", "删除角色", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
          callback: action => {
            if (action !== "confirm") return;
            this.$axios.delete(`roles/${id}`).then(response => {
              let res = response.data;
              if (res.meta.status !== 200)
                return this.$message({
                  message: "删除角色失败！",
                  type: "error",
                  showClose: true
                });
              this.$message({
                message: "删除用户成功！",
                type: "success",
                showClose: true
              });
              this.getRolesList();
            });
          }
        });
      },
      // 根据Id删除对应的权限
      async removeRightById() {
        // 提示用户是否删除
        const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if(confirmResult !== 'confirm') {
          return this.$message.info('取消了删除!')
        }
      }
    }
  }
</script>

<style lang="less" scoped>
 .el-tag{margin:7px;}
 .bdtop{border-top:1px solid #eee;}
 .bdbottom{border-bottom:1px solid #eee;}
 .vcenter{display:flex;align-items:center;}
</style>
