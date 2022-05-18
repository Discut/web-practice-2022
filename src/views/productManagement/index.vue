<template>
    <div>
        <div style="margin-left:10px;">
            <p>在这里，您可以
                <el-button type="success" size="small" @click="createProduct">创建项目</el-button>
                或者查看项目情况，若要删除项目请联系管理员。如果项目结项请点击查看进入项目工作台进行结项。
            </p>
        </div>
        <el-divider></el-divider>
        <ComplexTable></ComplexTable>
    </div>
</template>
<script>
    import ComplexTable from "./components/complex-table.vue"

    export default {
        components: {
            ComplexTable
        },
        methods: {
            createProduct() {
                console.log(123)
                this.$router.push({
                    path : './createProduct',
                    query: {
                        t: +new Date() //保证每次点击路由的query项都是不一样的，确保会重新刷新view
                    }
                })
                // this.$router.go({
                //     path: './createProduct',
                //     force: true,
                // })
            }
        }
    }
</script>