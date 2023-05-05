<template>
  <div>
    <!-- 搜索栏 -->
    <el-card id="search" shadow="always">
      <el-row>
        <el-col :span="20">
          <el-input
            v-model="searchModel.username"
            @keyup.enter.native="getUserList"
            placeholder="用户名"
            clearable
          ></el-input>
          <el-input
            v-model="searchModel.phone"
            @keyup.enter.native="getUserList"
            placeholder="电话"
            clearable
          ></el-input>
          <el-button
            @click="getUserList"
            type="primary"
            icon="el-icon-search"
            round
            >查询</el-button
          >
        </el-col>
        <el-col :span="4" align="right">
          <el-button
            @click="openEditUI(null)"
            type="primary"
            icon="el-icon-plus"
            circle
          ></el-button>
        </el-col>
      </el-row>
    </el-card>

    <!-- 结果列表 -->
    <el-card>
      <el-table :data="userList" border style="width: 100%" :stripe="true">
        <el-table-column label="#" width="80">
          <template slot-scope="scope">
            {{
              (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1
            }}
          </template>
        </el-table-column>
        <el-table-column prop="id" label="用户ID" width="100"></el-table-column>
        <el-table-column prop="username" label="用户名" width="180">
        </el-table-column>
        <el-table-column prop="phone" label="电话" width="180">
        </el-table-column>
        <el-table-column prop="status" label="用户状态" width="100">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status == 1">正常</el-tag>
            <el-tag v-else type="danger">异常</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column label="操作" width="100">
          <template slot-scope="scope">
            <!-- <el-tooltip
              class="item"
              effect="dark"
              content="修改"
              placement="left"
            > -->
            <el-button
              @click="openEditUI(scope.row.id)"
              type="primary"
              icon="el-icon-edit"
              size="mini"
              circle
            ></el-button>
            <!-- </el-tooltip> -->

            <!-- <el-tooltip
              class="item"
              effect="dark"
              content="删除"
              placement="right"
            > -->
            <el-button
              @click="deleteUser(scope.row)"
              type="danger"
              icon="el-icon-delete"
              size="mini"
              circle
            ></el-button>
            <!-- </el-tooltip> -->
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="searchModel.pageNo"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="searchModel.pageSize"
      layout="  ->,total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>

    <!-- 用户信息编辑对话框 -->
    <el-dialog
      @close="clearForm"
      :title="title"
      :visible.sync="dialogFormVisible"
    >
      <el-form :model="userForm" ref="userFormRef" :rules="rules">
        <!-- 用户名 -->
        <el-form-item
          label="用户名"
          prop="username"
          :label-width="formLabelWidth"
          ><i class="el-icon-user-solid"></i>
          <el-input
            v-model="userForm.username"
            @keyup.enter.native="saveUser"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item
          v-if="userForm.id == null || userForm.id == undefined"
          label="登录密码"
          prop="password"
          :label-width="formLabelWidth"
          ><i class="el-icon-key"></i>
          <el-input
            type="password"
            v-model="userForm.password"
            autocomplete="off"
            show-password
          ></el-input>
        </el-form-item>

        <el-form-item
          label="联系电话"
          prop="phone"
          :label-width="formLabelWidth"
          ><i class="el-icon-phone"></i>
          <el-input v-model="userForm.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="用户状态" :label-width="formLabelWidth">
          <i class="el-icon-warning"></i>
          <el-switch
            v-model="userForm.status"
            :active-value="1"
            inactive-value="0"
          >
          </el-switch>
        </el-form-item>
        <el-form-item
          label="电子邮件"
          prop="email"
          :label-width="formLabelWidth"
          ><i class="el-icon-message"></i>
          <el-input
            v-model="userForm.email"
            @keyup.enter.native="saveUser"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveUser">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import userApi from "@/api/userManage";
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      var reg = /^([a-zA-Z\d][\w-]{2,})@(\w{2,})\.([a-z]{2,})(\.[a-z]{2,})?$/;
      if (!reg.test(value)) {
        return callback(new Error("邮箱格式错误"));
      }
      callback();
    };
    var checkPhone = (rule, value, callback) => {
      let reg = /^1[3|4|5|7|8][0-9]{9}$/;
      if (!reg.test(value)) {
        return callback(new Error("号码格式错误"));
      }
      callback();
    };
    return {
      formLabelWidth: "130px",
      userForm: {},
      dialogFormVisible: false,
      title: "",
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 5,
      },
      userList: [],
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          {
            min: 3,
            max: 20,
            message: "长度在 3 到 20 个字符",
            trigger: "blur",
          },
        ],
        password: [
          { required: true, message: "密码不能为空", trigger: "blur" },
          {
            min: 6,
            max: 20,
            message: "长度在 6 到 16 个字符",
            trigger: "blur",
          },
        ],
        phone: [
          { required: true, message: "请输入手机号码", trigger: "blur" },
          { validator: checkPhone, trigger: "blur" },
        ],
        email: [
          { required: true, message: "请输入电子邮件", trigger: "blur" },
          { validator: checkEmail, trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    deleteUser(user) {
      this.$confirm(`你确认删除用户 ${user.username} ?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          userApi.deleteUserById(user.id).then((response) => {
            this.$message({
              type: "success",
              message: response.message,
            });
            this.getUserList();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    saveUser() {
      // alert("1234");
      /* 先触发表单验证 */
      this.$refs.userFormRef.validate((valid) => {
        if (valid) {
          // 提交给后台
          userApi.saveUser(this.userForm).then((response) => {
            // 成功提示
            this.$message({
              message: response.message,
              type: "success",
            });
            // 关闭对话框
            this.dialogFormVisible = false;
            // 刷新表格
            this.getUserList();
          });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    clearForm() {
      this.userForm = {};
      this.$refs.userFormRef.clearValidate();
    },
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize;
      this.getUserList();
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo;
      this.getUserList();
    },
    getUserList() {
      userApi.getUserList(this.searchModel).then((response) => {
        this.userList = response.data.rows;
        this.total = response.data.total;
      });
    },
    openEditUI(id) {
      if (id == null) {
        this.title = "新增用户";
      } else {
        this.title = "修改用户";
        // 根据id查询用户数据
        userApi.getUserById(id).then((response) => {
          this.userForm = response.data;
        });
      }

      this.dialogFormVisible = true;
    },
  },
  created() {
    this.getUserList();
  },
};
</script>
<style>
#search .el-input {
  width: 200px;
  margin: 0 20px 0 0;
}

.el-dialog {
  width: 30%;
}

.el-dialog .el-input {
  width: 80%;
  padding-left: 10px;
  height: 37px;
}
.el-dialog .el-switch {
  margin-left: 10px;
}

.el-tag {
  margin-left: 20%;
}

/* .el-dialog .el-input {
  height: 37px;
} */
</style>