<script setup>
import { ref, onMounted } from "vue";
import service from "@/utils/request.js";
import { codeToName } from "@/utils/area-function.js";
import { useRouter } from "vue-router";
const router = useRouter();

onMounted(() => {
  getOrderDetail();
});

// 获取订单信息
const orderDetail = ref({});
const addressDetail = ref("")
const getOrderDetail = () => {
  service
    .get("/order/detail?orderId=" + window.localStorage.getItem("orderId"))
    .then((res) => {
      orderDetail.value = res.data.data;
      addressDetail.value = codeToName(orderDetail.value.provinceCode, orderDetail.value.cityCode, orderDetail.value.districtCode) + orderDetail.value.address
      countdown();
    });
};

// 取消理由
const cancelReasonList = [
  "不想要了",
  "价格有点贵",
  "收货地址拍错",
  "规格/款式拍错",
  "暂时不需要了",
  "其他",
];

//倒计时
const min = ref(0);
const sec = ref(0);
const countdown = () => {
  const now = Date.parse(new Date());
  const end = Date.parse(new Date(orderDetail.value.payLatestTime));
  const msec = end - now;

  if (msec < 0) return;

  min.value = parseInt((msec / 1000 / 60) % 60);
  sec.value = parseInt((msec / 1000) % 60);
  min.value = min.value > 9 ? min.value : "0" + min.value;
  sec.value = sec.value > 9 ? sec.value : "0" + sec.value;
  if (min.value >= 0 && sec.value >= 0) {
    //倒计时结束关闭订单
    if (min.value == 0 && sec.value == 0) {
      return;
    }
    setTimeout(function () {
      countdown();
    }, 1000);
  }
};

// 支付
const pay = () => {
  service
    .post("/order/pay?orderId=" + window.localStorage.getItem("orderId"))
    .then((res) => {
      if (res.data.code == 200) {
        router.push({ path: "/order" });
      }
    });
};

// 取消订单
const cancelReason = ref("")
const cancel = () => {
  service
    .post("/order/cancel?orderId=" + window.localStorage.getItem("orderId") + "&&cancelReason=" + cancelReason.value)
    .then((res) => {
      if (res.data.code == 200) {
        setTimeout(() => {
          router.push({ path: "/order" });
        }, 500);
      }
    });
}
</script>

<template>
  <div class="checkout-page-wrapper pt-120 pb-90 pt-sm-58 pb-sm-54">
    <div class="container">
      <div class="row">
        <div class="col-12">
          <div class="checkoutaccordion" id="checkOutAccordion">
            <div class="card">
              <h3 class="mb-0" style="display: inline-block">
                <span style="color: #000; font-size: 18px; margin: 30px">剩余支付时间（超时自动关闭）</span>
                <span class="time">{{ min }}分钟 {{ sec }}秒</span>
              </h3>
            </div>
            <div class="card">
              <h3 class="mb-0" style="display: inline-block">
                <span style="color: #000; font-size: 18px; margin: 30px">地址</span>
                <span style="color: #ff7e67">{{ addressDetail }}</span>
              </h3>
            </div>
          </div>
        </div>
      </div>
      <div style="margin-top: 40px">
        <div class="order-summary-details mt-md-26 mt-sm-26">
          <div class="order-summary-content mb-sm-4">
            <div class="order-summary-table table-responsive text-center mt-0">
              <table class="table table-bordered">
                <thead>
                  <tr>
                    <th style="width: 140px">Thumbnail</th>
                    <th>Products</th>
                    <th>Quantity</th>
                    <th>Total</th>
                    <th>Remark</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in orderDetail.orderItemVOList">
                    <td><img :src="item.specImage" /></td>
                    <td>{{ item.productName }} - {{ item.specName }}</td>
                    <td>× {{ item.amount }}</td>
                    <td>￥{{ item.price }}+{{ item.freight }}</td>
                    <td>{{ item.remark }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
      <div class="d-block d-md-flex w-30" style="display: float; float: left; width: 30%; margin-top: 30px">
        <div class="cart-update">
          <a class="sqr-btn" @click="cancel">取消订单</a>
        </div>
        <select class="form-select mx-4" style="
            height: 42px;
            display: inline-block;
            width: 60%;
            padding-left: 0px;
            background-color: #f7f7f7;
            border: 1px solid #cccccc;
            border-radius: 0;
            text-indent: 6px;
          " aria-label="Default select example" v-model="cancelReason">
          <option v-for="item in cancelReasonList" :value="item">{{ item }}</option>
        </select>
      </div>
      <div class="d-block d-md-flex justify-content-between" style="display: float; float: right; width: 30%">
        <div class="cart-update ml-auto">
          <span class="sqr-btn-f" style="margin-top: 30px">合计: ￥{{ orderDetail.payment }}元</span>
          <a class="sqr-btn text-white" @click="pay()">支付</a>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
