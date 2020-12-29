<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-alert
        show-icon
        :closable="false"
        title="注意：只允许为第三级分类设置参数！"
        type="warning"
      >
      </el-alert>

      <!-- 显示商品选择区域 -->
      <el-row class="rowMargin">
        <span>商品选择：</span>

        <!-- 级联选择器区域 -->
        <el-cascader
          clearable
          expandTrigger="hover"
          v-model="cateList.cat_id"
          :options="cateList"
          :props="cateProps"
          @change="handleChange"
        ></el-cascader>
      </el-row>

      <!-- 标签页区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 控制动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            :disabled="isBtnDisaple"
            @click="addDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 动态参数的表格区域 -->
          <el-table :data="manyCateList" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  @close="tagClosed(scope.row, i)"
                  >{{ item }}</el-tag
                >
                <!-- 动态加载标签 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="editDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="delParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 控制静态属性面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            :disabled="isBtnDisaple"
            @click="addDialogVisible = true"
            >添加属性</el-button
          >
          <!-- 静态属性的表格区域 -->
          <el-table :data="onlyCateList" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  @close="tagClosed(scope.row, i)"
                  >{{ item }}</el-tag
                >
                <!-- 动态加载标签 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="editDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="delParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 添加对话框的表单区域 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 编辑对话框的表单区域 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //商品分类数据保存数组
      cateList: [],
      //级联选择器渲染配置
      cateProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      //选择器选中的值保存数组
      selectKeys: [],
      //当前活跃的标签Tab
      activeName: "many",
      //存储请求到的many数据
      manyCateList: [],
      //存储请求到的only数据
      onlyCateList: [],
      //控制添加对话框的显示与隐藏
      addDialogVisible: false,
      //添加对话框的表单的存储对象
      addForm: {
        attr_name: "",
      },
      //添加对话框的表单验证对象
      addFormRules: {
        attr_name: [{ required: true, message: "请输入名称", trigger: "blur" }],
      },
      //控制编辑对话框的显示与隐藏
      editDialogVisible: false,
      //编辑对话框表单的额存储对象
      editForm: {
        attr_name: "",
        attr_id: "",
      },
      // 编辑对话框的表单验证对象
      editFormRules: {
        attr_name: [
          { required: true, message: "请输入活动名称", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getParamsList();
  },

  methods: {
    async getParamsList() {
      const { data: res } = await this.$http.get("categories");
      // 将请求回的数据赋值给 cateList
      this.cateList = res.data;
      if (res.meta.status !== 200) {
        return this.$message.error("获取分类数据失败！");
      }
    },
    //监听选择器选择改变的事件
    handleChange() {
      this.selectKeys = this.cateList.cat_id;
      this.manyCateList = [];
      this.onlyCateList = [];
      this.getCateList();
    },
    //监听标签页的改变事件
    handleTabClick() {
      this.getCateList();
    },
    //请求分类数据列表
    async getCateList() {
      if (this.selectKeys.length !== 3) {
        return;
      }
      const { data: res } = await this.$http.get(
        `categories/${this.selectId}/attributes`,
        {
          params: { sel: this.activeName },
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("请求分类数据失败!");
      }
      //对请求得到的数据中的 attr_vals 进行以空格为分隔符的数组
      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : [];
        // 控制自己的动态Tag
        item.inputVisible = false;
        // 控制自己的动态Tag数据
        item.inputValue = "";
      });

      // 判断请求到的是many还是only的数据
      if (this.activeName === "many") {
        this.manyCateList = res.data;
      } else {
        this.onlyCateList = res.data;
      }
    },
    //添加对话框关闭事件处理函数
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    //添加对话框确认按钮的处理函数
    addParams() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          `categories/${this.selectId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加失败！");
        }
        // 关闭对话框
        this.addDialogVisible = false;
        // 提示用户添加成功
        this.$message.success("添加成功！");
        // 重新渲染列表
        this.getCateList();
      });
    },
    //编辑对话框的关闭事件处理函数
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    //点击按钮，弹出编辑对话框
    editDialog(edit) {
      this.editDialogVisible = true;
      this.editForm.attr_name = edit.attr_name;
      this.editForm.attr_id = edit.attr_id;
    },
    //点击按钮，确认修改编辑对话框内容
    editParams() {
      console.log(this.activeName);
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return;
        // 发起编辑提交参数请求
        const { data: res } = await this.$http.put(
          `categories/${this.selectId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("修改参数失败！");
        }
        this.$message.success("修改参数成功！");
        this.editDialogVisible = false;
        this.getCateList();
      });
    },
    //删除属性处理函数
    async delParams(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该属性, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);

      if (confirmResult !== "confirm") {
        // 用户取消了删除操作
        return this.$message.info("已取消删除操作!");
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.selectId}/attributes/${id}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败!");
      }
      this.$message.success("删除成功");
      this.getCateList();
    },
    //文本框失去焦点触发事件
    async handleInputConfirm(row) {
      // 判断输入内容是否为空,若为空,则重置
      if (row.inputValue.trim().length === 0) {
        row.inputValue = "";
        row.inputVisible = false;
        return;
      }
      // 若不为空,添加进attr_vals数组中
      row.attr_vals.push(row.inputValue.trim());
      row.inputValue = "";
      row.inputVisible = false;

      // 再发起请求,保存操作
      this.saveEditParams(row);
    },
    //点击按钮,获取文本框
    showInput(row) {
      row.inputVisible = true;
      // 自动获得焦点
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    //监听Tag标签的关闭事件
    async tagClosed(row, i) {
      const resultConfirm = await this.$confirm(
        "此操作将永久删除该参数, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);

      if (resultConfirm !== "confirm") {
        // 用户取消的删除操作
        return this.$message.info("已取消了删除操作!");
      }

      // 若用户确认删除,删除attr_vals中数组对应的项
      row.attr_vals.splice(i, 1);
      this.saveEditParams(row);
    },
    // 编辑提交参数处理函数
    async saveEditParams(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.selectId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: this.activeName,
          attr_vals: row.attr_vals.join(" "),
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("保存失败!");
      }
      this.$message.success("添加成功!");
    },
  },

  computed: {
    isBtnDisaple() {
      if (this.selectKeys.length !== 3) {
        return true;
      }
      return false;
    },
    // 选中分类的ID
    selectId() {
      if (this.selectKeys.length === 3) {
        return this.selectKeys[2];
      }
      return null;
    },
    // 添加对话框标题
    titleText() {
      if (this.activeName === "many") {
        return "动态参数";
      }
      return "静态属性";
    },
  },
};
</script>

<style lang="less" scoped>
.rowMargin {
  margin: 15px 0;
}

.el-tag {
  margin: 10px;
}

.input-new-tag {
  width: 120px;
}
</style>>
