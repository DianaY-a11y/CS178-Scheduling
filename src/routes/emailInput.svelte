<script lang="ts">
    let email = '';
    let editingEmail = true;
  
    let initialSelectionState = false;
  
    // Input Email Functions
      // Function to handle double-click for name
      function handleNameDblClick() {
      editingEmail = true;
      // Focus on the input element after it has been rendered
      setTimeout(() => {
        const emailInput = document.getElementById('email-input');
        if (emailInput !== null) {
            emailInput.focus();
    }
      });
      
    }

      // Function to save the name and stop editing on Enter key press
      function saveEmailInput(event: any) {
      if (event.key === 'Enter') {
        email = event.target.value;
        editingEmail = false;
        console.log("saved")
      }
    }
  </script>
  
  <style>
    input {
      margin: 0px 10px 10px -10px;
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

  </style>
  
  <div class="input-container">
    {#if editingEmail}
      <input
        id="email-input"
        type="text"
        value={email}
        on:blur={() => editingEmail = false}
        on:keydown={event => saveEmailInput(event)}
        placeholder="Enter your email"
      />
    {:else}
      <span on:dblclick={handleNameDblClick}>{email || 'Enter your email'}</span> 
    {/if}
  
  </div>
  