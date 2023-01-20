<template>
  <div v-scroll="onScroll" class="q-pt-sm bg-blue">
    <div class="fixed-bottom-right q-ma-sm">
      <q-btn round color="primary" icon="restart_alt" size="lg" @click="reset"/>
    </div>
    <div class="q-ma-lg q-pa-xl q-mt-none shadow-7 bg-white rounded-borders">
      <p class="text-h2 text-center justify-center q-ma-lg">
        Should sampling be used in the census?
      </p>
    </div>
    <div class="q-ma-lg q-pa-xl q-mt-none shadow-7 bg-white rounded-borders text-subtitle1 text-center">
      <q-splitter
        v-model="splitter"
        disable
      >
        <template v-slot:before>
          <div class="q-pa-md">
            <div class="text-h4 q-mb-md text-center">Population</div>
            <div class="q-my-md">A population is all items of interest in a study. This could be all of the people in San Francisco, for example. </div>
          </div>
        </template>
        <template v-slot:after>
          <div class="q-pa-md">
            <div class="text-h4 q-mb-md text-center">Sample</div>
            <div class="q-my-md">A sample is a subset of the population, ideally one that is representative of the makeup of the population, random, and non-biased. </div>
          </div>
        </template>
      </q-splitter>
      <div class="q-mt-lg">
        <h6 class="q-ma-sm">Do you think the US census is a population or a sample?</h6>
        <q-btn color="white" text-color="black" label="Population" @click="wrong = !wrong; right = false" class="q-mx-sm"/>
        <q-btn color="white" text-color="black" label="Sample" @click="right = !right; wrong = false" class="q-mx-sm"/>
      </div>
      <div class="q-mt-lg shadow-3" v-if="wrong || right">
        <p v-if="wrong" class="q-pa-md rounded-borders">You're partially right. The United States Census tries to survey an entire population to “provide the truest measure of all minority groups”, while minimizing sampling errors. However, these ideals fall short. First of all, the census requires a monstrous budget to execute, and it is difficult to administer to all members of the United States. Non-response bias, or a bias that occurs where people who don’t respond to surveys are not counted, accounts for a huge disparity in the enumeration of minority groups and children in particular. For example, the National Academy of Sciences examined data and found a “probable undercount of 1.8% of the general population” (Lerner, Science Clarified).</p>
        <p v-if="right" class="q-pa-md rounded-borders">You're wrong, but a lot of people think you should be right. Because the Census can't find the whole population anyway, some people suggest that we make it a sample instead. Sampling is both cost-efficient and easier to execute because it uses small sample sizes and statistics to evaluate the size and makeup of a population. Through reliable mathematical formulas based on birth, death, emigration, and immigration records it can count marginalized groups and minorities with accuracy. Also, the errors that proceed from statistics-based sampling are not based on race, difficulty of acess to certain groups, and are altogether easier to identify, quantify, and rectify.</p>
      </div>
    </div>
    <div class="q-ma-lg q-pa-xl q-mb-none shadow-7 bg-white rounded-borders">
      <div class="text-center text-h4">Try it Yourself</div>
      <q-input v-model="population" label="Population" />
      <q-input v-model="sample_one" label="Sample One Size" />
      <q-input v-model="sample_two" label="Sample Two Size" />
      <q-btn color="blue" text-color="white" label="Submit" @click="guess" class="q-ma-sm"/>
      <div v-if="answered" class="row text-center text-white">
        <div :class="['col', 'q-ma-sm', 'q-mb-lg', 'rounded-borders', 'bg-' + color]" style="height: 25vh;">
          <h3>Result</h3>
          <h5>{{ result.toLocaleString("en-US") }}</h5>
          <q-tooltip max-width="300px">
            This is the population that the algorithm guessed. Keep in mind that, even though you inputted the population, the algorithm only knew samples of it and had to extrapolate from there.
          </q-tooltip>
        </div>
        <div :class="['col', 'q-ma-sm', 'q-mb-lg', 'rounded-borders', 'bg-' + color]" style="height: 25vh;">
          <h3>Difference</h3>
          <h5>{{ difference.toLocaleString("en-US") }} ({{ percent_difference }}%)</h5>
          <q-tooltip max-width="300px">
            This is the difference between the actual population and the guessed population as well as a percentage difference. Differences of &lt;0.5% are shown as green, differences of &lt;2.5% are shown as orange, and anything above that is red.
          </q-tooltip>
        </div>
      </div>
      <q-list v-if="answered" bordered class="rounded-borders q-ma-lg">
        <q-expansion-item
          expand-separator
          icon="school"
          label="What is the Lincoln Index equation?"
        >
          <q-card>
            <q-card-section>
              The Lincoln Index is a popular method for counting the population of fish, the Lincoln Index is used by taking a sample of a population, then taking a second sample and marking the overlap between the two.
            </q-card-section>
          </q-card>
        </q-expansion-item>
        <q-expansion-item
          expand-separator
          icon="calculate"
          label="How does the Lincoln Index work?"
        >
          <q-card>
            <q-card-section>
              Where E1 = number of people in the first sample, E2 = number of people in the second sample, S = number of overlapping people, and L  = total population, the Lincoln Index say   L = (E1*E2)/S
            </q-card-section>
          </q-card>
        </q-expansion-item>
        <q-expansion-item
          expand-separator
          icon="help"
          label="Why does the Lincoln Index work?"
        >
          <q-card>
            <q-card-section>
              The Lincoln Index assumes the ratio of the number of people in the first sample to the total population is equal to the number of overlapping people and the number of people in the second sample: E1/L = S/E2
            </q-card-section>
          </q-card>
        </q-expansion-item>
      </q-list>
    </div>
    <div class="q-ma-lg q-pa-xl q-mt-none shadow-7 bg-white rounded-borders">
      <h4 class="text-center">Why does this even matter?</h4>
      <p>Firstly, the Census forms the foundation of the organization of congressional districts in the U.S., which influences how votes are apportioned and delegates are selected. If there is an under-assessment of minorities, for example, this poses both the problem of unequal representation and insufficient amounts of government aid (for the clinically disabled, etc.). Finally, a huge fiscal problem would be solved should the Census be reformed: it is estimated that approximately $100 million could be solved, “assuming an overall sampling rate of 75 percent." </p>
    </div>
    <div class="text-center q-pa-md bg-white shadow-up-3">
      <p class="q-mb-none">Made with Vue3 and Quasar</p>
      <p class="q-mb-none">Created by Lucas Chang, Erin Neal, and Elsa Krummel</p>
      <p class="q-mb-none"><a href="https://github.com/lukajaa/census-sample" target="_blank">Source Code</a> / Bibliography</p>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue';

