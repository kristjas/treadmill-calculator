<template>
  <div class="container">
    <h2 class="title">Treadmill cardio calculator</h2>
    <!-- CALCULATION -->
    
    <div v-if="!calculated">
      <div class="input-group">
        <label>Current Weight (kg)</label>
        <input v-model.number="currentWeight" type="number" />
      </div>

      <div class="input-group">
        <label>Height (cm)</label>
        <input v-model.number="height" type="number" />
      </div>

      <div class="input-group">
        <label>Age</label>
        <input v-model.number="age" type="number" />
      </div>


  <div class="input-group">
  <label>How many kcal do you roughly consume per day on average?</label>

  <div class="radio-group">
    <label class="radio-option">
      <input
        type="radio"
        value="auto"
        v-model="calorieMode"
        @change="updateKcal"
      />
      <span>I eat about as many calories as I burn (maintenance)</span>
    </label>

    <label class="radio-option">
      <input
        type="radio"
        value="manual"
        v-model="calorieMode"
      />
      <span>I want to enter my calorie intake manually</span>
    </label>
  </div>

  <input
  v-model.number="kcal"
  type="number"
  :disabled="calorieMode === 'auto'"
  :class="{ 'disabled-input': calorieMode === 'auto' }"
  placeholder="Enter your daily calorie intake"
/>
    </div>



      <div class="input-group">
        <label>How active are you?</label>
        <div class="genderAndActivity-buttons">
          <button :class="{ active: activity === 1.38 }" @click="selectActivity(1.38)">Lightly active</button>
          <button :class="{ active: activity === 1.48 }" @click="selectActivity(1.48)">Moderately active</button>
          <button :class="{ active: activity === 1.58 }" @click="selectActivity(1.58)">Very active</button>
        </div>
      </div>


      <div class="input-group">
        <label>Gender</label>
        <div class="genderAndActivity-buttons">
          <button :class="{ active: gender === 'male' }" @click="selectGender('male')">Male</button>
          <button :class="{ active: gender === 'female' }" @click="selectGender('female')">Female</button>
        </div>
      </div>


      <div class="input-group">
        <label>Weight Loss Goal</label>
        <select v-model.number="goalPerWeek">
          <option :value="0.25">Lose 0.25 kg per week</option>
          <option :value="0.5">Lose 0.5 kg per week</option>
          <option :value="1">Lose 1 kg per week</option>
        </select>
      </div>


      <div class="input-group">
        <label>Training Sessions per Week: {{ sessionsPerWeek }}</label>
        <input v-model.number="sessionsPerWeek" type="range" min="1" max="7" />
      </div>

      <button class="button" @click="calculateBMR">Calculate BMR</button>
      <p v-if="submitted && !formComplete" class="error-text">Please fill all the fields.</p>

    </div>






    <!-- RESULTS -->







    <div v-else>
      <!-- NEED TO DO CARDIO -->
      <div v-if="kcalPerSession > 0" class="resultbox">



        <div class="controls">
          <h3>Select Treadmill Incline</h3>
          <div class="button-group">
            <button :class="{ active: incline === 0 }" @click="selectIncline(0)">0°</button>
            <button :class="{ active: incline === 10 }" @click="selectIncline(10)">10°</button>
            <button :class="{ active: incline === 15 }" @click="selectIncline(15)">15°</button>
          </div>

          <h3>Select Speed</h3>
          <div class="button-group">
            <button :class="{ active: speed === 3 }" @click="selectSpeed(3)">3 km/h</button>
            <button :class="{ active: speed === 5 }" @click="selectSpeed(5)">5 km/h</button>
            <button :class="{ active: speed === 7 }" @click="selectSpeed(7)">7 km/h</button>
          </div>

          <div class="result">
            <p>You need to do</p>
            <p><strong>{{ selectedMinutes.toFixed(2) }}</strong> minutes of treadmill walking on the selected setting</p>
            <p><strong>{{ sessionsPerWeek }}</strong> times a week</p>
          </div>


        </div>

     <div class="weekly-schedule">
  <h3>Sample Weekly Workout Schedule</h3>
  <div class="week-table">
    <div class="day" v-for="(day, i) in weekDays" :key="i">
      <div class="day-header">{{ day }}</div>
      <div
        class="day-cell"
        :class="{ workout: workoutDays.includes(i), rest: !workoutDays.includes(i) }"
      >
        <template v-if="workoutDays.includes(i)">
          Workout<br />
          <strong>{{ selectedMinutes.toFixed(2) }} min</strong>
        </template>
        <template v-else>
          Rest
        </template>
      </div>
    </div>
  </div>
