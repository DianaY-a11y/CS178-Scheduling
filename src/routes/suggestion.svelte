<script lang="ts" context="module">
   let message = 'Scheduling Failed. 4/5 participants are available Tuesday 2:00pm - 3:00pm. Can you adjust?';
   let meetingFinalized = false;

   // Re-selection Modal
 
   export function showDialog() {
    console.log("show dialog")
     const elementId = document.getElementById('dialog');
     if (elementId !== null){
      elementId.showModal();
     }
   }
 
   export function closeDialog() {
    const elementId = document.getElementById('dialog');
    if (elementId !== null){
      elementId.close();
    }
   }

   // Function to adjust the meeting based on user's choice
   function adjustMeeting(adjust: boolean) {
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

 </style>
 
 
 <dialog
  id="dialog">
   <button on:click={closeDialog}>Close</button>
   <p>{message}</p>
   {#if !meetingFinalized}
     <button on:click={() => adjustMeeting(true)}>Yes</button>
     <button on:click={() => adjustMeeting(false)}>No</button>
   {/if}
 </dialog>
 
 