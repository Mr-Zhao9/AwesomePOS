<template>
    <div class="pos">
      <div>
        <el-row>
          <el-col :span="7" class="left-order" id="order-list">
            <el-tabs>
              <el-tab-pane label="点餐">
                <el-table :data="tableData" border>
                  <el-table-column prop="goodsName" label="商品名称"></el-table-column>
                  <el-table-column prop="count" label="量"></el-table-column>
                  <el-table-column prop="price" label="金额"></el-table-column>
                  <el-table-column label="操作" fixed="right">
                    <template scope="scope">
                      <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                      <el-button type="text" size="small" @click="addorderList(scope.row)">增加</el-button>
                    </template>
                  </el-table-column>
                </el-table>
                <div class="tableDiv">
                  <small>数量：</small>{{totalCount}}&nbsp;&nbsp;&nbsp;&nbsp;<small>金额：</small>{{totalMoney}}
                </div>
                <div class="div-btn">
                  <el-button type="warning">挂单</el-button>
                  <el-button type="danger" @click="delAllGoods">删除</el-button>
                  <el-button type="success" @click="checkout">结账</el-button>
                </div>
              </el-tab-pane>
              <el-tab-pane label="挂单">
                挂单
              </el-tab-pane>
              <el-tab-pane label="外卖">
                外卖
              </el-tab-pane>
            </el-tabs>
          </el-col>
          <el-col :span="17">
            <div class="often-goods">
              <div class="title">常用商品</div>
              <div class="often-goods-list">
                <ul>
                  <li v-for="goods in oftenGoods" @click="addorderList(goods)">
                    <span>{{goods.goodsName}}</span>
                    <span class="o-price">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
              <div class="goods-type">
                <el-tabs>
                  <el-tab-pane label="汉堡">
                    <ul class='cookList'>
                      <li v-for="goods in type0Goods" @click="addorderList(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                    </ul>
                  </el-tab-pane>
                  <el-tab-pane label="小食">
                    <ul class='cookList'>
                      <li v-for="goods in type1Goods" @click="addorderList(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                    </ul>
                  </el-tab-pane>
                  <el-tab-pane label="饮料">
                    <ul class='cookList'>
                      <li v-for="goods in type2Goods" @click="addorderList(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                    </ul>
                  </el-tab-pane>
                  <el-tab-pane label="套餐">
                    <ul class='cookList'>
                      <li v-for="goods in type3Goods" @click="addorderList(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                    </ul>
                  </el-tab-pane>
                </el-tabs>
              </div>
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
</template>

<script>
  import axios from 'axios';
    export default {
        name: "Pos",
        data(){
          return{
            tableData:[],
            oftenGoods:[],
            type0Goods:[],
            type1Goods:[],
            type2Goods:[],
            type3Goods:[],
            totalMoney: 0, //订单总价格
            totalCount: 0  //订单商品总数量
          }
        },
        created:function(){
          axios.get('https://www.easy-mock.com/mock/5b8b30dbf032f03c5e71de7f/kuaican/oftenGoods')
            .then(response=>{
              this.oftenGoods=response.data;
            })
            .catch(error=>{
              //console.log(error);
              alert('网络错误不能访问');
            });
          axios.get('https://www.easy-mock.com/mock/5b8b30dbf032f03c5e71de7f/kuaican/typeGoods')
            .then(response=>{
              //console.log(response);
              this.type0Goods=response.data[0];
              this.type1Goods=response.data[1];
              this.type2Goods=response.data[2];
              this.type3Goods=response.data[3];
            })
            .catch(error=>{
              alert('网络错误不能访问');
            })
        },
        mounted:function () {
          var orderHeight=document.body.clientHeight;
          document.getElementById('order-list').style.height=orderHeight+'px';
        },
        methods:{
          addorderList(goods){
            /*this.totalCount=0; //汇总数量清0
            this.totalMoney=0;*/
            //判断是否这个商品已经存在于订单列表
            let isHave=false;
            for (let i = 0;i<this.tableData.length;i++){
              //console.log(this.tableData[i].goodsId);
              if (this.tableData[i].goodsId==goods.goodsId){
                isHave=true;
              }
            }
            //根据isHave的值判断订单列表中是否已经有此商品
            if(isHave){
              let arr = this.tableData.filter(o =>o.goodsId==goods.goodsId);
              arr[0].count++;
              //console.log(arr);
            }else{
              let newGoods={goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
              this.tableData.push(newGoods);
            }
            this.getAllMoney();
          },
          //删除单个商品
          delSingleGoods(goods){
            this.tableData=this.tableData.filter(o=>o.goodsId!=goods.goodsId);
            this.getAllMoney();
          },
          //删除全部商品
          delAllGoods(){
            this.tableData=[];
            this.totalCount=0;
            this.totalMoney=0;
          },
          //汇总数量和金额
          getAllMoney(){
            this.totalMoney=0;
            this.totalCount=0;
            if (this.tableData){
              //进行数量和价格的汇总计算
              this.tableData.forEach((element) => {
                this.totalCount+=element.count;
                //console.log(this.totalMoney);
                this.totalMoney=this.totalMoney+(element.price*element.count);
              });
            }
          },
          //结账
          checkout(){
            if (this.totalCount!=0){
              this.tableData=[];
              this.totalCount=0;
              this.totalMoney=0;
              this.$message({
                message:'结账成功，感谢你又为店里尽了一份力！',
                type:'success'
              });
            }else{
              this.$message.error('不能为空，老板了解你急切的心情！');
            }
          }
        }
    }
</script>

<style scoped>
  .left-order{
    background: #f9fafc;
    border-right: 1px solid #42b983;
  }
  .div-btn{
    margin-top: 20px;
  }
  .title{
    height: 20px;
    border-bottom: 1px solid #d3dce6;
    background: #f9fafc;
    padding: 10px;
  }
  .often-goods-list li{
    list-style: none;
    float:left;
    padding: 10px;
    margin: 5px;
    background: #fff;
  }
  .o-price{
    color: #58b7ff;
  }
  .goods-type{
    clear: both;
  }
  .cookList li{
    list-style: none;
    width:23%;
    border:1px solid #E5E9F2;
    height: auto;
    overflow: hidden;
    background-color:#fff;
    padding: 2px;
    float:left;
    margin: 2px;
  }
  .cookList li span{
    display: block;
    float:left;
  }
  .foodImg{
    width: 40%;
  }
  .foodName{
    font-size: 18px;
    padding-left: 10px;
    color:brown;
  }
  .foodPrice{
    font-size: 16px;
    padding-left: 10px;
    padding-top:10px;
  }
  li{
    cursor: pointer;
  }
  .tableDiv{
    background: #fff;
    height: 20px;
    padding: 10px;
    text-align: right;
    border-bottom: 1px solid #E5E9F2;
  }
</style>
