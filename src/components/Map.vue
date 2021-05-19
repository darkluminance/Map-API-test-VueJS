<template>
	<div id="mapid"></div>
</template>

<script>
	import { computed, onMounted, ref } from 'vue';
	export default {
		props: {},
		setup(props, context) {
			var curlocation = ref([0, 0]); //Storing the current location
			var startlocation = ref([0, 0]); //Storing the current location
			var startlocationname = ref(''); //Storing the start location name
			var targetlocation = ref([0, 0]); //Storing the target location
			var targetlocationname = ref(''); //Storing the target location name
			var mymap = null;
			var locmarker = ref(null);
			var routedvals = ref([]);
			var routestorage = null;
			var firsttime = true;
			let rfare = 0;
			let rprem = 0;

			const message = computed(() => {
				return startlocationname.value;
			});

			//Get the map data from the API
			const getMapData = () => {
				//Making the map and adding the tiles

				mymap = L.map('mapid');

				L.tileLayer(
					'https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}',
					{
						attribution:
							'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
						maxZoom: 21,
						id: 'mapbox/streets-v11',
						tileSize: 512,
						zoomOffset: -1,
						accessToken:
							'pk.eyJ1IjoiZGFya2x1bWluYW5jZSIsImEiOiJja29zcGJrdDIwM291Mm9xanczMnRtNWc1In0.4y8psgTFHMCq2CR0A6AszQ',
					}
				).addTo(mymap);

				/* L.tileLayer('http://tiles.mapc.org/basemap/{z}/{x}/{y}.png', {
					attribution:
						'Tiles by <a href="http://mapc.org">MAPC</a>, Data by <a href="http://mass.gov/mgis">MassGIS</a>',
					maxZoom: 17,
					minZoom: 9,
				}).addTo(mymap); */

				//Adding the marker with custom icon
				var mapIcon = L.icon({
					iconUrl: 'https://i.imgur.com/QEBEeOP.png',

					iconSize: [28, 28], // size of the icon
					iconAnchor: [14, 14], // point of the icon which will correspond to marker's location
				});

				locmarker = L.marker([0, 0], { icon: mapIcon }).addTo(mymap);

				//search functionality
				/* var searchControl = L.esri.Geocoding.geosearch().addTo(mymap);

				var results = L.layerGroup().addTo(mymap);

				searchControl.on('results', function(data) {
					results.clearLayers();
					for (var i = data.results.length - 1; i >= 0; i--) {
						results.addLayer(L.marker(data.results[i].latlng));
						let pointloc = [];
						pointloc.push(data.results[i].latlng.lat);
						pointloc.push(data.results[i].latlng.lng);
						targetlocation.value = pointloc;
						//gettheroute(pointloc[0], pointloc[1]);
					}
				}); */
				L.Control.geocoder().addTo(mymap);

				mymap.on('click', function(e) {
					if (e.originalEvent.altKey) {
						startlocation.value = [e.latlng.lat, e.latlng.lng];
						getstartpositionname(e.latlng.lat, e.latlng.lng);
					} else {
						let pointlocc = [e.latlng.lat, e.latlng.lng];
						targetlocation.value = pointlocc;
					}
					gettheroute(targetlocation.value[0], targetlocation.value[1]);
				});
			};

			function getLocation() {
				if (navigator.geolocation) {
					return new Promise((showPosition) => {
						navigator.geolocation.getCurrentPosition(showPosition);
					});
				} else {
					alert(
						'Cannot find the location. Please check your GPS or browser compatibility.'
					);
				}
			}

			function showPosition(position) {
				return [position.coords.latitude, position.coords.longitude];
			}

			async function findloc() {
				let calculatedPos = await getLocation();
				let foundpos = [
					calculatedPos.coords.latitude,
					calculatedPos.coords.longitude,
				];
				setthepos(foundpos);
			}

			function setthepos(loca) {
				locmarker.setLatLng(loca);

				//Add map circle 	ONLY IF FIRST TIME
				if (firsttime) {
					var radius = 80;

					//---------------CHANGE THIS circleMarker() to circle() AND REMOVE SET RADIUS PART IF ANY ERROR ----------
					L.circle(loca, radius, {
						opacity: 0.5,
						weight: 1,
						fillOpacity: 0.2,
					})
						// .setRadius(420 / zoomlevel.value)
						.addTo(mymap);
					mymap.setView(loca, 18, { animation: true });
				}

				curlocation.value = loca;
			}

			//routing
			async function gettheroute(xpos, ypos) {
				//Clear previous routes
				if (routestorage != null) {
					mymap.removeControl(routestorage);
					routestorage = null;
				}

				//Set the mapview to middle point of the two points
				mymap.setView(
					[
						(xpos + startlocation.value[0]) / 2,
						(ypos + startlocation.value[1]) / 2,
					],
					13,
					{ animation: true }
				);

				//Custom marker for route points
				var fromIcon = new L.Icon({
					iconUrl: 'https://i.imgur.com/dj2M62Z.png',
					iconSize: [20, 20],
					iconAnchor: [10, 12],
				});
				var toIcon = new L.Icon({
					iconUrl: 'https://i.imgur.com/i0DSvtU.png',
					iconSize: [20, 20],
					iconAnchor: [10, 12],
				});

				//Store the new route and add it to the map
				routestorage = L.Routing.control({
					waypoints: [
						L.latLng(startlocation.value[0], startlocation.value[1]),
						L.latLng(xpos, ypos),
					],
					lineOptions: {
						styles: [{ color: '#6c5ce7', opacity: 1, weight: 5 }],
					},
					routeWhileDragging: true,
					autoRoute: true,

					createMarker: function(i, wp, nWps) {
						if (i === 0) {
							// here change the starting and ending icons
							return L.marker(wp.latLng, {
								icon: fromIcon, // here pass the custom marker icon instance
							});
						} else if (i === nWps - 1) {
							// here change the starting and ending icons
							return L.marker(wp.latLng, {
								icon: toIcon, // here pass the custom marker icon instance
							});
							/* .bindPopup('Hello World')
								.on('add', function(event) {
									event.target.openPopup();
								}); */
						} else {
							// here change all the others
							return L.marker(wp.latLng, {
								icon: toIcon,
							});
						}
					},
				}).addTo(mymap);
				routestorage.hide();

				let routedist = 0;
				let routedtime = 0;

				await new Promise(() =>
					routestorage.on('routesfound', function(e) {
						var routes = e.routes;
						var summary = routes[0].summary;
						// alert distance and time in km and minutes
						/* alert(
						'Total distance is ' +
							summary.totalDistance / 1000 +
							' km and total time is ' +
							(summary.totalTime % 3600) / 60 +
							' minutes'
					); */
						routedist = summary.totalDistance;
						routedtime = summary.totalTime;
						var basefare = 40;
						var basefareprem = 80;
						var costperminute = 3;
						var timeofride = routedtime / 60;
						var costperkm = 21;
						var costperkmprem = 25;
						var ridedistance = routedist / 1000;
						var bookingfee = 30;

						var ridefare =
							basefare +
							(costperminute * timeofride + costperkm * ridedistance) +
							bookingfee;

						var ridefareprem =
							basefareprem +
							(costperminute * timeofride + costperkmprem * ridedistance) +
							bookingfee;

						ridefare = Math.round(ridefare);
						ridefareprem = Math.round(ridefareprem);

						rfare = ridefare;
						rprem = ridefareprem;

						getpositionname(xpos, ypos);

						setTimeout(() => {
							console.log(
								'Total distance is : ' +
									routedist / 1000 +
									' km.\n' +
									'Total time is : ' +
									routedtime / 60 +
									' minutes.\n' +
									'Destination is : ' +
									targetlocationname.value +
									'\nFrom : ' +
									startlocationname.value +
									'\nEstimated fare for the trip is : Tk. ' +
									ridefare
							);
							context.emit(
								'updatestartlocationnamevalue',
								startlocationname.value,
								targetlocationname.value,
								ridefare,
								ridefareprem
							);
						}, 1000);

						/*
					console.log(summary.totalDistance, summary.totalTime);
					//console.log(a, aa);
					console.log('the routed:');
					console.log(routedist, routedtime); */
						routedvals.value = [routedist, routedtime];
					})
				);
			}

			async function getpositionname(x, y) {
				var geocoding = new require('reverse-geocoding');
				var config = {
					latitude: x,
					longitude: y,
					key: 'af039561477a4dc08203210bd02e06a3',
					url:
						'https://api.opencagedata.com/geocode/v1/json?q=latitude+longitude&key=af039561477a4dc08203210bd02e06a3&pretty=1',
				};
				await new Promise(() =>
					geocoding(config, function(err, data) {
						if (err) {
							console.log(err);
						} else {
							//console.log(data.results[0]);
							console.log(
								data.results[0].components.house_number +
									' ' +
									data.results[0].components.road +
									' ' +
									data.results[0].components.residential +
									' ' +
									data.results[0].components.suburb +
									' ' +
									data.results[0].components.city
							);
							targetlocationname.value = data.results[0].formatted;
							// console.log(data.results[0]);
						}
					})
				);
			}

			async function getstartpositionname(x, y) {
				var geocoding = new require('reverse-geocoding');
				var config = {
					latitude: x,
					longitude: y,
					key: 'af039561477a4dc08203210bd02e06a3',
					url:
						'https://api.opencagedata.com/geocode/v1/json?q=latitude+longitude&key=af039561477a4dc08203210bd02e06a3&pretty=1',
				};
				await new Promise(() =>
					geocoding(config, function(err, data) {
						if (err) {
							console.log(err);
						} else {
							startlocationname.value = data.results[0].formatted;
						}
					})
				);
			}

			onMounted(() => {
				getMapData();
				findloc();
				setTimeout(() => {
					//console.log('ready', curlocation.value);
					startlocation.value = curlocation.value;
					getstartpositionname(startlocation.value[0], startlocation.value[1]);
					console.log(startlocation.value, curlocation.value);

					setTimeout(() => {
						if (
							startlocation.value[0] === curlocation.value[0] &&
							startlocation.value[1] === curlocation.value[1]
						)
							startlocationname.value = 'Current location';
						context.emit(
							'updatestartlocationnamevalue',
							startlocationname.value,
							targetlocationname.value
						);
						firsttime = false;
					}, 800);
				}, 800);

				setInterval(() => {
					findloc();
					setTimeout(() => {
						console.log('calling again', curlocation.value);
						if (firsttime) startlocation.value = curlocation.value;
						getstartpositionname(
							startlocation.value[0],
							startlocation.value[1]
						);
						setTimeout(() => {
							if (
								startlocation.value[0] === curlocation.value[0] &&
								startlocation.value[1] === curlocation.value[1]
							)
								startlocationname.value = 'Current location';
							context.emit(
								'updatestartlocationnamevalue',
								startlocationname.value,
								targetlocationname.value,
								rfare,
								rprem
							);
						}, 800);
					}, 800);
				}, 3000);
				//gettheroute();
			});

			return {
				mymap,
				getLocation,
				showPosition,
				getMapData,
				locmarker,
				gettheroute,
				routedvals,
				curlocation,
				targetlocation,
				routestorage,
				getpositionname,
				targetlocationname,
				startlocationname,
				getstartpositionname,
				message,
				firsttime,
				rfare,
				rprem,
			};
		},
	};
</script>

<style scoped>
	#mapid {
		height: 100vh;
		width: 75vw;
		float: right;
	}

	.leaflet-control-container .leaflet-routing-container-hide {
		display: none;
		width: 0px;
		height: 0px;
	}

	.leaflet-touch .leaflet-control-layers,
	.leaflet-touch .leaflet-bar {
		border: 0px solid rgba(0, 0, 0, 0.2);
		background-clip: padding-box;
	}
</style>
