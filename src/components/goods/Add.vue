<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      ></el-alert>

      <!-- 步骤条区域 -->
      <el-steps :active="activeIndex - 0" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- 表单区域 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-position="top"
      >
        <!-- 标签页区域 -->
        <el-tabs tab-position="left" @tab-click="handleTabClick">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input type="number" v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input type="number" v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input type="number" v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <!-- 级联选择器 -->
              <el-cascader
                expandTrigger="hover"
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="cateListProps"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>

          <el-tab-pane label="商品参数" name="1" :disabled="tabDisabled">
            <!-- 渲染表单的item项 -->
            <el-form-item v-for="item in manyTableData" :key="item.attr_id">
              {{ item.attr_name }}
              <!-- 渲染item的attr_vals项 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="item2"
                  v-for="(item2, i) in item.attr_vals"
                  :key="i"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>

          <el-tab-pane label="商品属性" name="2" :disabled="tabDisabled">
            <el-form-item v-for="item in onlyTableData" :key="item.attr_id">
              {{ item.attr_name }}
              <el-input
                v-for="(item, i) in item.attr_vals"
                :key="i"
                :value="item"
              ></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3" :disabled="tabDisabled">
            <el-upload
              :action="uploadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headersObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4" :disabled="tabDisabled">
            <!-- 富文本编辑器区域 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <!-- 添加按钮区域 -->
            <el-button type="primary" class="btnAdd" @click="add"
              >添加商品</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 预览上传图片的对话框 -->
    <el-dialog
      title="图片预览"
      :visible.sync="previewDialogVisible"
      width="50%"
    >
      <img :src="previewURL" alt="" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 步骤条与标签页对应的参数
      activeIndex: 0,
      //添加商品绑定的存储对象
      addForm: {
        goods_name: "",
        goods_price: "",
        goods_weight: "0",
        goods_number: "1",
        goods_cat: [0],
        pics: [],
        goods_introduce: "",
        attrs: [],
      },
      //添加商品的验证规则
      addFormRules: {
        goods_name: [
          { required: true, message: "请输入商品名称", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "请输入商品价格", trigger: "blur" },
        ],
        goods_weight: [
          { required: true, message: "请输入商品价格", trigger: "blur" },
        ],
        goods_number: [
          { required: true, message: "请输入商品价格", trigger: "blur" },
        ],
      },

      //商品分类数据的存储对象
      cateList: [],
      //商品分类的配置对象
      cateListProps: {
        label: "cat_name",
        value: "cat_id",
        children: "children",
      },
      // 级联选择器选择的分类ID值
      selectID: "",

      //未选择分类属性，标签页为禁用
      tabDisabled: true,

      //通过 many 请求的商品列表数据
      manyTableData: [],
      //静态属性数据列表
      onlyTableData: [],
      // 上传图片时的URL地址
      uploadURL: "http://127.0.0.1:8888/api/private/v1/upload",
      // 图片上传时组件的 Headers 请求头对象
      headersObj: {
        Authorization: window.sessionStorage.getItem("token"),
      },
      //存放所中图片的URL
      previewURL: "",
      //控制预览图片对话框的显示与隐藏
      previewDialogVisible: false,
    };
  },

  created() {
    this.getCateList();
  },

  methods: {
    // 切换Tab按钮选中时触发的处理函数
    async handleTabClick(tab) {
      this.activeIndex = tab.name;

      if (this.activeIndex === "1") {
        // 发起该分类的网络请求
        const { data: res } = await this.$http.get(
          `categories/${this.selectID}/attributes`,
          {
            params: { sel: "many" },
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("获取参数列表失败！");
        }
        this.$message.success("获取商品参数成功！");
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(" ");
        });
        this.manyTableData = res.data;
      } else if (this.activeIndex === "2") {
        // 发起该分类的网络请求
        const { data: res } = await this.$http.get(
          `categories/${this.selectID}/attributes`,
          {
            params: { sel: "only" },
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("获取静态属性列表失败！");
        }
        this.$message.success("获取静态属性列表成功！");
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(" ");
        });
        this.onlyTableData = res.data;
      }
    },

    //获取商品分类的数据列表
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("请求商品分类数据失败！");
      }
      this.$message.success("请求商品分类数据成功！");
      this.cateList = res.data;
    },

    //级联选择器改变选择时触发的处理函数
    async handleChange() {
      if (this.addForm.goods_cat.length === 3) {
        this.selectID = this.addForm.goods_cat[2];
        this.tabDisabled = false;
      } else {
        this.addForm.goods_cat = [];
        return (this.tabDisabled = true);
      }
    },

    //点击文件列表中已上传的文件时的钩子
    handlePreview(file) {
      this.previewURL = file.response.data.url;
      this.previewDialogVisible = true;
    },

    //文件列表移除文件时的钩子
    handleRemove(file) {
      // 1.找出要删除的文件的 tmp_path
      const filePath = file.response.data.tmp_path;
      // 2/根据文件的 tmp_path 找出pics数组中的索引值
      const i = this.addForm.pics.findIndex((x) => x.pic === filePath);
      // 3.根据索引值把对应的pic项从pics中删除
      this.addForm.pics.splice(i, 1);
    },

    //文件上传成功时的钩子
    handleSuccess(response) {
      // 1.拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path };

      // 2.将图片信息对象push到商品绑定的存储对象的 pics 数组中
      this.addForm.pics.push(picInfo);
    },

    //监听添加商品按钮
    add() {
      // 先验证表单是否填完完毕
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error("请填写完所需表单项！");

        // 对商品分类的数据处理
        this.addForm.goods_cat = this.addForm.goods_cat.join(",");
        // 处理动态参数
        this.manyTableData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(" "),
          };
          this.addForm.attrs.push(newInfo);
        });

        //处理静态参数
        this.onlyTableData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals,
          };
          this.addForm.attrs.push(newInfo);
        });
        console.log(this.addForm);
        // 发起添加商品的请求
        // 商品名称要唯一
        const { data: res } = await this.$http.post("goods", this.addForm);
        if (res.meta.status !== 201) {
          return this.$message.error("添加商品失败！");
        }
        this.$message.success("添加商品成功");
        this.$router.push("/goods");
      });
    },
  },
};
</script>

<style lang="less" scoped>
.el-steps {
  margin-top: 15px;
}

.el-checkbox {
  margin: 0 10px 5px 0 !important;
}

.previewImg {
  width: 100%;
}

.btnAdd {
  margin-top: 15px;
}
</style>>
