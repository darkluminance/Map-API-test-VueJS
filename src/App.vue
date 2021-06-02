<template>
	<!-- The root vue file -->

	<!-- If client -->
	<div v-if="userdata.type === 'C'" class="Client">
		<div v-if="loaded" class="appcontainer">
			<WelcomeMenu
				:user="this.userdata"
				:welcomep1="'Click on any part of the map'"
				:welcomep2="'to set the destination'"
				:type="'C'"
				v-if="!isRouteSearching"
			></WelcomeMenu>

			<Menubar
				:sl="this.sloc"
				:el="this.tloc"
				:sp="this.rfarestan"
				:pp="this.rfareprem"
				@resetRoutes="clearRoutes"
				v-if="isRouteSearching"
			></Menubar>

			<MapClient
				@updatestartlocationnamevalue="getname"
				@routesearched="isRouteSearching = true"
				@updatelocation="updateuserlocation"
				ref="mapComponent"
			></MapClient>
		</div>
	</div>

	<!-- If driver -->
	<div class="driver" v-if="userdata.type === 'D'" style="display: flex;">
		<map-driver @updatelocation="updateuserlocation"></map-driver>

		<welcome-menu
			:user="this.userdata"
			:welcomep1="'Click on the GO button'"
			:welcomep2="'to start searching for trips'"
			:type="'D'"
		></welcome-menu>
	</div>

	<!-- Loading screen -->
	<div v-if="!loaded" class="loading">
		<div>
			<img src="/assets/uberlogob.png" alt="" srcset="" style="width: 280px;" />
		</div>
		<CircleSpinner style="height: 48px;"></CircleSpinner>
		<div style="margin: 0; margin-bottom: 96px;"></div>
		<h2>
			Logging in...
		</h2>
	</div>
</template>

<script>
	import WelcomeMenu from './components/WelcomeMenu.vue';
	import Menubar from './components/Menubar.vue';
	import MapClient from './components/MapClient.vue';
	import MapDriver from './components/MapDriver.vue';
	import CircleSpinner from './components/Animations/CircleSpinner';

	export default {
		/* props: {
			startlocname: String,
		}, */
		name: 'App',
		components: {
			WelcomeMenu,
			Menubar,
			MapClient,
			MapDriver,
			CircleSpinner,
		},
		data() {
			return {
				sloc: '',
				tloc: '',
				rfarestan: 0,
				rfareprem: 0,
				isRouteSearching: false,
				uid: '',
				userdata: {},
				loaded: false,
			};
		},

		methods: {
			getname(value, value2, f1, f2) {
				this.sloc = value;
				this.tloc = value2;
				this.rfarestan = f1;
				this.rfareprem = f2;
			},

			clearRoutes() {
				this.$refs.mapComponent.clearTheRoute();
				this.isRouteSearching = false;
			},

			async getUserID(un) {
				let user_id = null;

				await fetch(`http://localhost:5000/getuserid/${un}`)
					.then((res) => res.json())
					.then((data) => {
						user_id = data;
						this.uid = user_id[0];
					})
					.catch((err) => {
						console.log(err, 'Error!!!');
						alert('Sorry could not find user. Try again');
						location.reload();
					});
			},

			async updateuserlocation(loc) {
				var sendingdata = {};

				if (this.userdata.type === 'C') {
					sendingdata = {
						u_id: this.userdata.uid,
						user_typ: 'C',
						C_Location_X: loc[0],
						C_Location_Y: loc[1],
					};
				} else if (this.userdata.type === 'D') {
					sendingdata = {
						u_id: this.userdata.uid,
						user_typ: 'D',
						DR_Location_X: loc[0],
						DR_Location_Y: loc[1],
					};
				}

				// console.log('to send', sendingdata);

				fetch('http://localhost:5000/updateuserlocation', {
					method: 'POST',
					headers: {
						'Content-Type': 'application/json',
					},
					body: JSON.stringify(sendingdata),
				})
					.then((response) => response.json())
					.then((data) => {
						// console.log('Success:', data);
					})
					.catch((error) => {
						console.error('Error:', error);
						alert('An error occured. Please reload the page again');
					});
			},

			async getUserData() {
				let udata = null;

				let fetched = await fetch(
					`http://localhost:5000/getuserdata/${this.uid}`
				)
					.then((res) => res.json())
					.then((data) => {
						udata = data[0];
					})
					.catch((err) => console.log(err, 'Error!!!'));

				setTimeout(() => {
					this.userdata = {
						uid: udata[0],
						name: udata[1] + ' ' + udata[2],
						phone: udata[3],
						dob: udata[4],
						age: udata[5],
						type: udata[6],
						username: udata[7],
					};

					//Data is loaded
					this.loaded = true;
				}, 2800);
			},
		},

		mounted() {
			//WHen loaded, asks the user to enter the username. Then find the user id of the username
			var promp = prompt('Please enter your username', '');
			this.getUserID(promp);

			//Now after 800ms search the database for the user data of that user id
			setTimeout(() => {
				this.getUserData();
			}, 800);
		},
	};
</script>

<style>
	#app {
		/* font-family: Avenir, Helvetica, Arial, sans-serif; */
		/* font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; */
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
			Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;

		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #eee;
		/* margin-top: 60px; */
	}
	body {
		margin: 0;
		background: rgb(28, 28, 28);
	}
	.appcontainer {
		width: 100vw;
		height: 100vh;
		display: flex;
	}
	.loading {
		background: #eee;
		width: 100%;
		height: 100%;
		z-index: 999;
		position: fixed;
		top: 0%;
		left: 0%;
		color: rgb(28, 28, 28);
		font-size: 1.18rem;

		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}
	.loading h2 {
		font-size: 2rem;
		font-style: bold;
	}
</style>
