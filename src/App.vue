<template>
    <div class="LeftBox"> 
        <div class="ScheduleListBox">
            <!-- 제목 -->
            <header>
                <h3>Schedule📅</h3>
            </header>
            <!-- 출력 -->
            <div class="MainScheduleBox" @click="AddScheduleBt(nowdata.date, nowdata.time)">
                <TransitionGroup name="list" tag="ul">
                    <template v-for="(Schedule, index) in ScheduleList" :key="Schedule.id">
                        <li 
                            class='MainSchedulelist' 
                            v-if="(
                                Schedule.EndDate >= (
                                    new Date().getFullYear() + 
                                    ('0' + (new Date().getMonth() + 1)).slice(-2) + 
                                    ('0' + new Date().getDate()).slice(-2)
                                )
                            )"
                            @click="ViewScheduleBt(index)"
                        >
                            <template
                                v-if="(
                                    Schedule.StartDate > (
                                        new Date().getFullYear() + 
                                        ('0' + (new Date().getMonth() + 1)).slice(-2) + 
                                        ('0' + new Date().getDate()).slice(-2)
                                    )
                                )"
                            >
                                <span class="Schedule">{{ Schedule.Schedule  }}</span>
                                <span class="ScheduleDate">{{ Schedule.StartDate + ' ~'}}</span>
                            </template>
                            <template v-else>
                                <span class="Schedule">{{ Schedule.Schedule  }}</span>
                                <span class="ScheduleDate">{{ '~ ' + Schedule.EndDate}}</span>
                            </template>
                        </li>
                    </template>
                </TransitionGroup>
            </div>
        </div>
        <div class="TodolistBox">
            <todolist @AddTodoBt="AddTodoBt" @CheckTodoBt="CheckTodoBt"></todolist>
        </div>
    </div>
    <div v-if="nowdata.bt === 'MonthBt'" class="CalendarBox">
        <month @WeekBt="WeekBt" @DayBt="DayBt" :ScheduleList="ScheduleList" @AddScheduleBt="AddScheduleBt" @ViewScheduleBt="ViewScheduleBt"></month>
    </div>
    <div v-else-if="nowdata.bt === 'WeekBt'" class="CalendarBox">
        <week @MonthBt="MonthBt" @DayBt="DayBt" :ScheduleList="ScheduleList" @AddScheduleBt="AddScheduleBt" @ViewScheduleBt="ViewScheduleBt"></week>
    </div>
    <div v-else class="CalendarBox">
        <day @MonthBt="MonthBt" @WeekBt="WeekBt" :ScheduleList="ScheduleList" @AddScheduleBt="AddScheduleBt" @ViewScheduleBt="ViewScheduleBt"></day>
    </div>
    <div class="ScheduleClickBox" v-if="BtState">
        <div class="SchedulePositionBox">
            <schedulelist @AddScheduleList="AddScheduleList" @CancelBt="CancelBt" @DeleteSchedule="DeleteSchedule" @EditScheduleList="EditScheduleList"></schedulelist>
        </div>
    </div>
</template>

