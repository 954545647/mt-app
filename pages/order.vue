<template>
  <div class="page-order">
    <el-row class="page">
      <el-col :span="4" class="navbar">
        <h3>我的美团</h3>
        <dl>
          <dt>我的订单</dt>
          <dd>
            全部订单
            <i class="el-icon-arrow-right"></i>
          </dd>
          <dd>
            待付款
            <i class="el-icon-arrow-right"></i>
          </dd>
          <dd>
            待使用
            <i class="el-icon-arrow-right"></i>
          </dd>
          <dd>
            待评价
            <i class="el-icon-arrow-right"></i>
          </dd>
          <dd>
            退款/售后
            <i class="el-icon-arrow-right"></i>
          </dd>
        </dl>
        <dl>
          <dt>我的收藏</dt>
          <dd>
            收藏的商家
            <i class="el-icon-arrow-right"></i>
          </dd>
          <dd>
            收藏的团购
            <i class="el-icon-arrow-right"></i>
          </dd>
        </dl>
        <dl>
          <dt>抵用券</dt>
          <dd>
            可用劵
            <i class="el-icon-arrow-right"></i>
          </dd>
          <dd>
            失效劵
            <i class="el-icon-arrow-right"></i>
          </dd>
        </dl>
        <dl>
          <dt>个人信息</dt>
          <dd>
            帐号设置
            <i class="el-icon-arrow-right"></i>
          </dd>
        </dl>
      </el-col>
      <el-col :span="19" class="table">
        <el-tabs v-model="activeName" @tab-click="handleClick">
          <el-tab-pane label="全部订单" name="all">
            <list :cur="list"/>
          </el-tab-pane>
          <el-tab-pane label="待付款" name="unpay">
            <list :cur="Unpay"/>
          </el-tab-pane>
          <el-tab-pane label="待评价" name="unuse">
            <list :cur="HasPay"/>
          </el-tab-pane>
          <el-tab-pane label="已评价" name="unreplay">
            <list :cur="HasComment"/>
          </el-tab-pane>
        </el-tabs>
      </el-col>
    </el-row>
  </div>
</template>


<script>
import list from "@/components/order/order.vue";
export default {
  components: {
    list
  },
  data() {
    return {
      activeName: "all",
      list: [], //全部数据
      cur: [], //用来父子传递的
      Unpay: [], //未付款
      HasPay: [], //已经付款的
      HasComment: []//已经评论的
    };
  },
  watch: {
    activeName: function(val) {}
  },
  methods: {
    handleClick() {},
    unUse() {}
  },
  mounted() {
    let city = sessionStorage.getItem("currentCity");
  },
  async asyncData(ctx) {
    let {
      status,
      data: { user }
    } = await ctx.$axios.get("/users/getUser");
    let {
      status: status2,
      data: { code, list }
    } = await ctx.$axios.post("/order/getOrders", {
      user: user
    });
    if (status2 === 200 && code === 0 && list.length) {
      // 必须返回对象的形式,不然不会和data中的数据融合
      return {
        // 全部数据
        list: list.map(item => {
          return {
            img: item.imgs.length ? item.imgs[0].url : "/logo.png",
            name: item.name,
            count: item.count,
            total: item.total,
            status: item.status,
            statusTxt: item.status === 0 ? "待付款" : "已付款",
            id: item.id
          };
        }),
        // 未付款
        Unpay: list
          .filter(item => {
            return !item.status;
          })
          .map(item => {
            return {
              img: item.imgs.length ? item.imgs[0].url : "@/3.png",
              name: item.name,
              count: item.count,
              total: item.total,
              status: item.status,
              statusTxt: "待付款",
              id: item.id
            };
          }),
        // 已经付款了的!
        HasPay: list
          .filter(item => {
            return item.status===1;
          })
          .map(item => {
            return {
              img: item.imgs.length ? item.imgs[0].url : "@/3.png",
              name: item.name,
              count: item.count,
              total: item.total,
              status: item.status,
              statusTxt: "已付款",
              id: item.id
            };
          }),

        // 已经评价了的!
        HasComment: list
          .filter(item => {
            return item.status === 2;
          })
          .map(item => {
            return {
              img: item.imgs.length ? item.imgs[0].url : "@/3.png",
              name: item.name,
              count: item.count,
              total: item.total,
              status: item.status,
              statusTxt: "已评价",
              id: item.id
            };
          })
      };
    }
  }
};
</script>


<style lang="scss">
@import "@/assets/css/order/index.scss";
.page {
  box-sizing: border-box;
  .navbar {
    margin-right: 5px;
    background: #fff;
    border: 1px solid #e5e5e5;
    border-radius: 4px;
    padding-bottom: 20px;
    font-size: 16px;
    color: #222;
    box-sizing: border-box;
    padding: 30px 20px 20px 30px;
    box-sizing: border-box;
    > h3 {
      font-size: 16px;
      font-weight: 500;
      margin-bottom: 10px;
    }
    dd {
      font-size: 14px;
      color: #666;
      margin: 15px 0;

      > i {
        float: right;
        margin-top: 3px;
      }
    }
  }
  .el-tabs__item {
    font-size: 18px;
    color: #222;
    line-height: 26px;
    text-align: center;
    height: 38px;
    font-weight: 500;
    cursor: pointer;
  }
  .clearfloat:after {
    display: block;
    clear: both;
    content: "";
    width: 0;
    height: 0;
    visibility: hidden;
  }
}
</style>
