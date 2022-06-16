<template>
  <view>
    <view class="mi_calendar">
      <view class="top-bar">
        <i class="top-change-month" @click="changeMonth('prev')" />
        <view class="top-bar-ym">{{ y }}.{{ m }}</view>
        <i class="top-change-month next-month" @click="changeMonth('next')" />
      </view>
      <view class="week">
        <view class="week-day" v-for="(item, index) in weekDay" :key="index">{{
          item
        }}</view>
      </view>
      <view
        :class="{ hide: !monthOpen }"
        class="content"
        :style="{ height: height }"
      >
        <view :style="{ top: positionTop + 'rpx' }" class="days">
          <view
            :class="['item', { nolm: !item.lm }]"
            v-for="(item, index) in dates"
            :key="index"
          >
            <view
              class="day"
              :key="`${item.year}-${item.month + 1}-${item.date}`"
              @click="selectOne(item, $event)"
              :class="{
                choose: choose == `${item.year}-${item.month}-${item.date}`,
              }"
            >
              {{ item.date }}
            </view>
            <view
              class="flag_icon late"
              v-if="filterType('LEAVE', item.year, item.month, item.date)"
            ></view>
            <view
              class="flag_icon truancy"
              v-if="filterType('SUSPENSION', item.year, item.month, item.date)"
            ></view>
            <view
              class="flag_icon normal"
              v-if="filterType('NORMAL', item.year, item.month, item.date)"
            ></view>
            <view
              class="today-text"
              v-if="isToday(item.year, item.month, item.date)"
              >今</view
            >
          </view>
        </view>
      </view>
    </view>
    <view class="type-check">
      <view class="label" v-for="(item, index) in statusText" :key="index"
        ><i></i><text>{{ item }}</text></view
      >
    </view>
  </view>
</template>

<script>
export default {
  name: "mi-calendar",
  props: {
    // 第一列星期几
    weekstart: {
      type: Number,
      default: 1,
    },
    // 请假日期
    leaveDateList: {
      type: Array,
      default: () => [],
    },
    // 停课日期
    suspensionDateList: {
      type: Array,
      default: () => [],
    },
    // 正常考勤日期
    normalDateList: {
      type: Array,
      default: () => [],
    },
    // 是否展开 暂未实现
    open: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      statusText: ["出勤", "请假", "停课"],
      text: {
        year: "年",
        month: "月",
        week: ["一", "二", "三", "四", "五", "六", "日"],
        today: "今",
      },
      y: new Date().getFullYear(), // 年
      // m: new Date().getMonth() >= 10 ? new Date().getMonth() + 1 : `0${new Date().getMonth() + 1}`, // 月
      m: new Date().getMonth() + 1,
      dates: [], // 当前月日期集合
      positionTop: 0,
      monthOpen: true,
      choose: "",
    };
  },
  created() {
    this.dates = this.monthDay(this.y, this.m);
    // console.log(this.y, this.m);
    // !this.open && this.trgWeek();
  },
  mounted() {
    const date = new Date();
    const y = date.getFullYear();
    const m = date.getMonth() + 1;
    const d = date.getDate();
    // const M = m >= 10 ? m + 1 : `0${m + 1}`;
    // const D = d >= 10 ? d : `0${d}`;
    this.choose = `${y}-${m}-${d}`;
    console.log("this.dates", this.choose);
  },
  computed: {
    // 顶部星期栏目
    weekDay() {
      return this.text.week
        .slice(this.weekstart - 1)
        .concat(this.text.week.slice(0, this.weekstart - 1));
    },
    height() {
      return (this.dates.length / 7) * 80 + "rpx";
    },
  },
  methods: {
    // 获取当前月份天数
    monthDay(y, m) {
      let firstDayOfMonth = new Date(y, m - 1, 1).getDay(); // 当月第一天星期几
      let lastDateOfMonth = new Date(y, m, 0).getDate(); // 当月最后一天
      let lastDayOfLastMonth = new Date(y, m - 1, 0).getDate(); // 上一月的最后一天
      let dates = []; // 所有渲染日历
      let weekstart = this.weekstart == 7 ? 0 : this.weekstart; // 方便进行日期计算，默认星期从0开始
      let startDay = (() => {
        // 周初有几天是上个月的
        if (firstDayOfMonth == weekstart) {
          return 0;
        } else if (firstDayOfMonth > weekstart) {
          return firstDayOfMonth - weekstart;
        } else {
          return 7 - weekstart + firstDayOfMonth;
        }
      })();
      let endDay = 7 - ((startDay + lastDateOfMonth) % 7); // 结束还有几天是下个月的
      for (let i = 1; i <= startDay; i++) {
        const date = lastDayOfLastMonth - startDay + i;
        dates.push({
          date,
          day: weekstart + i - 1 || 7,
          month: m - 1 >= 0 ? m - 1 : 12,
          year: m - 1 >= 0 ? y : y - 1,
        });
      }
      for (let j = 1; j <= lastDateOfMonth; j++) {
        dates.push({
          date: j,
          day: (j % 7) + firstDayOfMonth - 1 || 7,
          month: m,
          year: y,
          lm: true,
        });
      }
      for (let k = 1; k <= endDay; k++) {
        dates.push({
          date: k,
          day: (lastDateOfMonth + startDay + weekstart + k - 1) % 7 || 7,
          month: m + 1 <= 11 ? m + 1 : 0,
          year: m + 1 <= 11 ? y : y + 1,
        });
      }
      return dates;
    },
    getFormatString(m) {
      return m >= 10 ? m : `0${m}`;
    },
    filterType(type, y, m, d) {
      const opt = {
        LEAVE: "leaveDateList", // 请假
        SUSPENSION: "suspensionDateList", // 停课
        NORMAL: "normalDateList", // 出勤
      };
      const list = opt[type];
      const dataList = this[list];
      let flag = false;
      for (let i = 0; i < dataList.length; i++) {
        const M = this.getFormatString(m);
        const D = this.getFormatString(d);
        const dy = `${y}-${M}-${D}`;
        if (dataList[i] == dy) {
          flag = true;
          break;
        }
      }
      return flag;
    },
    isToday(y, m, d) {
      let date = new Date();
      return (
        y == date.getFullYear() && m == date.getMonth() && d == date.getDate()
      );
    },
    // 切换成周模式
    trgWeek() {
      this.monthOpen = !this.monthOpen;
      if (this.monthOpen) {
        this.positionTop = 0;
      } else {
        let index = -1;
        this.dates.forEach((i, x) => {
          this.isToday(i.year, i.month, i.date) && (index = x);
        });
        this.positionTop = -((Math.ceil((index + 1) / 7) || 1) - 1) * 80;
      }
    },
    // 点击回调
    selectOne({ year, month, date }, event) {
      let date_time = `${year}-${this.getFormatString(
        month
      )}-${this.getFormatString(date)}`;
      if (month != this.m) {
        console.log("不在可选范围内");
        return false;
      }
      this.choose = `${year}-${month}-${date}`;
      this.$emit("change", date_time);
    },
    // 月份切换
    changeMonth(action) {
      if (action === "next") {
        if (this.m == 12) {
          this.m = 1;
          this.y++;
        } else {
          this.m++;
        }
      } else {
        if (this.m == 1) {
          this.m = 12;
          this.y--;
        } else {
          this.m--;
        }
      }
      this.dates = this.monthDay(this.y, this.m);
    },
  },
};
</script>

