<template>
  <div class="about">
    <h1>About This App</h1>
    <p>
      This is a treadmill steady-state cardio calculator app designed to help you understand how much additional cardio you may need to meet your weight loss goals.
    </p>
    <p>
      We estimate your daily maintenance calories using your weight, height, age, gender, and activity level. This gives us a solid estimate of how many calories you burn each day to maintain your current weight.
    </p>
    <p>
      You then provide your estimated daily calorie intake. By comparing this with your maintenance level, we determine whether you're currently in a calorie surplus or deficit.
    </p>
    <p>
      If you’re not in a sufficient deficit to meet your weight loss target, we calculate how many additional calories you need to burn each week. Based on your preferred treadmill settings—incline, speed, and number of sessions per week—we tell you exactly how many minutes of cardio you need per session.
    </p>
    <p>
      This tool is designed for steady-state treadmill walking or jogging. It helps take the guesswork out of cardio planning so you can stay focused and efficient in your training.
    </p>
  </div>
</template>

<style scoped>
.about {
  max-width: 700px;
  margin: 60px auto;
  padding: 30px;
  background-color: #ffffff;
  color: #000000;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  font-family: Avenir, Helvetica, Arial, sans-serif;
}

.about h1 {
  font-size: 28px;
  margin-bottom: 20px;
  font-weight: bold;
}

.about p {
  font-size: 16px;
  line-height: 1.7;
  margin-bottom: 16px;
}
</style>
