<script>
    let exportedData = '';
    let importData = '';

    function exportToJson() {
        const bayesianEvents = localStorage.getItem('bayesianEvents') || '[]';
        const bayesianLogs = localStorage.getItem('bayesianLogs') || '[]';

        const exportData = {
            events: JSON.parse(bayesianEvents),
            logs: JSON.parse(bayesianLogs)
        };

        exportedData = JSON.stringify(exportData, null, 2);
    }

    function importFromJson() {
        try {
            const importedData = JSON.parse(importData);
            
            if (importedData.events && importedData.logs) {
                localStorage.setItem('bayesianEvents', JSON.stringify(importedData.events));
                localStorage.setItem('bayesianLogs', JSON.stringify(importedData.logs));
                alert('Data imported successfully!');
            } else {
                throw new Error('Invalid data format');
            }
        } catch (error) {
            alert('Error importing data. Please check the JSON format.');
        }
    }
</script>

<main>
    <div class="container">
        <h1>Settings</h1>
        
        <section>
            <h2>Export Data</h2>
            <button on:click={exportToJson}>Export to JSON</button>
            {#if exportedData}
                <textarea rows="10" readonly>{exportedData}</textarea>
            {/if}
        </section>

        <section>
            <h2>Import Data</h2>
            <textarea bind:value={importData} rows="10" placeholder="Paste JSON data here"></textarea>
            <button on:click={importFromJson}>Import from JSON</button>
        </section>
    </div>
</main>

<style>
    .container {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
    }

    textarea {
        width: 100%;
        margin-top: 10px;
    }

    button {
        margin-top: 10px;
    }

    section {
        margin-bottom: 30px;
    }
</style>