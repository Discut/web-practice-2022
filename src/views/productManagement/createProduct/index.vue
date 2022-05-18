<template>
    <div class="app-container create-product-box">
        <el-form ref="form" :model="form" label-width="120px">
            <el-form-item label="项目名称*">
                <el-input v-model="form.product_name" placeholder="请以签约协议的项目书为准" />
            </el-form-item>
            <el-form-item label="项目服务点*">
                <el-input v-model="form.product_server_point" placeholder="请按照XX街道XX社区为准撰写，如果不是请据实撰写" />
            </el-form-item>
            <el-form-item label="实施主体*">
                <el-select v-model="form.activity_object" placeholder="下拉选择">
                    <el-option label="TODO 这里请使用v-if 绘制" value="shanghai" />
                    <el-option label="Zone two" value="beijing" />
                </el-select>
            </el-form-item>
            <el-form-item label="项目负责人*">
                <el-input v-model="form.product_leader" placeholder="TODO 默认是创建人，可以修改" />
            </el-form-item>
            <el-form-item label="项目背景*">
                <el-input v-model="form.product_background" type="textarea" placeholder="请采用无格式的文字粘贴，注意段首的空格。" />
            </el-form-item>
            <el-form-item label="项目概述*">
                <el-input v-model="form.product_summary" type="textarea" placeholder="请采用无格式的文字粘贴，注意段首的空格。" />
            </el-form-item>
            <el-form-item label="项目目标*">
                <el-input v-model="form.product_target" type="textarea" placeholder="请采用无格式的文字粘贴，注意段首的空格。" />
            </el-form-item>
            <el-form-item label="项目时间*" class="">
                <el-date-picker v-model="form.product_start_date" type="date" placeholder="开始时间" />
                <span class="pick-date-box">至</span>
                <el-date-picker v-model="form.product_end_date" type="date" placeholder="结束时间" />
            </el-form-item>
            <el-form-item label="项目效果">
                <el-input v-model="form.product_effect" type="textarea" placeholder="请采用无格式的文字粘贴，注意段首的空格。" />
            </el-form-item>
            <el-form-item label="项目内容*">
                <el-input v-model="form.product_content" type="textarea" placeholder="请采用无格式的文字粘贴，注意段首的空格。" />
            </el-form-item>
        </el-form>
        <el-button type="primary" @click="onSubmit">创建</el-button>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                form: {
                    product_name: '',
                    product_server_point: '',
                    activity_object: '',
                    product_ad: '',
                    product_leader: '',
                    product_background: '',
                    product_summary: '',
                    product_target: '',
                    product_start_date: null,
                    product_end_date: null,
                    product_effect: '',
                    product_content: '',
                }
            }
        },
        methods: {
            onSubmit() {
                this.$message('submit!')
            },
            onCancel() {
                this.$message({
                    message: 'cancel!',
                    type: 'warning'
                })
            }
        }
    }
</script>

<style scoped>
    .line {
        text-align: center;
    }

    .create-product-box {
        margin-left: 10px;
    }

    .pick-date-box {
        margin: 0 30px;
    }
</style>