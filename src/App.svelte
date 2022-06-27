<script lang="ts">
	import { Router, Link, Route } from "svelte-routing";
	import Layout from "./__layout.svelte";
	import Home from "./Home.svelte";
	import Login from "./Login.svelte";
	import Groups from "./Groups.svelte";
</script>

<main>
	<Layout />

	<Router>
		<nav>
			<Link to="/login">Log in</Link>
			<Link to="/">Home</Link>
		</nav>
			<Route path="groups/:id" component={Groups}></Route>
			<Route path="/login" component={Login}></Route>
			<Route path="/" component={Home}></Route>
	</Router>
</main>

<style>
</style>
