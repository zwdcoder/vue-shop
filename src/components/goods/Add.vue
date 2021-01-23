<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品添加</el-breadcrumb-item>
        </el-breadcrumb>
        <br />
        <el-card>
            <el-alert center :closable="false" title="添加商品信息" type="info" show-icon></el-alert>

            <!-- tab栏区域 -->
            <el-steps
                :active="activeIndex-0"
                finish-status="success"
                simple
                style="margin-top: 20px"
            >
                <el-step title="基本信息"></el-step>
                <el-step title="商品参数"></el-step>
                <el-step title="商品属性"></el-step>
                <el-step title="商品图片"></el-step>
                <el-step title="商品内容"></el-step>
                <el-step title="完成"></el-step>
            </el-steps>

            <el-form
                :model="addForm"
                ref="addFormRef"
                :rules="addFormRules"
                label-position="top"
                label-width="100px"
            >
                <el-tabs
                    :tab-position="'left'"
                    v-model="activeIndex"
                    :before-leave="beforeTabLeave"
                    @tab-click="tabClicked"
                >
                    <el-tab-pane label="基本信息" name="0">
                        <el-form-item label="商品名称" prop="goods_name">
                            <el-input v-model="addForm.goods_name"></el-input>
                        </el-form-item>
                        <el-form-item label="商品价格" prop="goods_price">
                            <el-input v-model="addForm.goods_price"></el-input>
                        </el-form-item>
                        <el-form-item label="商品重量" prop="goods_weight">
                            <el-input v-model="addForm.goods_weight"></el-input>
                        </el-form-item>
                        <el-form-item label="商品数量" prop="goods_number">
                            <el-input v-model="addForm.goods_number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品分类">
                            <el-cascader
                                :options="cateList"
                                :props="{value:'cat_id',label:'cat_name',children:'children'}"
                                v-model="addForm.goods_cat"
                                @change="handleChange"
                            ></el-cascader>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品参数" name="1">
                        <el-form-item
                            :label="item.attr_name"
                            v-for="item in manyTableData"
                            :key="item.attr_id"
                        >
                            <el-checkbox-group v-model="item.attr_vals">
                                <el-checkbox
                                    :label="cb"
                                    v-for="(cb,i) in item.attr_vals"
                                    border
                                    :key="i"
                                >{{cb}}</el-checkbox>
                            </el-checkbox-group>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品属性" name="2">
                        <el-form-item
                            :label="item.attr_name"
                            v-for="item in onlyTableData"
                            :key="item.attr_id"
                        >
                            <el-input v-model="item.attr_vals"></el-input>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品图片" name="3">
                        <el-upload
                            :action="upLoadURL"
                            :on-preview="handlePreview"
                            :on-remove="handleRemove"
                            list-type="picture"
                            :headers="headerObj"
                            :on-success="handleSuccess"
                        >
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-tab-pane>
                    <el-tab-pane label="商品内容" name="4">
                        <!-- 富文本 -->
                        <quill-editor
                            v-model="addForm.goods_introduce"
                        />
                        <br>
                        <el-button type="primary" @click="add">添加商品</el-button>

                    </el-tab-pane>
                </el-tabs>
            </el-form>
        </el-card>

        <!-- 图片预览 -->
        <el-dialog title="图片预览" :visible.sync="priviewDialog" width="50%">
            <img :src="priviewPath" alt />
        </el-dialog>
    </div>
