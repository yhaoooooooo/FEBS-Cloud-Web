<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="queryParams.tableName" :placeholder="$t('table.gen.generate.tableName')" class="filter-item search-item" />
      <el-select v-model="queryParams.datasource" :placeholder="$t('table.gen.generate.datasource')" class="filter-item search-item" @change="search">
        <el-option
          v-for="item in datasourcesName"
          :key="item"
          :label="item"
          :value="item"
        />
      </el-select>
      <el-button class="filter-item" type="primary" plain @click="search">
        {{ $t('table.search') }}
      </el-button>
      <el-button class="filter-item" type="success" plain @click="reset">
        {{ $t('table.reset') }}
      </el-button>
      <el-button class="filter-item" type="success" plain @click="gen(null, true)">
        生成
      </el-button>
    </div>
    <el-table
      ref="table"
      :key="tableKey"
      v-loading="loading"
      :data="list"
      border
      fit
      style="width: 100%;"
      @selection-change="selectionChanged"
    >
      <el-table-column type="selection" />
      <el-table-column :label="$t('table.gen.generate.tableName')" prop="name" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.gen.generate.remark')" prop="remark" :show-overflow-tooltip="true" align="center" max-width="200px">
        <template slot-scope="scope">
          <span>{{ scope.row.remark }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.gen.generate.dataRows')" prop="dataRows" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.dataRows }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.gen.generate.createTime')" prop="createTime" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.gen.generate.updateTime')" prop="updateTime" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.updateTime }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.operation')" align="center" min-width="90px" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <i v-has-permission="['gen:generate:gen']" class="el-icon-download table-operation" style="color: rgb(45, 183, 245)" @click="gen(row)" />
          <el-link v-has-no-permission="['gen:generate:gen']" class="no-perm">
            {{ $t('tips.noPermission') }}
          </el-link>
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="total>0" :total="total" :page.sync="pagination.num" :limit.sync="pagination.size" @pagination="search" />
  </div>
</template>
<script>
import Pagination from '@/components/Pagination'

export default {
  name: 'Generate',
  components: { Pagination },
  data() {
    return {
      tableKey: 0,
      loading: false,
      list: null,
      total: 0,
      queryParams: {},
      pagination: {
        size: 10,
        num: 1
      },
      datasourcesName: [],
      selections: []
    }
  },
  mounted() {
    this.fetch()
    this.getDatasources()
  },
  methods: {
    selectionChanged(selection) {
      console.log(selection)
      this.selections = []
      for (var i = 0; i < selection.length; i++) {
        this.selections[i] = selection[i].name
      }
      console.log(this.selections)
    },
    search() {
      this.fetch({
        ...this.queryParams
      })
    },
    getDatasources() {
      this.$get('generator/datasources').then((r) => {
        this.datasourcesName = r.data.data
      })
    },
    gen(row, multi) {
      if (multi) {
        if (this.selections.length === 0) {
          this.$message.info('请选择行')
        } else {
          this.$download('generator', {
            names: this.selections,
            datasource: this.queryParams.datasource,
            remark: '批量生成'
          }, `${this.queryParams.datasource}_code.zip`)
        }
      } else {
        var select = []
        select[0] = row.name
        this.$download('generator', {
          names: select[0],
          datasource: this.queryParams.datasource,
          remark: row.remark
        }, `${row.name}_code.zip`)
      }
    },
    reset() {
      this.queryParams = {}
      this.search()
    },
    fetch(params = {}) {
      this.loading = true
      params.pageSize = this.pagination.size
      params.pageNum = this.pagination.num
      this.$get('generator/tables', { ...params }).then((r) => {
        const data = r.data.data
        this.total = data.total
        this.list = data.rows
        this.loading = false
      })
    }
  }
}
</script>
