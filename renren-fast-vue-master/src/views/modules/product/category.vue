<template>
  <div>
    <!-- 添加分类对话框 -->
    <el-dialog :title=title :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类信息">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>

    <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="draggable" type="primary" @click="batchSave">批量保存</el-button>

    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
      :show-checkbox="true" node-key="catId" :default-expanded-keys="expandedKey" :draggable="draggable"
      :allow-drop="allowDrop" @node-drop="handleDrop">
      <!-- node代表当前结点（是否展开等信息，element-ui自带属性），data是结点数据，是自己的数据。 -->
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
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
import { title } from 'process';

export default {
  name: 'category',
  data() {
    return {
      pCid: [],
      draggable: false,
      updateNodes: [], //用来记录拖拽后的节点
      maxLevel: 1, //用来记录当前节点的最大深度
      title: '',//对话框的标题
      dialogType: "",//add,edit
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: '',
        productUnit: ''
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
    //批量保存方法
    batchSave() {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          message: '菜单顺序修改成功',
          type: 'success'
        });
        //刷新出菜单
        this.getMenus();
        //设置默认展开的菜单
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        //this.pCid = 0;

      })
    },



    //新增方法
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop: ", draggingNode, dropNode, dropType);
      //1、当前节点最新父节点的id
      let pCid = 0;
      //拖拽后的兄弟节点，分两种情况，一种是拖拽到两侧，一种是拖拽到内部
      let sibings = null;
      if (dropType == "before" || dropType == "after") {
        pCid = dropNode.parent.data.catId == undefined ? 0 : dropNode.parent.data.catId;
        sibings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        sibings = dropNode.childNodes;
      }
      this.pCid.push(pCid);

      //2、当前拖拽节点的最新顺序
      //遍历所有的兄弟节点，如果是拖拽节点，传入（catId，sort，parentCid，catLevel），如果是兄弟节点传入（catId，sort）
      for (let i = 0; i < sibings.length; i++) {
        if (sibings[i].data.catId == draggingNode.data.catId) {
          //如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level;
          if (sibings[i].level != draggingNode.level) {
            //当前节点的层级发生变化
            catLevel = sibings[i].level;
            //修改他子节点的层级
            this.updateChildNodeLevel(sibings[i]);
          }
          this.updateNodes.push({ catId: sibings[i].data.catId, sort: i, parentCid: pCid, catLevel: catLevel });
        } else {
          this.updateNodes.push({ catId: sibings[i].data.catId, sort: i });
        }

      }

      //3、当前拖拽节点的最新层级
      console.log("updateNodes: ", this.updateNodes);



      //每次拖拽后把数据清空，否则要修改的节点将会越拖越多

    },

    // 修改拖拽节点的子节点的层级
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          //遍历子节点，传入（catId，catLevel）
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({ catId: cNode.catId, catLevel: node.childNodes[i].level });
          //处理子节点的子节点
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    //拖动节点定义
    allowDrop(draggingNode, dropNode, type) {
      console.log("allowDrag:", draggingNode, dropNode, type);
      //节点的最大深度
      this.countNodeLevel(draggingNode);
      console.log("level:", this.maxLevel);
      //当前节点的深度
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      console.log(deep)
      if (type == "inner") {
        return (deep + dropNode.level) <= 3;
      } else {
        return (deep + dropNode.parent.level) <= 3;
      }
    },

    //计算当前节点的最大深度
    countNodeLevel(node) {
      //找到所有的子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },


    //拖拽成功后触发的方法



    //提交表单的方法
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      } else if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    //编辑分类的方法
    editCategory() {
      //只发送所要修改的数据
      let { name, catId, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ name, catId, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          message: '菜单修改成功',
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

    //编辑方法（对应按钮）
    edit(data) {
      this.title = "编辑分类";
      this.dialogType = "edit";
      this.dialogVisible = true;

      //发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({ data }) => {
        console.log("要回显的数据", data);
        this.category.name = data.category.name;
        this.category.catId = data.category.catId;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid;
      })
    },
    //添加方法（对应按钮）
    append(data) {
      this.title = "添加分类";
      this.dialogType = "add";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel + 1;

      this.category.name = '';
      this.category.catId = null;
      this.category.icon = '';
      this.category.productUnit = '';
      this.category.sort = 0;
      this.category.showStatus = 1;
    },

    //表单的添加方法
    addCategory() {
      console.log("提交的三级分类数据", this.category);
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
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