<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i>法律法规</el-breadcrumb-item>
                <el-breadcrumb-item>添加法律法规</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box" style="width:700px">
                <el-form ref="form" :model="form" :rules="rules" label-width="110px">
                    <el-input v-model="uid" style="display: none"></el-input>
                    <el-form-item label="法律法规标题" prop="name">
                        <el-input v-model="form.name"></el-input>
                    </el-form-item>
                    <el-form-item label="封面图" prop="imgSrc">
                        <div v-show="imgVisible">
                            <img :src="form.imgSrc" style="width:auto;height: 200px">
                        </div>
                        <div  class="el-upload__tip">只能上传jpg/png文件</div>
                        <el-upload
                                class="upload-demo"
                                action=""
                                :before-upload="beforeUpload1"
                                accept=".jpg,.jpeg,.png,.gif,.pdf,.JPG,.JPEG,.GIF,.PDF"
                                :auto-upload="true">
                            <el-button size="small" type="primary" v-show="anniu">点击上传</el-button>
                            <div slot="tip" v-show="anniu1">正在上传</div>
                        </el-upload>
                        <div class="anniu" v-show="anniu" @click="clears">清空</div>
                    </el-form-item>
                    <el-form-item label="法律分类" prop="type">
                        <el-select v-model="form.type" placeholder="请选择">
                            <el-option
                                    :value="0"
                                    label="国家法律"
                            >
                            </el-option>
                            <el-option
                                    :value="1"
                                    label="地方法律"
                            >
                            </el-option>
                            <el-option
                                    :value="2"
                                    label="权力和义务"
                            >
                            </el-option>

                        </el-select>
                    </el-form-item>
                    <el-form-item label="附件" prop="fileList1">
                        <el-upload
                                v-model="form.fileList1"
                                class="upload-demo"
                                action=""
                                :on-preview="handlePreview"
                                :on-remove="handleRemove"
                                :before-remove="beforeRemove"
                                :before-upload="beforeUpload"
                                :limit="1"
                                :on-exceed="handleExceed"
                                :file-list="fileList"
                                >
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

                imgVisible:false,
                content: '',
                uid:'',
                fileList:[],
                anniu1:false,
                anniu:true,
                fileListname:"",
                dialogVisible: false,
                name:'',
                receivers:[],
                imageUrl: '',
                imgsrc:"",
                form: {
                    imgSrc:'',
                    name: '',
                    desc: '',
                    type:"",
                    fileList1:"",
                },
                rules:{
                    name :[{required: true, message: '请输入法律法规标题', trigger: 'blur'}],
                    imgSrc:[{required: true, message: '请上传法律法规封面图', trigger: 'blur'}],
                    type:[{required: true, message: '请选择法律分类', trigger: 'blur'}],
                    fileList1:[{required: true, message: '请上传法律法规附件', trigger: 'blur'}],
                }
            }
        },

        created () {
            this.uid=this.$route.query.uid;
        },
        methods: {
            clears(){
                this.imgVisible = false;
                this.form.imgSrc = "";
            },
            beforeUpload1(file){
                let token = localStorage.getItem('ms_token');
                let fd = new FormData();
                fd.append('file',file);//传文件
                fd.append('uid',this.uid);//传其他参数
                let that=this;
                this.$axios.post('/manager/legals/upload',fd,{
                    headers: {
                        'Authorization':token,
                    }
                }).then(function(res){
                    if(res.data.status == 'OK'){
                        that.form.imgSrc=that.GLOBAL.BASE_URL + res.data.data.url;
                        that.$message.success('上传成功！');
                    }else{
                        that.$message.success('上传失败！');
                        that.imgVisible = false;
                    }

                })
                this.imgVisible = true;
            },
            beforeUpload(file){
                let token = localStorage.getItem('ms_token');
                let fd = new FormData();
                fd.append('file',file);//传文件
                fd.append('uid',this.uid);//传其他参数
                let that=this;
                this.$axios.post('/manager/legals/upload',fd,{
                    headers: {
                        'Authorization':token,
                    }
                }).then(function(res){
                    that.form.fileList1=res.data.data.url;
                    that.fileListname=res.data.data.name;
                    that.fileList.push(res.data.data);
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
                        this.$axios.post('/manager/legals', {
                            content: this.form.fileList1,
                            cover: this.form.imgSrc,
                            title: this.form.name,
                            type:this.form.type,
                            name: this.fileListname,
                            uid: this.uid
                        },{
                            headers: {
                                'Authorization':token,
                            }
                        }).then((res) => {
                            console.log(res)
                            this.$message.success('提交成功！');
                            this.$router.push({path: "/ltable"})
                        })
                    }})

            },
            handleExceed(files, fileList) {
                this.$message.warning(`当前限制选择 1 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
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
                this.$router.push({path:"/ltable"})
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
    .anniu{
        color: #fff;
        background-color: #409EFF;
        border-color: #409EFF;
        font-size: 12px;
        border-radius: 3px;
        width: 60px;
        height: 20px;
        padding:5px 5px;
        line-height: 20px;
        text-align: center;
        float:left;
        margin-top: -50px;
        margin-left: 110px;

    }
</style>