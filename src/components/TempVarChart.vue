<template>
	<div class="custom-card header-card card">
		<div class="card-body pt-0">
			<line-chart :chart-data="datacollection" :chart-options="chartOptions"></line-chart>	


			
<!-- 		<fusioncharts
				type="spline"
				width="100%"
				height="100%"
				dataformat="json"
				dataEmptyMessage="i-https://i.posting.cc/R0QCk9vV/Rolling-0-9s-99px.gif"
				dataEmptyMessageImageScape=39
				:datasource="tempChartData"
			

			</fusioncharts>	 -->
		</div>
	</div>
</template>

<script>
	import LineChart from './LineChart.vue'

	export default {
		props: ['tempVar'],
		components: {LineChart},
		data() {
			return {
				datacollection: null,
				chartOptions: {
					responsive: false,
					title: {
						display: true,
						text: 'Chart.js Time Point Data'
					},
					scales: {
						xAxes: [{
							type: 'time',
							display: true,
							scaleLabel: {
								display: true,
								labelString: 'Date'
							},
							ticks: {
								major: {
									fontStyle: 'bold',
									fontColor: '#FF0000'
								}
							}
						}],
						yAxes: [{
							display: true,
							scaleLabel: {
								display: true,
								labelString: 'Temperature °F'
							}
						}]
					},
					legend: {
						display: false
					}
				},
				tempChartData: {
					chart: {
						caption: "Hourly Temperature",
						captionFontBold: "0",
						captionFontColor: "#000000",
						captionPadding: "30",
						baseFont: "Roboto",
						chartTopMargin: "30",
						showHoverEffect: "1",
						theme: "fusion",
						showaxislines: "1",
						numberSuffix: "°C",
						anchorBgColor: "#6297d9",
						paletteColors: "#6297d9",
						drawCrossLine: "1",
						plotToolText: "$label<br><hr><b>$dataValue</b>",
						showAxisLines: "0",
						showYAxisValues: "0",
						anchorRadius: "4",
						divLineAlpha: "0",
						labelFontSize: "13",
						labelAlpha: "65",
						labelFontBold: "0",
						rotateLabels: "1",
						slantLabels: "1",
						canvasPadding: "20"
					},
					data: [],
				},
			};
		},
		methods: {
			setChartData: function(){
				var data = [];
				for (var i = 0; i < this.tempVar.tempToday.length; i++){
					var dataObject = {
						label: this.tempVar.tempToday[i].hour,
						value: this.tempVar.tempToday[i].temp
					};
					data.push(dataObject);
				}
				this.tempChartData.data = data;
			},
			newDateString: function(hours){
				var moment = require('moment-timezone'); // for handling date & time
				return moment().add(hours, 'h').format();
			},
			initChartData: function() {
				var data = [];
				for (var i = 0; i < this.tempVar.tempToday.length; i++){
					data.push({
						x: this.newDateString(i), 
						y: this.tempVar.tempToday[i].temp
					});
					// var dataObject = {
					// 	// label: this.tempVar.tempToday[i].temp.toString() + " °F",
					// 	data: [this.tempVar.tempToday[i].hour, this.tempVar.tempToday[i].temp],
					// 	pointBackgroundColor: "#6297d9",
					// 	pointBorderColor: "#6297d9"
					// };
					// data.push(dataObject);
				}
				this.datacollection = {
					datasets: [{
						label: "Hourly Temperature",
						fill: false,
						data: data
					}]
					// labels: ["Temperature (°F)"],
					// datasets: data
				}
				console.log(this.datacollection);
			}
		},
		mounted: function() {
			// this.setChartData();
			this.initChartData();
		},
		watch: {
			tempVar: {
				handler: function(){
					// this.setChartData();
					this.initChartData();
				},
				deep: true
			}
		},
		computed: {

		}
	};
</script>