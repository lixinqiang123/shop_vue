<template>

  <div>

    <div>

        <div style="margin-top: 100px;margin-left: 500px">
            <el-row>
              <el-col :span="12">
                  <div class="grid-content bg-purple-dark" >

                    <el-steps :active="active" finish-status="success">
                      <el-step title="填写商品信息"></el-step>
                      <el-step title="填写商品的属性"></el-step>
                      <el-step title="步骤 3"></el-step>
                    </el-steps>

                  </div>
              </el-col>
            </el-row>
        </div>


      <div style="margin-top: 100px;margin-left: 500px">

        <el-row>
              <el-col :span="12">
                <div class="grid-content bg-purple-dark">

                  <el-form ref="form" :model="AddProductform" label-width="80px">

                    <el-form-item label="商品名称">
                      <el-input v-model="AddProductform.name"></el-input>
                    </el-form-item>

                    <el-form-item label="商品标题">
                      <el-input v-model="AddProductform.name"></el-input>
                    </el-form-item>

                    <el-form-item label="商品品牌">
                      <el-select v-model="AddProductform.region" placeholder="请选择商品品牌">
                        <el-option v-for="item in brandSelect" :label="item.name" :value="item.id" :key="item.id"></el-option>
                      </el-select>
                    </el-form-item>


                    <el-form-item label="商品介绍">
                      <el-input type="textarea" v-model="AddProductform.desc"></el-input>
                    </el-form-item>


                    <el-form-item label="商品售价">
                      <el-input v-model="AddProductform.name"></el-input>
                    </el-form-item>

                    <el-form-item label="商品库存">
                      <el-input v-model="AddProductform.name"></el-input>
                    </el-form-item>

                    <el-form-item label="商品排序">
                      <el-input v-model="AddProductform.name"></el-input>
                    </el-form-item>


                    <el-form-item>
                      <el-button type="primary" v-on:click="next">下一步</el-button>
                      <el-button type="primary" v-on:click="next2">上一步</el-button>
                      <el-button>取消</el-button>
                    </el-form-item>
                  </el-form>

                </div>
              </el-col>
        </el-row>

      </div>


    </div>







  </div>

</template>

<script>

    import  axios from 'axios'
    import  qs from 'qs'
    export default {
        name: "AddProduct",

        data(){

          return{

            /*品牌下拉框数据*/
            brandSelect:[],

            /* 步骤条 */
            active:0,

            AddProductform:{

            }

          }
        },

        /*函数方法*/
        methods:{

          //下一步 函数
          next:function () {


            if (this.active++>2){
              this.active = 2;
            }

          },

          //上一步函数
          next2:function () {
            if (this.active--<2){
              this.active =0;
            }
          },


          /* 查询品牌信息*/
          queryBrand:function () {
            axios.get("http://localhost:8082/api/brand/queryBrand").then(rs=>{

              this.brandSelect=rs.data.data;

            }).catch();
          }
        },

        /*初始化钩子函数 查询品牌数据*/
        created:function () {

            this.queryBrand();
        }

    }
</script>

<style scoped>

</style>
