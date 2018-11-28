<template>
  <div id="app">

<panel> </panel>

  <!-- use the modal component, pass in the prop -->


  </div>
</template>

<script>
import panel from './components/search-panel.vue';
import {Button, Modal} from 'iview';

export default {
   data: function() {
        return {
    showModal: false,

            myData: [],//用来接收ajax得到的数据
            keyword: '',//v-model绑定的输入框的value
            now: -1,
            searchIndex: 0,
            logoData: [{
                name: '360搜索',
                searchSrc: 'https://www.so.com/s?ie=utf-8&shb=1&src=360sou_newhome&q='
            }, {
                name: '百度搜索',
                searchSrc: 'https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd='
            }, {
                name: '搜狗搜索',
                searchSrc: 'https://www.sogou.com/web?query='
            }]
        }
    },
  components: {
    panel,Button,Modal
  },
   methods: {
       ok () {
                console.log("dsfddddd");

            },
            cancel () {
                     console.log("dsfddddd");

            },
     clixx: function() {
       console.log("dsfddddd");
       this.showModal = true;
     }
   }
}
</script>

<style>
html {
  height: 100%;
}

body {
  height: 100%;
  background: url('./assets/background.png') no-repeat;
  background-size: 100% 100%
}

#app {
  color: #2c3e50;
  width: 600px;
  font-family: Source Sans Pro, Helvetica, sans-serif;
  text-align: center;
  position: absolute;
  top: 35%;
  left: 50%;
  margin-left: -300px;
  margin-top: -100px;
}

</style>

