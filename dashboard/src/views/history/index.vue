<template>
  <div class="app-container">
    <span style="font-size: large; color: black; ">{{ appName }} / {{ clusterName }} / {{ namespaceName }}</span>

    <el-input v-model="search" placeholder="输入tag搜索" size="mini" align="right" />
    <el-table
      v-loading="Loading"
      :data="list.filter(data => !search || String(data.tag).toLowerCase().includes(search.toLowerCase()))"
      :default-sort="{prop: 'createdAt', order: 'descending'}"
      element-loading-text="Loading"
      style="width: 100%"
    >
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form inline class="table-expand">
            <el-form-item label="value">
              <el-input
                v-model="props.row.value"
                autosize
                type="textarea"
                placeholder=""
                :disabled="true"
              />
            </el-form-item>
            <el-form-item label="备注">
              <span>{{ props.row.comment }}</span>
            </el-form-item>
          </el-form>
        </template>

      </el-table-column>
      <el-table-column label="Tag" prop="tag" />
      <!--      <el-table-column label="应用" prop="appName" />-->
      <!--      <el-table-column label="集群" prop="clusterName" />-->
      <!--      <el-table-column label="namespace" prop="namespaceName" />-->
      <el-table-column label="创建时间" prop="createdAt" sortable>
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.createdAt | parseTime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="类型" prop="type" />

      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button v-if="scope.row.type === 'release'" size="mini" type="warning" @click="rollback(scope.row)">回滚</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { listReleaseHistory } from '@/api/release'
import { rollback } from '@/api/release'

export default {
  name: 'App',
  data() {
    return {
      Loading: true,
      appName: '',
      clusterName: '',
      namespaceName: '',
      list: [],
      search: ''
    }
  },
  created() {
    this.appName = this.$route.params.app
    this.clusterName = this.$route.params.cluster
    this.namespaceName = this.$route.params.namespace

    this.fetchData()
  },
  methods: {
    fetchData() {
      this.Loading = true
      listReleaseHistory({
        appName: this.appName,
        clusterName: this.clusterName,
        namespaceName: this.namespaceName
      })
        .then(response => {
          console.log(response)
          this.list = response
          this.Loading = false
        })
    },
    rollback(row) {
      this.$confirm('此操作将回滚配置, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          rollback({
            appName: this.appName,
            clusterName: this.clusterName,
            namespaceName: this.namespaceName,
            tag: row.tag
          })
            .then(response => {
              console.log(response)
              this.fetchData()
              this.$message.success('回滚成功')
            })
            .catch(() => {
              this.$message.error('回滚失败')
            })
        })
        .catch(() => {
          this.$message.info('已取消回滚')
        })
    }
  }
}
</script>

<style scoped>
  .table-expand {
    font-size: 0;
  }
  .table-expand label {
    width: 90px;
    color: #99a9bf;
  }
  .table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 100%;
  }
</style>
