<!--登录页-->
<template>
    <div class="login1" v-if="isRender">
        <canvas id="flash-dome"></canvas>
        <canvas id="flash-dome-hide"></canvas>
        <img src="../../assets/login/logo.png" alt="" class="logo">
        <div class="main-content">
            <div class="left-content">
                <div class="chart-content">
                    <div id="canvas-frame"></div>
                    <div class="bottom-box"></div>
                </div>
            </div>
            <div class="right-content">
            <!-- 登陆/账号登陆 -->
            <transition name="el-zoom-in-center">
                <div class="login_con" v-if="isFindPassword" style="height:9999px;">
                    <div :class="activeName==1?'codeC':'loginC'" @click="changeLogin"></div>
                    <div class="tip">
                        {{loginType}}
                    </div>
                    <!-- 扫码界面 -->
                    <div class="login_type" v-show="activeName == 0">
                        <!-- 扫码展示 -->
                        <div class="please-code" v-show="sancCode">
                            <div class="qrCode" id="qrCode" v-loading="loading"></div>
                            <p class="code_login">
                                <img class="img" src="../../assets/sweep-code.png" alt="">
                                扫码登陆慧云
                            </p>
                            <p class="code-title">请使用慧云APP扫一扫</p>
                        </div>
                        <!-- 扫码成功展示 -->
                        <div class="success-code" v-if="successCode">
                            <div class="success-code" style="text-align: center"><img style="width: auto;" src="../../assets/code-phone.png" alt=""></div>
                            <p class="code_login">扫码成功</p>
                            <p class="code-title">请在手机慧云中点击登陆</p>
                        </div>
                        <!-- 加载失败展示 -->
                        <div class="loser-code" v-if="loserCode">
                            <div class="qrCode" v-loading="loading"><img src="../../assets/loser-img.png" alt=""></div>
                            <div class="new-btn">
                                <el-button class="loser-btn" @click="addCode">刷 新</el-button>
                            </div>
                            <p class="code-title">请使用慧云APP扫一扫</p>
                        </div>
                    </div>
                    <!-- 账号登录界面 -->
                    <div class="login_type" v-if="activeName === '1'">
                        <div class="title" style="margin-bottom: 20px;">
                            <p class="title-z">Hi</p>
                            <p class="title-b">欢迎回来</p>
                        </div>
                        <div class="show-error" v-show="errorMsg">{{errorMsg}}</div>
                        <el-form class="elForm" label-position="top"  :model="ruleForm"  ref="ruleForm" style="width:100%">
                            <el-form-item class="user-name">
                                <el-input placeholder="用户名" @input="checkUserName" ref="inputName" @keyup.enter.native="loginSubmit" v-model="ruleForm.userName"  style="width:100%;"></el-input>
                                <div class="flag"></div>
                                <i class="icon-username"></i>
                            </el-form-item>
                            <el-form-item class="pass-word">
                                <el-input type="password"  placeholder="密码" @input="checkPassword" @keyup.enter.native="loginSubmit" v-model="ruleForm.userPassword" style="width:100%;margin-top:8px;"></el-input>
                                <div class="flag"></div>
                                <i class="icon-password"></i>
                            </el-form-item>
                            <el-button class="btn btnv" :class="{btnColr:(!isCheckMap.userName && !isCheckMap.password)}" type="primary" @click="loginSubmit" :disabled="isCheckMap.userName || isCheckMap.password">{{loginText}}</el-button>
                            <el-row  class="elRow"  :gutter='20'>
                                <el-col :span='24'><span class="forget-password" @click="goPassword">忘记密码</span></el-col>
                            </el-row>
                        </el-form>
                    </div>
                </div>
            </transition>
                <!-- 忘记密码界面展示 -->
                <div class="login_password" v-if="!isFindPassword">
                    <template v-if="!findSuccess">
                        <div class="header-btn">
                            <ul class="list">
                                <li class="list-item" v-for="(item,index) in tabArr" :key="index" :class="{active:index == num}" @click="goBtn(index)">{{item}}</li>
                            </ul>
                            <div class="border"></div>
                        </div>
                        <!-- 手机找回 -->
                        <div class="show-error" :class="'status'+statusType">{{errorMsg}}</div>
                        <div class="content" v-show="num == 0">
                            <el-form class="elForm" label-position="top"  :model="ruleForm"  ref="ruleForm" style="width:100%">
                                <el-row v-show="formTow">
                                    <el-form-item class="user-name">
                                        <el-input placeholder="请输入手机号码" @input="checkPhone" type="number" v-model="ruleForm.phone"  style="width:100%;"></el-input>
                                    </el-form-item>
                                    <el-form-item class="pass-word">
                                        <el-input name="code123" v-model="ruleForm.code" type="number" @input="checkCode" placeholder="请输入短信验证码" style="width:198px;margin-top:16px;"></el-input>
                                        <button class="obtain" type="button" :disabled="isCheckMap.phone || countDown" @click="getPhoneCode()" v-text="showBtnText"></button>
                                    </el-form-item>
                                    <div id="slide"></div>
                                    <el-button class="btn btnv" type="primary" style="margin-top:40px;" :class="{btnColr:(!isCheckMap.phone && !isCheckMap.code && !isCheckMap.slide)}" :disabled="isCheckMap.phone || isCheckMap.code || isCheckMap.slide || isInActive" @click="phoneNext">下一步</el-button>
                                    <el-row  class="elRow"  :gutter='20'>
                                        <el-col :span='24'><span class="return-login" @click="goCode">返回登录</span></el-col>
                                    </el-row>
                                </el-row>
                                <!-- 重置密码 -->
                                <el-row v-show="!formTow">
                                    <el-form-item class="pass-word">
                                        <el-input type="password"  placeholder="请输入新密码" @input="checkPasswordConfirm(1)"  v-model="ruleForm.userPassword1" style="width:100%"></el-input>
                                    </el-form-item>
                                    <el-form-item class="pass-word">
                                        <el-input type="password"  placeholder="再次确认新密码" @input="checkPasswordConfirm(0)"  v-model="ruleForm.userPassword2" style="width:100%;margin-top:16px;"></el-input>
                                    </el-form-item>
                                    <el-button class="btn btnv" type="primary" style="margin-top:40px;" @click="resetPassword" :class="{btnColr:(!isCheckMap.userPassword1 && !isCheckMap.userPassword2)}" :disabled="isCheckMap.userPassword1 || isCheckMap.userPassword2">确认重置</el-button>
                                    <el-row  class="elRow" :gutter='20'>
                                        <el-col :span='24'><span class="return-login" @click="goOne">返回上一步</span></el-col>
                                    </el-row>
                                </el-row>
                            </el-form>
                        </div>
                        <!-- 邮箱找回 -->
                        <div class="content content-code" v-show="num == 1">
                            <el-form class="elForm" label-position="top"  ref="ruleForm" style="width:100%">
                                <el-form-item class="user-name">
                                    <el-input placeholder="请输入邮箱" @input="checkEmail" v-model="ruleForm.email" style="width:100%;"></el-input>
                                </el-form-item>
                                <el-button class="btn btnv" type="primary" :class="{btnColr:(!isCheckMap.email)}" :disabled="isCheckMap.email || isInActive" @click="sendEmail" style="margin-top:40px;">发送验证</el-button>
                                <el-row  class="elRow"  :gutter='20'>
                                    <el-col :span='24'><span class="return-login" @click="goCode">返回登录</span></el-col>
                                </el-row>
                            </el-form>
                        </div>
                    </template>
                    <!-- 邮箱重置密码界面 -->
                    <div class="mailbox-password" v-else-if="mailboxCode">
                        <div class="title-header">重置密码</div>
                        <div class="show-error" :class="'status'+statusType">{{errorMsg}}</div>
                        <el-form class="elForm" label-position="top"  :model="ruleForm"  ref="ruleForm" style="width:100%">
                            <el-form-item class="pass-word">
                                <el-input type="password"  placeholder="请输入新密码" @input="checkPasswordConfirm(1)"  v-model="ruleForm.userPassword1" style="width:100%;margin-top:16px;"></el-input>
                            </el-form-item>
                            <el-form-item class="pass-word">
                                <el-input type="password"  placeholder="再次确认新密码" @input="checkPasswordConfirm(0)"  v-model="ruleForm.userPassword2" style="width:100%;margin-top:16px;"></el-input>
                            </el-form-item>
                            <el-button class="btn btnv" type="primary" style="margin-top:40px;" :class="{btnColr:(!isCheckMap.userPassword1 && !isCheckMap.userPassword2)}" @click="resetPassword" :disabled="isCheckMap.userPassword1 || isCheckMap.userPassword2">确认重置</el-button>
                            <el-row  class="elRow" :gutter='20'>
                                <el-col :span='24'><span class="return-login" @click="goOne">返回上一步</span></el-col>
                            </el-row>
                        </el-form>
                    </div>
                    <!-- 重置密码成功界面 -->
                    <div class="reset-pass" v-else-if="findSuccess">
                        <img src="../../assets/sucess-copy.png" alt="" style="display: block;margin: 0 auto">
                        <p class="pass-title">重置成功</p>
                        <el-row  class="elRow"  :gutter='20'>
                            <el-col :span='24'>3秒后自动跳转<a class="return-login" @click="toLogin">重新登录</a>!</el-col>
                        </el-row>
                    </div>
                </div>
                <div class="icp">
                    <el-row  class="elTow"  :gutter='20'>
                        <el-col :span='24'>用户登陆慧云默认同意<router-link to="">《慧云用户协议》</router-link></el-col>
                    </el-row>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import FlashChart from '../../assets/js/flashChart'
    import QRCode from 'qrcodejs2';

    import initThree from '../../assets/js/earch'
    import {destroyAnimate} from '../../assets/js/earch'
    import api from '../../fetch/api.js';
    import * as access from "../../config/access";

    let intervalId = null
    let flashChart
    export default {
        name: "login1",
        data() {
            return {
                codeText:[
                    {code:401,text:'邮箱验证，跳到重置的界面!'},
                    {code:402,text:'超时！'},
                    {code:403,text:'校验不通过！'}
                ],
                activeName:'1',
                num:"0",
                validate:'',
                sancCode:true,
                countDown:false,
                successCode:false,
                newImg:false,
                loginText:'登录',
                loading:false,
                findSuccess:false,
                isFindPassword:true,
                formTow:true,
                isRender:true,
                isInActive:false,
                mailboxCode:false,
                loserCode:false,
                statusType:'',
                showBtnText:'获取验证码',
                errorMsg:'',
                uuid:'',
                customerImg:'',
                isGetCoding:false,
                isCheckMap:{
                    phone:true,
                    code:true,
                    slide:true,
                    userName:true,
                    password:true,
                    email:true,
                    userPassword1:true,
                    userPassword2:true
                },
                tabArr:["手机找回","邮箱找回"],
                ruleForm:{
                    userName:'',
                    userPassword:'',
                    phone:'',
                    code:'',
                    userPassword1:'',
                    userPassword2:''
                }
            }
        },
        beforeRouteLeave(to, from, next){
            destroyAnimate()
            flashChart.destroy()
            this.isRender = false
            next()
        },
        methods: {
            addCode(){
                this.successCode = false
                this.sancCode = true
                this.loserCode = false
                this.loading = true
                let url = '/uaa_user/saveLgtoken';
                this.uuid = this.getUuid();
                this.isGetCoding = true
                api.http({
                    url: url,
                    params:{
                        lgtoken: this.uuid
                    }
                }).then(res=>{
                    this.loading = false
                    if(res.code == 200){
                        this.qrCode();
                    }else{
                        this.isGetCoding = false;
                    }
                }).catch(()=>{
                    this.loading = false
                })
            },
            getUuid(){
                let chars = '0123456789abcdefghijklmnopqrstuvwxyz',
                    uuid = new Array(36), rnd=0, r;
                for (let i = 0; i < 36; i++) {
                    if (i===14) {
                        uuid[i] = '4';
                    } else {
                        if (rnd <= 0x02) rnd = 0x2000000 + (Math.random()*0x1000000)|0;
                        r = rnd & 0xf;
                        rnd = rnd >> 4;
                        uuid[i] = chars[(i === 19) ? (r & 0x3) | 0x8 : r];
                    }
                }
                return uuid.join('');
            },
            //获取二维码
            qrCode () {
                document.querySelector('#qrCode').innerHTML = '';
                new QRCode('qrCode', {
                    width: 160,
                    height: 160, // 高度
                    text: 'http://www.yidayuntu.com?type=login&lgtoken='+this.uuid
                })
                clearInterval(intervalId);
                intervalId = setInterval(()=>{
                    this.getAppStatus();
                }, 2000);
            },
            // 账号登陆/密码登陆切换事件
            changeLogin(){
                this.activeName = this.activeName ==='0' ? '1' : '0';
                this.$nextTick(()=>{
                    if(this.activeName === '0'){
                        this.addCode()
                    }else {
                        clearInterval(intervalId);
                    }
                })
            },
            getAppStatus(){
                api.http({
                    url: '/uaa_user/qrcodelogin',
                    params:{
                        lgtoken: this.uuid
                    }
                }).then(res=>{
                    if(res.code == 200){
                        if(res.data.code == 1){//APP已扫码
                            console.log(res.data.imgList,'已经扫码')
                            this.successCode = true
                            this.sancCode = false
                        }else if(res.data.code == 2){//用户确认登录
                            clearInterval(intervalId);
                            this.parseLogin(res.data.jwt)
                        }else if(!res.data.aBoolean){//二维码失效
                            this.successCode = false
                            this.sancCode = false
                            this.loserCode = true
                            clearInterval(intervalId);
                        }
                    }
                })
            },
            //跳转到登录
            toLogin(){
                this.$router.push('/login')
                window.location.reload()
            },
            //重置密码
            resetPassword(){
                api.http({
                    url:'/uaa_user/resetPassword',
                    type:'post',
                    params:{
                        email:this.email,
                        mobile:this.ruleForm.phone,
                        newPassword:this.ruleForm.userPassword1,
                        confirmPassword:this.ruleForm.userPassword2
                    }
                }).then(res => {
                    if(res.code === 200){
                        this.mailboxCode = false
                        this.findSuccess = true
                        setTimeout(()=>{
                            this.toLogin()
                        },3000)
                    }else {
                        this.statusType = 'error'
                        this.errorMsg = res.message
                    }
                })
            },
            //校验密码
            checkPasswordConfirm(type){
                if(this.ruleForm.userPassword1 && this.ruleForm.userPassword2){
                    if(this.ruleForm.userPassword1 !== this.ruleForm.userPassword2){
                        this.statusType = 'error'
                        this.errorMsg = '两次密码不一致！'
                        this.isCheckMap.userPassword1 = true
                        this.isCheckMap.userPassword2 = true
                    }else {
                        this.errorMsg = ''
                        this.isCheckMap.userPassword1 = false
                        this.isCheckMap.userPassword2 = false
                    }
                }else {
                    if(type){
                        if(!this.ruleForm.userPassword1){
                            this.isCheckMap.userPassword1 = true
                            this.statusType = 'error'
                            this.errorMsg = '密码不可为空！'
                        }
                    }else {
                        if(!this.ruleForm.userPassword2){
                            this.isCheckMap.userPassword2 = true
                            this.statusType = 'error'
                            this.errorMsg = '密码不可为空！'
                        }
                    }
                }
            },
            // 发送验证码
            getPhoneCode(){
                api.http({
                    url:'/uaa_user/shortMessageRetrievingPassword',
                    type:'post',
                    params:{
                        mobile:this.ruleForm.phone
                    }
                }).then(res => {
                    if(res.code === 200){
                        this.showBtnText = '60s'
                        let setTimeoutId
                        this.countDown = true
                        const changeText = (i)=>{
                            setTimeoutId = setTimeout(()=>{
                                this.showBtnText = `${i}s`
                                i = i - 1
                                if(i > 0){
                                    changeText(i)
                                }else {
                                    this.countDown = false
                                    this.showBtnText = '重新获取'
                                    clearTimeout(setTimeoutId)
                                }
                            },1000)
                        }
                        changeText(59)
                    }else {
                        this.statusType = 'error'
                        this.errorMsg = res.message
                    }
                }).catch(()=>{
                    this.statusType = 'error'
                    this.errorMsg = '验证手机号失败！'
                })
            },
            //手机找回密码，下一步
            phoneNext(){
                this.isInActive = true
                api.http({
                    url:'/uaa_user/shortMessageVerificationCode',
                    params:{
                        mobile:this.ruleForm.phone,
                        captcha:this.ruleForm.code,
                        NECaptchaValidate:this.validate
                    },
                    type:'post'
                }).then(res => {
                    this.isInActive = false
                    if(res.code === 200){
                        this.formTow = false;
                    }else {
                        this.statusType = 'error'
                        this.errorMsg = res.message
                    }
                }).catch(()=>{
                    this.statusType = 'error'
                    this.errorMsg = '发送失败！'
                })
            },
            //登录成功，项目初始化
            initProjects(callback){
                let url = '/security_v2/user/projects';
                api.http({
                    url: url
                }).then(res=>{
                    if(res.code==200){
                        access.storage("city", res.data.projectVO.city);
                        access.storage("projectId", res.data.projectVO.id);
                        access.storage("projectName", res.data.projectVO.projectName);
                        access.storage("projectCode", res.data.projectVO.projectCode);
                        access.storage("leaseOrSale", res.data.projectVO.leaseOrSale);
                        typeof callback === 'function' && callback();
                    }
                })
            },
            //登录成功
            parseLogin(res){
                //新版本
                access.session('managerNo',res.managerNo)
                access.session('userNo',{userNo: res.managerNo});
                access.session('token', {token: res.access_token});
                access.session('refresh_token', {token: res.refresh_token});
                access.session('userId',{userId: res.userid});
                access.session('userName',{userName: this.ruleForm.userName});
                access.session('realName',{realName: res.realName});
                access.session('companyId',{companyId: res.companyId});
                access.session('managerType',{managerType: res.managerType});
                //增加项目id初始化
                const fn = ()=>{
                    this.$emit('login', this.$router.currentRoute.query.from);
                }
                this.initProjects(fn)
            },
            //登录
            loginSubmit(){
                if(!this.ruleForm.userName || !this.ruleForm.userPassword){
                    return
                }
                const val = {
                    grant_type: 'password',
                    username: this.ruleForm.userName,
                    password: this.ruleForm.userPassword
                };
                this.loginText = '登录中...'
                api.login(val).then(res => {
                    if(res.access_token) {
                        this.parseLogin(res);
                    }else{
                        this.loginText = '登录'
                        this.statusType = 'error'
                        this.errorMsg = '用户名密码错误！'
                    }
                }).catch(error => {
                    this.loginText = '登录'
                    if (error.response) {
                        if (error.response.status == "401") {
                            this.statusType = 'error'
                            this.errorMsg = error.response.data.error_description || '登录失败'
                        }
                    } else if (error.request) {
                        console.log(error.request);
                    } else {
                        console.log('Error', error.message);
                    }
                })
            },
            //验证邮箱
            checkEmail(){
                if(!this.ruleForm.email){
                    this.statusType = 'error'
                    this.errorMsg = '邮箱不可为空！'
                    this.isCheckMap.email = true
                    return false
                }else {
                    const TEL_REGEXP = /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/;
                    if(!TEL_REGEXP.test(this.ruleForm.email)){
                        this.statusType = 'error'
                        this.errorMsg = '邮箱号格式不正确！'
                        this.isCheckMap.email = true
                        return false
                    }else {
                        this.errorMsg = ''
                        this.isCheckMap.email = false
                        return true
                    }
                }
            },
            // 发送邮件
            sendEmail(){
                this.statusType = 'warn'
                this.isInActive = true
                this.errorMsg = '发送中....'
                api.http({
                    url:'/uaa_user/sendMailboxVerification',
                    params:{
                        email:this.ruleForm.email,
                        publicPath:window.location.protocol + '//' + window.location.host
                    },
                    type:'post'
                }).then(res => {
                    this.isInActive = false
                    if(res.code === 200){
                        this.statusType = 'success'
                        this.errorMsg = res.message
                    }else {
                        this.statusType = 'error'
                        this.errorMsg = res.message
                    }
                }).catch(()=>{
                    this.isInActive = false
                    this.statusType = 'warning'
                    this.errorMsg = '发送失败！'
                })
            },
            checkPhone(){
                if(!this.ruleForm.phone){
                    this.statusType = 'error'
                    this.errorMsg = '手机号不可为空！'
                    this.isCheckMap.phone = true
                    return false
                }else {
                    const TEL_REGEXP = /^1\d{10}$/;
                    if(!TEL_REGEXP.test(this.ruleForm.phone)){
                        this.statusType = 'error'
                        this.errorMsg = '手机号格式不正确！'
                        this.isCheckMap.phone = true
                        return false
                    }else {
                        this.errorMsg = ''
                        this.isCheckMap.phone = false
                        return true
                    }
                }
            },
            checkUserName(){
                if(!this.ruleForm.userName){
                    this.statusType = 'error'
                    this.errorMsg = '用户名不可为空！'
                    this.isCheckMap.userName = true
                    return false
                }else {
                    this.errorMsg = ''
                    this.isCheckMap.userName = false
                    return true
                }
            },
            checkPassword(){
                if(!this.ruleForm.userPassword){
                    this.statusType = 'error'
                    this.errorMsg = '密码不可为空！'
                    this.isCheckMap.password = true
                    return false
                }else {
                    this.errorMsg = ''
                    this.isCheckMap.password = false
                    return true
                }
            },
            checkCode(){
                if(!this.ruleForm.code){
                    this.statusType = 'error'
                    this.errorMsg = '验证码不可为空！'
                    this.isCheckMap.code = true
                    return false
                }else {
                    this.errorMsg = ''
                    this.isCheckMap.code = false
                    return true
                }
            },
            checkEmailReset(){
                api.http({
                    url:'/uaa_user/mailConnection',
                    params:{
                        email:this.email,
                        sid:this.sid
                    },
                    type:'post'
                }).then(res => {
                    if(res.code === 200){
                        this.isFindPassword = false
                        this.mailboxCode = true
                        this.findSuccess = true
                    }else {
                        this.isFindPassword = false
                        this.findSuccess = false
                        this.num = 1
                        this.statusType = 'error'
                        this.errorMsg = res.message
                    }
                }).catch(()=>{
                    this.isFindPassword = false
                    this.findSuccess = false
                    this.num = 1
                    this.statusType = 'warning'
                    this.errorMsg = '邮箱校验失败，请重新发送邮件！'
                })
            },
            // 忘记密码事件
            goPassword(){
                this.errorMsg = ""
                this.isFindPassword = false;
                this.$nextTick(()=>{
                    this.initCap()
                })
            },
            goBtn(index){
                this.errorMsg = ""
                this.num = index;
                if(!this.num){
                    this.ruleForm.phone = ''
                    this.ruleForm.code = ''
                    this.isCheckMap.phone = true
                    this.isCheckMap.code = true
                    this.$nextTick(()=>{
                        this.validate = ''
                        this.isCheckMap.slide = true
                        this.initCap()
                        this.ruleForm = {}
                    })
                }else {
                    this.ruleForm.email = ''
                    this.isCheckMap.email = true
                }
            },
            // 返回登陆
            goCode(){
                this.errorMsg = ""
                this.$router.push('/login')
                this.isFindPassword = true
            },
            // 返回上一步
            goOne(){
                this.errorMsg = ""
                this.formTow = true
            },
            initCap(){
                //滑动验证码
                initNECaptcha && initNECaptcha({
                    captchaId: '2e737a7bcb5a4b33ab90645a919cd6b8',
                    element: '#slide',
                    mode: 'float',
                    width: 308,
                    onReady: function (instance) {
                        // 验证码一切准备就绪，此时可正常使用验证码的相关功能
                    },
                    onVerify: (err, data) => {
                        if(err){
                            //验证失败
                            this.isCheckMap.slide = true
                        }
                        if(data){
                            this.validate = data.validate
                            this.isCheckMap.slide = false
                            this.errorMsg = ''
                        }else {
                            this.errorMsg = '请重新滑动滑块！'
                            this.isCheckMap.slide = true
                            this.validate = ''
                        }
                    }
                }, function onload (instance) {
                    console.log(instance,'12')
                    // 初始化成功
                }, function onerror (err) {
                    // 验证码初始化失败处理逻辑，例如：提示用户点击按钮重新初始化
                })
            }
        },
        computed:{
            loginType(){
                return this.activeName == 0?'账号登录':'扫码登录';
            },
            sid(){
                return this.$route.query.sid
            },
            email(){
                return this.$route.query.email
            }
        },
        mounted() {
            initThree()
            this.$nextTick(()=>{
                this.$refs.inputName.focus();
            })
            flashChart = new FlashChart('flash-dome')
            flashChart.initFun()
        },
        created: function () {
            sessionStorage.clear();
            //这里判断是否是邮箱终止密码的
            if(this.sid){
                //校验邮箱这个链接
                this.checkEmailReset()
            }
        }
    }