const population = ref(100000);
const sample_one = ref(75000);
const sample_two = ref(75000);
const result = ref(0);
const difference = ref(0);
const percent_difference = ref(0);
const answered = ref(false);
const wrong = ref(false);
const right = ref(false);
const splitter = ref(50);
const color = computed(function() {
  if (Math.abs(percent_difference.value) > 2.5) {
    return 'red'
  } else if (Math.abs(percent_difference.value) > 0.5) {
    return 'orange'
  } else {
    return 'green'
  }
})

function onScroll (position : number) {
  console.log(position)
}

function getRandomSubarray(arr : Array<number>, size : number) {
    var shuffled = arr.slice(0), i = arr.length, temp, index;
    while (i--) {
        index = Math.floor((i + 1) * Math.random());
        temp = shuffled[index];
        shuffled[index] = shuffled[i];
        shuffled[i] = temp;
    }
    return shuffled.slice(0, size);
}

function reset() {
  window.scroll({
    top: 0,
    left: 0,
    behavior: 'smooth'
  });
  answered.value = false;
  population.value = 100000;
  sample_one.value = 75000;
  sample_two.value = 75000;
  wrong.value = false
  right.value = false
}

function guess() {
  var people = []
  for (let i = 0; i <= population.value; i++) {
    people.push(i)
  }
  // shuffle
  const sample_one_items = getRandomSubarray(people, sample_one.value)
  const sample_two_items = getRandomSubarray(people, sample_two.value)

  let r = 0
  sample_one_items.forEach(function (item) {
    people[item] = 'tagged'
  });
  sample_two_items.forEach(function (item) {
    if (people[item] == 'tagged') {
      r += 1
    }
  });
  result.value = Math.round(sample_one.value * sample_two.value / r)
  difference.value = Math.abs(population.value - result.value)
  percent_difference.value = Math.round((result.value - population.value) / ((population.value + result.value) / 2) * 100000) / 1000
  answered.value = true
}
</script>
