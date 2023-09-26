<template lang="">
    <div class="w-full flex flex-col justify-center items-center m-3">
        <div class="text-3xl">Wordle</div>
         {{word}}
        <div class="text-xl">
            {{message}}
        </div>
        <div class="flex flex-col justify-center items-center mt-2">
            <div v-for="(tries, i) in dimensiones.row" class="flex justify-center">
                <input @input="changeInput($event, j)" :id="`input${i}${j}`" v-for="(item, j) in dimensiones.col" v-model='values[i][j]' type="text" class="uppercase border w-10 border-black text-3xl text-center m-1" maxlength="1" :disabled="current_iteration != i">
            </div>
        </div>
        <input v-if="!game_finished" type="submit" class="bg-blue-200 p-3 rounded-lg" @click="submitIteration" name="submit" value="Comprobar">
        <input v-else type="submit" class="bg-blue-200 p-3 rounded-lg" @click="clearGame" name="submit" value="Intenta nuevamente">
        
        <div class="text-xs text-justify mt-5 w-1/2">Instrucciones: Solo tienes 6 intentos para comprobar la palabra, las casillas marcadas en <small class="bg-green-200">verde</small> significan que tienen la letra en la posicion correcta, y las letras marcadas en en <small class="bg-yellow-200">amarillo</small> son aquellas que pertenecen a la palabra pero que no estan en el orden correcto.</div>

    </div>
</template>
<script>
export default {
    name: "homepage",
    data() {
        return {
            wordsList:[],
            word: '',
            dimensiones: {
                col: 5,
                row: 6
            },
            values:[],
            current_iteration: 0,
            message:'',
            game_finished:false
        }
    },
    beforeMount() {
        this.makeRequest();
        // Default values dimension init based on the configured values for the state
        this.values = Array(this.dimensiones.row).fill(0).map(() => new Array(this.dimensiones.col).fill(''));
    },
    methods: {
        async makeRequest(){
            try{
                let data = await $fetch('https://challenge.trio.dev/api/v1/wordle-words');
                this.words = data;
                let wordIndex = this.getRange(data.length - 1);
                this.word = data[wordIndex];
            } catch (e){
                console.log(e);
            }
        },
        async submitIteration (){
            const current_chars = this.values[this.current_iteration]
            let asserts = 0
            let expected_indexes = Array();
            current_chars.forEach((element, current_index) => {
                const current_word = element.toUpperCase();
                const target = document.getElementById(`input${this.current_iteration}${current_index}`);
                
                let counter=-1;
                while ((counter = this.word.indexOf(current_word, counter+1)) != -1){
                    expected_indexes.push(counter);
                }
                if(expected_indexes.includes(current_index)){
                    target.classList.add("bg-green-200");
                    asserts++;
                } else if (this.word.indexOf(current_word) > -1) {
                    target.classList.add("bg-yellow-200");
                }
                expected_indexes = Array();
            });

            if(asserts == this.dimensiones.col){
                this.endGame("Juego Terminado, has encontrado la palabra!")
            } else if(this.current_iteration == 5){
                this.endGame("Juego terminado, no has adivinado la palabra!")
            } else {
                this.current_iteration++;
            }
        },
        endGame(message, ){
            this.message = message;
            this.current_iteration = -1;
            this.game_finished = true;
        },
        changeInput(event, j){
            if(j < (this.dimensiones.col - 1) && event.target.value != ''){
                document.getElementById(`input${this.current_iteration}${j+1}`).focus();
            } 
        },
        getRange(max){
            return Math.floor(Math.random() * max)
        },
        clearGame(){
            this.makeRequest();
            this.values = Array(this.dimensiones.row).fill(0).map(() => new Array(this.dimensiones.col).fill(''));
            this.current_iteration = 0;
            this.game_finished = false;
            this.message = '';
            let correct_elements = document.getElementsByClassName('bg-green-200');
            let wrong_elements = document.getElementsByClassName('bg-yellow-200');
            let merged_elements = [...correct_elements, ...wrong_elements];
            for (var i = 0; i < merged_elements.length; i++) {
                merged_elements[i].classList.remove('bg-green-200');
                merged_elements[i].classList.remove('bg-yellow-200');
            }
        }
    },
}
</script>
<style lang="">
    
</style>