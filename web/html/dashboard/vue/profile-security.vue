<template>
<div class="db-security form">
    <h3 class="title">安全设置</h3><br/><br/>
    <div class="item">
        <div class="col-sm-2">更换邮箱</div>
        <div class="col-sm-6">{{user.email}}</div>
        <div class="col-sm-2"><input type="button" value="修改" v-on:click="emailModal=true" class="btn btn-info"></div>
    </div>
    <div class="item">
        <div class="col-sm-2">更换密码</div>
        <div class="col-sm-6">建议您3个月修改一次</div>
        <div class="col-sm-2"><input type="button" value="修改" v-on:click="passwordModal=true" class="btn btn-info"></div>
    </div>

</div>
    <div class="modal" v-cloak v-if="emailModal">
        <div class="modal-header">
            <i class="iconfont icon-close modal-close" v-on:click="emailModal=false"></i>
        </div>
        <div class="modal-content">
            <validator name="ef">
            <div class="modal-layout2 form">
                <p class="title">修改邮箱</p>
                <div class="item">
                    <div class="col-sm-12">
                        <input type="text" initial="off" v-model="email" tabindex="1" maxlength="45" v-validate:email="['required','email','newEmail']"  class="k1 text" placeholder="请输入新的邮箱"/>
                        <p v-cloak class="tip" v-if="$ef.email.invalid">{{$ef.email.errors[0].message}}</p>
                    </div>
                </div>

                <div class="item">
                    <div class="col-sm-8">
                        <input type="text" class="text k1" tabindex="2" initial="off" v-model="captcha" v-validate:captcha="['required']" placeholder="请输入验证码">
                        <p v-cloak class="tip" v-if="$ef.captcha.invalid">{{$ef.captcha.errors[0].message}}</p>
                    </div>
                    <div class="col-sm-4"><button class="btn btn-info k1" v-on:click="sendCaptcha">{{time>0?(time+'s'):'获取验证码'}}</button></div>
                </div>
                <div>
                    <div class="ta-c actions">
                        <button class="btn btn-info btn-lg" tabindex="4" v-on:click="emailModal=false">取消
                        </button>
                        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                        <button class="btn btn-danger btn-lg" v-on:click="ok" tabindex="3">确定</button>
                    </div>
                </div>
            </div>
            </validator>
        </div>
    </div>
    <div class="modal" v-cloak v-if="passwordModal">
        <div class="modal-header">
            <i class="iconfont icon-close modal-close" v-on:click="passwordModal=false"></i>
        </div>
        <div class="modal-content">
            <validator name="pf">
                <div class="modal-layout2 form">
                    <p class="title">修改密码</p>
                    <div class="item">
                        <div class="col-sm-12">
                            <input type="password" initial="off" v-model="password" tabindex="1" maxlength="45" v-validate:password="{required:true,minlength:{rule:6,message:'请输入最少6位密码'}}"  class="k1 text" placeholder="请输入新的密码"/>
                            <p v-cloak class="tip" v-if="$pf.password.invalid">{{$pf.password.errors[0].message}}</p>
                        </div>
                    </div>
                     <div class="item">
                        <div class="col-sm-12">
                            <input type="password" initial="off" v-model="repassword" tabindex="1" maxlength="45" v-validate:repassword="{required:true,minlength:{rule:6,message:'请输入最少6位密码'}}"  class="k1 text" placeholder="请确认密码"/>
                            <p v-cloak class="tip" v-if="$pf.repassword.invalid">{{$pf.repassword.errors[0].message}}</p>
                        </div>
                    </div>


                    <div>
                        <div class="ta-c actions">
                            <button class="btn btn-info btn-lg" tabindex="4" v-on:click="passwordModal=false">取消
                            </button>
                            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                            <button class="btn btn-danger btn-lg" v-bind:disabled="$pf.invalid" v-on:click="passwordOk" tabindex="3">确定</button>
                        </div>
                    </div>
                </div>
            </validator>
        </div>
    </div>

</template>
<script>
    import utils from '../../src/utils';
    export default {
        data: function () {
            return {
                user:utils.user(),
                emailModal:false,
                passwordModal:false,
                password:'',
                repassword:'',
                email:'',
                captcha:'',
                time:0
            }
        },
        validators:{
            newEmail:{
                message:'新邮箱与当前邮箱一样',
                check:function(value){
                    return value != this.vm.user.email;
                }
            }
        },
        methods:{
            sendCaptcha:function(){
                this.$validate('email',true);
                if(this.$ef.email.invalid){
                    return false;
                }
                if(this.time>0){
                    return false;
                }
                var self = this;
                utils.post('/user/email/captcha.json',{email:this.email},function(rs){
                    self.time = 30;
                    var interval = window.setInterval(function(){
                        if(self.time>0){
                            self.time--;
                        }else{
                            window.clearInterval(interval);
                        }
                    },1000);
                });
            },
            ok:function(){
                this.$validate(true);
                if(this.$ef.invalid){
                    return false;
                }
                var self = this;
                utils.post('/user/email/new.json',{email:this.email,code:this.captcha},function(rs){
                    toastr.success('修改成功');
                    self.emailModal=false;
                    self.user.email=self.email;
                    utils.user(self.user);
                });
            },
            passwordOk:function(){
                this.$validate(true);
                if(this.$pf.invalid){
                    return false;
                }
                if(this.password != this.repassword){
                    toastr.error('两次密码不一样');
                    return false;
                }
                var self = this;
                utils.post('/user/password.json',{password:this.password},function(rs){
                    toastr.success('修改成功');
                    self.passwordModal = false;
                });
            }
        }
    }
</script>