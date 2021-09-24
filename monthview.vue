<template>
  <div class="app-container">
    <el-row>
      <el-col :span="7" style="position: absolute;left: 35%;">
        <el-card style="width: 375px">
          <div class="calendar_head">
            <el-button @click="onMonthClick('last')">上个月</el-button>
            <span>{{ year_month }}</span>
            <el-button @click="onMonthClick('add')">下个月</el-button>
          </div>

          <div class="month_text">
            <div v-for="(item,idx) in monthList">
              <span>{{ item.name }}</span>
              <div v-for="(v,cdx) in item.child">
                <a
                    @click="clickDay(v,cdx)"
                    v-if="v.monthType == 1"
                    :class="{ list__Background:changeBackground == v.num,day__text:'day__text'}"
                >
                  <i v-if="v.type" :style="{'background-color':v.type == 1 ? '#FF3B30' : '#6d6d6d'}"
                     class="day_dot"></i>
                  {{ v.num }}
                </a>
                <a class="day__text" v-else style="color:#B3B0B0">
                  {{ v.num }}
                </a>
              </div>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import moment from "moment";

export default {
  name: "month_component",
  components: {},
  data() {
    return {
      changeBackground: -1,
      nextDay: 1,//用来填充下个月的日期
      openfill: false,//是否开启补足上下月日期功能
      year_month: "",//显示上方月份
      monthList: []//月视图列表
    };
  },
  created() {
    this.initMonthList();
    this.year_month = moment().startOf("month").format("YYYY-MM");
    this.formatDateByMonth(this.year_month);
  },
  methods: {

    initMonthList() {
      this.monthList = ["一", "二", "三", "四", "五", "六", "日"].map(item => {
        return {
          name: item,
          child: [
            { num: 0 },
            { num: 0 },
            { num: 0 },
            { num: 0 },
            { num: 0 },
            { num: 0 }
          ]
        };
      });
    },

    clickDay(v) {
      this.changeBackground = (this.changeBackground == -1 || this.changeBackground != v.num) ? v.num : -1;
      console.log("this.changeBackground", this.changeBackground);//如果该值大于0，获取该天的课程，-1说明取消选中，获取当月所有课程
      let startDay = moment(this.year_month).startOf("month"); //根据显示日期获取该月第一天
      let onDay = moment(startDay).add(v.num - 1, "days").format("YYYY-MM-DD"); //获取当前点击的日期
      if (this.changeBackground == -1) { //调用整月课程
        startDay = moment(startDay).format("YYYY-MM-DD")
        let endDay = moment(moment(this.year_month).endOf("month")).format("YYYY-MM-DD");
        console.log("当前月的第一天和最后一天", startDay, endDay);
      }
      else {
        //调用点击的日期课程 onDay
        console.log('当天日期',onDay);
      }
    },

    onMonthClick(type) {
      this.changeBackground = -1;
      let month = type == "add" ?
          moment(this.year_month).add(1, "month").format("YYYY-MM")
          :
          moment(this.year_month).subtract(1, "month").format("YYYY-MM");
      this.year_month = moment(month).startOf("month").format("YYYY-MM");
      this.formatDateByMonth(this.year_month);
    },

    formatDateByMonth(day) {
      let allDay = moment(day).daysInMonth();//查出该月有多少天
      let today_weekNum = moment(day).format("E");//月份第一天是周几

      this.indexDay = 1;//基准天数,填充num值

      //填充第一行:根据周几来判断
      let lastMonthAllDay = moment(day).subtract(1, "months").daysInMonth(); //上个月的总天数
      for (let i = today_weekNum - 2; i >= 0; i--) {
        if (this.openfill) {
          this.monthList[i].child[0]["num"] = lastMonthAllDay;
          this.monthList[i].child[0]["monthType"] = 2;
          lastMonthAllDay -= 1;
        }
        else {
          this.monthList[i].child[0]["monthType"] = 2;
          this.monthList[i].child[0]["num"] = " ";
        }
      }
      for (let i = today_weekNum - 1; i < 7; i++) {
        this.monthList[i].child[0]["num"] = this.indexDay;
        this.monthList[i].child[0]["monthType"] = 1;
        this.indexDay += 1;
      }

      //填充天数数组
      for (let j = 1; j < 6; j++) {
        for (let i = 0; i < 7; i++) {
          if (this.openfill) {
            this.nextDay = this.indexDay > allDay ? this.nextDay : 1;//用来填充下个月的日期
            if (this.indexDay <= allDay) {
              this.monthList[i].child[j]["monthType"] = 1;
              this.monthList[i].child[j]["num"] = this.indexDay;
              this.indexDay += 1;
            }
            else {
              this.monthList[i].child[j]["monthType"] = 2;
              this.monthList[i].child[j]["num"] = this.nextDay;
              this.nextDay += 1;
            }
          }
          else {
            this.monthList[i].child[j]["monthType"] = this.indexDay <= allDay ? 1 : 2;
            this.monthList[i].child[j]["num"] = this.indexDay <= allDay ? this.indexDay : " ";
            this.indexDay += 1;
          }
        }
      }

      //调用课程列表接口

      //接口数据传给月视图
      this.initDaysStatus([...new Set([1630598400000, 1631203200000, 1632499200000])]);
    },

    //根据传入的时间戳判断该天是否有课,并且小于今天的日期全部灰色,今天以后的全部红色
    initDaysStatus(arr) {
      this.monthList.map(item => {
        let day = moment(this.year_month).startOf("month"); //根据上方显示日期获取该月第一天
        for (let i = 0; i < item.child.length; i++) {
          let stamp = moment(day).add(item.child[i]["num"] - 1, "days").format("x"); //获取当前点击的日期
          if (arr.includes(+stamp)) {
            item.child[i].type = 1;//1灰色 2红色
            let todayStamp = moment().format("x");
            if (+stamp >= todayStamp) {
              item.child[i].type = 2;
            }
          }
          else{
            delete item.child[i].type //不包含就删除该字段,否则切换月份后还会显示小数点
          }
        }
      });
      console.log("this.monthList", this.monthList);
    }

  }
};
</script>

<style lang="scss" scoped>
.month_text {
  display: flex;
  flex-direction: row;
  align-content: space-around;
  justify-content: space-around;
  text-align: center;
  user-select: none;
  box-sizing: border-box;
}

.calendar_head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}


.list__Background {
  background-color: dodgerblue;
}

.day__text {
  position: relative;
  cursor: pointer;
  width: 40px;
  height: 40px;
  display: block;
  margin: 10px auto;
  line-height: 40px;
  font-size: 20px;
  border-radius: 50%;

  .day_dot {
    width: 5px;
    height: 5px;
    border-radius: 50%;
    position: absolute;
    left: 18px;
    top: 35px;
  }

}
</style>
