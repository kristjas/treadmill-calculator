<template>
  <div class="container">
    
    <h2 class="title">Treadmill cardio calculator <img :src="logo" alt="Logo" class="logo" /></h2>
    
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
      <p v-if="submitted && !formComplete" class="error-text">Please fill all the fields.</p>
      <button class="button" @click="calculateBMR">Calculate</button>
      

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

          <div class="result1">
            <p><strong>You need to do:</strong></p>
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
import logo from '@/assets/logo.png';
import './HomeView.css';

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
  await nextTick(); 
  window.scrollTo({ top: 0, behavior: 'smooth' });

}



const selectedMinutes = computed(() => {
  const key = `inc${incline.value}s${speed.value}`;
  switch (key) {
    case 'inc0s3':  return MINUTES.inc0s3.value;
    case 'inc0s5':  return MINUTES.inc0s5.value;
    case 'inc0s7':  return MINUTES.inc0s7.value;
    case 'inc10s3': return MINUTES.inc10s3.value;
    case 'inc10s5': return MINUTES.inc10s5.value;
    case 'inc10s7': return MINUTES.inc10s7.value;
    case 'inc15s3': return MINUTES.inc15s3.value;
    case 'inc15s5': return MINUTES.inc15s5.value;
    case 'inc15s7': return MINUTES.inc15s7.value;
    default: return 0;
  }
});
</script>







