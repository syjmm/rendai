<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i>代表小组</el-breadcrumb-item>
                <el-breadcrumb-item>添加小组</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box">
                <el-form ref="form" :model="form" :rules="rules" label-width="90px">
                    <el-form-item label="小组名称" prop="name">
                        <el-input v-model="form.name"></el-input>
                    </el-form-item>
                    <el-form-item label="小组介绍">
                        <el-input type="textarea" v-model="form.desc"></el-input>
                    </el-form-item>
                    <el-form-item label="小组工作站">
                        <el-select v-model="form.work" placeholder="请选择">
                            <el-option
                                    v-for="item in options"
                                    :key="item.uid"
                                    :label="item.name"
                                    :value="item.uid"
                            >
                            </el-option>
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
    export default {
        name: 'baseform',
        data: function(){
            return {
                options:[],
                uid:'',
                dialogVisible: false,
                form: {
                    name: '',
                    desc: '',
                    work:''
                },
                rules:{
                    name :[{required: true, message: '请输入小组名称', trigger: 'blur'}],
                }
            }
        },
        created(){
            this.uid = this.$route.query.uid;
            this.getdata();
            let token = localStorage.getItem('ms_token');
            this.$axios.get('/manager/work_stations/all',{
                headers: {
                    'Authorization':token,
                }
            }).then(res=>{
                this.options = res.data.data;
            })
        },
        methods: {
            //获取数据
            getdata(){
                let token = localStorage.getItem('ms_token');
                this.$axios.get('/manager/npc_member_groups/details',{
                    headers: {
                        'Authorization':token,
                    },
                    params:{uid:this.uid}}).then(res=>{
                        this.form.name = res.data.data.name;
                        this.form.desc = res.data.data.description;
                        this.form.work = res.data.data.workStation.name;
                        console.log( res.data.data.workStation.uid)
                })
            },
            onSubmit(formName) {
                let token = localStorage.getItem('ms_token');
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.$axios.post('/manager/npc_member_groups/update',
                            qs.stringify({
                                description: this.form.desc,
                                name: this.form.name,
                                uid: this.uid,
                                workStation: this.form.work,
                            }),{
                                headers: {
                                    'Authorization':token,
                                }
                            }).then((res) => {
                            console.log(res)
                            this.$message.success('提交成功！');
                            console.log()
                            this.$router.push({path: "/gtable"})
                        })
                    }})
            },
            cancel(){
                this.$router.push({path:"/gtable"})
            }
        }
    }
</script>
<style scoped>
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