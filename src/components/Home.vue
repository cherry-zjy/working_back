<template>
  <div id="app">
    <!-- header -->
    <div>
      <el-header style="background-color: #D33A31;">
        <!-- <div style="float: left;height: 60px;">
          <img src="../assets/logo.png" style="height:50px;width:50px;margin:5px 10px;" alt="">
        </div> -->
        <div v-if="welcome" style="float: left;height: 60px;color:#fff">欢迎来到全球英才小程序后台管理系统</div>
        <div style="float: right;color:#fff; font-size: 12px;margin-right:50px;">
          <el-dropdown>
            <i class="el-icon-setting" style="margin-right: 15px;color:#fff;"></i>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item>
                <div @click="jumpTo('/manage')">
                  <span style="color: #555;font-size: 14px;">个人信息</span>
                </div>
              </el-dropdown-item>
              <el-dropdown-item divided @click.native="logout">退出登录</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
          <span @click="aaa()">{{userName}}</span>
        </div>
        <div class="clear"></div>
      </el-header>
    </div>
    <!-- body -->
    <el-container>
      <aside style="background-color: #FAF5F5;">
        <div style="text-align:center">
          <img v-if="!iscloseNav" src="../../static/images/open_nav.png" style="width:30px;height:30px;margin:5px 10px;" alt="" @click="closeNav()">
          <img v-else src="../../static/images/close_nav.png" style="width:30px;height:30px;margin:5px 10px;" alt="" @click="closeNav()">
        </div>
        <el-menu style="background-color: #FAF5F5;" class="el-menu-vertical-demo" :default-active="$route.path" router :unique-opened='true'
          @select="handleSelect" :collapse='iscloseNav'>
          <template v-for="(item,index) in menuList">
            <el-submenu :index="index+''" :key="index">
              <template slot="title">
                <img src="../../static/images/menu_list.png" v-if="!iscloseNav" alt="">
                <i v-else class="el-icon-menu"></i>
                <span slot="title" style="color:#000">{{item.name}}</span>
              </template>
              <el-menu-item v-for="(ll,ii) in item.snippet" :key="ii" :index="'/'+ll.urlf.split('.')[0]">{{ll.title}}</el-menu-item>
            </el-submenu>
          </template>
        </el-menu>
      </aside>
      <!-- 右侧路由信息 -->
      <el-container>
        <el-main v-if="collapsed">
          <router-view></router-view>
        </el-main>
        <el-main v-else>
          <h2>全球英才小程序后台管理</h2>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
  export default {
    data: function () {
      return {
        defaultActiveIndex: ["0"],
        menuList: [{
        "name": "人员管理",
          "snippet": [{
            "title": "管理员管理",
            "urlf": "manage.html",
          }]
        }, {
          "name": "打工管理",
          "snippet": [{
            "title": "企业列表",
            "urlf": "company.html",
          }, {
            "title": "门店列表",
            "urlf": "store.html",
          }, {
            "title": "报名列表",
            "urlf": "enter.html",
          }]
        }, {
          "name": "用户管理",
          "snippet": [{
            "title": "普通用户列表",
            "urlf": "user.html",
          }, {
            "title": "工资列表",
            "urlf": "wagelist.html",
          }, {
            "title": "经纪人列表",
            "urlf": "agent.html",
          }]
        }, {
          "name": "提现管理",
          "snippet": [{
            "title": "提现列表",
            "urlf": "withdraw.html",
          }]
        },{
          "name": "平台管理",
          "snippet": [{
            "title": "今日推荐",
            "urlf": "banner.html",
          },{
            "title": "意见反馈列表",
            "urlf": "feedback.html",
          },{
            "title": "评价列表",
            "urlf": "comment.html",
          }]
        },],
        userName: "",
        collapsed: false,
        iscloseNav: false,
        welcome: true
      };
    },
    mounted() {
      var w = window.innerWidth;
      if (w < 500) {
        this.welcome = false;
      }
      this.userName = getCookie("username");
      var url = window.location.href;
      if (url.split("#")[1] == "/") {} else {
        this.collapsed = true;
      }
      var tt = this;
      if (getCookie("token")) {
        // this.$http
        //   .get("api/Menu/GetMenus", {
        //     params: {
        //       Token: getCookie("token")
        //     }
        //   })
        //   .then(
        //     function (response) {
        //       var status = response.data.Status;
        //       if (status === 1) {
        //         this.menuList = response.data.Result;
        //       } else if (status === 40001) {
        //         this.$message({
        //           showClose: true,
        //           type: "warning",
        //           message: response.data.Result
        //         });
        //         setTimeout(() => {
        //           this.$router.push({
        //             path: "/login"
        //           });
        //         }, 1500);
        //       } else {
        //         this.$message({
        //           showClose: true,
        //           type: "warning",
        //           message: response.data.Result
        //         });
        //       }
        //     }.bind(this)
        //   )
        //   .catch(
        //     function (error) {
        //       this.$notify.error({
        //         title: "错误",
        //         message: "错误：请检查网络"
        //       });
        //       setTimeout(() => {
        //         this.$router.push({
        //           path: "/error"
        //         });
        //       }, 1500);
        //     }.bind(this)
        //   );
      } else {
        this.$message({
          showClose: true,
          type: "warning",
          message: "请先登陆"
        });
        setTimeout(() => {
          this.$router.push({
            path: "/login"
          });
        }, 1500);
      }
    },
    methods: {
      // 	index: 选中菜单项的 index, indexPath: 选中菜单项的 index path
      handleSelect(index) {
        this.collapsed = true;
        // this.defaultActiveIndex = [index];
        // console.log(this.$route.path);
      },
      // 个人中心  修改密码
      jumpTo(url) {
        // this.defaultActiveIndex = url;
        console.log(url);
        this.collapsed = true;
        this.$router.push(url);
      },
      // 退出
      logout() {
        let that = this;
        this.$confirm("确认退出吗?", "提示", {
            confirmButtonClass: "el-button--warning"
          })
          .then(() => {
            //确认
            that.loading = true;
            delCookie("token");
            this.$router.push("/login");
          })
          .catch(() => {});
      },
      closeNav() {
        this.iscloseNav = !this.iscloseNav;
        if (this.iscloseNav) {
          $(".el-aside").css({
            width: "60px"
          });
        }
      }
    }
  };

</script>

<style scoped>
  /* @import "../../static/css/index.css"; */

  .el-header {
    color: #333;
    line-height: 60px;
  }

</style>
