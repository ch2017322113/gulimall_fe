<!--  -->
<template>
  <div>
    <el-switch
      v-model="allowDragB"
      active-text="启动拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch
    ><br />
    <el-button v-if="allowDragB" type="info" @click="batchSave"
      >批量保存</el-button
    >
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menu"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandeKeys"
      :draggable="allowDragB"
      :allow-drop="allowDrop"
      :allow-drag="allowDrag"
      @node-drop="handleDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>

          <el-button type="text" size="mini" @click="edit(data)">
            Edit
          </el-button>

          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-form :model="category">
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-form :model="category">
        <el-form-item label="单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import Vue from "vue";
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      pCid: [],
      allowDragB: false,
      updateNodes: [],
      maxLevel: 0,
      title: "",
      dialogType: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        productUnit: "",
        icon: "",
      },
      menu: [],
      expandeKeys: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      dialogVisible: false,
    };
  },
  methods: {
    batchDelete() {
      let catIds = [];
      let catNames = [];
      let checkNodes = this.$refs.menuTree.getCheckedNodes();
      for (let i = 0; i < checkNodes.length; i++) {
        catIds.push(checkNodes[i].catId);
        catNames.push(checkNodes[i].name);

      }
      this.$confirm(`将批量删除[${catNames}], 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: `批量删除成功!`,
            });
            this.getMenus();
          });
        })
        .catch(() => {});
    },
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: `拖拽修改成功!`,
        });
        //刷新出新的菜单
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = this.pCid;
        //将更新节点置空
        this.updateNodes = [];
        this.maxLevel = 0;
        this.pCid = 0;
      });
    },
    submitData() {
      if (this.dialogType == "append") {
        this.addCategory();
      } else if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    editCategory() {
      var { name, catId, icon, productUnit } = this.category;
      var data = { name, catId, icon, productUnit };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: `${this.category.name}-修改成功!`,
        });
        this.dialogVisible = false;
        this.getMenus();
        this.expandeKeys = [this.category.parentCid];
      });
    },
    addCategory() {
      console.log("addCategory", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: `${this.category.name}-保存成功!`,
        });
        this.dialogVisible = false;
        this.getMenus();
        this.expandeKeys = [this.category.parentCid];
      });
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("success!!", data.data);
        this.menu = data.data;
      });
    },
    edit(data) {
      // console.log("edit的data", data);
      this.title = "编辑";
      this.dialogType = "edit";
      this.dialogVisible = true;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("将回显的数据", data);
        /*               
        category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        productUnit: "",
        icon: "",}
      }, */
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
        this.category.sort = data.data.sort;
        this.category.catLevel = data.data.catLevel;
        this.category.showStatus = data.data.showStatus;
      });
    },
    allowDrop(draggingNode, dropNode, type) {
      // console.log(draggingNode, dropNode, type);
      this.countLevel(draggingNode);
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      // console.log(deep)
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
      // console.log(this.maxLevel);
      /* if (countLevel(draggingNode) + this.countLevel(dropNode) > 3) {
        return false;
      }else{
        return true;
      } */
    },
    allowDrag(draggingNode) {
      return this.allowDragB;
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("tree drop: ", draggingNode, dropNode, dropType, ev);
      let pCid = 0;
      let siblings = null;
      if (dropType == "after" || dropType == "before") {
        pCid = dropNode.data.parentCid;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }

      this.pCid.push(pCid);

      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            catLevel = siblings[i].level;
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }

      console.log("updateNodes", this.updateNodes);
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    countLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let index = 0; index < node.childNodes.length; index++) {
          if (node.childNodes[index].level > this.maxLevel) {
            this.maxLevel = node.childNodes[index].level;
          }
          this.countLevel(node.childNodes[index]);
        }
      }
    },

    append(data) {
      console.log("append", data);
      this.title = "添加";
      this.dialogType = "append";
      this.dialogVisible = true;

      this.category.name = "";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.sort = 0;
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.showStatus = 1;
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`此操作将永久删除[${data.name}], 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: `${node.data.name}-删除成功!`,
            });
            this.getMenus();
            this.expandeKeys = [node.parent.data.catId];
          });
        })
        .catch(() => {});

      console.log("remove", node, data);
    },
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>