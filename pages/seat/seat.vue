<template>
	<view>
    <view class="film_info">
      <view class="film_name">变形金刚：超能勇士崛起</view>
      <view class="film_time">2023-06-08 23:59:00 原版 3D</view>
    </view>
    <view class="hall_name render_nimation_anim">
      vip 影厅
    </view>
    <movable-area  class="seating_map">
      <movable-view class="movable_view" direction="all" :scale="true" :scale-min="1" :scale-max="1.5" :x="movableX" :y="movableY" :scale-value="scaleValue" @change="dragChange" @scale="scaleChange">
        <view :style="{transform: `translateX(${xValue}px) translateY(${yValue}px) scale(${gradeScale})`,transition:executionTime}" @touchstart="touchstart" @touchmove="touchmove" @touchend="touchend">
          <view v-for="(item,index) in seatLists" :key="index" class="row_list render_nimation_anim">
            <view v-for="(item2,index2) in item" :key="index2" class="columnNo_list" @click="selectSeat(item2)">
              <!-- 普通座位 -->
              <block v-if="item2.lovestatus == 0">
                <!-- 普通--未选座位 -->
                <block v-if="item2.type === 0">
                  <!-- 普通--座位价格存在 -->
                  <block v-if="item2.price">
                    <!-- 未分区座位 -->
                    <image
                        v-if="item2.price == settlePrice || item2.price == Math.ceil(settlePrice)"
                        src="@/static/unselected.png"
                        mode="aspectFit"
                        :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                    >
                    </image>
                    <!-- 分区座位 -->
                    <image
                        v-else
                        src="@/static/unselected2.png"
                        mode="aspectFit"
                        :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                    ></image>
                  </block>
                  <!-- 普通--座位价格不存在 -->
                  <block v-else>
                    <image
                        src="@/static/unselected.png"
                        mode="aspectFit"
                        :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                    ></image>
                  </block>
                </block>
                <!-- 普通--已选座位 -->
                <image v-else-if="item2.type === 1" src="@/static/iselected.png" mode="aspectFit" :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"></image>
                <!-- 普通--已售座位 -->
                <image v-else-if="item2.type === 2" src="@/static/bought.png" mode="aspectFit" :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"></image>
              </block>
              <!-- 情侣座位 左边-->
              <block v-if="item2.lovestatus == 1">
                <image
                    v-if="item2.type === 0"
                    src="@/static/loveseatleft.png"
                    mode="aspectFit"
                    :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                ></image>
                <image
                    v-if="item2.type === 1"
                    src="@/static/icon_seat_lovers.png"
                    mode="aspectFit"
                    :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                ></image>
                <image
                    v-if="item2.type === 2"
                    src="@/static/loveseatysleft.png"
                    mode="aspectFit"
                    :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                ></image>
              </block>
              <!-- 情侣座位 右边-->
              <block v-if="item2.lovestatus == 2">
                <image
                    v-if="item2.type === 0"
                    src="@/static/loveseatright.png"
                    mode="aspectFit"
                    :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                ></image>
                <image
                    v-if="item2.type === 1"
                    src="@/static/icon_seat_lovers.png"
                    mode="aspectFit"
                    :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                ></image>
                <image
                    v-if="item2.type === 2"
                    src="@/static/loveseatysright.png"
                    mode="aspectFit"
                    :style="{width:itemImgSize+'px',height:itemImgSize+'px'}"
                ></image>
              </block>
            </view>
          </view>
        </view>
      </movable-view>
    </movable-area>
    <view class="grade_list grade_show_anim" :style="{top:gradeTop+'px',transform:`scale(${gradeScale}) translateY(${yValue}px)`,transition:executionTime}">
      <span v-for="item in gradeList" :key="item" :style="{height:itemImgSize+'px',lineHeight:itemImgSize+'px'}">{{item}}</span>
    </view>
	</view>
</template>

