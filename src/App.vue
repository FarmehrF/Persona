<script setup>
import { ref, computed } from 'vue'

let ws = new WebSocket('wss://websocketserver-0d0561efc4b0.herokuapp.com:443');

const questions = ref([
  {
    question: 'How do you typically feel about recent events?',
    answer: 0,
    options: [
      'Really positive!',
      'Neutral.',
      'Prefer solitude to reflect.'
    ],
    selected: null
  },
  {
    question: 'How do you approach preparing for new experiences?',
    answer: 0,
    options: [
      'Collaborate with others for planning.',
      'Seek input from close contacts.',
      'Embrace spontaneity or reluctance to plan.'
    ],
    selected: null
  },
  {
    question: 'How do social interactions impact your mood?',
    answer: 0,
    options: [
      'Energizing and uplifting.',
      'Inspiring but occasionally overwhelming.',
      'Draining, prompting a need for solitude.'
    ],
    selected: null
  },
  {
    question: 'What\'s your preference in collaborative settings?',
    answer: 0,
    options: [
      'Enjoy working with large, diverse teams.',
      'Prefer smaller, more intimate groups.',
      'Prefer individual work.'
    ],
    selected: null
  },
  {
    question: 'Describe your ideal outing:',
    answer: 0,
    options: [
      'Vibrant and communal.',
      'Culturally enriching and intimate.',
      'Quiet and private.'
    ],
    selected: null
  },
  {
    question: 'How do you typically behave in social gatherings?',
    answer: 0,
    options: [
      'Sociable, engaging with many.',
      'Selective, focusing on quality interactions.',
      'Observant, keeping to oneself.'
    ],
    selected: null
  },
  {
    question: 'How do you address personal challenges?',
    answer: 0,
    options: [
      'Seek support and collaboration.',
      'Rely on trusted confidants.',
      'Prefer to solve problems independently.'
    ],
    selected: null
  },
  {
    question: 'Thoughts on public speaking?',
    answer: 0,
    options: [
      'Enjoy the attention and engagement.',
      'Willing to do it if necessary.',
      'Feel nervous and uncomfortable.'
    ],
    selected: null
  },
  {
    question: 'How do others perceive your social demeanor?',
    answer: 0,
    options: [
      'Outgoing and expressive.',
      'Balanced between outgoing and reserved.',
      'Reserved and contemplative.'
    ],
    selected: null
  },
  {
    question: 'Do you enjoy meeting new people?',
    answer: 0,
    options: [
      'Embrace new connections eagerly.',
      'Open to new interactions depending on circumstance.',
      'Prefer familiarity over new acquaintances.'
    ],
    selected: null
  },
  {
    question: 'Preferred communication style:',
    answer: 0,
    options: [
      'Value face-to-face interactions.',
      'Adapt communication as needed.',
      'Prefer written communication for convenience.'
    ],
    selected: null
  }
]);


const quizCompleted = ref(false)
const currentQuestion = ref(0)
const score = computed(() => {
	let value = 0
	questions.value.map(q => {
		// if (q.selected != null && q.answer == q.selected) {
      if (q.selected != null) {
			// console.log(q.selected);
			value = value + parseInt(q.selected)
		}
	})
	return value
})

const getCurrentQuestion = computed(() => {
	let question = questions.value[currentQuestion.value]
	question.index = currentQuestion.value
	return question
})

const SetAnswer = (e) => {
	questions.value[currentQuestion.value].selected = e.target.value
	e.target.value = null
}

const NextQuestion = () => {
	if (currentQuestion.value < questions.value.length - 1) {
		currentQuestion.value++
		return
	}
	
	quizCompleted.value = true
	ws.send(JSON.stringify({ 'Score': score.value / 22 }));
}

ws.addEventListener('open', (event) => {
  console.log('Socket connection open');
  // alert('Successfully connected to socket server 🎉');
  ws.send('pong');
});


ws.addEventListener('message', (message) => {
  if (message && message.data) {
    if (message.data === "ping") {
      console.log("got ping");
      ws.send("pong");
      return;
    }
    let data = JSON.parse(message.data);
    if (data) {
      if ("slider1" in data) {
        controlledByTD.value = data["slider1"] * 100;
      }
      console.log("got data", data);
    }
  }
  console.log("message", message)
});

