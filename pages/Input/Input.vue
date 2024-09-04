<template>
  <view class="container">
    <!-- 日历部分 -->
    <view class="calendar">
      <view class="calendar-header">
        <button @click="prevMonth">‹</button>
        <text>{{ currentYear }}年{{ currentMonth }}月</text>
        <button @click="nextMonth">›</button>
      </view>
      <view class="calendar-body">
        <view class="day" v-for="(day, index) in daysInMonth" :key="index" @click="selectDate(day)">
          <text :class="{ selected: selectedDate === day }">{{ day }}</text>
          <text class="eye-pressure-text" v-if="eyePressureData[formattedDate(day)]">眼压: {{ eyePressureData[formattedDate(day)] }}</text>
        </view>
      </view>
    </view>

    <!-- 输入眼压值部分 -->
    <view class="input-area">
      <input type="number" v-model="eyePressure" placeholder="请输入眼压值" />
      <button @click="saveEyePressure">保存</button>
    </view>

    <!-- 时间范围选择部分 -->
    <view class="chart-controls">
      <input type="date" v-model="startDate" />
      <input type="date" v-model="endDate" />
      <button @click="updateChart">更新图表</button>
    </view>

    <!-- 图表部分 -->
    <view class="chart-container">
      <div id="pressureChart"></div>
    </view>
  </view>
</template>

<script>
import * as d3 from 'd3';

