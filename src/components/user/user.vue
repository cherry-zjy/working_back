<template>
  <div>
    <el-breadcrumb separator="|" class="crumb">
      <el-breadcrumb-item :to="{ path: '/' }">后台管理</el-breadcrumb-item>
      <el-breadcrumb-item>普通用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--检索条-->
    <el-col class="toolbar" style="padding-top: 15px;">
      <el-form :inline="true" :model="filters">
        <el-form-item label="关键字">
          <el-input v-model="filters.keyword" placeholder="关键字" prefix-icon="el-icon-search"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getUsers()">查询</el-button>
        </el-form-item>
        <el-form-item style="float:right;">
          <el-button type="primary" @click="handleAdd()">导入工资条</el-button>
          <input id="file" type="file" @change="FileIn()" />
        </el-form-item>
      </el-form>
    </el-col>
    <!-- table 内容 -->
    <el-table :data="List" style="width: 100%" :border='true'>
      <el-table-column label="用户名" prop="Name" width="200">
      </el-table-column>
      <el-table-column label="联系电话" prop="Phone" width="200">
      </el-table-column>
      <el-table-column label="经纪人" prop="Agent" width="200">
      </el-table-column>
      <el-table-column label="推荐人" prop="Referee" width="200">
      </el-table-column>
      <el-table-column label="账户余额" prop="Money" width="200">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" @click="setagent(scope.row.ID,scope.row.Agent)">设置经纪人</el-button>
          <el-button size="mini" type="warning" @click="detail(scope.row.ID)">用户详情</el-button>
          <el-button size="mini" type="success" @click="experience(scope.row.ID)">工作经历</el-button>
          <el-button v-if="scope.row.IDcard !== ''" size="mini" type="info" @click="wage(scope.row.IDcard)">工资信息</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div class="block">
      <el-pagination @current-change="handleCurrentChange" layout="prev, pager, next,jumper" :page-count="pageCount">
      </el-pagination>
    </div>

    <!-- 模态框 -->
    <el-dialog title="设置经纪人" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="phoneList" :rules="agentrules" ref="phoneList" label-width="100px" class="demo-ruleForm">
        <el-select v-model="phoneList.Phone" placeholder="请选择经纪人">
          <el-option v-for="item in phoneList" :key="item.Phone" :label="item.Name" :value="item.Phone"></el-option>
        </el-select>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="agent('phoneList')">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>
<script>
  export default {
    data() {
      return {
        pageIndex: 1,
        pageSize: 12,
        pageCount: 1,
        List: [], //管理员角色列表
        // 搜索关键字
        filters: {
          Query: "",
        },
        dialogFormVisible: false,
        mainurl: '',
        phoneList: {},
        agentrules: {
          Phone: [{
            required: true,
            message: '请选择经纪人',
            trigger: 'change'
          }],
        }
      }
    },
    methods: {
      getInfo() {
        const loading = this.$loading({
          lock: true,
          text: "Loading",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)"
        });
        this.$http
          .get("api/Back/QueryUser", {
            params: {
              pageIndex: this.pageIndex,
              pageSize: this.pageSize,
              Query: (this.filters.Query == '') ? '-1' : this.filters.Query,
              Token: getCookie("token"),
              Type: 0
            }
          })
          .then(
            function (response) {
              loading.close();
              var status = response.data.Status;
              if (status === 1) {
                this.List = response.data.Result.list;
                this.pageCount = response.data.Result.page;
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
      getagent() {
        const loading = this.$loading({
          lock: true,
          text: "Loading",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)"
        });
        // 获取详情
        this.$http
          .get("api/Back/QueryUser", {
            params: {
              pageIndex: 1,
              pageSize: 999,
              Query: -1,
              Type: 1,
              Token: getCookie("token"),
            }
          })
          .then(
            function (response) {
              loading.close();
              var status = response.data.Status;
              if (status === 1) {
                this.phoneList = response.data.Result.list;
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
              //   this.$notify.error({
              //     title: "错误",
              //     message: "错误：请检查网络"
              //   });
            }.bind(this)
          );
      },
      //设置经纪人
      setagent(id,agent) {
        this.phoneList.Phone = agent;
        this.dialogFormVisible = true;
        this.phoneList.ID = id
      },
      agent(formName) {
        this.$refs[formName].validate(valid => {
          if (valid) {
            //判断是否填写完整  --true
            const loading = this.$loading({
              lock: true,
              text: "Loading",
              spinner: "el-icon-loading",
              background: "rgba(0, 0, 0, 0.7)"
            });
            this.$http
              .get("api/Back/AddUserUp", {
                params: {
                  Token: getCookie("token"),
                  ID: this.phoneList.ID,
                  Phone: this.phoneList.Phone
                }
              })
              .then(
                function (response) {
                  loading.close();
                  var status = response.data.Status;
                  if (status === 1) {
                    this.$message({
                      showClose: true,
                      type: "success",
                      message: response.data.Result
                    });
                    this.dialogFormVisible = false;
                    this.getInfo()
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
          }
        })
      },
      
      //用户详情
      detail(id){
        this.$router.push("/user/userdetail/id=" + id);
      },
      //工作经历
      experience(id){
        this.$router.push("/user/experience/id=" + id);
      },
      //工资详情
      wage(id){
        this.$router.push("/user/wage/id=" + id);
      },
      handleCurrentChange(val) {
        this.pageIndex = val;
        this.getInfo();
      },
      //导入工资条
      FileIn() {
      const loadingDR = this.$loading({
        lock: true,
        text: "Loading",
        spinner: "el-icon-loading",
        background: "rgba(0, 0, 0, 0.7)"
      });
      var formdata = new FormData();
      formdata.append("file", $("#file")[0].files[0]); //获取文件法二
      this.$http
        .post(
          "api/Back/Import",
          formdata
          // qs.stringify({
          //   params: formdata
          // })
          // ,
          // {
          //   headers: { "Content-Type": "application/octet-stream" }
          // } //添加请求头
        )
        .then(
          function(response) {
            loadingDR.close();
            var status = response.data.Status;
            if (status === 1) {
              this.getInfo();
              this.$message({
                showClose: true,
                type: "success",
                message: "导入成功"
              });
            } else if (status === -1) {
              this.$message({
                showClose: true,
                type: "warning",
                message: "请不要重复导入"
              });
            }
          }.bind(this)
        )
        // 请求error
        .catch(
          function(error) {
            loadingDR.close();
            this.$notify.error({
              title: "错误",
              message: "错误：请检查网络"
            });
          }.bind(this)
        );
    },
    },
    mounted() {
      this.mainurl = mainurl;
      this.getInfo();
      this.getagent();
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

  #file {
    position: absolute;
    right: 0;
    top: 0;
    width: 70px;
    height: 40px;
    opacity: 0;
  }

</style>
