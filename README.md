# CS224-HW
import * as readline from 'readline';

// Define enums for card suits and ranks
enum Suit {
    Hearts,
    Spades,
    Clubs,
    Diamonds
}

enum Rank {
    Ace = 1,
    Two,
    Three,
    Four,
    Five,
    Six,
    Seven,
    Eight,
    Nine,
    Ten,
    Jack = 10,
    Queen = 10,
    King = 10,
}

// Represents a playing card
class PlayingCard {
    suit: Suit;
    rank: Rank;

    constructor(suit: Suit, rank: Rank) {
        this.suit = suit;
        this.rank = rank;
    }

    // Get the value of the card
    get value(): number {
        return this.rank;
    }

    // Convert the card to a string representation
    toString(): string {
        return `${Rank[this.rank]} of ${Suit[this.suit]}`;
    }
}

// Represents a deck of playing cards
class Deck {
    cards: PlayingCard[] = [];

    constructor() {
        this.initDeck();
    }

    // Initialize the deck with all possible combinations of suits and ranks
    initDeck() {
        Object.values(Suit).forEach(suit => {
            if (typeof suit === 'number') {
                Object.values(Rank).forEach(rank => {
                    if (typeof rank === 'number') {
                        this.cards.push(new PlayingCard(suit, rank));
                    }
                });
            }
        });
    }

    // Shuffle the deck
    shuffle() {
        for (let i = this.cards.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [this.cards[i], this.cards[j]] = [this.cards[j], this.cards[i]];
        }
    }

    // Deal a card from the deck
    deal(): PlayingCard | undefined {
        return this.cards.pop();
    }
}

// Represents a player in the game
class Player {
    hand: PlayingCard[] = [];
    name: string;

    constructor(name: string) {
        this.name = name;
    }

    // Draw a card and add it to the hand
    drawCard(card: PlayingCard) {
        this.hand.push(card);
    }

    // Calculate the score of the hand, considering possible Ace values
    get score(): number {
        let score = 0;
        let aceCount = 0;

        for (const card of this.hand) {
            score += card.value;
            if (card.rank === Rank.Ace) {
                aceCount++;
            }
        }

        while (score > 21 && aceCount > 0) {
            score -= 10;
            aceCount--;
        }

        return score;
    }

    // Convert the hand to a string representation
    handToString(): string {
        return this.hand.map(card => card.toString()).join(',');
    }
}

// Main game class
class BlackjackGame {
    private rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout
    });

    player: Player;
    dealer: Player;
    deck: Deck;

    constructor() {
        this.deck = new Deck();
        this.player = new Player("Player");
        this.dealer = new Player("Dealer");
    }

    // Start the game
    async start() {
        this.deck.shuffle();
        this.dealCards();
        this.showInitialCards();

        await this.playerTurn();
        if (this.player.score <= 21) {
            this.dealerTurn();
        }
        this.determineOutcome();
        this.rl.close();
    }

    // Deal initial cards to player and dealer
    private dealCards() {
        for (let i = 0; i < 2; i++) {
            this.player.drawCard(this.deck.deal()!);
            this.dealer.drawCard(this.deck.deal()!);
        }
    }

    // Show initial cards of dealer and player
    private showInitialCards() {
        console.log(`Dealer's card: ${this.dealer.hand[1]?.toString()}`);
        console.log(`Player's cards: ${this.player.handToString()}, Score: ${this.player.score}`);
    }

    // Player's turn to draw cards or stay
    private async playerTurn() {
        let move: string;
        do {
            move = await this.getPlayerMove();
            if (move === 'h') {
                this.player.drawCard(this.deck.deal()!);
                console.log(`Player hit: ${this.player.handToString()}, Score: ${this.player.score}`);
                if (this.player.score > 21) {
                    console.log("Player bust");
                    break;
                }
            }
        } while (move !== 's')
    }

    // Get player's move from input
    private getPlayerMove(): Promise<string> {
        return new Promise((resolve) => {
            this.rl.question('Select to (H)it or (S)tand. ', (answer) => {
                resolve(answer.trim().toLowerCase());
            })
        })
    }

    // Dealer's turn to draw cards until score >= 17
    private dealerTurn() {
        console.log(`Dealer's hand: ${this.dealer.handToString()}, Score: ${this.dealer.score}`);
        while (this.dealer.score < 17) {
            this.dealer.drawCard(this.deck.deal()!);
            console.log(`Dealer hits: ${this.dealer.handToString()}, Score: ${this.dealer.score}`);
        }
        if (this.dealer.score > 21) {
            console.log("Dealer bust");
        }
    }

    // Determine the outcome of the game
    private determineOutcome() {
        if (this.player.score > 21) {
            console.log("Dealer wins");
        } else if (this.player.score > this.dealer.score || this.dealer.score > 21) {
            console.log("Player wins");
        } else if (this.player.score < this.dealer.score) {
            console.log("Dealer wins");
        } else {
            console.log("Player and Dealer push")
        }
    }
}

// Start the game
const game = new BlackjackGame();
game.start().catch(err => console.error(err));
