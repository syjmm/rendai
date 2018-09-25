<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i>代表</el-breadcrumb-item>
                <el-breadcrumb-item>添加代表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box">
                <el-form ref="form" :model="form" :rules="rules" label-width="120px">
                    <el-form-item label="代表名称" prop="name">
                        <el-input type="text" v-model="form.name"></el-input>
                    </el-form-item>

                    <el-form-item label="代表正面照" prop="imgSrc">
                        <div v-show="imgVisible" >
                            <img :src="GLOBAL.BASE_URL + form.imgSrc" style="width:auto;height: 250px">
                        </div>
                        <div class="el-upload__tip">只能上传jpg/png文件</div>
                        <el-upload
                                class="upload-demo"
                                action=""
                                :before-upload="beforeUpload"
                                accept=".jpg,.jpeg,.png,.gif,.bmp,.pdf,.JPG,.JPEG,.PBG,.GIF,.BMP,.PDF"
                                :auto-upload="true">
                            <el-button size="small" type="primary">点击上传</el-button>

                        </el-upload>

                    </el-form-item>

                    <el-form-item label="代表出生时间" prop="time">
                        <el-date-picker
                                v-model="form.time"
                                type="date"
                                @change="getTime"
                                :picker-options="pickerOptions0"
                        value-format="yyyy-MM-dd"
                                placeholder="选择日期时间">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item label="性别" prop="sex">
                        <el-radio-group v-model="form.sex">
                            <el-radio label="1" value="1">男</el-radio>
                            <el-radio label="0" value="0">女</el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="任职职位" prop="role">
                        <el-select v-model="form.role" placeholder="请选择">
                            <el-option
                                    v-for="item in optionss"
                                    :key="item.uid"
                                    :label="item.name"
                                    :value="item.uid"
                            >
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="所属小组" prop="gourp">
                        <el-select v-model="form.gourp" placeholder="请选择">
                            <el-option
                                    v-for="item in options"
                                    :key="item.uid"
                                    :label="item.name"
                                    :value="item.uid"
                            >
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="联系方式" prop="phone">
                        <el-input type="text" v-model="form.phone"></el-input>
                    </el-form-item>
                    <el-form-item label="代表简介" prop="desc">
                        <el-input type="textarea" v-model="form.desc"></el-input>
                    </el-form-item>


                    <el-form-item label="履职状态" prop="status">
                        <el-select v-model="form.status" placeholder="请选择">
                            <el-option label="在职" value="0"></el-option>
                            <el-option label="往届" value="1"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="onSubmit('form')">提交</el-button>
                        <el-button @click="cancel">取消</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </div>

    </div>
</template>

<script>
    import qs from 'qs'
    import {isvalidPhone} from '../../../config/validate'
    var validPhone=(rule, value,callback)=>{
        if (!value){
            callback(new Error('请输入电话号码'))
        }else  if (!isvalidPhone(value)){
            callback(new Error('请输入正确的11位手机号码'))
        }else {
            callback()
        }
    }
    export default {
        name: 'baseform',
        data: function(){
            return {
                dialogVisible: false,
                options:[],
                optionss:[],
                pickerOptions0: {
                    disabledDate(time) {
                        return time.getTime() > Date.now() - 8.64e7;
                    }
                },
                imgVisible:false,
                form: {
                    name: '',
                    phone:'',
                    rename:'',
                    time:'',
                    sex:'',
                    status:'',
                    gourp:'',
                    desc:'',
                    role:'',
                    imgSrc:'',
                },
                rules:{
                    name :[{required: true, message: '请输入代表名称', trigger: 'blur'}],
                    desc :[{required: true, message: '请输入代表简介', trigger: 'blur'}],
                    time :[{required: true, message: '请选择出生日期', trigger: 'blur'}],
                    sex :[{required: true, message: '请选择性别', trigger: 'blur'}],
                    phone :[{required: true, trigger: 'blur',validator: validPhone}],
                    gourp :[{required: true, message: '请选择所属小组', trigger: 'blur'}],
                    role :[{required: true, message: '请选择任职职位', trigger: 'blur'}],
                    status :[{required: true, message: '请选择履职状态', trigger: 'blur'}],
                    imgSrc :[{required: true, message: '请选择履职状态', trigger: 'blur'}],
                }
            }
        },
        created(){
            this.uid=this.$route.query.uid;
            let token = localStorage.getItem('ms_token');
            this.$axios.get('/manager/npc_member_groups/all',{
                headers: {
                    'Authorization':token,
                }
            }).then(res=>{
                this.options=res.data.data;
            })
            this.$axios.get('/manager/roles/all',{
                headers: {
                    'Authorization':token,
                }
            }).then(res=>{
                console.log(res.data.data)
                this.optionss=res.data.data;
            })
        },

        methods: {

            getTime(val){
                this.form.time=val;
            },
            onSubmit(formName) {
                let token = localStorage.getItem('ms_token');
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.$axios.post('/manager/npc_members',
                            qs.stringify({
                                avatar: this.form.imgSrc,
                                birthday: this.form.time,
                                gender: this.form.sex,
                                group: this.form.gourp,
                                introduction: this.form.desc,
                                mobile: this.form.phone,
                                name: this.form.name,
                                status: this.form.status,
                                role: this.form.role,
                                uid: this.uid
                            }),
                            {
                                headers: {
                                    'Authorization':token,
                                }
                            }).then(res => {
                            // console.log(res)
                            if (res.data.status == 'OK') {
                                this.$message.success('提交成功！');
                                this.$router.push({path: '/rtable'})
                            } else {
                                this.$message.error('提交失败！');
                            }
                        })
                    }})

            },
            beforeUpload(file){

                let fd = new FormData();
                fd.append('file',file);//传文件
                fd.append('uid',this.uid);//传其他参数
                let that=this;
                let token = localStorage.getItem('ms_token');
                this.$axios.post('/manager/npc_members/upload',fd,{
                    headers: {
                        'Authorization':token,
                    }
                }).then(function(res){
                    if(res.data.status == 'OK'){
                        that.form.imgSrc=res.data.data.url;
                        that.$message.success('上传成功！');
                    }else{
                        that.$message.error('只允许上传jpg/png格式！');
                    }

                })
                this.imgVisible = true;
            },
            newSubmitForm(){//确定上传
                this.$refs.newupload.submit();
            },
            cancel(){
                this.$router.push({path:'/rtable'})
            }
        }
    }
</script>
<style>
    .el-upload__tip{
        margin-top: -7px !important;
    }
    .upload-demo .el-upload--text {
        width: 100px !important;
        height: 40px !important;
        border:none !important;
        background: transparent !important;
    }
    .upload-demo .el-upload {
        width: 100px !important;
        height: 40px !important;
        border:none !important;
        background: transparent !important;
    }
    .content-title{
        font-weight: 400;
        line-height: 50px;
        margin: 10px 0;
        font-size: 22px;
        color: #1f2f3d;
    }
    .pre-img{
        width: 100px;
        height: 100px;
        background: #f8f8f8;
        border: 1px solid #eee;
        border-radius: 5px;
    }
    .crop-demo{
        display: flex;
        align-items: flex-end;
    }
    .crop-demo-btn{
        position: relative;
        width: 100px;
        height: 40px;
        line-height: 40px;
        padding: 0 20px;
        margin-left: 30px;
        background-color: #409eff;
        color: #fff;
        font-size: 14px;
        border-radius: 4px;
        box-sizing: border-box;
    }
    .crop-input{
        position: absolute;
        width: 100px;
        height: 40px;
        left: 0;
        top: 0;
        opacity: 0;
        cursor: pointer;
    }
</style>