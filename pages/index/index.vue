<template>
    <view :class="['container', backgroundClass]" :key="backgroundClass">
        <view class="header">
            <text class="title">🌤️ Weather Forecast</text>
            <button class="location-btn" @click="getCurrentLocation">📍 Auto-Locate</button>
        </view>

        <view class="search-box">
            <input 
                type="text" 
                v-model="searchText" 
                placeholder="Search city (e.g. Sydney, AU)"
                @confirm="fetchWeather"
            />
            <button @click="fetchWeather">🔍</button>
        </view>

        <view v-if="weatherData" class="weather-card">
            <view class="location-info">
                <text class="city">{{ weatherData.name }}, {{ weatherData.sys.country }}</text>
                <text class="date">{{ formattedDate }}</text>
            </view>

            <view class="weather-status">
                <image :src="getWeatherIcon()" class="weather-icon"></image>
                <text class="condition">{{ weatherData.weather[0].description }}</text>
            </view>

            <view class="temperature-info">
                <text class="current-temp">{{ currentTemp }}°C</text>
                <text class="temp-range">
                    {{ minTemp }}°C / {{ maxTemp }}°C
                </text>
            </view>

            <view class="additional-info">
                <view class="info-item">
                    <text class="value">{{ weatherData.main.humidity }}</text>
                    <text class="label">Humidity</text>
                </view>
                <view class="info-item">
                    <text class="value" aria-label="Wind">{{ weatherData.wind.speed }}</text>
                    <text class="label">Wind</text>
                </view>
                <view class="info-item">
                    <text class="value">{{ weatherData.main.pressure }}</text>
                    <text class="label">Pressure</text>
                </view>
                <view class="info-item">
                    <text class="value">{{ weatherData.visibility / 1000 }}</text>
                    <text class="label">Visibility</text>
                </view>
            </view>

            <view class="sun-info">
                <text class="sunrise">🌅 Sunrise: {{ formattedSunrise }}</text>
                <text class="sunset">🌇 Sunset: {{ formattedSunset }}</text>
            </view>
        </view>

        <view v-else-if="loading" class="loading-screen">
            <text class="loading-text">Fetching weather...</text>
            <loading type="circular" color="#007AFF" size="large" v-show="loading"></loading>
        </view>

        <view v-else class="error-screen">
            <text class="error-message">{{ error }}</text>
            <button class="retry-btn" @click="fetchWeather">🔄 Retry</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            searchText: '',
            weatherData: null,
            loading: false,
            error: null,
            apiKey: '92bb0fc16b81af4918833fed91ac4de7',
            formattedDate: '',
            formattedSunrise: '',
            formattedSunset: '',
            backgroundClass: 'default'
        };
    },
    computed: {
        currentTemp() {
            return this.weatherData ? 
                (this.weatherData.main.temp - 273.15).toFixed(0) : 
                '-';
        },
        minTemp() {
            return this.weatherData ? 
                (this.weatherData.main.temp_min - 273.15).toFixed(0) : 
                '-';
        },
        maxTemp() {
            return this.weatherData ? 
                (this.weatherData.main.temp_max - 273.15).toFixed(0) : 
                '-';
        }
    },
    methods: {
        async fetchWeather() {
            this.loading = true;
            this.error = null;
            const queryCity = this.searchText;

            try {
                const response = await uni.request({
                    url: `https://api.openweathermap.org/data/2.5/weather?q=${queryCity},AU&appid=${this.apiKey}`,
                    method: 'GET'
                });

                if (response.statusCode !== 200) throw new Error('Network error');
                if (response.data.cod !== 200) throw new Error(response.data.message);

                this.weatherData = response.data;
                this.formatTime();
                this.setBackground();
            } catch (err) {
                console.error('Failed to fetch weather data:', err);
                this.error = err.message || 'Location not found';
            } finally {
                this.loading = false;
            }
        },

        getWeatherIcon() {
            return `https://openweathermap.org/img/wn/${this.weatherData.weather[0].icon}@2x.png`;
        },

        formatTime() {
            const now = new Date();
            this.formattedDate = now.toLocaleDateString('en-US', {
                weekday: 'long',
                month: 'long',
                day: 'numeric',
                year: 'numeric'
            });

            const sunrise = new Date(this.weatherData.sys.sunrise * 1000);
            const sunset = new Date(this.weatherData.sys.sunset * 1000);
            this.formattedSunrise = sunrise.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            this.formattedSunset = sunset.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
        },

        async getCurrentLocation() {
            try {
                const { latitude, longitude } = await uni.getLocation({
                    type: 'wgs84'
                });
                const response = await uni.request({
                    url: `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${this.apiKey}`,
                    method: 'GET'
                });
                this.weatherData = response.data;
                this.formatTime();
                this.setBackground();
            } catch (err) {
                this.error = 'Location permission denied';
            }
        },

        setBackground() {
            const condition = this.weatherData?.weather?.[0]?.main?.toLowerCase() || 'default';
            let bgClass = 'default';

            if (condition.includes('clear')) {
                bgClass = 'sunny';
            } else if (condition.includes('cloud')) {
                bgClass = 'cloudy';
            } else if (condition.includes('rain')) {
                bgClass = 'rainy';
            } else if (condition.includes('snow')) {
                bgClass = 'snowy';
            } else if (condition.includes('thunderstorm')) {
                bgClass = 'thunder';
            } else if (condition.includes('drizzle')) {
                bgClass = 'drizzle';
            } else if (condition.includes('mist')) {
                bgClass = 'misty';
            }

            this.backgroundClass = bgClass;
        }
    },
    mounted() {
        // Test data
        // this.weatherData = {
        //     weather: [{ main: 'Rain' }]
        // };
        // this.setBackground();
    }
};
</script>

