<script setup>
  import { $vfm, VueFinalModal, ModalsContainer } from 'vue-final-modal'
  import { ref, onUnmounted, onMounted, computed } from 'vue'
  import Board from './components/Board.vue'
  import Keyboard from './components/Keyboard.vue'
  import { PencilIcon, QuestionMarkCircleIcon, LightBulbIcon, XIcon, ChartBarIcon, RefreshIcon } from '@heroicons/vue/outline'
  import { acceptedWordList } from './assets/js/accepted_word_list.js'
  import { wordList } from './assets/js/wordlist.js'

  function CalculateWordColors(answer, guess) {
    answer = answer.toLowerCase();
    guess = guess.toLowerCase();

    let len = answer.length;
    if (guess.length != len) {
        throw "Guess and answer must be the same length";
    }

    let numExpectedCounts = new Array(26).fill(0);
    for (let i = 0; i < len; i++) {
        let answerChar = answer[i];
        let guessChar = guess[i];
        if (answerChar != guessChar) {
            let answerIndex = answerChar.charCodeAt(0) - 'a'.charCodeAt(0);
            numExpectedCounts[answerIndex]++;
        }
    }

    let numSeenCounts = new Array(26).fill(0);
    let colors = '';
    for (let i = 0; i < len; i++) {
        let guessChar = guess[i];
        let answerChar = answer[i];
        if (guessChar < 'a' || guessChar > 'z') {
            colors += 'x';
            continue;
        }

        if (guessChar == answerChar) {
            colors += 'g';
        } else {
            let guessIndex = guessChar.charCodeAt(0) - 'a'.charCodeAt(0);
            let numExpected = numExpectedCounts[guessIndex];
            let numSeen = numSeenCounts[guessIndex];
            numSeenCounts[guessIndex]++;

            if (numSeen < numExpected) {
                colors += 'y';
            } else {
                colors += 'x';
            }
        }
    }

    return colors;
  }


  function GetFairAnswer(answerList, forcedGuesses) {
    if (answerList.length == 0 || forcedGuesses.length == 0) {
        throw "No answers or forced guesses";
    }

    let colorsMap = new Map();
    for (let answer of answerList) {
        if (forcedGuesses.includes(answer)) {
            continue;
        }

        // Calculate the colors for this answer and concatinate them into one string
        let answerColors = forcedGuesses.map(guess => CalculateWordColors(answer, guess)).join('');

        // Insert or update the colorsMap
        if (colorsMap.has(answerColors)) {
            colorsMap.get(answerColors).push(answer);
        } else {
            colorsMap.set(answerColors, [answer]);
        }
    }

    let fairAnswers = [];
    for (let [_, answers] of colorsMap) {
        if (answers.length > 1 && answers.length <= 4) {
            fairAnswers.push.apply(fairAnswers, answers);
        }
    }

    if (fairAnswers.length == 0) {
        return '';
    }

    // Return a random fair answer
    return fairAnswers[Math.floor(Math.random() * fairAnswers.length)];
  }

  const genNewWords = function() {
    while ( words.length < 5 ){
      words = []

      while ( words.length < 4 ) {
        let word = wordList[getRandomInt()]
        if ( !words.includes(word) ) {
          words.push(word)
        }
      }
      let fair = GetFairAnswer(wordList,words)
      if ( fair != '' ) {
        words.push(fair)
      }
    }
  }

  let words = []
  genNewWords()

  let keyboardRows = ref([
    [
      {
        'letter': 'q',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'w',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'e',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'r',
        'state': 1,
        'colored': true
      },
      {
        'letter': 't',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'y',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'u',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'i',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'o',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'p',
        'state': 1,
        'colored': true
      }
    ],
    [
      {
        'letter': 'a',
        'state': 1,
        'colored': true
      },
      {
        'letter': 's',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'd',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'f',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'g',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'h',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'j',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'k',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'l',
        'state': 1,
        'colored': true
      },
      {
        'letter': '_',
        'state': 1,
        'colored': true
      }
    ],
    [
      {
        'letter': 'enter',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'z',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'x',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'c',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'v',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'b',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'n',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'm',
        'state': 1,
        'colored': true
      },
      {
        'letter': 'del',
        'state': 1,
        'colored': true
      }
    ]
  ])

  const wordLength = words[0].length > 0 ? words[0].length : 0
  let numberOfGuesses = 6
  let showWinModal = ref(false)
  let showHelpModal = ref(false)
  let showConfirmModal = ref(false)
  let allGuesses = ref(Array())

  const loadGuesses = function() {
    allGuesses.value.length = 0
    allGuesses.value.push([])
    for ( let i=1; i < words.length; i++ ) {
      let guesses = []
      if ( i < words.length - 1 ) {
        let letters = []
        for ( let x=0; x < wordLength; x++ ) {
          letters.push({ 'letter': words[i-1].charAt(x), 'state': 0, 'initialized': true, 'colored': true })
        }
        guesses.push(letters)
      } else {
        for ( let x=0; x < words.length - 1; x++ ){
          let guess = []
          for ( let j=0; j < wordLength; j++ ) {
            guess.push({ 'letter': words[x].charAt(j), 'state': 0, 'initialized': true, 'colored': true })
          }
          guesses.push(guess)
        }
      }
      allGuesses.value.push(guesses)
    }
    for ( let i=0; i < allGuesses.value.length; i++ ) {
      for ( let x=allGuesses.value[i].length; x <= numberOfGuesses - 1; x++ ){
        let initialGuess = []
        for ( let j=0; j < wordLength; j++ ) {
          initialGuess.push({ 'letter': '', 'state': 0, 'initialized': false, 'colored': false, 'completed': false })
        }
        allGuesses.value[i].push(initialGuess)
      }
    }
  }
  loadGuesses()
  let currentGuess = ref(0)
  let currentGame = ref(0)
  let playerGuessCount = ref(1)
  let guessCounts = ref(Array())
  let givenWordCount = 0
  let currentPosition = ref(0)
  let gameResults = ref('')
  let finished = ref(false)
  let correct = ref(false)
  let notInDictionary = ref(false)
  let gaveUp = ref(false)
  let totalWins = ref(0)
  let totalGames = ref(0)
  let currentStreak = ref(0)
  let maxStreak = ref(0)
  let remainingHours = ref(0)
  let remainingMinutes = ref(0)
  let remainingSeconds = ref(0)
  let showModal = ref(false)
  let msg = computed(() => {
    if ( !finished.value ) {
      return ''
    }
    return correct.value ? 'Nice!' : 'Too Bad!'
  })

  let boardOneDisabled = computed(() => {
    return false
  })
  let boardTwoDisabled = computed(() => {
    if ( finished.value ) {
      if ( currentGame.value > 0 ) {
        return false
      }
    } else {
      if ( currentGame.value > 0 ) {
        return false
      }
    }
    return true
  })
  let boardThreeDisabled = computed(() => {
    if ( finished.value ) {
      if ( currentGame.value > 1 ) {
        return false
      }
    } else {
      if ( currentGame.value > 1 ) {
        return false
      }
    }
    return true
  })
  let boardFourDisabled = computed(() => {
    if ( finished.value ) {
      if ( currentGame.value > 2 ) {
        return false
      }
    } else {
      if ( currentGame.value > 2 ) {
        return false
      }
    }
    return true
  })
  let boardFiveDisabled = computed(() => {
    if ( finished.value ) {
      if ( currentGame.value > 3 ) {
        return false
      }
    } else {
      if ( currentGame.value > 3 ) {
        return false
      }
    }
    return true
  })

  const completeRow = function(skipAnimation,skipKeyboard) {
    let word = words[currentGame.value]
    let skip = skipAnimation ? 0 : 1
    let guess = allGuesses.value[currentGame.value][currentGuess.value]
    for ( let i = 0; i < guess.length; i++){
      if ( guess[i]['letter'] === '' ) {
        return
      }
    }

    let answerLetters = word.split('')
    let playerAnswer = guess.map((e) => e['letter']).join('')
    correct.value = ( playerAnswer === word )
    if ( !correct.value && !acceptedWordList.includes(playerAnswer.toUpperCase()) && !skipAnimation ){
      showWordMissingMessage()
      return
    }
    let changedLetters = []
    let keyboardUpdates = []
    const colors = CalculateWordColors(word,playerAnswer)
    const states = [ '', '', 'x', 'y', 'g' ]

    for ( let i=0; i < colors.length; i++ ) {
      let state = states.indexOf(colors.charAt(i))
      if ( !skipKeyboard ){
        keyboardUpdates.push([guess[i]['letter'],state])
      }
      setTimeout( () => {
        guess[i]['state'] = state
      }, 150 * (i) * skip)
      setTimeout( () => {
        guess[i]['colored'] = true
      }, (450 + 150 * (i)) * skip)

    }

    setTimeout( () => {
      for ( let i=0; i < keyboardUpdates.length; i++ ) {
        updateKeyboard(keyboardUpdates[i][0],keyboardUpdates[i][1])
      }
      if ( finished.value ) {
        showWinModal.value = true
      }
    }, 300 * guess.length * skip)

    if ( correct.value ) {
      if (currentGame.value == words.length - 1){
        finished.value = true
        currentPosition.value = -1
        currentGuess.value = -1
      } else {
        currentPosition.value = 0
        currentGuess.value = -1
        setTimeout(() => {
          currentPosition.value = 0
          currentGuess.value = 0
          currentGame.value++
          resetKeyboard()
          completeRow(true)
          if ( currentGame.value == 4 ) {
            completeRow(true)
            completeRow(true)
            completeRow(true)
          }
        },2500)
      }
      return
    }

    currentGuess.value++
    currentPosition.value = 0
    if (!skipAnimation ){
      if (currentGuess.value == numberOfGuesses ){
        finished.value = true
        showWinModal.value = true
      }
    }
  }

  const resetKeyboard = function (){
    for ( let i=0; i < 3; i++ ) {
      for ( let x=0; x < keyboardRows.value[i].length; x++ ){
        keyboardRows.value[i][x]['state'] = 1
      }
    }
  }

  const updateKeyboard = function(letter,state) {
    let keyPosition = findKey(letter)
    if ( keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] < state ){
      keyboardRows.value[keyPosition[0]][keyPosition[1]]['state'] = state
    }
  }

  const findKey = function(letter) {
    for ( let i=0; i < 3; i++ ) {
      for ( let x=0; x < keyboardRows.value[i].length; x++ ){
        if ( letter === keyboardRows.value[i][x]['letter'] ) {
          return [i,x]
        }
      }
    }
  }

  const onKeyup = (e) => onKey(e.key)
  const tileClick = function(e) {
    if ( finished.value ){
      return
    }
    currentPosition.value = e.detail
  }
  window.addEventListener('keyup', onKeyup)
  const tileClickEvent = new Event('tile-click');
  window.addEventListener('tile-click',tileClick)
  onUnmounted(() => {
    window.removeEventListener('keyup', onKeyup)
    window.removeEventListener('tile-click',tileClick)
  })

  const onKey = function(key) {
    if (showModal.value || finished.value ) {
      return
    }
    if (/^[a-zA-Z_\-]$/.test(key)) {
      if (key == '-') {
        key = '_'
      }
      fillTile(key.toLowerCase())
    } else if (key === 'Backspace' || key === 'Delete') {
      clearTile()
    } else if (key === 'Enter') {
      completeRow(false)
    } else if ( key == 'ArrowLeft' && currentPosition.value > 0 ) {
      currentPosition.value--
    } else if ( ( key == 'ArrowRight' || key == ' ' ) && currentPosition.value < wordLength - 1 ) {
      currentPosition.value++
    }
  }

  const fillTile = function(key){
    let tile = {}
    if ( currentPosition.value == wordLength ){
      tile = allGuesses.value[currentGame.value][currentGuess.value][currentPosition.value - 1]
    } else {
      tile = allGuesses.value[currentGame.value][currentGuess.value][currentPosition.value]
    }
    tile['letter'] = key
    if ( currentPosition.value < wordLength - 1  ){
      currentPosition.value++
    }
  }

  const clearTile = function() {
    if ( currentPosition.value > 0 && allGuesses.value[currentGame.value][currentGuess.value][currentPosition.value]['letter'] == '' ){
      currentPosition.value--
    }
    let tile = allGuesses.value[currentGame.value][currentGuess.value][currentPosition.value]
    tile['letter'] = ''
  }

  const guessNotInDictionary = computed(() => {
    if ( currentGame.value < 0 || currentGame.value > 4 || currentGuess.value < 0 || finished.value ){
      return false
    }
    let playerAnswer = allGuesses.value[currentGame.value][currentGuess.value].map((e) => e['letter']).join('')
    if ( words[currentGame.value] == playerAnswer ){
      return false
    }

    if ( playerAnswer.length != wordLength || playerAnswer.match(/_/) ) {
      return false;
    }

    return !acceptedWordList.includes(playerAnswer.toUpperCase())
  })

  const showWordMissingMessage = function() {
    notInDictionary.value = true
    setTimeout(() => {
      notInDictionary.value = false
    },1500)
  }

  const playAgain = function() {
    resetKeyboard()
    currentGame.value = 0
    currentGuess.value = 0
    currentPosition.value = 0
    correct.value = false
    finished.value = false
    words = []
    genNewWords()
    loadGuesses()
    showWinModal.value = false
  }

  const giveUp = function() {
    showConfirmModal.value = false;
    finished.value = true
    gaveUp.value = true
    showWinModal.value = true
  }

  const openWinModal = function() {
    if ( !finished.value ){
      return
    }
    showWinModal.value = true
  }

  function getRandomInt() {
    let min = Math.ceil(0);
    let max = Math.floor(wordList.length);
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }
</script>

