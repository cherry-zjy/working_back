<template>
  <div>
    <el-breadcrumb separator="|" class="crumb">
      <el-breadcrumb-item :to="{ path: '/' }">后台管理</el-breadcrumb-item>
      <el-breadcrumb-item>管理员管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!--检索条-->
    <el-col class="toolbar" style="padding-top: 15px;">
      <el-form :inline="true" :model="filters">
        <!-- <el-form-item>
          <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getUsers()">查询</el-button>
        </el-form-item> -->
        <el-form-item style="float:right;">
          <el-button type="primary" @click="handleAdd()">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>
    <!-- table 内容 -->
    <el-table :data="manageList" style="width: 100%" :border='true'>
      <el-table-column label="用户名" prop="Name">
      </el-table-column>
      <el-table-column label="是否锁定" prop="IsLock" :formatter="IsLock">
      </el-table-column>
      <el-table-column label="创建时间" prop="CreateTime" :formatter="CreateTime">
      </el-table-column>
      <el-table-column label="操作者" prop="Role">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" plain icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <!-- <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button> -->
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div class="block">
      <el-pagination @current-change="handleCurrentChange" layout="prev, pager, next,jumper" :page-count="pageCount">
      </el-pagination>
    </div>
    <!--编辑界面-->
    <el-dialog title="编辑" :visible.sync="editFormVisible">
      <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
        <el-form-item label="账号" prop="Name">
          <el-input v-model="editForm.Name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="Password">
          <el-input type="password" v-model="editForm.Password" :maxlength="20" :clearable='true'></el-input>
        </el-form-item>
        <el-form-item label="锁定">
          <el-radio-group v-model="editForm.IsLock">
            <el-radio class="radio" :label="true">是</el-radio>
            <el-radio class="radio" :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="角色">
          <el-select v-model="editForm.RoleID" placeholder="请选择角色">
            <el-option v-for="item in roleList" :key="item.ID" :label="item.Name" :value="item.ID"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
      </div>
    </el-dialog>
    <!-- 新增界面 -->
    <el-dialog title="新增" :visible.sync="addFormVisible">
      <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
        <el-form-item label="账号" prop="Name">
          <el-input v-model="addForm.Name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="Password">
          <el-input type="password" v-model="addForm.Password" :maxlength="20" :clearable='true'></el-input>
        </el-form-item>
        <!-- <el-form-item label="锁定">
          <el-radio-group v-model="addForm.IsLock">
            <el-radio class="radio" :label="true">是</el-radio>
            <el-radio class="radio" :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item> -->
        <el-form-item label="角色" prop="RoleID">
          <el-select v-model="addForm.RoleID" placeholder="请选择角色">
            <el-option v-for="item in roleList" :key="item.ID" :label="item.Name" :value="item.ID"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="addFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import md5 from "js-md5";

  export default {
    data() {
      return {
        manageList: [], //管理员列表
        pageIndex: 1,
        pageSize: 12,
        pageCount: 1,
        roleList: [], //管理员角色列表
        // 搜索关键字
        filters: {
          keyword: ""
        },
        //编辑界面是否显示
        editFormVisible: false,
        editLoading: false,

        //编辑界面数据
        editForm: {
          IsLock: false,
          Name: "",
          Role: "",
          RoleID: "",
          Password: ""
        },
        editFormRules: {
          Name: [{
            required: true,
            message: "请输入账户",
            trigger: "blur"
          }],
          Password: [{
            required: true,
            message: "请输入密码",
            trigger: "blur"
          }]
        },
        //新增界面是否显示
        addFormVisible: false,
        addLoading: false,
        addFormRules: {
          Name: [{
            required: true,
            message: "请输入账户",
            trigger: "blur"
          }],
          Password: [{
            required: true,
            message: "请输入密码",
            trigger: "blur"
          }],
          RoleID: [{
            required: true,
            message: "请选择角色",
            trigger: "change"
          }]
        },
        //新增界面数据
        addForm: {
          IsLock: false,
          Name: "",
          RoleID: "",
          Password: ""
        }
      };
    },
    methods: {
      /*
           1、获取管理员列表 渲染列表
           2、格式化时间
           3、格式化是否锁定
           4、分页
        */
      getInfo() {
        const loading = this.$loading({
          lock: true,
          text: "Loading",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)"
        });
        this.$http
          .get("api/Admin/GetAdmin", {
            params: {
              pageIndex: this.pageIndex,
              pageSize: this.pageSize
            }
          })
          .then(
            function (response) {
              loading.close();
              var status = response.data.Status;
              if (status === 1) {
                this.manageList = response.data.Result.data;
                this.pageCount = response.data.Result.PageIndex;
              } else if (status === 40001) {
                this.$message({
                  showClose: true,
                  type: "warning",
                  message: response.data.Result
                });
                setTimeout(() => {
                  this.$router.push({
                    path: "/login"
                  });
                }, 1500);
              } else {
                loading.close();
                this.$message({
                  showClose: true,
                  type: "warning",
                  message: response.data.Result
                });
              }
            }.bind(this)
          )
          // 请求error
          .catch(
            function (error) {
              loading.close();
              this.$notify.error({
                title: "错误",
                message: "错误：请检查网络"
              });
            }.bind(this)
          );
      },
      CreateTime(row, time) {
        var date = row[time.property];
        return date.replace("T", " ").split(".")[0];
      },
      IsLock(row, lock) {
        var lock = row[lock.property];
        return lock ? "是" : "否";
      },
      handleCurrentChange(val) {
        this.pageIndex = val;
        this.getInfo();
      },
      /*
          1、添加编辑时获取角色列表，渲染下拉菜单
          2、点击管理员列表的编辑，弹出模态框
          3、点击新增管理严，弹出模态框
          4、保存修改
          5、保存添加
        */
      getRoleList() {
        // 获取角色列表
        this.$http
          .get("api/Role/GetRoles", {
            params: {
              PageIndex: 1,
              PageSize: 999
            }
          })
          .then(
            function (response) {
              var status = response.data.Status;
              if (status === 1) {
                this.roleList = response.data.Result.data;
              } else if (status === 40001) {
                this.$message({
                  showClose: true,
                  type: "warning",
                  message: response.data.Result
                });
                setTimeout(() => {
                  this.$router.push({
                    path: "/login"
                  });
                }, 1500);
              } else {
                this.$message({
                  showClose: true,
                  type: "warning",
                  message: response.data.Result
                });
              }
            }.bind(this)
          )
          // 请求error
          .catch(
            function (error) {
              this.$notify.error({
                title: "错误",
                message: "错误：请检查网络"
              });
            }.bind(this)
          );
      },
      handleEdit(index, row) {
        // console.log(Object.assign({}, row));
        var obj = Object.assign({}, row);
        this.editFormVisible = true;
        // 根据id获取用户信息
        this.$http
          .get("api/Admin/GetAdminByID", {
            params: {
              ID: obj.ID
            }
          })
          .then(
            function (response) {
              var status = response.data.Status;
              if (status === 1) {
                this.editForm = response.data.Result;
                // 将管理员ID传入参数中
                this.editForm.ID = obj.ID;
              } else if (status === 40001) {
                this.$message({
                  showClose: true,
                  type: "warning",
                  message: response.data.Result
                });
                setTimeout(() => {
                  this.$router.push({
                    path: "/login"
                  });
                }, 1500);
              } else {
                this.$message({
                  showClose: true,
                  type: "warning",
                  message: response.data.Result
                });
              }
            }.bind(this)
          )
          // 请求error
          .catch(
            function (error) {
              this.$notify.error({
                title: "错误",
                message: "错误：请检查网络"
              });
            }.bind(this)
          );
        // 获取角色列表
        this.getRoleList();
      },
      handleAdd() {
        this.addFormVisible = true;
        // 获取角色列表
        this.getRoleList();
      },
      editSubmit() {
        this.$refs.editForm.validate(valid => {
          if (valid) {
            //判断是否填写完整  --true
            this.$confirm("确认提交吗？", "提示", {}).then(() => {
              this.editLoading = true;
              var para = Object.assign({}, this.editForm);
              if (para.Password.length > 20) {} else {
                para.Password = md5(para.Password);
              }
              // 将token传入参数中
              para.Token = getCookie("token");
              // 发保存请求
              this.$http
                .get("api/Admin/Edit", {
                  params: para
                })
                .then(
                  function (response) {
                    this.editLoading = false;
                    var status = response.data.Status;
                    if (status === 1) {
                      // 表单重置
                      this.$refs["editForm"].resetFields();
                      this.editFormVisible = false;
                      this.getInfo();
                    } else if (status === 40001) {
                      this.$message({
                        showClose: true,
                        type: "warning",
                        message: response.data.Result
                      });
                      setTimeout(() => {
                        this.$router.push({
                          path: "/login"
                        });
                      }, 1500);
                    } else {
                      this.$message({
                        showClose: true,
                        type: "warning",
                        message: response.data.Result
                      });
                    }
                  }.bind(this)
                )
                // 请求error
                .catch(
                  function (error) {
                    this.$notify.error({
                      title: "错误",
                      message: "错误：请检查网络"
                    });
                  }.bind(this)
                );
            });
          }
        });
      },
      addSubmit() {
        this.$refs.addForm.validate(valid => {
          if (valid) {
            //判断是否填写完整  --true
            this.$confirm("确认提交吗？", "提示", {}).then(() => {
              this.addLoading = true;
              //NProgress.start();
              var para = Object.assign({}, this.addForm);
              if (para.Password.length > 20) {} else {
                para.Password = md5(para.Password);
              }
              // 将token传入参数中
              para.Token = getCookie("token");
              // 发保存请求
              this.$http
                .get("api/Admin/Add", {
                  params: para
                })
                .then(
                  function (response) {
                    this.addLoading = false;
                    var status = response.data.Status;
                    if (status === 1) {
                      // 表单重置
                      this.$refs["addForm"].resetFields();
                      this.addFormVisible = false;
                      this.getInfo();
                    } else if (status === 40001) {
                      this.$message({
                        showClose: true,
                        type: "warning",
                        message: response.data.Result
                      });
                      setTimeout(() => {
                        this.$router.push({
                          path: "/login"
                        });
                      }, 1500);
                    } else {
                      this.$message({
                        showClose: true,
                        type: "warning",
                        message: response.data.Result
                      });
                    }
                  }.bind(this)
                )
                // 请求error
                .catch(
                  function (error) {
                    this.$notify.error({
                      title: "错误",
                      message: "错误：请检查网络"
                    });
                  }.bind(this)
                );
            });
          }
        });
      }
    },
    mounted() {
      this.getInfo();
    }
  };

</script>
<style scoped>
  /* 面包屑 */

  .crumb {
    height: 36px;
    line-height: 36px;
  }

  .block {
    text-align: center;
    padding: 20px 0;
  }

</style>
