<template>
  <div class="login">
    <div class="left">
      <div class="logo-wrap">
        <span>
          <img src="@/assets/img/logo_yq.png" width="120" />
        </span>
        <span>
          <img src="@/assets/img/logo_audi.png" width="240" />
        </span>
      </div>
      <div class="copy-wrap">
        <img src="@/assets/img/login_copy.png" width="640" />
      </div>
      <div class="btn-wrap">
        <a href="javascript:;" class="audi-button">
          <span>了解详情</span>
          <i class="el-icon-right" />
        </a>
      </div>
    </div>
    <div class="right">
      <div class="wrap">
        <h3>
          <img src="@/assets/img/login_title.png" width="260" />
        </h3>
        <div class="main">
          <div class="top">
            <template v-for="(item, index) in labels">
              <span :key="item.value" :class="{ actived: currentMark === item.value }" @click="clickHandler(item.value)">{{ item.text }}</span>
              <i v-if="index !== labels.length - 1" :key="index">|</i>
            </template>
          </div>
          <div class="container">
            <div class="scroll" :style="scrollTranslate">
              <div class="box">
                <el-form ref="form-user" size="medium" :model="form" :rules="rules">
                  <el-form-item prop="username">
                    <el-input v-model="form.username" prefix-icon="el-icon-user" placeholder="请输入用户名" auto-complete="on" />
                  </el-form-item>
                  <el-form-item prop="password">
                    <el-input v-model="form.password" :type="passwordType" placeholder="请输入密码" prefix-icon="el-icon-lock" auto-complete="on" @keyup.enter.native="loginHandle" />
                    <span class="show-pwd" @click="showPwdHandle">
                      <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
                    </span>
                  </el-form-item>
                </el-form>
                <div class="forget">
                  <a href="javascript:;" @click="forgetPwdHandle">忘记密码</a>
                </div>
              </div>
              <div class="box">
                <el-form ref="form-phone" size="medium" :model="form" :rules="rules">
                  <el-form-item prop="phone">
                    <el-input v-model="form.phone" prefix-icon="el-icon-mobile-phone" placeholder="请输入手机号" />
                  </el-form-item>
                  <el-form-item>
                    <el-input v-model="form.vcode" type="password" class="fl" placeholder="验证码" style="width: 50%" prefix-icon="el-icon-message" />
                    <el-button class="fr" style="width: 38%">获取验证码</el-button>
                  </el-form-item>
                </el-form>
              </div>
            </div>
            <div style="padding-top: 30px;">
              <multiuse-button type="primary" size="medium" class="login-btn" :click="loginHandle">登 录</multiuse-button>
            </div>
          </div>
          <div class="footer">
            <div class="quick">
              <span @click="showDialogHandle('Wx')">
                <i class="iconfont icon-weixin" />
                微信登录
              </span>
              <span @click="showDialogHandle('App')">
                <i class="iconfont icon-appxiazai" />
                APP下载
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <BaseDialog :visible.sync="actionWx.visible" :title="actionWx.title" width="500px" destroy-on-close>
      <div class="tc">
        <img :src="wxLoginQrcode" width="300" />
      </div>
      <div class="tc" style="padding-bottom: 15px;">{{ actionApp.desc }}</div>
    </BaseDialog>
    <BaseDialog :visible.sync="actionApp.visible" :title="actionApp.title" width="500px" destroy-on-close>
      <div class="tc">
        <img :src="appDevQrcode" width="300" />
      </div>
      <div class="tc" style="padding-bottom: 15px;">{{ actionApp.desc }}</div>
    </BaseDialog>
    <BaseDialog :visible.sync="actionPwd.visible" :title="actionPwd.title" width="500px" destroy-on-close>
      <backPwd @close="closePwdBackHandle" />
    </BaseDialog>
  </div>
</template>

<script>
import { mapActions } from 'vuex';
import { sleep } from '@/utils';
import { phone } from '@/utils/validate';
import { doLogin } from '@/api/login';

import backPwd from './backPwd.vue';

// app 在 dev 和 prod 环境的下载二维码
const appDevQrcode = require('@/assets/img/app_dev_qrcode.png');
const appProdQrcode = require('@/assets/img/app_prod_qrcode.png');

// 微信登录的二维码
const wxLoginQrcode = require('@/assets/img/wx_qrcode.png');