<style>
.container {
    padding: 20rpx;
    flex: 1;
    min-height: 100vh;
    background-size: cover;
    transition: background-color 0.5s ease;
    position: relative;
    overflow: visible;
}

@keyframes gradientCycle {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}

.sunny {
    background: linear-gradient(
        45deg,
        #FFC3A0,
        #FFAFBD,
        #FF6B6B,
        #FF9A8B,
        #FFE53B
    );
    background-size: 800% 800%;
    animation: gradientCycle 20s ease infinite;
}

.rainy {
    background: linear-gradient(
        135deg,
        #000428,
        #004e92,
        #0074D9,
        #7FDBFF,
        #39CCCC
    );
    background-size: 800% 800%;
    animation: gradientCycle 15s ease infinite;
}

.cloudy {
    background: linear-gradient(
        225deg,
        #E0EAFC,
        #CFDEF3,
        #A6C1EE,
        #6387FF,
        #FFBC52
    );
    background-size: 800% 800%;
    animation: gradientCycle 18s ease infinite;
}

.snowy {
    background: linear-gradient(
        315deg,
        #E0FFFF,
        #AFEEEE,
        #B0E0E6,
        #ADD8E6,
        #87CEFA
    );
    background-size: 800% 800%;
    animation: gradientCycle 22s ease infinite;
}

.thunder {
    background: linear-gradient(
        135deg,
        #141E30,
        #243B55,
        #364F6B,
        #435976,
        #526D8F
    );
    background-size: 800% 800%;
    animation: gradientCycle 12s ease infinite;
}

.drizzle {
    background: linear-gradient(
        135deg,
        #77A1D3,
        #79CBCA,
        #E684AE,
        #F7971E,
        #FFD200
    );
    background-size: 800% 800%;
    animation: gradientCycle 16s ease infinite;
}

.misty {
    background: linear-gradient(
        135deg,
        #BDC3C7,
        #2C3E50,
        #4CA1AF,
        #C4E0E5,
        #D3CCE3
    );
    background-size: 800% 800%;
    animation: gradientCycle 25s ease infinite;
}

