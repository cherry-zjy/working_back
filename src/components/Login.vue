<template>
  <div class="login_page fillcontain">
    <transition name="form-fade" mode="in-out">
      <div class="form_img">
        <div class="manage_tip">
            <p>全球英才小程序后台</p>
          </div>
      </div>
    </transition>
    <transition name="form-fade" mode="in-out">
      <section class="form_contianer">
        <div>
          
          <el-form :model="loginForm" :rules="rules" ref="loginForm">
            <el-form-item prop="username">
              <el-input v-model="loginForm.username" placeholder="用户名"></el-input>
            </el-form-item>
            <el-form-item prop="password">
              <el-input type="password" placeholder="密码" v-model="loginForm.password"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="danger" @click="submitForm('loginForm')" class="submit_btn">登陆</el-button>
            </el-form-item>
          </el-form>
        </div>
      </section>
    </transition>
  </div>
</template>

<script>
// import { login, getAdminInfo } from "@/api/getData";
// import { mapActions, mapState } from "vuex";
import md5 from "js-md5";
import qs from "qs";
export default {
  data() {
    return {
      loginForm: {
        username: "",
        password: ""
      },
      rules: {
        username: [
          {
            required: true,
            message: "请输入用户名",
            trigger: "blur"
          }
        ],
        password: [
          {
            required: true,
            message: "请输入密码",
            trigger: "blur"
          }
        ]
      }
    };
  },
  mounted() {},
  computed: {},
  methods: {
    submitForm(formName) {
      var tt = this;
      this.$refs[formName].validate(async valid => {
        if (valid) {
          this.$http
            .post(
              "api/Admin/Login",
              qs.stringify({
                Name: this.loginForm.username,
                Password: md5(this.loginForm.password)
              })
            )
            .then(
              function(response) {
                var status = response.data.Status;
                if (status === 1) {
                  setCookie("token", response.data.Result);
                  setCookie("username", this.loginForm.username);
                  this.$message({
                    showClose: true,
                    type: "success",
                    message: "登陆成功"
                  });
                  setTimeout(() => {
                    this.$router.push({
                      path: "/"
                    });
                  }, 1000);
                } else {
                  this.$message({
                    showClose: true,
                    type: "warning",
                    message: response.data.Result
                  });
                }
              }.bind(this)
            )
            .catch(
              function(error) {
                this.$notify.error({
                  title: "错误",
                  message: "错误：请检查网络"
                });
              }.bind(this)
            );
        } else {
          this.$notify.error({
            title: "错误",
            message: "请输入用户名和密码",
            offset: 100
          });
          return false;
        }
      });
    }
  },
  watch: {
    adminInfo: function(newValue) {}
  }
};
</script>

<style scoped>
/*@import "../../static/css/mixin.css";*/
.login_page {
  /*background-color: #c8161d;*/
  height: 100%;
  text-align: center;
}

.manage_tip {
  position: absolute; 
  width: 100%;
  top: calc(50% - 42.5px);
  color: #fff;
  left: 0;
}

.manage_tip p {
  font-size: 34px;
  padding: 20px 0;
}

.form_img {
  background-color: #D33A31;
  height: 30%;
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  border-radius: 5px;
  text-align: center;
}

.form_contianer {
  position: absolute;
  left: calc(50% - 190px);
  top: calc(30% + 105px);
  /* .wh(320px, 210px);
  .ctp(320px, 210px); */
  width: 320px;
  /* height: 210px; */
  padding: 30px;
  /* box-sizing: border-box; */
  border-radius: 5px;
  text-align: center;
  background-color: #fff;
}

.submit_btn {
  width: 100%;
  font-size: 16px;
  background-color: #D33A31;
}

.form-fade-enter-active,
.form-fade-leave-active {
  transition: all 1s;
}

.form-fade-enter,
.form-fade-leave-active {
  transform: translate3d(0, -50px, 0);
  opacity: 0;
}
</style>
