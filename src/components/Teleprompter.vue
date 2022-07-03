<script setup>
import { reactive, ref, onMounted, watch } from 'vue';
import stringSimilarity from 'string-similarity';

const text =
  'Dies ist ein Typoblindtext. An ihm kann man sehen, ob alle Buchstaben da sind und wie sie aussehen. Manchmal benutzt man Worte wie Hamburgefonts, Rafgenduks oder Handgloves, um Schriften zu testen. Manchmal Sätze, die alle Buchstaben des Alphabets enthalten - man nennt diese Sätze »Pangrams«. Sehr bekannt ist dieser: The quick brown fox jumps over the lazy old dog.';

const words = reactive(text.split(' '));

var recog = reactive({});

var interimResults = ref('');

var progress = ref(0);

const SpeechRecognition =
  window.SpeechRecognition || window.webkitSpeechRecognition;
recog.current = new SpeechRecognition();
recog.current.continuous = true;
recog.current.interimResults = true;

const scrollRef = ref(null);

const startRecognition = () => {
  console.log('start recognition');

  recog.current.addEventListener('result', handleRecognitionResult);
  recog.current.start();
};

const stopRecognition = () => {
  console.log('stop recognition');

  recog.current.stop();
  recog.current.removeEventListener('result', handleRecognitionResult);
};

const handleRecognitionResult = ({ results }) => {
  const interim = Array.from(results)
    .filter((r) => !r.isFinal)
    .map((r) => r[0].transcript)
    .join(' ');

  interimResults.value += ' ' + interim;

  const newIndex = interim.split(' ').reduce((memo, word) => {
    if (memo.value >= words.length) {
      return memo;
    }

    console.log('words', word, memo.value, words[memo]);

    const similarity = stringSimilarity.compareTwoStrings(
      // cleanWord(word),
      // cleanWord(words[memo])
      word,
      words[memo]
    );

    memo.value += similarity > 0.75 ? 1 : 0;

    return memo;
  }, progress);

  if (newIndex > progress && newIndex <= words.length) {
    progress = newIndex;
  }

  console.log('handle recognition result: ', results, interim);
};

const cleanWord = (word) => {
  word
    .trim()
    .toLocaleLowerCase()
    .replace(/[^a-z]/gi, '');
};

watch(progress, (value) => {
  console.log('watch progress');

  scrollRef.value
    .querySelector(`[data-index='${progress + 3}']`)
    ?.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'start',
    });
});

// onMounted(() => {
//   console.log('on mounted');

//   console.log('scroll ref', scrollRef);

// });
</script>

<template>
  <h1>Teleprompter</h1>

  <div class="controls">
    <button @click="startRecognition">Start</button>
    <button @click="stopRecognition">Stop</button>
  </div>

  <div ref="scrollRef" class="prompterText">
    <span
      class="word"
      v-for="(word, index) in words"
      :key="index"
      :data-index="index"
      >{{ word }}
    </span>
  </div>

  <div class="interim">
    {{ interimResults }}
  </div>
</template>

<style scoped>
.controls {
  margin-bottom: 10px;
}

.word {
  margin-right: 5px;
}

.prompterText {
  word-break: break-word;
}

.interim {
  margin-top: 20px;
}
</style>