<script>
    import todolist from './components/todolist.vue'
    import month from './components/month.vue'
    import week from './components/week.vue'
    import day from './components/day.vue'
    import schedulelist from './components/schedulelist.vue'

    export default {
        name: 'App',
        components: {
            todolist,
            month,
            week,
            day,
            schedulelist
        },
        data() {
            return {
                ScheduleList: [],
                nowdata: {},
                todaydata: {
                    year: new Date().getFullYear(),
                    month: new Date().getMonth() + 1,
                    date: new Date().getDate(),
                    time: ('0' + new Date().getHours()).slice(-2) + ":" + ('0' + new Date().getMinutes()).slice(-2),
                    bt : "MonthBt",
                    schedule: "add"
                },
                BtState: false
            }
        },
        created() {
            this.LoadCalendarDate();
        },
        methods: {
            LoadCalendarDate() {
                var savedata = localStorage.getItem("nowdata");
                this.nowdata = (savedata != null) ? JSON.parse(savedata) : this.todaydata;
                localStorage.setItem("nowdata", JSON.stringify(this.nowdata));
                var saveschedulelist = localStorage.getItem("schedulelist");
                this.ScheduleList = (saveschedulelist != null) ? JSON.parse(saveschedulelist) : [];
            },
            MonthBt() {
                var savedata = localStorage.getItem("nowdata");
                this.nowdata = (savedata != null) ? JSON.parse(savedata) : this.todaydata;
                this.nowdata.bt = "MonthBt";
                localStorage.setItem("nowdata", JSON.stringify(this.nowdata));
            },
            WeekBt() {
                var savedata = localStorage.getItem("nowdata");
                this.nowdata = (savedata != null) ? JSON.parse(savedata) : this.todaydata;
                this.nowdata.bt = "WeekBt";
                localStorage.setItem("nowdata", JSON.stringify(this.nowdata));
            },
            DayBt() {
                var savedata = localStorage.getItem("nowdata");
                this.nowdata = (savedata != null) ? JSON.parse(savedata) : this.todaydata;
                this.nowdata.bt = "DayBt";
                localStorage.setItem("nowdata", JSON.stringify(this.nowdata));
            },
            AddTodoBt(NewTodo) {
                var NewTodoJson = {};

                if(NewTodo == ''){
                    alert('값을 입력해 주세요.');
                    return;
                }

                var savetodolist = localStorage.getItem("todolist");
                var todolist = (savetodolist != null) ? JSON.parse(savetodolist) : []; 

                NewTodoJson.id = (todolist.length !== 0) ? Number([...todolist].sort((a, b) => b['id'] - a['id'] )[0].id) + 1 : 1;
                NewTodoJson.Todo = NewTodo;

                todolist.push(NewTodoJson);
                todolist.sort((a, b) => a['checkstate'] - b['checkstate'] )
                localStorage.setItem("todolist", JSON.stringify(todolist));
                this.LoadCalendarDate();
            },
            CheckTodoBt(index) {
                var savetodolist = localStorage.getItem("todolist");
                var todolist = (savetodolist != null) ? JSON.parse(savetodolist) : []; 
                todolist.splice(index, 1);
                localStorage.setItem("todolist", JSON.stringify(todolist));
                this.LoadCalendarDate();
            },
            AddScheduleBt(date, time) {
                var savedata = localStorage.getItem("nowdata");
                this.nowdata = (savedata != null) ? JSON.parse(savedata) : this.todaydata;
                this.nowdata.date = date;
                this.nowdata.time = time;
                this.nowdata.schedule = "add";
                localStorage.setItem("nowdata", JSON.stringify(this.nowdata));
                this.BtState = true;
                this.LoadCalendarDate();
            },
            CancelBt() {
                this.BtState = false;
                this.LoadCalendarDate();
            },
            AddScheduleList(NewSchedule, NewStartDate, NewStartTime, NewEndDate, NewEndTime) {
                var NewScheduleJson = {};

                if(NewSchedule == ''){
                    alert('값을 입력해 주세요.');
                    return;
                } else if(NewStartDate > NewEndDate){
                    alert('시작날짜가 종료날짜보다 빨라야 합니다.');
                    return;
                } else if(NewStartDate === NewEndDate && NewStartTime > NewEndTime){
                    alert('시작시간이 종료시간보다 빨라야 합니다.');
                    return;
                }

                var saveschedulelist = localStorage.getItem("schedulelist");
                var schedulelist = (saveschedulelist != null) ? JSON.parse(saveschedulelist) : []; 

                NewScheduleJson.id = (schedulelist.length !== 0) ? Number([...schedulelist].sort((a, b) => b['id'] - a['id'] )[0].id) + 1 : 1;
                NewScheduleJson.Schedule = NewSchedule;
                NewScheduleJson.StartDate = NewStartDate;
                NewScheduleJson.StartTime = NewStartTime;
                NewScheduleJson.EndDate = NewEndDate;
                NewScheduleJson.EndTime = NewEndTime;

                schedulelist.push(NewScheduleJson);
                schedulelist.sort((a, b) => a['EndDate'] - b['EndDate'] || a['EndTime'] - b['EndTime'])
                localStorage.setItem("schedulelist", JSON.stringify(schedulelist));
                this.CancelBt();
            },
            ViewScheduleBt(index) {
                var savedata = localStorage.getItem("nowdata");
                this.nowdata = (savedata != null) ? JSON.parse(savedata) : this.todaydata;
                this.nowdata.schedule = "view";
                this.nowdata.scheduleindex = index;
                localStorage.setItem("nowdata", JSON.stringify(this.nowdata));
                this.BtState = true;
            },
            DeleteSchedule(index) {
                var saveschedulelist = localStorage.getItem("schedulelist");
                var schedulelist = (saveschedulelist != null) ? JSON.parse(saveschedulelist) : []; 
                schedulelist.splice(index, 1);
                localStorage.setItem("schedulelist", JSON.stringify(schedulelist));
                this.CancelBt();
                this.LoadCalendarDate();
            },
            EditScheduleList(NewSchedule, NewStartDate, NewStartTime, NewEndDate, NewEndTime, index) {
                if(NewSchedule == ''){
                    alert('값을 입력해 주세요.');
                    return;
                }

                var saveschedulelist = localStorage.getItem("schedulelist");
                var schedulelist = (saveschedulelist != null) ? JSON.parse(saveschedulelist) : []; 

                schedulelist[index].Schedule = NewSchedule;
                schedulelist[index].StartDate = NewStartDate;
                schedulelist[index].StartTime = NewStartTime;
                schedulelist[index].EndDate = NewEndDate;
                schedulelist[index].EndTime = NewEndTime;

                schedulelist.sort((a, b) => a['EndDate'] - b['EndDate'] || a['EndTime'] - b['EndTime'])
                localStorage.setItem("schedulelist", JSON.stringify(schedulelist));
                this.CancelBt();
            },
        } 
    }
