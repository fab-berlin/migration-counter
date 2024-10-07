<template>
  <div class="widget">
    <div class="widget-content">
      <div class="circle">
        <div class="content">
          <div class="value">{{ migrationData.currentWeek.percent }}</div>
          <div class="label">%</div>
        </div>
      </div>
      <div :class="arrowClasses">
        <div class="shaft"></div>
        <div class="arrow-head"></div>
      </div>
    </div>
    <div class="description">
      <p>{{ migrationData.currentWeek.week }}</p>
      <p>
        {{ migrationData.currentWeek.current }} von
        {{ migrationData.currentWeek.total }} Datensätzen migriert
      </p>
    </div>
    <Confetti v-if="showConfetti" />
  </div>
</template>

<script>
import axios from 'axios'
import { ref, reactive } from 'vue'
import Confetti from '@/components/Confetti.vue'

export default {
  name: 'App',
  components: { Confetti },
  setup() {
    const migrationData = reactive({
      currentWeek: { current: 0, total: 0, percent: 0, week: '' },
      lastWeek: { current: 0, total: 0, percent: 0 },
      trend: 0
    });

    const confettiThresholds = [20,40,50,70,75,80,100];

    const arrowClasses = ref('')
    const classes = {
      '-1': 'down',
      0: 'unmodified',
      1: 'up'
    }

    const showConfetti = ref(false);

    axios.get('./data.csv').then((response) => {
      const data = response.data.split('\n').reverse()
      processMigrationData(data)
    })

    const processMigrationData = (data) => {
      // current week
      migrationData.currentWeek.current = parseInt(data[0].split(',')[1])
      migrationData.currentWeek.total = parseInt(data[0].split(',')[2])
      migrationData.currentWeek.percent = (Math.round(
        (migrationData.currentWeek.current / migrationData.currentWeek.total) * 100 *10
      ) / 10).toFixed(1);
      migrationData.currentWeek.week = data[0].split(',')[0]

      // last week
      migrationData.lastWeek.current = parseInt(data[1].split(',')[1])
      migrationData.lastWeek.total = parseInt(data[1].split(',')[2])
      migrationData.lastWeek.percent = Math.round(
        (migrationData.lastWeek.current / migrationData.lastWeek.total) * 100 * 10
      ) / 10;

      if (Math.ceil(migrationData.lastWeek.percent) < Math.floor(migrationData.currentWeek.percent) && confettiThresholds.includes(Math.floor(migrationData.currentWeek.percent))) {
        showConfetti.value = true;
      }

      // trend
      migrationData.trend =
        migrationData.currentWeek.percent > migrationData.lastWeek.percent
          ? 1
          : migrationData.currentWeek.percent < migrationData.lastWeek.percent
            ? -1
            : 0
      arrowClasses.value = 'arrow arrow-' + classes[migrationData.trend]
    }

    return {
      migrationData,
      arrowClasses,
      showConfetti
    }
  }
}
</script>

<style scoped>
.widget {
  display: flex;
  flex-direction: column;
  align-items: center;
  border: 1px solid #000;
  padding: 15px;
  width: 350px;
  position: relative;
  overflow: hidden;
}
.widget-content {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  align-items: center;
  gap: 25px;
}
.circle {
  font-family: sans-serif;
  width: 200px;
  height: 200px;
  border-radius: 200px;
  border: 1px solid #333;
  background-color: #f6f6f6;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 15px;
}

.content {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  align-items: baseline;
  color: #333;
}

.value {
  font-size: 80px;
  font-weight: bold;
}
.label {
  font-size: 24px;
}

.arrow {
  position: relative;
  display: flex;
  flex-direction: column;
  flex-wrap: nowrap;
  align-items: center;
  transform: rotate(270deg);
  transition: all 1s ease-in-out;
}
.arrow-unmodified {
  transform: rotate(-90deg);
}
.arrow-up {
  transform: rotate(180deg);
}
.arrow-down {
  transform: rotate(0deg);
}

.arrow-head {
  position: relative;
  width: 0;
  height: 0;
  border-left: 20px solid transparent; /* Half of the arrow base */
  border-right: 20px solid transparent; /* Half of the arrow base */
  border-top: 40px solid #ffd72b; /* Height of the arrow */

  transition: all 1s ease-in-out;
}

.shaft {
  position: relative;
  width: 13px;
  height: 30px;
  background-color: #ffd72b;
  transition: all 1s ease-in-out;
}

.arrow-down .arrow-head {
  border-top-color: #ec1616;
}
.arrow-down .shaft {
  background-color: #ec1616;
}
.arrow-up .arrow-head {
  border-top-color: #7bad18;
}
.arrow-up .shaft {
  background-color: #7bad18;
}

.description {
  font-family: sans-serif;
  color: #333;
}
.description p:first-child {
  font-weight: bold;
}
.description p {
  margin: 0;
}
</style>
