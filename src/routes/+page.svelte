<script>
  let isSelecting = false;
  let slots = [];
  let selectedSlots = new Set();

  // Initialize time slots for the calendar
  const times = ['9:00 AM', '10:00 AM', '11:00 AM', '12:00 PM', '1:00 PM', '2:00 PM', '3:00 PM', '4:00 PM', '5:00 PM'];
  const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
  
  // Populate the slots array with objects representing each time slot
  for (let day of days) {
    for (let time of times) {
      slots.push({ day, time, selected: false });
    }
  }

  function mouseDown(slot) {
    isSelecting = true;
    toggleSlot(slot);
  }

  function mouseOver(slot) {
    if (isSelecting) {
      toggleSlot(slot);
    }
  }

  function mouseUp() {
    isSelecting = false;
  }

  function toggleSlot(slot) {
    let slotId = `${slot.day}-${slot.time}`;
    if (selectedSlots.has(slotId)) {
      selectedSlots.delete(slotId);
    } else {
      selectedSlots.add(slotId);
    }
    slot.selected = !slot.selected;
  }
</script>

<style>
  .calendar {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 1px;
    background: black;
    user-select: none;
  }
  .time-slot {
    background: #fff;
    padding: 10px;
    text-align: center;
    border: 1px solid #ddd;
  }
  .selected {
    background-color: green;
  }
</style>

<div class="calendar" on:mouseup={mouseUp}>
  {#each days as day}
    <div class="header">{day}</div>
  {/each}
  
  {#each slots as slot}
    <div
      class="time-slot {slot.selected ? 'selected' : ''}"
      on:mousedown={() => mouseDown(slot)}
      on:mouseover={() => mouseOver(slot)}
      on:mouseup={mouseUp}
    >
      {slot.time}
    </div>
  {/each}
</div>
