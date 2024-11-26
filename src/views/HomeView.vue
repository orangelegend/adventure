<script setup>

</script>

<template>


  <section class="story">
      <!-- Dynamically display an image if available for the current story point -->
      <img v-if="currentStory.image" :src="currentStory.image" alt="Story image" class="story-image" />

      <!-- Display the text of the current story dynamically from CSV data -->
      <p>{{ currentStory.text }}</p>


  <section class="action">
      <!-- <p>Enter the Azure Nebula?</p> -->
      <p>{{ currentStory.prompt }}</p>
  </section>


  </section>


  <section class="choices">
      <div class="choices-container">


      <div v-if="hasChoices">
        <!-- Display the first choice if available -->
        <div v-if="currentStory.choice1_text">
            <button @click="makeChoice(currentStory.choice1_next)" class="btn1">{{ currentStory.choice1_text }}</button>
        </div>
        <!-- Display the second choice if available -->
        <div v-if="currentStory.choice2_text">
            <button @click="makeChoice(currentStory.choice2_next)" class="btn2">{{ currentStory.choice2_text }}</button>
        </div>
        <div v-if="currentStory.choice3_text">
            <button @click="makeChoice(currentStory.choice3_next)" class="btn1">{{ currentStory.choice3_text }}</button>
        </div>
        <div v-if="currentStory.choice4_text">
            <button @click="makeChoice(currentStory.choice4_next)" class="btn2">{{ currentStory.choice4_text }}</button>
        </div>
        <div v-if="currentStory.choice5_text">
            <button @click="makeChoice(currentStory.choice5_next)" class="btn1">{{ currentStory.choice5_text }}</button>
        </div>
      </div>
     
      <!-- If no choices are available, display "The End" message -->
      <div v-else class="theend">
          <h1>The End</h1>
          <p>Try Playing Again!</p>
      </div>
    </div>
  </section>


  <div>
    <!-- Save slot management for saving and loading progress -->
    <section class="saveslots">
      <h1>Save Slots</h1>
      <!-- Loop through save slots, display status and action buttons for each slot -->
      <div v-for="(slot, index) in saveSlots" :key="index" class="save-slot">
        <p>Slot {{ index + 1 }} - {{ slot !== null ? 'Saved' : 'Empty' }}</p>
        <!-- Save the current progress to the selected slot -->
        <button @click="saveProgress(index)">Save to Slot {{ index + 1 }}</button>
        <!-- Load the progress from the selected slot, disabled if the slot is empty -->
        <button @click="loadProgress(index)" :disabled="slot === null">Load from Slot {{ index + 1 }}</button>
        <!-- Reset (clear) the selected slot, disabled if the slot is empty -->
        <button @click="resetSlot(index)" :disabled="slot === null">Reset Slot {{ index + 1 }}</button>
        <br/><br/>
      </div>
    </section>

  </div>


</template>


<script>
// import Papa from 'papaparse';


export default {
data() {
  return {
    currentIndex: 0,          // Tracks the current story point (index in the story array)
    story: [],                // The story data loaded from the CSV file will be stored here
    saveSlots: Array(3).fill(null) // Initializes three empty save slots for saving game progress
  };
},
computed: {
  // Computes the current story based on the currentIndex; if index is invalid, returns an empty object
  currentStory() {
    return this.story[this.currentIndex] || {};
  },
  hasChoices() {
  return this.currentStory.choice1_text || this.currentStory.choice2_text ||
         this.currentStory.choice3_text || this.currentStory.choice4_text ||
         this.currentStory.choice5_text;
  }
},
methods: {
  // Method to load the story data from a CSV file using fetch
  loadStory() {
    fetch('/story.csv')       // Fetches the CSV file from the server
      .then(response => response.text()) // Reads the file content as text
      .then(csvText => {
        Papa.parse(csvText, {            // Parses the CSV text using PapaParse
          header: true,                  // Treats the first row as headers for the data
          complete: (result) => {
            this.story = result.data;    // Stores the parsed data into the 'story' array
          },
          skipEmptyLines: true           // Ignores empty lines in the CSV file
        });
      })
      .catch(error => console.error('Error loading CSV:', error)); // Logs an error if the CSV fails to load
  },
  // Method to handle user choice, updates currentIndex to point to the next story segment
  makeChoice(nextIndex) {
    this.currentIndex = parseInt(nextIndex); // Converts nextIndex to an integer and updates currentIndex
  },


  // Save the current progress (story index) into a selected save slot
  saveProgress(slotIndex) {
    this.saveSlots[slotIndex] = {         // Save current story index to the chosen slot
      currentIndex: this.currentIndex
    };
    // Persist save slots to localStorage so progress is retained even if the page is refreshed
    localStorage.setItem('saveSlots', JSON.stringify(this.saveSlots));
  },
  // Load the saved progress from the selected slot and update currentIndex
  loadProgress(slotIndex) {
    const savedSlot = this.saveSlots[slotIndex];
    if (savedSlot !== null) {
      this.currentIndex = savedSlot.currentIndex; // Restore the saved story index
    }
  },
  // Reset the selected save slot (clear the saved progress)
  resetSlot(slotIndex) {
    this.saveSlots[slotIndex] = null;      // Clear the slot by setting it to null
    // Update the save slots in localStorage after reset
    localStorage.setItem('saveSlots', JSON.stringify(this.saveSlots));
  },
  // Load the saved slots from localStorage (if any) when the app starts
  loadSaveSlots() {
    const savedSlots = localStorage.getItem('saveSlots'); // Retrieve save slots from localStorage
    if (savedSlots !== null) {
      this.saveSlots = JSON.parse(savedSlots); // Parse and load the saved slots into saveSlots array
    }
  }


},
mounted() {
  this.loadSaveSlots();  // Load saved slots from localStorage when the component is mounted
  this.loadStory();      // Load the story from CSV when the component is mounted
}
};
</script>