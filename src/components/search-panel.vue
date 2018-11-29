<template>
  <div>
    <!-- 组件不能写到template的根节点上，即每个组件只有一个根节点，这里的div就是这个template的根节点 -->
    <!--  getindex是自定义事件 -->
    <logo-select @getindex='getIndex'></logo-select>

    <div class="search-input">
      <!-- $event是实参，表示event对象 -->
      <!--
                输入搜索内容即时搜索，所以有一个keyup事件。
                按回车键有一个进入搜索内容页面，所以有一个keydown.enter事件
                按上下键可以选择列表条目
            -->
      <input
        placeholder="输入18位身份证信息"
        type="text"
        v-model="keyword"
      >
      <!-- 这是一个小叉叉，点击它可清除输入框内容 -->
      <button
        class="search-btn"
        @click="search()"
      >查询</button>
      <div class="search-select">
        <!-- transition-group也是vue2.0中的新特性,tag='ul'表示用ul包裹v-for出来的li -->
        <transition-group
          name="itemfade"
          tag="ul"
          mode="out-in"
          v-cloak
        >
          <li
            v-for="(value,index) in myData"
            :class="{selectback:index==now}"
            class="search-select-option search-select-list"
            @mouseover="selectHover(index)"
            @click="selectClick(index)"
            :key="value"
          >
            {{value}}
          </li>
        </transition-group>
      </div>
    </div>
    <Modal
      v-model="showModal"
      title="查询完成"
      class-name="vertical-center-modal"
      @on-ok="ok"
      @on-cancel="cancel"
    >
      <p>支付¥16.9的查看结果</p>
    </Modal>
  </div>
</template>

<script type="text/javascript">
// import xxx from someSrc     es6中得到模块的方法。
import logoSelect from "./logo-select.vue";
import { Button, Modal, Col, Spin } from "iview";
import md5 from "js-md5";