ws.addEventListener('error', (error) => {
    console.error('Error in the connection', error);
    alert('error connecting socket server', error);
});

ws.addEventListener('close', (event) => {
    console.log('Socket connection closed');
    alert('closing socket server');
});




const textInput = ref('');

const sendMessage = () => {
	const message = textInput.value.trim();
	if (message !== '') {
		// window.location.reload();
		ws.send(JSON.stringify({ 'Name': message }))
  };
}







</script>

<template>
	<main class="app">
		<h1>Persona</h1>
		
		<section class="quiz" v-if="!quizCompleted">
			<div class="quiz-info">
				<span class="question">{{ getCurrentQuestion.question }}</span>
			</div>
			
			<div class="options">
				<label 
					v-for="(option, index) in getCurrentQuestion.options" 
					:for="'option' + index" 
					:class="`option ${
						getCurrentQuestion.selected == index 
							? index == getCurrentQuestion.answer 
								? 'correct' 
								: 'wrong'
							: ''
					} ${
						getCurrentQuestion.selected != null &&
						index != getCurrentQuestion.selected
							? 'disabled'
							: ''
					}`">
					<input 
						type="radio" 
						:id="'option' + index" 
						:name="getCurrentQuestion.index" 
						:value="index" 
						v-model="getCurrentQuestion.selected" 
						@change="SetAnswer" 
					/>
					<span>{{ option }}</span>
				</label>
			</div>
			
			<button 
				@click="NextQuestion" 
				:disabled="!getCurrentQuestion.selected">
				{{ 
					getCurrentQuestion.index == questions.length - 1 
						? 'Finish' 
						: getCurrentQuestion.selected == null
							? 'not ready yet?'
							: 'are you sure?'
				}}
			</button>
		</section>

		<section v-else>
			<h2>Manifestation in process</h2>
    		<input type="text" :value="textInput" @input="textInput = $event.target.value" placeholder="Give it a name...">
			<button @click="sendMessage">Send</button>
		<!-- <p>Your score is {{ score }}/{{ questions.length }}</p> -->
		</section>
   
  </main>

</template>

<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: Arial, Helvetica;
}

body {
	background-color: #000;
	color: #FFF;
}

.app {
	display: flex;
	flex-direction: column;
	align-items: center;
	padding: 2rem;
	height: 50vh;
}

h1 {
	font-size: 1.8rem;
	margin-bottom: 2rem;
}

.quiz {
	background-color: #070707;
	padding: 1rem;
	width: 100%;
	max-width: 640px;
}

.quiz-info {
	display: flex;
	justify-content: space-between;
	margin-bottom: 1rem;
}

.quiz-info .question {
	color: #FFF;
	font-size: 1.25rem;
}

.quiz-info.score {
	color: #000;
	font-size: 1.25rem;
}

.options {
	margin-bottom: 1rem;
}

.option {
	padding: 1rem;
	display: block;
	background-color: #141414;
	margin-bottom: 0.5rem;
	border-radius: 0.5rem;
	cursor: pointer;
}

.option:hover {
	background-color: #414141;
}

.option.correct {
	background-color: #B0B0B0;
}

.option.wrong {
	background-color: #B0B0B0;
}

.option:last-of-type {
	margin-bottom: 0;
}

.option.disabled {
	opacity: 0.5;
}

.option input {
	display: none;
}

.custom-input {
  padding: 1rem;
  border: 0.2rem solid #ccc;
  border-radius: 0.5rem;
  font-size: 1rem;
}

.custom-input:focus {
  outline: none;
  border-color: #007bff;
}

button {
	appearance: none;
	outline: none;
	border: none;
	cursor: pointer;
	padding: 0.5rem 0.5rem;
	background-color: #FF0000;
	color: #fff;
	font-weight: 700;
	text-transform: none;
	font-size: 1.2rem;
	border-radius: 0.5rem;
}

button:disabled {
	opacity: 0.5;
}

input[type="text"]{
  color: #FFF;
  width: 100%;
  height: 20%;
  box-sizing: border-box;
  letter-spacing: 1px;
  outline:none;
  padding: 1rem 1rem;
  margin-bottom: 1.5rem;
}

h2 {
	font-size: 2rem;
	margin-bottom: 2rem;
	text-align: center;
}

p {
	color: #8F8F8F;
	font-size: 1.5rem;
	text-align: center;
}

</style>