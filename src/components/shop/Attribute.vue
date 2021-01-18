<template>

  <div>

    <div style="margin-top: 20px;width: 50%">
      <el-input placeholder="请输入内容" v-model="name" class="input-with-select">

        <el-button slot="append" icon="el-icon-search" v-on:click="queryAttribute(1)"></el-button>
      </el-input>
    </div>

    <br>
    <br>
    <br>
    <br>
    <br>

    <!--属性展示的表格-->
    <el-table :data="attribute" border>

      <el-table-column prop="id" label="序号" width="180"> </el-table-column>

      <el-table-column prop="name" label="属性英文名" width="180"> </el-table-column>

      <el-table-column prop="nameCH" label="属性中文名"  width="180"></el-table-column>

      <el-table-column prop="typeName" label="属于是什么类" width="180"></el-table-column>

      <el-table-column prop="isSKU" label="是否是sku属性" width="180">



          <el-switch v-model="skuisattr"></el-switch>


      </el-table-column>

      <el-table-column prop="type" label="是什么类型的" :formatter="attrFormatter" width="180"></el-table-column>

      <el-table-column  label="操作" width="130px">

        <template slot-scope="scope">

          <el-button @click="deleteBrand(scope.row)" type="text" size="small">删除</el-button>

          <el-button type="text" v-if="scope.row.type!=4" size="small" @click="attrValueShow(scope.row)">属性值维护</el-button>

        </template>

      </el-table-column>
    </el-table>

    <!--属性分页的插件-->
    <div>
      <el-pagination
        @size-change="sizeChange"
        @current-change="jumpBrand"
        :page-sizes="sizes"
        :page-size="size"
        layout="total, sizes, prev, pager, next, jumper"
        :total="count"
      >
      </el-pagination>
    </div>



    <el-button type="primary" v-on:click="showAddForm">新增</el-button>






    <!--新增属性的弹框-->
    <el-dialog title="商品信息录入" :visible.sync="dialogFormVisible">

      <el-form ref="form" :model="form" label-width="80px">

        <el-form-item label="分类">
          <el-select v-model="form.typeId" placeholder="请选择分类">
            <el-option label="请选择" :value="-1"></el-option>

            <el-option v-for="item in types" :label="item.name" :value="item.id"></el-option>

          </el-select>
        </el-form-item>

        <el-form-item label="属性英文">
          <el-input v-model="form.name"></el-input>
        </el-form-item>

        <el-form-item label="属性中文">
          <el-input v-model="form.nameCH"></el-input>
        </el-form-item>


        <el-form-item label="是否是sku">
          <el-switch v-model="form.isSKU"></el-switch>
        </el-form-item>


        <el-form-item label="属性类型">
          <el-radio-group v-model="form.type">
            <el-radio label="1">下拉框</el-radio>
            <el-radio label="2">单选框</el-radio>
            <el-radio label="3">复选框</el-radio>
            <el-radio label="4">输入框</el-radio>
          </el-radio-group>
        </el-form-item>



      </el-form>



      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addAttr">确 定</el-button>
      </div>

    </el-dialog>






    <!--属性值的展示和维护  table-->
    <el-dialog title="属性值" :visible.sync="dialogTable">

      <el-button type="primary" plain v-on:click="addShowAttrValue">新增属性值</el-button>

      <el-table :data="DataTable">

        <el-table-column property="id" label="序号" width="150"></el-table-column>
        <el-table-column property="name" label="属性值英文名" width="200"></el-table-column>
        <el-table-column property="nameCH" label="属性值中文名"></el-table-column>

        <el-table-column  label="操作" width="130px">

          <template slot-scope="scope">

            <el-button @click="deleteBrand(scope.row)" type="text" size="small">删除</el-button>

            <el-button type="text" size="small" @click="updateAttrValue(scope.row)">属性值修改</el-button>

          </template>

        </el-table-column>

      </el-table>

    </el-dialog>


    <!--新增属性值的 弹框-->
    <el-dialog title="属性值信息录入" :visible.sync="dialogAttrValue">



      <el-form ref="attrvalueform" :rules="rules" :model="attrvalueform" label-width="80px">


        <el-form-item label="属性值英文" prop="name">
          <el-input v-model="attrvalueform.name"></el-input>
        </el-form-item>

        <el-form-item label="属性值中文" prop="nameCH">
          <el-input v-model="attrvalueform.nameCH"></el-input>
        </el-form-item>


        <input v-model="attrId" hidden/>



      </el-form>



      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addAttrValue">确 定</el-button>
      </div>

    </el-dialog>



    <!--修改属性值的 弹框-->
    <el-dialog title="属性值信息录入" :visible.sync="updateAttrvaluedia">



      <el-form ref="form" :model="updatevalueform" label-width="80px">

        <input  v-model="updatevalueform.id" hidden/>
        <el-form-item label="属性值英文">
          <el-input v-model="updatevalueform.name"></el-input>
        </el-form-item>

        <el-form-item label="属性值中文">
          <el-input v-model="updatevalueform.nameCH"></el-input>
        </el-form-item>
      </el-form>



      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateValue">确 定</el-button>
      </div>

    </el-dialog>

  </div>


