<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i>工作管理</el-breadcrumb-item>
                <el-breadcrumb-item>添加工作</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box" style="width:700px">
                <el-form ref="form" :model="form" :rules="rules" label-width="140px">
                    <el-input v-model="uid" style="display: none"></el-input>
                    <el-form-item label="工作标题" prop="name">
                        <el-input v-model="form.name"></el-input>
                    </el-form-item>
                    <el-form-item label="工作内容" prop="content">
                        <el-input type="textarea" v-model="form.content"></el-input>
                    </el-form-item>
                    <el-form-item label="接收代表" prop="receivers">
                        <el-checkbox-group v-model="form.receivers" >
                            <el-checkbox  v-for="item in options":label="item.uid"  :key="item.uid">{{item.name}}</el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                    <el-form-item label="备注">
                        <el-input type="textarea" v-model="form.desc"></el-input>
                    </el-form-item>
                    <el-form-item label="完成工作截至时间" prop="time">
                        <el-date-picker
                                v-model="form.time"
                                type="date"
                                @change="getTime"
                                value-format="yyyy-MM-dd"
                                :picker-options="pickerOptions0"
                                placeholder="选择日期时间">
                        </el-date-picker>
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
                pickerOptions0: {
                    disabledDate(time) {
                        return time.getTime() < Date.now() - 8.64e7;
                    }
                },
                filename:'',
                fileListname:'',
                dialogVisible: false,
                form: {
                    time: '',
                    name:'',
                    receivers:[],
                    content: '',
                    desc:''
                },
                rules:{
                    name :[{required: true, message: '请输入工作标题', trigger: 'blur'}],
                    content :[{required: true, message: '请输入工作内容', trigger: 'blur'}],
                    receivers :[{required: true, message: '请选择接收代表', trigger: 'blur'}],
                    time :[{required: true, message: '请选择时间', trigger: 'blur'}],
                }
            }
        },

        created () {
            this.uid=this.$route.query.uid;
            let token = localStorage.getItem('ms_token');
            this.$axios.get('/manager/npc_members/working',{
                headers: {
                    'Authorization':token,
                }}).then(res=>{
                this.options = res.data.data;
            })
        },
        methods: {
            getTime(val){
                console.log(val)
                this.form.time=val;
            },

            onSubmit(formName) {
                let token = localStorage.getItem('ms_token');
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.$axios.post('/manager/jobs', {
                            content: this.form.content,
                            endAt: this.form.time,
                            receivers: this.form.receivers,
                            title: this.form.name,
                            remarks:this.form.desc

                        },{
                            headers: {
                                'Authorization':token,
                            }
                        }).then((res) => {
                            this.$message.success('提交成功！');
                            this.$router.push({path: "/mtable"})
                        })
                    }})

            },
            cancel(){
                this.$router.push({path:"/mtable"})
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