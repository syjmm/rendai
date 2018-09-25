<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i>通知公告</el-breadcrumb-item>
                <el-breadcrumb-item>添加通知</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box" style="width:700px">
                <el-form ref="form" :model="form" :rules="rules" label-width="110px">
                    <el-input v-model="uid" style="display: none"></el-input>
                    <el-form-item label="通知公告标题" prop="name">
                        <el-input v-model="form.name"></el-input>
                    </el-form-item>
                    <el-form-item label="通知公告内容" prop="content">
                        <el-input type="textarea" v-model="form.content"></el-input>
                    </el-form-item>
                    <el-form-item label="接收代表" prop="receivers">
                        <el-checkbox-group v-model="form.receivers" >
                            <el-checkbox  v-for="item in options":label="item.uid"  :key="item.uid">{{item.name}}</el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                    <el-form-item label="附件" prop="fileList">
                        <el-upload
                                v-model="form.fileList"
                                class="upload-demo"
                                action=""
                                :on-preview="handlePreview"
                                :on-remove="handleRemove"
                                :before-remove="beforeRemove"
                                :before-upload="beforeUpload"
                                :on-exceed="handleExceed"
                                :file-list="fileList">
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
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
    export default {
        name: 'baseform',
        data: function(){
            return {
                uid:'',
                options: [],
                fileList:[],
                filename:'',
                fileListname:'',
                dialogVisible: false,
                form: {
                    desc: '',
                    name:'',
                    receivers:[],
                    content: '',
                },
                rules:{
                    name :[{required: true, message: '请输入通知公告标题', trigger: 'blur'}],
                    content :[{required: true, message: '请输入公告内容', trigger: 'blur'}],
                    receivers :[{required: true, message: '请选择接收代表', trigger: 'blur'}],
                    fileList :[{required: true, message: '请上传附件', trigger: 'blur'}],
                }
            }
        },

        created () {
            this.uid=this.$route.query.uid;
            let token = localStorage.getItem('ms_token');
            this.$axios.get('/manager/npc_members/working',{
                headers: {
                    'Authorization':token,
                }
            }).then(res=>{
                this.options = res.data.data;
            })
        },
        methods: {
            beforeUpload(file){
                let token = localStorage.getItem('ms_token');
                let fd = new FormData();
                fd.append('file',file);//传文件
                fd.append('uid',this.uid);//传其他参数
                let that=this;
                this.$axios.post('/manager/notifications/upload',fd,{
                    headers: {
                        'Authorization':token,
                    }
                }).then(function(res){
                    that.fileList.push(res.data.data);
                    that.form.fileList=that.fileList;
                    that.fileListname=res.data.data.name;
                    that.$message.success('上传成功！');
                })
            },
            newSubmitForm(){//确定上传
                this.$refs.newupload.submit();
            },
            onSubmit(formName) {
                let token = localStorage.getItem('ms_token');
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.$axios.post('/manager/notifications', {
                            content: this.form.content,
                            attachments: this.form.fileList,
                            receivers: this.form.receivers,
                            title: this.form.name,
                            name:this.fileListname,
                            uid: this.uid

                        },{
                            headers: {
                                'Authorization':token,
                            }
                        }).then((res) => {
                            this.$message.success('提交成功！');
                            this.$router.push({path: "/itable"})
                        })
                    }})

            },
            handleRemove(file, fileList) {
                console.log(file, fileList);
            },
            handlePreview(file) {
                console.log(file);
            },
            handleExceed(files, fileList) {

            },
            beforeRemove(file, fileList) {
                return this.$confirm(`确定移除 ${ file.name }？`);
            },
            onEditorChange({ editor, html, text }) {
                this.content = html;
            },
            cancel(){
                this.$router.push({path:"/itable"})
            }
        }
    }
</script>
<style>
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
</style>