<script>
import seatDate from './seat.json'
	export default {
		data() {
			return {
        scaleValue:1,
        seatLists:[],
        seatCol:0,
        seatRow:0,
        minCol:0,
        minRow:0,
        originalSeat:[],
        areaPrice:'',
        settlePrice:'',
        windowWidth:0,
        startX:0,
        startY:0,
        xValue:0,
        yValue:0,
        gradeList:[],
        itemImgSize:0,
        gradeTop:0,
        gradeScale:1,
        executionTime:'none',
        initialDistance:0,
        initialScale:0,
        movableX:0,
        movableY:0
			}
		},
    watch:{
      seatLists(){
        setTimeout(()=>{
          this.init()
        },100)
      }
    },
		onLoad(option) {
      let settle_price = seatDate.data.show_info.settle_price,
          area_price = JSON.parse(seatDate.data.show_info.area_price) || ''
      this.areaPrice = area_price
      this.settlePrice = settle_price
      this.originalSeat = seatDate.data.seat_data.seats
      this.windowWidth = uni.getSystemInfoSync().windowWidth
      this.layoutStructure(this.originalSeat)
      setTimeout(()=>{
        this.init()
      },100)
		},
    onReady(){
      setTimeout(()=>{
        this.init()
      },100)
    },
		methods: {
      /*
       *  计算行列
       *  date 原始座位数据
       *  price 分区价格
       * */
      layoutStructure(date){
        let result = date.reduce(
            ({ minCol, minRow, maxCol, maxRow }, item) => ({
              minCol: Math.min(minCol, item.columnNo),
              minRow: Math.min(minRow, item.rowNo),
              maxCol: Math.max(maxCol, item.columnNo),
              maxRow: Math.max(maxRow, item.rowNo)
            }),
            { minCol: date[0].columnNo, minRow: date[0].rowNo, maxCol: date[0].columnNo, maxRow: date[0].rowNo }
        );

        let { minCol, minRow, maxCol, maxRow } = result;
        this.seatCol = maxCol - minCol + 1
        this.seatRow = maxRow - minRow + 1
        this.minCol = minCol
        this.minRow = minRow
        // this.xValue = (this.windowWidth - ((this.windowWidth * this.seatCol * 60) / 750)) / 2
        this.itemImgSize = ((this.windowWidth - this.convertPX(45)) / this.seatCol) - this.convertPX(10)
        this.initializationSeat()
      },
      /*
       *  初始化座位数据
       *  columnNo 纵坐标  rowNo 横坐标  areaId 分区ID  seatId 座位ID seatNo 座位名称
       *  status 售罄状态 (N可售，LK不可售)
       *  lovestatus 是否情侣座 (0为非情侣座 1为情侣座左 2为情侣座右)
       * */
      initializationSeat(){
        let seatArr = Array(this.seatRow).fill([]).map(item=>Array(this.seatCol).fill({
          type: -1,
          SeatCode: "",
          RowNum: "",
          ColumnNum: "",
          lovestatus: 0,
          seatNo: "",
          price: 0,
          area_id: "",
        }));
        let gradeArr = []
        for (let i = 0;i<this.originalSeat.length;i++){
          let {columnNo,rowNo,status,areaId,seatId,lovestatus,seatNo,seat_type} = this.originalSeat[i];
          let seatStatus = status === "N" ? 0: status === "LK" ? 2 : -1,
              price = 0;
          if(Array.isArray(this.areaPrice) && this.areaPrice.filter(item=>item.area == areaId).length>0){
            price = this.areaPrice.filter(item=>item.area == areaId)[0].price.user_price
          }else {
            price = this.settlePrice
          }
          seatArr[parseInt(rowNo - this.minRow)][parseInt(columnNo - this.minCol)] = {
            type: seatStatus,
            SeatCode: seatId,
            RowNum: seatNo.substring(0,1),
            ColumnNum: columnNo,
            lovestatus: lovestatus,
            seatNo: seatNo,
            area_id: areaId,
            price: price,
            seat_type: seat_type,
          }
          gradeArr.push(seatNo.substring(0,1))
        }
        this.seatLists = seatArr
        this.gradeList = [...new Set(gradeArr)]
      },
      /*
       *  拖动
       * */
      dragChange(){
      },
      /*
       *  缩放
       * */
      scaleChange(obj){
        this.gradeScale = obj.detail.scale
      },
      touchstart(event){
        const firstTouch = event.touches[0];
        this.startX = firstTouch.clientX;
        this.startY = firstTouch.clientY;
        // if (event.touches.length === 2) {
        //   const xMove = event.touches[1].clientX - event.touches[0].clientX;
        //   const yMove = event.touches[1].clientY - event.touches[0].clientY;
        //   const distance = Math.sqrt(xMove * xMove + yMove * yMove);
        //   this.initialDistance = distance
        //   this.initialScale = this.gradeScale
        // }
      },
      touchmove(event){
        // if (event.touches.length === 2) {
        //   const xMove = event.touches[1].clientX - event.touches[0].clientX;
        //   const yMove = event.touches[1].clientY - event.touches[0].clientY;
        //   const distance = Math.sqrt(xMove * xMove + yMove * yMove);
        //   const distanceDiff = distance - this.initialDistance;
        //   let scale = this.initialScale + 0.005 * distanceDiff;
        //   this.gradeScale = scale
        // }
        const firstTouch = event.changedTouches[0];
        this.xValue = firstTouch.clientX - this.startX;
        this.yValue = firstTouch.clientY - this.startY;
      },
      touchend(){
        this.executionTime = 'all 0.2s'
        this.xValue = this.movableX = 0
        this.yValue = this.movableY = 0
        this.gradeScale > 1.5 ? this.gradeScale = 1.5 : ''
        this.gradeScale < 1 ? this.gradeScale = 1 : ''
            setTimeout(()=>{
          this.executionTime = 'none'
        },200)
      },
      /*
       *  选择座位
       * */
      selectSeat(){},
      /*
       *  获取座位图位置信息
       * */
      init(){
        const query = uni.createSelectorQuery().in(this)
        query
            .selectAll('.movable_view')
            .boundingClientRect(res => {
              if(res){
                this.gradeTop = res[0].top
              }
            })
            .exec()
      },
      /*
       * rpx转px
       *  */
      convertPX(obj){
        return (this.windowWidth / 750) * obj
      },
		}
	}