</template>
<script>
export default {
    data() {
        return {
            activeIndex: "0",
            addForm: {
                goods_name: "",
                goods_number: "0",
                goods_price: "0",
                goods_weight: "0",
                goods_cat: [],
                pics: [],
                goods_introduce:'',
                attrs:[]
            },
            addFormRules: {
                goods_name: [
                    {
                        required: true,
                        message: "请输入商品名称",
                        trigger: "blur"
                    }
                ],
                goods_price: [
                    {
                        required: true,
                        message: "请输入商品价格",
                        trigger: "blur"
                    }
                ],
                goods_weight: [
                    {
                        required: true,
                        message: "请输入商品重量",
                        trigger: "blur"
                    }
                ],
                goods_number: [
                    {
                        required: true,
                        message: "请输入商品数量",
                        trigger: "blur"
                    }
                ],
                goods_cat: [
                    {
                        required: true,
                        message: "请输入商品数量",
                        trigger: "blur"
                    }
                ]
            },
            cateList: [],
            manyTableData: [],
            onlyTableData: [],
            upLoadURL: "http://119.23.53.78:8888/api/private/v1/upload",
            headerObj: {
                Authorization: sessionStorage.getItem("token")
            },
            priviewPath: "",
            priviewDialog: false
        };
    },
    methods: {
        getCateList() {
            this.$axios.get("categories").then(response => {
                let res = response.data;
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
            console.log(this.addForm.goods_cat);
            if (this.addForm.goods_cat.length !== 3) {
                this.addForm.goods_cat = [];
            }
        },
        beforeTabLeave(activeName, oldActiveName) {
            if (
                oldActiveName === "0" &&
                (this.addForm.goods_name === "" ||
                    this.addForm.goods_cat.length !== 3)
            ) {
                this.$message({
                    message: "请检查信息是否完整",
                    type: "error",
                    showClose: true
                });
                return false;
            }
        },
        tabClicked() {
            if (this.activeIndex === "1") {
                this.$axios
                    .get(`categories/${this.cateID}/attributes`, {
                        params: {
                            sel: "many"
                        }
                    })
                    .then(response => {
                        let res = response.data;
                        if (res.meta.status !== 200)
                            return this.$message({
                                message: "获取动态参数失败！",
                                type: "error",
                                showClose: true
                            });
                        res.data.forEach(item => {
                            item.attr_vals = item.attr_vals
                                ? item.attr_vals.split(",")
                                : [];
                        });
                        this.manyTableData = res.data;
                    });
            } else if (this.activeIndex === "2") {
                this.$axios
                    .get(`categories/${this.cateID}/attributes`, {
                        params: {
                            sel: "only"
                        }
                    })
                    .then(response => {
                        let res = response.data;
                        if (res.meta.status !== 200)
                            return this.$message({
                                message: "获取静态属性失败！",
                                type: "error",
                                showClose: true
                            });
                        console.log(res.data);
                        this.onlyTableData = res.data;
                    });
            }
        },
        handlePreview(file) {
            console.log(file);
            this.priviewPath =
                "http://119.23.53.78:8888/" + file.response.data.tmp_path;
            this.priviewDialog = true;
        },
        handleRemove(file) {
            console.log(file);
            const tempPath = file.response.data.tmp_path;
            const i = this.addForm.pics.findIndex(x => x.pic === tempPath);
            this.addForm.pics.splice(i, 1);
        },
        handleSuccess(res) {
            const picInfo = { pic: res.data.tmp_path };
            this.addForm.pics.push(picInfo);
        },
        add(){
            this.$refs.addFormRef.validate(valid=>{
                if(!valid)return this.$message({
                    message: '请检查信息是否完整！',
                    type: 'error',
                    showClose: true,
                });
                this.addForm.goods_cat=this.addForm.goods_cat.join(',')
                this.manyTableData.forEach(item=>{
                    const newInfo={
                        attr_id:item.attr_id,
                        attr_value:item.attr_vals.join(',')
                    }
                    this.addForm.attrs.push(newInfo)
                })
                this.onlyTableData.forEach(item=>{
                    const newInfo={
                        attr_id:item.attr_id,
                        attr_value:item.attr_vals
                    }
                    this.addForm.attrs.push(newInfo)
                })
                this.$axios.post('goods',this.addForm).then(response=>{
                    let res=response.data
                    if(res.meta.status!==201)
                        return this.$message({
                            message: '添加商品失败！',
                            type: 'error',
                            showClose: true,
                        });
                    this.$message({
                        message: '添加商品成功！',
                        type: 'success',
                        showClose: true,
                    });
                    this.$router.push('/goods')
                })
            })
        }
    },
    created() {
        this.getCateList();
    },
    computed: {
        cateID() {
            if (this.addForm.goods_cat.length === 3) {
                return this.addForm.goods_cat[2];
            }
            return null;
        }
    }
};
</script>
<style lang="less" scoped>
.el-checkbox {
    margin: 0 10px 0 0 !important ;
}
img {
    width: 100%;
}
</style>
