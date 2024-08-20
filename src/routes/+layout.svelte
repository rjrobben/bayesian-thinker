<nav>
	<a href="/">Home</a>
	<a href="/calculator">Calculator</a>
    <!-- <a href="/individual_events">Individual Events</a> -->
    <a href="/combined_events">Combined Events</a>
    <!-- <a href="/events_after">After Events</a> -->
	<a href="/settings">Settings</a>
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