var Wi = [7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2, 1];
// 身份证验证位值.10代表X
var ValideCode = [1, 0, 10, 9, 8, 7, 6, 5, 4, 3, 2];
const that = this;
export default {
  //注册组件
  components: {
    "logo-select": logoSelect,
    Modal
  },
  data: function() {
    return {
      showModal: false,
      showloading: false,
      myData: [], //用来接收ajax得到的数据
      keyword: "", //v-model绑定的输入框的value
      now: -1,
      searchIndex: 0,
      logoData: [
        {
          name: "360搜索",
          searchSrc: "https://www.so.com/s?ie=utf-8&shb=1&src=360sou_newhome&q="
        },
        {
          name: "百度搜索",
          searchSrc:
            "https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd="
        },
        {
          name: "搜狗搜索",
          searchSrc: "https://www.sogou.com/web?query="
        }
      ]
    };
  },
  created: function() {
    console.log("sdfdf" + location.host);
    var shengfengzheng = this.getUrlParam("sfz");
    if (shengfengzheng) {
      this.$Modal.info({
        title: shengfengzheng + "入住记录查询结果",
        content: "没有查询到" + shengfengzheng + "的入住记录"
      });
    }
  },
  methods: {
    getUrlParam: function(key) {
      var name, value;
      var str = location.href; //取得整个地址栏
      var num = str.indexOf("?");
      str = str.substr(num + 1); //取得所有参数   stringvar.substr(start [, length ]

      var arr = str.split("&"); //各个参数放到数组里
      for (var i = 0; i < arr.length; i++) {
        num = arr[i].indexOf("=");
        if (num > 0) {
          name = arr[i].substring(0, num);
          value = arr[i].substr(num + 1);
          if (name == key) return value;
        }
      }
    },
    ok() {
      var name = this.getUrlParam("id") ? this.getUrlParam("id") : "yy";
      var pay_type = "wechat";
      var price = 0.01;
      var order_id = "111";
      var order_uid = "sdfsdf";
      var notify_url = location.host + "notify_url.php"; // 回调通知地址
      var return_url = location.host + "?sfz=" + this.keyword; // 支付成功页面跳转地址
      var feedback_url = "";
      var secret = "04182da4271d48a2b37a6d143a48fdf8"; // app secret, 在个人中心配置页面查看

      var sign = md5(
        name +
          pay_type +
          price +
          order_id +
          order_uid +
          notify_url +
          return_url +
          feedback_url +
          secret
      );
      var api_url = "https://bufpay.com/api/pay/1138?format=json";
      var data = {
        name: name,
        pay_type: pay_type,
        price: price,
        order_id: order_id,
        order_uid: order_uid,
        notify_url: notify_url,
        return_url: return_url,
        feedback_url: feedback_url,
        sign: sign
      };

      window.open(
        location.host +
          "bufpay_php_demo.php?" +
          "price =" +
          price +
          "&order_id=" +
          order_id +
          "&name=" +
          name +
          "&return_url=" +
          decodeURIComponent(return_url)
      );

      //   this.$http
      //     .jsonp(
      //       "https://sug.so.360.cn/suggest?word=" +
      //         this.keyword +
      //         "&encodein=utf-8&encodeout=utf-8"
      //     )
      //     .then(function(res) {
      //       this.myData = res.data.s;
      //     });

      //   this.$http.post(api_url, data).then(data => {
      //     console.log("dddd" + JSON.stringify(data));
      //   });
    },
    cancel() {},
    // &event是实参，表示event对象
    get: function(ev) {
      // 如果按得键是上或者下，就不进行ajax
      if (ev.keyCode == 38 || ev.keyCode == 40) {
        return;
      }

      this.$http
        .jsonp(
          "https://sug.so.360.cn/suggest?word=" +
            this.keyword +
            "&encodein=utf-8&encodeout=utf-8"
        )
        .then(function(res) {
          this.myData = res.data.s;
        });
    },
    selectDown: function() {
      this.now++;
      //到达最后一个时，再按下就回到第一个
      if (this.now == this.myData.length) {
        this.now = 0;
      }
      this.keyword = this.myData[this.now];
    },
    selectUp: function() {
      this.now--;
      //同上
      if (this.now == -1) {
        this.now = this.myData.length - 1;
      }
      this.keyword = this.myData[this.now];
    },
    search: function() {
      if (!this.IdentityCodeValid(this.keyword)) {
        // if(!this.IdCardValidate(this.keyword)){
        this.$Modal.error({
          title: "错误",
          content: "身份证格式错误"
        });
      } else {
        this.$Spin.show({
          render: h => {
            return h("div", [
              h("Icon", {
                class: "demo-spin-icon-load",
                props: {
                  type: "ios-loading",
                  size: 18
                }
              }),
              h("div", "查询中")
            ]);
          }
        });
        setTimeout(() => {
          this.$Spin.hide();
          this.showModal = true;
        }, 300);
      }

      //  this.$Modal.success({
      //                             title: "支付",
      //                             content: "查询需要支付¥16.9 的查询费用"
      //                         });
      //打开对应的搜索界面
      // window.open(this.logoData[this.searchIndex].searchSrc + this.keyword);
    },
    selectHover: function(index) {
      this.now = index;
    },
    selectClick: function(index) {
      this.keyword = this.myData[index];
      this.search();
    },
    clearInput: function() {
      this.keyword = "";
      this.$http
        .jsonp(
          "https://sug.so.360.cn/suggest?word=" +
            this.keyword +
            "&encodein=utf-8&encodeout=utf-8"
        )
        .then(function(res) {
          this.myData = res.data.s;
        });
    },
    getIndex: function(index) {
      this.searchIndex = index;
    },
    IdentityCodeValid: function(code) {
      return true;
      var city = {
        11: "北京",
        12: "天津",
        13: "河北",
        14: "山西",
        15: "内蒙古",
        21: "辽宁",
        22: "吉林",
        23: "黑龙江 ",
        31: "上海",
        32: "江苏",
        33: "浙江",
        34: "安徽",
        35: "福建",
        36: "江西",
        37: "山东",
        41: "河南",
        42: "湖北 ",
        43: "湖南",
        44: "广东",
        45: "广西",
        46: "海南",
        50: "重庆",
        51: "四川",
        52: "贵州",
        53: "云南",
        54: "西藏 ",
        61: "陕西",
        62: "甘肃",
        63: "青海",
        64: "宁夏",
        65: "新疆",
        71: "台湾",
        81: "香港",
        82: "澳门",
        91: "国外 "
      };
      var tip = "";
      var pass = true;

      if (
        !code ||
        !/^\d{6}(18|19|20)?\d{2}(0[1-9]|1[12])(0[1-9]|[12]\d|3[01])\d{3}(\d|X)$/i.test(
          code
        )
      ) {
        tip = "身份证号格式错误";
        pass = false;
      } else if (!city[code.substr(0, 2)]) {
        tip = "地址编码错误";
        pass = false;
      } else {
        //18位身份证需要验证最后一位校验位
        if (code.length == 18) {
          code = code.split("");
          //∑(ai×Wi)(mod 11)
          //加权因子
          var factor = [7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2];
          //校验位
          var parity = [1, 0, "X", 9, 8, 7, 6, 5, 4, 3, 2];
          var sum = 0;
          var ai = 0;
          var wi = 0;
          for (var i = 0; i < 17; i++) {
            ai = code[i];
            wi = factor[i];
            sum += ai * wi;
          }
          var last = parity[sum % 11];
          if (parity[sum % 11] != code[17]) {
            tip = "校验位错误";
            pass = false;
          }
        }
      }
      if (!pass);
      return pass;
    }
  }
};
</script>

