<template>
  <view class="sessions">
    <view class="film_info">
      <view class="film_name">变形金刚：超能勇士崛起</view>
      <view class="film_time">2023-06-08 23:59:00 原版 3D</view>
    </view>
    <view class="hall_name render-nimation">
      {{ hallName }}
    </view>
    <movable-area class="seating_map">
      <movable-view  direction="all" :inertia="true" :scale="true" :scale-min="1"
                    :scale-max="2" @scale="seatScale" @change="seatChange"
                    :style="{width: scaleNum*90+'vw',height: 500+'rpx',marginTop:scaleNum*10+'rpx',marginLeft:scaleNum*5+'vw'}">

        <view v-for="(item,index) in seatLists" :key="index" class="rowList render-nimation" :style="{marginLeft:scaleNum*5+'vw'}">
          <view v-for="(item2,index2) in item" :key="index2" class="columnNoList"
                :style="{width:seatWidth+'rpx',height:seatHeight+'rpx',border:item2.seatNo?'':'none'}"
                @click="selectSeat(item2)">
            <!-- 普通座位 -->
            <block v-if="item2.lovestatus == 0">
              <!-- 普通--未选座位 -->
              <block v-if="item2.type === 0">
                <!-- 普通--座位价格存在 -->
                <block v-if="item2.price">
                  <!-- 未分区座位 -->
                  <image
                      v-if="item2.price == settlePrice ||item2.price == Math.ceil(settlePrice)"
                      src="@/static/unselected.png"
                      mode="aspectFit"
                  >
                  </image>
                  <!-- 分区座位 -->
                  <image
                      v-else
                      src="@/static/unselected2.png"
                      mode="aspectFit"
                  ></image>
                </block>
                <!-- 普通--座位价格不存在 -->
                <block v-else>
                  <image
                      src="@/static/unselected.png"
                      mode="aspectFit"
                  ></image>
                </block>
              </block>
              <!-- 普通--已选座位 -->
              <image v-else-if="item2.type === 1" src="@/static/iselected.png" mode="aspectFit"></image>
              <!-- 普通--已售座位 -->
              <image v-else-if="item2.type === 2" src="@/static/bought.png" mode="aspectFit"></image>
            </block>
            <!-- 情侣座位 左边-->
            <block v-if="item2.lovestatus == 1">
              <image
                  v-if="item2.type === 0"
                  src="@/static/loveseatleft.png"
                  mode="aspectFit"
              ></image>
              <image
                  v-if="item2.type === 1"
                  src="@/static/icon_seat_lovers.png"
                  mode="aspectFit"
              ></image>
              <image
                  v-if="item2.type === 2"
                  src="@/static/loveseatysleft.png"
                  mode="aspectFit"
              ></image>
            </block>
            <!-- 情侣座位 右边-->
            <block v-if="item2.lovestatus == 2">
              <image
                  v-if="item2.type === 0"
                  src="@/static/loveseatright.png"
                  mode="aspectFit"
              ></image>
              <image
                  v-if="item2.type === 1"
                  src="@/static/icon_seat_lovers.png"
                  mode="aspectFit"
              ></image>
              <image
                  v-if="item2.type === 2"
                  src="@/static/loveseatysright.png"
                  mode="aspectFit"
              ></image>
            </block>
          </view>
        </view>
      </movable-view>
    </movable-area>
    <view class="row_tips render-nimation" :style="{top:(y+285)*scaleNum+'rpx',transform: 'scale(' + scaleNum + ')'}">
      <span v-for="item in seatLists.length" :key="item" :style="{height:seatHeight+'rpx',lineHeight:seatHeight+'rpx'}">{{item+1}}</span>
    </view>
  </view>
</template>

