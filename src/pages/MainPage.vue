<template>
  <div class="q-ma-md">
    <q-input v-model="population" label="Population" />
    <q-input v-model="sample_size" label="Sample Size" />
    <q-btn color="white" text-color="black" label="Submit" @click="guess"/>
    <p>Result: {{ result }}</p>
  </div>
</template>

<script lang="ts" setup>
import {ref} from 'vue';

const population = ref(100000);
const sample_size = ref(50000);
const result = ref(0);

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

function guess() {
  var people = []
  for (let i = 0; i <= population.value; i++) {
    people.push(i)
  }
  // shuffle
  const sample_one = getRandomSubarray(people, sample_size.value)
  const sample_two = getRandomSubarray(people, sample_size.value)

  let r = 0
  sample_one.forEach(function (item) {
    people[item] = 'tagged'
  });
  sample_two.forEach(function (item) {
    if (people[item] == 'tagged') {
      r += 1
    }
  });
  result.value = sample_size.value**2 / r
}

</script>
