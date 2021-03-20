<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
              :gutter="10"
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag>{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightsById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      :key="item3.id"
                      v-for="(item3, i3) in item2.children"
                      type="warning"
                      closable
                      @close="removeRightsById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRoleById(scope.row.id)"
              >删除</el-button
            >

            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form :model="addForm" ref="addFormRef" label-width="70px">
        <el-form-item label="角色id" prop="roleId">
          <el-input v-model="addForm.roleId"></el-input>
        </el-form-item>
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色对话框 -->
    <el-dialog
      title="角色信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form ref="editRoleRef" :model="editForm" label-width="80px">
        <el-form-item label="角色ID">
          <el-input v-model="editForm.roleId" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色名称">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      rolesList: [],
      //是否显示修改角色对话框
      editDialogVisible: false,
      //控制显示与隐藏添加角色对话框
      addDialogVisible: false,
      //添加角色的表单数据
      addForm: {
        roleId: "",
        roleName: "",
        roleDesc: "",
      },
      //查询到的角色信息
      editForm: [],
      //控制分配权限的对话框显示与隐藏
      setRightDialogVisible: false,
      //获取到的权限数据
      rightslist: [],
      //树形控件的属性绑定对象
      treeProps: {
        label: "authName",
        children: "children",
      },
      //默认选中的Id值数组
      defKeys: [],
      //当前即将分配权限的角色Id
      roleId: "",
    };
  },
  created() {
    //获取所有角色列表
    this.getRolesList();
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      console.log(res);
      if (res.meta.status != 200) {
        return this.$message.error("获取角色列表失败！");
      }
      this.rolesList = res.data;
    },
    //监听添加用户对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    //点击确定，添加角色
    addUser() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error("添加角色失败！");
        }
        const { data: res } = await this.$http.post("roles", {
          roleName: this.addForm.roleName,
          roleDesc: this.addForm.roleDesc,
        });
        if (res.meta.status != 201) {
          return this.$message.error("添加失败！");
        }
        //隐藏添加用户对话框
        this.addDialogVisible = false;
        //重新获取用户列表
        this.getRolesList();
        //通知
        this.$message.success("添加成功！");
      });
    },
    //点击按钮，显示修改角色对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status != 200) {
        return this.$message.error("获取角色信息失败！");
      }
      console.log(res);
      this.editForm = res.data;
      this.editDialogVisible = true;
    },
    //监听修改用户对话框的关闭事件
    editDialogClosed() {
      this.$refs.editRoleRef.resetFields();
    },
    //修改角色信息并提交
    editRoleInfo() {
      this.$refs.editRoleRef.validate(async (valid) => {
        if (!valid) return;
        //发起修改角色信息的请求
        const { data: res } = await this.$http.put(
          "roles/" + this.editForm.id,
          {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc,
          }
        );
        // console.log(this.editForm);
        if (res.meta.status != 200 && res.meta.status != 400) {
          console.log(res.meta.status);
          return this.$message.error("获取失败！");
        }
        //隐藏对话框
        this.editDialogVisible = false;
        //更新用户列表
        this.getRolesList();
        //提示更新用户信息成功
        this.$message.success("更新角色信息成功！");
      });
    },
    //根据id删除角色
    async removeRoleById(id) {
      //弹框询问是否删除角色
      const confirmResult = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      //如果用户确认了删除，confirmResult为字符串confirm
      //如果用户取消了删除，confirmResult为字符串cancel
      if (confirmResult != "confirm") {
        return this.$message.info("已经取消了删除！");
      }
      const { data: res } = await this.$http.delete("roles/" + id);
      if (res.meta.status != 200) {
        return this.$$message.error("删除角色失败！");
      }
      this.$message.success("删除角色成功！");
      //刷新角色数据列表
      this.getRolesList();
    },
    //根据id删除对应的权限
    async removeRightsById(role, rightId) {
      //弹框提示用户删除权限
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult != "confirm") {
        return this.$message.info("取消了删除！");
      }

      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );

      if (res.meta.status != 200) {
        return this.$message.error("删除权限失败！");
      }

      // this.getRolesList(); 会重新渲染页面
      role.children = res.data;
    },
    //展示分配权限的对话框
    async showSetRightDialog(role) {
      //将角色信息保存到 data 中
      this.roleId = role.id;
      //获取所有权限的数据
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status != 200) {
        return this.$message.error("获取所有权限数据失败！");
      }
      //把获取到的权限数据保存到data中
      this.rightslist = res.data;
      // console.log(this.rightslist);

      //递归调用三级节点id
      this.getLeafKeys(role, this.defKeys);

      this.setRightDialogVisible = true;
    },
    //通过递归的形式，获取角色下所有三级权限的ID，并保存到 defKeys数组中
    getLeafKeys(node, arr) {
      //如果 node 节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id);
      }

      node.children.forEach((item) => this.getLeafKeys(item, arr));
    },
    //监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = [];
    },
    //点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ];

      const idStr = keys.join(",");

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      );

      if (res.meta.status != 200) {
        return this.$message.error("分配权限失败！");
      }
      this.$message.success("分配权限成功！");
      this.getRolesList();
      this.setRightDialogVisible = false;
    },
  },
};
</script>

<style scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>