<template>
  <div class="app-container">
    <!-- 表格的查询和修改按钮 -->
    <div class="filter-container button-group">
      <el-space wrap :size="10">
        <el-date-picker v-model="searchInfo.startDate" type="date" placeholder="开始时间" />
        <el-date-picker v-model="searchInfo.endDate" type="date" placeholder="结束时间" />
        <el-select v-model="listQuery.type" placeholder="请选择搜索项" clearable class="filter-item" style="width: 130px">
          <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'"
            :value="item.key" />
        </el-select>
        <el-input v-model="listQuery.title" placeholder="Title" style="width: 200px;" class="filter-item"
          @keyup.enter.native="handleFilter" />
        <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
          Search
        </el-button>
        <el-button style="float: right;" v-waves :loading="downloadLoading" class="filter-item" type="primary"
          icon="el-icon-download" @click="handleDownload">
          导出项目清单
        </el-button>
      </el-space>
    </div>
    <!-- 表格 -->
    <el-table :key="tableKey" v-loading="listLoading" :data="currentPage" border fit highlight-current-row
      style="width: 100%;">
      <el-table-column label="项目编号" prop="id" align="center" width="100">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="项目名称" align="center" width="150">
        <template slot-scope="{row}">
          <span>{{ row.project_name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="项目状态" align="center" width="90">
        <template slot-scope="{row}">
          <span>{{ row.project_status }}</span>
        </template>
      </el-table-column>
      <el-table-column label="项目服务点" align="center" width="200">
        <template slot-scope="{row}">
          <span>{{ row.project_server_point }}</span>
        </template>
      </el-table-column>
      <el-table-column label="实施主体" align="center" min-width="180">
        <template slot-scope="{row}">
          <span>{{ row.project_mainBody }}</span>
        </template>
      </el-table-column>
      <el-table-column label="项目负责人" align="center" width="100">
        <template slot-scope="{row}">
          <span>{{ row.project_ad }}</span>
        </template>
      </el-table-column>
      <el-table-column label="进度" align="center" width="80">
        <template slot-scope="{row}">
          <span>{{ row.project_progress }}</span>
        </template>
      </el-table-column>
      <el-table-column label="准时" align="center" width="80">
        <template slot-scope="{row}">
          <span>{{ row.project_onTime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="资金" align="center" width="80">
        <template slot-scope="{row}">
          <span>{{ row.project_money }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Actions" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button v-if="row.status!='published'" size="mini" type="success"
            @click="handleModifyStatus(row,'published')">
            查看
          </el-button>
          <el-button v-if="row.status!='deleted'" size="mini" type="danger" @click="handleDelete(row,$index)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
      :current-page.sync="currentPage2" :page-sizes="[5, 10, 15, 20]" :page-size="listInfo.currentPageSize"
      layout="sizes, prev, pager, next" :total="10">
    </el-pagination>
    <!-- <el-pagination background layout="prev, pager, next" :total="1000" /> -->

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit"
      @pagination="getList" />
  </div>
</template>
<style>
  .button-group{
    margin-bottom: 10px;
  }
</style>

<script>
  import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
  import waves from '@/directive/waves' // waves directive
  import { parseTime } from '@/utils'
  import Pagination from '@/components/Pagination' // secondary package based on el-pagination

  const calendarTypeOptions = [
    { key: 'CN', display_name: 'China' },
    { key: 'US', display_name: 'USA' },
    { key: 'JP', display_name: 'Japan' },
    { key: 'EU', display_name: 'Eurozone' }
  ]

  // arr to obj, such as { CN : "China", US : "USA" }
  const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
    acc[cur.key] = cur.display_name
    return acc
  }, {})

  export default {
    name: 'ComplexTable',
    components: { Pagination },
    directives: { waves },
    filters: {
      statusFilter(status) {
        const statusMap = {
          published: 'success',
          draft: 'info',
          deleted: 'danger'
        }
        return statusMap[status]
      },
      typeFilter(type) {
        return calendarTypeKeyValue[type]
      }
    },
    data() {
      return {
        tableKey: 0,
        searchInfo: {
          startDate: null,
          endDate: null,
          info: null
        },
        list: [
          {
            id: '1',
            project_name: 'project-name',
            project_status: 'normal',
            project_server_point: 'Beijin',
            project_mainBody: 'HYJ',
            project_ad: 'HYJ',
            project_progress: '20%',
            project_onTime: '28%',
            project_money: '80%',
          },
          {
            id: '2',
            project_name: 'project-name',
            project_status: 'normal',
            project_server_point: 'Beijin',
            project_mainBody: 'HYJ',
            project_ad: 'HYJ',
            project_progress: '20%',
            project_onTime: '28%',
            project_money: '80%',
          },
          {
            id: '3',
            project_name: 'project-name',
            project_status: 'normal',
            project_server_point: 'Beijin',
            project_mainBody: 'HYJ',
            project_ad: 'HYJ',
            project_progress: '20%',
            project_onTime: '28%',
            project_money: '80%',
          },
          {
            id: '4',
            project_name: 'project-name',
            project_status: 'normal',
            project_server_point: 'Beijin',
            project_mainBody: 'HYJ',
            project_ad: 'HYJ',
            project_progress: '20%',
            project_onTime: '28%',
            project_money: '80%',
          },
          {
            id: '114514',
            project_name: 'project-name',
            project_status: 'normal',
            project_server_point: 'Beijin',
            project_mainBody: 'HYJ',
            project_ad: 'HYJ',
            project_progress: '20%',
            project_onTime: '28%',
            project_money: '80%',
          },
          {
            id: '114514',
            project_name: 'project-name',
            project_status: 'normal',
            project_server_point: 'Beijin',
            project_mainBody: 'HYJ',
            project_ad: 'HYJ',
            project_progress: '20%',
            project_onTime: '28%',
            project_money: '80%',
          },
          {
            id: '114514',
            project_name: 'project-name',
            project_status: 'normal',
            project_server_point: 'Beijin',
            project_mainBody: 'HYJ',
            project_ad: 'HYJ',
            project_progress: '20%',
            project_onTime: '28%',
            project_money: '80%',
          },
        ],
        currentPage: [],
        total: 0,
        listLoading: true,
        listQuery: {
          page: 1,
          limit: 5,
          importance: undefined,
          title: '标题',
          type: ''
        },
        listInfo: {
          size: 200,
          currentPageSize: 5,
        },
        importanceOptions: [1, 2, 3],
        calendarTypeOptions,
        sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
        statusOptions: ['published', 'draft', 'deleted'],
        showReviewer: false,
        temp: {
          id: undefined,
          importance: 1,
          remark: '',
          timestamp: new Date(),
          title: '',
          type: '',
          status: 'published'
        },
        dialogFormVisible: false,
        dialogStatus: '',
        textMap: {
          update: 'Edit',
          create: 'Create'
        },
        dialogPvVisible: false,
        pvData: [],
        rules: {
          type: [{ required: true, message: 'type is required', trigger: 'change' }],
          timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
          title: [{ required: true, message: 'title is required', trigger: 'blur' }]
        },
        downloadLoading: false
      }
    },
    created() {
      this.listLoading = false
      this.getList()
    },
    methods: {
      //点击页面码 改变数据
      //同时负责刷新数据
      handleCurrentChange(data) {
        this.listLoading = true;
        window.setTimeout(() => {
          this.listLoading = false;
          let list = this.list.concat();
          let tempList = list.splice((data - 1) * this.listInfo.currentPageSize, data * this.listInfo.currentPageSize);
          this.currentPage = tempList;

        }, 500);
      },
      //页面显示最大数据修改时的回调函数
      handleSizeChange(data) {
        this.listLoading = true;
        this.listInfo.currentPageSize = data;
        this.handleCurrentChange(1);
      },

      getList() {
        this.listLoading = true



        //向后端请求数据  先注释
        // fetchList(this.listQuery).then(response => {
        //   this.list = response.data.items
        //   this.total = response.data.total

        //   // Just to simulate the time of the request
        //   setTimeout(() => {
        //     this.listLoading = false
        //   }, 1.5 * 1000)
        // })
        // CODE END

        //请求到数据后
        setTimeout(() => {
          let list = this.list.concat();
          let tempList = list.splice(0, this.listInfo.currentPageSize);
          this.currentPage = tempList;
          this.listLoading = false;
        }, 500)
      },

      handleFilter() {
        this.listQuery.page = 1
        this.getList()
      },
      handleModifyStatus(row, status) {
        this.$message({
          message: '操作Success',
          type: 'success'
        })
        row.status = status
      },
      // sortChange(data) {
      //   const { prop, order } = data
      //   if (prop === 'id') {

      //     if (this.listQuery.sort == '+id'){
      //       //升序
      //       this.list.sort((a,b)=>{
      //         return a[prop]-b[prop];
      //       })
      //     }else{
      //       //降序
      //       this.list.sort((a,b)=>{
      //         return b[prop]-a[prop];
      //       })
      //     }
      //     this.sortByID(order)
      //   }
      // },
      resetTemp() {
        this.temp = {
          id: undefined,
          importance: 1,
          remark: '',
          timestamp: new Date(),
          title: '',
          status: 'published',
          type: ''
        }
      },
      handleCreate() {
        this.resetTemp()
        this.dialogStatus = 'create'
        this.dialogFormVisible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].clearValidate()
        })
      },
      createData() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
            this.temp.author = 'vue-element-admin'
            createArticle(this.temp).then(() => {
              this.list.unshift(this.temp)
              this.dialogFormVisible = false
              this.$notify({
                title: 'Success',
                message: 'Created Successfully',
                type: 'success',
                duration: 2000
              })
            })
          }
        })
      },
      handleUpdate(row) {
        this.temp = Object.assign({}, row) // copy obj
        this.temp.timestamp = new Date(this.temp.timestamp)
        this.dialogStatus = 'update'
        this.dialogFormVisible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].clearValidate()
        })
      },
      updateData() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            const tempData = Object.assign({}, this.temp)
            tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
            updateArticle(tempData).then(() => {
              const index = this.list.findIndex(v => v.id === this.temp.id)
              this.list.splice(index, 1, this.temp)
              this.dialogFormVisible = false
              this.$notify({
                title: 'Success',
                message: 'Update Successfully',
                type: 'success',
                duration: 2000
              })
            })
          }
        })
      },
      handleDelete(row, index) {
        this.$notify({
          title: 'Success',
          message: 'Delete Successfully',
          type: 'success',
          duration: 2000
        })
        this.list.splice(index, 1)
      },
      handleFetchPv(pv) {
        fetchPv(pv).then(response => {
          this.pvData = response.data.pvData
          this.dialogPvVisible = true
        })
      },
      handleDownload() {
        this.downloadLoading = true
        import('@/vendor/Export2Excel').then(excel => {
          const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
          const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
          const data = this.formatJson(filterVal)
          excel.export_json_to_excel({
            header: tHeader,
            data,
            filename: 'table-list'
          })
          this.downloadLoading = false
        })
      },
      formatJson(filterVal) {
        return this.list.map(v => filterVal.map(j => {
          if (j === 'timestamp') {
            return parseTime(v[j])
          } else {
            return v[j]
          }
        }))
      },

    }
  }
</script>