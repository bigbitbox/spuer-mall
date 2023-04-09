<template>
  <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false" :show-checkbox="true" node-key="catId">
    <!-- node代表当前结点（是否展开等信息，element-ui自带属性），data是结点数据，是自己的数据。 -->
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <=2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
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
      console.log("remove", node, data);
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