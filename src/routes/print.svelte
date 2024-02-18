<script>
    export let data = [];
</script>
    
{#each data as row}
    <div>
        {Object.entries(row).map(([key, value]) => `${key}: ${value}`).join(', ')}
    </div>
{/each}
    