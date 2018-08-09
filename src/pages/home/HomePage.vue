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
  // width: 500px;
  // margin: 0 auto;
}
.child_calendar {
  display: inline-block;
}
#buttom_area{
  text-align: center;
  margin-top: 10px;
}
.analyze_buttom{
  display: inline-block;
}
.clear_buttom{
  display: inline-block;
}
</style>

<template>
  <v-ons-page>
    <navbar></navbar>
    <v-ons-card modifier="material">
      <div id="calendar-nav">
        <i class="zmdi zmdi-comment-outline" tappable @click="moveLastMonth"></i>
        <span>{{calData.year}} - {{getMonthName(calData.month)}}</span>
        <i class="zmdi zmdi-comment-outline" tappable @click="moveNextMonth"></i>
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
    </v-ons-card>
    <v-ons-card>
      <v-ons-list-item>
        <textarea style="width:100%;height:90px" name="code_ireru" v-model='postdata.document.content' placeholder="文章を入力"></textarea>
      </v-ons-list-item>
      <div id = "buttom_area">
        <div class="analyze_buttom">
          <v-ons-button  @click="go()">感情分析</v-ons-button>
        </div>
        <div class="clear_buttom">
          <v-ons-button style="background-color:rgb(156, 20, 20)" @cick="textClear()">クリア</v-ons-button>
        </div>
      <v-ons-list-item>
        <textarea style="width:100%;height:90px" name="code_ireru" v-model="gomidata" placeholder="文章を入力"></textarea>
      </v-ons-list-item>
      <div class="g_buttom">
        <v-ons-button  @click="separate()">分別</v-ons-button>
      </div>
      </div>
      <div>
        <p>点数！</p>
        <p>{{sentiment_score}}</p>
        <p>{{this.gomidata}}</p>
        <p>{{result_type}}</p>
        <p>¥{{result_value}}</p>

      
      </div>
      
  </v-ons-card>
  <v-ons-button @click="page()">ページ遷移 </v-ons-button>
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
      var monthName = ['January','February','March','April','May','June','July','August','September','October','November','December'];
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
    go(){
      this.axios.post('https://language.googleapis.com/v1/documents:analyzeSentiment?key=AIzaSyDzdTDDNFQ9STkA1bfGEcUnlxgpvFLrEL0',this.postdata)
      .then((res) => {
        console.log(res);
        this.sentiment_score = res.data.documentSentiment.score  * 100
      })
      .catch(error => {
          this.sending = false
          throw error
      })
    },
    textClear(){
      this.postdata.document.content = "" 
      this.postdata.document.garbage = "" 
      this.sentiment_score = 0
    },
    page(){
      this.$router.push({ name: 'sentiment'}); 
    },
    separate(){
      
      for (var g_data in this.results){
        if (this.results[g_data].name == this.gomidata){
          console.log(this.results[g_data].name);
          console.log(this.results[g_data].type);
          console.log(this.results[g_data].value);
          this.result_type = this.results[g_data].type
          this.result_value = this.results[g_data].value
        }
      }
    }
  },

  data() {
    return {
      weeks: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
      calData: {year: 0, month: 0},
      gomidata: '',
      result_type: '',
      result_value: 0,
      postdata: {
        document: {  
          type:"PLAIN_TEXT",
        　content:"あなたのことが大好きだ"
        },
        encodingType: 'UTF8'
      },
      res_data:[],
      sentiment_score: 0,
      results: [],
    };
  },
  created() {
    var date = new Date();
    this.calData.year = date.getFullYear();
    this.calData.month = date.getMonth() + 1;
  },
  mounted() {
    this.results = require("../../json/garbage.json");
    console.log(this.results);
  },
  computed: {
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
