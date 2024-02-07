<script>
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
</script>

<h1>About Us</h1>
<p>This is the about page.</p>

<style>
  {#if selectedMeetingTime}
    <p>hello there</p>
    <p>{selectedMeetingTime}</p>
  {/if}

</style>