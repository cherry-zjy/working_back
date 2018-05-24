<template>
  <div>
    <el-breadcrumb separator="|" class="crumb">
      <el-breadcrumb-item :to="{ path: '/' }">后台管理</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/company' }">企业列表</el-breadcrumb-item>
      <el-breadcrumb-item>企业列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- table 内容 -->
    <el-row>
      <el-form :rules="editForm" ref="editForm" label-width="150px" class="demo-editForm" :label-position="labelPosition">
        <el-col :span="12">
          <p class="title">基本信息</p>
          <el-form-item label="企业名：" prop="Name">
            <el-input v-model="editForm.Name"></el-input>
          </el-form-item>
          <el-form-item label="企业图" prop="Logo">
            <el-upload v-model="editForm.Logo" class="avatar-uploader" :action="action" :show-file-list="false" :on-success="handleAvatarSuccess"
              :before-upload="beforeAvatarUpload">
              <img v-if="imageUrl" :src="imageUrl" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item label="补贴（元）：" prop="Subsidy">
            <el-input v-model="editForm.Subsidy"></el-input>
          </el-form-item>
          <el-form-item label="企业标签：">
            <el-tag type='danger' :key="tag" v-for="tag in dynamicTags" closable :disable-transitions="false" @close="handleClose(tag)">
              {{tag}}
            </el-tag>
            <el-input class="input-new-tag" v-if="inputVisible" v-model="inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm"
              @blur="handleInputConfirm">
            </el-input>
            <el-button v-else class="button-new-tag" size="small" @click="showInput">+</el-button>
          </el-form-item>
          <el-form-item label="工作方式：" prop="Workstyle">
            <el-input v-model="editForm.Workstyle"></el-input>
          </el-form-item>
          <el-form-item label="发薪日：" prop="SalaryDate">
            <el-input v-model="editForm.SalaryDate"></el-input>
          </el-form-item>
          <el-form-item label="企业规划（人）：" prop="EnterprisePeople">
            <el-input v-model="editForm.EnterprisePeople"></el-input>
          </el-form-item>

          <p class="title">企业轮播图</p>
          <el-form-item label="厂区："></el-form-item>
          <el-upload class="upload-demo" :action="action" :on-preview="handlePreview" :on-remove="handleRemove" :file-list="cqimg"
            list-type="picture-card">
            <i class="el-icon-plus"></i>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
          <el-form-item label="吃饭："></el-form-item>
          <el-upload class="upload-demo" :action="action" :on-preview="handlePreview1" :on-remove="handleRemove1" :file-list="eatimg"
            list-type="picture-card">
            <i class="el-icon-plus"></i>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
          <el-form-item label="住宿："></el-form-item>
          <el-upload class="upload-demo" :action="action" :on-preview="handlePreview2" :on-remove="handleRemove2" :file-list="stayimg"
            list-type="picture-card">
            <i class="el-icon-plus"></i>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
          <el-form-item label="工资条："></el-form-item>
          <el-upload class="upload-demo" :action="action" :on-preview="handlePreview3" :on-remove="handleRemove3" :file-list="wageimg"
            list-type="picture-card">
            <i class="el-icon-plus"></i>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
        </el-col>
        <el-col :span="12">
          <p class="title">工资说明</p>
          <el-form-item label="综合薪资：">
            <el-input v-model="editForm.Salary"></el-input>
          </el-form-item>
          <el-form-item label="住宿：">
            <el-input v-model="editForm.SSStay" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="底薪：">
            <el-input v-model="editForm.BasicSalary"></el-input>
          </el-form-item>
          <el-form-item label="合同说明：">
            <el-input v-model="editForm.Contract" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="薪资结构：">
            <el-input v-model="editForm.SalaryExplain" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="工资发放：">
            <el-input v-model="editForm.PayMoney" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="伙食：">
            <el-input v-model="editForm.SSEat" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="保险说明：">
            <el-input v-model="editForm.Insurance" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="温馨提示：">
            <el-input v-model="editForm.Prompt" type="textarea"></el-input>
          </el-form-item>
          <p class="title" style="width:50%">录用条件
            <el-button size="mini" type="danger" @click="dialogFormVisible = true" style="float:right">+</el-button>
          </p>

          <el-table :data="WorkRuler" style="width: 100%">
            <el-table-column prop="Title" width="180">
            </el-table-column>
            <el-table-column prop="Content" width="180">
            </el-table-column>
          </el-table>
          <p class="title">岗位说明</p>
          <el-form-item label="工作内容：">
            <el-input v-model="editForm.WorkContent"></el-input>
          </el-form-item>
          <el-form-item label="两班倒：">
            <el-input v-model="editForm.Changework"></el-input>
          </el-form-item>
          <el-form-item label="工资环境：">
            <el-input v-model="editForm.WorkEnvir"></el-input>
          </el-form-item>
          <p class="title">企业说明</p>
          <el-form-item label="企业说明：">
            <el-input v-model="editForm.EnterpriseExplain" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="企业地址：">
            <el-input v-model="editForm.Adress"></el-input>
          </el-form-item>
        </el-col>
      </el-form>
    </el-row>
    <center>
      <el-button type="primary" @click="back()" class="centerbtn">返回</el-button>
      <el-button type="primary" @click="submitList(listrules)" class="centerbtn">保存</el-button>
    </center>
    <!-- 录用条件模态框 -->
    <el-dialog title="录用条件" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="标题" prop="Title">
          <el-input v-model="ruleForm.Title" style="width:100%"></el-input>
        </el-form-item>
        <el-form-item label="条件" prop="Content">
          <el-input v-model="ruleForm.Content" style="width:100%"></el-input>
        </el-form-item>
        <el-form-item>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm('ruleForm')">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>
