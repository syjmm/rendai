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
                <el-form ref="form" :model="form" :rules="rules"label-width="100px">
                    <el-input v-model="uid" style="display: none"></el-input>
                    <el-form-item label="法律法规标题">
                        <el-input v-model="form.name"></el-input>
                    </el-form-item>
                    <el-form-item label="封面图">
                        <div>
                            <img :src="form.imgSrc" style="width:200px;height: 200px">
                        </div>
                        <div  class="el-upload__tip">只能上传jpg/png文件</div>
                        <el-upload
                                class="upload-demo"
                                action=""
                                :before-upload="beforeUpload1"
                                accept=".jpg,.jpeg,.png,.gif,.bmp,.pdf,.JPG,.JPEG,.PBG,.GIF,.BMP,.PDF"
                                :auto-upload="true">
                            <el-button size="small" type="primary">点击上传</el-button>

                        </el-upload>
                    </el-form-item>
                    <el-form-item label="法律分类" prop="type">
                        <el-select v-model="form.type" placeholder="请选择">
                            <el-option
                                    :value="0"
                                    label="国家法律法规"
                            >
                            </el-option>
                            <el-option
                                    :value="1"
                                    label="地方法律法规"
                            >
                            </el-option>
                            <el-option
                                    :value="2"
                                    label="权力义务"
                            >
                            </el-option>

                        </el-select>
                    </el-form-item>
                    <el-form-item label="附件">
                        <el-upload
                                class="upload-demo"
                                action=""
                                :on-preview="handlePreview"
                                :on-remove="handleRemove"
                                :before-remove="beforeRemove"
                                :before-upload="beforeUpload"
                                multiple
                                :limit="1"
                                :on-exceed="handleExceed"
                                :file-list="fileList">
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="onSubmit">提交</el-button>
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
                imgSrc:'',
                content: '',
                uid:'',
                fileList:[],
                fileListname:"",
                dialogVisible: false,
                name:'',
                receivers:"",
                imageUrl: '',
                form: {
                    name: '',
                    desc: '',
                    type:"",
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
            var _this = this;
            let token = localStorage.getItem('ms_token');
            this.$axios.get('/manager/legals/details',{
                headers: {
                    'Authorization':token,
                },
                params:{uid:this.uid}}).then(res=>{
                this.form.name = res.data.data.title;
                this.form.imgSrc = res.data.data.cover;
                this.form.type= res.data.data.type;
                this.receivers=res.data.data.content;
                this.fileListname=res.data.data.name;
                this.fileList.push({name:_this.fileListname,url:_this.receivers})

            })
        },
        methods: {
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
                        that.imgSrc=that.GLOBAL.BASE_URL + res.data.data.url;
                        that.$message.success('上传成功！');
                    }else{
                        that.$message.success('上传失败！');
                        that.imgVisible = false;
                    }
                })
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
                    // that.fileList[0]=res.data.data;
                    // console.log(that.fileList)
                    that.fileList.pop()
                    that.fileList.push(res.data.data);
                    that.fileListname=res.data.data.name;
                    that.receivers=res.data.data.url;

                    that.receivers=res.data.data.url;
                    that.$message.success('上传成功！');
                })
            },
            newSubmitForm(){//确定上传
                this.$refs.newupload.submit();
            },
            onSubmit() {
                let token = localStorage.getItem('ms_token');
                this.$axios.post('/manager/legals/update', {
                        content: this.receivers,
                        cover:this.form.imgSrc,
                        type:this.form.type,
                        title: this.form.name,
                        uid:this.uid,
                        name:this.fileListname
                    },{
                    headers: {
                        'Authorization':token,
                    }
                }).then((res) => {
                    console.log(res)
                    this.$message.success('提交成功！');
                    this.$router.push({path:"/ltable"})
                })

            },
            handleRemove(file, fileList) {
                console.log(file, fileList);
                this.fileList.pop()
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
</style>