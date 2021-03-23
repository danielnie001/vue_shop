<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加商品分类按钮区域 -->
      <el-row>
        <el-button type="primary" @click="addCateDialog">添加分类</el-button>
      </el-row>
      <!-- 商品分类表格区域 --使用第三方插件 vue-table-with-tree-grid -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            size="mini"
            type="success"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="showEditCateDialog(scope.row)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="danger"
            icon="el-icon-delete"
            @click="deleteEditCateConfirm(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes,prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="resetCateForm()"
    >
      <!-- 添加分类内容主题区域 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
        status-icon
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <!-- options指定数据源 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :key="cascaderKey"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="resetEditCateForm"
    >
      <el-form label-width="100px" :model="editCateForm">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <!-- options指定数据源 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :key="cascaderKey"
            :props="cascaderProps"
            @change="parentCateChanged"
            disabled
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    // 这里存放数据
    return {
      // 商品分类列表
      cateList: [],
      //   查询信息
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      //   为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          //   表示，将当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用模板名称
          template: 'isok'
        },
        {
          label: '排序',
          //   表示，将当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用模板名称
          template: 'order'
        },
        {
          label: '操作',
          //   表示，将当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用模板名称
          template: 'opt'
        }
      ],
      //   隐藏添加分类对话框
      addCateDialogVisible: false,
      //   添加分类表单分类信息
      addCateForm: {
        // 分类名称
        cat_name: '',
        // 分类父id
        cat_pid: 0,
        // 分类等级
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [{ required: true, message: '分类名称必填', trigger: 'blur' }]
      },
      //   父级分类列表
      parentCateList: [],
      //   级联选择器的配置对象
      cascaderProps: {
        // 指定触发方式
        expandTrigger: 'hover',
        // 数据源parantList中的cat_id做数据绑定
        value: 'cat_id',
        // 数据源parantList的cat_name渲染出来的内容
        label: 'cat_name',
        // 数据源parantList的children做嵌套
        children: 'children',
        // 父子节点不相互关联，均可选择
        checkStrictly: true
      },
      //   选中父级分类的数组
      selectedKeys: [],
      cascaderKey: 0,
      // 编辑分类对话框显示与隐藏
      editCateDialogVisible: false,
      // 编辑分类名称信息
      editCateForm: {
        cat_name: '',
        cat_id: 0
      }
    }
  },
  watch: {
    selectedKeys() {
      this.cascaderKey++
    }
  },
  // 方法集合
  methods: {
    async getCatgories() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('商品获取列表失败！')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCatgories()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCatgories()
    },
    // 显示添加分类对话框
    addCateDialog() {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      //   再展示出对话框
      this.addCateDialogVisible = true
    },
    // 重置表单信息
    resetCateForm() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
      this.cascaderKey = 0
    },
    // 获取父级分类数据的列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败！')
      }
      this.parentCateList = res.data
    },
    // 级联选择项发生变化触发
    parentCateChanged() {
      //   当前分类的父级id
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 当前分类等级
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮添加新的分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCatgories()
        this.addCateDialogVisible = false
      })
    },
    // 单击显示编辑分类对话框
    async showEditCateDialog(cateInfo) {
      if (cateInfo.cat_level === 1) {
        this.selectedKeys.push(cateInfo.cat_pid)
      } else if (cateInfo.cat_level === 2) {
        const { data: res } = await this.$http.get(
          'categories/' + cateInfo.cat_pid
        )
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        const pInfo = res.data
        this.selectedKeys.push(pInfo.cat_pid, cateInfo.cat_pid)
      }
      this.editCateForm.cat_name = cateInfo.cat_name
      this.editCateForm.cat_id = cateInfo.cat_id
      this.getParentCateList()
      this.editCateDialogVisible = true
    },
    // 点击显示提示删除当前分类
    async deleteEditCateConfirm(id) {
      const result = await this.$confirm(
        '此操作将永久删除该分类, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (result !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.getCatgories()
    },
    // 关闭编辑分类对话框时
    resetEditCateForm() {
      this.selectedKeys = []
      this.editCateForm.cat_name = ''
      this.editCateForm.cat_id = 0
      this.cascaderKey = 0
    },
    // 编辑提交分类
    async editCate() {
      const id = this.editCateForm.cat_id
      const { data: res } = await this.$http.put('categories/' + id, {
        cat_name: this.editCateForm.cat_name
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      console.log(res.data)
      this.$message.success(res.meta.msg)
      this.getCatgories()
      this.editCateDialogVisible = false
    }
  },
  // 生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getCatgories()
  },
  // 单击label能被级联选择器选中
  mounted() {
    // 点击文本就让它自动点击前面的input就可以触发选择。但是因组件阻止了冒泡，暂时想不到好方法来触发。
    // 这种比较耗性能，暂时想不到其他的，能实现效果了。
    setInterval(function() {
      document.querySelectorAll('.el-cascader-node__label').forEach(el => {
        el.onclick = function() {
          if (this.previousElementSibling) this.previousElementSibling.click()
        }
      })
    }, 1000)
  }
}
</script>
<style lang="less" scoped>
.zk-table {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
