<script lang="ts">
 import EmailInput from './emailInput.svelte';
 import {format, addMinutes} from 'date-fns';


 // Defining data types 
 type ReservedSlots = {[day: string]: {[time: string]: ReservedSlotInfo;}};
 type Selection = { day: string; time: string } | null;
 type AvailabilityCounts = { [day: string]: { [time: string]: number } };
 type ReservedSlotInfo = {current: number; total: number;};


  let day, time, overlap: boolean;
  let message = '';

  let isSelecting = false;
  let startSelection: Selection = null;
  let endSelection: Selection = null;
  let initialSelectionState: boolean = false;
  let availabilityCounts: AvailabilityCounts = {};
  let numberSubmitted = 5;
  
  // Setting calendar 
  const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
  const times = ['8:00', '9:00', '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', '19:00', '20:00', '21:00', '22:00', '23:00', '24:00'];

  // Initializing schedule data structure to contain day and time that maps to boolean
  // value of selected or not selected
  let schedule: { [day: string]: { [time: string]: boolean } } = days.reduce((acc, day) => {
  acc[day] = times.reduce((timeAcc, time) => {
    timeAcc[time] = false; 
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

  // Initiates the selection/deselection process in calendar
function handleMouseDown(event: MouseEvent, day: string, time: string) {
  event.preventDefault(); 
  isSelecting = true; 
  startSelection = { day, time }; 
  initialSelectionState = !schedule[day][time]; 
  toggleTimeSlot(day, time);
  
}

// Dragging function
function handleMouseOver(event: MouseEvent, day: string, time: string) {
  event.preventDefault();
  if (isSelecting) {
    if (startSelection && (day !== startSelection.day || time !== startSelection.time)) {
        selectRange(initialSelectionState);
        endSelection = { day, time }; 
      }
  }
}

function selectRange(isSelected: boolean) {
    if (startSelection && endSelection && startSelection.day === endSelection.day) {
      const startIndex = times.indexOf(startSelection.time);
      const endIndex = times.indexOf(endSelection.time);
      for (let i = Math.min(startIndex, endIndex); i <= Math.max(startIndex, endIndex); i++) {
        schedule[startSelection.day][times[i]] = isSelected;
      }
      schedule = {...schedule};
    }
  }

// Finalizes the selection/deselection process
function handleMouseUp(event: MouseEvent) {
    isSelecting = false; 
    // Clear the selection start and end
    startSelection = null;
    endSelection = null;
}

// Toggles the selected state (select or deselect) of a time slot
function toggleTimeSlot(day: string, time: string) {
    // If the slot is not reserved, toggle its state
    schedule[day][time] = !schedule[day][time];
    // Update availability counts if necessary
    if (schedule[day][time]) {
      availabilityCounts[day][time] = (availabilityCounts[day][time] || 0) + 1;
    } else {
      availabilityCounts[day][time] = Math.max(0, (availabilityCounts[day][time] || 1) - 1);
    }
  schedule = {...schedule};
}

  // Indicates available for majority of previous users
  function isReserved(day: string, time: string) {
    return reservedSlots[day] && reservedSlots[day][time];
  }

  // All previous users are available
  function isReservedBest(day: string, time: string) {
    return reservedSlots[day] && reservedSlots[day][time] && reservedSlots[day][time]['current'] === reservedSlots[day][time]['total'];
  }


  // Recommend user certain time adjustment if needed
  function checkForOverlap(reserved, schedule) {
    for (const day of days) {
      for (const time of times) {
        if (schedule[day][time] && reserved[day] && reserved[day][time]) {
          return { day, time, overlap: true };
        }
      }
    }
    for (const day of days) {
      for (const time of times) {
        if (reserved[day] && reserved[day][time] && !schedule[day][time]) {
          return { day, time, overlap: false };
        }
      }
    }
    return null;
  }

  // Svelte Reactivity used
  // Calculates for overlap and recommended time to adjust 
  $: result = checkForOverlap(reservedSlots, schedule);
  $: if (result) {
    ({ day, time, overlap } = result);
    if (overlap) {
      message = `Perfect! We are awaiting 5 more people to submit their availability. Results will be sent to your email.`;
    } else {
      message = `Your availability differs a lot from other participants. Most participants are free on ${day} at ${time}. Can you adjust your availability?`;
    }
  }

  //Display modal 
  function showDialog() {
     const elementId = document.getElementById('dialog');
     if (elementId !== null) {
      elementId.showModal();
     }
   }
 
   function closeDialog() {
    const elementId = document.getElementById('dialog');
    if (elementId !== null) {
      if (message == `Perfect! We are awaiting 5 more people to submit their availability. Results will be sent to your email.`) {
        numberSubmitted = 6
      }
      elementId.close();
    }
   }

   function adjustMeeting(adjust: boolean) {
     if (adjust) {
      closeDialog();
     } else {
       message = "Availability submitted";
       numberSubmitted = 6
     }
   }

</script>

<style>

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
    margin: 0px 10px 10px -10px;
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

  .yes-button {
    background-color: green;
  }

  .no-button {
    background-color: red;
  }

  dialog {
    border-width: 2px;
    border-color: darkgray;
    border-radius: 5px;
    padding: 50px;
  }

  .reserved {
    background-color: #f0e68c;
    color: black; 
  }

  .reserved-best {
    background-color: #f2dd20;
    color: black; 
  }

  .reserved-selected {
    background-color: lightgreen; 
  }

  .reserved-selected-best {
    background-color: green; 
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
</style>

<h1>CS178 Meeting</h1>
<h2>{numberSubmitted}/10 participants have submitted their availability</h2>

<EmailInput />

<dialog
  id="dialog">
  <button on:click={closeDialog}>Close</button>
  <p>{message}</p>
  {#if !overlap && message !== "Availability submitted"}
    <button class="yes-button" on:click={() => adjustMeeting(true)}>Yes</button>
    <button class="no-button" on:click={() => adjustMeeting(false)}>No</button>
  {/if}
</dialog>

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
      {#each times as time, i}
        <tr>
          <td class="time-column">{time}</td>
          {#each days as day}
            <td
              class:selected={!isReserved(day, time) && schedule[day]?.[time]}
              class:reserved={isReserved(day, time) && !isReservedBest(day, time) && !schedule[day]?.[time]}
              class:reserved-best={isReserved(day, time) && isReservedBest(day, time) && !schedule[day]?.[time]}
              class:reserved-selected={isReserved(day, time) && !isReservedBest(day, time) && schedule[day]?.[time]}
              class:reserved-selected-best={isReserved(day, time) && isReservedBest(day, time) && schedule[day]?.[time]}
              on:mousedown={(event) => handleMouseDown(event, day, time)}
              on:mouseover={(event) => handleMouseOver(event, day, time)}
              on:mouseup={handleMouseUp}
            >
            {#if isReserved(day, time)}
            <span 
              class:reserved-selected={!isReservedBest(day, time) && schedule[day]?.[time]}
              class:reserved-selected-best={isReservedBest(day, time) && schedule[day]?.[time]}
            >
              {schedule[day]?.[time] 
                ? `${reservedSlots[day][time].current + 1}/${reservedSlots[day][time].total + 1} Available`
                : `${reservedSlots[day][time].current}/${reservedSlots[day][time].total} Available`}
            </span>
          {/if}
            </td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>


