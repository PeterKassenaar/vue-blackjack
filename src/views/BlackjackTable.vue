<template>
    <div>
        <div class="row">
            <div class="col-9">
                <div v-if="bet.placed">
                    <h2>House:</h2>
                    <hand v-if="house.hand.length > 0" :cards="house.hand"></hand>
                    <div>Total score: {{ house.handValue }}</div>
                    <h2>Player:</h2>
                    <hand v-if="player.hand.length > 0" :cards="player.hand"></hand>
                    <div>Total score: {{ player.handValue }}</div>
                    <div class="alert alert-info" v-if="debugMode">
                        <h4>Debugging/development box</h4>
                        <p v-if="player.aces > 0">Aces: {{player.aces }}</p>
                        <p v-if="player.numCards > 0">no. of cards: {{player.numCards}}</p>
                        <p>blackjack: {{checkBlackjack()}}</p>
                        <p>Pair: {{ isPair }} </p>
                    </div>
                    <button class="btn btn-warning" @click="houseTurn(house)" :disabled="!playerTurn">Stand</button>
                    <button class="btn btn-warning" @click="playerHit" :disabled="!playerTurn">Hit</button>
                    <button class="btn btn-warning" @click="playerDouble" :disabled="!canDouble">Double</button>
                    <button class="btn btn-warning" :disabled="!playerTurn">Split</button>
                    <button class="btn btn-warning" @click="resetGame" :disabled="playerTurn">New Round</button>
                </div>
            </div>
            <div class="col-3 sidebar">
                <message-box v-if="message" :message="message"></message-box>
                <div class="cash-box">
                    <h4>Available cash: €{{ this.cash }}</h4>
                </div>
                <betting-interface :cash="cash" v-model="bet" @input="startGame"></betting-interface>
                <game-history v-if="gameResults.length > 0" :gameResults="gameResults"></game-history>
            </div>
        </div>
    </div>
</template>

<script>
    import Hand from "@/components/Hand.vue";
    import MessageBox from "@/components/MessageBox.vue";
    import BettingInterface from "@/components/BettingInterface.vue";
    import GameHistory from "@/components/GameHistory.vue";
    import CardsApi from "@/api/CardsApi";

    export default {
        name: "BlackjackTable",
        components: {
            Hand,
            MessageBox,
            BettingInterface,
            GameHistory
        },
        data() {
            return {
                deckId: '',
                player: {
                    hand: [],
                    handValue: 0,
                    aces: 0,
                    numCards: 0
                },
                house: {
                    hand: [],
                    handValue: 0,
                    aces: 0,
                    numCards: 0
                },
                message: "",
                cash: 1000,
                bet: {
                    placed: false,
                    size: 25
                },
                gameEnd: false,
                playerTurn: false,
                gameResults: [],
                debugMode: true // DEBUGGING FLAG
            };
        },
        mounted() {
            CardsApi.shuffleNewDeck().then(response => {
                this.deckId = response.deck_id;
            });
        },
        methods: {
            startGame() {
                this.cash -= this.bet.size;
                this.newGame();
            },
            resetGame() {
                this.bet.placed = false;
                this.bet.size = 25;
                this.gameEnd = false;
                this.playerTurn = false;
                this.message = "";
                this.player.numCards = 0;
                this.player.aces = 0;
            },
            newGame() {
                this.message = "";
                this.player.hand = [];
                this.house.hand = [];
                this.addCardToHand(this.player, 2);
                this.addCardToHand(this.house, 1);
                this.playerTurn = true;
            },
            addCardToHand(player, number) {
                return CardsApi.draw(this.deckId, number).then(response => {
                    response.cards.forEach(card => {
                        player.hand.push(card);
                        player.numCards += 1;
                    });
                    // calculate initial value of player hand
                    player.handValue = this.calculateHandValue(player);
                    // check if player has blackjack
                    if (this.checkBlackjack()) {
                        this.determineWinner();
                    }
                });
            },
            calculateHandValue(player) {
                let cardTotal = 0;
                player.hand.forEach(card => {
                    if (
                        card.value === "QUEEN" ||
                        card.value === "JACK" ||
                        card.value === "KING"
                    ) {
                        cardTotal += 10;
                    } else if (card.value === "ACE") {
                        // Ace is 11 or 1, depending on current cardTotal
                        if (cardTotal < 11) {
                            cardTotal += 11;
                        } else {
                            cardTotal += 1;

                        }
                        player.aces += 1;
                    } else {
                        cardTotal += parseInt(card.value);
                    }
                });
                return cardTotal;
            },
            playerHit() {
                return this.addCardToHand(this.player, 1).then(() => {
                    this.determineWinner();
                });
            },
            playerDouble() {
                // Double down only allowed on first two cards
                if (this.player.numCards === 2) {
                    this.cash -= this.bet.size;
                    this.bet.size *= 2;
                    this.playerHit().then(() => {
                        if (!this.gameEnd) {
                            this.houseTurn();
                        }
                    });
                }
            },
            checkBlackjack() {
                return this.player.numCards === 2 && this.player.handValue === 21;
            },
            houseTurn() {
                this.playerTurn = false;
                this.addCardToHand(this.house, 1).then(() => {
                    if (this.house.handValue < 17) {
                        this.houseTurn();
                    } else {
                        this.determineWinner();
                    }
                });
            },
            determineWinner() {
                if (this.player.handValue > 21) {
                    this.message = "House WINS!";
                    this.gameEnd = true;
                    this.playerTurn = false;
                    this.writeResult("lost", this.bet.size);
                } else if (!this.playerTurn) {
                    if (this.house.handValue > 21 || this.house.handValue < this.player.handValue
                    ) {
                        this.message = "Player WINS!";
                        this.gameEnd = true;
                        this.cash += 2 * this.bet.size;
                        this.writeResult("won", this.bet.size);
                    } else if (this.house.handValue === this.player.handValue) {
                        this.message = "PUSH, no winner";
                        this.gameEnd = true;
                        //TODO: bij push zonder parseInt lijkt bet.size geconcatenate te worden?
                        this.cash += parseInt(this.bet.size);
                        this.writeResult("push", '-');
                    } else if (!this.playerTurn) {
                        this.message = "House WINS!";
                        this.gameEnd = true;
                        this.writeResult("lost", this.bet.size);
                    }
                }
            },
            writeResult(result, amount) {
                this.gameResults.push({
                    result: result,
                    amount: amount
                });
            }
        },
        computed: {
            canDouble() {
                return this.player.numCards === 2;
            },
            isPair() {
                if (this.player.numCards !== 2) {
                    return false;
                } else {
                    return this.player.hand[0].value === this.player.hand[1].value;
                }
            }
        }
    };
</script>

<style>
    .cash-box {
        background-color: #ffc107;
        padding: 1rem;
        color: black;
    }

    h1 {
        color: white;
    }

    .btn {
        width: 150px;
        margin: 1rem;
    }
</style>
