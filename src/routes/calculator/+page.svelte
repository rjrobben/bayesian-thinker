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

	let hypothesis= '';
	let evidence = '';
	let priorA = '';
	let probBGivenA = '';
	let probBGivenNotA = '';
	let posteriorA = '';
	let reasoning = '';
	let result = '';
	let eventId = '';
	let events = [];
	let logs = [];
	let selectedEventLogs = [];
	let selectedEvent = null;

	onMount(() => {
		loadEvents();
	});

	function loadEvents() {
		const storedEvents = JSON.parse(localStorage.getItem('bayesianEvents') || '[]');
		const storedLogs = JSON.parse(localStorage.getItem('bayesianLogs') || '[]');
		events = storedEvents;
		logs = storedLogs;
	}

	// function handleEventSelection(event) {
	//     selectedEvent = event ? JSON.parse(event.target.value) : null;
	//     if (selectedEvent) {
	//         eventId = selectedEvent.id;
	//         eventA = selectedEvent.eventA;
	//         eventB = selectedEvent.eventB;
	//     } else {
	//         resetForm();
	//     }
	// }
	function handleEventSelection(event) {
		const selectedValue = event.target.value;
		if (selectedValue === '') {
			// User selected "Create New Event"
			resetForm();
			selectedEvent = null;
		} else {
			selectedEvent = JSON.parse(selectedValue);
			eventId = selectedEvent.id;
			hypothesis = selectedEvent.hypothesis;
			evidence = selectedEvent.evidence;

        // Find the latest log for this event
        const latestLog = logs
            .filter(log => log.eventId === selectedEvent.id)
            .sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp))[0];

        // Use the posteriorA from the latest log if available, otherwise leave it empty
        priorA = latestLog ? latestLog.posteriorA.toString() : '';
			// Clear other fields
			probBGivenA = '';
			probBGivenNotA = '';
			posteriorA = '';
			reasoning = '';
		}
	}

	function resetForm() {
		eventId = '';
		hypothesis = '';
		evidence = '';
		priorA = '';
		probBGivenA = '';
		probBGivenNotA = '';
		posteriorA = '';
		reasoning = '';
	}
	function saveEvent() {
		if (selectedEvent) {
			// Update existing event
			const index = events.findIndex((e) => e.id === selectedEvent.id);
			if (index !== -1) {
				events[index] = { ...events[index], hypothesis, evidence };
			}
		} else {
			// Create new event
			const newEvent = {
				id: Date.now().toString(),
				hypothesis,
				evidence
			};
			events = [...events, newEvent];
		}

		calculatePosterior();

		// Create a new log entry
		const log = {
			eventId: selectedEvent ? selectedEvent.id : events[events.length - 1].id,
			priorA: parseFloat(priorA),
			probBGivenA: parseFloat(probBGivenA),
			probBGivenNotA: parseFloat(probBGivenNotA),
			posteriorA: parseFloat(posteriorA),
			reasoning,
			timestamp: new Date().toISOString()
		};

		logs = [...logs, log];

		localStorage.setItem('bayesianEvents', JSON.stringify(events));
		localStorage.setItem('bayesianLogs', JSON.stringify(logs));

		loadEvents();
		resetForm();
		selectedEvent = null;
	}

	function calculatePosterior() {
		const priorAValue = parseFloat(priorA);
		const probBGivenAValue = parseFloat(probBGivenA);
		const probBGivenNotAValue = parseFloat(probBGivenNotA);

		if (
			[priorAValue, probBGivenAValue, probBGivenNotAValue].some(isNaN) ||
			[priorAValue, probBGivenAValue, probBGivenNotAValue].some((p) => p < 0 || p > 1)
		) {
			alert('Please enter valid probabilities between 0 and 1.');
			return;
		}

		const priorNotA = 1 - priorAValue;
		const probB = probBGivenAValue * priorAValue + probBGivenNotAValue * priorNotA;
		const posterior = (probBGivenAValue * priorAValue) / probB;
		//  set posteriorA,
		posteriorA = posterior.toFixed(4);

		result = `
        <p>P(A|B) - Posterior Probability: ${posterior.toFixed(4)}</p>
        <p>Hypothesis: ${hypothesis}</p>
        <p>Evidence: ${evidence}</p>
        <p>Reasoning: ${reasoning}</p>
      `;
	}
</script>

<main>
	<div class="container">
		<h1>Bayesian Thinking</h1>
		<div class="input-group">
			<label for="eventSelect">Select Event:</label>
			<select id="eventSelect" on:change={handleEventSelection}>
				<option value="">Create New Event</option>
				{#each events as event}
					<option value={JSON.stringify(event)}>{event.hypothesis} | {event.evidence}</option>
				{/each}
			</select>
		</div>
		<div class="input-group">
			<label for="hypothesis">Hypothesis:</label>
			<textarea id="hypothesis" bind:value={hypothesis} placeholder="Describe Hypothesis"></textarea>
		</div>
		<div class="input-group">
			<label for="evidence">Evidence:</label>
			<textarea id="evidence" bind:value={evidence} placeholder="Describe Evidence"></textarea>
		</div>
		<div class="input-group">
			<label for="priorA">P(H) - Prior Probability:</label>
			<input
				type="number"
				id="priorA"
				bind:value={priorA}
				min="0"
				max="1"
				step="0.01"
				placeholder="Enter P(H) (0-1)"
			/>
		</div>
		<div class="input-group">
			<label for="probBGivenA">P(E|H) - Probability of Evidence appearing given Hypothesis:</label>
			<input
				type="number"
				id="probBGivenA"
				bind:value={probBGivenA}
				min="0"
				max="1"
				step="0.01"
				placeholder="Enter P(E|H) (0-1)"
			/>
		</div>
		<div class="input-group">
			<label for="probBGivenNotA">P(E|not H) - Probability of Evidence given Hypothesis is False:</label>
			<input
				type="number"
				id="probBGivenNotA"
				bind:value={probBGivenNotA}
				min="0"
				max="1"
				step="0.01"
				placeholder="Enter P(E|not H) (0-1)"
			/>
		</div>
		<div class="input-group">
			<label for="reasoning">Reasoning and Notes:</label>
			<textarea
				id="reasoning"
				bind:value={reasoning}
				placeholder="Document your reasoning and how you estimated these probabilities"
			></textarea>
		</div>
		<button on:click={saveEvent}>Save and Calculate</button>
		<div id="result">
			{@html result}
		</div>
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
	.input-group {
		margin-bottom: 20px;
	}
	label {
		display: block;
		margin-bottom: 5px;
		font-weight: bold;
		color: #34495e;
	}
	input[type='number'],
	textarea {
		width: 100%;
		padding: 10px;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 16px;
		transition: border-color 0.3s;
	}
	input[type='number']:focus,
	textarea:focus {
		border-color: #3498db;
		outline: none;
	}
	textarea {
		height: 80px;
		resize: vertical;
	}
	button {
		display: block;
		width: 100%;
		padding: 12px;
		background: #3498db;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-size: 16px;
		transition: background-color 0.3s;
	}
	button:hover {
		background: #2980b9;
	}
	#result {
		margin-top: 20px;
		font-weight: bold;
		text-align: center;
	}
</style>
