<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>
        <br />
        <el-card>
            <el-alert title="注意：只允许为第三级分类设置参数" type="warning" show-icon :closable="false"></el-alert>
            <el-row class="cat_opt">
                <span>选择商品分类：</span>
                <el-cascader
                    :options="cateList"
                    :props="{value:'cat_id',label:'cat_name',children:'children'}"
                    v-model="selectedCateKeys"
                    @change="handleChange"
                ></el-cascader>
            </el-row>

            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button
                        type="primary"
                        @click="addDialog=true"
                        size="small"
                        :disabled="isBtnDisabled"
                    >添加参数</el-button>
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag
                                    v-for="(item,i) in scope.row.attr_vals"
                                    :key="i"
                                    closable
                                    @close="handleClose(i,scope.row)"
                                >{{item}}</el-tag>
                                <!-- 输入文本框 -->
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                ></el-input>
                                <el-button
                                    v-else
                                    class="button-new-tag"
                                    size="small"
                                    @click="showInput(scope.row)"
                                >+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-row>
                                    <el-button
                                        type="primary"
                                        @click="showEditDialog(scope.row.attr_id)"
                                        icon="el-icon-edit"
                                        size="small"
                                    >修改</el-button>
                                    <el-button
                                        type="danger"
                                        @click="removeRarams(scope.row.attr_id)"
                                        icon="el-icon-delete"
                                        size="small"
                                    >删除</el-button>
                                </el-row>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button
                        type="primary"
                        @click="addDialog=true"
                        size="small"
                        :disabled="isBtnDisabled"
                    >添加属性</el-button>
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag
                                    v-for="(item,i) in scope.row.attr_vals"
                                    :key="i"
                                    closable
                                    @close="handleClose(i,scope.row)"
                                >{{item}}</el-tag>
                                <!-- 输入文本框 -->
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                ></el-input>
                                <el-button
                                    v-else
                                    class="button-new-tag"
                                    size="small"
                                    @click="showInput(scope.row)"
                                >+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-row>
                                    <el-button
                                        type="primary"
                                        icon="el-icon-edit"
                                        @click="showEditDialog(scope.row.attr_id)"
                                        size="small"
                                    >修改</el-button>
                                    <el-button
                                        type="danger"
                                        @click="removeRarams(scope.row.attr_id)"
                                        icon="el-icon-delete"
                                        size="small"
                                    >删除</el-button>
                                </el-row>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>

        <!-- 添加参数对话框 -->
        <el-dialog
            :title="'添加'+titleText"
            :visible.sync="addDialog"
            width="30%"
            @close="addDialogClosed"
        >
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name" required></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer">
                <el-button @click="addDialog = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 修改参数对话框 -->
        <el-dialog
            :title="'修改'+titleText"
            :visible.sync="editDialog"
            width="30%"
            @close="editDialogClosed"
        >
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name" required></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer">
                <el-button @click="editDialog = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
    data() {
        return {
            cateList: [],
            selectedCateKeys: [],
            activeName: "many",
            manyTableData: [],
            onlyTableData: [],
            addDialog: false,
            addForm: {
                attr_name: ""
            },
            addFormRules: {
                attr_name: [
                    { required: true, message: `请输入内容`, trigger: "blur" }
                ]
            },
            editForm: {},
            editFormRules: {
                attr_name: [
                    { required: true, message: `请输入内容`, trigger: "blur" }
                ]
            },
            editDialog: false
        };
    },
    methods: {
        getCateList() {
            this.$axios.get("categories").then(response => {
                let res = response.data;
                console.log(res);
                if (res.meta.status !== 200)
                    return this.$message({
                        message: "获取商品分类失败！",
                        type: "error",
                        showClose: true
                    });
                this.cateList = res.data;
            });
        },
        handleChange() {
            this.getParamsData();
        },
        handleTabClick() {
            this.getParamsData();
        },
        getParamsData() {
            if (this.selectedCateKeys.length !== 3) {
                this.selectedCateKeys = [];
                return;
            }
            this.$axios
                .get(`categories/${this.cateID}/attributes`, {
                    params: { sel: this.activeName }
                })
                .then(response => {
                    let res = response.data;
                    console.log(res);
                    if (res.meta.status !== 200)
                        return this.$message({
                            message: "获取参数列表失败！",
                            type: "error",
                            showClose: true
                        });
                    res.data.forEach(item => {
                        item.attr_vals = item.attr_vals
                            ? item.attr_vals.split(",")
                            : [];
                        item.inputVisible = false;
                        item.inputValue = "";
                    });
                    if (this.activeName === "many") {
                        this.manyTableData = res.data;
                    } else {
                        this.onlyTableData = res.data;
                    }
                });
        },
        addDialogClosed() {
            this.$refs.addFormRef.resetFields();
        },
        addParams() {
            this.$refs.addFormRef.validate(valid => {
                if (!valid) return;
                this.$axios
                    .post(`categories/${this.cateID}/attributes`, {
                        attr_name: this.addForm.attr_name,
                        attr_sel: this.activeName
                    })
                    .then(response => {
                        let res = response.data;
                        if (res.meta.status !== 201)
                            return this.$message({
                                message: "添加参数失败！",
                                type: "error",
                                showClose: true
                            });
                        this.$message({
                            message: "添加参数成功！",
                            type: "success",
                            showClose: true
                        });
                        this.addDialog = false;
                        this.getParamsData();
                    });
            });
        },
        showEditDialog(id) {
            console.log(this.cateID);
            this.$axios
                .get(`categories/${this.cateID}/attributes/${id}`, {
                    params: {
                        attr_sel: this.activeName
                    }
                })
                .then(response => {
                    let res = response.data;
                    if (res.meta.status !== 200)
                        return this.$message({
                            message: "获取失败！",
                            type: "error",
                            showClose: true
                        });
                    this.editForm = res.data;
                });
            this.editDialog = true;
        },
        editDialogClosed() {
            this.$refs.editFormRef.resetFields();
        },
        editParams() {
            this.$refs.editFormRef.validate(valid => {
                if (!valid) return;
                this.$axios
                    .put(
                        `categories/${this.cateID}/attributes/${this.editForm.attr_id}`,
                        {
                            attr_name: this.editForm.attr_name,
                            attr_sel: this.activeName
                        }
                    )
                    .then(response => {
                        let res = response.data;
                        if (res.meta.status !== 200)
                            return this.$message({
                                message: "修改参数失败！" + res.meta.msg,
                                type: "error",
                                showClose: true
                            });
                        this.$message({
                            message: "修改参数成功！",
                            type: "success",
                            showClose: true
                        });

                        this.getParamsData();
                        this.editDialog = false;
                    });
            });
        },
        removeRarams(id) {
            this.$confirm("是否确认删除该参数？", "删除参数", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning",
                callback: action => {
                    if (action !== "confirm") return;
                    this.$axios
                        .delete(`categories/${this.cateID}/attributes/${id}`)
                        .then(response => {
                            let res = response.data;
                            if (res.meta.status !== 200)
                                return this.$message({
                                    message: "删除参数失败！",
                                    type: "error",
                                    showClose: true
                                });
                            this.$message({
                                message: "删除参数成功！",
                                type: "success",
                                showClose: true
                            });
                            this.getParamsData();
                        });
                }
            });
        },
        handleInputConfirm(row) {
            if (row.inputValue.trim().length === 0) {
                row.inputValue = "";
                row.inputVisible = false;
                return;
            }
            row.attr_vals.push(row.inputValue);
            row.inputValue = "";
            row.inputVisible = false;
            this.saveAttrValues(row)
        },
        saveAttrValues(row){
          this.$axios
            .put(`categories/${this.cateID}/attributes/${row.attr_id}`, {
              attr_name: row.attr_name,
              attr_sel: row.attr_sel,
              attr_vals: row.attr_vals.join(",")
            })
            .then(response => {
              let res = response.data;
              if (res.meta.status !== 200)
                return this.$message({
                    message: "修改参数信息失败！",
                    type: "error",
                    showClose: true
                });
              this.$message({
                message: "修改参数信息成功！",
                type: "success",
                showClose: true
              });
            });
        },
        showInput(row) {
            row.inputVisible = true;
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus();
            });
        },
        handleClose(i,row){
            row.attr_vals.splice(i,1)
            this.saveAttrValues(row)
        }
    },
    created() {
        this.getCateList();
    },
    computed: {
        isBtnDisabled() {
            if (this.selectedCateKeys.length !== 3) return true;
            return false;
        },
        cateID() {
            if (this.selectedCateKeys.length === 3)
                return this.selectedCateKeys[2];
            return null;
        },
        titleText() {
            if (this.activeName === "many") return "动态参数";
            return "静态属性";
        }
    }
};
</script>
<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}
.el-tag {
    margin: 10px;
}
.input-new-tag {
    width: 120px;
}
</style>