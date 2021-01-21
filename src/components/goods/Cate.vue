<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <br />
        <el-card>
            <el-row>
                <el-col :span="2">
                    <el-button type="primary" @click="showAddCate">添加分类</el-button>
                </el-col>
            </el-row>

            <el-table
                :data="cateList"
                :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
                row-key="cat_id"
                border
                stripe
            >
                <el-table-column type="index"></el-table-column>
                <el-table-column label="分类名称" prop="cat_name"></el-table-column>
                <el-table-column label="是否有效">
                    <template slot-scope="scope">
                        <i
                            class="el-icon-circle-check"
                            style="color:green;"
                            v-if="!scope.row.cat_deleted"
                        ></i>
                        <i class="el-icon-circle-close" v-else></i>
                    </template>
                </el-table-column>
                <el-table-column label="排序">
                    <template slot-scope="scope">
                        <el-tag type="primary" v-if="scope.row.cat_level===0">一级</el-tag>
                        <el-tag type="warning" v-else-if="scope.row.cat_level===1">二级</el-tag>
                        <el-tag type="danger" v-else>三级</el-tag>
                    </template>
                </el-table-column>

                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button
                            type="primary"
                            icon="el-icon-edit"
                            size="small"
                            @click="showEditCateDialog(scope.row.cat_id)"
                        >编辑</el-button>
                        <el-button
                            type="danger"
                            icon="el-icon-delete"
                            size="small"
                            @click="deleteCateById(scope.row.cat_id)"
                        >删除</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <el-pagination
                @size-change="sizeChange"
                @current-change="currentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[3,5,10,15]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
                background
            ></el-pagination>
        </el-card>

        <!-- 添加分类对话框 -->
        <el-dialog
            title="添加分类:"
            :visible.sync="addCateDialog"
            width="50%"
            @close="addCateDialogClosed"
        >
            <el-form
                :model="addCateForm"
                :rules="addCateFormRules"
                ref="addCateForm"
                label-width="110px"
            >
                <el-form-item label="分类名称:" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类:">
                    <el-cascader
                        :options="parentCateList"
                        v-model="selectedKeys"
                        @change="parentCateChange"
                        clearable
                        :props="{value:'cat_id',label:'cat_name',children:'children',checkStrictly:true}"
                    ></el-cascader>
                </el-form-item>
            </el-form>

            <span slot="footer">
                <el-button @click="addCateDialog = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改分类对话框 -->
        <el-dialog title="修改分类" :visible.sync="editCateDialog" width="50%">
            <el-form :model="editForm" ref="editFormRef" :rules="editFormRules" label-width="100px">
                <el-form-item label="分类名称：">
                    <el-input v-model="editForm.cat_name"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer">
                <el-button @click="editCateDialog = false">取 消</el-button>
                <el-button type="primary" @click="editCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
    data() {
        return {
            cateList: [],
            queryInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            total: 0,
            addCateDialog: false,
            addCateForm: {
                cat_pid: 0,
                cat_name: "",
                cat_level: 0
            },
            parentCateList: [],
            selectedKeys: [],
            addCateFormRules: {
                cat_name: [
                    {
                        required: true,
                        message: "请输入分类名称",
                        trigger: "blur"
                    },
                    {
                        max: 15,
                        message: "分类名称不超过15个字符",
                        trigger: "blur"
                    }
                ]
            },
            editFormRules: {
                cat_name: [
                    {
                        required: true,
                        message: "请输入分类名称",
                        trigger: "blur"
                    },
                    {
                        max: 15,
                        message: "分类名称不超过15个字符",
                        trigger: "blur"
                    }
                ]
            },
            editForm: {},
            editCateDialog: false
        };
    },
    created() {
        this.getCateList();
    },
    methods: {
        getCateList() {
            this.$axios
                .get("categories", { params: this.queryInfo })
                .then(response => {
                    let res = response.data;
                    if (res.meta.status !== 200)
                        return this.$message({
                            message: "获取商品分类失败！",
                            type: "error",
                            showClose: true
                        });
                    this.cateList = res.data.result;
                    this.total = res.data.total;
                });
        },
        sizeChange(newSize) {
            this.queryInfo.pagesize = newSize;
            this.getCateList();
        },
        currentChange(newPage) {
            this.queryInfo.pagenum = newPage;
            this.getCateList();
        },
        showAddCate() {
            this.getParentCateList();
            this.addCateDialog = true;
        },
        getParentCateList() {
            this.$axios
                .get("categories", { params: { type: 2 } })
                .then(response => {
                    let res = response.data;
                    if (res.meta.status !== 200)
                        return this.$message({
                            message: "获取数据失败！",
                            type: "error",
                            showClose: true
                        });

                    this.parentCateList = res.data;
                });
        },
        parentCateChange() {
            console.log(this.selectedKeys);
            if (this.selectedKeys.length > 0) {
                this.addCateForm.cat_pid = this.selectedKeys[
                    this.selectedKeys.length - 1
                ];
                this.addCateForm.cat_level = this.selectedKeys.length;
            } else {
                this.addCateForm.cat_pid = 0;
                this.addCateForm.cat_level = 0;
            }
        },
        addCate() {
            this.$refs.addCateForm.validate(valid => {
                if (!valid) return;
                this.$axios
                    .post("categories", this.addCateForm)
                    .then(response => {
                        let res = response.data;
                        if (res.meta.status !== 201)
                            return this.$message({
                                message: "添加分类失败！",
                                type: "error",
                                showClose: true
                            });
                        this.$message({
                            message: "添加分类成功",
                            type: "success",
                            showClose: true
                        });
                        this.getCateList();
                        this.addCateDialog = false;
                    });
            });
        },
        addCateDialogClosed() {
            this.$refs.addCateForm.resetFields();
            this.selectedKeys = [];
            this.addCateForm.cat_pid = 0;
            this.addCateForm.cat_level = 0;
        },
        deleteCateById(id) {
            this.$confirm("确定要删除该分类吗？", "删除分类", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning",
                callback: action => {
                    if (action !== "confirm") return;
                    this.$axios.delete(`categories/${id}`).then(response => {
                        let res = response.data;
                        if (res.meta.status !== 200)
                            return this.$message({
                                message: "删除分类失败！",
                                type: "error",
                                showClose: true
                            });
                        this.$message({
                            message: "删除分类成功！",
                            type: "success",
                            showClose: true
                        });
                        this.getCateList();
                    });
                }
            });
        },
        showEditCateDialog(id) {
            this.$axios.get(`categories/${id}`).then(response => {
                let res = response.data;
                if (res.meta.status !== 200)
                    return this.$message({
                        message: "获取分类失败！",
                        type: "error",
                        showClose: true
                    });
                this.editForm = res.data;
                this.editCateDialog = true;
            });
        },
        editCate() {
            this.$axios
                .put(`categories/${this.editForm.cat_id}`, this.editForm)
                .then(response => {
                    let res = response.data;
                    if (res.meta.status !== 200)
                        return this.$message({
                            message: "修改分类失败！",
                            type: "error",
                            showClose: true
                        });
                    this.$message({
                        message: "修改分类成功！",
                        type: "success",
                        showClose: true
                    });
                    this.editCateDialog = false;
                    this.getCateList();
                });
        }
    }
};
</script>
<style lang="less" scoped>
.el-cascader {
    width: 100%;
}
</style>