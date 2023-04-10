<template>
  <div>
      <!-- 添加分类对话框 -->
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类信息" :label-width="formLabelWidth">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>


    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
      :show-checkbox="true" node-key="catId" :default-expanded-keys="expandedKey">
      <!-- node代表当前结点（是否展开等信息，element-ui自带属性），data是结点数据，是自己的数据。 -->
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            Append
          </el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
export default {
  name: 'category',
  data() {
    return {
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0
      },
      dialogVisible: false,
      expandedKey: [],
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  methods: {
    //对应三级分类的添加和删除方法
    append(data) {
      this.dialogVisible = true;
      console.log("append", data);
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel + 1;
    },

    //表单的添加方法
    addCategory(){
      console.log("提交的三级分类数据",this.category);
      this.$http({
      url:this.$http.adornUrl('/product/category/save'),
      method:'post',
      data: this.$http.adornData(this.category, false)
      }).then(({data})=>{ 
        this.$message({
          message: '添加成功',
          type: 'success'
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新出菜单
        this.getMenus();
        //设置默认展开的菜单
        this.expandedKey = [this.category.parentCid];
        //清空表单
        this.category.name = '';
      })
    },


    remove(node, data) {
      let ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => (
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(() => {
          console.log("删除成功...")
          //刷新出菜单
          this.getMenus();
          //设置默认展开的菜单
          this.expandedKey.push[node.parent.data.catId];
          this.$message({
            message: '删除成功',
            type: 'success'
          })
        })
      )).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },

    handleNodeClick(data) {
      console.log(data);
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        console.log("获取到数据...", data.data);
        this.menus = data.data;
      });
    }
  },
  created() {
    this.getMenus();
  },
  mounted() {

  }
}
</script>

<style lang="scss" scoped></style>