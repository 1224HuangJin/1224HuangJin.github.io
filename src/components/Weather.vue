<template>
  <div class="weather-wrapper" v-if="weatherData.city && weatherData.data.type">
    <div class="static-side">
      <span class="city-name">{{ displayCity }}</span>
      <span class="divider">|</span>
    </div>

    <div class="carousel-container">
      <transition name="slide-fade" mode="out-in">
        <div v-if="step === 0" :key="0" class="weather-item">
          <span class="index-tag">温度</span>
          <span class="weather-icon">{{ extractEmoji(weatherData.data.type) }}</span>
          <span class="temp" :style="{ color: getTempColor(weatherData.data.temp) }">
            {{ weatherData.data.temp }}°C
          </span>
        </div>

        <div v-else-if="step === 1" :key="1" class="weather-item">
          <span class="index-tag">天气状况</span>
          <span class="weather-type">{{ weatherData.data.type }}</span>
        </div>

        <div v-else :key="2" class="weather-item">
          <span class="index-tag">风力级别</span>
          <span class="wind-full">
            {{ weatherData.data.fengxiang }} | {{ weatherData.data.windSpeed }} km/h
          </span>
        </div>
      </transition>
    </div>
  </div>
  <div class="weather-error" v-else>
    <span>加载天气中...</span>
  </div>
</template>

<script setup>
/* 1. 引入 computed */
import { reactive, onMounted, onUnmounted, ref, computed } from "vue";
import axios from "axios";

const weatherData = reactive({
  city: null,
  data: { type: null, temp: null, fengxiang: null, windSpeed: null },
});

/* 2. 创建计算属性：超过 10 个字自动截断并加省略号 */
const displayCity = computed(() => {
  const name = weatherData.city || "";
  return name.length > 10 ? name.slice(0, 10) + "..." : name;
});

const step = ref(0);
let timer = null;

const getTempColor = (temp) => {
  if (temp <= 0) return "#1E90FF";    // 极冷（深蓝色）
  if (temp <= 10) return "#00BFFF";   // 冷（浅蓝色）
  if (temp <= 22) return "#50C878";   // 舒适（翠绿色）
  if (temp <= 30) return "#FFCC00";   // 稍热（暖黄色）
  if (temp <= 38) return "#FF8C00";   // 炎热（深橙色）
  if (temp <= 50) return "#FF4500";   // 极热（橙红色）
  
  // 这里的颜色就是你说的“岩浆感”
  return "#8B0000";                   // 危险/暗红岩浆色 (DarkRed)
}

const extractEmoji = (type) => type?.match(/[\uD800-\uDBFF][\uDC00-\uDFFF]|\u2600|\u2601|\u26C5/)?.[0] || "🌡️";

const weatherMap = {
  0: "☀️ 晴朗", 
  1: "🌤️ 多云", 
  2: "⛅️ 阴天", 
  3: "☁️ 阴云密布", // 或
  45: "🌫 雾", 
  48: "🌫 霾/沙尘", // 对应 Calima（沙尘/烟霞）
  51: "🌧️ 细雨", 
  61: "🌧 小雨", 
  71: "🌨 小雪", 
  80: "🌧️ 阵雨", 
  95: "⛈️ 雷阵雨"
};
const getWindDirLabel = (deg) => {
/*  
const dirs = ["Norte", "Noreste", "Este", "Sureste", "Sur", "Suroeste", "Oeste", "Noroeste"]; 
*/
const dirs = ["北", "东北", "东", "东南", "南", "西南", "西", "西北"];
  return dirs[Math.round(deg / 45) % 8];
};

const getWeatherData = async () => {
  try {
    const locRes = await axios.get("https://ipapi.co/json/");
    weatherData.city = locRes.data.city || "未知";
    const res = await axios.get(`https://api.open-meteo.com/v1/forecast?latitude=${locRes.data.latitude}&longitude=${locRes.data.longitude}&current_weather=true`);
    if (res.data?.current_weather) {
      const now = res.data.current_weather;
      weatherData.data = {
        type: weatherMap[now.weathercode] || "未知",
        temp: Math.round(now.temperature),
        fengxiang: getWindDirLabel(now.winddirection),
        windSpeed: now.windspeed,
      };
    }
  } catch (error) {
    console.error("Error:", error);
  }
};

onMounted(() => {
  getWeatherData();
  timer = setInterval(() => { step.value = (step.value + 1) % 3; }, 4000);
});
onUnmounted(() => clearInterval(timer));
</script>

<style scoped>
/* 保持你原有的样式即可 */
.weather-wrapper {
  display: inline-flex;
  height: 30px;
  align-items: center;
  padding: 0 12px;
  color: #fff;
  font-family: 'Segoe UI', system-ui, sans-serif;
  overflow: hidden; 
}
.static-side {
  display: flex;
  align-items: center;
  flex-shrink: 0;
}
.city-name {
  font-size: 14px;
  font-weight: 700;
  color: #ffffff;
  white-space: nowrap;
}
.divider {
  margin: 0 10px;
  color: rgba(255, 255, 255, 0.2);
  user-select: none;
}
.carousel-container {
  position: relative;
  display: inline-flex;
  align-items: center;
  width: 160px; 
  height: 100%;
  flex-shrink: 0;
}
.weather-item {
  position: absolute;
  left: 0;
  display: flex;
  align-items: center;
  white-space: nowrap;
}
.index-tag {
  font-size: 9px;
  background: #007aff; 
  color: #fff;
  padding: 1px 5px;
  border-radius: 3px;
  margin-right: 8px;
  font-weight: 800;
  flex-shrink: 0;
}
.temp { font-weight: 700; font-size: 15px; }
.weather-type, .wind-full {
  font-size: 13px;
  color: #efefef;
  font-weight: 500;
}
.slide-fade-enter-active, .slide-fade-leave-active {
  transition: all 0.5s ease;
}
.slide-fade-enter-from { transform: translateY(12px); opacity: 0; }
.slide-fade-leave-to { transform: translateY(-12px); opacity: 0; }
</style>