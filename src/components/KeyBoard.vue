<script setup>
import { ref, computed, onMounted, reactive, watch } from 'vue';

const characterList = reactive([]);
const currentIndex = ref(0);
const gameTime = ref(30);
const number_words = ref(10);
let startTyping = false;
let time;

onMounted(() => {
    newGame();
});

const getRaondomWords = async (number) => {
    let result = await fetch(`https://random-word-api.herokuapp.com/word?number=${number}`);
    return result.json();
};

const resetTimer = () => {
    startTyping = false;
    clearInterval(time);
    gameTime.value = 30;
};

const GameOver = () => {
    gameTime.value = 0;
    clearInterval(time);
    alert('End');
    newGame();
};

const newGame = async () => {
    resetTimer();

    currentIndex.value = 0;
    let wordList = await getRaondomWords(number_words.value);
    characterList.length = 0;
    wordList.forEach(word => {
        word.split('').forEach((character)=>{
            characterList.push({'character':character, 'typeState':'normal', 'typeCursor':''});
        });
        characterList.push({'character':' ', 'typeState':'normal', 'typeCursor':''});
    });
    characterList[0].typeCursor = 'cursor';
};

const PressKeyListener = (e) => {
    const key = e.key;
    let temp_characterList = characterList[currentIndex.value];
    
    if(!startTyping){
        startTyping = true;
        countDownTimer();
    }

    if(key === 'Backspace' && currentIndex.value >= 0){
            characterList[currentIndex.value-1].typeState = 'normal';
            currentIndex.value--;
    } else if (currentIndex.value < characterList.length - 1){
        if(key === temp_characterList.character){
            temp_characterList.typeState = 'correct';
            currentIndex.value++;
        }else{
            temp_characterList.typeState = 'inCorrect';
            currentIndex.value++;
        }
    }
};

const countDownTimer = () => {
    time = setInterval(()=>{
        gameTime.value--;
    }, 1000);
};

watch(currentIndex, (newIndex, oldIndex)=>{
    characterList[newIndex].typeCursor = 'cursor';
    characterList[oldIndex].typeCursor = '';

    if(newIndex === characterList.length - 1){
        GameOver();
    }
});

watch(gameTime, (newTime, oldTime)=>{
    if(newTime <= 0){
        GameOver();
    }
});

// limit text number
watch(number_words, (newVal, oldVal)=>{
    if(newVal > 99 || newVal < 0){
        number_words.value = oldVal;
    }
});
</script>

<template>
<div class="container">
    <h1>Typing Game</h1>
    <div id="header">
        <div id="info"><span id="info-text">Time：</span>{{ gameTime }}</div>
        <div id="input-num">
            <div class="input-num-text">Enter text number</div>
            <input v-model="number_words" class="number_words" type="number">
        </div>
        <div id="game-over-button">
            <button @click="newGame">New Game</button>
        </div>
    </div>

    <div @keyup="e => PressKeyListener(e)" class="" id="game" tabindex="1">
        <div id="words">
            <span v-for="(item, index) in characterList" :key="index" class="character" :class="[item.typeState, item.typeCursor]">{{ item.character }}</span>
        </div>
        <div id="focus-error">Click here to focus</div>
    </div>
</div>

</template>

<style lang="scss" scoped>
@import "../assets/style.scss";
.container{
    padding: 20px;
    width: 100vw;

    h1{
        padding-left: 0px;
        margin: 50px auto;
        font-size: 40px;
        color: $primaryColor;
    }

    #header{
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;

        #info{
            color: $primaryColor;
            font-size: 25px;
            #info-text{
                color: $textPrimary;
            }
        }

        #input-num{
            display: flex;
            justify-content: center;
            align-items: center;
        
            .input-num-text{
                font-size: 25px;
            }
            .number_words{
                width: 50px;
                margin: 10px;
                background-color: $bgColor;
                color: $primaryColor;
                outline: none;
                font-size: 25px;
                border: none;
                list-style: none;
                text-decoration: none;
            }
        }

        #game-over-button{
        text-align: right;

            button{
                background-color: rgba(255,255,255,.2);
                border: 0;
                color: rgba(255,255,255,.5);
                padding: 5px 20px;
                border-radius: 5px;
                cursor: pointer;
            }
        }
    }

    #game{
        position: relative;
        margin-top: 20px;
        #words{
            font-size: 25px;
            font-weight: 600;
            font-family: monospace;
            line-height: 1.8;
            filter: blur(5px);
            word-break: normal;
            .word{
                padding-right: 12px;
            }
            .character{
                position: relative;
            }
        }
        #focus-error{
            position: absolute;
            inset: 0;
            text-align: center;
            padding-top: 35px;
            font-size: 20px;
        }
    }

    #game:focus #words{
        filter: blur(0px);
        color: $textSecondary;
    }
    #game:focus #focus-error{
        display: none;
    }
}

.character.correct{
    color: $correct;
}
.character.inCorrect{
    color: $inCorrect;
}
.character.normal{
    color: $textSecondary;
}

@keyframes blink{
    0%{
        opacity: 1;
    }
    50%{
        opacity: 0;
    }
    100%{
        opacity: 1;
    }
}

.cursor{
    position: relative;
}
.cursor::before{
    content: '　';
    display: inline-block;
    position: absolute;
    top: 0;
    border-left: 2px solid $primaryColor;
    width: 1px;
    height: 2rem;
    animation: blink .7s infinite;
}


</style>
