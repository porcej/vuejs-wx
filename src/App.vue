<template>
    <div id="ancestor">
        <div class="container-fluid" id="app">
            <div class="row">
                <div id="sidebar" class="col-md-3 col-sm-4 col-xs-12 sidebar">
                    <div id="search">
                        <input
                            id="location-input"
                            type="text"
                            ref="input"
                            placeholder="Location?"
                            @keyup.enter="organizeAllDetails"
                        >
                        <button id="search-btn" @click="organizeAllDetails">
                            <img src="./assets/Search.svg" width="24" height="24">
                        </button>
                    </div>
                    <current-conditions :currentWeather="currentWeather"></current-conditions>
                </div>
                <dashboard-content
                    class="col-md-9 col-sm-8 col-xs-12 content"
                    id="dashboard-content"
                    :highlights="highlights"
                    :tempVar="tempVar"
                ></dashboard-content>
            </div>
        </div>
    </div>
</template>

<script>
import Utils from './assets/js/wxUtils.js';
import Content from './components/Content.vue';
import CurrentCond from './components/TextInfo.vue';

export default {
    name: 'app',
    components: {
        'dashboard-content': Content,
        'current-conditions': CurrentCond,
    },
    data() {
        return {
            weatherDetails: false,
            location: '', // raw location from input
            lat: '', // raw latitude from google maps api response
            long: '', // raw longitude from google maps api response
            completeWeatherApi: '', // weather api string with lat and long
            rawWeatherData: '', // raw response from weather api
            currentWeather: {
                full_location: '', // for full address
                formatted_lat: '', // for N/S
                formatted_long: '', // for E/W
                time: '',
                temp: '',
                todayHighLow: {
                    todayTempHigh: '',
                    todayTempHighTime: '',
                    todayTempLow: '',
                    todayTempLowTime: ''
                },
                summary: '',
                possibility: ''
            },
            tempVar: {
                tempToday: [
                    // gets added dynamically by this.getSetHourlyTempInfoToday()
                ],
            },
            highlights: {
                uvIndex: '',
                visibility: '',
                windStatus: {
                    windSpeed: '',
                    windDirection: '',
                    derivedWindDirection: ''
                },
            }
        };
    },
    methods: {
        makeInputEmpty: function(){
            this.$refs.input.value = '';
        },
        makeTempVarTodayEmpty: function(){
            this.tempVar.tempToday = [];
        },
        detectEnterKeyPress: function(){
            var input = this.$refs.input;
            input.addEventListener('keyip', function(event){
                event.preventDefault();
                var enterKeyCode = 13;
                if (event.keyCode === enterKeyCode){
                    this.setHitEnterKeyTrue();
                }
            });
        },
        locationEntered: function() {
            var input = this.$refs.input;
            if (input.value === '') {
                this.location = "New York";
            } else {
                this.location = Utils.convertToTitleCase(input.value);
            }
            this.makeInputEmpty();
            this.makeTempVarTodayEmpty();
        },
        // Get Coordinates from Google Maps API
        getCoordinates: function() {
            this.locationEntered();
            var loc = this.location;
            var coords;
            var geocoder = L.mapquest.geocoding();
            return new Promise(function(resolve, reject) {
                geocoder.geocode([loc], function(error, response) {
                    var status = response.info.statuscode;
                    if (status == 0) {
                        var result = response.results[0];
                        coords = {
                            lat:  result.locations[0].latLng.lat,
                            long: result.locations[0].latLng.lng,
                            full_location: result.providedLocation.location
                        };
                        resolve(coords);
                    } else {
                        alert("Oops! Couldn't get data for the location");
                    }
                });
            });
        },
        /*
        The coordinates that Google Maps Geocoder API returns are way too accurate
        for our requirements. We need to bring it into shape before passing the coordinates on 
        to the weather API. Although this is a data processing method in its own right, we can’t 
        help mentioning it right now, because the data acquisition method for the weather API has dependency on the output of this method. 
        */
        setFormatCoordinates: async function() {
            var coordinates = await this.getCoordinates();
            // var coordinates = {
            //     lat: 38.7654628,
            //     long: -77.1594546,
            //     full_location: "6965 Old Brentford Rd, Alexandria, VA"
            // }
            this.lat = coordinates.lat;
            this.long = coordinates.long;
            this.currentWeather.full_location = coordinates.full_location;
            // Remember to beautify lat for N/S
            if (coordinates.lat > 0) {
                this.currentWeather.formatted_lat =
                    (Math.round(coordinates.lat * 10000) / 10000).toString() + '°N';
            } else if (coordinates.lat < 0) {
                this.currentWeather.formatted_lat =
                    (-1 * (Math.round(coordinates.lat * 10000) / 10000)).toString() +
                    '°S';
            } else {
                this.currentWeather.formatted_lat = (
                    Math.round(coordinates.lat * 10000) / 10000
                    ).toString();
            }
            // Remember to beautify long for N/S
            if (coordinates.long > 0) {
                this.currentWeather.formatted_long =
                    (Math.round(coordinates.long * 10000) / 10000).toString() + '°E';
            } else if (coordinates.long < 0) {
                this.currentWeather.formatted_long =
                    (-1 * (Math.round(coordinates.long * 10000) / 10000)).toString() +
                    '°W';
            } else {
                this.currentWeather.formatted_long = (
                    Math.round(coordinates.long * 10000) / 10000
                    ).toString();
            }
        },
        /*
        This method dynamically creates the the correct weather API query URL, based on the 
        formatted latitude and longitude. The complete URL is then fed to the method querying for 
        weather data.
        Notice that the base URL used in this method (without the coordinates) points towards a 
        FusionCharts server — we must redirect our GET request to the weather API through a server to avoid the CORS error.
        */
        fixWeatherApi: async function() {
            await this.setFormatCoordinates();
            var weatherApi = 
                'https://csm.fusioncharts.com/files/assets/wb/wb-data.php?src=darksky&lat=' +
                this.lat +
                '&long=' +
                this.long;
                this.completeWeatherApi = weatherApi;
        },
        fetchWeatherData: async function() {
            await this.fixWeatherApi();
            var axios = require('axios'); // for handling weather api promise
            var weatherApiResponse = await axios.get(this.completeWeatherApi);
            if (weatherApiResponse.status === 200) {
                this.rawWeatherData = weatherApiResponse.data;
                // console.log(this.rawWeatherData);
            } else {
                alert('Hmm... Seems like our weather experts are busy!');
            }
        },
        getTimezone: function() {
            return this.rawWeatherData.timezone;
        },
        getSetCurrentTime: function() {
            var currentTime = this.rawWeatherData.currently.time;
            var timezone = this.getTimezone();
            this.currentWeather.time = Utils.unixToHuman(timezone, currentTime).fullTime;
        },
        getSetSummary: function() {
            var currentSummary = Utils.convertToTitleCase(
                this.rawWeatherData.currently.summary
                );
            if (currentSummary.includes(' And')) {
                currentSummary = currentSummary.replace(' And', ',');
            }
            this.currentWeather.summary = currentSummary;
        },
        getSetPossibility: function() {
            var possible = Utils.formatPossibility(this.rawWeatherData.daily.icon);
            if (possible.includes(' And')) {
                possible = possible.replace(' And', ',');
            }
            this.currentWeather.possibility = possible;
        },
        getSetCurrentTemp: function() {
            var currentTemp = this.rawWeatherData.currently.temperature;
            // this.currentWeather.temp = Utils.fahToCel(currentTemp);
            this.currentWeather.temp = currentTemp;
        },
        getTodayDetails: function() {
            return this.rawWeatherData.daily.data[0];
        },
        getSetTodayTempHighLowWithTime: function() {
            var timezone = this.getTimezone();
            var todayDetails = this.getTodayDetails();
            this.currentWeather.todayHighLow.todayTempHigh = todayDetails.temperatureMax;
            // this.currentWeather.todayHighLow.todayTempHigh = Utils.fahToCel(
            //   todayDetails.temperatureMax
            // );
            this.currentWeather.todayHighLow.todayTempHighTime = Utils.unixToHuman(
                timezone,
                todayDetails.temperatureMaxTime
                ).onlyTime;
            this.currentWeather.todayHighLow.todayTempLow = todayDetails.temperatureMin;
            // this.currentWeather.todayHighLow.todayTempLow = Utils.fahToCel(
            //   todayDetails.temperatureMin
            // );
            this.currentWeather.todayHighLow.todayTempLowTime = Utils.unixToHuman(
                timezone,
                todayDetails.temperatureMinTime
                ).onlyTime;
        },
        getHourlyInfoToday: function() {
            return this.rawWeatherData.hourly.data;
        },
        getSetHourlyTempInfoToday: function() {
            var unixTime = this.rawWeatherData.currently.time;
            var timezone = this.getTimezone();
            var todayMonthDate = Utils.unixToHuman(timezone, unixTime).onlyMonthDate;
            var hourlyData = this.getHourlyInfoToday();
            for (var i = 0; i < hourlyData.length; i++) {
                var hourlyTimeAllTypes = Utils.unixToHuman(timezone, hourlyData[i].time);
                var hourlyOnlyTime = hourlyTimeAllTypes.onlyTime;
                var hourlyMonthDate = hourlyTimeAllTypes.onlyMonthDate;
                if (todayMonthDate === hourlyMonthDate) {
                    var hourlyObject = { hour: '', temp: '' };
                    hourlyObject.hour = hourlyOnlyTime;
                    // hourlyObject.temp = this.fahToCel(hourlyData[i].temperature).toString();
                    hourlyObject.temp = hourlyData[i].temperature.toString();
                    this.tempVar.tempToday.push(hourlyObject);
                    /*
                    Since we are using array.push(), we are just adding elements
                    at the end of the array. Thus, the array is not getting emptied
                    first when a new location is entered.
                    to solve this problem, a method this.makeTempVarTodayEmpty()
                    has been created, and called from this.locationEntered().
                    */
                }
            }
            /*
            To cover the edge case where the local time is between 10 — 12 PM,
            and therefore there are only two elements in the array
            this.tempVar.tempToday. We need to add the points for minimum temperature
            and maximum temperature so that the chart gets generated with atleast four points.
            */
            if (this.tempVar.tempToday.length <= 2) {
                var minTempObject = {
                    hour: this.currentWeather.todayHighLow.todayTempHighTime,
                    temp: this.currentWeather.todayHighLow.todayTempHigh
                };
                var maxTempObject = {
                    hour: this.currentWeather.todayHighLow.todayTempLowTime,
                    temp: this.currentWeather.todayHighLow.todayTempLow
                };
                /*
                Typically, lowest temp are at dawn,
                highest temp is around mid day.
                Thus we can safely arrange like min, max, temp after 10 PM.
                */
                // array.unshift() adds stuff at the beginning of the array.
                // the order will be: min, max, 10 PM, 11 PM.
                this.tempVar.tempToday.unshift(maxTempObject, minTempObject);
            }
        },
        getSetUVIndex: function() {
            var uvIndex = this.rawWeatherData.currently.uvIndex;
            this.highlights.uvIndex = uvIndex;
        },
        getSetVisibility: function() {
            var visibilityInMiles = this.rawWeatherData.currently.visibility;
            // this.highlights.visibility = Utils.mileToKilometer(visibilityInMiles);
            this.highlights.visibility = visibilityInMiles;
        },
        getSetWindStatus: function() {
            var windSpeedInMiles = this.rawWeatherData.currently.windSpeed;
            // this.highlights.windStatus.windSpeed = Utils.mileToKilometer(
            //   windSpeedInMiles
            // );
            this.highlights.windStatus.windSpeed = windSpeedInMiles;
            var absoluteWindDir = this.rawWeatherData.currently.windBearing;
            this.highlights.windStatus.windDirection = absoluteWindDir;
            this.highlights.windStatus.derivedWindDirection = Utils.deriveWindDir(
                absoluteWindDir
            );
        },
        // Top level for info section
        // Data in this.currentWeather
        organizeCurrentWeatherInfo: function() {
            // data in this.currentWeather
            /*
            Coordinates and location is covered (get & set) in:
            — this.getCoordinates()
            — this.setFormatCoordinates()
            There are lots of async-await involved there.
            So it's better to keep them there.
            */
            this.getSetCurrentTime();
            this.getSetCurrentTemp();
            this.getSetTodayTempHighLowWithTime();
            this.getSetSummary();
            this.getSetPossibility();
        },
        // Top level for highlights
        organizeTodayHighlights: function() {
            // top level for highlights
            this.getSetUVIndex();
            this.getSetVisibility();
            this.getSetWindStatus();
        },
        // Top level organization and rendering
        organizeAllDetails: async function() {
            // top level organization
            await this.fetchWeatherData();
            this.organizeCurrentWeatherInfo();
            this.organizeTodayHighlights();
            this.getSetHourlyTempInfoToday();
        }
    },
    mounted: async function() {
        this.location = "New York";
        await this.organizeAllDetails();
    },
    computed: {}
}
</script>