export default {
  data() {
    return {
      currentYear: new Date().getFullYear(),
      currentMonth: new Date().getMonth() + 1,
      daysInMonth: [],
      selectedDate: null,
      eyePressure: '',
      eyePressureData: {},
      startDate: '',
      endDate: '',
    };
  },
  methods: {
    initializeDaysInMonth() {
      const days = new Date(this.currentYear, this.currentMonth, 0).getDate();
      this.daysInMonth = Array.from({ length: days }, (_, i) => i + 1);
    },
    formattedDate(day) {
      return `${this.currentYear}-${String(this.currentMonth).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
    },
    selectDate(day) {
      this.selectedDate = day;
      this.eyePressure = ''; // 清空输入框
    },
    saveEyePressure() {
      if (this.selectedDate && this.eyePressure) {
        this.$set(this.eyePressureData, this.formattedDate(this.selectedDate), this.eyePressure);
        this.eyePressure = '';
        this.saveToLocalStorage();
      }
    },
    saveToLocalStorage() {
      localStorage.setItem('eyePressureData', JSON.stringify(this.eyePressureData));
    },
    loadFromLocalStorage() {
      const storedData = localStorage.getItem('eyePressureData');
      if (storedData) {
        this.eyePressureData = JSON.parse(storedData);
      }
    },
    updateChart() {
      if (!this.startDate || !this.endDate) {
        alert('请先选择开始和结束日期');
        return;
      }

      const filteredData = Object.keys(this.eyePressureData)
        .filter(date => date >= this.startDate && date <= this.endDate)
        .map(date => ({
          date: date,
          value: this.eyePressureData[date]
        }));

      if (filteredData.length === 0) {
        alert('所选日期范围内没有数据');
        return;
      }

      this.createD3Chart(filteredData);
    },
	
	createD3Chart(data) {
	  // 清空之前的图表内容
	  d3.select("#pressureChart").selectAll("*").remove();
	
	  const containerWidth = document.getElementById('pressureChart').clientWidth;
	  const margin = { top: 20, right: 30, bottom: 50, left: 50 };
	  const width = containerWidth - margin.left - margin.right;
	  const height = 300 - margin.top - margin.bottom;
	
	  const svg = d3.select("#pressureChart")
	    .append("svg")
	    .attr("width", width + margin.left + margin.right)
	    .attr("height", height + margin.top + margin.bottom)
	    .append("g")
	    .attr("transform", `translate(${margin.left},${margin.top})`);
	
	  const x = d3.scaleTime()
	    .domain(d3.extent(data, d => new Date(d.date)))
	    .range([0, width]);
	
	  const y = d3.scaleLinear()
	    .domain([0, d3.max(data, d => d.value)])
	    .range([height, 0]);
	
	  const allDates = data.map(d => new Date(d.date));
	
	  // 设置 x 轴
	  svg.append("g")
	    .attr("transform", `translate(0,${height})`)
	    .call(d3.axisBottom(x)
	      .tickValues(allDates)
	      .tickFormat(d3.timeFormat('%Y-%m-%d'))
	    )
	    .selectAll("text")
	    .style("text-anchor", "end")
	    .style("font-size", "12px")  // 调整字体大小
	    .attr("dx", "-.8em")
	    .attr("dy", ".15em")
	    .attr("transform", "rotate(-45)");  // 优化旋转角度
	
	  // 设置 y 轴
	  svg.append("g")
	    .call(d3.axisLeft(y)
	      .ticks(6)  // 控制 y 轴的刻度数量
	      .tickSize(-width)  // 将网格线延伸到整个图表宽度
	      .tickPadding(10)
	    )
	    .selectAll("line")
	    .attr("stroke", "#e0e0e0")  // 将网格线设置为浅灰色，减少干扰
	    .attr("stroke-dasharray", "2,2");  // 使用虚线
	
	  svg.append("g")
	    .call(d3.axisLeft(y))
	    .selectAll("text")
	    .style("font-size", "12px");  // 调整字体大小
	
	  // 定义线性渐变
	  const gradient = svg.append("defs")
	    .append("linearGradient")
	    .attr("id", "lineGradient")
	    .attr("x1", "0%")
	    .attr("x2", "100%")
	    .attr("y1", "0%")
	    .attr("y2", "0%");
	
	  gradient.append("stop")
	    .attr("offset", "0%")
	    .attr("stop-color", "steelblue");
	
	  gradient.append("stop")
	    .attr("offset", "100%")
	    .attr("stop-color", "lightblue");
	
	  // 绘制平滑的折线图
	  svg.append("path")
	    .datum(data)
	    .attr("fill", "none")
	    .attr("stroke", "url(#lineGradient)")
	    .attr("stroke-width", 2.5)
	    .attr("d", d3.line()
	      .x(d => x(new Date(d.date)))
	      .y(d => y(d.value))
	      .curve(d3.curveCatmullRom)  // 使用平滑曲线
	    );
	
	  // 添加数据点
	  svg.selectAll("circle")
	    .data(data)
	    .enter().append("circle")
	    .attr("cx", d => x(new Date(d.date)))
	    .attr("cy", d => y(d.value))
	    .attr("r", 5)  // 加大圆圈半径
	    .attr("fill", "steelblue")
	    .attr("stroke", "white")
	    .attr("stroke-width", 2)
	    .attr("class", "data-point")  // 为每个数据点添加类
	    .on("mouseover", function(event, d) {
	      d3.select(this)
	        .transition()
	        .duration(200)
	        .attr("r", 7);  // 鼠标悬停时增大数据点
	    })
	    .on("mouseout", function(event, d) {
	      d3.select(this)
	        .transition()
	        .duration(200)
	        .attr("r", 5);  // 鼠标离开时恢复数据点大小
	    });
	},


	
	
    prevMonth() {
      if (this.currentMonth === 1) {
        this.currentYear -= 1;
        this.currentMonth = 12;
      } else {
        this.currentMonth -= 1;
      }
      this.initializeDaysInMonth();
      this.selectedDate = null; // 重置选中的日期
    },
    nextMonth() {
      if (this.currentMonth === 12) {
        this.currentYear += 1;
        this.currentMonth = 1;
      } else {
        this.currentMonth += 1;
      }
      this.initializeDaysInMonth();
      this.selectedDate = null; // 重置选中的日期
    }
  },
  mounted() {
    this.initializeDaysInMonth();
    this.loadFromLocalStorage(); // 页面加载时从本地存储加载数据
  }
};

</script>

<style scoped>
.container {
  padding: 20px;
  background-color: #eef7f9; /* 增加背景色，使整体更柔和 */
}

.calendar {
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 12px; /* 增加圆角 */
  padding: 15px; /* 增加内边距 */
  background-color: #ffffff; /* 设置为纯白色背景，增加简洁感 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* 增加阴影，使其有浮动感 */
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: bold;
  font-size: 18px; /* 增加字体大小 */
  margin-bottom: 10px;
  color: #333; /* 使用深色字体 */
}

.calendar-header button {
  background-color: #66c1d6; /* 调整按钮颜色，增加饱和度 */
  border: none;
  padding: 8px 12px; /* 增加按钮内边距 */
  font-size: 14px; /* 增大按钮文字 */
  color: white; /* 使用白色字体 */
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.3s; /* 增加过渡效果 */
}

.calendar-header button:hover {
  background-color: #51a5bb; /* 鼠标悬停时按钮颜色变化 */
}

.calendar-body {
  display: flex;
  flex-wrap: wrap;
  background-color: #f9f9f9;
}

.day {
  width: 14.28%;
  text-align: center;
  margin-bottom: 10px;
  cursor: pointer;
  padding: 10px 0;
  transition: background-color 0.3s; /* 添加平滑的背景色过渡效果 */
}

.day:hover {
  background-color: #d4e8eb; /* 鼠标悬停时改变背景色 */
  border-radius: 50%;
}

.day text {
  display: block;
  margin: 5px 0;
  font-size: 14px; /* 增加日期字体 */
  color: #333; /* 深色字体 */
}

.selected {
  background-color: #82c7e6; /* 选中的日期背景色 */
  color: white; /* 选中时字体变为白色 */
  border-radius: 50%;
  transition: background-color 0.3s; /* 平滑的过渡效果 */
}

.eye-pressure-text {
  font-size: 14px; /* 增加字体 */
  color: #666; /* 使用稍深的灰色 */
}

.input-area {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

input {
  flex: 2;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin-right: 10px;
  font-size: 16px; /* 增大输入框字体 */
  transition: border-color 0.3s; /* 添加边框过渡效果 */
}

input:focus {
  border-color: #82c7e6; /* 输入框聚焦时边框颜色变化 */
}

button {
  padding: 8px 16px; /* 增大按钮内边距 */
  background-color: #66c1d6;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px; /* 增大按钮文字 */
  transition: background-color 0.3s;
}

button:hover {
  background-color: #51a5bb; /* 鼠标悬停时颜色变化 */
}

.chart-controls {
  display: flex;
  justify-content: space-around;
  margin-bottom: 40px;
}

.chart-container {
  width: 100%;
  height: auto;
  background-color: #ffffff; /* 使用白色背景，简洁干净 */
  border: 1px solid #ddd; /* 设置边框颜色 */
  border-radius: 12px;
  padding: 15px; /* 增加内边距 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* 增加阴影效果 */
  margin-bottom: 40px;
}

</style>
