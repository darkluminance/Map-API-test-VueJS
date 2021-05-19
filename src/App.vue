<template>
	<div class="appcontainer">
		<Menubar
			:sl="this.sloc"
			:el="this.tloc"
			:sp="this.rfarestan"
			:pp="this.rfareprem"
		></Menubar>
		<Map @updatestartlocationnamevalue="getname"></Map>
	</div>
</template>

<script>
	import Map from './components/Map.vue';
	import Menubar from './components/Menubar.vue';

	export default {
		/* props: {
			startlocname: String,
		}, */
		name: 'App',
		components: {
			Map,
			Menubar,
		},
		data() {
			return {
				sloc: '',
				tloc: '',
				rfarestan: 0,
				rfareprem: 0,
			};
		},

		methods: {
			getname(value, value2, f1, f2) {
				this.sloc = value;
				this.tloc = value2;
				this.rfarestan = f1;
				this.rfareprem = f2;
			},
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
</style>