</script>

<style scoped lang="scss">
.film_info {
  margin-left: 30rpx;
  margin-top: 20rpx;

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
}
.seating_map{
  width: 100%;
  height: 100%;
  overflow: hidden;
  .movable_view{
    width: 100%;
    height: 100vh;
    position: relative;
    .row_list {
      display: flex;
      align-items: center;
      margin-top: 10rpx;
      margin-left: 45rpx;

      .columnNo_list {
        margin-left: 2rpx;
        display: flex;
        flex-direction: column;
        align-items: center;

        image {
          margin-left: 10rpx;
        }
      }
    }
  }
}
.grade_list{
  width: 35rpx;
  position: fixed;
  left: -50rpx;
  background-color: black;
  color: #FFFFFF;
  border-radius: 50rpx;
  padding-bottom: 10rpx;
  span{
    display: inline-block;
    width: 100%;
    margin-top: 10rpx;
    font-size: 28rpx;
    text-align: center;
  }
}

.render_nimation_anim{
  animation-name: circle-in-center;
  animation-duration: .3s;
}
.grade_show_anim{
  animation-name: gradeShow;
  animation-duration: .2s;
  animation-delay: 200ms;
  animation-fill-mode: forwards
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
@keyframes gradeShow {
  0% {
    left: -50rpx;
  }

  100% {
    left: 10rpx;
  }
}
</style>
