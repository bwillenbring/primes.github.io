<template>

    <div :id="id" class="gameContainer"> 
        <div class="title">Is It <span class="text-warning">Prime</span>?</div> 
        <div class="card mb-4 thick_border">
            <div class="card-body p-0">
                
                <div class="number">
                    <!-- NUMBER -->
                    <span id="number" v-cloak>{{currentNumber}}</span>
                </div>

                <!-- Scoreboard -->
                <div class="container-fluid">
                    <div class="row scoreboard">

                        <!-- Score --> 
                        <div class="col"><span class="badge badge-success" style="transform:scale(1.2);">Score: {{score}}</span></div>
                        
                        <!-- Animation Layer -->
                        <div class="col d-relative" id="msg">
                            <div id="animation" class="position-absolute"><span id="face"></span></div>
                        </div>

                        <!-- Wrong Counter -->
                        <div class="col">
                            <span class="badge badge-danger" style="transform:scale(1.2);">
                                <span class="mr-1">Wrong:</span>
                                <span id="incorrect">0</span>
                            </span>
                        </div>

                    </div>
                </div>

                
            </div>
            <div class="card-footer game_footer">
                <div class="container-fluid text-right">
                    <div class="row text-right">
                        <div class="container-fluid pt-2 pb-2 text-center">
                            <!-- <button @click="getNextNumber()" type="button" class="btn btn-lg btn-warning">Next number</button> -->
                            <button data-type="control" @click="answer('no')" type="button" class="btn btn-lg btn-secondary">not prime!</button>
                            <button data-type="control" @click="answer('yes')" type="button" class="btn btn-lg btn-light">prime!</button>
                            <!-- Timer -->
                            <div v-if="show_timer" id="timer" class="timer d-inline-block ml-4"> 
                                <span><i class="far fa-clock"></i></span>
                                <div id="readout" class="d-inline-block" data-value="0"></div>
                            </div>
                        </div>
                    </div>
                </div>

            </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="modal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLongTitle" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="exampleModalLongTitle">😡 Game Over!</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <p>You have achieved a score of: <b>{{score}}</b>. Keep trying.
                    </p>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                </div>
                </div>
            </div>
            </div>
        </div>

    </div>
</template>

<script>
module.exports = {
    props: {
        id: {
            type: String,
            default: function() {
                return 'prime_numbers_game_0';
            } 
        },
        show_timer: {
            type: Boolean,
            default: function() {
                return false;
            }
        },
        primes: {
            type: Array,
            default: function() {
                return[];
            }
        }
    },
    data: function() {
        return {
            test: 'foo',
            currentNumber: this.randBetween(2, 50),
            score: 0,
            incorrect: 0
        }
    },
    computed: { 
        numberToDisplay: function() {
            // Just return a sample value for now 
            return 55;
        }
    },
    mounted () {
        this.loadSounds();
        const reset = this.resetGame;
        $('#modal').on('hidden.bs.modal', function(event) {
            reset();
        });
    },
    watch: {
        primes: function(a, b) { // a = new value
            if (a.length > 0) {
                this.currentNumber = this.primes[0];
            }
        },
        currentNumber: function(a, b) {
            if (a !== b) {
                console.log('yeah');
            }
        },
        incorrect: function(a, b) {
            if (a > b && a <= 3) {
                this.playSound('wrong');
            }
            if (a===3) {
                this.gameOver();
            }
            let container = $('#incorrect');
            let mark = `<i class="fas fa-times-circle text-light" style="transform:scale(.8)"></i>`;
            let html = (a > 0) ? new Array(a).fill(mark).join('') : '0';
            container.html(html);

        },
        score: function(a,b) {
            if (a > 100) {
                this.playSound('awesome');
            }
            else {
                this.playSound('correct');
            }
        }
    },
    methods: {
        answer: function(response) {
            const msg = $('#msg');
            // Find out if the current number is prime 
            const is_prime = this.primes.indexOf(this.currentNumber) !== -1;
            const responseMapping = {
                'yes': true,
                'no': false
            }
            const msgMapping = {
                'yes': `${this.currentNumber} is Prime!`,
                'no': `${this.currentNumber} is not prime`
            }
            const correct = (is_prime === responseMapping[response]);
            if (correct) {
                // User is right 
                this.score += this.currentNumber;
            }
            else {
                // User is wrong 
                this.incorrect += 1;
            }
            this.animateCorrect(correct);
            
        },
        getNextNumber: function() {
            // Locate pos of current number 
            let c = Number(this.currentNumber) + 1;
            this.currentNumber = this.randBetween(c, c+50);
            const fn = () => {
                $('button[data-type="control"]').attr('disabled', false).show();
            }
            window.setTimeout(fn, 250);
        },
        animateCorrect: function(correct) {
            let completed = false;
            let t = (correct) ? '😹' : '😢';
            const fn = this.getNextNumber;
            const btns = $('button[data-type="control"]');
            const disable_buttons = ()=> { btns.attr('disabled', true) }
            disable_buttons();

            let nmbr = $('#number');
            let b = anime({
                targets: '#number',
                opacity: 0,
                duration:250,
                delay:500,
                easing: 'linear',
                complete: function(anim) {
                    let o = anime.get('#number', 'opacity');
                    console.log(o);
                    if (o=== 0 && completed === false) {
                        fn();
                        completed = true;
                        anim.reverse();
                        anim.play();
                    }
                }
            });
            $('#animation').html(t);
            let a = anime({
                targets: '#animation',
                keyframes: [
                    {
                        top: -50,
                        opacity: 1,
                        scale: 4,
                        duration: 750
                    },
                    {
                        top: 0,
                        opacity: 0,
                        scale: 1,
                        duration: 1500
                    }
                ]
            });

        },
        gameOver: function() {
            this.playSound('game_over');
            let m = $('#modal')
            m.modal('show');
        },
        randBetween: function(min, max) {
            // let n = Math.floor(Math.random() * max) + min;
            let n = Math.floor(Math.random() * (max - min + 1)) + min;
            console.log(`Generating random number between ${min}-${max}, returning...${n}`);
            return n;
        },
        loadSounds: function() {
            createjs.Sound.registerSound("sounds/explosion.mp3", 'wrong');
            createjs.Sound.registerSound("sounds/game_over.mp3", 'game_over');
            createjs.Sound.registerSound("sounds/correct.mp3", 'correct');
            createjs.Sound.registerSound("sounds/correct_cant_touch.mp3", 'awesome');
            // createjs.Sound.registerSound("sounds/correct.m4a", 'correct');
            console.log('registered sounds!');
        },
        playSound: function(soundID) {
            createjs.Sound.play(soundID);
        },
        resetGame: function() {
            this.currentNumber = this.randBetween(2, 50)
            this.incorrect = 0;
            this.score = 0;
            console.log('RESETTING GAME!!!');
        }
    }
}
</script>

<style scoped>
  span {
    /* background-color: yellow; */
  }
</style>