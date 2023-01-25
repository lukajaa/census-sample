<template>
  <div v-scroll="onScroll" class="q-pt-sm bg-blue">
    <Transition>
      <div v-if="restart" class="fixed-bottom-right q-ma-sm" transition-show="fade">
        <q-btn round color="primary" icon="restart_alt" size="lg" @click="reset"/>
      </div>
    </Transition>
    <div class="q-ma-lg q-pa-xl q-mt-none shadow-7 bg-white rounded-borders">
      <p class="text-h2 text-center justify-center q-ma-lg">
        Should sampling be used in the census?
      </p>
    </div>
    <div class="q-ma-lg q-pa-xl q-mt-none shadow-7 bg-white rounded-borders">
      <p class="text-h4 text-center justify-center q-ma-lg">
        What is the census?
      </p>
      <p class="text-subtitle1">Established in 1790 and administered every decade, the United States Census attempts to survey the entire population and provide the truest measure of all groups through targeted questions and an accumulation of a wide variety of data. While data collection and computation initially began with door-to-door efforts, it has evolved to make use of more complex forms of digital surveying and calculations. </p>
      <div class="q-mt-lg shadow-3 q-pa-sm">
        <p class="text-h6 text-center">Get a question from any historical census!</p>
        <p class="text-center text-subtitle1">{{ current_question }}</p>
        <q-btn color="primary" @click="random_question" label="New Question" />
      </div>
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
            <div class="q-my-md">A sample is a subset of the population that is representative, random, and non-biased. For example, one thousand random people from San Francisco.</div>
          </div>
        </template>
      </q-splitter>
      <div class="q-mt-lg">
        <h6 class="q-ma-sm">Do you think the US census is a population or a sample?</h6>
        <q-btn color="white" text-color="black" label="Population" @click="wrong = !wrong; right = false" class="q-mx-sm"/>
        <q-btn color="white" text-color="black" label="Sample" @click="right = !right; wrong = false" class="q-mx-sm"/>
      </div>
      <Transition>
        <div class="q-mt-lg shadow-3" v-if="wrong || right">
          <p v-if="wrong" class="q-pa-md rounded-borders">You're wrong. The United States Census tries to survey an entire population to “provide the truest measure of all minority groups”, while minimizing sampling errors. However, these ideals fall short. First of all, the census requires a monstrous budget to execute, and it is difficult to administer to all members of the United States. Non-response bias, or a bias that occurs where people who don’t respond to surveys are not counted, accounts for a huge disparity in the enumeration of minority groups and children in particular. For example, the National Academy of Sciences examined data and found a “probable undercount of 1.8% of the general population” (Lerner, Science Clarified).</p>
          <p v-if="right" class="q-pa-md rounded-borders">You're wrong, but a lot of people think you should be right. The Census can't find the whole population anyway because there will always be people who don't respond, so some people suggest we make it a sample instead. Sampling is both cost-efficient and easier to execute because it uses small sample sizes and statistics to evaluate the size and makeup of a population. Through reliable mathematical formulas based on birth, death, emigration, and immigration records it can count marginalized groups and minorities with accuracy. Also, the errors produced from statistics-based sampling are not based on race, difficulty of access to certain groups, and are altogether easier to identify, quantify, and rectify.</p>
        </div>
      </Transition>
    </div>
    <div class="q-ma-lg q-pa-xl q-mb-none shadow-7 bg-white rounded-borders">
      <div class="text-center text-h4">Try Sampling Yourself!</div>
      <div class="row">
        <div class="col q-mx-sm">
          <q-input v-model="population" label="Population" />
        </div>
        <div class="col q-mx-sm">
          <q-select v-model="state" :options="Object.keys(options)" label="State" />
        </div>
      </div>
      <div class="row">
        <div class="col q-mx-sm">
          <q-input v-model="sample_one" label="Sample One Size" />
        </div>
        <div class="col q-mx-sm">
          <q-input v-model="sample_two" label="Sample Two Size" />
        </div>
      </div>
      <q-btn color="blue" text-color="white" label="Submit" @click="guess" class="q-ma-sm"/>
      <Transition>
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
            This is the difference between the actual population and the guessed population as well as a percentage difference. Any difference better than the census (&lt;1.8%) is green of, differences of &lt;5% are shown as orange, and anything above that is red.
          </q-tooltip>
        </div>
      </div>
      </Transition>
      <Transition>
      <q-list v-if="answered" bordered class="rounded-borders q-ma-lg">
        <q-expansion-item
          expand-separator
          icon="school"
          label="What is the Lincoln Index equation?"
        >
          <q-card>
            <q-card-section>
              The Lincoln Index is a popular method for counting populations using sampling. Ecologists first executed the idea by capturing sample of fish, marking them, and after releasing them again, taking a new sample. The ecologists then would note the amount of previously-marked fish in the second sample. By counting the overlap between the two samples, you can make estimates towards the size of the whole population.
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
              Where E1 = number of people in the first sample, E2 = number of people in the second sample, S = number of overlapping people, and L  = total population, the Lincoln Index says L = (E1*E2)/S
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
      </Transition>
    </div>
    <div class="q-ma-lg q-pa-xl q-mt-none shadow-7 bg-white rounded-borders">
      <h4 class="text-center">Why does this even matter?</h4>
      <p class="text-subtitle1">Firstly, the Census forms the foundation of congressional districts in the U.S., which influences how votes are apportioned and delegates are selected. If there is an under-assessment of minorities, for example, this poses both a couple of problems: unequal representation and insufficient amounts of government aid. Finally, a huge fiscal problem would be solved should the Census be reformed: it is estimated that approximately $100 million could be saved, “assuming an overall sampling rate of 75 percent" (Lerner, Science Clarified).</p>
    </div>
    <div class="text-center q-pa-md bg-white shadow-up-3">
      <p class="q-mb-none">Made with Vue3 and Quasar</p>
      <p class="q-mb-none">Created by Lucas Chang, Erin Neal, and Elsa Krummel</p>
      <p class="q-mb-none"><a href="https://github.com/lukajaa/census-sample" target="_blank">Source Code</a> / <a @click="sourcesModal = !sourcesModal">Bibliography</a></p>
    </div>
    <q-dialog
      v-model="sourcesModal"
    >
      <q-card style="width: 700px; max-width: 80vw;">
        <q-card-section>
          <div class="text-h6">Sources Used</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <p>- Nagel, M C. “Should Statistical Sampling Be Used in the United States Census?” Science Clarified, Science Clarified, www.scienceclarified.com/dispute/Vol-2/Should-statistical-sampling-be-used-in-the-United-States-Census.html.</p>
          <p>- “Lincoln Index.” Wikipedia, Wikimedia Foundation, 11 Aug. 2021, en.wikipedia.org/wiki/Lincoln_index.</p>
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, watch } from 'vue';

