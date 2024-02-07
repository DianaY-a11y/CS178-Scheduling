<script lang="ts">
  let name = '';
  let eventName = '';
  let email = '';
  let message = 'Scheduling Failed. 4/5 participants are available Tuesday 2:00pm - 3:00pm. Can you adjust?';
  let editingName = true;
  let editingEventName = true;
  let showNameInput = true;
  let showEventNameInput = true;
  let isSelecting = false;
  let startSelection = null;
  let endSelection = null;
  let meetingFinalized = false;
  const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
  const times = ['8:00', '9:00', '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', '19:00', '20:00', '21:00', '22:00', '23:00', '24:00'];

  // Preset reserved slots with labels
  let reservedSlots = {
    'Monday': {'10:00': '4/5 Available', '11:00': '4/5 Available', '12:00': '4/5 Available', '15:00': '5/5 Available'},
    'Wednesday': {'11:00': '5/5 Available'},
    // Define more as needed
  };

  let schedule = days.reduce((acc, day) => {
    acc[day] = times.map(() => false);
    return acc;
  }, {});


  let initialSelectionState = false;

  function handleMouseDown(day, time) {
	event.preventDefault();
    isSelecting = true;
    startSelection = { day, time };
    initialSelectionState = !schedule[day][time]; // toggle the state
    selectSlot(day, time, initialSelectionState);
  }

  function isReserved(day, time) {
    return reservedSlots[day] && reservedSlots[day][time];
  }

  function handleMouseOver(day, time) {
	event.preventDefault();
    if (isSelecting) {
      endSelection = { day, time };
      selectRange(initialSelectionState);
    }
  }

  function selectSlot(day, time) {
    let dayIndex = days.indexOf(day);
    let timeIndex = times.indexOf(time);
    schedule[day][timeIndex] = !schedule[day][timeIndex];
	
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

  function handleMouseUp() {
    isSelecting = false;
    // Reset the selection range
    startSelection = null;
    endSelection = null;
  }

    // Function to handle double-click for name
  function handleNameDblClick() {
    editingName = true;
    // Focus on the input element after it has been rendered
    setTimeout(() => {
      document.getElementById('name-input').focus();
    });
  }

    // Function to handle double-click for name
  function handleEventNameDblClick() {
    editingEventName = true;
    // Focus on the input element after it has been rendered
    setTimeout(() => {
      document.getElementById('event-name-input').focus();
    });
  }

  // Function to save the name and stop editing on Enter key press
  function saveNameInput(event) {
    if (event.key === 'Enter') {
      name = event.target.value;
      editingName = false;
      console.log("saved")
    }
  }

  // Function to save the name and stop editing on Enter key press
  function saveEventNameInput(event) {
    if (event.key === 'Enter') {
      eventName = event.target.value;
      editingEventName = false;
      console.log("saved")
    }
  }

  function showDialog() {
    document.getElementById('dialog').showModal();
    selectRandomSlot()
    console.log(selectedMeetingTime)
  }

  function closeDialog() {
    document.getElementById('dialog').close();
  }

  let selectedMeetingTime = '';

  // Function to get all available slots
  function getAvailableSlots() {
    console.log("get slots")
      const availableSlots = [];
    console.log("get avaialble 1")
      for (const day of days) {
        for (const [index, time] of times.entries()) {
          if (schedule[day][index]) {
            availableSlots.push({ day, time });
        console.log("get avaialble 1")
          }
        }
      }
      return availableSlots;
  }

  // Function to select a random slot
  function selectRandomSlot() {
    console.log('hello')
    const availableSlots = getAvailableSlots();
    console.log(availableSlots, "available slots");
    if (availableSlots.length > 0) {
      const randomIndex = Math.floor(Math.random() * availableSlots.length);
      const slot = availableSlots[randomIndex];
      selectedMeetingTime = `Meeting from ${slot.time} on ${slot.day} has been chosen`;
    } else {
      selectedMeetingTime = 'No available slots to select.';
    }
    // Force Svelte to recognize the update
    selectedMeetingTime = selectedMeetingTime + '';
  }

  let count = 0;
  function inc() {
    count++;
  }

  // Function to adjust the meeting based on user's choice
  function adjustMeeting(adjust) {
    if (adjust) {
      message = "Meeting scheduled ✅.";
    } else {
      message = "Schedule failed ❌.";
    }
	meetingFinalized = true;
  }


</script>

<style>
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
    width: 100%; 
    table-layout: fixed; 
    border-collapse: collapse;
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
    justify-content: space-between; 
    align-items: center; 
    padding: 0.5em;
  }

  .input-field, .input-text {
    flex: 1; 
    margin: 0 0.5em; 
  }

  .input-field {
    padding: 0.5em;
  }

  .button-container {
    margin: 10px;
    position: relative;
  }

  .reserved {
    background-color: #f0e68c;
    color: black; 
  }

  .reserved-selected {
    background-color: green; 
  }
</style>

<h1>CS178 Meeting</h1>

<div class="input-container">
  {#if editingName}
    <input
      id="name-input"
      type="text"
	  value={name}
      on:blur={() => editingName = false}
      on:keydown={event => saveNameInput(event)}
      placeholder="Enter your name"
    />
  {:else}
    <span on:dblclick={handleNameDblClick}>{name || 'Enter your name'}</span> <!-- Fallback text if name is empty -->
  {/if}

</div>

<div class="button-container">
  <button 
  on:click={showDialog}>
    Submit Availability
  </button>
</div>

<dialog
 id="dialog">
  <button on:click={closeDialog}>Close</button>
  <p>{message}</p>
  {#if !meetingFinalized}
    <button on:click={() => adjustMeeting(true)}>Yes</button>
    <button on:click={() => adjustMeeting(false)}>No</button>
  {/if}

</dialog>

<div class="table-container">
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
			  class:reserved={isReserved(day, time)}
			  class:reserved-selected={isReserved(day, time) && schedule[day]?.[time]}
              on:mousedown={() => handleMouseDown(day, time)}
              on:mouseover={() => handleMouseOver(day, time)}
              on:mouseup={handleMouseUp}
            >
			{#if isReserved(day, time)}
                {reservedSlots[day][time]}
              {/if}
            </td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>

