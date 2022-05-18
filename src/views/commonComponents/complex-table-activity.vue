<template>
  <div class="app-container">
    <el-table :key="tableKey" v-loading="listLoading" :data="currentPage" border fit highlight-current-row
      style="width: 100%;">
      <el-table-column label="开展时间" align="center" width="100">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="行动名称" align="center" width="150">
        <template slot-scope="{row}">
          <span>{{ row.project_name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="项目名称" align="center" min-width="180">
        <template slot-scope="{row}">
          <span>{{ row.project_status }}</span>
        </template>
      </el-table-column>
      <el-table-column label="项目模块" align="center" width="200">
        <template slot-scope="{row}">
          <span>{{ row.project_server_point }}</span>
        </template>
      </el-table-column>
      <el-table-column label="项目负责人" align="center" min-width="80">
        <template slot-scope="{row}">
          <span>{{ row.project_mainBody }}</span>
        </template>
      </el-table-column>
      <el-table-column label="提交时间" align="center" width="100">
        <template slot-scope="{row}">
          <span>{{ row.project_ad }}</span>
        </template>
      </el-table-column>
      <el-table-column label="按时情况" align="center" width="80">
        <template slot-scope="{row}">
          <span>{{ row.project_progress }}</span>
        </template>
      </el-table-column>
 
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button v-if="row.status!='published'" size="mini" type="success">
            下载
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

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px"
        style="width: 400px; margin-left:50px;">
        <el-form-item label="Type" prop="type">
          <el-select v-model="temp.type" class="filter-item" placeholder="Please select">
            <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name"
              :value="item.key" />
          </el-select>
        </el-form-item>
        <el-form-item label="Date" prop="timestamp">
          <el-date-picker v-model="temp.timestamp" type="datetime" placeholder="Please pick a date" />
        </el-form-item>
        <el-form-item label="Title" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="Status">
          <el-select v-model="temp.status" class="filter-item" placeholder="Please select">
            <el-option v-for="item in statusOptions" :key="item" :label="item" :value="item" />
          </el-select>
        </el-form-item>
        <el-form-item label="Imp">
          <el-rate v-model="temp.importance" :colors="['#99A9BF', '#F7BA2A', '#FF9900']" :max="3"
            style="margin-top:8px;" />
        </el-form-item>
        <el-form-item label="Remark">
          <el-input v-model="temp.remark" :autosize="{ minRows: 2, maxRows: 4}" type="textarea"
            placeholder="Please input" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          Confirm
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

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
        list: [
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
          type: '',
          sort: '+id'
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
      sortChange(data) {
        const { prop, order } = data
        if (prop === 'id') {
          this.sortByID(order)
        }
      },
      sortByID(order) {
        if (order === 'ascending') {
          this.listQuery.sort = '+id'
        } else {
          this.listQuery.sort = '-id'
        }
        this.handleFilter()
      },
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
      getSortClass: function (key) {
        const sort = this.listQuery.sort
        return sort === `+${key}` ? 'ascending' : 'descending'
      }
    }
  }
</script>