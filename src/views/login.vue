<style lang="less">
    @import './login.less';
</style>

<template>
    <div class="login" @keydown.enter="handleSubmit">
        <div class="login-con">
            <Card :bordered="false">
                <p slot="title">
                    <Icon type="log-in"></Icon>
                    欢迎登录天马人事系统
                </p>
                <div class="form-con">
                    <Form ref="loginForm" :model="form" :rules="rules">
                        <FormItem prop="userName">
                            <Input v-model="form.userName" placeholder="请输入用户名" size="large">
                                <span slot="prepend">
                                    <Icon :size="16" type="person"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                        <FormItem prop="passWord">
                            <Input type="password" v-model="form.passWord" placeholder="请输入密码" size="large">
                                <span slot="prepend">
                                    <Icon :size="14" type="locked"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                        <FormItem prop="code">
                            <Input v-model="form.code" placeholder="请输入验证码" size="large">
                            <span slot="prepend">
                                    <Icon :size="16" type="android-checkmark-circle"></Icon>
                                </span>
                            </Input>
                            <img src="/oa/login/geneCode" id="validate-code-img" @click="getCode" ref="codeImg" style="cursor: pointer;"/>
                        </FormItem>
                        <FormItem>
                            <Button @click="handleSubmit"
                                    type="primary"
                                    :loading="loading"
                                    long size="large">
                                <span v-if="!loading">登录</span>
                                <span v-else>登录中...</span>
                            </Button>
                        </FormItem>
                    </Form>
                </div>
            </Card>
        </div>
    </div>
</template>

<script>
import Cookies from 'js-cookie';
import {appRouter, page404} from '@/router/router';
import util from '@/libs/util';
export default {
    data () {
        return {
            loading: false,
            form: {
                userName: '',
                passWord: '',
                code: ''
            },
            rules: {
                userName: [
                    { required: true, message: '账号不能为空', trigger: 'blur' }
                ],
                passWord: [
                    { required: true, message: '密码不能为空', trigger: 'blur' }
                ],
                code: [
                    {required: true, message: '验证码不能为空', trigger: 'blur'}
                ]
            }
        };
    },
    activated() {
    },
    mounted() {
        this.getCode();
    },
    methods: {
        getCode() {
            let time = +(new Date());
            let picUrl = '/oa/login/geneCode?t=' + time;
            this.$refs.codeImg.setAttribute('src', picUrl);
        },
        handleSubmit () {
            this.$refs.loginForm.validate((valid) => {
                if (valid) {
                    this.loading = true;
                    this.$http.post('/login/login', this.form).then((res) => {
                        if (res.success) {
                            let allUserInfo = res.data[0];
                            let userInfo = {};
                            userInfo.userName = allUserInfo.realname;
                            userInfo.organizeName = allUserInfo.organizename;
                            userInfo.postName = allUserInfo.postname;
                            userInfo.tmCoin = allUserInfo.tm_coin;
                            Cookies.set('user', this.form.userName);
                            Cookies.set('userInfo', userInfo);
                            Cookies.set('password', this.form.password);
                            this.$store.commit('setUserInfo', userInfo);
                            this.$store.commit('setAvator', 'https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3448484253,3685836170&fm=27&gp=0.jpg');
                            Cookies.set('token', '1010101010');
                            this.$router.push({
                                name: 'home_index'
                            });
                        }
                    }).finally(() => {
                        this.loading = false;
                    });
                }
            });
        }
    }
};
</script>

<style>

</style>
