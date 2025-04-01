Core File Description
	1.	pages/index/index.vue
	•	Main page component, includes:
	•	City/Zip code input field
	•	Weather query button
	•	Local weather button
	•	Weather information display area
	•	Data interaction logic (fetch weather data via API)
	•	Background color changes based on weather condition

	2.	App.vue
	•	Global app configuration component
	•	Global style definitions
	•	Application-level lifecycle hooks
	•	Global event listeners

	3.	index.html
	•	Entry file for browser-side (H5)

	4.	main.js
	•	Program entry file
	•	Vue initialization configuration
	•	UniApp framework initialization
	•	Global component/directive registration

	5.	pages.json
	•	Routing configuration file:
	•	Defines page paths and window styles
	•	Global settings for navigation bar, titles, etc.

	6.	manifest.json
	•	Multi-platform packaging configuration:
	•	App name, icon, version
	•	Platform-specific settings (e.g., WeChat Mini Program AppID)
	•	Permission declarations (e.g., location access)

	7.	static directory
	•	Stores static resources not involved in compilation (not used in this project):
	•	Weather icons
	•	Background images
	•	Third-party font files

	8.	uni.promisify.adaptor.js
	•	Promise adapter
   
	9.	uni.scss
	•	Custom styles
	•	Stores custom style files
	•	Supports SCSS preprocessor
	•	Recommended for organizing global styles

⸻

Features
	•	Supports multi-platform deployment (H5 / Mini Programs / App)
	•	Responsive layout for various screen sizes
	•	Uses OpenWeatherMap API to request weather data
	•	Supports both city name and zip code search
	•	Displays real-time weather information (temperature, humidity, wind, etc.)

⸻

Tech Stack
	•	Framework: UniApp 3.x
	•	Language: Vue 2
	•	UI Components: Native components + custom styles
	•	Network Request: uni.request API
	•	Data Processing: JSON format parsing