<style lang="scss" scoped>
$working: #80b2ff;
$leave: #ffcd3c;
$suspend: #f46864;
.mi_calendar {
  color: #1a1a1a;
  font-size: 28rpx;
  text-align: center;
  padding-bottom: 10rpx;
  margin: 36rpx 24rpx;
  background-color: #fff;
  border-radius: 36rpx;
  box-shadow: 0 3rpx 32rpx 0rpx rgba(0, 0, 0, 0.1);
  overflow: hidden;

  .top-bar {
    display: flex;
    height: 80rpx;
    align-items: center;
    justify-content: center;
    .top-bar-ym {
      font-size: 32rpx;
      height: 80rpx;
      line-height: 80rpx;
    }
    .top-change-month {
      width: 23rpx;
      height: 13rpx;
      background: url("../static/arrowDown.png");
      background-size: 23rpx 13rpx;
      transform: rotate(90deg);
      margin: 0 30rpx;
    }
    .next-month {
      transform: rotate(-90deg);
    }
  }
  .week {
    display: flex;
    align-items: center;
    height: 80rpx;
    line-height: 80rpx;
    view {
      flex: 1;
    }
    .week-day {
      font-size: 26rpx;
      color: #1a1a1a;
    }
  }
  .content {
    position: relative;
    overflow: hidden;
    transition: height 0.4s ease;
    .days {
      transition: top 0.3s;
      display: flex;
      align-items: center;
      flex-wrap: wrap;
      position: relative;
      .item {
        position: relative;
        display: block;
        height: 80rpx;
        line-height: 80rpx;
        width: calc(100% / 7);

        .day {
          font-style: normal;
          display: inline-block;
          vertical-align: middle;
          width: 60rpx;
          height: 60rpx;
          line-height: 60rpx;
          overflow: hidden;
          border-radius: 50%;

          &.choose {
            background: linear-gradient(180deg, #fade7b, #f9c34e);
            color: #333;
          }
        }
        &.nolm {
          color: #999;
          opacity: 0.3;
        }
        .flag_icon {
          bottom: 0;
          left: 50%;
          font-style: normal;
          width: 12rpx;
          height: 12rpx;
          border-radius: 6rpx;
          position: absolute;
          margin-left: -6rpx;
          pointer-events: none;
        }
        .late {
          background: #ffcd3c;
        }
        .truancy {
          background: #f46864;
        }
        .normal {
          background: #80b2ff;
        }
        .today-text {
          position: absolute;
          font-size: 20rpx;
          font-weight: normal;
          width: 20rpx;
          height: 20rpx;
          line-height: 20rpx;
          right: 0;
          top: 10rpx;
          color: #fff;
        }
      }
    }
  }

  .hide {
    height: 80rpx !important;
  }

  .weektoggel {
    width: 80rpx;
    height: 40rpx;
    margin: 10rpx auto 0;

    &.down {
      transform: rotate(180deg);
    }
  }
}
.type-check {
  overflow: hidden;
  margin: 0 24rpx;
  text-align: right;
  .label {
    display: inline-block;
    margin-right: 24rpx;
    i {
      display: inline-block;
      width: 10rpx;
      height: 10rpx;
      border-radius: 50%;
      vertical-align: middle;
      margin-right: 10rpx;
    }
    text {
      color: #1a1a1a;
      font-size: 26rpx;
    }
    &:nth-child(1) {
      i {
        background-color: $working;
      }
    }
    &:nth-child(2) {
      i {
        background-color: $leave;
      }
    }
    &:nth-child(3) {
      i {
        background-color: $suspend;
      }
    }
  }
}
</style>
