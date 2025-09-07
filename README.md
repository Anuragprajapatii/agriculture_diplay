
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Farmers Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables for Firebase access
        window.initializeApp = initializeApp;
        window.getAuth = getAuth;
        window.signInAnonymously = signInAnonymously;
        window.signInWithCustomToken = signInWithCustomToken;
        window.getFirestore = getFirestore;
        window.doc = doc;
        window.getDoc = getDoc;
        window.onSnapshot = onSnapshot;
        window.collection = collection;
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0fdf4;
            display: flex;
            flex-direction: column;
            min-height: 100vh;
        }
.markdown-body p a{ 
display:none;
}
      
        /* Fix for Google Translate widget visibility */
        .skiptranslate iframe, .skiptranslate .goog-te-gadget-simple {
            display: block !important;
            visibility: visible !important;
            top: 0 !important;
            right: 0 !important;
            width: auto !important;
            height: auto !important;
        }
        .nav-icon {
            font-size: 1.5rem;
            line-height: 1;
        }
        .active-link {
            color: #16a34a;
        }
        /* Custom styles for the weather page based on weather condition */
        #weather-screen.rainy-bg {
            background-color: #d1d8e0;
            background-image: linear-gradient(to bottom, #d1d8e0, #a9b7c7);
            color: #2c3e50;
        }
        #weather-screen.sunny-bg {
            background-color: #fce38a;
            background-image: linear-gradient(to bottom, #fce38a, #f7ac57);
            color: #d35400;
        }
        .scrollable-list {
            max-height: calc(100vh - 20rem);
            overflow-y: auto;
        }
        /* Hide scrollbar for a cleaner look */
        .scrollable-list::-webkit-scrollbar {
            display: none;
        }
        .scrollable-list {
            -ms-overflow-style: none; /* IE and Edge */
            scrollbar-width: none; /* Firefox */
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .fade-in-card {
            animation: fadeIn 0.5s ease-out;
        }
        .VIpgJd-ZVi9od-ORHb {
          display: none;
    margin: 0;
    background-color: #E4EFFB;
    overflow: hidden;
}
        table {
    display: none;
    border-collapse: separate;
    box-sizing: border-box;
    text-indent: initial;
    unicode-bidi: isolate;
    line-height: normal;
    font-weight: normal;
    font-size: medium;
    font-style: normal;
    color: -internal-quirk-inherit;
    text-align: start;
    border-spacing: 2px;
    border-color: gray;
    white-space: normal;
    font-variant: normal;
}
    </style>
</head>
<body class="bg-gray-100 flex flex-col min-h-screen">

    <!-- Main Content Display with different screens -->
    <main class="flex-1 overflow-y-auto p-4 pb-20">
        <div class="container mx-auto max-w-lg">
            <!-- Google Translate Widget -->
            <div id="google_translate_element" class="fixed top-4 right-4 z-50"></div>

            <!-- Home Screen -->
            <section id="home-screen" class="active-screen">
                <h1 class="text-3xl font-bold text-gray-800 text-center mt-8 mb-6">Field Monitor</h1>
                
                <!-- Temperature Card -->
                <div class="bg-white rounded-2xl shadow-lg p-6 mb-6 fade-in-card">
                    <div class="flex items-center justify-between mb-4">
                        <h2 class="text-xl font-semibold text-gray-700">Temperature</h2>
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-red-500">
                            <path d="M14 14.76V3.5a2.5 2.5 0 0 0-5 0v11.26a4.5 4.5 0 1 0 5 0z"></path>
                        </svg>
                    </div>
                    <div class="text-center font-bold text-gray-800 text-2xl" id="temp-display">25°C</div>
                    <p class="text-sm text-gray-500 text-center mt-2">Current temperature from sensor.</p>
                </div>

                <!-- Water Level Card -->
                <div class="bg-white rounded-2xl shadow-lg p-6 mb-6 fade-in-card">
                    <div class="flex items-center justify-between mb-4">
                        <h2 class="text-xl font-semibold text-gray-700">Water Level</h2>
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-blue-500">
                            <path d="M12 2C6.477 2 2 6.477 2 12s4.477 10 10 10 10-4.477 10-10S17.523 2 12 2z"></path>
                            <path d="M12 6v12"></path>
                            <path d="M12 18a6 6 0 0 0 6-6"></path>
                            <path d="M12 18a6 6 0 0 1-6-6"></path>
                        </svg>
                    </div>
                    <div class="relative pt-1">
                        <div class="overflow-hidden h-4 mb-4 text-xs flex rounded-full bg-blue-200">
                            <div id="water-level-bar" style="width: 75%" class="shadow-none flex flex-col text-center whitespace-nowrap text-white justify-center bg-blue-500 rounded-full"></div>
                        </div>
                    </div>
                    <div class="text-center font-bold text-blue-500 text-2xl" id="water-level-display">75%</div>
                    <p class="text-sm text-gray-500 text-center mt-2">Sufficient for this time of day.</p>
                </div>

                <!-- Humidity Card -->
                <div class="bg-white rounded-2xl shadow-lg p-6 mb-6 fade-in-card">
                    <div class="flex items-center justify-between mb-4">
                        <h2 class="text-xl font-semibold text-gray-700">Humidity</h2>
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-gray-500">
                            <path d="M18 8a6 6 0 0 0-9.673 3.484A8 8 0 0 0 4.588 15h.001"></path>
                            <path d="M18 8a6 6 0 0 0-9.673 3.484A8 8 0 0 0 4.588 15h.001"></path>
                            <path d="M17.886 17.51a5.614 5.614 0 0 0-7.85-2.091"></path>
                            <path d="M16 11.233A5.996 5.996 0 0 0 9.236 7"></path>
                        </svg>
                    </div>
                    <div class="text-center font-bold text-gray-700 text-2xl" id="humidity-display">60%</div>
                    <p class="text-sm text-gray-500 text-center mt-2">Optimal conditions for crop health.</p>
                </div>

                <!-- Irrigation Control Card -->
                <div class="bg-white rounded-2xl shadow-lg p-6 mb-6 fade-in-card">
                    <h2 class="text-xl font-semibold text-gray-700 mb-4 text-center">Irrigation Control</h2>
                    <button id="irrigation-btn" class="w-full py-4 text-lg font-bold text-white rounded-full transition-all duration-300 transform hover:scale-105 active:scale-95 shadow-md bg-green-500">
                        Turn ON Irrigation
                    </button>
                    <div id="status-message" class="text-center mt-4 text-sm font-semibold text-gray-600 hidden"></div>
                </div>
            </section>

            <!-- Weather Screen (Initially hidden) -->
            <section id="weather-screen" class="hidden transition-colors duration-500 rounded-3xl p-6 fade-in-card">
                 <button id="back-from-weather" class="p-2 text-gray-600 hover:text-green-600 rounded-full transition-colors duration-200 absolute top-4 left-4">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M19 12H5"></path>
                        <polyline points="12 19 5 12 12 5"></polyline>
                    </svg>
                </button>
                <div class="text-center">
                    <h1 class="text-4xl font-bold mb-4" id="weather-location"></h1>
                    <div id="weather-icon-container" class="my-4">
                        <!-- Weather icon will be dynamically inserted here -->
                    </div>
                    <p class="text-8xl font-bold leading-none" id="weather-temp"></p>
                    <p class="text-2xl font-semibold mt-2" id="weather-condition"></p>
                    <p class="text-xl mt-4" id="weather-description"></p>
                </div>
            </section>

            <!-- Maps Screen (for States) -->
            <section id="maps-screen" class="hidden flex flex-col md:flex-row gap-6 p-4">
                <div class="w-full md:w-1/2 flex flex-col items-center">
                    <h1 class="text-2xl font-bold text-center mb-4 text-gray-800">Map of India</h1>
                    <div class="bg-gray-200 rounded-lg w-full h-80 flex items-center justify-center p-4">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" class="w-full h-full text-gray-500 opacity-75">
                            <rect width="100" height="100" fill="#E5E7EB" rx="10" ry="10"/>
                            <path d="M50 10 L80 30 L80 70 L50 90 L20 70 L20 30 Z" fill="#9CA3AF" stroke="#6B7280" stroke-width="2"/>
                            <text x="50" y="55" font-family="Arial" font-size="10" fill="#4B5563" text-anchor="middle">Map Placeholder</text>
                        </svg>
                    </div>
                    <div class="text-center mt-4 text-gray-600 font-semibold">
                        (Interactive map coming soon. For now, click a state from the list below.)
                    </div>
                </div>

                <div class="w-full md:w-1/2 bg-white rounded-2xl shadow-lg p-6">
                    <h2 class="text-2xl font-bold text-center mb-4 text-gray-800">Select a State</h2>
                     <input type="text" id="state-search" placeholder="Search for a state..." class="w-full px-4 py-2 mb-4 rounded-lg border border-gray-300 focus:outline-none focus:ring-2 focus:ring-green-500">
                    <div id="states-list" class="scrollable-list h-96">
                        <!-- State buttons will be dynamically inserted here -->
                    </div>
                </div>
            </section>

            <!-- Cities Screen (Initially hidden) -->
            <section id="cities-screen" class="hidden flex flex-col gap-6 p-4 fade-in-card">
                <div class="flex items-center gap-4">
                    <button id="back-to-states" class="p-2 text-gray-600 hover:text-green-600 rounded-full transition-colors duration-200">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M19 12H5"></path>
                            <polyline points="12 19 5 12 12 5"></polyline>
                        </svg>
                    </button>
                    <h1 class="text-2xl font-bold text-center text-gray-800" id="cities-header">Cities</h1>
                </div>
                <input type="text" id="city-search" placeholder="Search for a city..." class="w-full px-4 py-2 rounded-lg border border-gray-300 focus:outline-none focus:ring-2 focus:ring-green-500">
                <div id="cities-list" class="scrollable-list h-96">
                    <!-- City buttons will be dynamically inserted here -->
                </div>
            </section>

        </div>
    </main>

    <!-- Bottom Navigation Bar -->
    <nav class="fixed bottom-0 left-0 w-full bg-white border-t border-gray-200 z-50">
        <div class="container mx-auto px-2 h-16 flex items-center justify-around">

            <!-- Left options -->
            <button id="home-nav" class="flex flex-col items-center p-2 text-green-600 transition-colors duration-200 active-link">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                    <polyline points="9 22 9 12 15 12 15 22"></polyline>
                </svg>
                <span class="text-xs font-medium mt-1">Home</span>
            </button>
            <button id="weather-nav" class="flex flex-col items-center p-2 text-gray-500 hover:text-green-600 transition-colors duration-200">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M3 6l4.5 4.5L3 15"></path>
                    <path d="M21 6l-4.5 4.5L21 15"></path>
                    <path d="M12 3v18"></path>
                </svg>
                <span class="text-xs font-medium mt-1">Weather</span>
            </button>

            <!-- Search Button (Center) -->
            <button id="search-btn" class="w-16 h-16 -mt-10 bg-green-500 text-white rounded-full shadow-lg flex items-center justify-center transition-all duration-300 transform hover:scale-110 focus:outline-none">
                <svg xmlns="http://www.w3.org/2000/svg" width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="11" cy="11" r="8"></circle>
                    <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
                </svg>
            </button>

            <!-- Right options -->
            <button id="maps-nav" class="flex flex-col items-center p-2 text-gray-500 hover:text-green-600 transition-colors duration-200">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5A5.4 5.4 0 0 1 7.5 3c2.24 0 4.08 1.48 4.5 3.65C12.42 4.48 14.26 3 16.5 3A5.4 5.4 0 0 1 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"></path>
                </svg>
                <span class="text-xs font-medium mt-1">Maps</span>
            </button>
            <button id="profile-nav" class="flex flex-col items-center p-2 text-gray-500 hover:text-green-600 transition-colors duration-200">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"></path>
                    <circle cx="12" cy="7" r="4"></circle>
                </svg>
                <span class="text-xs font-medium mt-1">Profile</span>
            </button>
        </div>
    </nav>

    <script src="http://translate.google.com/translate_a/element.js?cb=loadGoogleTranslate">
    </script>

    <script>
        // Global variables provided by the Canvas environment for Firebase
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        let db;
        let auth;
        let userId = null;
        
        // UI elements
        const homeScreen = document.getElementById('home-screen');
        const weatherScreen = document.getElementById('weather-screen');
        const mapsScreen = document.getElementById('maps-screen');
        const citiesScreen = document.getElementById('cities-screen');
        const navButtons = document.querySelectorAll('nav button');

        const irrigationBtn = document.getElementById('irrigation-btn');
        const statusMessage = document.getElementById('status-message');
        const statesList = document.getElementById('states-list');
        const citiesList = document.getElementById('cities-list');
        const weatherLocation = document.getElementById('weather-location');
        const weatherTemp = document.getElementById('weather-temp');
        const weatherCondition = document.getElementById('weather-condition');
        const weatherDescription = document.getElementById('weather-description');
        const weatherIconContainer = document.getElementById('weather-icon-container');
        const citiesHeader = document.getElementById('cities-header');
        const stateSearchInput = document.getElementById('state-search');
        const citySearchInput = document.getElementById('city-search');

        const waterLevelBar = document.getElementById('water-level-bar');
        const waterLevelDisplay = document.getElementById('water-level-display');
        const humidityDisplay = document.getElementById('humidity-display');
        const tempDisplay = document.getElementById('temp-display');
        
        const backToStatesBtn = document.getElementById('back-to-states');
        const backFromWeatherBtn = document.getElementById('back-from-weather');

        // State variables
        let isIrrigationOn = false;
        let previousScreen = 'home-screen';
        
        // Data for states and cities
        const indiaStates = [
            "Andhra Pradesh", "Arunachal Pradesh", "Assam", "Bihar", "Chhattisgarh",
            "Goa", "Gujarat", "Haryana", "Himachal Pradesh", "Jharkhand",
            "Karnataka", "Kerala", "Madhya Pradesh", "Maharashtra", "Manipur",
            "Meghalaya", "Mizoram", "Nagaland", "Odisha", "Punjab", "Rajasthan",
            "Sikkim", "Tamil Nadu", "Telangana", "Tripura", "Uttar Pradesh",
            "Uttarakhand", "West Bengal"
        ];
        
        const citiesData = {
            "Andhra Pradesh": ["Visakhapatnam", "Vijayawada", "Tirupati"],
            "Arunachal Pradesh": ["Itanagar", "Tawang", "Naharlagun"],
            "Assam": ["Guwahati", "Dispur", "Dibrugarh"],
            "Bihar": ["Patna", "Gaya", "Bhagalpur"],
            "Chhattisgarh": ["Raipur", "Bilaspur", "Durg"],
            "Goa": ["Panaji", "Vasco da Gama", "Margao"],
            "Gujarat": ["Ahmedabad", "Surat", "Vadodara"],
            "Haryana": ["Faridabad", "Gurugram", "Panipat"],
            "Himachal Pradesh": ["Shimla", "Manali", "Dharamshala"],
            "Jharkhand": ["Ranchi", "Jamshedpur", "Bokaro"],
            "Karnataka": ["Bengaluru", "Mysuru", "Mangaluru"],
            "Kerala": ["Thiruvananthapuram", "Kochi", "Kozhikode"],
            "Madhya Pradesh": ["Bhopal", "Indore", "Jabalpur"],
            "Maharashtra": ["Mumbai", "Pune", "Nagpur"],
            "Manipur": ["Imphal", "Churachandpur", "Thoubal"],
            "Meghalaya": ["Shillong", "Tura", "Jowai"],
            "Mizoram": ["Aizawl", "Lunglei", "Champhai"],
            "Nagaland": ["Kohima", "Dimapur", "Mokokchung"],
            "Odisha": ["Bhubaneswar", "Cuttack", "Rourkela"],
            "Punjab": ["Amritsar", "Ludhiana", "Jalandhar"],
            "Rajasthan": ["Jaipur", "Jodhpur", "Udaipur"],
            "Sikkim": ["Gangtok", "Namchi", "Pelling"],
            "Tamil Nadu": ["Chennai", "Coimbatore", "Madurai"],
            "Telangana": ["Hyderabad", "Warangal", "Karimnagar"],
            "Tripura": ["Agartala", "Udaipur", "Dharmnagar"],
            "Uttar Pradesh": ["Lucknow", "Kanpur", "Varanasi"],
            "Uttarakhand": ["Dehradun", "Haridwar", "Nainital", "Rishikesh", "Srinagar", "Pauri"],
            "West Bengal": ["Kolkata", "Howrah", "Siliguri"]
        };


        // --- Weather API Integration ---
        const apiKey = "3d15dc6e6cb431fc73cb07b823942a58";
        const apiUrl = "https://api.openweathermap.org/data/2.5/weather?units=metric&q=";

        // Function to fetch and display weather data from API
        async function fetchWeather(location) {
            showScreen('weather-screen');
            statusMessage.textContent = 'Fetching weather data...';
            statusMessage.classList.remove('hidden');

            try {
                const response = await fetch(apiUrl + encodeURIComponent(location) + `&appid=${apiKey}`);
                if (!response.ok) {
                    throw new Error(`Weather data not found for ${location}.`);
                }
                const data = await response.json();
                
                // Update the UI with the fetched data
                updateWeatherUI({
                    location: data.name,
                    weather: data.weather[0].main.toLowerCase(),
                    temperature: `${Math.round(data.main.temp)}°C`,
                    description: data.weather[0].description
                });
                statusMessage.classList.add('hidden');
            } catch (error) {
                console.error("Failed to fetch weather data:", error);
                statusMessage.textContent = error.message;
            }
        }
        
        // Function to update the weather UI based on data
        function updateWeatherUI(data) {
            weatherLocation.textContent = data.location;
            weatherTemp.textContent = data.temperature;
            weatherCondition.textContent = data.weather.charAt(0).toUpperCase() + data.weather.slice(1);
            weatherDescription.textContent = data.description;
            
            // Clear previous classes
            weatherScreen.classList.remove('sunny-bg', 'rainy-bg');
            weatherIconContainer.innerHTML = '';
            
            const isRainy = ['rain', 'drizzle', 'thunderstorm', 'snow'].includes(data.weather);
            const isSunny = ['clear'].includes(data.weather);
            
            if (isRainy) {
                weatherScreen.classList.add('rainy-bg');
                // Rain icon
                weatherIconContainer.innerHTML = `
                    <svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="text-white">
                        <path d="M12 2v2"></path>
                        <path d="M12 22v2"></path>
                        <path d="M4.93 4.93l1.41 1.41"></path>
                        <path d="M17.66 17.66l1.41 1.41"></path>
                        <path d="M2 12h2"></path>
                        <path d="M20 12h2"></path>
                        <path d="M4.93 19.07l1.41-1.41"></path>
                        <path d="M17.66 6.34l1.41-1.41"></path>
                        <path d="M16 12.5a4 4 0 0 1-8 0"></path>
                        <path d="M14 15.5c-.5-.7-1-1.5-2-1.5s-1.5.8-2 1.5"></path>
                    </svg>
                `;
            } else if (isSunny) {
                weatherScreen.classList.add('sunny-bg');
                // Sun icon
                weatherIconContainer.innerHTML = `
                    <svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="text-white">
                        <circle cx="12" cy="12" r="5"></circle>
                        <path d="M12 1v2"></path>
                        <path d="M12 21v2"></path>
                        <path d="M4.22 4.22l1.42 1.42"></path>
                        <path d="M18.36 18.36l1.42 1.42"></path>
                        <path d="M1 12h2"></path>
                        <path d="M21 12h2"></path>
                        <path d="M4.22 19.78l1.42-1.42"></path>
                        <path d="M18.36 5.64l1.42-1.42"></path>
                    </svg>
                `;
            } else {
                 weatherScreen.classList.add('rainy-bg');
                 // Default to a cloud icon for other conditions like "Clouds"
                 weatherIconContainer.innerHTML = `
                    <svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="text-white">
                        <path d="M17.5 19H17A5 5 0 0 0 12 14v-1.5a2.5 2.5 0 0 1 5 0v.5a.5.5 0 0 0 1 0v-.5a2.5 2.5 0 0 1 5 0v.5a.5.5 0 0 0 1 0v-.5A5.5 5.5 0 0 0 17.5 19zM2.5 19H2A5 5 0 0 1 7 14v-1.5a2.5 2.5 0 0 1 5 0v.5a.5.5 0 0 0 1 0v-.5a2.5 2.5 0 0 1 5 0v.5a.5.5 0 0 0 1 0v-.5A5.5 5.5 0 0 0 2.5 19zM12 2v2"></path>
                    </svg>
                 `;
            }
        }
        
        // --- Backend Integration with Firebase Firestore (Simulated) ---
        async function initializeFirebase() {
            if (firebaseConfig) {
                try {
                    const app = initializeApp(firebaseConfig);
                    auth = getAuth(app);
                    db = getFirestore(app);
                    
                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }

                    // Get the user ID after authentication
                    auth.onAuthStateChanged(user => {
                        if (user) {
                            userId = user.uid;
                            console.log(`Firebase initialized. User ID: ${userId}`);
                            // Listen for real-time sensor data updates
                            listenForSensorData();
                        } else {
                            console.log('User signed out or failed to authenticate.');
                        }
                    });

                } catch (error) {
                    console.error("Error initializing Firebase:", error);
                }
            } else {
                console.warn("Firebase configuration not found. Running in simulation mode.");
            }
        }

        // Function to listen for real-time sensor data
        function listenForSensorData() {
            if (!db || !userId) {
                console.error("Firestore not initialized or user not authenticated.");
                return;
            }

            // Path to the sensor data in Firestore
            const sensorDataPath = `/artifacts/${appId}/users/${userId}/sensor_data`;
            
            // This is how you would set up a real-time listener to a document.
            // onSnapshot(doc(db, sensorDataPath, "field_sensors"), (docSnapshot) => {
            //     if (docSnapshot.exists()) {
            //         const data = docSnapshot.data();
            //         console.log("Real-time data received:", data);
            //         // Update UI with the new data
            //         updateDashboardUI(data);
            //     } else {
            //         console.log("No sensor data found.");
            //     }
            // });

            // Since we're in a simulated environment, let's just use mock data
            // to show how the UI would update.
            const mockData = {
                temperature: Math.floor(Math.random() * (40 - 20 + 1) + 20),
                waterLevel: Math.floor(Math.random() * 100),
                humidity: Math.floor(Math.random() * 100),
            };
            updateDashboardUI(mockData);
            
            // Periodically update mock data for demonstration
            setInterval(() => {
                const updatedData = {
                    temperature: Math.floor(Math.random() * (40 - 20 + 1) + 20),
                    waterLevel: Math.floor(Math.random() * 100),
                    humidity: Math.floor(Math.random() * 100),
                };
                updateDashboardUI(updatedData);
            }, 5000);
        }

        function updateDashboardUI(data) {
            tempDisplay.textContent = `${data.temperature}°C`;
            waterLevelBar.style.width = `${data.waterLevel}%`;
            waterLevelDisplay.textContent = `${data.waterLevel}%`;
            humidityDisplay.textContent = `${data.humidity}%`;
        }
        
        // --- UI Logic and Event Handlers ---
        
        // Function to switch between screens
        function showScreen(screenId) {
            const screens = [homeScreen, weatherScreen, mapsScreen, citiesScreen];
            const currentScreen = screens.find(screen => !screen.classList.contains('hidden'));
            if (currentScreen) {
                previousScreen = currentScreen.id;
            }
            
            screens.forEach(screen => {
                if (screen.id === screenId) {
                    screen.classList.remove('hidden');
                } else {
                    screen.classList.add('hidden');
                }
            });

            // Update nav link active state
            navButtons.forEach(btn => {
                btn.classList.remove('active-link', 'text-green-600');
                btn.classList.add('text-gray-500', 'hover:text-green-600');
            });
            const activeNavBtn = document.getElementById(screenId.replace('-screen', '') + '-nav');
            if(activeNavBtn) {
                activeNavBtn.classList.add('active-link', 'text-green-600');
                activeNavBtn.classList.remove('text-gray-500');
            }
        }
        
        // Function to fetch and display crop data
        async function fetchCropData(state) {
            statusMessage.textContent = 'Fetching crop data...';
            statusMessage.classList.remove('hidden');

            try {
                // Simulating a LLM API call using Gemini
                const mockResponse = await new Promise(resolve => setTimeout(() => {
                    // Mock data based on common Indian crops
                    const mockCrops = {
                        "Uttar Pradesh": "Uttar Pradesh is a major producer of wheat and sugarcane. It is also known for producing rice, potatoes, and pulses.",
                        "Punjab": "Known as the 'Granary of India,' Punjab is a significant producer of wheat and rice. Other important crops include cotton and sugarcane.",
                        "Rajasthan": "Rajasthan is a leading producer of bajra (pearl millet), mustard, and other oilseeds. Wheat and pulses are also widely cultivated.",
                        "Maharashtra": "Maharashtra is a key producer of cotton, sugarcane, and grapes. It also contributes significantly to the production of rice and wheat.",
                        "West Bengal": "West Bengal is the top producer of rice and jute. It also cultivates tea, potatoes, and pulses.",
                        "Karnataka": "Karnataka is known for its production of coffee, maize, and pulses. It also grows a variety of other crops like sugarcane and cotton.",
                        "Kerala": "Kerala's agriculture is centered around spices like pepper and cardamom, along with coconut and rubber plantations.",
                    };
                    const text = mockCrops[state] || "Information for this state is not available. Please try another one.";
                    resolve({ text: text });
                }, 1500));
                
                statusMessage.textContent = mockResponse.text;
                statusMessage.classList.remove('hidden');

            } catch (error) {
                console.error("Failed to fetch crop data:", error);
                statusMessage.textContent = 'Error fetching crop data.';
            }
        }

        // Function to populate the list of Indian states
        function populateStatesList(filter = '') {
            statesList.innerHTML = '';
            const filteredStates = indiaStates.filter(state => 
                state.toLowerCase().includes(filter.toLowerCase())
            );

            filteredStates.forEach(state => {
                const stateButton = document.createElement('button');
                stateButton.className = "w-full py-3 px-4 mb-2 text-left bg-gray-50 hover:bg-gray-100 rounded-lg transition-colors duration-200";
                stateButton.textContent = state;
                stateButton.addEventListener('click', () => {
                    populateCitiesList(state);
                });
                statesList.appendChild(stateButton);
            });
        }
        
        // Function to populate the list of cities for a given state
        function populateCitiesList(state, filter = '') {
            showScreen('cities-screen');
            citiesHeader.textContent = `Cities in ${state}`;
            citiesList.innerHTML = '';
            
            const cities = citiesData[state] || [];
            const filteredCities = cities.filter(city => 
                city.toLowerCase().includes(filter.toLowerCase())
            );

            filteredCities.forEach(city => {
                const cityButton = document.createElement('button');
                cityButton.className = "w-full py-3 px-4 mb-2 text-left bg-gray-50 hover:bg-gray-100 rounded-lg transition-colors duration-200";
                cityButton.textContent = city;
                cityButton.addEventListener('click', () => {
                    fetchWeather(city);
                });
                citiesList.appendChild(cityButton);
            });
            
            // Add crop data for the state
            fetchCropData(state);
        }

        // Event Listeners for Nav Buttons
        document.getElementById('home-nav').addEventListener('click', () => showScreen('home-screen'));
        document.getElementById('weather-nav').addEventListener('click', () => {
            showScreen('weather-screen');
            fetchWeather("Srinagar"); // Using a default city.
        });
        document.getElementById('maps-nav').addEventListener('click', () => {
            showScreen('maps-screen');
            populateStatesList();
            stateSearchInput.value = ''; // Clear search on screen change
        });
        document.getElementById('profile-nav').addEventListener('click', () => {
            statusMessage.textContent = 'Profile functionality is not yet implemented.';
            statusMessage.classList.remove('hidden');
            setTimeout(() => statusMessage.classList.add('hidden'), 3000);
        });
        
        // Event listeners for searches and back button
        document.getElementById('back-to-states').addEventListener('click', () => {
            showScreen('maps-screen');
            populateStatesList(stateSearchInput.value);
            citySearchInput.value = '';
        });
        
        document.getElementById('back-from-weather').addEventListener('click', () => {
            showScreen(previousScreen);
        });

        stateSearchInput.addEventListener('keyup', (e) => {
            populateStatesList(e.target.value);
        });

        citySearchInput.addEventListener('keyup', (e) => {
            // Find the current state from the header to populate cities
            const currentState = citiesHeader.textContent.replace('Cities in ', '');
            populateCitiesList(currentState, e.target.value);
        });

        // Irrigation button logic
        irrigationBtn.addEventListener('click', () => {
            isIrrigationOn = !isIrrigationOn;
            const buttonText = isIrrigationOn ? 'Turning OFF Irrigation...' : 'Turning ON Irrigation...';
            const statusText = isIrrigationOn ? 'Irrigation is now ON.' : 'Irrigation is now OFF.';
            const buttonColor = isIrrigationOn ? 'bg-red-500' : 'bg-green-500';

            irrigationBtn.textContent = buttonText;
            irrigationBtn.disabled = true;
            
            // Simulate a network request with a delay
            setTimeout(() => {
                irrigationBtn.textContent = isIrrigationOn ? 'Turn OFF Irrigation' : 'Turn ON Irrigation';
                irrigationBtn.disabled = false;
                statusMessage.textContent = statusText;
                statusMessage.classList.remove('hidden');
                
                irrigationBtn.classList.remove('bg-green-500', 'bg-red-500');
                irrigationBtn.classList.add(buttonColor);
                statusMessage.classList.remove('text-green-600', 'text-red-600', 'text-gray-600');
                statusMessage.classList.add(isIrrigationOn ? 'text-green-600' : 'text-red-600');
            }, 1000);
        });
        
        // Google Translate Callback function
        function loadGoogleTranslate() {
            new google.translate.TranslateElement({
                pageLanguage: 'en',
                includedLanguages: 'en,hi,ta,te,kn,ml,mr,pa,gu,bn', // Example Indian languages
                layout: google.translate.TranslateElement.InlineLayout.SIMPLE,
                autoDisplay: false
            }, 'google_translate_element');
        }

        // Initialize the app on window load
        window.onload = function() {
            // Start Firebase initialization
            initializeFirebase();
            // Show the home screen by default
            showScreen('home-screen');
        };

    </script>
</body>
</html>
