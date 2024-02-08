<script lang="ts">
 import EmailInput from './emailInput.svelte';
 import Suggestion, { showDialog, closeDialog } from './suggestion.svelte';
 import {format, addMinutes} from 'date-fns';

 type ReservedSlots = {[day: string]: {[time: string]: ReservedSlotInfo;}};
 type Selection = { day: string; time: string } | null;
 type Schedule = { [day: string]: { [time: string]: boolean } };
 type AvailabilityCounts = { [day: string]: { [time: string]: number } };
 type ReservedSlotInfo = {current: number; total: number;};



  let isSelecting = false;
  let startSelection: Selection = null;
  let endSelection: Selection = null;
  let initialSelectionState: boolean = false;
  let availabilityCounts: AvailabilityCounts = {};

  const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
  const times = ['8:00', '9:00', '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', '19:00', '20:00', '21:00', '22:00', '23:00', '24:00'];

  let schedule: { [day: string]: { [time: string]: boolean } } = days.reduce((acc, day) => {
  acc[day] = times.reduce((timeAcc, time) => {
    timeAcc[time] = false; // Initialize each time slot as not selected
    return timeAcc;
  }, {} as { [time: string]: boolean });
  return acc;
}, {} as { [day: string]: { [time: string]: boolean } });

let reservedSlots: ReservedSlots = {
  'Monday': {
    '10:00': { current: 4, total: 5 },
    '11:00': { current: 4, total: 5 },
    '12:00': { current: 4, total: 5 },
    '15:00': { current: 5, total: 5 },
  },
  'Wednesday': {
    '11:00': { current: 5, total: 5 },
  },
};

  function handleMouseDown(event: MouseEvent, day: string, time: string) {
    event.preventDefault();
    isSelecting = true;
    startSelection = { day, time };
    initialSelectionState = !schedule[day][time];
  }
  function handleMouseUp(event: MouseEvent) {
    if (isSelecting) {
      // If the mouse is released without moving, toggle the initial slot
      if (startSelection && !endSelection) {
        toggleTimeSlot(startSelection.day, startSelection.time);
      }
      isSelecting = false;
      startSelection = null;
      endSelection = null;
  }
}

  // Function to toggle the selected state of a time slot
  function toggleTimeSlot(day: string, time: string) {
    if (!schedule[day][time]) { // Check if not reserved or already selected
      schedule[day][time] = !schedule[day][time]; // Toggle the state
    }
    // For reactivity, you may need to update the schedule object in a way that triggers Svelte's reactivity
    schedule = {...schedule};
  }

  function isReserved(day: string, time: string) {
    return reservedSlots[day] && reservedSlots[day][time];
  }

  function handleMouseOver(event: any, day: string, time: string) {
	event.preventDefault();
    if (isSelecting) {
      endSelection = { day, time };
      selectSlot(day, time);
    }
  }

  function updateReservedSlotAvailability(day: string, time: string) {
    const availabilityText = reservedSlots[day][time]; // e.g., "4/5 Available"
    const matches = availabilityText.match(/(\d+)\/(\d+)/);
    if (matches) {
      let [current, total] = matches.slice(1).map(Number); // Extract current and total counts
      current += 1; // Increment current count as another person selects this time
      reservedSlots[day][time] = `${current}/${total + 1} Available`; // Update with new availability
    }
  }

  // Updates the availability count and selection state for a single slot
  function selectSlot(day: string, time: string) {
    if (isReserved(day, time)) {
    // If slot is reserved and being selected, update its availability count
    updateReservedSlotAvailability(day, time);
  } else {
    // Handle non-reserved slot selection
    schedule[day][time] = !schedule[day][time];
    if (schedule[day][time]) {
      availabilityCounts[day][time] = (availabilityCounts[day][time] || 0) + 1;
    } else {
      availabilityCounts[day][time]--;
    }
  }
  schedule = {...schedule}; // Trigger Svelte reactivity
}

  // Updates the availability count and selection state for a range of slots
  // function selectRange(isSelected: boolean) {
  //   if (startSelection && endSelection && startSelection.day === endSelection.day) {
  //     const startIndex = times.indexOf(startSelection.time);
  //     const endIndex = times.indexOf(endSelection.time);
  //     for (let i = Math.min(startIndex, endIndex); i <= Math.max(startIndex, endIndex); i++) {
  //       if (!isReserved(startSelection.day, times[i])) {
  //         schedule[startSelection.day][times[i]] = isSelected;
  //         isSelected ? availabilityCounts[startSelection.day][times[i]]++ : availabilityCounts[startSelection.day][times[i]]--;
  //       }
  //     }
  //     schedule = {...schedule}; // Trigger Svelte reactivity
  //   }
  // }

</script>

<style>

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

<EmailInput />

<Suggestion />

<div class="button-container">
  <button 
  on:click={showDialog}>
    Submit Availability
  </button>
</div>

<div class="table-container">
  <table on:mouseup={handleMouseUp}>
    <thead>
      <tr>
        <th>Time/Day</th>
        {#each days as day}
          <th>{day}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each times as timeString, i}
        <tr>
          <td class="time-column">{timeString}</td>
          {#each days as day}
            <td
              class:selected={schedule[day]?.[timeString]}
              class:reserved={isReserved(day, timeString)}
              class:reserved-selected={isReserved(day, timeString) && schedule[day]?.[timeString]}
              on:mousedown={(event) => handleMouseDown(event, day, timeString)}
              on:mouseover={(event) => handleMouseOver(event, day, timeString)}
              on:mouseup={(event) => toggleTimeSlot(day, timeString)}
            >
            {#if isReserved(day, timeString)}
              {`${reservedSlots[day][timeString].current}/${reservedSlots[day][timeString].total} Available`}
            {/if}
            </td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>


