<template>
  <div>
    <el-tree
      :data="menu"
      :props="defaultProps"
      @node-click="handleNodeClick"
      node-key="catId"
      show-checkbox
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.catLevel <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="data.children.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCatagory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import 引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    return {
      //三级分类的所有数据
      menu: [],
      // 添加菜单的对话框
      dialogVisible: false,
      // 添加菜单对话框中的数据
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      //默认展开的节点的 key 的数组
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    handleNodeClick(data) {
      // console.log(data);
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
        // params: this.$http.adornParams({
        //   page: this.pageIndex,
        //   limit: this.pageSize,
        //   roleName: this.dataForm.roleName,
        // }),
      }).then(({ data }) => {
        console.log(data.data);
        this.menu = data.data;
      });
    },
    //点击append按钮，弹出添加菜单的对话框
    append(data) {
      console.log("append方法中的data参数：");
      console.log(data);
      this.dialogVisible = true;
      //<点击append按钮的节点>为<要添加的节点>的父节点
      this.category.parentCid = data.catId;
      //<要添加的节点>的父节点的层级为<点击append按钮的节点>+1
      //data.catLevel*1 可以将 String类型的 data.catLevel 转为 int 类型
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    //向后端发送删除请求
    remove(node, data) {
      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          var ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ datas }) => {
            this.$message({
              type: "success",
              message: `【${data.name}】删除成功`,
            });
            //重新发送请求,更新数据
            this.getMenus();
            //默认展开的菜单节点
            this.expandedKey = [node.parent.data.catId];
            //或者写成this.expandedKey=[data.parentCid]
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });

      console.log("remove方法中的node参数：");
      console.log(node);
      console.log("remove方法中的data参数：");
      console.log(data);
    },
    //向后端发送添加请求
    addCatagory() {
      this.dialogVisible = false;
      console.log("catagory中的数据:");
      console.log(this.category);

      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: `添加成功`,
        });
        //重新获取数据
        this.getMenus();
        //默认展开的菜单节点
        this.expandedKey = [this.category.parentCid];
      });
    },
  },
  //计算属性 类似于 data 概念
  computed: {},
  //监控 data 中的数据变化
  watch: {},

  //生命周期 - 创建完成（可以访问当前 this 实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有 keep-alive 缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>