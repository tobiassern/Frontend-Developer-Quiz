<template>
	<div id="app">
		<div class="loading loading-q" v-if="loading">
		</div>
		<div class="question-wrap">
			<div v-if="!loading && startGame && !showResult" class="question">
				<div class="quiz-info">
					<span>Question {{totalAnswers + 1}} / {{questionAmount}}</span>
				</div>
				<h4>
					<div class="chip-sm">
					    <span class="chip-name">{{question.category}}</span>
					</div>
					<span v-html="question.question"></span>
					<span v-if="question.helper">
						<div class="divider"></div>
						<small v-html="question.helper"></small>
					</span>
				</h4>
				<div class="form-group" v-if="question.type == 'radio'">
					<div v-for="(answer, index) in question.answers">
						<label class="form-radio">
		                    <input type="radio" v-bind:value="index" v-model="pickedAnswer">
		                    <i class="form-icon"></i> {{answer}}
		                </label>
		            </div>
		        </div>
				<div class="form-group" v-if="question.type == 'checkbox'">
					<div v-for="(answer, index) in question.answers">
						<label class="form-switch">
		                    <input type="checkbox" v-bind:value="index" v-model="pickedAnswer">
		                    <i class="form-icon"></i> {{answer}}
		                </label>
		            </div>
		        </div>
		        <div class="form-group" v-if="question.type == 'input'">
	            	<div class="form-group">
        			<input class="form-input" type="text" v-model="pickedAnswer" placeholder="Answer" />
    			</div>
		        </div>
	            <button class="btn btn-block" v-on:click="checkAnswer(question)">
	            	<span>Answer</span>
            	</button>
	        </div>
	        <div v-if="showResult">
		        <h4 class="text-center">Your result {{correct}} / {{totalAnswers}}</h4>
		        <h6 class="text-center">{{resultMessage}}</h6>
		        <div class="divider"></div>
		        <p class="text-center"><small>Want to play again?</small></p>
	        </div>
			<div v-if="!startGame" class="start-game-wrap">
				<div class="form-group">
					<select v-model="questionAmount" class="form-select select-lg">
						<option value="5">5 questions</option>
					  	<option value="10">10 questions</option>
					  	<option value="15">15 questions</option>
					  	<option value="20">20 questions</option>
					</select>
				</div>
				<button class="btn btn-block btn-lg" v-on:click="loadGame()">Start Game</button>
			</div>
		</div>
		<div class="toast-wrap" v-if="wrongAnswers">
			<div class="toast toast-danger">
	    		<button class="btn btn-clear float-right" v-on:click="closeWrongAnswer()"></button>
	    		<span class="icon icon-error_outline"></span>
	    		That answer was wrong
			</div>
		</div>
	</div>
</template>

