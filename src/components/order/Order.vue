<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <!-- 搜索框区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="orderList" border stripe style="width: 100%">
        <!-- 索引列 -->
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="order_number" label="订单编号">
        </el-table-column>
        <el-table-column prop="order_price" label="订单价格" width="80px">
        </el-table-column>
        <el-table-column label="是否付款" width="80px">
          <template slot-scope="scope">
            <el-tag type="danger" v-if="scope.row.pay_status === '0'"
              >未付款</el-tag
            >
            <el-tag type="success" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货" width="80px">
        </el-table-column>
        <el-table-column label="下单时间" width="200px">
          <template slot-scope="scope">
            {{ scope.row.update_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="150px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEdit"
            ></el-button>
            <el-button
              size="mini"
              type="success"
              icon="el-icon-setting"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 页码区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="this.queryInfo.pagenum"
        :page-sizes="[10, 50, 80, 100]"
        :page-size="this.queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="this.total"
        background
      >
      </el-pagination>
    </el-card>

    <!-- 修改地址对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 修改地址表单区域 -->
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            v-model="addressForm.address1"
            :options="cityData"
            :props="{ expandTrigger: 'hover' }"
            clearable
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editDialogVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 物流对话框 -->
    <el-dialog title="物流信息" :visible.sync="progressVisible" width="50%">
      <span>这是一段信息</span>
    </el-dialog>
  </div>
</template>

<script>
// 导入城市数据
import cityData from "./citydata.js";

export default {
  data() {
    return {
      // 查询参数
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      // 总数据条数
      total: 0,
      // 订单列表数据存储数组
      orderList: [],
      //控制修改地址对话框的显示与隐藏
      editDialogVisible: false,
      //  修改地址表单存储
      addressForm: {
        address1: [],
        address2: "",
      },
      addressFormRules: {
        address1: [{ required: true, message: "请选择", trigger: "blur" }],
        address2: [
          { required: true, message: "请填写详细地址", trigger: "blur" },
        ],
      },
      // 导入城市数据
      cityData,
      //控制物流对话框的显示与隐藏
      progressVisible: false,
      // 物流信息保存的数组
      progressList: [],
    };
  },

  created() {
    this.getOrderList();
  },

  methods: {
    // 请求订单数据
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取订单列表失败！");
      }
      this.$message.success("获取订单列表成功！");
      this.total = res.data.total;
      this.orderList = res.data.goods;
      console.log(this.orderList);
    },
    //控制一个页码展示多少条数据
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getOrderList();
    },
    // 控制当前页面处于哪一页
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getOrderList();
    },
    // 显示修改地址对话框
    showEdit() {
      this.editDialogVisible = true;
    },
    //监听修改地址对话框的关闭事件
    editDialogClosed() {
      this.$refs.addressFormRef.resetFields();
    },
    //展示物流对话框
    showProgressBox() {
      // const res = this.$http.get("/kuaidi/804909574412544580");
      // const { data: res } = await this.$http.get("/kuaidi/804909574412544580");
      // if (res.meta.status !== 200) {
      //   return this.$message.error("获取物流信息失败！");
      // }
      // this.$message.success("获取物流信息成功！");
      // this.progressList = res.data;
      this.progressVisible = true;
      // console.log(this.progressList);
    },
  },
};
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>>