<template>
	<!-- The menu container -->
	<div class="menu">
		<!-- Navigation bar -->
		<div class="navcontainer">
			<img src="/assets\uberlogo.png" alt="" srcset="" id="uberlogo" />
			<img src="/assets\settings.png" alt="" srcset="" id="settingsicon" />
		</div>

		<!-- Header -->
		<div class="welcome">
			<h2>Welcome,<br />{{ user.name }}!</h2>

			<p>{{ welcomep1 }}</p>
			<p>{{ welcomep2 }}</p>

			<!-- If the user is a driver, show the GO button to start looking for trips -->
			<div v-if="type === 'D'" class="driverbtn">
				<button>GO</button>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		props: {
			user: {},
			welcomep1: '', //Text to show if user is Client
			welcomep2: '', //Text to show if user is Driver
			type: '', //Type of User
		},
	};
</script>

<style scoped>
	.menu {
		width: 24vw;
		height: 100vh;
		margin: 0;
		float: left;

		-webkit-user-select: none;
		-moz-user-select: none;
		-ms-user-select: none;
		user-select: none;
	}
	.menu img#uberlogo {
		width: 128px;
		height: auto;
		float: left;
		position: relative;
		top: 0%;
	}
	.menu img#settingsicon {
		width: 28px;
		height: auto;
		float: right;
		position: relative;
		top: 0%;
	}
	.navcontainer {
		margin-top: 45px;
		margin-left: 13%;
		margin-right: 13%;
		height: 80px;
	}

	h2 {
		margin-bottom: 40px;
		color: #eee;
		font-size: 2rem;
		-webkit-user-select: none; /* Safari */
		-moz-user-select: none; /* Firefox */
		-ms-user-select: none; /* IE10+/Edge */
		user-select: none; /* Standard */
	}

	.welcome {
		margin-top: 10%;
	}

	.driverbtn {
		margin-top: 8em;
	}

	.driverbtn button {
		border: none;
		box-shadow: 0 0 0 5px rgb(28, 28, 28), 0 0 0 8px #a29bfe;
		border-radius: 50%;
		width: 100px;
		height: 100px;
		background: #6c5ce7;
		color: #eee;
		padding: 0;
		font-size: 1.3rem;
		font-weight: bold;
		animation: pulsebutton 1.3s infinite ease-in-out;
		animation-delay: 3s;
		cursor: pointer;
	}
	.driverbtn button:hover {
		opacity: 0.69;
		transform: scale(1.008);
	}

	@keyframes pulsebutton {
		0% {
			-webkit-transform: scale(0);
			transform: scale(0.69);
			opacity: 0.38;
		}
		75% {
			-webkit-transform: scale(1);
			transform: scale(1);
			opacity: 1;
		}
		100% {
			-webkit-transform: scale(1);
			transform: scale(1);
			opacity: 1;
		}
	}
</style>