<template>
  <div class="container wrap">
    <div class="header row">
      <div class="col-md-3">
      </div>
      <div class="col-md-6">
        <span class="title">Gauntletle</span>
      </div>
      <div class="col-md-3 help">
        <XIcon class="give-up-icon" @click="showConfirmModal = (true && !finished)"></XIcon>
        <ChartBarIcon @click="openWinModal" :class="{ inactive: !finished }" ></ChartBarIcon>
        <QuestionMarkCircleIcon @click="showHelpModal = true"></QuestionMarkCircleIcon>
      </div>
    </div>
    <div class="info">
      <h5 class="warning-message" :class="{'shown': notInDictionary}">Word not in dictionary.</h5>
    </div>
    <div class="row">
      <div class="col-2"></div>
      <ul class="nav nav-pills nav-justified col-8" id="boardsNav" role="tablist">
        <li class="nav-item" role="presentation">
          <button class="nav-link" :class="{'active': currentGame == 0}" id="board1" data-bs-toggle="pill" data-bs-target="#board-one-pane" type="button" role="tab" aria-controls="board-one-pane" aria-selected="true" :disabled="boardOneDisabled">1</button>
        </li>
        <li class="nav-item" role="presentation">
          <button class="nav-link" :class="{'active': currentGame == 1}" id="board2" data-bs-toggle="pill" data-bs-target="#board-two-pane" type="button" role="tab" aria-controls="board-two-pane" aria-selected="false" :disabled="boardTwoDisabled">2</button>
        </li>
        <li class="nav-item" role="presentation">
          <button class="nav-link" :class="{'active': currentGame == 2}" id="board3" data-bs-toggle="pill" data-bs-target="#board-three-pane" type="button" role="tab" aria-controls="board-three-pane" aria-selected="false" :disabled="boardThreeDisabled">3</button>
        </li>
        <li class="nav-item" role="presentation">
          <button class="nav-link" :class="{'active': currentGame == 3}" id="board4" data-bs-toggle="pill" data-bs-target="#board-four-pane" type="button" role="tab" aria-controls="board-four-pane" aria-selected="false" :disabled="boardFourDisabled">4</button>
        </li>
        <li class="nav-item" role="presentation">
          <button class="nav-link" :class="{'active': currentGame == 4}" id="board5" data-bs-toggle="pill" data-bs-target="#board-five-pane" type="button" role="tab" aria-controls="board-five-pane" aria-selected="false" :disabled="boardFiveDisabled">5</button>
        </li>
      </ul>
      <div class="col-2"></div>
      <div class="tab-content" id="boards">
        <div class="tab-pane fade" :class="{'active': currentGame == 0, 'show': currentGame == 0}" id="board-one-pane" role="tabpanel" aria-labelledby="board-one" tabindex="-1">
          <Board :guesses="allGuesses[0]" :guessNotInDictionary="guessNotInDictionary" :currentGuess="currentGuess" :currentPosition="currentPosition" :wordLength="wordLength"></Board>
        </div>
        <div class="tab-pane fade" :class="{'active': currentGame == 1, 'show': currentGame == 1}" id="board-two-pane" role="tabpanel" aria-labelledby="board-two" tabindex="-1">
          <Board :guesses="allGuesses[1]" :guessNotInDictionary="guessNotInDictionary" :currentGuess="currentGuess" :currentPosition="currentPosition" :wordLength="wordLength"></Board>
        </div>
        <div class="tab-pane fade" :class="{'active': currentGame == 2, 'show': currentGame == 2}" id="board-three-pane" role="tabpanel" aria-labelledby="board-three" tabindex="-1">
          <Board :guesses="allGuesses[2]" :guessNotInDictionary="guessNotInDictionary" :currentGuess="currentGuess" :currentPosition="currentPosition" :wordLength="wordLength"></Board>
        </div>
        <div class="tab-pane fade" :class="{'active': currentGame == 3, 'show': currentGame == 3}" id="board-four-pane" role="tabpanel" aria-labelledby="board-four" tabindex="-1">
          <Board :guesses="allGuesses[3]" :guessNotInDictionary="guessNotInDictionary" :currentGuess="currentGuess" :currentPosition="currentPosition" :wordLength="wordLength"></Board>
        </div>
        <div class="tab-pane fade" :class="{'active': currentGame == 4, 'show': currentGame == 4}" id="board-five-pane" role="tabpanel" aria-labelledby="board-five" tabindex="-1">
          <Board :guesses="allGuesses[4]" :guessNotInDictionary="guessNotInDictionary" :currentGuess="currentGuess" :currentPosition="currentPosition" :wordLength="wordLength"></Board>
        </div>
      </div>
    </div>
    <Keyboard :rows="keyboardRows"></Keyboard>
    <div class="footer">
      Created by theasylm
    </div>
    <vue-final-modal
        name="winModal"
        classes="modal-container"
        :click-to-close="false"
        :esc-to-close="true"
        v-model="showWinModal"
        content-class="modal-content"
        :max-width="500"
      >
      <div class="close-modal-div">
        <XIcon @click="showWinModal = false"></XIcon>
      </div>
      <span class="modal__title" id="win-msg">{{msg}}</span>
      <div class="modal__content">
        <div class="solution" v-if="finished && !correct">
          Solutions:
          <span class="solution-word" v-for="(word,i) in words">{{i + 1}}: {{word.toUpperCase()}}</span>
        </div>
        <button class="btn btn-primary" @click="playAgain">Play Again</button>
      </div>
    </vue-final-modal>
    <vue-final-modal
      name="giveUpModal"
      classes="modal-container"
      :click-to-close="false"
      :esc-to-close="true"
      v-model="showConfirmModal"
      content-class="modal-content"
      :max-width="500"
    >
      <div class="close-modal-div">
        <XIcon @click="showConfirmModal = false"></XIcon>
      </div>
      <div class="modal__content confirm">
        <div class="hint-div">
          <div class="warning">Are you sure you wish to give up?</div>
        </div>
      </div>
      <div class="modal__action">
        <div class="mb-3 row">
          <div class="col-sm-4">
            <button @click="showConfirmModal = false" class="btn btn-primary">Keep Thinking</button>
          </div>
          <div class="col-sm-4">
            <button @click="giveUp" class="btn btn-danger">Give Up</button>
          </div>
        </div>
      </div>
    </vue-final-modal>
    <vue-final-modal
        name="helpModal"
        classes="modal-container help-modal"
        :click-to-close="false"
        :esc-to-close="true"
        v-model="showHelpModal"
        content-class="modal-content"
        :max-width="600"
      >
        <div class="close-modal-div">
          <XIcon @click="showHelpModal = false"></XIcon>
        </div>
        <div class="modal__content">
          <h2>How to Play</h2>
          <div class="mb-3 row">
            <div class="col-sm-12">
              Guess the five words in the given number of tries. After each guess, the tiles will be colored to indicate how close to the target word your guess was.
              <img src="./assets/green_clue.png"/>
              Green indicates the N is in the correct spot.
              <img src="./assets/yellow_clue.png"/>
              Yellow indicates the U is in the word, but in another position.
              <p><img src="./assets/grey_clue.png"/>
              Grey indicates the P is not in the word.</p>
              <p>After the first word, you will be seeded with the previous word as your starting guess.</p>
              <p>On the final word, you will be seeded with the previous four words as starting guesses.</p>
              <p>If you wish to give up, you can hit the red X. You will be asked to confirm your decision.</p>
              <hr/>
              <small>Special thanks for Rangsk for the extra code!</small>
            </div>
          </div>
        </div>
      </vue-final-modal>
  </div>
