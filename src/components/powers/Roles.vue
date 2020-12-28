<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRole">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
              :class="['bdBottom', i1 === 0 ? 'bdTop' : '', 'icenter']"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="[i2 === 0 ? '' : 'bdTop', 'icenter']"
                >
                  <!-- 渲染二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      closable
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              class="el-icon-edit"
              type="primary"
              size="mini"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              class="el-icon-delete"
              type="danger"
              size="mini"
              @click="showDelRole(scope.row.id)"
              >删除</el-button
            >
            <el-button
              class="el-icon-setting"
              type="warning"
              size="mini"
              @click="showSettingDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色按钮对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialog"
      width="50%"
      @close="addRoleClosed"
    >
      <el-form ref="addRoleRef" :model="addRoleForm" :rules="addRoleRules">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialog = false">取 消</el-button>
        <el-button type="primary" @click="saveAddRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色信息对话框 -->
    <el-dialog title="修改角色" :visible.sync="editRoleDialog" width="50%">
      <el-form :model="editRoleForm" :rules="editRoleRules" ref="editFormRef">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialog = false">取 消</el-button>
        <el-button type="primary" @click="saveEditRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="settingDialogVisible"
      width="50%"
      @close="settingDialogClosed()"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightList"
        :props="rightProps"
        ref="treeRef"
        default-expand-all
        show-checkbox
        node-key="id"
        :default-checked-keys="defaultKeys"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="settingDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveSetRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //角色列表存储数组
      rolesList: [],

      //权限列表存储数组
      rightList: [],

      //树形控件绑定属性
      rightProps: {
        children: "children",
        label: "authName",
      },

      //已有权限的id存储
      defaultKeys: [],

      //存储准备更改的角色id
      roleIdRight: "",

      //添加角色显示与隐藏对话框
      addRoleDialog: false,

      //编辑角色对话框的显示与隐藏
      editRoleDialog: false,

      //分配权限对话框的显示与隐藏
      settingDialogVisible: false,

      //添加角色表单存储对象
      addRoleForm: {
        roleName: "",
        roleDesc: "",
      },
      //编辑角色表单存储对象
      editRoleForm: {
        roleName: "",
        roleDesc: "",
      },
      //验证添加角色表单的规则对象
      addRoleRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          {
            min: 1,
            max: 5,
            message: "用户名的长度在1~5个字符之间",
            trigger: "blur",
          },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" },
          {
            min: 1,
            max: 5,
            message: "用户名的长度在1~5个字符之间",
            trigger: "blur",
          },
        ],
      },
      //验证修改角色表单的规则对象
      editRoleRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          {
            min: 1,
            max: 5,
            message: "用户名的长度在1~5个字符之间",
            trigger: "blur",
          },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" },
          {
            min: 1,
            max: 5,
            message: "用户名的长度在1~5个字符之间",
            trigger: "blur",
          },
        ],
      },
    };
  },

  created() {
    this.getRolesList();
  },

  methods: {
    // 获取角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles", this.addRoleForm);
      if (res.meta.status !== 200) {
        this.$message.error("请求角色列表失败！");
      }
      this.rolesList = res.data;
    },

    //监听添加角色按钮
    addRole() {
      this.addRoleDialog = true;
    },

    //监听添加角色对话框关闭
    addRoleClosed() {
      this.$refs.addRoleRef.resetFields();
    },

    //监听确认添加角色按钮
    saveAddRole() {
      this.$refs.addRoleRef.validate(async (valid) => {
        if (!valid) return;
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post("roles", this.addRoleForm);
        if (res.meta.status !== 201) {
          return this.$message.error("创建用户失败！");
        }
        // 关闭对话框
        this.addRoleDialog = false;
        // 重新请求角色列表数据
        this.getRolesList();
        this.$message.success("添加用户成功！");
      });
    },

    //监听编辑角色按钮
    async showEditDialog(id) {
      this.editRoleDialog = true;
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询角色失败！");
      }
      this.editRoleForm = res.data;
    },

    //监听编辑角色确认按钮
    async saveEditRole() {
      // 1.进行表单校验
      this.$refs.editFormRef.validate((valid) => {
        if (!valid) return;
      });
      // 2.进行编辑提交的网络请求
      const { data: res } = await this.$http.put(
        "roles/" + this.editRoleForm.roleId,
        {
          roleName: this.editRoleForm.roleName,
          roleDesc: this.editRoleForm.roleDesc,
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("角色提交失败！");
      }
      // 1.提示用户提交成功
      this.$message.success("角色提交成功！");
      // 2.重新请求角色信息
      this.getRolesList();
      // 3.关闭对话框
      this.editRoleDialog = false;
    },

    //监听删除角色按钮
    async showDelRole(id) {
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

      const { data: res } = await this.$http.delete("roles/" + id);

      if (res.meta.status !== 200) {
        return this.$message.error("删除用户失败！");
      }

      this.$message.success("删除用户成功！");
      this.getRolesList();
    },

    //根据用户id删除对应的权限
    async removeRightById(role, rightId) {
      //确认是否删除
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);

      if (confirmResult !== "confirm") {
        return this.$message.info("取消了删除操作！");
      }

      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );

      role.children = res.data;
      this.$message.success("删除成功！");
    },

    //通过递归方式把三级权限的id存储到defaultKey数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id);
      }

      node.children.forEach((item) => this.getLeafKeys(item, arr));
    },

    //点击分配权限的对话框
    async showSettingDialog(role) {
      this.roleIdRight = role.id;
      // 请求数据
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status !== 200) {
        this.$message.error("获取权限失败！");
      }
      this.rightList = res.data;
      //获取已有权限的ID
      this.getLeafKeys(role, this.defaultKeys);
      this.settingDialogVisible = true;
    },

    //监听分配权限对话框的关闭事件
    settingDialogClosed() {
      this.defaultKeys = [];
    },

    //保存已更改的权限
    async saveSetRight() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ];
      const idStr = keys.join(",");
      // 发起网络请求
      const { data: res } = await this.$http.post(
        `roles/${this.roleIdRight}/rights`,
        {
          rids: idStr,
        }
      );
      if (res.meta.status !== 200) {
        this.$message.error("更改权限失败！");
      }

      this.$message.success("更改权限成功！");
      this.getRolesList();
      this.settingDialogVisible = false;
    },
  },
};
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdTop {
  border-top: 1px solid #eee;
}

.bdBottom {
  border-bottom: 1px solid #eee;
}

.icenter {
  display: flex;
  align-items: center;
}
</style>