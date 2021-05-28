<template>
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
				ref="mapComponent"
			></MapClient>
		</div>
	</div>
	<!-- If driver -->
	<div class="driver" v-if="userdata.type === 'D'" style="display: flex;">
		<map-driver></map-driver>

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
				uid: '46A5BC83613C4B2EB2C431CC940C1908',
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
				// this.$children[0].clearTheRoute();
			},

			async getUserID(un) {
				let user_id = null;

				await fetch(`http://localhost:5000/getuserid/${un}`)
					.then((res) => res.json())
					.then((data) => {
						user_id = data;
						this.uid = user_id[0];
						console.log('User ID found', user_id[0], this.uid, Date.now());
					})
					.catch((err) => {
						console.log(err, 'Error!!!');
						alert('Sorry could not find user. Try again');
						location.reload();
					});
			},

			async getUserData() {
				let udata = null;
				console.log('I am here to fetch user data ', Date.now());

				let fetched = await fetch(
					`http://localhost:5000/getuserdata/${this.uid}`
				)
					.then((res) => res.json())
					.then((data) => {
						udata = data[0];
						console.log('Fetched userdata', udata[0]);
					})
					.catch((err) => console.log(err, 'Error!!!'));

				setTimeout(() => {
					// console.log(udata);

					this.userdata = {
						uid: udata[0],
						name: udata[1] + ' ' + udata[2],
						phone: udata[3],
						dob: udata[4],
						age: udata[5],
						type: udata[6],
						username: udata[7],
					};

					//.log(this.userdata);
					this.loaded = true;
				}, 2800);
			},
		},

		mounted() {
			var promp = prompt('Please enter your username', '');
			this.getUserID(promp);

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
