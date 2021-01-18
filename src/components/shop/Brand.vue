<template>

  <div>


    <el-form :inline="true" :model="serchBrand" class="demo-form-inline">

      <el-button type="primary" v-on:click="addFormShow">新增</el-button>

      <el-form-item label="品牌名">
        <el-input v-model="serchBrand.name"></el-input>
      </el-form-item>

      <el-form-item label="首字母">
        <el-input v-model="serchBrand.bandE"></el-input>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="queryBrandData(1)">查询</el-button>
      </el-form-item>

    </el-form>


    <div>

      <el-table :data="brandData" border style="width: 100%">

        <el-table-column prop="id" label="序号" width="180"> </el-table-column>

        <el-table-column prop="name" label="姓名" width="180"> </el-table-column>

        <el-table-column prop="bandE" label="首字母"  width="180"></el-table-column>

        <el-table-column prop="imgpath" label="图片" width="180">

          <!-- 按文本处理   :formatter="formatImg"    -->
          <!-- 模板处理  html  -->
          <template slot-scope="scope">
            <!-- <img src="192.168.1.43:8080/imgFiless/f86a6cd6-a0e3-47a6-ba03-62a68ff41c99.gif"/>-->
            <img width="50px" :src="scope.row.imgpath"/>
          </template>

        </el-table-column>

        <el-table-column prop="bandDesc" label="品牌故事" width="180"></el-table-column>

        <el-table-column prop="ord" label="排序字段" width="180"></el-table-column>

        <el-table-column prop="createDate" label="创建时间" width="180"></el-table-column>

        <el-table-column prop="updateDate" label="updateDate" width="180"></el-table-column>

        <el-table-column  label="操作" width="100">

          <template slot-scope="scope">

            <el-button @click="deleteBrand(scope.row)" type="text" size="small">删除</el-button>

            <el-button type="text" size="small" @click="updateShow(scope.row)">编辑</el-button>

          </template>

        </el-table-column>


      </el-table>

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

    </div>



    <!--新增弹框-->
    <el-dialog title="新增信息" :visible.sync="addVisAble">

      <el-form ref="form" :model="addFrom" label-width="100px">
        <el-form-item label="品牌名称">
          <el-input v-model="addFrom.name" style="width: 300px"></el-input>
        </el-form-item>

        <el-form-item label="首字母">
          <el-input v-model="addFrom.bandE" style="width: 300px"></el-input>
        </el-form-item>

        <el-form-item label="上传品牌图片">

          <el-upload
            class="avatar-uploader"
            action="http://localhost:8082/api/brand/uploadoss"
            :show-file-list="false"
            :on-success="addformImg"
            name="img"
            >
            <img v-if="imageUrl" :src="imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>

        </el-form-item>

        <el-form-item label="品牌故事">
          <el-input type="textarea" style="width: 300px" v-model="addFrom.bandDesc" maxlength="200" show-word-limit></el-input>
        </el-form-item>

        <el-form-item label="排序字段">
          <el-input v-model="addFrom.ord" style="width: 300px"></el-input>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="okAddForm">确定</el-button>
          <el-button>取消</el-button>
        </el-form-item>

      </el-form>

    </el-dialog>



    <!--修改品牌数据的弹框-->
    <el-dialog title="修改的信息" :visible.sync="UpdateVisable">

      <el-form ref="form" :model="updateFrom" label-width="100px">

        <input v-model="updateFrom.id" hidden/>

        <el-form-item label="品牌名称">
          <el-input v-model="updateFrom.name" style="width: 300px"></el-input>
        </el-form-item>

        <el-form-item label="首字母">
          <el-input v-model="updateFrom.bandE" style="width: 300px"></el-input>
        </el-form-item>

        <el-form-item label="上传品牌图片">

          <el-upload
            class="avatar-uploader"
            action="http://localhost:8082/api/brand/uploadoss"
            :show-file-list="false"
            :on-success="addformImg"
            name="img"
          >
            <img v-if="updateFrom.imgpath" :src="updateFrom.imgpath" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>

        </el-form-item>

        <el-form-item label="品牌故事">
          <el-input type="textarea" style="width: 300px" v-model="updateFrom.bandDesc" maxlength="200" show-word-limit></el-input>
        </el-form-item>

        <el-form-item label="排序字段">
          <el-input v-model="updateFrom.ord" style="width: 300px"></el-input>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="okUpdateForm">确定</el-button>
          <el-button>取消</el-button>
        </el-form-item>

      </el-form>
    </el-dialog>


  </div>
</template>

<script>

    import  axios from 'axios'

    import qs from 'qs'

    export default {
        name: "Brand",

        data(){

          return{

            /* 品牌 展示的数据*/
            brandData:[],

            /* 条件查询的数据 */
            serchBrand:{

              name:"",
              bandE:""
            },

            /*分页插件需要的数据*/
            size:5,
            sizes:[5,10,15],
            //总条数
            count:0,

            //是否弹框 新增品牌的弹框 初始化false
            addVisAble:false,

            /* 是否弹框  修改的弹框 */
            UpdateVisable:false,


            imageUrl:"",

            /*新增品牌的数据*/
            addFrom:{
              name:"",
              bandE:"",
              imgpath:"",
              bandDesc:"",
              ord:""
            },


            /*修改品牌的数据*/
            updateFrom:{
              id:"",
              name:"",
              bandE:"",
              imgpath:"",
              bandDesc:"",
              ord:""
            }

          }
        },

        //函数方法
        methods:{

          //删除
          deleteBrand:function(row){

            axios.delete("http://localhost:8082/api/brand/deleteBrand?id="+row.id).then(rs=>{

              this.queryBrandData(1);
            }).catch();

          },

            //跳转页面 默认传值是1  页面改变的时候 会监听到的
          jumpBrand:function(page){

            this.queryBrandData(page);

          },

          //每页展示的条数发生变化的时候 会进行改变 也可以直接监听到
          sizeChange:function(size){
              this.size=size;
              this.queryBrandData(1);
          },

            //查询函数
            queryBrandData:function (page) {

                axios.get("http://localhost:8082/api/brand/getData?page="+page+"&size="+this.size+"&"+qs.stringify(this.serchBrand)).then(rs=>{

                  console.log(rs);
                  this.brandData=rs.data.data.data;

                  this.count=rs.data.data.count;
                }).catch();
            },
          
          
            //图片上传成功后的 回调函数
          addformImg:function (response, file, fileList) {

                this.imageUrl=response.data.data;
                this.addFrom.imgpath=response.data.data;
                this.updateFrom.imgpath=response.data.data;
          },

          //展示新增的form表单
          addFormShow:function () {
              this.imageUrl="";

            this.addVisAble=true;
          },

          //新增
          okAddForm:function () {


                axios.post("http://localhost:8082/api/brand/add",qs.stringify(this.addFrom)).then(rs=>{

                  //重新查询
                  this.queryBrandData(1)

                  //关闭弹框
                  this.addVisAble=false;
                }).catch();


          },

          //展示修改数据
          updateShow:function (row) {
              //获取到图片的路径 给src赋值
              this.updateFrom=row;

              this.UpdateVisable=true;
          },

          /*修改的函数*/
          okUpdateForm:function () {

            axios.post("http://localhost:8082/api/brand/updateBrand",qs.stringify(this.updateFrom)).then(rs=>{

              //重新查询
              this.queryBrandData(1)

              //关闭弹框
              this.UpdateVisable=false;
            }).catch();
          }
        },

        //钩子函数
        created:function () {

            //调用查询的函数
            this.queryBrandData(1);
        }
    }
</script>

<style scoped>

  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }

</style>
