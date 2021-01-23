<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>订单管理</el-breadcrumb-item>
            <el-breadcrumb-item>订单列表</el-breadcrumb-item>
        </el-breadcrumb>
        <br />
        <el-card>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input
                        placeholder="请输入内容"
                        clearable
                        @clear="searchOpt"
                        v-model="queryInfo.query"
                    >
                        <el-button slot="append" icon="el-icon-search" @click="searchOpt"></el-button>
                    </el-input>
                </el-col>
            </el-row>
            <el-table :data="orderList" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="订单编号" prop="order_number"></el-table-column>
                <el-table-column label="订单价格" prop="order_price"></el-table-column>
                <el-table-column label="是否付款" prop="pay_status">
                    <template slot-scope="scope">
                        <el-tag type="success" :closable="false" v-if="scope.row.pay_status!==0">已付款</el-tag>
                        <el-tag type="danger" :closable="false" v-else>未付款</el-tag>
                    </template>
                </el-table-column>
                <el-table-column label="是否发货" prop="is_send"></el-table-column>
                <el-table-column label="下单时间">
                    <template slot-scope="scope">{{scope.row.create_time|dateFormat}}</template>
                </el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-tooltip placement="top" content="修改地址" effect="dark">
                            <el-button
                                type="primary"
                                @click="editAddress"
                                size="small"
                                icon="el-icon-edit"
                            ></el-button>
                        </el-tooltip>

                        <el-tooltip content="物流信息" placement="top" effect="dark">
                            <el-button
                                type="success"
                                @click="showProgress"
                                size="small"
                                icon="el-icon-location"
                            ></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <el-pagination
                @size-change="goodsSizeChange"
                @current-change="goodsPageChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[5,10,15,20]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
                background
            ></el-pagination>
        </el-card>

				<!--修改地址对话框-->
				<el-dialog title="修改地址" :visible.sync="addressVisible" width="50%" @close="addressDialogClosed">
				  <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
						<el-form-item label="省市区/县" prop="address1">
							<el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
						</el-form-item>
						<el-form-item label="详细地址" prop="address2">
							<el-input v-model="addressForm.address2"></el-input>
						</el-form-item>
					</el-form>
				</el-dialog>

        <!-- 物流信息对话框 -->
        <el-dialog title="物流进度" :visible.sync="progressDialog" width="50%">
            <el-timeline>
                <el-timeline-item
                    v-for="(activity, index) in progressInfo"
                    :key="index"
                    :timestamp="activity.time"
                >{{activity.context}}</el-timeline-item>
            </el-timeline>
        </el-dialog>
    </div>
</template>
<script>
import cityData from './citydata.js'
export default {
    data() {
        return {
            queryInfo: {
                query: "",
                pagenum: 1,
                pagesize: 10
            },
						addressVisible: false,
            orderList: [],
            total: 0,
            progressDialog: false,
            progressInfo: [],
						addressForm: {
							address1: [],
							address2: ''
						},
						addressFormRules: {
							address1: [
								{required: true, message: '请选择省市区县', trigger: 'blur'}
							],
							address2: [
								{required: true, message: '请填写详细地址', trigger: 'blur'}
							]
						},
            cityData: cityData
        };
    },
    created() {
        this.getOrderList();
    },
    methods: {
        getOrderList() {
            this.$axios
                .get("orders", { params: this.queryInfo })
                .then(response => {
                    let res = response.data;
                    console.log(res);
                    if (res.meta.status !== 200)
                        return this.$message({
                            message: "获取订单列表失败！",
                            type: "error",
                            showClose: true
                        });
                    this.orderList = res.data.goods;
                    this.total = res.data.total;
                });
        },
        addressDialogClosed() {
          this.$refs.addressFormRef.resetFields()
        },
        searchOpt() {
            this.queryInfo.pagenum = 1;
            this.getOrderList();
        },
        goodsSizeChange(newSize) {
            this.queryInfo.pagesize = newSize;
            this.getOrderList();
        },
        goodsPageChange(newPage) {
            this.queryInfo.pagenum = newPage;
            this.getOrderList();
        },
        editAddress() {
            this.addressVisible = true
        },
        showProgress() {
            this.progressDialog = true;
            this.$axios.get("/kuaidi/801609574412544580").then(response => {
                let res = response.data;
                if (res.meta.status !== 200)
                    return this.$message({
                        message: "获取物流进度失败！",
                        type: "error",
                        showClose: true
                    });
                this.progressInfo = res.data;
                console.log(this.progressInfo);
            });
        }
    }
};
</script>
<style lang="less" scoped>
  .el-cascader{width: 100%;}
</style>
