<script>
    import { base } from '$app/paths';
</script>

<nav>
	<a href="{base}/">Home</a>
	<a href="{base}/calculator">Calculator</a>
    <!-- <a href="/individual_events">Individual Events</a> -->
    <a href="{base}/combined_events">Combined Events</a>
    <!-- <a href="/events_after">After Events</a> -->
	<a href="{base}/settings">Settings</a>
</nav>

<slot></slot>

<style>
	nav {
		background-color: #333;
		padding: 1rem;
		display: flex;
		justify-content: space-around;
	}

	a {
		color: white;
		text-decoration: none;
		padding: 0.5rem 1rem;
		border-radius: 4px;
		transition: background-color 0.3s ease;
	}

	a:hover {
		background-color: #555;
	}
</style>