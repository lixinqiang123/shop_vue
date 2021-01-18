<template>
  <div>
    <h1>商品分类管理</h1>


    <el-tree
      :data="data"
      :props="defaultProps"
      show-checkbox
      accordion
      check-strictly="true"
      ref="tree"
      expand-on-click-node="false"
    >

    </el-tree>

    <!--按钮-->
    <el-row>
      <el-button type="primary" icon="el-icon-circle-plus-outline" circle v-on:click="nodeChckbox"></el-button>
      <el-button type="primary" icon="el-icon-edit" circle v-on:click="updateNode"></el-button>
      <el-button type="danger" icon="el-icon-delete" circle></el-button>
    </el-row>


    <!-- 新增的弹框-->
    <el-dialog title="节点信息" :visible.sync="typeVisable" width="50%">

      <el-form :model="typeForm" label-width="80px">

        <el-input v-model="typeForm.id" disabled type="hidden">父节点id</el-input>

        <el-form-item label="父节点名称" :label-width="formLabelWidth">

          <el-input v-model="typeForm.prentname" style="width: 50%"  disabled autocomplete="off"></el-input>

        </el-form-item>

        <el-form-item label="子节点名称" :label-width="formLabelWidth">
          <el-input v-model="typeForm.name" style="width: 50%" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="子节点" :label-width="formLabelWidth">
          <el-radio v-model="typeForm.isDel" label="0">显示</el-radio>
          <el-radio v-model="typeForm.isDel" label="1">不显示</el-radio>
        </el-form-item>

      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="typeVisable = false">取 消</el-button>
        <el-button type="primary" @click="okAddNode">确 定</el-button>
      </div>

    </el-dialog>

  </div>
</template>

<script>

  import axios from 'axios'

  import qs from 'qs'

  export default {
    name: "Type",

    data() {
      return {

        //远程请求的数据
        ajaxData: [],

        //创建一个空字符串用来承载数据
        jsonStr: "",

        data: [],
        defaultProps: {
          label: 'label',
          children: "children"
        },

        //是否显示表单
        typeVisable: false,

        //节点信息
        typeForm: {
          id: "",
          name: "",
          prentname: "",
          isDel: ""
        },

        //修改节点的信息
        updateVisable:false,
        updatetypeForm:{
          id:"",
          name:""
        }
      };
    },
    methods: {

      //改变函数
      changeData: function () {

        //循环遍历所有的数据
        for (let i = 0; i < this.ajaxData.length; i++) {


          //判断进来的数据的 pid是不是0  0代表父节点
          if (this.ajaxData[i].pid == 0) {

            //调用递归函数
            this.diguiNode(this.ajaxData[i]);

            //中断循环
            break;
          }
        }

        //最终将 字符串 转化为对象放入我们的数组中
        this.data.push(JSON.parse(this.jsonStr));
      },

      //递归
      diguiNode: function (node) {
        //调用我们 判断是不是父节点的函数
        var bf = this.isParent(node);


        //如果返回true了 代表我这条 数据下是由子节点的 拼接子节点
        if (bf == true) {
          //{"id":1,"label":"分类",}
          //{"id":1,"label":"分类","children":[]}
          this.jsonStr += '{"id":' + node.id + ',"label":"' + node.name + '","children":[';

          console.log(this.jsonStr);
          //拼接子节点
          //查找此节点
          var count = 0;
          for (let i = 0; i < this.ajaxData.length; i++) {
            //判断是否为当前节点的 子节点
            if (node.id == this.ajaxData[i].pid) {

              count++;
              //自己调自己 开始递归
              this.diguiNode(this.ajaxData[i]);
              this.jsonStr += ",";
            }
          }

          //处理多余的逗号 品完整
          if (count != 0) {
            this.jsonStr = this.jsonStr.substr(0, this.jsonStr.length - 1);
          }


          //拼完整
          this.jsonStr += ']}';

        } else {
          this.jsonStr += '{"id":' + node.id + ',"label":"' + node.name + '"}';
        }
      },


      //判断当前这条数据 是不是父节点
      isParent: function (node) {


        //循环遍历 数组的所有数据 然后和我传过来的数据进行对比  如果所有的数据里面 pid 有指向当前这条数据的id的时候 代表我这条数据就是父节点

        for (let i = 0; i < this.ajaxData.length; i++) {

          if (node.id == this.ajaxData[i].pid) {

            return true;
          }
        }

        return false;
      },


      //回显当前选中节点
      nodeChckbox: function () {

        var cknodes = this.$refs.tree.getCheckedNodes();

        if(cknodes.length>1){
          alert("节点选择过多 一次只能选择一个");
          return;
        }

        if (cknodes.length > 0) {

          var id = cknodes[0].id;
          var name = cknodes[0].label;

          this.typeForm.id = id;
          this.typeForm.prentname = name;

          //弹框
          this.typeVisable = true;
        } else {
          alert("没有选择节点");
        }

      },

      //新增节点
      okAddNode: function () {

        console.log(this.typeForm);
        //创建空json
        var jsonType = {pid: this.typeForm.id, name: this.typeForm.name, isDel: this.typeForm.isDel};

        axios.post("http://localhost:8082/api/type/add", qs.stringify(jsonType)).then(rs => {

          location.reload();
        }).catch();
      }

    },


    //钩子函数
    created: function () {

      axios.get("http://localhost:8080/api/type/getData").then(rs => {

        //把数据库查出来的数据给定义的属性
        console.log(rs);
        this.ajaxData = rs.data.data;

        //并且调用一个新的函数
        this.changeData();
      }).catch();
    }

  };
</script>

<style scoped>

</style>
