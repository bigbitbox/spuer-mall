<template>
<el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
    export default {
        name: 'category',
        data() {
      return {
        data: [ ],
        defaultProps: {
          children: 'children',
          label: 'label'
        }
      };
    },
    methods: {
      handleNodeClick(data) {
        console.log(data);
      }
    },
        mounted() {

        }
    }
</script>

<style lang="scss" scoped>

</style>