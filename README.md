# vue_weather_dashboard

> A Simple Weather Dashboard using Darksky API in Vue JS

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```


## Getting started

1. Download *.zip or clone project to your machine.
2. Run `npm install` from the project directory
3. Go to *[MapQuest Open API](https://developer.mapquest.com/plan_purchase/steps/business_edition/business_edition_free/register)* and create your own key
4. Copy this key
5. Go to file `index.html` in the project and insert key instead of `insert_mapquest_open_api_key_here`
6. Run `npm run dev`
For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).

## Building For Production
1. Follow the steps in Getting started
2. Run `npm run build`


## Notes
- Based on Smashing Magazine's Article [Using Vue.js To Create an Interactive Weather Dashboard With APIs](https://www.smashingmagazine.com/2019/02/interactive-weather-dashboard-api-vue-js/).
- *wxUtils.js* was created to pull the utility functions out of *App.vue*

## TODO
- Change WX API as DarkSky will be going away due to its acquisition by Apple.