<script>
export default {
  	name: 'questions',
  	data () {
    	return {
    		startGame: false,
    		questionAmount: 5,
    		loading: false,
      		correct: 0,
      		totalAnswers: 0,
      		wrongAnswers: false,
      		question: null,
      		allQuestions: null,
      		pickedAnswer: null,
      		showResult: false
    	}
  	},
  	computed: {
  		resultMessage: function() {
  			// Calculating the result message based on the percentage of correct answers
  			var resultPercent = this.correct / this.totalAnswers
  			if (resultPercent == 1) {
  				return "You are a true master and code ninja!"
  			} else if (resultPercent >= 0.8) {
  				return "You are truly good at coding, great job!"
  			} else if (resultPercent >= 0.6) {
  				return "You have coded for a while and are starting to get the hang of it"
			} else if (resultPercent >= 0.4) {
				return "You are still a beginner, but getting somewhere"
			} else if (resultPercent>= 0.2) {
  				return "You know some stuff, but you have a long way to go"
			} else {
				return "Did you even try?"
			}

  		}
  	},
  	methods: {
  		loadGame: function() {
  			// Reset the values needed to start a fresh game
  			this.startGame = true
  			this.loading = true
  			this.showResult = false
  			this.correct = 0
  			this.totalAnswers = 0
  			self.wrongAnswers = false
  			this.loadAllQuestions()
  		},
  		loadAllQuestions: function() {
  			self = this
  			// Fetching the data from the data.json file
  			$.getJSON("data/data.json",function (data) {
  				// Putting all the questions inside the data > questions

  				// To add new questions use the following format
  			// 	{
					// "question" : "What is the definition of CSS?",
					// "type" : "radio", // can be radio, checkbox or input
					// "correct" : 0, // Correct answer 0 = Cascading Style Sheet. If type == checkbox then this must be an array
					// "category" : "CSS",
					// "answers" : [
					// 		"Cascading Style Sheets",
					// 		"Computer Styled Site",
					// 		"Composed Styling Syntax",
					// 		"Cascading Style Syntax"
					// ] // answers is not needed on input, only correct
    	// 		},

  				self.allQuestions = data.questions
  				// Check if the amount of questions picked really exists
  				if(self.questionAmount > data.questions.length) {
  					alert("There are only " + data.questions.length + " questions in this set. But you picked " + self.questionAmount + " questions.")
  					// If there is less questions then the amount picked set the questionAmount to the total number of questions
  					self.questionAmount = data.questions.length
  				}
  				// Load question
  				self.loadQuestion()
  			});
  		},
  		loadQuestion: function() {
  			var self = this
  			// Check if there are any more questions to answer
  			if(self.totalAnswers + 1 <= self.questionAmount) {
				var pickQ = Math.floor(Math.random()*self.allQuestions.length)
				var question = self.allQuestions[pickQ]

				// Check what type of question it is
				if(question.type == 'radio' || question.type == 'input') {
					self.pickedAnswer = null
				} else if (question.type == 'checkbox') {
					self.pickedAnswer = [] // setting the self.pickedAnswer to [] otherwise it won't Vue.js won't count checkboxes
				}
				// Remove this question from the total pool of questions so it won't show again
				self.allQuestions.splice(pickQ, 1)
				// Set the vue data question to the question
				self.question = question
				// Turn of loading, this goes quickly so loading almost never shows
				self.loading = false
			} else {
				// IF no more questions then showResult
				self.showResult = true
				self.startGame = false
			}
  		},
  		checkAnswer: function(question) {
  			self = this
  			// Check type of question in order to correct it
  			if(question.type == 'radio' || question.type == 'input') {
				if(question.correct == self.pickedAnswer) {
					self.correct++
					self.wrongAnswers = false
				} else {
					self.wrongAnswers = true
				}
			} else if (question.type == 'checkbox') {
				// IF it is checkboxes transform the objects into array first
				var pickedAnswers = $.makeArray( self.pickedAnswer )
				var correctAnswers = $.makeArray(question.correct)

				//Then sort both arrays so they are comparable
				if(pickedAnswers.sort().join(',') === correctAnswers.sort().join(',')){
					self.correct++
					self.wrongAnswers = false
				} else {
					self.wrongAnswers = true
				}
			}
  			self.totalAnswers++
  			self.loadQuestion()
  		},
  		closeWrongAnswer: function() {
  			self.wrongAnswers = false
  		}
  	}
}
</script>

<style lang="sass">
.question-wrap {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	height: 100vh;
	width: 100%;
	max-width: 540px;
	margin: auto;
	.question {
		width: 100%;
		pre {
			font-size: 80%;
		}
	}
	.start-game-wrap {
		display: flex;
	}
}
.loading-q {
	min-height: 3.2rem;
	height: 100vh;
	position: fixed;
	width: 100%;
	top: 0;
	left: 0;
	&:before, &:after {
		content: '';
		border-radius: 1.6rem;
		height: 3.2rem;
		margin-left: -1.6rem;
		margin-top: -1.6rem;
		position: absolute;
	 	width: 3.2rem;
	 	top: 50%;
    	left: 50%;
	    border: .2rem solid;
	    border-radius: 1.6rem;
	}
	&:after {
		border-color: #5764c6;
	    border-right-color: transparent;
    	border-top-color: transparent;
	}
	&:before {
		border-color: #e0e0e0;
	}
}
.toast-wrap {
	position: fixed;
	right: 10px;
	bottom: 10px;
	width: 280px;
	max-width: 100%;
	.toast {
		margin-bottom: 10px;
		&:last-child {
			margin-bottom: 0;
		}
	}
}
</style>