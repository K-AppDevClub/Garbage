<style lang='scss' scoped>
.page-title {
  text-align: center;
}
.body {
  margin-top: 50px;
}
#calendar-nav {
  text-align: center;
}

#calendar-nav span {
  display: inline-block;
  width: 200px;
}

#calendar-nav i:hover {
  cursor: pointer;
}

/* カレンダーのスタイル */
.table th, td{
  text-align: center;
}

#calendar th:first-child {
  background-color: #FEEEFF;
}
#calendar td:first-child {
  background-color: #FEEEFF;
}
#calendar th:nth-child(7) {
  background-color: #DFFFFF
}
#calendar td:nth-child(7) {
  background-color: #DFFFFF
}

#calendar td:hover {
  opacity: 0.6;
}
.calendar {
  text-align: center;
}
.child_calendar {
  display: inline-block;
}
.analyze_buttom{
  display: inline-block;
}
.clear_buttom{
  display: inline-block;
}
.research{
  text-align: center;
}
</style>

<template>
  <v-ons-page>
    <navbar></navbar>
    <v-ons-card modifier="material">
      <div style="font-size:0.8em; margin:0 0 5px;">あなたの地区では，もやせるごみが</div>
      <div style="margin: 0 0 30px">
        <span v-for="(select_region, $index) in select_regions" :key="select_region" tappable>
          <label class="left">
            <v-ons-radio
              :input-id="'radio-' + $index"
              :value="select_region"
              v-model="region"
            >
            </v-ons-radio>
          </label>
          <label :for="'radio-' + $index" class="center">
            {{ select_region }}
          </label>
        </span>
      </div>
      <div style="font-size:0.8em; margin:0 0 5px;">{{ getMonthName(calData.month) }}{{ calData.date }}日 {{ dayOfWeekStr }}曜，あなたの地区は</div>
      <div v-if="region=='月曜と木曜'"><span style="color: red">{{ this.g_type1[this.calData.week] }}</span> です！</div>
      <div v-if="region=='火曜と金曜'"><span style="color: red">{{ this.g_type2[this.calData.week] }}</span> です！</div>
    </v-ons-card>
    <v-ons-card modifier="material">
      <div class="research">
        <v-ons-input style="width:70%;margin:auto" name="code_ireru" v-model="gomidata" placeholder="分別したいゴミ"></v-ons-input>
        <span><v-ons-button style="background-color:rgb(156, 20, 20)" @click="textClear()">クリア</v-ons-button></span>
      </div>
      <div>
        <p>{{ calData.day }}</p>
        <p>検索結果：{{this.gomidata}}</p>
        <p></p>
        <p v-if="separate.type != 'データがありません'">{{separate.type}}</p>
        <p v-if="separate.cost != 'データがありません'">¥{{separate.cost}}</p>
        <p v-else>”{{this.gomidata}}”は登録されていません</p>
      </div>
    </v-ons-card>
    <!-- <v-ons-card modifier="material">
      <div id="calendar-nav">
        <i class="zmdi zmdi-arrow-left" tappable @click="moveLastMonth"></i>
        <span>{{calData.year}} - {{getMonthName(calData.month)}}</span>
        <i class="zmdi zmdi-arrow-right" tappable @click="moveNextMonth"></i>
      </div>
      <br>

      <div class="calendar">
        <div class="child_calendar">
        <table id="calendar" class="table table-bordered">
          <thead>
            <tr>
              <th width="50" v-for="week in weeks">{{week}}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="week in calendar">
              <td v-for="day in week" tappable @click="$router.push({name:'detail', params:{year: calData.year, month:getMonthName(calData.month), day: day.day}})">{{day.day}}</td>
            </tr>
          </tbody>
        </table>
        </div>
      </div>
    </v-ons-card> -->
  </v-ons-page>
</template>

<script>
import LoadingIndicator from '../../components/loading-indicator/LoadingIndicator';
import Navbar from '../../components/navbar/Navbar';
import Config from '../../config/Config';

export default {
  name: 'posts-page',
  components: {
    LoadingIndicator,
    Navbar,
  },
  params: {
    year: {
      default: null
    },
    month: {
      default: null
    },
    day: {
      default: null
    }
  },
  methods: {
    getMonthName: function(month) {
      var monthName = ['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月'];
      return monthName[month - 1];
    },
    moveLastMonth: function() {
      if (this.calData.month == 1) {
          this.calData.year--;
          this.calData.month = 12;
      }
      else {
          this.calData.month--;
      }
    },
    moveNextMonth: function() {
      if (this.calData.month == 12) {
          this.calData.year++;
          this.calData.month = 1;
      }
      else {
          this.calData.month++;
      }
    },
    textClear(){
      this.gomidata = ''
    },
  },

  data() {
    return {
      weeks: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
      calData: {year: 0, month: 0, date: 0, week: 0},
      g_type1: ['なし', 'もやせるごみ', '古紙／衣類', '金属 or 缶・瓶／PET or もやせないごみ', 'もやせるごみ', 'プラスチック容器類', 'なし'],
      g_type2: ['なし', '古紙／衣類', '金属 or 缶・瓶／PET or もやせないごみ', 'もやせるごみ', 'プラスチック容器類', 'もやせるごみ', 'なし'],
      type: '',
      select_regions: ['月曜と木曜', '火曜と金曜'],
      region: '月曜と木曜',
      dayOfWeekStr: '',
      gomidata: '',
      result_type: '',
      result_cost: 0,
      sentiment_score: 0,
      results: [],
      result: [],
    };
  },
  created() {
    var date = new Date();
    this.calData.year = date.getFullYear();
    this.calData.month = date.getMonth() + 1;
    this.calData.date = date.getDate();
    this.calData.week = date.getDay();
    this.dayOfWeekStr = [ "日", "月", "火", "水", "木", "金", "土" ][this.calData.week];
  },
  mounted() {
    this.results = require("../../json/garbage.json");
    console.log(this.results);
    this.type = this.g_type1[this.calData.week];
  },
  computed: {
    separate(){
      for (var g_data in this.results){
        if (this.results[g_data].name == this.gomidata){
          console.log(this.results[g_data].name);
          console.log(this.results[g_data].type);
          console.log(this.results[g_data].cost);
          this.result_type = this.results[g_data].type
          this.result_cost = this.results[g_data].cost
          break;
        }
        else{
          this.result_type = 'データがありません'
          this.result_cost = 'データがありません'
        }
      }
      this.result = {type: this.result_type, cost: this.result_cost}
      return this.result
      // return this.result_type;
    },
    calendar () {
      // 1日の曜日
      var firstDay = new Date(this.calData.year, this.calData.month - 1, 1).getDay();
      // 晦日の日にち
      var lastDate = new Date(this.calData.year, this.calData.month, 0).getDate();
      // 日にちのカウント
      var dayIdx = 1;

      var calendar = [];
      for (var w = 0; w < 6; w++) {
        var week = [];

        // 空白行をなくすため
        if (lastDate < dayIdx) {break;}

        for (var d = 0; d < 7; d++) {
          if (w == 0 && d < firstDay) {
            week[d] = {day: ''};
          } else if (w == 6 && lastDate < dayIdx) {
            week[d] = {day: ''};
            dayIdx++;
          } else if (lastDate < dayIdx) {
            week[d] = {day: ''};
            dayIdx++;
          } else {
            week[d] = {day: dayIdx};
            dayIdx++;
          }
        }
        calendar.push(week);
      }
      return calendar;
    }
  }
};
</script>
