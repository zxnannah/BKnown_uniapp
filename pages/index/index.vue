<template>
  <view class="container">
	<image class="logo" src="/static/bknown.png" @click="navigateToIntroduce" mode="widthFix"></image>
	<image class="image" src="/static/image.png" mode="widthFix"></image>	  
	<image class="navigate-image-1" src="/static/eye-test.png" @click="navigateToInput" mode="widthFix"></image>
	<image class="navigate-image-2" src="/static/blackboard.png" @click="navigateToAnalysis" mode="widthFix"></image>
  </view>

</template>

<script>
export default {
  methods: {
    navigateToInput() {
      uni.navigateTo({
        url: '/pages/Input/Input'
      });
    },
    navigateToAnalysis() {
      uni.navigateTo({
        url: '/pages/Analyse/Analyse'
      });
    },
	navigateToIntroduce() {
	  uni.navigateTo({
	    url: '/pages/Introduce/Introduce'
	  });
	}
  }
}
</script>


<style>
.container {
  padding: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f5f5f5;
}

.navigate-image-1 {
	width: 50px;
	height: 50px;
	position: fixed;
	bottom: 50px;
	left: 100px; /* 设置距离左边的位置 */
}

.navigate-image-2 {
	width: 50px;
	height: 50px;
	position: fixed;
	bottom: 50px;
	right: 100px; /* 设置距离右边的位置 */
}

.logo {
	width: 10cm;
	height: 4cm;
	position: fixed;
	top: 150px;
}

.image{
	position: fixed;
}

</style>