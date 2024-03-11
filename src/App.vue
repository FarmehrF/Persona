<script setup>
import { ref, computed } from 'vue'

let ws = new WebSocket('ws://localhost:5001');

const questions = ref([
  {
	question: 'How was your weekend?',
	answer: 0,
	options: [
		'It was amazing!!',
		'Fine, Yours?',
		'Leave me alone!',
	],
	selected: null
  },
  {
	question: 'To prepare for a night out...',
	answer: 2,
	options: [
		'I get all my friends together to plan where we will dance the night away.',
		'Call a few of my closest friends to see if they will be there.',
		'Prepare? My friends have to drag me out most nights.',
	],
	selected: null
  },
  {
	question: 'Being around people makes me feel...',
	answer: 1,
	options: [
		"Like I'm alive!",
		"Inspired. I feed off of others' energy but there are times when I'd rather be alone.",
		"A bit exhausted. Being around others can be draining.",
	],
	selected: null
  },
  {
	question: "When given a choice between working as part of a team or working as a group, I would prefer to...",
	answer: 1,
	options: [
	"Work with as many people as possible.",
	"Work as part of a small group.",
	"Work by myself.",
	],
	selected: null
  },
    {
	question: "What's your idea of the perfect date?",
	answer: 1,
	options: [
	"A live concert in Central Park.",
	"Dinner and a Broadway show.",
	"Wine and Netflix.",
	],
	selected: null
  },
    {
	question: "During parties or social gatherings, I tend to...",
	answer: 1,
	options: [
	"Talk to as many people as I can. I've been called a social butterfly.",
	"Spend time with a few people that I know. It's about quality not quanitity.",
	"Keep to myself. You can find me planted on the wall or by the punch bowl.",
	],
	selected: null
  },
    {
	question: "When I'm dealing with a personal problem, I prefer to...",
	answer: 1,
	options: [
	"Talk about it with others and enlist their help.",
	"Call my closest friend and discuss on the phone.",
	"Solve it on my own. I keep my private life private.",
	],
	selected: null
  },
  {
	question: "On the topic of public speaking...",
	answer: 1,
	options: [
	"I love it. All eyes on me!",
	"It's not my strongest suit but I'll do it if I have to.",
	"I get super nervous. I'd rather write 5 research papers.",
	],
	selected: null
  },
  {
	question: "Do people describe you as thoughtful, reserved, and soft-spoken?",
	answer: 1,
	options: [
	"Not a chance, I am a very talkative person.",
	"Sometimes, but I can be quite talkative with my inner circle.",
	"All the time, I am a very quiet and mellow person.",
	],
	selected: null
  },
  {
	question: "Do you enjoy meeting new people?",
	answer: 1,
	options: [
	"Yes, I make a new friend every time I go grocery shopping.",
	"Sometimes, but I have to be in the right mood.",
	"I rarely find myself in position to meet anyone new.",
	],
	selected: null
  },
  {
	question: "How do you prefer to communicate?",
	answer: 1,
	options: [
	"In person. Whatever you have to say, say it to my face.",
	"It depends on the situation but I can be flexible.",
	"Just text me please!",
	],
	selected: null
  },
])

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
						:disabled="getCurrentQuestion.selected"
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