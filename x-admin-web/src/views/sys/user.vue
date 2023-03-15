<template>
  <div>
    <!-- 検索 -->
    <el-card id="search">
      <el-row>
        <el-col :span="20">
          <el-input
            v-model="searchModel.username"
            placeholder="ユーザ名"
            clearable
          ></el-input>
          <el-input
            v-model="searchModel.phone"
            placeholder="電話番号"
            clearable
          ></el-input>
          <el-button
            @click="getUserList"
            type="primary"
            round
            icon="el-icon-search"
            >検索</el-button
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

    <!-- 结果List -->
    <el-card>
      <el-table :data="userList" stripe style="width: 100%">
        <el-table-column label="#" width="80">
          <template slot-scope="scope">
            <!-- (pageNo-1) * pageSize + index + 1 -->
            {{
              (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1
            }}
          </template>
        </el-table-column>
        <el-table-column prop="id" label="ユーザID" width="180">
        </el-table-column>
        <el-table-column prop="username" label="ユーザ名" width="180">
        </el-table-column>
        <el-table-column prop="phone" label="電話番号" width="180">
        </el-table-column>
        <el-table-column prop="status" label="在籍判断" width="180">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status == 1">在籍</el-tag>
            <el-tag v-else type="danger">退職</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="email" label="メールアドレス"> </el-table-column>
        <el-table-column label="操作" width="180"> 
          <template slot-scope="scope">
            <el-button @click="openEditUI(scope.row.id)" type="primary" icon="el-icon-edit" size="mini" circle></el-button>
            <el-button @click="deleteUser(scope.row)" type="danger" icon="el-icon-delete" size="mini" circle></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分页组件 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="searchModel.pageNo"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="searchModel.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
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
        <el-form-item
          label="ユーザ名"
          prop="username"
          :label-width="formLabelWidth"
        >
          <el-input v-model="userForm.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item v-if="userForm.id == null || userForm.id == undefined"
          label="パスワード"
          prop="password"
          :label-width="formLabelWidth"
        >
          <el-input
            type="password"
            v-model="userForm.password"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="電話番号" :label-width="formLabelWidth">
          <el-input v-model="userForm.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="在籍判断" :label-width="formLabelWidth">
          <el-switch
            v-model="userForm.status"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch>
        </el-form-item>
        <el-form-item
          label="メールアドレス"
          prop="email"
          :label-width="formLabelWidth"
        >
          <el-input v-model="userForm.email" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">キャンセル</el-button>
        <el-button type="primary" @click="saveUser">確認</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import userApi from "@/api/userManage";
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      var reg =
        /^[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*@[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*\.[a-z]{2,}$/;
      if (!reg.test(value)) {
        return callback(new Error("メールアドレス形式が間違っています"));
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
        pageSize: 10,
      },
      userList: [],
      rules: {
        username: [
          { required: true, message: "ユーザ名を入力してください", trigger: "blur" },
          {
            min: 3,
            max: 50,
            message: "３ー５０の文字を入力してください",
            trigger: "blur",
          },
        ],
        password: [
          { required: true, message: "初期パスワードを入力してください", trigger: "blur" },
          {
            min: 6,
            max: 16,
            message: "６ー１６の文字を入力してください",
            trigger: "blur",
          },
        ],
        email: [
          { required: true, message: "メールアドレスを入力してください", trigger: "blur" },
          { validator: checkEmail, trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    deleteUser(user){
      this.$confirm(`アカウントを削除してもよろしいですか ${user.username} ?`, 'ヒント', {
          confirmButtonText: '確認',
          cancelButtonText: 'キャンセル',
          type: 'warning'
        }).then(() => {
          userApi.deleteUserById(user.id).then(response => {
            this.$message({
              type: 'success',
              message: response.message
            });
            this.getUserList();
          });
          
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '削除をキャンセルしました。'
          });          
        });
    },
    saveUser() {
      
      this.$refs.userFormRef.validate((valid) => {
        if (valid) {
          
          userApi.saveUser(this.userForm).then(response => {
            // 成功メッセージ
            this.$message({
              message: response.message,
              type: 'success'
            });
           
            this.dialogFormVisible = false;
            
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
    openEditUI(id) {
      if( id == null){
        this.title = "新增用户";
      }else{
        this.title = "修改用户";
        // ユーザデータを検索by ID
        userApi.getUserById(id).then(response => {
          this.userForm = response.data;
        });
      }
      this.dialogFormVisible = true;
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
  },
  created() {
    this.getUserList();
  },
};
</script>

<style>
#search .el-input {
  width: 200px;
  margin-right: 10px;
}
.el-dialog .el-input {
  width: 85%;
}
</style>