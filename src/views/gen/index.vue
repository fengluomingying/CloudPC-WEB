<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">客户名称</label>
        <el-input v-model="query.customerName" clearable placeholder="客户名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">归属区县</label>
        <el-input v-model="query.county" clearable placeholder="归属区县" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">问题分类</label>
        <el-input v-model="query.problemCategory" clearable placeholder="问题分类" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">是否解决</label>
        <el-input v-model="query.isResolved" clearable placeholder="是否解决" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">支撑状态</label>
        <el-input v-model="query.supportState" clearable placeholder="支撑状态" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.time"
          start-placeholder="timeStart"
          end-placeholder="timeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="时间">
            <el-date-picker v-model="form.time" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="客户名称">
            <el-input v-model="form.customerName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="归属区县">
            <el-select v-model="form.county" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.county"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="问题分类">
            未设置字典，请手动设置 Select
          </el-form-item>
          <el-form-item label="支撑进度">
            <el-input v-model="form.technicalSupportProgress" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否解决">
            <el-input v-model="form.isResolved" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="支撑状态">
            未设置字典，请手动设置 Select
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="id" />
        <el-table-column prop="time" label="时间" />
        <el-table-column prop="customerName" label="客户名称" />
        <el-table-column prop="county" label="归属区县">
          <template slot-scope="scope">
            {{ dict.label.county[scope.row.county] }}
          </template>
        </el-table-column>
        <el-table-column prop="problemCategory" label="问题分类" />
        <el-table-column prop="technicalSupportProgress" label="支撑进度" />
        <el-table-column prop="isResolved" label="是否解决" />
        <el-table-column prop="supportState" label="支撑状态" />
        <el-table-column v-if="checkPer(['admin','cloudPc:edit','cloudPc:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudCloudPc from '@/api/cloudPc'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, time: null, customerName: null, county: null, problemCategory: null, technicalSupportProgress: null, isResolved: null, supportState: null }
export default {
  name: 'CloudPc',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['county'],
  cruds() {
    return CRUD({ title: '云电脑支撑接口生成', url: 'api/cloudPc', idField: 'id', sort: 'id,desc', crudMethod: { ...crudCloudPc }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'cloudPc:add'],
        edit: ['admin', 'cloudPc:edit'],
        del: ['admin', 'cloudPc:del']
      },
      rules: {
      },
      queryTypeOptions: [
        { key: 'customerName', display_name: '客户名称' },
        { key: 'county', display_name: '归属区县' },
        { key: 'problemCategory', display_name: '问题分类' },
        { key: 'isResolved', display_name: '是否解决' },
        { key: 'supportState', display_name: '支撑状态' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
