<template>
	<view>
    <view class="film_info">
      <view class="film_name">变形金刚：超能勇士崛起</view>
      <view class="film_time">2023-06-08 23:59:00 原版 3D</view>
    </view>
    <view class="hall_name render-nimation">
      vip 影厅
    </view>
    <movable-area class="seating_map">
      <movable-view direction="all" :inertia="true" :out-of-bounds="true" :scale="true" :scale-min="1" :scale-value="scaleValue" @change="dragChange" @scale="scaleChange">
          <view v-for="(item,index) in seatLists" :key="index" class="rowList render-nimation" >
            <view v-for="(item2,index2) in item" :key="index2" class="columnNoList" @click="selectSeat(item2)">
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
        settlePrice:''
			}
		},
		onLoad(option) {
      let settle_price = seatDate.data.show_info.settle_price,
          area_price = JSON.parse(seatDate.data.show_info.area_price) || ''
      this.areaPrice = area_price
      this.settlePrice = settle_price
      this.originalSeat = seatDate.data.seat_data.seats
      this.layoutStructure(this.originalSeat)
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
        for (let i = 0;i<this.originalSeat;i++){
          console.log(i)
          let {columnNo,rowNo,status,areaId,seatId,lovestatus,seatNo,seat_type} = this.originalSeat[i];
          let seatStatus = status === "N" ? 0: status === "LK" ? 2 : -1,
              price = 0;
          if(Array.isArray(this.areaPrice) && this.areaPrice.filter(item=>item.area == areaId).length>0){
            price = this.areaPrice.filter(item=>item.area == areaId)[0].price.user_price
          }else {
            price = this.settlePrice
          }
          console.log('rowNo - this.minRow',rowNo - this.minRow)
          console.log('columnNo - this.minCol',columnNo - this.minCol)
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
        }
        console.log('seatArr',seatArr)
      },
      /*
       *  拖动
       * */
      dragChange(){},
      /*
       *  缩放
       * */
      scaleChange(){},
      /*
       *  选择座位
       * */
      selectSeat(){},
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
  z-index: 99;
}
.render-nimation{
  animation-name: circle-in-center;
  animation-duration: .3s;
}

.seating_map{
  width: 100%;
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
