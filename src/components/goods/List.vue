<template>
<div>
  <!--面包屑导航区域-->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>商品列表</el-breadcrumb-item>
  </el-breadcrumb>

  <el-card>
    <el-row :gutter="20">
      <el-col :span="8">
        <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
          <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
        </el-input>
      </el-col>
      <el-col :span="4">
        <el-button type="primary" @click="goAddpage">添加商品</el-button>
      </el-col>
    </el-row>

    <el-table :data="goodsList" border stripe>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="商品名称" prop="goods_name"></el-table-column>
      <el-table-column label="商品价格(元)" prop="goods_price" width="100px"></el-table-column>
      <el-table-column label="商品重量" prop="goods_weight" width="90px"></el-table-column>
      <el-table-column label="创建时间" prop="add_time" width="140px">
        <template slot-scope="scope">
          {{scope.row.add_time | dateFormat}}
        </template>
      </el-table-column>
      <el-table-column label="操作" width="130px">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"></el-button>
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
</div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodsList: [],
      total: 0
    }
  },
  created() {
    this.getGoodsList()
  },
  methods:{
    getGoodsList() {
      this.$axios.get('goods', {params: this.queryInfo})
      .then(response => {
        let res = response.data;
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品列表失败！')
        }
        console.log(res)
        this.$message.success('获取商品列表成功！')
        this.goodsList = res.data.goods
        this.total = res.data.total
      })
    },
    goodsSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    goodsPageChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    removeById(id) {
      this.$confirm("是否确认删除该商品？", "删除商品", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
        callback: action => {
          if (action !== "confirm") return;
          this.$axios.delete(`goods/${id}`).then(response => {
            let res = response.data;
            if (res.meta.status !== 200)
              return this.$message({
                message: "删除商品失败！",
                type: "error",
                showClose: true
              });
            this.$message({
              message: "删除商品成功！",
              type: "success",
              showClose: true
            })
            this.getGoodsList()
          })
        }
      })
    },
    goAddpage() {
      this.$router.push('/goods/add')
    }
  }
}
</script>

<style lang="less" scoped>
</style>
