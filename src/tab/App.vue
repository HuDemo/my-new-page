<template>
  <div>
    <!-- 日历 -->
    <div class="main">
      <div class="choose_year">
        <div class="icon" @click="chooseYears(-1)"><icon name="angle-double-left"></icon></div>
        <div class="icon" @click="chooseMonth(-1)"><icon name="angle-left"></icon></div>
        <div class="date">{{year}}.{{month.toString().padStart(2, '0')}}</div>
        <div class="icon" @click="chooseMonth(1)"><icon name="angle-right"></icon></div>
        <div class="icon" @click="chooseYears(1)"><icon name="angle-double-right"></icon></div>
      </div>
      <div class="days_area">
        <div class="day week" v-for="week in weeks" :key="week" :class="(week=='六'||week=='日')?'free-week':''">{{week}}</div>
        <div class="day" @click="chooseThisDay(day.gregorian)" v-for="(day, index) in days" :key="index" :class="day.gregorian === today ? 'choose_day' : ''">
          <p>{{day.gregorian}}</p>
          <span>{{day.lunar}}</span>
        </div>
      </div>
    </div>
    <!-- 书签 -->
    <div class="bookmark-area">
      <div class="bookmark-tab">
        <div class="tab-name"><icon name="folder-open" color="#5B97F6"></icon>工作区</div>
        <div class="tab-list">
          <p v-for="(item, index) in workMarks" :key="index">
            <icon name="keyboard" color="#C5C5C5"></icon><a :href="item.link">{{item.name}}</a>
          </p>
        </div>
      </div>
      <div class="bookmark-tab">
        <div class="tab-name"><icon name="coffee" color="#5B97F6"></icon>休闲区</div>
        <div class="tab-list">
          <p v-for="(item, index) in joyArea" :key="index">
            <icon name="headphones" color="#C5C5C5"></icon><a :href="item.link">{{item.name}}</a>
          </p>
        </div>
      </div>
    </div>
    <!-- todo list -->
    <div class="todo-area">
      <div class="todo-title">TODO LIST</div>
      <div class="had-area" v-if="todoList.length">
        <p v-for="(el, index) in todoList" :key="index">
          <icon name="lightbulb" color="#fff" class="todo-icon"></icon>
          <span class="todo-txt">{{el}}</span>
          <icon name="minus" color="#f00" class="todo-delete" @click="deleteIt(index)"></icon>
        </p>
      </div>
      <div class="none-tip" v-else>
        <p>暂无待做事项</p>
      </div>
      <div class="add-area">
        <input type="text" class="add-input" placeholder="记住自己没做出来单项删除，谨慎add" v-model="todo"/>
        <div class="add-btn" @click="storeTodo(todo)">add</div>
        <div class="add-btn" @click="clearTodo">clear</div>
      </div>
    </div>
    <!-- <div v-if="loading" class="centered">
      <p>Loading...</p>
    </div>
    <div v-else>
      <p class="joke">{{ joke }}</p>
      <div class="button-container">
        <button @click="likeJoke" :disabled="likeButtonDisabled" class="btn"><icon name="thumbs-up"></icon></button>
        <button @click="logJokes" class="btn"><icon name="list"></icon></button>
        <button @click="clearStorage" class="btn"><icon name="trash"></icon></button>
      </div>
    </div> -->
  </div>
</template>