</script>

<style>
    body {
        height: 100%;
        overflow: "hidden";
    }
    #app {
        display: flex;
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
    .LeftBox {
        width: 18%;
        height: 1000px;
        margin: 0.5% 0.25% 0.5% 0.5%;
        background-color: rgb(240, 240, 240);
    }
    .ScheduleListBox {
        height: 495px;
        padding: 0% 1% 0.5% 1%;
    }
    h3 {
        color: #2F3B52;
        font-weight: 900;
        padding: 2.5rem 0 1.5rem;
        margin: 0;
    }
    .MainSchedulelist {
        display: flex;
        min-height: 35px;
        height: 35px;
        line-height: 35px;
        margin: 0.5rem 0;
        padding: 0 0.9rem;
        background: white;
        border-radius: 5px;
        z-index: 500;
    }
    .Schedule {
        font-family: "NotoSansBold";
        margin-right: auto;
    }
    .ScheduleDate {
        font-size: 12px;
        padding-top: 3px;
    }
    .TodolistBox {
        height: 500px;
        padding: 1% 3% 0% 3%;
        border-top: 5px dotted rgb(150, 150, 150);
    }
    .CalendarBox {
        width: 80%;
        height: 1000px;
        margin: 0.5% 0.5% 0.5% 0.25%;
        background-color: rgb(240, 240, 240);

    }
    /* CalendarNav */
    .CalendarNav {
        height: 9.5%;
        padding: 0% 1% 0.5% 1%;
        display: flex;
        justify-content: space-between;
    }
    .CalendarTitleBox {
        width: 22%;
        margin: auto 0 auto 0;
        display: flex;
    }
    .CalendarTitle {
        width: 70%;
        font-family: "NotoSansBold";
        font-size: 35px;
        font-weight: 900;
        margin-right: 15px;
        white-space:nowrap;
    }
    .PreTodayNextBt {
        width: 15%;
        margin: auto 0 auto 0;
        font-family: "NotoSansBold";
        font-size: 18px;
        font-weight: normal;
        display: flex;
        background-color: rgb(225, 225, 225);
        border-radius: 7px;
        cursor: pointer;
    }
    .PreBt {
        width: 33%;
        font-family: "NotoSansBold";
        font-size: 20px;
        font-weight: 900;
        border-right: 2px dotted rgb(150, 150, 150);
        padding-bottom: 5px;
        cursor: pointer;
    }
    .TodayBt {
        width: 33%;
        font-family: "NotoSansBold";
        font-size: 17px;
        font-weight: normal;
        border-right: 2px dotted rgb(150, 150, 150);
        padding: 4.5px 3px 0 3px;
        cursor: pointer;
    }
    .NextBt {
        width: 33%;
        font-family: "NotoSansBold";
        font-size: 20px;
        font-weight: 900;
        padding-bottom: 5px;
        cursor: pointer;
    }
    .CalendarToday {
        margin: auto 0 auto 0;
        padding: 5px;
        font-family: "NotoSansBold";
        font-size: 18px;
        font-weight: normal;
        background-color: rgb(225, 225, 225);
        border-radius: 7px;
        cursor: pointer;
    }
    .MonthWeekDayBt {
        width: 20%;
        margin: auto 0 auto 0;
        font-family: "NotoSansBold";
        font-size: 18px;
        font-weight: normal;
        display: flex;
        background-color: rgb(225, 225, 225);
        border-radius: 7px;
        cursor: pointer;
    }
    .MonthBt {
        width: 33%;
        font-family: "NotoSansBold";
        padding: 5px;
        font-size: 18px;
        font-weight: normal;
        border-right: 2px solid rgb(150, 150, 150);
        border-radius: 7px 0 0 7px;
        cursor: pointer;
    }
    .WeekBt {
        width: 33%;
        font-family: "NotoSansBold";
        padding: 5px;
        font-size: 18px;
        font-weight: normal;
        border-right: 2px solid rgb(150, 150, 150);
        cursor: pointer;
    }
    .DayBt {
        width: 33%;
        font-family: "NotoSansBold";
        padding: 5px;
        font-size: 18px;
        font-weight: normal;
        border-radius: 0 7px 7px 0;
    }
    .Select {
        color: rgb(250, 250, 250);
        background-color:  rgb(150, 150, 150);
    }
    .ScheduleClickBox {
        height: 100%;
        width: 100%;
        position: fixed;
        top: 0;
        bottom: 0;
        right: 0;
        left: 0;
    }
    .SchedulePositionBox {
        display: flex;
        flex-direction: column;
        position: fixed;
        top: 0;
        bottom: 0;
        right: 0;
        left: 0;
        align-items: center;
        justify-content: center;
        z-index: 1000;
        width: 100%;
        height: 100%;
        background-color: transparent;
        padding: 0 0.5% 0 9%;
    }
    .MainScheduleBox {
        height: 350px;
        overflow: scroll;
    }
    .MainScheduleBox::-webkit-scrollbar {
        display:none;
    }
</style>
