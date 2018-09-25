<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-tickets"></i> 新闻列表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">

                <el-input v-model="select_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="search" @click="search">搜索</el-button>

            </div>
            <el-table :data="tableData" border style="width: 100%" ref="multipleTable" @selection-change="handleSelectionChange">

                <el-table-column prop="name" label="代表名称" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.0" label="参加代表小组活动次数" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.1" label="联系选民次数" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.2" label="提出选民、建议、批评和意见次数" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.3" label="持代表证视察次数" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.4" label="持代表证视察次数" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.5" label="双带或结对帮扶工作次数" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.6" label="为民办实事次数" width="160" align="center">
                </el-table-column>
                <el-table-column prop="analysis.7" label="闭会期间的活动次数"  align="center">
                </el-table-column>

            </el-table>
            <div class="pagination">
                <el-pagination @current-change="handleCurrentChange" layout="prev, pager, next" :total="totalPages">
                </el-pagination>
            </div>
        </div>



    </div>
</template>

<script>
    import qs from 'qs'
    export default {
        name: 'basetable',
        data() {
            return {
                tableData: [],
                msg:"",//记录每一条的信息，便于取id
                delarr:"",//存放删除的数据
                uid:"",//返回新闻id
                pid:"",//存放发布新闻的id
                seedata:"",
                updatarr:"",//存放修改数据
                totalPages:0,
                cur_page: 1,
                multipleSelection: [],
                select_word: '',
                del_list: [],
                is_search: false,
                editVisible: false,
                delVisible: false,
                form: {
                    name: '',
                    date: '',
                    time:''
                },
                idx: -1
            }
        },
        created() {
            this.getData();
        },
        computed: {
            data() {
                return this.tableData.filter((d) => {
                    let is_del = false;
                    for (let i = 0; i < this.del_list.length; i++) {
                        if (d.name === this.del_list[i].name) {
                            is_del = true;
                            break;
                        }
                    }
                })
            }
        },
        methods: {
            // 分页导航
            handleCurrentChange(val) {
                this.cur_page = val;
                this.getData();
            },
            // 获取 easy-mock 的模拟数据
            getData() {

                this.$axios.get('/manager/npc_members/experiences/analysis', {params:{page: this.cur_page}

                }).then((res) => {
                    // console.log(res.data.data.content)
                    this.tableData = res.data.data.content;
                    this.cur_page=res.data.data.page;
                    this.totalPages=res.data.data.totalPages*10;
                    console.log(res.data.data.totalPages*10)
                })
            },
            getTime(val){
                console.log(val)
                this.form.time=val;
            },
            search() {
                this.$axios.get('/manager/npc_members/experiences/analysis', {params: {endAt:this.form.time,searchKey:this.select_word}}).then(res=>{
                    console.log(res.data.status)
                    if(res.data.status == 'OK'){
                        this.tableData=res.data.data.content;
                    }

                })

            },


            filterTag(value, row) {
                return row.tag === value;
            },





            handleSelectionChange(val) {
                this.multipleSelection = val;
            },

        }
    }

</script>

<style scoped>
    .handle-box {
        margin-bottom: 20px;
    }

    .handle-select {
        width: 120px;
    }

    .handle-input {
        width: 300px;
        display: inline-block;
    }
    .del-dialog-cnt{
        font-size: 16px;
        text-align: center
    }
</style>