</script>

<style lang="scss" scoped>
    #flash-dome{
        position: absolute;
        left: 0;
        top:0;
    }
    #flash-dome-hide{
        display: none;
    }
    #canvas-frame {
        border: none;
        cursor: pointer;
        width: 700px;
        height: 500px;
        z-index: 1;
        background-color: transparent;
    }

    @keyframes animationX {
        0% {
            left: 0;
        }
        100% {
            left: 709px;
        }
    }

    @keyframes animationY {
        0% {
            top: 0;
        }
        100% {
            top: 207px;
        }
    }

    @keyframes loop {
        0% {
            transform: rotateY(0)
        }
        100% {
            transform: rotateY(-360deg)
        }
    }
    #slide{
        margin-top: 20px;
    }
    .login1 {
        .flag{
            position: absolute;
            width: 1px;
            height: 19px;
            background:rgba(237,237,237,1);
        }
        .show-error{
            color: #F56C6C;
            font-size: 16px;
            &.success{
                color: #1ab394;
            }
            &.warn{
                color: #e6a23c;
            }
        }
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        padding: 30px 5.2% 30px 30px;
        box-sizing: border-box;
        background-color: #0A0E22;
        overflow: hidden;
        .logo {
            width: 160px;
            height: 50px;
            position: absolute;
            z-index: 5;
        }
    }

    .main-content {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%;
        .left-content {
            flex: auto;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            position: relative;
            .chart-content {
                height: 553px;
                display: flex;
                justify-content: center;
                align-items: center;
                z-index: 2;
                position: relative;
                .circle {
                    width: 707px;
                    border-radius: 50%;
                    position: absolute;
                    top: 27%;
                    height: 205px;
                    z-index: 2;
                    transform: rotateZ(7deg);
                    border: 1px solid #0EA9FF;
                    .mask{
                        width: 50px;
                        height: 50px;
                        position: absolute;
                        z-index: 5;
                        background-color: red;
                    }
                    .maskL{
                        left: 63px;
                        top: 18px;
                    }
                    .maskR{
                        right: 61px;
                        top:10px;
                    }
                }
                .bottom-box {
                    width: 1239px;
                    height: 285px;
                    position: absolute;
                    background-image: url("../../assets/login/circle-bottom.png");
                    background-size: contain;
                    background-repeat: no-repeat;
                    background-position: center;
                    bottom: -191px;
                }
                .box-list {
                    left: 350px;
                    top: 10px;
                    .box-item {
                        position: absolute;
                        left: 0;
                        width: 54px;
                        height: 54px;
                        margin-left: -27px;
                        margin-top: -27px;
                        color: #0EA9FF;
                        border-radius: 50%;
                        line-height: 54px;
                        text-align: center;
                        background-color: #0A0E22;
                        font-size: 16px;
                        display: inline-block;
                        border: 1px solid #0EA9FF;
                    }
                    .box-item:nth-child(1) {
                        animation: animationX 10s cubic-bezier(0.36, 0, 0.64, 1) -5s infinite alternate-reverse,
                        animationY 10s cubic-bezier(0.36, 0, 0.64, 1) 0s infinite alternate;
                    }
                    .box-item:nth-child(2) {
                        animation: animationX 10s cubic-bezier(0.36, 0, 0.64, 1) -2s infinite alternate-reverse,
                        animationY 10s cubic-bezier(0.36, 0, 0.64, 1) 3s infinite alternate;
                    }
                    .box-item:nth-child(3) {
                        animation: animationX 10s cubic-bezier(0.36, 0, 0.64, 1) 1s infinite alternate-reverse,
                        animationY 10s cubic-bezier(0.36, 0, 0.64, 1) 6s infinite alternate;
                    }
                    .box-item:nth-child(4) {
                        animation: animationX 10s cubic-bezier(0.36, 0, 0.64, 1) 4s infinite alternate-reverse,
                        animationY 10s cubic-bezier(0.36, 0, 0.64, 1) 9s infinite alternate;
                    }
                }
            }
        }
        .right-content {
            flex: 0 0 404px;
            width: 404px;
            height: 539px;
            box-sizing: border-box;
            background-color: #242E47;
            position: relative;
            z-index: 100;
            .icp{
                .elTow{
                    font-size: 12px;
                    font-family:PingFangSC-Regular;
                    font-weight:400;
                    color:rgba(175,175,175,.5);
                    position: absolute;
                    left: 94px;
                    bottom: 16px;
                    a{
                        color:rgba(175,175,175,.5);
                        // text-decoration: underline;
                    }
                }
            }
            .forgetBtn{
                position: absolute;
            }
            .login_con{
                padding: 110px 49px 0 49px;
                .tip{
                    width: 62px;
                    height: 24px;
                    padding-right: 8px;
                    position: absolute;
                    right: 56px;
                    top: 7px;
                    font-size:8px;
                    font-family:PingFangSC-Regular;
                    font-weight:400;
                    color: #76c5f7;
                    text-align: center;
                    line-height: 24px;
                    background: url(../../assets/bubble-img.png) center center no-repeat;
                    background-size:100% 100%;
                }
                .codeC,
                .loginC{
                    cursor: pointer;
                    position: absolute;
                    right: 9px;
                    top: 7px;
                    width: 40px;
                    height: 40px;
                    background: url(../../assets/code-img.png) center center no-repeat;
                }
                .loginC{
                    background-image: url(../../assets/computer-img.png);
                    width:30px;
                    height: 30px;
                    background-size:100% 100%;
                }
                .login_type{
                    .show-error{
                        margin-top: -9px;
                        margin-bottom: 8px;
                        font-size: 14px;
                    }
                    .please-code,.success-code,.loser-code{
                        .code-title{
                            margin-top: 20px;
                            font-size: 12px;
                            text-align: center;
                            font-family:PingFangSC-Regular;
                            font-weight:400;
                            color:rgba(175,175,175,.5);
                        }
                        .qrCode{
                            width: 180px;
                            height: 180px;
                            margin: 26px auto 0 auto;
                            background:rgba(255,255,255,1);
                            border-radius:4px;
                            padding: 10px;
                            box-sizing: border-box;
                        }
                        .success-code{
                            width: 128px;
                            height: 128px;
                            margin: 78px auto 0 auto;
                            background:rgba(255,255,255,1);
                            border-radius:4px;
                            padding: 6px;
                            box-sizing: border-box;
                            img{
                                width: 100%;
                                height: 100%;
                            }
                        }
                        .new-btn{
                            text-align: center;
                            margin-top: 31px;
                            .loser-btn{
                                width:75px;
                                height:26px;
                                border-radius:4px;
                                border:1px solid rgba(255,255,255,1);
                                background: #242E47;
                                font-size: 14px;
                                color: #fff;
                                line-height: 0px;
                            }
                            .loser-btn:hover{
                                border: 1px solid #1880DE;
                                color:#1880DE;
                            }
                        }
                        .connect{
                            font-size:12px;
                            font-family:PingFangSC-Regular;
                            font-weight:400;
                            color:rgba(255,255,255,.8);
                            position: absolute;
                            top: 104px;
                            left: 42%;
                        }
                        .code_login{
                            .img{
                                display: inline-block;
                                width: 18px;
                                height: 18px;
                                vertical-align: -2px;
                            }
                            text-align: center;
                            line-height: 30px;
                            height: 30px;
                            width: 100%;
                            margin: 0 auto;
                            margin-top: 27px;
                            color: #FFFFFF;
                            font-size: 20px;
                        }
                    }
                    .title{
                        .title-z{
                            margin-bottom: 8px;
                        }
                        .title-z,.title-b{
                            height: 36px;
                            font-size:36px;
                            font-family:PingFangSC-Light;
                            font-weight:300;
                            color:rgba(255,255,255,1);
                            line-height:34px;
                        }
                    }
                }
                .elForm{
                    .elRow{
                        text-align: center;
                        font-family:PingFangSC-Regular;
                        font-weight:400;
                        margin-top: 32px;
                        font-size: 14px;
                        span{
                            color:rgba(231,231,231,.5);
                            cursor: pointer;
                            &:hover{
                                color:rgba(231,231,231,1); ;
                            }
                        }
                        span:hover{
                            color: #fff;
                        }
                    }
                    .user-name{
                        position: relative;
                        .flag{
                            top: 12px;
                            left: 41px;
                        }
                        .icon-username{
                            position: absolute;
                            left: 14px;
                            top: 13px;
                            background-image: url("../../assets/login/icon-username.png");
                            background-size: contain;
                            width: 14px;
                            height: 16px;
                            display: inline-block;
                        }
                    }
                    .pass-word{
                        .flag{
                            top: 20px;
                            left: 41px;
                        }
                        position: relative;
                        .icon-password{
                            position: absolute;
                            left: 16px;
                            top: 20px;
                            width: 14px;
                            height: 16px;
                            display: inline-block;
                            background-image: url("../../assets/login/icon-password.png");
                            background-size: contain;
                        }
                    }
                    .btn{
                        width: 100%;
                        height: 40px;
                        margin-top:16px;
                        color:rgba(255,255,255,.5);
                        border: none;
                        font-size: 18px;
                        background:rgba(100,135,167,1);
                        border-radius:4px;
                    }
                    .btnColr:hover{
                        background: #1880DE;
                        color:#fff;
                    }
                }
            }
            .login_password{
                .show-error{
                    position: absolute;
                    top: 112px;
                    left: 48px;
                    font-size: 14px;
                }
                .reset-pass{
                    padding: 154px 132px 0 132px;
                    .pass-title{
                        margin-top: 30px;
                        font-size:22px;
                        color:rgba(255,255,255,.8);
                        text-align: center;
                    }
                    .elRow{
                        text-align: center;
                        margin-top: 24px;
                        color:rgba(175,175,175,1);
                        font-size: 12px;
                        .return-login{
                            color: #1880DE;
                            &:active{
                                color: #1880DE;
                            }
                            &:hover{
                                color: #1880DE;
                            }
                        }
                    }
                }
                .mailbox-password{
                    .title-header{
                        text-align: center;
                        height: 60px;
                        border-bottom:1px solid rgba(255,255,255,.3);
                        font-size:20px;
                        color:#fff;
                        line-height: 60px;
                    }
                    .elForm{
                        padding: 60px 48px 0 48px;
                        box-sizing: border-box;
                        .btn{
                            width: 100%;
                            height: 40px;
                            margin-top:16px;
                            color:rgba(255,255,255,.5);
                            border: none;
                            font-size: 16px;
                            background:rgba(100,135,167,1);
                            border-radius:4px;
                        }
                        .btnColr:hover{
                            background: #1880DE;
                            color:#fff;
                        }
                        .elRow{
                            margin-top: 16px;
                            text-align: center;
                            .return-login{
                                cursor: pointer;
                                font-size:12px;
                                font-family:PingFangSC-Regular;
                                font-weight:400;
                                color:rgba(255,255,255,.6);
                                text-decoration: underline;
                            }
                        }
                    }
                }
                .header-btn{
                    position: relative;
                    .border{
                        width:1px;
                        height:24px;
                        background:rgba(255,255,255,.3);
                        display:inline-block;
                        content:'';
                        position:absolute;
                        top:20px;
                        left:202px
                    }
                    .list{
                        height: 60px;
                        border-bottom:1px solid rgba(255,255,255,.3);
                        .list-item{
                            // padding: 18px 0 15px 0;
                            line-height: 60px;
                            margin-top: -1px;
                            box-sizing: border-box;
                            float: left;
                            font-size:20px;
                            font-family:PingFangSC-Regular;
                            font-weight:400;
                            color:rgba(255,255,255,.3);
                            cursor: pointer;
                        }
                        .list-item:hover{
                            color: #fff;
                        }
                        .active{
                            border-bottom: 2px solid #1880DE;
                            color: #FFFFFF;
                        }
                        .list-item:nth-child(1){
                            margin-left: 60px;
                        }
                        .list-item:nth-child(2){
                            float: right;
                            margin-right: 60px;
                        }
                    }
                }
                .content{
                    padding: 80px 48px 0 48px;
                    .elForm{
                        .pass-word{
                            .obtain{
                                float: right;
                                margin-top: 16px;
                                text-align: center;
                                width: 100px;
                                height: 40px;
                                line-height: 40px;
                                font-size: 16px;
                                background:rgba(255,255,255,1);
                                border-radius:4px;
                                color:#D1D1D1;
                                cursor: pointer;
                                border: 0;
                            }
                            .obtain:hover{
                                color: #666;
                            }
                        }
                        .slide{
                            font-size:14px;
                            font-family:PingFangSC-Regular;
                            font-weight:400;
                            color:rgba(102,102,102,.6);
                            width: 100%;
                            text-align: center;
                            margin-top: 16px;
                            line-height: 35px;
                            height:35px;
                            background:#fff;
                            border-radius:4px;
                        }
                        .btn{
                            width: 100%;
                            height: 40px;
                            margin-top:16px;
                            color:rgba(255,255,255,.5);
                            border: none;
                            font-size: 18px;
                            background:rgba(100,135,167,1);
                            border-radius:4px;
                        }
                        .btnColr:hover{
                            background: #1880DE;
                            color:#fff;
                        }
                        .elRow{
                            margin-top: 16px;
                            text-align: center;
                            .return-login{
                                cursor: pointer;
                                font-size:12px;
                                font-family:PingFangSC-Regular;
                                font-weight:400;
                                color:rgba(255,255,255,.6);
                                text-decoration: underline;
                            }
                            .return-login:hover{
                                color: #fff;
                            }
                        }
                    }
                }
            }
        }
    }
</style>
<style lang="scss">
.login1{
    .login_con{
        .el-input__inner {
            padding-left:56px;
        }
    }

    .el-input__inner {
        height: 40px;
        line-height: 40px;
        border-radius: 3px;
        color: #333333;
        font-size:16px;
    }
    .yidun_tips__text{
        color:#666;
    }
    .el-loading-mask{
        border-radius:4px;
    }
    .yidun_slide_indicator{
        border-color: #fff !important;
        background-color: #fff !important;
    }
}

</style>

