<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
        <!-- 表格 -->
        <tree-table
          class="tree-table"
          :data="catelist"
          :columns="columns"
          show-index
          :expand-type="false"
          :selection-type="false"
          index-text="#"
          border
          :show-row-hover="false"
        >
          <!-- 是否有效 -->
          <template slot="isok" slot-scope="scope">
            <i v-if="!scope.row.cat_deleted" class="el-icon-success" style="color:#67c23a"></i>
            <i v-if="scope.row.cat_deleted" class="el-icon-error" style="color:#F56C6C"></i>
          </template>
          <!-- 排序 -->
          <template slot="order" slot-scope="scope">
            <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag v-if="scope.row.cat_level === 1" type="success">二级</el-tag>
            <el-tag v-if="scope.row.cat_level === 2" type="warning">三级</el-tag>
          </template>
          <template slot="opt" slot-scope="scope">
            <el-button
              @click="showEditDialog(scope.row)"
              icon="el-icon-edit"
              size="mini"
              type="primary"
            >编辑</el-button>
            <el-button
              @click="deleteCate(scope.row)"
              icon="el-icon-delete"
              size="mini"
              type="danger"
            >删除</el-button>
          </template>
        </tree-table>
        <!-- 分页 -->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[5, 10, 30]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        ></el-pagination>
      </el-row>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" @close="addCateDialogClosed">
      <el-form
        :model="addCateForm"
        :label-position="'right'"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="parentCateSetting"
            @change="parentCateChanged"
            :clearable="true"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 编辑分类对话框 -->
    <el-dialog title="编辑分类" :visible.sync="editCateDialogVisible" @close="addCateDialogClosed">
      <el-form
        :model="editCateForm"
        :label-position="'right'"
        :rules="addCateFormRules"
        ref="editCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCateForm.cat_name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类的数据列表，默认为空
      catelist: [],
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 总数据条数
      total: 0,
      // 列配置数据
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt',
          minWidth: '150px'
        }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请选择输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      parentCateSetting: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: 'true'
      },
      selectedKeys: [],
      editCateDialogVisible: false,
      editCateForm: {
        cat_name: '',
        cat_id: 0
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.catelist = res.data.result
      this.total = res.data.total
    },
    // 处理页面大小改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 处理页码改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.parentCateList = res.data
    },
    parentCateChanged() {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return false
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success('添加分类成功')
        this.addCateDialogVisible = false
        this.getCateList()
      })
    },
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.selectedKeys = []
    },
    showEditDialog(cate) {
      this.editCateDialogVisible = true
      console.log(cate)
      this.editCateForm.cat_name = cate.cat_name
      this.editCateForm.cat_id = cate.cat_id
    },
    editCate() {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return false
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, {
          cat_name: this.editCateForm.cat_name
        })
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success('更新分类名称成功！')
        this.editCateDialogVisible = false
        this.getCateList()
      })
    },
    deleteCate(cate) {
      this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('categories/' + cate.cat_id)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.queryInfo.pagenum = 1
        this.getCateList()
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>
<style lang="less" scoped>
.tree-table {
  margin-top: 60px;
}
.el-cascader {
  width: 100%;
}
</style>