</template>

<style>
  body, html {
    height: 100%;
  }
  body {
    overflow: hidden;
    margin: 0px;
  }
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #efefef;
    /**background-color: #08111b;**/
    background-color: #011637;
    height: 100%;
    padding-top: 1em;
  }
  .nav-link {
    color: #efefef !important;
  }
  .title {
    font-size: 2em;
  }
  .info {
    position: relative;
  }
  .warning-message {
    visibility: hidden;
    margin: 0;
  }
  .warning-message.shown {
    visibility: visible;
  }
  #copiedMessage, #copiedResultsMessage, #copiedJustMessage {
    visibility: hidden;

  }
  #copiedMessage.shown, #copiedResultsMessage.shown, #copiedJustMessage.shown {
    visibility: visible;
    opacity: 0;
    animation: fade 2s linear forwards;
  }
  .share-button {
    margin-top: 2rem;
  }
  .close-modal-div, .reset-modal {
    position: absolute;
    cursor: pointer;
  }

  .close-modal-div {
    margin-top: -.5rem;
    width: 38px;
    right: .5rem;
  }

  .reset-modal {
    margin-top: -.45rem;
    width: 32px;
    right:  42px;
  }

  .left-align {
    text-align: left;
  }
  .help svg {
    width: 36px;
    margin: 0 .25rem;
    cursor: pointer;
  }
  .help svg.inactive {
    opacity: 50%;
    cursor:  default;
  }
  .help-modal img {
    width: 100%;
  }
  .help-modal h2 {
    text-align: center;
  }
  .help-modal hr {
    height: 2px;
    background-color: #efefef;
    border: none;
    opacity: 1;
    margin-bottom: 0;
  }
  #boardsNav {
    margin-bottom: 1rem;
  }
  .solution {
    font-size: 1.5rem;
    font-weight: 500;
    padding-bottom: 1rem;
  }
  @keyframes fade {
    0%,100% { opacity: 0 }
    50% { opacity: 1 }
  }
  .add-link {
    cursor:  pointer;
    color: #efefef;
    text-decoration: none;
  }
  .add-link:hover {
    color: #efefef;
  }

  .info span {
    margin: 0 .5rem;
    font-size:  1.25rem;
  }
  table.table {
    color: #efefef;
    border-top: 2px solid;
    border-bottom: 2px solid;
  }
  .has-error {
    border-color: #842029 !important;
    border-width: 4px;
  }
  .hint-span {
    font-size:  2.5rem;
  }
  .hint-div {
    min-height: 10rem;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  #win-msg {
    font-size:  2.25rem;
    margin-bottom: 1.5rem;
  }
  .solution-word {
    display: block;
  }
  .give-up-icon {
    color:  #842029;
    width:  46px !important;
  }
  .warning {
    font-size: 2rem;
  }
  .slider {
    display: flex;
    align-items: center;
  }
  .left {
    justify-content: left;
    display: flex;
  }
  .footer {
    position: absolute;
    bottom: 0.5rem;
    font-size: smaller;
    width: 100%;
    left: 0;
  }