<script>
// import axios from 'axios';
export default {
  data () {
    return {
      loading: true,
      joke: "",
      likeButtonDisabled: false,
      year: 0,
      month: 0,
      today: 0,
      days: [],
      weeks: ['一', '二', '三', '四', '五', '六', '日'],
      workMarks: [
        {link: 'https://shimo.im/space/tXpG8vYhjXdjvgCJ', name: '石墨文档'},
        {link: 'https://lanhuapp.com/web/#/item?fid=all&commonly=join', name: '蓝湖'},
        {link: 'https://free.modao.cc/dashboard/oBC1094409F1552976557667', name: '墨刀'},
        {link: 'https://mp.weixin.qq.com/wxopen/home?lang=zh_CN&token=2012615115', name: '微信公众平台'},
        {link: 'https://developers.weixin.qq.com/miniprogram/dev/framework/', name: '微信开发文档'},
        {link: 'https://admin.xiaoe-tech.com/login_page?xeuti=ituex#/acount', name: '小鹅通'},
        {link: 'https://www.iviewui.com/components/icon', name: 'iview'},
        {link: 'https://tinypng.com/', name: 'tiny png'}
      ],
      joyArea: [
        {link: 'https://www.ximalaya.com/search/%e9%83%ad%e5%be%b7%e7%ba%b2%e7%9b%b8%e5%a3%b0/', name: '郭德纲相声'}
      ],
      todoList: [],
      todo: ''
    }
  },
  methods: {
    likeJoke(){
      chrome.storage.local.get("jokes", (res) => {
        if(!res.jokes) res.jokes = [];
        res.jokes.push(this.joke)
        chrome.storage.local.set(res);
        this.likeButtonDisabled = true;
      });
    },
    logJokes(){
      chrome.storage.local.get("jokes", (res) => {
        if(res.jokes) res.jokes.map(joke => console.log(joke))
      });
    },
    clearStorage(){
      chrome.storage.local.clear();
    },
    getDays () { // 获取当前月份所有公历日期及其农历日期
      this.days = []
      const time = new Date()
      time.setFullYear(this.year, this.month, 0)
      const time1 = new Date()
      time1.setFullYear(this.year, this.month - 1, 1)
      for (let i = 1; i < time1.getDay(); i++) {
        this.days.push({gregorian: '', lunar: ''})
      }
      for (let i = 1; i <= time.getDate(); i++) {
        this.days.push({gregorian: i, lunar: this.getLunarDay(this.year, this.month, i)})
      }
    },
    chooseYears (state) { // 改变年份
      this.year += state
      this.today = 1
      this.getDays()
    },
    chooseMonth (state) { // 改变月份
      this.month += state
      this.today = 1
      if (this.month < 1) {
        this.month = 12
        this.chooseYears(-1)
      } else if (this.month > 12) {
        this.month = 1
        this.chooseYears(1)
      } else {
        this.getDays()
      }
    },
    chooseThisDay (day) { // 选择某天，主要是考虑可以当时间选择器用
      if (day > 0) {
        this.today = day
      }
    },
    getLunarDay (solarYear, solarMonth, solarDay) { // 拷贝别人又自己调整过的获取农历日期的代码，想要原来的代码估计百度一下就有了
      const madd = [0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334]
      // const tgString = '甲乙丙丁戊己庚辛壬癸'
      // const dzString = '子丑寅卯辰巳午未申酉戌亥'
      const numString = '一二三四五六七八九十'
      const monString = '正二三四五六七八九十冬腊'
      const CalendarData = [0xA4B, 0x5164B, 0x6A5, 0x6D4, 0x415B5, 0x2B6, 0x957, 0x2092F, 0x497, 0x60C96, 0xD4A, 0xEA5, 0x50DA9, 0x5AD, 0x2B6, 0x3126E, 0x92E, 0x7192D, 0xC95, 0xD4A, 0x61B4A, 0xB55, 0x56A, 0x4155B, 0x25D, 0x92D, 0x2192B, 0xA95, 0x71695, 0x6CA, 0xB55, 0x50AB5, 0x4DA, 0xA5B, 0x30A57, 0x52B, 0x8152A, 0xE95, 0x6AA, 0x615AA, 0xAB5, 0x4B6, 0x414AE, 0xA57, 0x526, 0x31D26, 0xD95, 0x70B55, 0x56A, 0x96D, 0x5095D, 0x4AD, 0xA4D, 0x41A4D, 0xD25, 0x81AA5, 0xB54, 0xB6A, 0x612DA, 0x95B, 0x49B, 0x41497, 0xA4B, 0xA164B, 0x6A5, 0x6D4, 0x615B4, 0xAB6, 0x957, 0x5092F, 0x497, 0x64B, 0x30D4A, 0xEA5, 0x80D65, 0x5AC, 0xAB6, 0x5126D, 0x92E, 0xC96, 0x41A95, 0xD4A, 0xDA5, 0x20B55, 0x56A, 0x7155B, 0x25D, 0x92D, 0x5192B, 0xA95, 0xB4A, 0x416AA, 0xAD5, 0x90AB5, 0x4BA, 0xA5B, 0x60A57, 0x52B, 0xA93, 0x40E95]
      if (!(solarYear < 1921 || solarYear > 2020)) {
        solarMonth = (parseInt(solarMonth) > 0) ? (solarMonth - 1) : 11
        const timeArr = [solarYear, solarMonth, solarDay]
        let TheDate = (timeArr.length !== 3) ? new Date() : new Date(timeArr[0], timeArr[1], timeArr[2])
        let total, m, n, k
        let isEnd = false
        let theDateYear = TheDate.getFullYear()
        total = (theDateYear - 1921) * 365 + Math.floor((theDateYear - 1921) / 4) + madd[TheDate.getMonth()] + TheDate.getDate() - 38
        if (theDateYear % 4 === 0 && TheDate.getMonth() > 1) {
          total++
        }
        for (m = 0; ; m++) {
          k = (CalendarData[m] < 0xfff) ? 11 : 12
          for (n = k; n >= 0; n--) {
            if (total <= this.getBit(CalendarData[m], n)) {
              isEnd = true
              break
            }
            total = total - this.getBit(CalendarData[m], n)
          }
          if (isEnd) {
            break
          }
        }
        let cMonth, cDay // cYear,
        // cYear = 1921 + m
        cMonth = k - n + 1
        cDay = total
        if (k === 12) {
          if (cMonth === Math.floor(CalendarData[m] / 0x10000) + 1) {
            cMonth = 1 - cMonth
          }
          if (cMonth > Math.floor(CalendarData[m] / 0x10000) + 1) {
            cMonth--
          }
        }
        // let run = ''
        let cDayStr = numString.charAt(cDay - 1)
        /* if (cMonth < 1) {
          run = '(闰)'
        } */
        if (cDay % 10 !== 0 || cDay === 10) {
          cDayStr = numString.charAt((cDay - 1) % 10)
        }
        return cDay === 1 ? monString.charAt(cMonth - 1) + '月' : (cDay < 11 ? '初' : (cDay < 20 ? '十' : (cDay < 30 ? '廿' : '三十'))) + cDayStr // tgString.charAt((cYear - 4) % 10) + dzString.charAt((cYear - 4) % 12) + '年 ' + run + monString.charAt(cMonth - 1) + '月' +
      }
    },
    getBit (m, n) { // 也是拷贝的，不是很明白这段代码干嘛的，不过很明显是处理二进制数据的
      return 29 + ((m >> n) & 1)
    },
    storeTodo (e) {
      console.log(204, e)
      chrome.storage.local.get("todos", (res) => {
        if(!res.todos) res.todos = [];
        res.todos.push(e)
        chrome.storage.local.set(res);
      })
      this.logTodos()
      this.todo = ''
    },
    logTodos () {
      chrome.storage.local.get("todos", (res) => {
        if(res.todos) res.todos.map(todos => console.log(todos))
        this.todoList = res.todos
      })
    },
    clearTodo () {
      chrome.storage.local.remove("todos", function () {
        // todo
      })
    },
    deleteIt (m) {
      console.log(234, m)
    }
  },
  mounted() {
    // axios.get(
    //   "https://icanhazdadjoke.com/",
    //   { 'headers': { 'Accept': 'application/json' } }
    // )
    //   .then(res => {
    //     this.joke = res.data.joke
    //     this.loading = false;
    //   });
    const now = new Date()
    this.year = now.getFullYear()
    this.month = now.getMonth() + 1
    this.today = now.getDate()
    let m = now.getDay() - 1
    this.getDays()
    chrome.storage.local.get("todos", (res) => {
      if (res.todos) {
        this.todoList = res.todos
      }
    })
    chrome.storage.onChanged.addListener(function(changes, areaName){
      console.log('Value in '+areaName+' has been changed:');
      console.log(changes);
      // this.todoList = changes.todos.newValue
      // this.todoList.push(changes.todos.newValue[changes.todos.oldValue.length])
      console.log(this.todoList)
    });
  }
}
</script>

