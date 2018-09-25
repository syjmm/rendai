<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i>工作站</el-breadcrumb-item>
                <el-breadcrumb-item>添加工作站</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box">
                <el-form ref="ruleForm" :model="ruleForm" :rules="rules" label-width="120px">
                    <el-form-item label="工作站名称" prop="name">
                        <el-input type="text" v-model="ruleForm.name"></el-input>
                    </el-form-item>
                    <el-form-item label="工作站图片" prop="imgSrc">
                        <div v-show="imgVisible">
                            <img :src="GLOBAL.BASE_URL + ruleForm.imgSrc" style="width:300px;height: 200px">
                        </div>
                        <div  class="el-upload__tip">只能上传jpg/png文件</div>
                        <el-upload
                                class="upload-demo"
                                action=""
                                :before-upload="beforeUpload"
                                :auto-upload="true">
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-form-item>
                    <el-form-item label="工作站描述" prop="desc">
                        <el-input type="textarea" v-model="ruleForm.desc"></el-input>
                    </el-form-item>
                    <el-form-item label="工作站联系方式" prop="phone">
                        <el-input type="text" v-model="ruleForm.phone"></el-input>
                    </el-form-item>
                    <el-form-item label="工作站地址" prop="address">
                        <el-input type="text" v-model="ruleForm.address"></el-input>
                    </el-form-item>

                    <el-form-item>
                        <el-button type="primary" @click="onSubmit('ruleForm')">提交</el-button>
                        <el-button @click="cancel">取消</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </div>

    </div>
</template>

<script>
    import qs from 'qs'
    export default {
        name: 'baseform',
        data: function(){
            return {

                dialogVisible: false,
                imgVisible:false,
                options:[],

                ruleForm: {
                    name: '',
                    phone:'',
                    address:'',
                    desc:'',
                    imgSrc:'',
                },
                rules:{
                    name :[{required: true, message: '请输入工作站名称', trigger: 'blur'}],
                    phone :[{required: true, message: '请输入工作站联系方式', trigger: 'blur'}],
                    address :[{required: true, message: '请输入工作站地址', trigger: 'blur'}],
                    desc :[{required: true, message: '请输入工作站简介', trigger: 'blur'}],
                    imgSrc :[{required: true, message: '请上传工作站图片', trigger: 'blur'}],
                }
            }
        },
        created(){
            this.uid=this.$route.query.uid;

        },

        methods: {

            onSubmit(formName) {
                let token = localStorage.getItem('ms_token');
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.$axios.post('/manager/work_stations',
                            qs.stringify({
                                avatar: this.ruleForm.imgSrc,
                                description: this.ruleForm.desc,
                                telephone: this.ruleForm.phone,
                                name: this.ruleForm.name,
                                address: this.ruleForm.address,
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
                                this.$router.push({path: '/wtable'})
                            } else {
                                this.$message.error('提交失败！');
                            }
                        })
                    }})

            },
            beforeUpload(file){
                let token = localStorage.getItem('ms_token');
                let fd = new FormData();
                fd.append('file',file);//传文件
                fd.append('uid',this.uid);//传其他参数
                let that=this;
                this.$axios.post('/manager/work_stations/upload',fd,{
                    headers: {
                        'Authorization':token,
                    }
                }).then(function(res){
                    that.ruleForm.imgSrc=res.data.data.url;
                    that.$message.success('上传成功！');
                })
                this.imgVisible = true;
            },
            newSubmitForm(){//确定上传
                this.$refs.newupload.submit();
            },
            cancel(){
                this.$router.push({path:'/wtable'})
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