</style>
<style scoped>
  ::v-deep .modal-content {
    position: relative;
    display: flex;
    flex-direction: column;
    max-height: 90%;
    margin: 0 1rem;
    padding: 1rem;
    border: 1px solid #efefef;
    border-radius: 0.25rem;
    background: #011637;
    width: 700px;
    min-height: 10rem;
  }
  .modal__content {
    scrollbar-color: white;
  }
  .modal__content::-webkit-scrollbar {
    scrollbar-color: white;
  }
  ::v-deep .help-modal .modal-content {
    width:  650px;
    text-align: left;
  }
  .help-modal ul {
    list-style:  none;
  }
  .modal__title {
    font-size: 1.5rem;
    font-weight: 700;
  }
  ::v-deep .modal-container {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .modal__content {
    flex-grow: 1;
    overflow-y: auto;
  }
  .modal__close {
    position: absolute;
    top: 0.5rem;
    right: 1.5rem;
  }
  .modal__action {
    padding: 1rem 0 0;
  }
  .new-button {
    width: 10rem;
  }
  .new-button svg {
    float: right;
    width: 24px;
  }
  @media (max-width: 584px){
    .newModal .btn {
      margin: .5rem 0;
      height: 4rem;
    }
    .left .tooltip-icon {
      margin-top: 1.75rem;
    }
    .left button {
      width: 90%;
    }

  }

  #boards div {
    outline: none;
  }
</style>

