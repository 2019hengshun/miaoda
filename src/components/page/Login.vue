<template>
  <div class="login-wrap">
    <div class="ms-title">
      秒达钱包管理系统
    </div>
    <div class="ms-login">
      <el-form :model="ruleForm" :rule="rules" ref="ruleForm" label-width="0px" class="demp-ruleForm">
        <el-form-item prop="username">
          <el-input v-model="ruleForm.username" placeholder="用户名"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input type="password" placeholder="密码" v-model="ruleForm.password"
                    @keyup.enter.native="submitForm('ruleForm')"></el-input>
        </el-form-item>
        <div class="login-btn" >
          <el-button type="primary" @click="submitForm()">
            登录
          </el-button>
          <p style="font-size:12px;line-height:30px;color:#999">
            Tips: 账户不存在或密码错误请联系管理员。
          </p>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script>
import { config } from "../../util/config";
import { Message } from "element-ui";

export default {
  data() {
    return {
      ruleForm: {
        username: "",
        password: ""
      },
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" }
        ],
        password: [{ required: true, message: "请输入密码", trigger: "blur" }]
      }
    };
  },
  methods: {
    submitForm() {
      if (this.ruleForm.username != "" && this.ruleForm.password != "") {
        let params = {
          username: this.ruleForm.username,
          password: this.ruleForm.password
        };
        this.$store.dispatch("SETLogin");
        $.ajax({
          type: "POST",
          url: config.baseURL + "/sys/login",
          data: params,
          success: data => {
            if (data.code === 200) {
              let info = {
                username: this.ruleForm.username,
                password: this.ruleForm.password,
                role: data.data,
                loginId: data.loginId,
                key: data.key,
                // key: data.key
                // role: [
                //   "authstatistics",
                //   "signature",
                //   "conditionofbusiness",
                //   "collection",
                //   "url1",
                //   "affair",
                //   "url2",
                //   "role",
                //   "url2",
                //   "message",
                //   "user",
                //   "recordStatistics",
                //   "recallsStatistics",
                //   "logging",
                //   "acyclicMessage",
                //   "passwordManagement",
                //   "myWorkbench",
                //   "systemManagement",
                //   "systemAnnouncement",
                //   "systemAccount",
                //   "getsales",
                //   "lateFeeManagement",
                //   "examinecount",
                //   "repaymentcount",
                //   "registercount",
                //   "presales",
                //   "baofuwithdrawList",
                //   "baofurepaymentlist",
                //   "auditedOrder",
                //   "collection",
                //   "electricityUserList",
                //   "electricalMyClient",
                //   "electricalStatistics",
                //   "paymentlistbycollectmanager",
                //   "paymentapplylist",
                //   "appVersion",
                //   "exceedcount",
                //   "platformexceedcount"
                // ]
              };
              this.$store
                .dispatch("Logins", info)
                .then(re => {})
                .catch(err => {});
              this.$message({
                message: "登录成功",
                type: "success"
              });
              sessionStorage.setItem("fk_username", this.ruleForm.username);
              this.$router.push(data.key);
            } else {
              Message({
                message: data.message,
                center: true
              });
            }
          },
          error: err => {
            Message({
              message: err,
              center: true
            });
          }
        });
      } else {
        Message({
          message: "填写用户名和密码",
          center: true
        });
      }

      /*this.$refs[formName].validate(valid => {
          if (valid) {
            localStorage.setItem("fk_username", this.ruleForm.username);
            this.$router.push("/");
          } else {
            return false;
          }
        });*/
    }
  },
  mounted() {
    sessionStorage.removeItem("USERNAME");
    sessionStorage.removeItem("ROLE");
    sessionStorage.removeItem("LOGINID");
    sessionStorage.removeItem("fk_username");
    // this.$store.dispatch("Logout");
  }
};
</script>

<style>
.login-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  background: #324157;
}

.ms-title {
  position: absolute;
  top: 50%;
  width: 100%;
  margin-top: -230px;
  text-align: center;
  font-size: 30px;
  color: #fff;
}

.ms-login {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 300px;
  height: 160px;
  margin: -150px 0 0 -190px;
  padding: 40px;
  border-radius: 5px;
  background: #fff;
}

.login-btn {
  text-align: center;
}

.login-btn button {
  width: 100%;
  height: 36px;
}
</style>

