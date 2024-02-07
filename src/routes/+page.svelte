<script lang="ts">
  let name = '';
  let eventName = '';
  let email = '';
  let showNameInput = true;
  let showEventNameInput = true;
  let schedule = {}; // A schedule object to keep track of selected slots
  let isSelecting = false;
  let startSelection = null;
  let endSelection = null;
  const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'];
  const times = ['9:00', '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', '19:00', '20:00', '21:00', '22:00', '23:00'];

  let initialSelectionState = false;

  function handleMouseDown(day, time) {
	event.preventDefault();
    isSelecting = true;
    startSelection = { day, time };
    // Record the initial state when selection starts
    initialSelectionState = !schedule[day][time]; // toggle the state
    selectSlot(day, time, initialSelectionState);
  }

  function handleMouseOver(day, time) {
	event.preventDefault();
    if (isSelecting) {
      endSelection = { day, time };
      selectRange(initialSelectionState);
    }
  }

  function selectSlot(day, time, isSelected) {
    if (!schedule[day]) {
      schedule[day] = {};
    }
    schedule[day][time] = isSelected;
  }

  function selectRange(isSelected) {
    // Logic to select or deselect all slots between startSelection and endSelection
    // This is a simplified example and will only work if selecting within a single day
    if (startSelection && endSelection && startSelection.day === endSelection.day) {
      const startIndex = times.indexOf(startSelection.time);
      const endIndex = times.indexOf(endSelection.time);
      for (let i = Math.min(startIndex, endIndex); i <= Math.max(startIndex, endIndex); i++) {
        schedule[startSelection.day][times[i]] = isSelected;
      }
    }
  }


  // Initialize the schedule object
  days.forEach(day => {
    schedule[day] = times.reduce((acc, time) => {
      acc[time] = false;
      return acc;
    }, {});
  });


  function handleMouseUp() {
    isSelecting = false;
    // Reset the selection range
    startSelection = null;
    endSelection = null;
  }

  function add(target, type) {
    if (type === 'name') {
      name = target.value;
      showNameInput = false;
    } else if (type === 'eventName') {
      eventName = target.value;
      showEventNameInput = false;
    }
  }
</script>

<style>
  .new-todo {
	font-size: 1.4em;
	width: 30%;
	margin: 2em 0 1em 0;
  }

  input {
	margin: 10px;
  }

  .selected {
    background-color: lightblue;
  }
  td {
    cursor: pointer;
    padding: 5px;
    border: 1px solid #ccc;
    text-align: center;
	height: 2em;
  }
  th {
    padding: 5px;
    border: 1px solid #ccc;
    text-align: center;
	height: 2em;
  }

  table {
    width: 100%; /* full width of the container */
    table-layout: fixed; /* ensures that all columns are of equal width */
    border-collapse: collapse; /* optional, for collapsed borders */
  }

    button {
    padding: 10px 20px;
    margin: 5px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  button:hover {
    background-color: #0056b3;
  }

  .input-container {
    display: flex;
    justify-content: space-between; /* This will justify the content */
    align-items: center; /* This will vertically align items if they have different heights */
    padding: 0.5em;
  }

  .input-field, .input-text {
    flex: 1; /* This will make each item take up equal space */
    margin: 0 0.5em; /* This adds some space between the items */
    /* Additional styles for your inputs and text */
  }

  .input-field {
    padding: 0.5em;
    /* Other input styles */
  }

  .input-text {
    /* Styles for the text after input is entered */
  }

</style>

<div>

<div class="input-container">
  {#if showNameInput}
    <input
      class="input-field"
      placeholder="Name"
      on:keydown={(event) => event.key === 'Enter' && add(event.target, 'name')}
    />
  {:else}
    <p class="input-text">Hi! {name}</p>
  {/if}

  {#if showEventNameInput}
    <input
      class="input-field"
      placeholder="Event Name"
      on:keydown={(event) => event.key === 'Enter' && add(event.target, 'eventName')}
    />
  {:else}
    <p class="input-text">{eventName}</p>
  {/if}
</div>


<table on:mouseup={handleMouseUp} on:mouseleave={handleMouseLeave}>
  <thead>
    <tr>
      <th>Time/Day</th>
      {#each days as day}
        <th>{day}</th>
      {/each}
    </tr>
  </thead>
  <tbody>
    {#each times as time, i}
      <tr>
        <td class="time-column">{time}</td>
        {#each days as day}
          <td
            class:selected={schedule[day]?.[time]}
            on:mousedown={() => handleMouseDown(day, time)}
            on:mouseover={() => handleMouseOver(day, time)}
            on:mouseup={handleMouseUp}
          >
          </td>
        {/each}
      </tr>
    {/each}
  </tbody>
</table>

<button on:click={handleClick}>
  Submit Availability
</button>

</div>