<script>
import seatDate from './seat.json'
export default {
  data() {
    return {
      hallName:'', // 影厅名称
      seatLists:[], // 座位图
      columnNo:0, // 最大列数
      newDate:[], // 初始化后的座位数据
      scaleNum:1, // 缩放倍数
      windowWidth:300, // 屏幕宽度
      x:0, // x轴坐标
      y:0, // y轴坐标
      seatWidth:17, // 座位的 width
      seatHeight:17, // 座位的 height
      settlePrice:0, // 座位最低价格(非分区价格)
    };
  },
  watch: {
    seatLists(newVal, oldVal) {
      setTimeout(() => this.init(), 200)
    },
  },
  onLoad() {
    console.log('seatDate',seatDate)
    this.hallName = seatDate.data.show_info.hall_name
    this.settlePrice = seatDate.data.show_info.settle_price
    this.initializationDate(seatDate.data.seat_data.seats,JSON.parse(seatDate.data.show_info.area_price))
    this.windowWidth = uni.getSystemInfoSync().windowWidth-75;
  },
  mounted() {
    this.init()
  },
  methods: {
    /*
    * 初始化座位数据
    * seatDate：原始座位数据
    * newDate：新的座位数据
    * oldDate：旧的座位数据
    * area_price：分区价格
    * columnNo：最大列数
    * */
    initializationDate(oldSeat,areaPrice) {
      let newDate = [],
          oldDate = oldSeat,
          area_price = areaPrice,
          columnNo=0
      oldDate.forEach((item) => {
        if(item.columnNo>columnNo)columnNo = item.columnNo
        area_price.forEach((areaItem)=>{
          if(item.areaId == areaItem.area) item.price = areaItem.price.user_price
        })
        if (!newDate[item.rowNo]) newDate[item.rowNo]=[]
        newDate[item.rowNo].push(item)
      })
      this.newDate = newDate
      this.columnNo = columnNo
      this.seatWidth = this.seatHeight = (this.windowWidth*2)/columnNo
      this.renderingSeats()
    },
    /*
    *
    * 渲染座位图
    *
    * */
    renderingSeats() {
      let seatArr = [],
          seatDefault = {
            type: -1,
            SeatCode: "",
            RowNum: "",
            ColumnNum: "",
            lovestatus: 0,
            seatNo: "",
            price: 0,
            area_id: "",
          }
      for (let i = 0; i < this.newDate.length-1; i++) {
        seatArr[i] = new Array(this.columnNo);
        for (let j = 0; j < this.columnNo; j++) {
          seatArr[i][j] = this.newDate[i+1][j] ? this.newDate[i+1][j]: seatDefault;
          seatArr[i][j].type = seatArr[i][j].status === 'N'?0:seatArr[i][j].status === 'LK'?2:''
        }
      }
      this.seatLists = seatArr
    },
    /*
    * 座位缩放
    * */
    seatScale(obj){
      setTimeout(()=>{
        obj.detail.scale>1.5?this.scaleNum = 2:this.scaleNum =1
        this.init()
      },1000)
    },
    /*
    * 座位拖动
    * */
    seatChange(obj){
      this.y = obj.detail.y*2
    },
    /*
    * 点击座位
    * */
    selectSeat(val){
      console.log('val',val.seatNo)
    },
    /*
    * 获取座位图位置信息
    * */
    init(){
      const query = uni.createSelectorQuery().in(this)
      query
          .select('.rowList')
          .boundingClientRect(res => {
            if (res) {
             console.log('座位图的位置',res)
            }
          })
          .exec()
    }
  },
};
</script>

<style lang="scss">
page {
  background-color: #F8F8F8;
}

.sessions {
  .film_info {
    margin-left: 30rpx;

    .film_name {
      font-size: 34rpx;
      font-weight: bold;
    }

    .film_time {
      color: #666666;
      font-size: 28rpx;
      margin-top: 15rpx;
    }
  }
  .hall_name {
    width: 100%;
    height: 136rpx;
    text-align: center;
    line-height: 136rpx;
    font-size: 22rpx;
    font-weight: 400;
    color: #9b9d9b;
    background-image: url("../../static/yingmu.png");
    background-size: 100% 100%;
    margin-top: 50rpx;
    z-index: 99;
  }

  .seating_map {
    width: 100vw;
    height: 600rpx;


    .rowList {
      display: flex;
      align-items: center;
      margin-top: 15rpx;

      .columnNoList {
        margin-left: 2rpx;
        display: flex;
        flex-direction: column;
        align-items: center;

        image {
          width: 100%;
          height: 100%;
        }
      }
    }

  }
  .row_tips{
    width: 35rpx;
    background-color: #2C405A;
    color: #FFFFFF;
    display: flex;
    flex-direction: column;
    align-items: center;
    border-radius: 20rpx;
    font-size: 24rpx;
    position: fixed;
    top: 285rpx;
    left: 30rpx;
    padding-top: 15rpx;
    transition: all 0.3s;
    span{
      display: inline-block;
      margin-bottom: 15rpx;
    }
  }
}
.render-nimation{
  animation-name: circle-in-center;
  animation-duration: .3s;
}

@keyframes circle-in-center {
  0% {
    opacity: 0;
    transform: scale(.2)
  }

  100% {
    opacity: 1;
    transform: scale(1)
  }
}

</style>
