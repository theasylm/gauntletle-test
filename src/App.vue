<script setup>
  import { $vfm, VueFinalModal, ModalsContainer } from 'vue-final-modal'
  import { ref, onUnmounted, onMounted, computed } from 'vue'
  import CryptoJS from 'crypto-js'
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

  const encodeGame = function() {
    let buffer = new ArrayBuffer(10)
    let bufferView16 = new Uint16Array(buffer)
    for ( let i=0; i < words.value.length; i++ ){
      bufferView16[i] = wordList.indexOf(words.value[i])
    }
    let bufferView8 = new Uint8Array(buffer)
    let binary = '';
    let len = bufferView8.byteLength;
    for (let i = 0; i < len; i++) {
      binary += String.fromCharCode(bufferView8[i]);
    }
    let s = window.btoa(binary).replaceAll('+', '-').replaceAll('/', '_').replace(/=+$/, '')
    return s
  }

  const decodeGame = function(encoded) {
    encoded = encoded.replaceAll('-', '+').replaceAll('_', '/')
    while (encoded.length % 4)
      encoded += '='
    let decoded = window.atob(encoded)
    let len = decoded.length;
    let decoded8 = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
        decoded8[i] = decoded.charCodeAt(i);
    }
    return new Uint16Array(decoded8.buffer);
  }

  const encodeCustom = function() {
    let s = newWord1.value.toLowerCase() + '|' + newWord2.value.toLowerCase() + '|' + newWord3.value.toLowerCase() + '|' + newWord4.value.toLowerCase() + '|' + newWord5.value.toLowerCase()
    return CryptoJS.enc.Base64.stringify(CryptoJS.enc.Utf8.parse(s))
  }

  const decodeCustom = function(s) {
    return CryptoJS.enc.Base64.parse(s).toString(CryptoJS.enc.Utf8).split('|')
  }

  const genNewWords = function() {
    while ( words.value.length < 5 ){
      words.value.length = 0

      while ( words.value.length < 4 ) {
        let word = wordList[getRandomInt()]
        if ( !words.value.includes(word) ) {
          words.value.push(word)
        }
      }
      let fair = GetFairAnswer(wordList,words.value)
      if ( fair != '' ) {
        words.value.push(fair)
      }
    }
  }

  const loadWords = function(p) {
    let a = decodeGame(p)
    for (let i=0; i < 5; i++){
      words.value[i] = wordList[a[i]]
    }
  }

  const loadCustom = function(c) {
    let a = decodeCustom(c)
    for (let i=0; i < 5; i++){
      words.value[i] = a[i]
    }
  }

  let words = ref(Array())

  let p = (new URL(document.location)).searchParams.get('p')
  let c = (new URL(document.location)).searchParams.get('c')
  let fromC = false
  if ( p ) {
    loadWords(p)
  } else if ( c ){
    loadCustom(c)
    fromC = true
  } else {
    genNewWords()
  }

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

  const wordLength = 5
  let numberOfGuesses = 6
  let showWinModal = ref(false)
  let showHelpModal = ref(false)
  let showConfirmModal = ref(false)
  let showFormModal = ref(false)
  let allGuesses = ref(Array())

  const loadGuesses = function() {
    allGuesses.value.length = 0
    allGuesses.value.push([])
    for ( let i=1; i < words.value.length; i++ ) {
      let guesses = []
      if ( i < words.value.length - 1 ) {
        let letters = []
        for ( let x=0; x < wordLength; x++ ) {
          letters.push({ 'letter': words.value[i-1].charAt(x), 'state': 0, 'initialized': true, 'colored': true })
        }
        guesses.push(letters)
      } else {
        for ( let x=0; x < words.value.length - 1; x++ ){
          let guess = []
          for ( let j=0; j < wordLength; j++ ) {
            guess.push({ 'letter': words.value[x].charAt(j), 'state': 0, 'initialized': true, 'colored': true })
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
  let finished = ref(false)
  let correct = ref(false)
  let notInDictionary = ref(false)
  let gaveUp = ref(false)
  let newWord1 = ref('')
  let newWord2 = ref('')
  let newWord3 = ref('')
  let newWord4 = ref('')
  let newWord5 = ref('')
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

  let newWord1Invalid = computed(() => {
    return newWord1.value.length != 5 || !newWord1.value.match(/^[a-zA-Z]+$/)
  })
  let newWord2Invalid = computed(() => {
    return newWord2.value.length != 5 || !newWord2.value.match(/^[a-zA-Z]+$/)
  })
  let newWord3Invalid = computed(() => {
    return newWord3.value.length != 5 || !newWord3.value.match(/^[a-zA-Z]+$/)
  })
  let newWord4Invalid = computed(() => {
    return newWord4.value.length != 5 || !newWord4.value.match(/^[a-zA-Z]+$/)
  })
  let newWord5Invalid = computed(() => {
    return newWord5.value.length != 5 || !newWord5.value.match(/^[a-zA-Z]+$/)
  })

  let formInvalid = computed(() => {
    return newWord1Invalid.value || newWord2Invalid.value || newWord3Invalid.value || newWord4Invalid.value || newWord5Invalid.value
  })

  let newWord1NotInDictionary = computed(() => {
    if (newWord1.value.length != 5) {
      return false
    }
    return ( !acceptedWordList.includes(newWord1.value.toUpperCase()) )
  })
  let newWord2NotInDictionary = computed(() => {
    if (newWord2.value.length != 5) {
      return false
    }
    return ( !acceptedWordList.includes(newWord2.value.toUpperCase()) )
  })
  let newWord3NotInDictionary = computed(() => {
    if (newWord3.value.length != 5) {
      return false
    }
    return ( !acceptedWordList.includes(newWord3.value.toUpperCase()) )
  })
  let newWord4NotInDictionary = computed(() => {
    if (newWord4.value.length != 5) {
      return false
    }
    return ( !acceptedWordList.includes(newWord4.value.toUpperCase()) )
  })
  let newWord5NotInDictionary = computed(() => {
    if (newWord5.value.length != 5) {
      return false
    }
    return ( !acceptedWordList.includes(newWord5.value.toUpperCase()) )
  })

  const completeRow = function(skipAnimation,skipKeyboard) {
    let word = words.value[currentGame.value]
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
      if (currentGame.value == words.value.length - 1){
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

  const onKeyup = (e) => {
    if ( e.ctrlKey || e.altKey || e.metaKey ) {
      return
    }
    onKey(e.key)
  }
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
    if (showWinModal.value || showHelpModal.value || showConfirmModal.value || showFormModal.value || finished.value ) {
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
    if ( words.value[currentGame.value] == playerAnswer ){
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
    words.value.length = 0
    genNewWords()
    loadGuesses()
    showWinModal.value = false
    fromC = false
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

  const copyResults = function() {
    let gameResults = ( fromC ? "Custom" : "Infinite" ) + " Gauntletle Results\n"
    let count = getPlayerGuessCount(0)
    gameResults += "1: " + ( count > 0 ? count : 'X' ) + "/6\n"
    for ( let i=1; i < 4; i++ ) {
      count = getPlayerGuessCount(i) - 1
      gameResults += (i + 1) + ": " + ( count > 0 ? count : 'X' ) + "/5\n"
    }
    count = getPlayerGuessCount(4) - 4
    gameResults += "5: " + ( count > 0 ? count : 'X' ) + "/2\n"
    gameResults += "How well can you do?\n"
    if ( fromC ) {
      gameResults += window.location
    } else {
      gameResults += "https://theasylm.github.io/gauntletle/?p=" + encodeGame()
    }
    navigator.clipboard.writeText(gameResults)
    let span = document.getElementById('copiedResultsMessage')
    let classes = span.className
    span.className += 'shown'
    setTimeout(() => {
      span.className = classes
    }, 2000)
  }

  const getPlayerGuess = function(board,guess) {
    let s = ''
    let g = allGuesses.value[board][guess]
    for ( let i=0; i < g.length; i++ ){
      s += g[i]['letter']
    }
    return s
  }

  const getPlayerGuessCount = function(board) {
    let guesses = allGuesses.value[board]
    let playerGuess = ''
    for ( let i=0; i < guesses.length; i++ ) {
      if ( ( board != 0 && i == 0 ) || ( board == 4 && i < 4 ))
        continue

      playerGuess = getPlayerGuess(board,i)
      if ( playerGuess == '' ) {
        return i
      }
    }
    if ( playerGuess == words.value[board] ) {
      return 6
    } else {
      return 0
    }
  }

  const showForm = function() {
    showFormModal.value = true
    setTimeout(() => {document.getElementById('word1').focus()},350)
  }

  const clearForm = function() {
    newWord1.value = ''
    newWord2.value = ''
    newWord3.value = ''
    newWord4.value = ''
    newWord5.value = ''
    setTimeout(() => {document.getElementById('word1').focus()},350)
  }

  const gotoUrl = function() {
    if ( formInvalid.value ){
      return
    }
    let url = "https://theasylm.github.io/gauntletle/?c=" + encodeCustom()
    window.open(url, '_blank');
  }

  const copyShare = function() {
    if ( formInvalid.value ){
      return
    }
    let text = "Try my custom Gauntletle!\nhttps://theasylm.github.io/gauntletle/?c=" + encodeCustom()
    navigator.clipboard.writeText(text)
    let span = document.getElementById('copiedMessage')
    let classes = span.className
    span.className += 'shown'
    setTimeout(() => {
      span.className = classes
    }, 2000)
  }

  const copyUrl = function() {
    if ( formInvalid.value ){
      return
    }
    let url = "https://theasylm.github.io/gauntletle/?c=" + encodeCustom()
    navigator.clipboard.writeText(url)
    let span = document.getElementById('copiedJustMessage')
    let classes = span.className
    span.className += 'shown'
    setTimeout(() => {
      span.className = classes
    }, 2000)
  }
</script>

<template>
  <div class="container wrap">
    <div class="header row">
      <div class="col-md-3">
        <button @click="showForm" class="new-button btn btn-primary">
          <PencilIcon></PencilIcon>New Custom
        </button>
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
        <button class="btn btn-primary share-button" @click="copyResults">Share results</button><br/>
          <span id="copiedResultsMessage">Copied!</span>
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
          </div>
        </div>
        <h2>How to Create</h2>
        <div class="row">
          <div class="col-sm-12">
            <p>Create a custom Gauntletle by clicking the New Custom button. Then you will be asked to enter the five words to be guessed in order.</p>
            <p>
              Once you've filled in all the words, hit either the 'Go to Puzzle' button (good for testing your puzzle); the 'Share Link' button, which will copy the puzzle link with a handy message ready for pasting in chat, email, or wherever; or the 'Copy Link' button, which will copy just the link to the puzzle.</p>
            <hr class="with-bottom"/>
            <p class="no-bottom">Special thanks for Rangsk for the extra code!</p>
          </div>
        </div>
      </div>
    </vue-final-modal>
    <vue-final-modal
      name="newWordle"
      classes="modal-container newModal"
      :click-to-close="false"
      :esc-to-close="true"
      v-model="showFormModal"
      content-class="modal-content"
      :max-width="500"
    >
      <div class="formIcons">
        <div class="reset-modal">
          <RefreshIcon @click="clearForm"></RefreshIcon>
        </div>
        <div class="close-modal-div">
         <XIcon @click="showFormModal = false"></XIcon>
       </div>
      </div>
      <span class="modal__title">New Custom Gauntletle</span>
      <div class="modal__content">
        <div class="mb-3 row">
          <div class="col-sm-4 col-form-label">
            <label for="word1">Word 1</label>
          </div>
          <div class="col-sm-8">
            <input type="text" class="form-control" id="word1" v-model="newWord1" :class="{'has-error': newWord1Invalid }"/>
            <span class="new-word-warning-message" :class="{'shown': newWord1NotInDictionary }">Warning: word not in dictionary.</span>
          </div>
        </div>
        <div class="mb-3 row">
          <div class="col-sm-4 col-form-label">
            <label for="word2">Word 2</label>
          </div>
          <div class="col-sm-8">
            <input type="text" class="form-control" id="word2" v-model="newWord2" :class="{'has-error': newWord2Invalid }"/>
            <span class="new-word-warning-message" :class="{'shown': newWord2NotInDictionary }">Warning: word not in dictionary.</span>
          </div>
        </div>
        <div class="mb-3 row">
          <div class="col-sm-4 col-form-label">
            <label for="word3">Word 3</label>
          </div>
          <div class="col-sm-8">
            <input type="text" class="form-control" id="word3" v-model="newWord3" :class="{'has-error': newWord3Invalid }"/>
            <span class="new-word-warning-message" :class="{'shown': newWord3NotInDictionary }">Warning: word not in dictionary.</span>
          </div>
        </div>
        <div class="mb-3 row">
          <div class="col-sm-4 col-form-label">
            <label for="word4">Word 4</label>
          </div>
          <div class="col-sm-8">
            <input type="text" class="form-control" id="word4" v-model="newWord4" :class="{'has-error': newWord4Invalid }"/>
            <span class="new-word-warning-message" :class="{'shown': newWord4NotInDictionary }">Warning: word not in dictionary.</span>
          </div>
        </div>
        <div class="mb-3 row">
          <div class="col-sm-4 col-form-label">
            <label for="word5">Word 5</label>
          </div>
          <div class="col-sm-8">
            <input type="text" class="form-control" id="word5" v-model="newWord5" :class="{'has-error': newWord5Invalid }"/>
            <span class="new-word-warning-message" :class="{'shown': newWord5NotInDictionary }">Warning: word not in dictionary.</span>
          </div>
        </div>
      </div>
      <div class="modal__action">
        <div class="mb-3 row">
          <div class="col-4">
            <button @click="gotoUrl" class="btn btn-primary" :disabled="formInvalid">Go to Puzzle</button>
          </div>
          <div class="col-4">
            <button @click="copyShare" class="btn btn-primary" :disabled="formInvalid">Share Link</button><br/>
            <span id="copiedMessage">Copied!</span>
          </div>
          <div class="col-4">
            <button @click="copyUrl" class="btn btn-primary" :disabled="formInvalid">Copy Link</button><br/>
            <span id="copiedJustMessage">Copied!</span>
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
  #copiedResultsMessage {
    margin-left: 8rem;
  }
  .share-button {
    margin-left: 2rem;
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
  .new-word-warning-message {
    visibility: hidden;
    color: #ffc107;
  }
  .new-word-warning-message.shown {
    visibility: visible;
  }
  hr.with-bottom {
    margin-bottom: 1rem;
  }
  .no-bottom {
    margin-bottom: 0;
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

