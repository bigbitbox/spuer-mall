<template>
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
</template>

<script>
export default {
  name: 'category',
  data() {
    return {
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
      console.log("append", data);
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