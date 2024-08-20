<script>
	// individual event
	//   - eventName
	//   - eventId
	// events happened after individual event
	//   - individual event id
	//   - eventName
	//   - description
	// combined event
	//   - individual event a id
	//   - individual event b id
	// event log
	//   - combined event id
	//   - ...
	import { onMount } from 'svelte';

	let hypothesis = '';
	let evidence = '';
	let priorA = '';
	let probBGivenA = '';
	let probBGivenNotA = '';
	let posteriorA = '';
	let reasoning = '';
	let eventId = '';
	let events = [];
	let logs = [];
	let selectedEventLogs = [];

	onMount(() => {
		loadEvents();
	});
	function loadEvents() {
		const storedEvents = JSON.parse(localStorage.getItem('bayesianEvents') || '[]');
		const storedLogs = JSON.parse(localStorage.getItem('bayesianLogs') || '[]');
		events = storedEvents;
		logs = storedLogs;
	}

	function loadEventLogs(id) {
		const event = events.find((e) => e.id === id);
		if (event) {
			eventId = id;
			hypothesis = event.hypothesis;
			evidence = event.evidence;
			selectedEventLogs = logs.filter((log) => log.eventId === id);
			if (selectedEventLogs.length > 0) {
				const latestLog = selectedEventLogs[selectedEventLogs.length - 1];
				priorA = latestLog.priorA;
				probBGivenA = latestLog.probBGivenA;
				probBGivenNotA = latestLog.probBGivenNotA;
				posteriorA = latestLog.posteriorA;
				reasoning = latestLog.reasoning;
			}
		}
	}
	function deleteLog(logTimestamp) {
		if (confirm('Are you sure you want to delete this log entry?')) {
			logs = logs.filter((log) => log.timestamp !== logTimestamp);
			selectedEventLogs = selectedEventLogs.filter((log) => log.timestamp !== logTimestamp);
			localStorage.setItem('bayesianLogs', JSON.stringify(logs));
		}
	}
	function deleteCombinedEvent(id) {
		if (confirm('Are you sure you want to delete this event and all its logs?')) {
			events = events.filter((event) => event.id !== id);
			logs = logs.filter((log) => log.eventId !== id);
			localStorage.setItem('bayesianEvents', JSON.stringify(events));
			localStorage.setItem('bayesianLogs', JSON.stringify(logs));

			// Reset current selection if the deleted event was selected
			if (eventId === id) {
				eventId = '';
				hypothesis = '';
				evidence = '';
				priorA = '';
				probBGivenA = '';
				probBGivenNotA = '';
				posteriorA = '';
				reasoning = '';
				selectedEventLogs = [];
			}
		}
	}
</script>

<main>
	<div class="container">
		<h1>Bayesian Thinking</h1>
		<h2>Saved Events</h2>
		<ul class="event-list">
			{#each events as event}
				<li>
					<div class="event-item">
						<button class="event-btn" on:click={() => loadEventLogs(event.id)}>
							{event.hypothesis} | {event.evidence} ({logs.filter((log) => log.eventId === event.id)
								.length}
							logs)
						</button>
						<button on:click={() => deleteCombinedEvent(event.id)} class="delete-btn"
							>Delete Event</button
						>
					</div>
				</li>
			{/each}
		</ul>

		{#if selectedEventLogs.length > 0}
			<h2>Event Logs</h2>
			<div class="table-container">
				<table>
					<thead>
						<tr>
							<th>Event ID</th>
							<th>Timestamp</th>
							<th>P(H)</th>
							<th>P(E|H)</th>
							<th>P(E|not H)</th>
							<th>Posteior Probability</th>
							<th>Reasoning</th>
							<th>Action</th>
						</tr>
					</thead>
					<tbody>
						{#each selectedEventLogs as log}
							<tr>
								<td>{log.eventId}</td>
								<td>{new Date(log.timestamp).toLocaleString()}</td>
								<td>{log.priorA}</td>
								<td>{log.probBGivenA}</td>
								<td>{log.probBGivenNotA}</td>
								<td>{log.posteriorA}</td>
								<td>{log.reasoning}</td>
								<td>
									<button on:click={() => deleteLog(log.timestamp)} class="delete-btn"
										>Delete</button
									>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		{/if}
	</div>
</main>

<style>
	:global(body) {
		font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
		line-height: 1.6;
		margin: 0;
		padding: 20px;
		background-color: #f0f2f5;
		color: #333;
	}
	.container {
		max-width: 800px;
		margin: auto;
		background: white;
		padding: 30px;
		border-radius: 8px;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	}
	h1,
	h2 {
		text-align: center;
		color: #2c3e50;
		margin-bottom: 20px;
	}
	.event-list {
		list-style-type: none;
		padding: 0;
	}
	.event-list li {
		margin-bottom: 10px;
	}
	/* .event-list button {
		width: 100%;
		text-align: left;
		padding: 10px;
		background-color: #ecf0f1;
		color: #2c3e50;
		border: 1px solid #bdc3c7;
	}
	.event-list button:hover {
		background-color: #d5dbdb;
	} */
	.table-container {
		overflow-x: auto;
	}
	table {
		width: 100%;
		border-collapse: collapse;
		margin-top: 20px;
	}
	th,
	td {
		padding: 12px;
		text-align: left;
		border-bottom: 1px solid #ddd;
	}
	th {
		background-color: #f2f2f2;
		font-weight: bold;
	}
	tr:hover {
		background-color: #f5f5f5;
	}
	.event-item {
		display: flex;
		gap: 10px;
	}

	.event-btn {
		flex-grow: 1;
		text-align: left;
		padding: 10px;
		background-color: #ecf0f1;
		color: #2c3e50;
		border: 1px solid #bdc3c7;
		border-radius: 4px;
		cursor: pointer;
		font-size: 16px;
		transition: background-color 0.3s;
	}
	.event-btn:hover {
		background-color: #d5dbdb;
	}
	.delete-btn {
		background-color: #ff4136;
		color: white;
		border: none;
		padding: 5px 10px;
		cursor: pointer;
		border-radius: 4px;
		font-size: 14px;
		transition: background-color 0.3s;
	}
	.delete-btn:hover {
		background-color: #ff1a1a;
	}
</style>