</div>

<div class="result">
          <p>
            Your maintenance calories are
            <strong>{{ BMR.toFixed(2) }}</strong> kcal/day. You eat
            <strong>{{ kcal.toFixed(2) }}</strong> kcal/day. You need to burn
            <strong>{{ kcalPerSession.toFixed(2) }}</strong> kcal per session to lose <strong>{{ goalPerWeek.toFixed(2) }}</strong>kg per week.
          </p>
        </div>

      </div>

      <!-- ALREADY IN DEFICIT -->
      <div v-else class="result-negative">
        <p>You don’t need to do additional cardio to meet your current weight loss goal.</p>
      </div>

      <!-- BACK -->
      <button class="button mt-4" @click="reset">
        ← Go back and edit inputs
      </button>
    </div>
  </div>
</template>






<script setup>
import { ref, computed, nextTick } from 'vue';

const calculated       = ref(false);
const submitted        = ref(false);
const currentWeight    = ref(null);
const height           = ref(null);
const age              = ref(null);
const kcal             = ref(null);
const goalPerWeek      = ref(0.5);
const sessionsPerWeek  = ref(3);
const gender           = ref(null);
const activity         = ref(1.48);

const BMR              = ref(0);
const currentkcal      = ref(0);
const kcalPerSession   = ref(0);

const incline          = ref(10);
const speed            = ref(5);

const calorieMode = ref('auto'); 



const weekDays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];

const workoutDays = computed(() => {
  const patterns = {
    1: [0],
    2: [0, 3],
    3: [0, 2, 4],
    4: [0, 2, 4, 6],
    5: [0, 1, 2, 3, 4],
    6: [0, 1, 2, 3, 4, 5],
    7: [0, 1, 2, 3, 4, 5, 6],
  };
  return patterns[sessionsPerWeek.value] || [];
});





function updateKcal() {
  if (calorieMode.value === 'auto') {
    kcal.value = null;
  }
}
const MINUTES = {
  inc0s3: ref(0), inc0s5: ref(0), inc0s7: ref(0),
  inc10s3: ref(0), inc10s5: ref(0), inc10s7: ref(0),
  inc15s3: ref(0), inc15s5: ref(0), inc15s7: ref(0),
};

const resultDiv = ref(null);

const selectGender   = g => gender.value = g;
const selectActivity = a => activity.value = a;
const selectIncline  = i => incline.value = i;
const selectSpeed    = s => speed.value = s;


const formComplete = computed(
  () =>
    currentWeight.value > 0 &&
    height.value > 0 &&
    age.value > 0 &&
    gender.value !== null
);

function reset() {
  calculated.value     = false;
  submitted.value      = false;
  kcal.value = null;
  BMR.value            = 0;
  currentkcal.value    = 0;
  kcalPerSession.value = 0;
}

async function calculateBMR() {
  submitted.value = true;

  
  if (!formComplete.value) return;

  if (gender.value === 'male') {
    BMR.value = (10 * currentWeight.value + 6.25 * height.value - 5 * age.value + 5) * activity.value;
  } else if (gender.value === 'female') {
    BMR.value = (10 * currentWeight.value + 6.25 * height.value - 5 * age.value - 161) * activity.value;
  }

  if (kcal.value == null){
    kcal.value = BMR.value;
  }

  currentkcal.value = BMR.value - kcal.value;

  let weeklyTarget = 0;

if (goalPerWeek.value === 0.25) {
  weeklyTarget = 1925;
} else if (goalPerWeek.value === 0.5) {
  weeklyTarget = 3850;
} else {
  weeklyTarget = 7700;
}
  
  kcalPerSession.value = (weeklyTarget - currentkcal.value * 7) / sessionsPerWeek.value;

  const mets = {
    inc0s3:  2.5, inc0s5:  3.9, inc0s7:  5.0,
    inc10s3: 5.3, inc10s5: 7.5, inc10s7: 9.3,
    inc15s3: 8.0, inc15s5: 10.5, inc15s7: 12.5,
  };
  for (const key in MINUTES) {
    MINUTES[key].value = (kcalPerSession.value / (mets[key] * currentWeight.value)) * 60;
  }

  calculated.value = true;

}