export default {
  name: 'Login',
  components: {
    backPwd
  },
  data() {
    // tab 标签
    this.labels = [
      { text: '用户名登录', value: 'user' },
      { text: '手机号登录', value: 'phone' }
    ];
    // app 二维码
    this.appDevQrcode = appDevQrcode;
    this.appProdQrcode = appProdQrcode;
    this.wxLoginQrcode = wxLoginQrcode;
    return {
      form: {
        username: '',
        password: '',
        phone: '',
        vcode: ''
      },
      rules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        phone: [{ required: true, validator: phone, trigger: 'blur' }]
      },
      passwordType: 'password',
      currentMark: 'user', // 当前标记
      actionWx: {
        title: '微信登录',
        visible: false,
        desc: '请使用微信扫描二维码登录'
      },
      actionApp: {
        title: '下载APP',
        visible: false,
        desc: '请扫描二维码下载奥迪销售助手APP'
      },
      actionPwd: {
        title: '密码找回',
        visible: false
      }
    };
  },
  computed: {
    scrollTranslate() {
      const index = this.labels.findIndex(x => x.value === this.currentMark);
      const val = -1 * (index / this.labels.length) * 100 + '%';
      return {
        transform: `translate3d(${val}, 0, 0)`
      };
    }
  },
  methods: {
    ...mapActions('app', ['createLoginInfo']),
    clickHandler(type) {
      this.currentMark = type;
    },
    showPwdHandle() {
      if (this.passwordType === 'password') {
        this.passwordType = '';
      } else {
        this.passwordType = 'password';
      }
    },
    showDialogHandle(type) {
      this[`action${type}`].visible = true;
    },
    forgetPwdHandle() {
      this.actionPwd.visible = true;
    },
    closePwdBackHandle(val) {
      this.actionPwd.visible = false;
    },
    async loginHandle() {
      const res = await doLogin({
        username: this.form.username,
        password: this.form.password
      });
      if (res.code === 1) {
        this.createLoginInfo({
          id: res.data.id,
          name: res.data.name,
          token: res.data.token
        });
        this.$router.push({ path: '/' });
      }
    }
  }
};
</script>

<style lang="less" scoped>
.login {
  display: flex;
  height: 100vh;
  background: #fff url(../../assets/img/login_bg.svg) no-repeat 50%;
  background-size: 100%;
  .left {
    flex: 1;
    background: url(../../assets/img/login_left_bg.jpg) 100% 0 no-repeat;
    background-size: cover;
    .logo-wrap {
      display: flex;
      padding: 8vh 10vh 0;
      justify-content: space-between;
      align-items: center;
    }
    .copy-wrap {
      padding: 15vh 10vh 0;
    }
    .btn-wrap {
      padding: 6vh 10vh 0;
    }
    .audi-button {
      display: inline-block;
      width: 240px;
      height: 46px;
      border: 1px solid #fff;
      border-radius: 2px;
      line-height: 46px;
      text-align: center;
      font-size: 16px;
      color: #fff;
      transition: all 0.3s ease;
      span {
        margin-right: 5px;
        vertical-align: middle;
      }
      i {
        transform: scale(1.3, 1);
        vertical-align: middle;
      }
      &:hover {
        background-color: @primaryColor;
        border-color: @primaryColor;
      }
    }
  }
  .right {
    width: 34%;
    overflow-y: auto;
    .wrap {
      padding: 15vh 40px 0;
      .main {
        padding-top: 10vh;
        .top {
          display: flex;
          color: @textColorSecondary;
          span {
            padding: 6px;
            cursor: pointer;
            &.actived {
              color: @primaryColor;
            }
          }
          i {
            font-size: 13px;
            padding: 6px 0;
          }
        }
        .container {
          width: 100%;
          padding-top: 30px;
          overflow: hidden;
          .scroll {
            width: 200%;
            height: 130px;
            display: flex;
            transition: transform 0.3s ease;
            .box {
              width: 50%;
              /deep/ .el-form-item {
                margin-bottom: 20px;
              }
              /deep/ .el-input__inner {
                font-size: 14px;
                border: 0;
                border-bottom: 1px solid #d6d6d6 !important;
              }
              /deep/ .el-input__icon {
                font-size: 18px;
              }
              .show-pwd {
                position: absolute;
                top: 0;
                right: 10px;
                font-size: 16px;
                color: @textColorSecondary;
                cursor: pointer;
                user-select: none;
              }
              .forget {
                margin-top: -5px;
                text-align: right;
                a {
                  font-size: 13px;
                  &:hover {
                    color: @primaryColor;
                  }
                }
              }
            }
          }
          .login-btn {
            width: 100%;
            height: 42px;
            font-size: 14px;
          }
        }
        .footer {
          margin-top: 15vh;
          .quick {
            line-height: 30px;
            span {
              line-height: 20px;
              margin-right: 20px;
              cursor: pointer;
              .iconfont {
                font-size: 18px;
                vertical-align: middle;
              }
              &:hover {
                color: @primaryColor;
              }
            }
          }
        }
      }
    }
  }
}
</style>