<style>
/* 清除默认样式 */
html, body, div, ul, li, h1, h2, h3, h4, h5, h6, p, dl, dt, dd, ol, form, input, textarea, th, td, select {margin: 0;padding: 0;}
*{box-sizing: border-box;}
html, body {min-height: 100%;}
h1, h2, h3, h4, h5, h6{font-weight:normal;}
ul,ol {list-style: none;}
img {border: none;vertical-align: middle;}
a {text-decoration: none;color: #232323;}
table {border-collapse: collapse;table-layout: fixed;}
input, textarea {outline: none;border: none;}
textarea {resize: none;overflow: auto;}
body {
  height: 98vh;
  text-align: center;
  color: #353638;
  font-size: 22px;
  line-height: 30px;
  font-family: Merriweather,Georgia,serif;
  background: url("https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/12/1544189726troll-dad.png") no-repeat 1% 99%;
  background-size: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}
/* 书签区 */
.bookmark-area{
  width: 30vw;
  min-width: 350px;
  height: 80vh;
  text-align: left;
}
.bookmark-tab{
  margin-bottom: 30px;
  font-size: 18px;
  border: 5px solid #eee;
  border-radius: 10px;
  padding: 15px;
}
.bookmark-tab:hover{
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2), 0 6px 20px 0 rgba(0,0,0,0.19);
}
.tab-name{
  font-size: 22px;
  line-height: 50px;
}
.tab-list{
  line-height: 40px;
}
.tab-list p:hover{
  background: #B7D5FD;
}
.tab-list a{
  display: inline-block;
  min-width: 250px;
}
.fa-icon{
  margin-right: 10px;
}
/* 日历 */
.main{
  /* width: 350px; */
  width: 30vw;
  max-width: 350px;
  height: auto;
  position: fixed;
  left: 0;
  top: 0;
  border: 2px solid #5B97F6;
  margin: 2vw;
}
.choose_year{
  display: flex;
}
.choose_year .icon{
  width: 10%;
  height: 5vh;
  line-height: 5vh;
  text-align: center;
}
.choose_year .date{
  width: 60%;
  text-align: center;
  height: 5vh;
  line-height: 5vh;
  font-size: 1.2rem;
}
.days_area{
  display: flex;
  flex-wrap: wrap;
}
.days_area .day{
  width: 14.28%;
  text-align: center;
  padding: 5px 0;
  margin-top: 5px;
}
.days_area .day p{
  margin: 0;
  font-size: 22px;
}
.days_area .day span{
  font-size: 16px;
  line-height: 16px;
}
.days_area .week{
  background-color: dodgerblue;
  color: #fff;
  font-weight: bold;
  height: 4vh;
  line-height: 3vh;
  margin: 0;
}
.days_area .choose_day{
  background-color: dodgerblue;
  color: #fff;
  font-weight: bold;
}
.days_area .free-week{
  background: #F74C6C;
}
/* todo-list */
.todo-area{
  background: #B7D5FD;
  width: 30vw;
  max-width: 350px;
  height: auto;
  min-height: 50vh;
  position: fixed;
  right: 0;
  top: 0;
  border-radius: 30px;
  padding: 20px 30px;
  text-align: left;
  margin: 20px 20px 0 0;
}
.todo-title{
  line-height: 50px;
  font-size: 24px;
  font-weight: bold;
  color: #fff;
}
.none-tip{
  font-size: 18px;
  color: #999;
}
.todo-icon{
  vertical-align: middle;
}
.todo-txt{
  font-size: 18px;
  vertical-align: middle;
}
.todo-delete{
  cursor: pointer;
  vertical-align: middle;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  padding: 3px;
}
.todo-delete:hover{
  color: #fff;
  background: #f00;
}
.add-area{
  margin-top: 10px;
}
.add-input{
  width: 100%;
  height: 30px;
  display: inline-block;
  padding: 5px;
  border-radius: 8px;
  border: none;
}
.add-btn{
  width: 100px;
  height: 30px;
  font-size: 18px;
  background: #fff;
  text-align: center;
  color: #555;
  border-radius: 5px;
  margin: 10px;
  display: inline-block;
  cursor: pointer;
}
.add-btn:hover{
  animation: hovershake 1s 1;
}
@keyframes hovershake {
  0% {
    transform: rotate(0deg);
  }
  20% {
    transform: rotate(5deg);
  }
  40% {
    transform: rotate(-5deg);
  }
  60% {
    transform: rotate(5deg);
  }
  80% {
    transform: rotate(-5deg);
  }
  100% {
    transform: rotate(0deg);
  }
}
</style>