<script>
  import qs from "qs";
  export default {
    data() {
      return {
        editForm: [],
        imageForm: [],
        mainurl: "",
        action: '',
        labelPosition: 'left',
        imageUrl: '',
        ServiceProvider: [],
        dynamicTags: [],
        inputVisible: false,
        inputValue: '',
        //工作图片
        cqimg: [],
        eatimg: [],
        stayimg: [],
        wageimg: [],
        //录用条件
        WorkRuler: [],
        dialogFormVisible: false,
        ruleForm: {
          Title: '',
          Content: '',
        },
        listrules: {
          Name: [{
            required: true,
            message: '请输入名称',
            trigger: 'blur'
          }],
          Logo: [{
            required: true,
            message: "请上传图片",
            trigger: "blur"
          }],
          Subsidy: [{
            required: true,
            message: "请输入补贴",
            trigger: "blur"
          }],
          Workstyle: [{
            required: true,
            message: "请输入工作方式",
            trigger: "blur"
          }],
          SalaryDate: [{
            required: true,
            message: "请输入补贴",
            trigger: "blur"
          }],
          EnterprisePeople: [{
            required: true,
            message: "请输入企业规模",
            trigger: "blur"
          }],
        },
        // 表单验证
        rules: {
          Title: [{
            required: true,
            message: '请输入标题',
            trigger: 'blur'
          }, ],
          Content: [{
            required: true,
            message: '请输入录用条件',
            trigger: 'blur'
          }, ],
        }
      };
    },
    computed: {
      userid: function () {
        return window.location.href.split("id=")[1]
      }
    },
    mounted() {
      this.mainurl = mainurl;
      this.action = this.mainurl + "/api/Photo/UpdateForImage?type=0";
      if (window.location.href.split("id=")[1] == '-1') {
        this.editForm = []
      } else {
        this.getInfo()
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
        // 获取详情
        this.$http
          .get("api/Back/Enterprise", {
            params: {
              ID: window.location.href.split("id=")[1],
              Token: getCookie("token")
            }
          })
          .then(
            function (response) {
              loading.close();
              var status = response.data.Status;
              if (status === 1) {
                this.editForm = response.data.Result;
                for (let index = 0; index < response.data.Result.Tag.length; index++) {
                  this.dynamicTags.push(response.data.Result.Tag[index])
                }
                this.WorkRuler = response.data.Result.WorkRuler;
                this.imageUrl = this.mainurl + response.data.Result.Logo;
                for (let i = 0; i < response.data.Result.WorkImages.length; i++) {
                  if (response.data.Result.WorkImages[i].Type == 0) {
                    // alert(1)
                    let arr = Array();
                    arr = {
                      url: this.mainurl + response.data.Result.WorkImages[i].Image
                    }
                    this.cqimg.push(arr)
                  } else if (response.data.Result.WorkImages[i].Type == 1) {
                    let arr = Array();
                    arr = {
                      url: this.mainurl + response.data.Result.WorkImages[i].Image
                    }
                    this.eatimg.push(arr)
                  } else if (response.data.Result.WorkImages[i].Type == 2) {
                    let arr = Array();
                    arr = {
                      url: this.mainurl + response.data.Result.WorkImages[i].Image
                    }
                    this.stayimg.push(arr)
                  } else if (response.data.Result.WorkImages[i].Type == 3) {
                    let arr = Array();
                    arr = {
                      url: this.mainurl + response.data.Result.WorkImages[i].Image
                    }
                    this.wageimg.push(arr)
                  }

                }
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
      back() {
        this.$router.push("/GetProductListIndex");
      },
      //批量上传图片
      handleRemove(file, fileList) {
        console.log(file, fileList);
      },
      handlePreview(file) {
        console.log(file);
      },
      handleRemove1(file, fileList) {
        console.log(file, fileList);
      },
      handlePreview1(file) {
        console.log(file);
      },
      handleRemove2(file, fileList) {
        console.log(file, fileList);
      },
      handlePreview2(file) {
        console.log(file);
      },
      handleRemove3(file, fileList) {
        console.log(file, fileList);
      },
      handlePreview3(file) {
        console.log(file);
      },
      //企业大图
      handleAvatarSuccess(res, file) {
        this.imageUrl = URL.createObjectURL(file.raw);
        this.getList.Image = res.Result[0];
      },
      beforeAvatarUpload(file) {
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isLt2M) {
          this.$message.error("上传头像图片大小不能超过 2MB!");
        }
        return isLt2M;
      },
      //标签管理
      handleClose(tag) {
        this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1);
      },

      showInput() {
        this.inputVisible = true;
        this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
      },

      handleInputConfirm() {
        let inputValue = this.inputValue;
        if (inputValue) {
          this.dynamicTags.push(inputValue);
        }
        this.inputVisible = false;
        this.inputValue = '';
      },
      // 添加录用条件
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.WorkRuler.push(this.ruleForm);
            this.dialogFormVisible = false;
            this.ruleForm = [];
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      // 保存
      submitList(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            // this.WorkRuler.push(this.ruleForm);
            // this.dialogFormVisible = false;
            // this.ruleForm = [];
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
    },

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

  .el-row {
    padding: 20px 50px;
  }

  .title {
    font-size: 22px;
    padding: 15px;
    font-weight: bolder;
  }

  .el-tag+.el-tag {
    margin-left: 10px;
  }

  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }

  .input-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }

  .el-input {
    width: 50%;
  }

  .el-textarea {
    width: 50%;
  }

  .el-table {
    width: 50%;
  }

</style>
