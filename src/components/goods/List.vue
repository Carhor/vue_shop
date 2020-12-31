<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <!-- 搜索区域 -->
        <el-col :span="8">
          <el-input
            v-model="queryInfo.query"
            placeholder="请输入内容"
            class="input-with-select"
            width="300px"
            clearable
            @clear="getGoodsList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getGoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- 商品列表表单区域 -->
      <el-table :data="goodsList" border stripe>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>

        <el-table-column prop="goods_name" label="商品名称"> </el-table-column>
        <el-table-column prop="goods_price" label="商品价格" width="120px">
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="120px">
        </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="150px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="120px">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="delGoods(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 页码区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[10, 100, 200, 300]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>

    <!-- 编辑商品对话框 -->
    <!-- <el-dialog
      title="修改商品属性"
      :visible.sync="editDialogVisible"
      width="50%"
    > -->
    <!-- 编辑商品表单区域 -->
    <!-- <el-form
        :model="editForm"
        :rules="editRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="editForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
          <el-input v-model="editForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="editForm.goods_weight"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEditForm">确 定</el-button>
      </span>
    </el-dialog> -->
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 搜索框中的值双向绑定
      searchInput: "",
      // 查询参数
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      // 保存请求得到的商品列表数据
      goodsList: [],
      // 数据总条数
      total: 0,
      // 控制编辑对话框的显示与隐藏
      editDialogVisible: false,
      // 编辑商品表单数据存储对象
      editForm: {
        goods_name: "",
        goods_price: "",
        goods_weight: "",
        goods_id: "",
        goods_number: "",
      },
      // 编辑商品表单验证规则数组
      editRules: {
        goods_name: [
          { required: true, message: "请输入商品名称", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "请输入商品价格", trigger: "blur" },
        ],
        goods_weight: [
          { required: true, message: "请输入商品重量", trigger: "blur" },
        ],
      },
    };
  },

  created() {
    this.getGoodsList();
  },

  methods: {
    // 获取商品列表数据
    async getGoodsList() {
      const { data: res } = await this.$http.get("goods", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品列表数据失败!");
      }
      this.$message.success("获取商品列表数据成功！");
      this.goodsList = res.data.goods;
      this.total = res.data.total;
    },
    //控制每页展示多少条数据
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },

    // 监听当前处于哪一页
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodsList();
    },

    //展示编辑商品对话框处理函数
    // showEditDialog(row) {
    //   this.editForm.goods_name = row.goods_name;
    //   this.editForm.goods_price = row.goods_price;
    //   this.editForm.goods_weight = row.goods_weight;
    //   this.editForm.goods_id = row.goods_id;
    //   this.editForm.goods_number = row.goods_number;

    //   this.editDialogVisible = true;
    // },

    //监听编辑对话框的关闭事件
    // editDialogClosed() {
    //   this.$refs.editFormRef.resetFields();
    // },

    // 编辑商品对话框确认按钮的处理函数
    // saveEditForm() {
    //   this.$refs.editFormRef.validate(async (valid) => {
    //     if (!valid) return;
    //     const { data: res } = this.$http.put(
    //       `goods/${this.editForm.goods_id}`,
    //       {
    //         params: {
    //           goods_name: this.editForm.goods_name,
    //           goods_price: this.editForm.goods_price,
    //           goods_number: this.editForm.goods_number,
    //           goods_weight: this.editForm.goods_weight,
    //         },
    //       }
    //     );
    //     if (res.meta.status !== 201) {
    //       return this.$message.error("提交商品数据失败！");
    //     }
    //     this.$message.success("提交商品数据成功！");
    //     this.getGoodsList();
    //     this.editDialogVisible = false;
    //   });
    // },

    //删除商品
    async delGoods(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该商品, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);

      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除操作！");
      }
      // 没有取消则发起删除操作的请求
      const { data: res } = await this.$http.delete("goods/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败！");
      }
      this.$message.success("删除成功！");
      this.getGoodsList();
    },

    //通过路由挑战到添加商品页面
    goAddpage() {
      this.$router.push("/good/add");
    },
  },
};
</script>

<style lang="less" scoped>
</style>