<style type="text/css">
.demo-spin-icon-load {
  animation: ani-demo-spin 1s linear infinite;
}
@keyframes ani-demo-spin {
  from {
    transform: rotate(0deg);
  }
  50% {
    transform: rotate(180deg);
  }
  to {
    transform: rotate(360deg);
  }
}
.demo-spin-col {
  height: 100px;
  position: relative;
  border: 1px solid #eee;
}
.search-input {
  height: 65px;
  width: 600px;
  margin: 0 auto;
  margin-top: 10px;
  position: relative;
}
.vertical-center-modal {
  display: flex;
  align-items: center;
  justify-content: center;

  .ivu-modal {
    top: 0;
  }
}
.search-input input {
  border: 1px solid #e4e4e4;
  box-sizing: border-box;
  width: 500px;
  height: 65px;
  font-size: 18px;
  float: left;
  padding-left: 10px;
  padding-right: 10px;
  overflow: hidden;
}

.search-btn {
  height: 65px;
  width: 100px;
  border: 1px solid mediumseagreen;
  background-color: mediumseagreen;
  color: white;
  font-size: 18px;
  font-weight: bold;
  float: left;
}

.search-btn {
  cursor: pointer;
}

.search-select {
  position: absolute;
  top: 45px;
  width: 500px;
  box-sizing: border-box;
  z-index: 999;
}

.search-select li {
  border: 1px solid #d4d4d4;
  border-top: none;
  border-bottom: none;
  background-color: #fff;
  width: 100%;
}

.search-select-option {
  box-sizing: border-box;
  padding: 7px 10px;
}

.selectback {
  background-color: #eee !important;
  cursor: pointer;
}

input::-ms-clear {
  display: none;
}

.search-reset {
  width: 21px;
  height: 21px;
  position: absolute;
  display: block;
  line-height: 21px;
  text-align: center;
  cursor: pointer;
  font-size: 20px;
  right: 110px;
  top: 12px;
}

.search-select-list {
  transition: all 0.5s;
}

.itemfade-enter,
.itemfade-leave-active {
  opacity: 0;
}

.itemfade-leave-active {
  position: absolute;
}

.selectback {
  background-color: #eee !important;
  cursor: pointer;
}
.search-select ul {
  margin: 0;
  text-align: left;
}
</style>