.default {
    background: linear-gradient(135deg, #ECEFF1 0%, #CFD8DC 100%);
}

.additional-info {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200rpx, 1fr));
    gap: 20rpx;
    margin: 40rpx 0;
}

.info-item {
    background-color: rgba(236, 236, 236, 0.9);
    padding: 20rpx;
    border-radius: 20rpx;
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    height: 200rpx;
}

.label {
    font-size: 28rpx;
    color: white;
    background-color: #333;
    padding: 8rpx 16rpx;
    border-radius: 10rpx;
    margin-top: auto;
    width: 100%;
    text-align: center;
}

.value {
    font-size: 40rpx;
    font-weight: bold;
    color: #333;
    margin-bottom: 10rpx;
}

.value::after {
    content: '°C';
    font-size: 28rpx;
    color: #666;
    margin-left: 5rpx;
}

.value[aria-label="Wind"]::after {
    content: 'm/s';
}

.value[aria-label="Pressure"]::after {
    content: 'hPa';
}

.value[aria-label="Visibility"]::after {
    content: 'km';
}

@media (max-width: 576rpx) {
    .additional-info {
        grid-template-columns: 1fr;
    }
}

.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 30rpx;
}

.title {
    font-size: 48rpx;
    font-weight: bold;
    color: #333;
}

.location-btn {
    padding: 15rpx 30rpx;
    background-color: #007AFF;
    color: white;
    border: none;
    border-radius: 30rpx;
    font-size: 32rpx;
    cursor: pointer;
    transition: transform 0.3s ease;
}

.location-btn:hover {
    transform: scale(1.05);
}

.search-box {
    display: flex;
    margin-bottom: 40rpx;
}

input {
    flex: 1;
    height: 80rpx;
    padding: 0 30rpx;
    border: none;
    border-radius: 50rpx;
    font-size: 32rpx;
    background-color: rgba(255,255,255,0.8);
}

button {
    margin-left: 20rpx;
    padding: 20rpx 40rpx;
    background-color: #007AFF;
    color: white;
    border: none;
    border-radius: 50rpx;
    font-size: 32rpx;
    cursor: pointer;
    transition: transform 0.3s ease;
}

button:hover {
    transform: scale(1.05);
}

.weather-card {
    background-color: rgba(255,255,255,0.9);
    padding: 40rpx;
    border-radius: 30rpx;
    box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.1);
}

.location-info {
    text-align: center;
    margin-bottom: 40rpx;
}

.city {
    font-size: 48rpx;
    font-weight: bold;
    color: #333;
    margin-bottom: 10rpx;
}

.date {
    font-size: 32rpx;
    color: #666;
}

.weather-status {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 40rpx 0;
}

.weather-icon {
    width: 180rpx;
    height: 180rpx;
    margin-right: 40rpx;
}

.condition {
    font-size: 40rpx;
    font-weight: 500;
    color: #333;
}

.temperature-info {
    text-align: center;
    margin: 40rpx 0;
}

.current-temp {
    font-size: 120rpx;
    font-weight: bold;
    color: #333;
    margin-bottom: 20rpx;
}

.temp-range {
    font-size: 40rpx;
    color: #666;
}

.sun-info {
    display: flex;
    justify-content: center;
    gap: 60rpx;
}

.sunrise, .sunset {
    text-align: center;
}

.sunrise-text, .sunset-text {
    font-size: 30rpx;
    color: #666;
}

.loading-screen {
    text-align: center;
    margin-top: 100rpx;
}

.loading-text {
    font-size: 40rpx;
    color: #666;
    margin-bottom: 40rpx;
}

.error-screen {
    text-align: center;
    margin-top: 100rpx;
}

.error-message {
    font-size: 40rpx;
    color: #ff4444;
    margin-bottom: 40rpx;
}

.retry-btn {
    padding: 20rpx 60rpx;
    background-color: #007AFF;
    color: white;
    border: none;
    border-radius: 50rpx;
    font-size: 32rpx;
    cursor: pointer;
}
</style>    