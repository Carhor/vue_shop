<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <!-- 添加分类区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- 商品分类表单区域 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        show-index
        index-text="#"
        border
        :selection-type="false"
        :show-row-hover="false"
        :expand-type="false"
        class="treeTable"
      >
        <!-- 是否有效模板区域 -->
        <template slot="isOk" slot-scope="scope">
          <i
            v-if="scope.row.cat_deleted === false"
            class="el-icon-success"
            style="color: lightgreen"
          ></i>
          <i v-else class="el-icon-error" style="color: red"></i>
        </template>

        <!-- 排序模板区域 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag
            v-else-if="scope.row.cat_level === 1"
            type="success"
            size="mini"
            >二级</el-tag
          >
          <el-tag v-else type="warning" size="mini">三级</el-tag>
        </template>

        <!-- 操作模板区域 -->
        <template slot="obt" slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="editCateDialog(scope.row)"
            >编辑</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="delCate(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 页码区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateFormClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateRules"
        ref="addCateRef"
        label-width="80px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            expand-trigger="hover"
            change-on-select="true"
            v-model="selectCateById"
            :options="parentList"
            :props="cascaderProp"
            clearable
            @change="selectChange"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
    >
      <el-form
        ref="editCateRef"
        :model="editCateForm"
        label-width="80px"
        :rules="editCateRules"
      >
        <el-form-item
          label="分类名称"
          prop="cat_name"
          placeholder="editCateForm.cat_name"
        >
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        // 每页显示的条数
        pagesize: 5,
      },
      // 数据总条数
      total: 0,
      //保存商品分类列表
      cateList: [],
      //为table指定列的定义
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        //自定义模板
        {
          label: "是否有效",
          prop: "cat_deleted",
          type: "template",
          template: "isOk",
        },
        {
          label: "排序",
          prop: "cat_level",
          type: "template",
          template: "order",
        },
        {
          label: "操作",
          type: "template",
          template: "obt",
        },
      ],
      //控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      //添加分类存储数据对象
      addCateForm: {
        cat_name: "",
        //添加分类的父ID
        cat_pid: 0,
        //添加分类的默认等级为一级
        cat_level: 0,
      },
      //添加分类表单验证规则对象
      addCateRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
          { min: 3, max: 8, message: "长度在 3 到 8 个字符", trigger: "blur" },
        ],
      },
      //储存父级分类数据的列表
      parentList: [],
      //配置级联选择器的渲染配置
      cascaderProp: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      //选择器的默认选定值
      selectCateById: [],
      //控制编辑对话框的显示与隐藏
      editCateDialogVisible: false,
      //编辑对话框表单存储的对象
      editCateForm: {
        cat_name: "",
        cat_id: "",
      },
      //编辑对话框表单的验证规则
      editCateRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
          { min: 3, max: 8, message: "长度在 3 到 8 个字符", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类列表失败！");
      }
      this.cateList = res.data.result;
      this.total = res.data.total;
    },
    //每页显示多少条数据
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    //当前处于哪一页
    handleCurrentChange(newNew) {
      this.queryInfo.pagenum = newNew;
      this.getCateList();
    },
    //添加分类按钮
    showAddCateDialog() {
      // 先获取父级分类数据列表
      this.getParentCateList();
      //再弹出对话框
      this.addCateDialogVisible = true;
    },
    //获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 },
      });
      if (res.meta.status !== 200) {
        this.$message.error("请求父级分类数据失败！");
      }
      this.parentList = res.data;
    },
    //监听选择器发生改变的事件
    selectChange() {
      if (this.selectCateById.length > 0) {
        //要添加分类的父分类ID
        this.addCateForm.cat_pid = this.selectCateById[
          this.selectCateById.length - 1
        ];
        //要添加分类的等级
        this.addCateForm.cat_level = this.selectCateById.length;
        return;
      }
    },
    //监听添加分类表单的关闭事件
    addCateFormClosed() {
      // 对表单进行重置
      this.$refs.addCateRef.resetFields();
      //重置添加分类存储数据对象
      this.addCateForm.cat_pid = 0;
      this.addCateForm.cat_level = 0;
      //重置选择器的默认选定值
      this.selectCateById = [];
    },
    //确认添加分类
    addCate() {
      // 对表单进行校验
      this.$refs.addCateRef.validate(async (valid) => {
        if (!valid) return;
        // 发起添加分类的请求
        const { data: res } = await this.$http.post("categories", {
          cat_pid: this.addCateForm.cat_pid,
          cat_name: this.addCateForm.cat_name,
          cat_level: this.addCateForm.cat_level,
        });
        if (res.meta.status !== 201) {
          this.$message.error("添加分类失败！");
        }
        this.$message.success("添加分类成功！");
        // 重新渲染列表
        this.getCateList();
        // 关闭对话框
        this.addCateDialogVisible = false;
      });
    },
    //监听编辑分类对话框按钮
    editCateDialog(cate) {
      this.editCateDialogVisible = true;
      this.editCateForm.cat_name = cate.cat_name;
      this.editCateForm.cat_id = cate.cat_id;
    },
    //监听编辑分类对话框的确认按钮
    async editCate() {
      // 进行表单预验证
      // this.$refs.editCateRef.validate(async (valid) => {
      //   if (!valid) return;
      //   // 进行编辑提交的网络请求;
      //   const { data: res } = await this.$http.put(
      //     `categories/${this.editCateForm.cat_id}/${this.editCateForm.cat_name}`
      //   );
      //   if (res.meta.status !== 200) {
      //     this.$message.error("更新分类失败！");
      //   }
      // });
      const { data: res } = await this.$http.put(
        `categories/${this.editCateForm.cat_id}`,
        {
          cat_name: this.editCateForm.cat_name,
        }
      );
      if (res.meta.status !== 200) {
        this.$message.error("更新分类失败！");
      }
      // 提示更新消息
      this.$message.success("更新成功！");
      // 重新渲染页面
      this.getCateList();
      //关闭编辑对话框
      this.editCateDialogVisible = false;
    },
    //监听删除分类按钮
    async delCate(id) {
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);

      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消了删除，则返回值为字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }
      // 发起删除分类的请求
      const { data: res } = await this.$http.delete(`categories/${id}`);
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败！");
      }
      this.$message.success(res.meta.msg);
      this.getCateList();
    },
  },
};
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>