//TEEME SWITCHIGA HILJEM
const selectedMinutes = computed(() => {
  if (incline.value === 0 && speed.value === 3)  return MINUTES.inc0s3.value;
  if (incline.value === 0 && speed.value === 5)  return MINUTES.inc0s5.value;
  if (incline.value === 0 && speed.value === 7)  return MINUTES.inc0s7.value;
  if (incline.value === 10 && speed.value === 3) return MINUTES.inc10s3.value;
  if (incline.value === 10 && speed.value === 5) return MINUTES.inc10s5.value;
  if (incline.value === 10 && speed.value === 7) return MINUTES.inc10s7.value;
  if (incline.value === 15 && speed.value === 3) return MINUTES.inc15s3.value;
  if (incline.value === 15 && speed.value === 5) return MINUTES.inc15s5.value;
  if (incline.value === 15 && speed.value === 7) return MINUTES.inc15s7.value;
  return 0;
});
</script>








<style scoped>
/* Base container */
.container {
  max-width: 480px;
  margin: 40px auto;
  padding: 30px;
  background: #ffffff;
  border: 1px solid #ccc; 
  border-radius: 0;        
  box-shadow: none;        
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Title */
.title {
  text-align: center;
  font-size: 24px;
  margin-bottom: 25px;
  color: #333;
}

/* Input groups */
.input-group {
  margin-bottom: 20px;
}
.input-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: 600;
  color: #444;
}
.input-group input,
.input-group select {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 15px;
  background-color: #fdfdfd;
  transition: border-color 0.2s;
}
.input-group input:focus,
.input-group select:focus {
  border-color: #007bff;
  outline: none;
}

/* Button groups */
.genderAndActivity-buttons,
.button-group {
  display: flex;
  gap: 10px;
  margin-top: 6px;
}
.radio-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 10px 0 16px;
}

.radio-option {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 14px;
  color: #333;
}

.radio-option input[type="radio"] {
  margin: 0;
}

.radio-option span {
  line-height: 1.4;
}


.genderAndActivity-buttons button,
.button-group button,
.button {
  flex: 1;
  padding: 10px;
  border: 2px solid transparent;
  border-radius: 8px;
  background-color: #e0e0e0;
  cursor: pointer;
  font-weight: 600;
  transition: background-color 0.2s, border-color 0.2s;
}

.genderAndActivity-buttons button.active,
.button-group button.active,
.button {
  background-color: #007bff;
  color: white;
}

.genderAndActivity-buttons button:hover,
.button-group button:hover,
.button:hover {
  background-color: #0056b3;
  color: white;
}

/* Results */
.controls {
  margin-top: 20px;
}

.controls h3 {
  text-align: center;
  font-size: 18px;
  margin: 20px 0 10px;
  color: #333;
}



.disabled-input {
  background-color: #d1d1d1 !important;
  color: #444 !important;
  cursor: not-allowed;
}

.resultbox {
  background: #f9f9ff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 100, 0.05);
  margin-top: 10px;
}

.resultbox .result p {
  margin: 10px 0;
  color: #444;
  font-size: 15px;
}

.result-negative {
  background: #fef2f2;
  color: #b30000;
  padding: 20px;
  border-radius: 10px;
  font-weight: 500;
  text-align: center;
}

/* Navigation */
.button.mt-4 {
  margin-top: 1.5rem;
  background-color: #555;
}

/* Error */
.error-text {
  color: #d93025;
  margin-top: 12px;
  text-align: center;
  font-weight: 500;
}

h3{
  color: #000;
}

.weekly-schedule {
  margin-top: 30px;
  text-align: center;
}

.week-table {
  display: grid;
  grid-template-columns: repeat(7, 1fr); 
  gap: 0;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #ccc;
  margin-top: 16px;
}

.day {
  display: flex;
  flex-direction: column;
  border-right: 1px solid #ccc;
}

.day:last-child {
  border-right: none;
}

.day-header {
  background-color: #acacac;
  color: #000;
  font-weight: 700;
  padding: 10px 0;
  font-size: 14px;
  border-bottom: 1px solid #ccc;
}

.day-cell {
  padding: 12px 0;
  font-size: 14px;
  font-weight: 500;
}

.day-cell.workout {
  background-color: #007bff;
  color: white;
}

.day-cell.rest {
  background-color: #f9f9f9;
  color: #777;
}

.result{
  background-color: #e0e0e0;
  color: #000;
  

}


</style>

