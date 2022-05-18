<template>
    <div class="top-box">
        <el-row>
            <el-col :span='24'>
                <div>{{userInfo.name}},欢迎来到{{xxxxx}}项目管理系统。</div>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span='24'>
                <div>这是您的第{{userInfo.logginCount}}次登录，上次登录时间为XXX年XXX月XXXX日</div>
            </el-col>
        </el-row>
        <el-divider />
        <el-row>
            <el-col :span="24">
                <span>您现在负责的项目如下：</span>
                <ComplexTable></ComplexTable>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="24">
                <span>您最近的行动清单如下：</span>
                <ComplexTableActivity></ComplexTableActivity>
            </el-col>
        </el-row>
    </div>

</template>
<style scoped>
    .top-box {
        margin-top: 10px;
        margin-left: 20px;
    }

    .el-col {
        margin-top: 10px;
    }
</style>



<script>
    import ComplexTable from "../commonComponents/complex-table.vue"
    import ComplexTableActivity from "../commonComponents/complex-table-activity.vue"
    export default {
        components: {
            ComplexTable,
            ComplexTableActivity
        },
        data() {
            return {
                userInfo: {
                    name: '测试名称',
                    phone: '17628442244',
                    logginCount: 123
                }
            }
        },
        methods: {
        }
    }
</script>