</template>

<script>

    import  axios from 'axios'
    import qs from 'qs'
    export default {
        name: "Attribute",

        data(){

          return{


            //属性值的表格数据
            dialogTable:false,
            DataTable:[],

            /*属性值的弹框*/
            dialogAttrValue:false,
            attrvalueform:{

              attrId:"",
              name:"",
              nameCH:""
            },

            attrId:"",

            /*属性值的修改*/
            updateAttrvaluedia:false,
            updatevalueform:{
              id:"",
              name:"",
              nameCH:"",
            },

            //分页的数据
            count:0,
            size:5,

            sizes:[5,10,15],

            //属性数据
            attribute:[],

            //商品属性的数据
            form:{
              name:"",
              nameCH:"",
              isSKU:false,
              type:"",
              typeId:""

            },

            //属性的条件查询
            name:"",

            //属性的 sku  开关
            skuisattr:true,

            //是否弹框默认是 false
            dialogFormVisible:false,

            //所有属性的数据
            attrArr:[],

            //所有子节点的数据
            types:[],

            typeName:"",

            str:"",

            /*验证规则*/
            rules: {
              name: [
                { required: true, message: '请输入属性值的英文名称', trigger: 'blur' },
                { max: 10, message: '长度最大不可以超过10个字符', trigger: 'blur' }
              ],
              nameCH: [
                { required: true, message: '请输入属性值的中文名称', trigger: 'change' }
              ]
            }
          };

        },

        //函数方法
        methods:{

          //确定修改属性值
          updateValue:function(){

            axios.post("http://localhost:8082/api/attrvalue/update",qs.stringify(this.updatevalueform)).then(rs=>{

              //修改成功 关闭弹框
              this.updateAttrvaluedia=false;


            }).catch(err=>{
              console.log(err)
            });

          },

          //修改弹框
          updateAttrValue:function(row){

            this.updateAttrvaluedia=true;

            this.updatevalueform=row;

          },

          //新增属性值
          addAttrValue:function(){


            this.attrvalueform.attrId=this.attrId;

            this.$refs["attrvalueform"].validate((valid) => {

              //验证通过
              if (valid) {

                axios.post("http://localhost:8082/api/attrvalue/add",qs.stringify(this.attrvalueform)).then(rs=>{

                  this.dialogAttrValue=false;

                  //提示信息
                  this.$message.success("新增成功")

                  //刷新table
                  this.queryAttrValue(this.attrvalueform.attrId);

                }).catch(err=>{
                  console.log(err)
                });

              } else {
                return false;
              }
            });

          },

          //属性值的弹框
          addShowAttrValue:function(){

            this.dialogAttrValue=true;
             this.attrvalueform.attId="";
             this.attrvalueform.name="";
             this.attrvalueform.nameCH="";
          },

          /*属性值弹框*/
          attrValueShow:function(row){

            alert(JSON.stringify(row));

            this.dialogTable=true;

            this.attrId=row.id;

            this.queryAttrValue(row.id);


          },


          //属性值查询
          queryAttrValue:function(id){

            axios.get("http://localhost:8082/api/attrvalue/getData?id="+id).then(rs=>{

              this.DataTable=rs.data.data.data;

            }).catch();
          },

          //新增
          addAttr:function(){

            this.form.isSkus=this.form.isSKU==true?1:2;

            this.form.isSKU=this.form.isSkus;
            console.log(this.form);

            axios.post("http://localhost:8082/api/attribute/add",qs.stringify(this.form)).then(rs=>{

              this.dialogFormVisible=false;

              this.queryAttribute(1);

            }).catch(err=>{

            });
          },

          //展示属性的弹框
          showAddForm:function(){

            this.dialogFormVisible=true;
            this.form={};
          },

            //查询属性
            queryAttribute:function (page) {

              axios.get("http://localhost:8082/api/attribute/getData?page="+page+"&size="+this.size+"&name="+this.name).then(rs=>{



                  this.attribute=rs.data.data.data;

                  this.count=rs.data.data.count;
              }).catch(err=>{

                alert("查询失败");
              })
            },


            //属性sku开关的事件
            switchShi:function(row){
                alert(JSON.stringify(row));
            },

          /*分页的一些函数*/

          //当前页数改变的时候会触发
          jumpBrand:function (page) {

              this.queryAttribute(page);
          },

          //条数改变的时候 会触发的函数
          sizeChange:function (size) {

              this.size=size;

              this.queryAttribute(1);

          },


          /* 查询分类数据库里的 所有数据*/
          queryType:function () {

            axios.get("http://localhost:8082/api/type/getData").then(rs=>{

              //所有的分类数据
              this.attrArr=rs.data.data;

              //调用新的函数  找到所有子节点的数据
              debugger;
              this.getChildrenType();


              //在遍历所有的子节点
              for (let i = 0; i <this.types.length ; i++) {

                  this.typeName="";

                  //发送一个新的函数
                  debugger;
                  this.diguiName(this.types[i]);

                  //把拼好的数据 赋值给数组的name
                  this.types[i].name=this.typeName.split("/").reverse().join("/").substring(0,this.typeName.split("/").reverse().join("/").length-1);
              }


              console.log(this.typeName);


            }).catch(err=>{

              console.log(err)
            });

          },

          diguiName:function(node){
            debugger;
            //判断 当前的pid是0 的数据 临界点
            if(node.pid!=0){
              this.typeName+="/"+node.name;

              //寻找子节点上面的节点
              for (let i = 0; i <this.attrArr.length; i++) {
                  //获取一条数据

                  if(node.pid==this.attrArr[i].id){
                      //然后就递归这条数据 在走一遍
                      this.diguiName(this.attrArr[i]);
                      break;
                  }
              }

            }else{
              this.typeName+="/"+node.name;
            }

          },


          //找到所有的子节点
          getChildrenType:function(){
            debugger;
            //遍历所有的节点数据
            for (let i = 0; i <this.attrArr.length; i++) {
                //得到第一条节点数据
                let node=this.attrArr[i];

                debugger;
                //在来一个函数 找急子节点
                this.isChildrenNode(node);
            }


          },


          //拿着传过来的 一条数据 找到子节点
          isChildrenNode:function(node){
              
            var rs=true;

            debugger;
            for (let i = 0; i < this.attrArr.length; i++) {
                //判断所有的 属性数据里 有没有 pid指向我这条数据 id 有的话是代表是父节点
              if(node.id==this.attrArr[i].pid){

                  debugger;
                  rs=false;
                  break;
              }

            }


            if(rs==true){
              debugger;
              this.types.push(node);
            }
            
          },
          
          //初始化 类型的 单元格
          attrFormatter:function (row,column,cellValue,index) {
              if(cellValue==1){
                return "下拉框";
              }
              if(cellValue==2){
                return "单选框";
              }
              if(cellValue==3){
                return "复选框";
              }
              if(cellValue==4){
                return "输入框";
              }
              return "未知";
          },

        },
        //初始化钩子函数
        created:function () {

            this.queryAttribute(1);

            //调用查询分类的所有数据
            this.queryType();
        }
    }
</script>

<style scoped>

</style>
