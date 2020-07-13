import random

suits = ('Hearts', 'Diamonds', 'Spades', 'Clubs')
ranks = ('Two', 'Three', 'Four', 'Five', 'Six', 'Seven', 'Eight', 'Nine', 'Ten', 'Jack', 'Queen', 'King', 'Ace')
values = {'Two':2, 'Three':3, 'Four':4, 'Five':5, 'Six':6, 'Seven':7, 'Eight':8, 
            'Nine':9, 'Ten':10, 'Jack':11, 'Queen':12, 'King':13, 'Ace':14}

class Card():
    """
    This creates cards with a specific suit and rank
    """
    def __init__(self,suit,rank):
        self.suit = suit
        self.rank = rank
        self.value = values[rank]
        
    def __str__(self):
        return f"{self.rank} of {self.suit}"
    
class Deck():
    """
    This creates the Deck
    """
    def __init__(self):
        self.cards = []
        for suit in suits:
            for rank in ranks:
                self.cards.append(Card(suit,rank))
                
    def shuffle(self):
        random.shuffle(self.cards)
                
    def deal_one(self):
        return self.cards.pop()
    
    def add_cards(self,new_cards):
        self.cards.append(new_cards)
    
class House():
    """
    This creates the house
    """
    def __init__(self,pool,bank,deck):
        self.pool = 0
        self.bank = 0
        self.deck = deck
        
    def __str__(self):
        return f"${self.pool} are currently in play"
    
    def add_cards(self,new_cards):
        self.deck.add_cards(new_cards)
        
min_bet = 5
        
class Player():
    """
    This creates the players and their possible actions
    """
    
    def __init__(self,name,money,cards,bet=min_bet,state="Good"):
        self.name = name
        self.money = money
        self.cards = cards
        self.bet = bet
        self.state = state
        print(f"{self.name} has entered the game!\n")
        
    def __str__(self):
        return f"{self.name} has ${self.money}"
    
    def add_cards(self,new_cards):
        self.cards.append(new_cards)
            
    def bets(self):
        add = float(input(f"{self.name}, how much more would you like to bet? "))
        while add > self.money:
            add = float(input(f"You only have ${self.money}! You can only bet less than that! "))
        self.bet += add
        return add
        
    def return_cards(self):
        return self.cards.pop()
    
    
"""
This is where I start defining the logic and functions of the game itself using the objects above
"""

        
players = []
house = House(0,0,Deck())
house.deck.shuffle()

def create_players():
    """
    This creates all the players
    """
    number = int(input("How many players are there? "))
    names = []
    for i in range(number):
        name = input(f"Player {i+1}, what is your name? ")
        players.append(Player(name,100,[]))
    return players
        
def start_round():
    for player in players:
        house.pool += min_bet
        player.money -= min_bet
        for i in range(2):
            player.add_cards(house.deck.deal_one())
            
    print("_"*100,"\n")
    print(f"\nNew Round\n")

def betting():
    """
    This updates the bets of all the players
    """
    for player in players:
        if player.state == "Bust":
            pass
        else:
            add = player.bets()
            house.pool += add
            player.money -= add
        
def hit():
    """
    Asks all the players if they want another card
    """
    hits = []
    for player in players:
        if player.state == "Bust":
            pass
        else:
            please = input(f"{player.name}, would you like another card? (y/n)")
            while please not in ['y','n']:
                player = input("Choose either y/n")
            if please == 'n':
                hits.append('n')
            if please == 'y':
                player.add_cards(house.deck.deal_one())
                hits.append('y')
    print("\n","-"*100)
    return hits
            
def count():
    """
    This counts up a person's card
    """
    amount = []
    for player in players:
        total = 0
        for card in player.cards:
            total+=card.value
        amount.append(total)
    return amount

def translate(values):
    """
    This checks if a person has busted
    """
    reading = []
    for value in values:
        if value>28:
            reading.append("Bust")
        else:
            reading.append(value)
    for i in range(len(reading)):
        if reading[i] == "Bust":
            players[i].state = "Bust"
    return reading
        
def win(readings):
    """
    This determines who wins a round
    """
    survivors = []
    winners = []
    for i in range(len(readings)):
        if readings[i] == "Bust":
            pass
        else:
            survivors.append((readings[i],i))
    if len(survivors)==0:
        pass
    else:
        ends=[]
        for thing in survivors:
            ends.append(thing[0])
        best = max(ends)
        for thing in survivors:
            if thing[0] == best:
                winners.append(thing)
    return winners
            
def winnings(winners):
    """
    This distributes the pool among the winners"""
    if len(winners)==0:
        house.bank += house.pool
    else:
        for winner in winners:
            players[winner[1]].money += house.pool/len(winners)
        for winner in winners:
            print(f"{players[winner[1]].name} has won ${house.pool/len(winners)-players[winner[1]].bet}")
            
def current():
    for player in players:
        if player.state == "Bust":
            print(f"{player.name} has bet ${player.bet}! {player.name} has Busted!")
            print("")
        else:
            print(f"{player.name} has bet ${player.bet}!:")
            for card in player.cards:
                print(card)
            print("")
        
def end_round():
    """
    This sets the game up for aother round
    """
    for player in players:
        for i in range(len(player.cards)):
            house.add_cards(player.return_cards())
        player.state = "Good"
        player.bet = min_bet
        print(f"{player.name} now has ${player.money}")
    house.deck.shuffle()
    house.pool = 0
    print("_"*100)
    
    
    

time = 1 
def blackjack():
    house = House(0,0,Deck())
    house.deck.shuffle()
    players = create_players()
    time = 1
    
    for i in range(2):
        start_round()
        current()
        
        states = []
        keep_going = []
        for player in players:
            states.append(player.state)
            keep_going.append("y")
        while states != ["Bust"]*len(players) and states != ["Bust","Good"] and states != ["Good","Bust"] and keep_going != ["n"]*len(players):
            keep_going = []
            states = []
            betting()
            keep_going.extend(hit())
            translate(count())
            news = input("Show results? ")
            if news == "":
                pass
            print("-"*100)
            current()
            for player in players:
                states.append(player.state)
        else:
            winnings(win(translate(count())))
            end_round()
            time += 1
            new = input("New Round? ")
            if new == '':
                pass
            

blackjack()
