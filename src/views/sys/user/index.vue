<script>
import { defineComponent } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import interactionPlugin from '@fullcalendar/interaction'
import { INITIAL_EVENTS, createEventId } from './event-utils'
import AddOrUpdate from './add-or-update.vue'

export default defineComponent({
	components: {
		FullCalendar,
		AddOrUpdate
	},
	data() {
		return {
			selectedView: 'timeGridWeek',
			calendarOptions: {
				plugins: [
					dayGridPlugin,
					timeGridPlugin,
					interactionPlugin // needed for dateClick
				],
				// headerToolbar: {
				// 	left: 'prev,next today',
				// 	center: 'title',
				// 	right: 'customSelect,dayGridMonth,timeGridWeek,timeGridDay'
				// },
				customButtons: {
					customSelect: {
						text: 'customSelect',
						click: this.handleCustomSelectClick
					}
				},
				initialView: 'timeGridWeek',
				locale: 'zh-cn',
				initialEvents: INITIAL_EVENTS, // alternatively, use the `events` setting to fetch from a feed
				headerToolbar:false,
				editable: true,
				selectable: true,
				selectMirror: true,
				dayMaxEvents: true,
				weekends: true,
				select: this.handleDateSelect,
				eventClick: this.handleEventClick,
				eventsSet: this.handleEvents,
				eventRender: this.handleEventRender,
				// 设置选择是否允许的回调函数
				selectAllow: this.checkSelectAllow
				// 设置选择约束函数
				// selectConstraint: {
				// 	start: this.getStartOfDay(),
				// 	end: this.getEndOfDay()
				// }
				/* you can update a remote database when these fire:
        eventAdd:
        eventChange:
        eventRemove:
        */
			},
			currentEvents: []
		}
	},
	methods: {
		handleDatesSet(info) {
			console.log(info, 'info')
		},
		handleCustomSelectClick() {
			// 处理自定义选择框按钮点击事件
			const selectedView = this.selectedView

			// 在这里添加处理选择框点击事件的逻辑
			console.log('Custom Select Button Clicked. Selected View:', selectedView)

			// 切换视图
			this.$refs.calendar.getApi().changeView(selectedView)
		},
		handleEventRender(info) {
			// 获取今天的日期
			const today = new Date()
			today.setHours(0, 0, 0, 0)

			// 比较当前日期是否在今天之前
			if (info.event.start < today) {
				// 设置今天之前日期的背景颜色
				const cell = info.el.closest('.fc-day')
				if (cell) {
					cell.style.backgroundColor = '#e0e0e0' // 你可以根据需要设置不同的颜色
				}
			}
		},
		checkSelectAllow(selectInfo) {
			// 判断是否允许选择跨天
			return selectInfo.start.getDate() === selectInfo.end.getDate()
		},
		getStartOfDay() {
			return new Date(new Date().setHours(0, 0, 0, 0))
		},
		getEndOfDay() {
			return new Date(new Date().setHours(23, 59, 59, 999))
		},
		handleWeekendsToggle() {
			this.calendarOptions.weekends = !this.calendarOptions.weekends // update a property
		},

		handleDateSelect(selectInfo) {
			console.log(selectInfo, 'selectInfo')
			const calendarApi = selectInfo.view.calendar
			calendarApi.unselect() // clear date selection
			this.$refs.addOrUpdateRef.init()
			calendarApi.addEvent({
				id: createEventId(),
				title: 'New Event',
				start: selectInfo.startStr,
				end: selectInfo.endStr,
				allDay: selectInfo.allDay
			})
		},
		handleEventClick(clickInfo) {
			console.log(clickInfo, 'clickInfo')
			// if (confirm(`Are you sure you want to delete the event '${clickInfo.event.title}'`)) {
			// 	clickInfo.event.remove()
			// }
		},
		handleEvents(events) {
			this.currentEvents = events
		},

		getDataList() {
			console.log('getDataList')
		}
	}
})
</script>

<template>
	<div class="demo-app">
		<div class="demo-app-sidebar">
			<div class="demo-app-sidebar-section">
				<h2>Instructions</h2>
				<ul>
					<li>Select dates and you will be prompted to create a new event</li>
					<li>Drag, drop, and resize events</li>
					<li>Click an event to delete it</li>
				</ul>
			</div>
			<div class="demo-app-sidebar-section">
				<label>
					<input type="checkbox" :checked="calendarOptions.weekends" @change="handleWeekendsToggle" />
					toggle weekends
				</label>
			</div>
			<div class="demo-app-sidebar-section">
				<h2>All Events ({{ currentEvents.length }})</h2>
				<ul>
					<li v-for="event in currentEvents" :key="event.id">
						<b>{{ event.startStr }}</b>
						<i>{{ event.title }}</i>
					</li>
				</ul>
			</div>
		</div>
		<div class="demo-app-main">
			<div class="fc-toolbar">
            <div class="fc-left">
				1
                <!-- <el-button-group>
                    <el-button @click="month"> 月 </el-button>
                    <el-button @click="week"> 周 </el-button>
                    <el-button @click="today"> 今天 </el-button>
                </el-button-group> -->
            </div>
            <div class="fc-center">
                <!-- <el-button icon="el-icon-arrow-left" @click="prev" /> -->
                <!-- <p class="title">
                    {{ title }}
                </p> -->
                <!-- <el-button icon="el-icon-arrow-right" @click="next" /> -->
            </div>
            <div>
                <el-select
                    style="margin-right: 20px"
                >
                    <el-option
                        v-for="item in options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value"
                    />
                </el-select>
                <!-- <el-button
                    class="search"
                    style="margin-right: 20px"
                    type="button"
                    @click="addEvent"
                >
                    新增
                </el-button>
                <el-button class="search" type="button" @click="search">
                    查询
                </el-button> -->
            </div>
        </div>
			<FullCalendar class="demo-app-calendar" ref="calendar" :options="calendarOptions" @datesSet="handleDatesSet">
				<template v-slot:eventContent="arg">
					<b>{{ arg.timeText }}</b>
					<i>{{ arg.event.title }}哈哈哈</i>
				</template>
			</FullCalendar>
		</div>
		<add-or-update ref="addOrUpdateRef" @refresh-data-list="getDataList"></add-or-update>
	</div>
</template>

<style lang="css">
h2 {
	margin: 0;
	font-size: 16px;
}

ul {
	margin: 0;
	padding: 0 0 0 1.5em;
}

li {
	margin: 1.5em 0;
	padding: 0;
}

b {
	/* used for event dates/times */
	margin-right: 3px;
}

.demo-app {
	display: flex;
	min-height: 100%;
	font-family: Arial, Helvetica Neue, Helvetica, sans-serif;
	font-size: 14px;
}

.demo-app-sidebar {
	width: 300px;
	line-height: 1.5;
	background: #eaf9ff;
	border-right: 1px solid #d3e2e8;
}

.demo-app-sidebar-section {
	padding: 2em;
}

.demo-app-main {
	flex-grow: 1;
	padding: 3em;
}

.fc {
	/* the calendar root */
	max-width: 1100px;
	margin: 0 auto;
}
.fc {
    /* the calendar root */
    max-width: 1100px;
    margin: 0 auto;
}
.fc-toolbar {
    width: 100%;
    margin: 30px auto;
    display: flex;
    flex: 1;
    justify-content: space-around;
    align-content: center;
}
.fc-center {
    display: flex;
    align-content: center;
}
.fc-center .title {
    font-size: 16px;
    padding: 0 15px;
    font-weight: 700;
}
</style>
