<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i>新闻</el-breadcrumb-item>
                <el-breadcrumb-item>修改新闻</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box" style="width: 80%">
                <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                    <el-input v-model="uid" style="display: none"></el-input>
                    <el-form-item label="新闻标题" prop="name">
                        <el-input v-model="form.name" style="width: 30%"></el-input>
                    </el-form-item>
                    <el-form-item label="新闻封面" prop="imgSrc">
                        <div>
                            <img :src="GLOBAL.BASE_URL + form.imgSrc" style="width:300px;height: 200px">
                        </div>
                        <div  class="el-upload__tip">只能上传jpg/png文件</div>
                        <el-upload
                                class="upload-demo"
                                action=""
                                :before-upload="beforeUpload"
                                accept=".jpg,.jpeg,.png,.gif,.bmp,.pdf,.JPG,.JPEG,.PBG,.GIF,.BMP,.PDF"
                                :auto-upload="true">

                            <el-button size="small" type="primary">点击上传</el-button>

                        </el-upload>

                    </el-form-item>
                    <el-form-item label="新闻分类" prop="type">
                        <el-select v-model="form.type" placeholder="请选择">
                            <el-option
                                    :value="1"
                                    label="人大要闻"
                            >
                            </el-option>
                            <el-option
                                    :value="2"
                                    label="主要活动"
                            >
                            </el-option>
                            <el-option
                                    :value="3"
                                    label=" 重要会议"
                            >
                            </el-option>
                            <el-option
                                    :value="4"
                                    label=" 党委相关新闻"
                            >
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="新闻内容" width="100%" prop="content">
                        <quill-editor ref="myTextEditor" v-model="form.content" :options="editorOption" ></quill-editor>
                        <!--<el-button class="editor-btn" type="primary" @click="submit">提交</el-button>-->
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="onSubmit('form')">确认修改</el-button>
                        <el-button @click="cancel">取消</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </div>

    </div>
</template>

<script>
    import qs from 'qs'
    import 'quill/dist/quill.core.css';
    import 'quill/dist/quill.snow.css';
    import 'quill/dist/quill.bubble.css';
    import { quillEditor } from 'vue-quill-editor';
    import {quillRedefine} from 'vue-quill-editor-upload'
    import VueCropper  from 'vue-cropperjs';
    export default {
        name: 'updataform',
        data: function(){
            return {
                content: '',
                uid:'',
                editorOption: {},
                dialogVisible: false,
                name:'',
                imageUrl: '',
                imgsrc:"",
                form: {
                    name: '',
                    imgSrc:'',
                    content: '',
                    desc: '',
                    type:'',
                },
                rules:{
                    name :[{required: true, message: '请输入新闻标题', trigger: 'blur'}],
                    imgSrc:[{required: true, message: '请上传新闻封面图', trigger: 'blur'}],
                    type:[{required: true, message: '请选择新闻类型', trigger: 'blur'}],
                    content:[{required: true, message: '请输入新闻内容', trigger: 'blur'}],
                }
            }
        },
        components: {
            VueCropper,
            quillEditor,
            quillRedefine
        },
        created () {
            this.uid=this.$route.query.uid;
            this.getdata();
            let token = localStorage.getItem('ms_token');
            this.editorOption = quillRedefine(
                {
                    // 图片上传的设置
                    uploadConfig: {
                        action:  this.GLOBAL.BASE_URL+'/api/manager/news/upload',  // 必填参数 图片上传地址
                        // 必选参数  res是一个函数，函数接收的response为上传成功时服务器返回的数据
                        // 你必须把返回的数据中所包含的图片地址 return 回去
                        name:"file",
                        res: (respnse) => {
                            // console.log(respnse.data.url)
                            return this.GLOBAL.BASE_URL + respnse.data.url  // 这里切记要return回你的图片地址
                        },
                        header: (xhr, formData) => {
                            xhr.setRequestHeader('Authorization',token);
                            formData.append('uid', this.uid)
                        },
                    }
                }
            )
            // console.log(this.editorOption)
        },
        methods: {
            //获取数据
            getdata(){
                let token = localStorage.getItem('ms_token');
              this.$axios.get('/manager/news/details',{
                  headers: {
                      'Authorization':token,
                  },
                  params:{uid:this.uid}}).then(res=>{
                  this.form.name=res.data.data.title;
                  this.form.imgSrc=res.data.data.cover;
                  this.form.content=res.data.data.content;
                  this.form.type=res.data.data.type;
              })
            },
            beforeUpload(file){
                let token = localStorage.getItem('ms_token');
                let fd = new FormData();
                fd.append('file',file);//传文件
                fd.append('uid',this.uid);//传其他参数
                let that=this;
                this.$axios.post('/manager/news/upload',fd,{
                    headers: {
                        'Authorization':token,
                    }}).then(function(res){
                        if(res.data.status == 'OK'){
                            that.form.imgSrc=res.data.data.url;
                            that.$message.success('上传成功！');
                        }else{
                            that.$message.error('只允许上传jpg/png格式的图片！');
                            that.imgVisible = false;
                        }
                })
            },
            newSubmitForm(){//确定上传
                this.$refs.newupload.submit();
            },
            onSubmit(formName) {
                let token = localStorage.getItem('ms_token');
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.$axios.post('/manager/news/update',
                            qs.stringify({
                                content: this.form.content,
                                cover: this.form.imgSrc,
                                title: this.form.name,
                                uid: this.uid
                            }),
                            {
                                headers: {
                                    'Authorization': token,
                                }
                            }).then((res) => {
                            console.log(res)
                            this.$message.success('修改成功！');
                            this.$router.push({path: "/table"})
                        })
                    }})
                // this.$message.success('提交成功！');
            },
            onEditorChange({ editor, html, text }) {
                this.content = html;
            },
            cancel(){
                this.$router.push({path:"/table"})
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
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }
    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 178px;
        height: 178px;
        line-height: 178px;
        text-align: center;
    }
    .avatar {
        width: 178px;
        height: 178px;
        display: block;
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