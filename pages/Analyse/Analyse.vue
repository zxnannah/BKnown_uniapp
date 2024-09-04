<template>
  <view class="container">
    <!-- 时间范围选择部分 -->
    <view class="card date-picker-card">
      <p class='title'>选择时间范围：</p>
      <view class="date-picker">
        <input type="date" v-model="startDate" />
        <input type="date" v-model="endDate" />
      </view>
      <button @click="analyzeData">分析数据</button>
    </view>

    <!-- 分析结果展示部分 -->
    <view class="analysis-container" v-if="showResults">
      <view class="card">
        <p class="test">眼压平均值</p>
        <p class="value">😊 {{ averagePressure }}</p>
      </view>

      <view class="card">
        <p class="test">最高眼压值</p>
        <p class="value">😱 {{ maxPressure }}</p>
      </view>

      <view class="card">
        <p class="test">眼压趋势分析</p>
        <p class="value">{{ pressureTrend }}</p>
      </view>

      <view v-if="averagePressure > threshold" class="card alert">
        <p>平均眼压高于 {{ threshold }}，请及时就医！</p>
      </view>

      <view v-else class="card">
        <p class="advice">饮食建议：保持清淡饮食，避免摄入过多的盐分和高脂肪食物。</p>
        <p class="advice">用眼建议：避免长时间近距离用眼，适当进行户外活动。</p>
      </view>
    </view>
  </view>
</template>




<script>
export default {
  data() {
    return {
      startDate: '',
      endDate: '',
      eyePressureData: {}, // 从 localStorage 加载的数据
      averagePressure: 0,
      maxPressure: 0,
      pressureTrend: '', // 用于存储眼压趋势
      threshold: 21, // 假设正常眼压的阈值
      showResults: false
    };
  },
  methods: {
    analyzeData() {
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

      this.calculateStatistics(filteredData);
      this.calculateTrend(filteredData); // 调用趋势计算函数
      this.showResults = true;
    },
    calculateStatistics(data) {
      const values = data.map(d => parseFloat(d.value));
      const sum = values.reduce((a, b) => a + b, 0);
      this.averagePressure = (sum / values.length).toFixed(2);
      this.maxPressure = Math.max(...values).toFixed(2);
    },
    calculateTrend(data) {
      // 通过计算前后数据的差异判断趋势
      const values = data.map(d => parseFloat(d.value));

      let increasing = true;
      let decreasing = true;

      for (let i = 1; i < values.length; i++) {
        if (values[i] > values[i - 1]) {
          decreasing = false;
        }
        if (values[i] < values[i - 1]) {
          increasing = false;
        }
      }

      if (increasing) {
        this.pressureTrend = '眼压逐渐升高';
      } else if (decreasing) {
        this.pressureTrend = '眼压逐渐降低';
      } else {
        this.pressureTrend = '眼压趋于平稳';
      }
    },
    loadFromLocalStorage() {
      const storedData = localStorage.getItem('eyePressureData');
      if (storedData) {
        this.eyePressureData = JSON.parse(storedData);
      }
    }
  },
  mounted() {
    this.loadFromLocalStorage(); // 页面加载时从本地存储加载数据
  }
};


</script>

<style scoped>
.container {
  padding: 20px;
  background: linear-gradient(135deg, #e2f7e7, #c8e9db); /* 柔和的渐变背景 */
}

.date-picker-card {
  background-color: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.date-picker {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
  margin-top: 15px;
}

input[type="date"] {
  font-size: 25px; /* 增大字体大小 */
  padding: 18px; /* 增大内边距 */
  border-radius: 8px; /* 添加圆角，优化视觉效果 */
  border: 1px solid #ccc; /* 添加边框 */
  width: 48%; /* 保证两个日期选择器在一行 */
}

input[type="date"]::placeholder {
  font-size: 25px; /* 如果有占位符，也将其字体增大 */
}

.title{
	font-size: 20px;
	font-weight: bold;
}

button {
  width: 100%;
  padding: 4px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 25px;
  font-weight: bold;
  font-family: 'FZLanTingHei', sans-serif;
  cursor: pointer;
  margin-top: 15px;
}

button:hover {
  background-color: #45a049;
}

.analysis-container {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.card {
  background-color: white;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.test{
	 font-size: 25px;
	 font-family: 'FZLanTingHei', sans-serif;
	 color:#1d1d59 ;
}

.value {
  font-size: 40px;
  font-weight: bold;
  margin-top: 5px;
  font-family: 'FZLanTingHei', sans-serif;
   color:#3f6a9f ;
}

.advice{
	font-size:22px;
	font-weight: bold;
	font-family: 'FZLanTingHei', sans-serif;
}

.alert {
  color: red;
  font-weight: bold;
  font-size: 30px;
  font-family: 'FZLanTingHei', sans-serif;
}

</style>
