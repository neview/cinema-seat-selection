<template>
	<view class="page">
    <view class="tips show_tips" :class="tipsIndex === item ? 'hide_tips':''" v-for="item in list" :key="item">{{item}}</view>
    <view class="add_tips" @click="addTips">add</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
        list:[],
        tipsIndex:0,
        num:0
			}
		},
		onLoad(option) {
		},
		methods: {
      addTips(){
          let timer = setInterval(()=>{
            if(this.list.length > 7 ){
              clearInterval(timer)
              return
            }
            this.num++
            this.list.unshift(this.num)
          },100)
          if(this.list.length > 3){
            this.tipsIndex = this.list[this.list.length-1]
            setTimeout(()=>{
              this.list.splice(this.list.length-1,1)
            },300)
          }
      }
		}
	}
</script>

<style>
.page{

}
.tips{
  width: 50vw;
  height: 70rpx;
  background-color: #3F536E;
  border-radius: 20rpx;
  margin: 0 auto;
  margin-top: 20rpx;
  color: #FFFFFF;
  text-align: center;
  line-height: 70rpx;
}
.add_tips{
  position: fixed;
  bottom: 20rpx;
  left: 50px;
}

.show_tips{
  animation: showTips 0.5s;
  animation-fill-mode: forwards;
}

@keyframes showTips {
  0%{
    height: 0;
    transform: scale(0);
  }
  100%{
    height: 70rpx;
    transform: scale(1);
  }
}

.hide_tips{
  animation: hideTips 0.5s;
  animation-fill-mode: forwards;
}

@keyframes hideTips {
  0%{
    transform: scale(1);
  }
  100%{
    transform: scale(0);
  }
}
</style>