const population = ref(5024279);
const sourcesModal = ref(false);
const state = ref('Alabama');
const options = ref({'California':39538223,'Texas':29145505,'Florida':21538187,'New York':20201249,'Pennsylvania':13002700,'Illinois':12812508,'Ohio':11799448,'Georgia':10711908,'North Carolina':10439388,'Michigan':10077331,'New Jersey':9288994,'Virginia':8631393,'Washington':7705281,'Arizona':7151502,'Massachusetts':7029917,'Tennessee':6910840,'Indiana':6785528,'Maryland':6177224,'Missouri':6154913,'Wisconsin':5893718,'Colorado':5773714,'Minnesota':5706494,'South Carolina':5118425,'Alabama':5024279,'Louisiana':4657757,'Kentucky':4505836,'Oregon':4237256,'Oklahoma':3959353,'Connecticut':3605944,'Utah':3271616,'Iowa':3190369,'Nevada':3104614,'Arkansas':3011524,'Mississippi':2961279,'Kansas':2937880,'New Mexico':2117522,'Nebraska':1961504,'Idaho':1839106,'West Virginia':1793716,'Hawaii':1455271,'New Hampshire':1377529,'Maine':1362359,'Rhode Island':1097379,'Montana':1084225,'Delaware':989948,'South Dakota':886667,'North Dakota':779094,'Alaska':733391,'Vermont':643077,'Wyoming':576851});
const questions = ref(['How many people are living or staying in this house, apartment, or mobile home?', 'Does Person 1 sometimes live or stay somewhere else?', 'What state or country was this person from?', 'Has this person ever served on active duty in the U.S. Armed Forces, military Reserves, or National Guard?', 'What kind of work was this person doing?', 'About when was this building first built?', 'Do you have complete plumbing facilities in this house, apartment or mobile home?', 'Do you have complete kitchen facilities in this house, apartment or mobile home?', 'Is this building connected to the public sewer?', 'Do you have complete plumbing facilities in your living quarters, that is, hot and cold piped water, a flush toilet, and a bathtub or shower?', 'Do you have air conditioning?', 'How many passenger automobiles are owned or regularly used by members of your household?', 'Do you have a flush toilet?', 'Do you have a bathtub or shower?', 'Is there a basement in this building?', 'Do you have a battery-operated radio?', 'Do you have a television set?', 'Does the household have an electric or gas clothes dryer?', 'Does the household have a home food freezer which is separate from its refrigerator?', 'What city and county did this person work in last week?', 'Corresponding agriculture questionnaire number', 'Is this house on a farm (or ranch)? (Yes or No)', 'Is the person a survivor of the Union or Confederate Army or Navy?', 'Can the person not read?', 'Can the person not write?']);
const current_question = ref('How many people are in your household?')
const sample_one = ref(2512140);
const sample_two = ref(1256070);
const result = ref(0);
const difference = ref(0);
const percent_difference = ref(0);
const answered = ref(false);
const wrong = ref(false);
const right = ref(false);
const splitter = ref(50);
const restart = ref(false);
const color = computed(function() {
  if (Math.abs(percent_difference.value) > 5) {
    return 'red'
  } else if (Math.abs(percent_difference.value) > 1.8) {
    return 'orange'
  } else {
    return 'green'
  }
})

function onScroll (position) {
  if (position > 250) {
    restart.value = true
  } else {
    restart.value = false
  }
}

function random_question() {
  current_question.value = questions.value[Math.floor(Math.random() * questions.value.length)]
}

watch(state, (state_name : string) => {
  console.log(state_name)
  population.value = options.value[state_name]
  sample_one.value = Math.round(population.value * 0.5)
  sample_two.value = Math.round(population.value * 0.25)
})

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
  population.value = 5024279;
  sample_one.value = 2512140;
  sample_two.value = 1256070;
  state.value = 'Alabama'
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
  const pop = Number(population.value)
  result.value = Math.round(sample_one.value * sample_two.value / r)
  difference.value = Math.abs(pop - result.value)
  percent_difference.value = Math.round((result.value - pop) / ((pop + result.value) / 2) * 100000) / 1000
  console.log(result.value, difference.value, population.value)
  answered.value = true
}
</script>
