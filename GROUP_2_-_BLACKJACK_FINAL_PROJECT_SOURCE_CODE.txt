# ------------------------------------------------------------------------------------------------------------
############################################# AUTHOR'S NOTES #################################################
# ------------------------------------------------------------------------------------------------------------
#
# 30-April-2023
# CSC-130-02 Intro to Scripting Fundamentals / SPR '23 Semester
# Hagerstown Community College
# Group 2
# Final Project: BlackJack Game

# This game is a collaborative effort between the following group members. (The names are listed in alphabetical order by last name):

# Elkins Berrios-Santos
# Rylee Cutter
# Gayla Dunphy
# Ben Hinz
# Collin Le

# ------------------------------------------------------------------------------------------------------------
########################################### Modules to import ################################################
# ------------------------------------------------------------------------------------------------------------

import random # Used in random generation throughout the script

import sys  # Will be used to terminate the game at any time 

import os # Will only be used to "clear screen" for better visibility to the player 

# ------------------------------------------------------------------------------------------------------------
############################################# GAME BANNER ####################################################
# ------------------------------------------------------------------------------------------------------------

ascii_banner = """
oooooooooo.  oooo                      oooo           oooo                     oooo        
`888'   `Y8b `888                      `888           `888                     `888        
 888     888  888   .oooo.    .ooooo.   888  oooo      888  .oooo.    .ooooo.   888  oooo  
 888oooo888'  888  `P  )88b  d88' `"Y8  888 .8P'       888 `P  )88b  d88' `"Y8  888 .8P'   
 888    `88b  888   .oP"888  888        888888.        888  .oP"888  888        888888.    
 888    .88P  888  d8(  888  888   .o8  888 `88b.      888 d8(  888  888   .o8  888 `88b.  
o888bood8P'  o888o `Y888""8o `Y8bod8P' o888o o888o .o. 88P `Y888""8o `Y8bod8P' o888o o888o 
                                                   `Y888P                                  
"""
print(ascii_banner)
print("Welcome to Black Jack! To quit the game at any time, type \"bye\". ")

# -----------------------------------------------------------------------------------------------------------
# ########################################## RULES ##########################################################
# -----------------------------------------------------------------------------------------------------------

rules = """
  ____ ____ ____ ____ ____ 
||                        ||
||  R   U    L    E    S  ||
||____ ____ ____ ____ ____|| 

Objective: Beat the dealer! (computer)!

How to win!:
    1) Your hand value is higher than the dealer's (but still under 21).
    2) The dealer "busts." (Dealer's hand value exceeds 21.)
    3) Draw a hand value of 21 on your first 2 cards, when dealer doesn't.

How you lose:
    1) You "bust." (Your hand value goes over 21)
    2) Dealer's hand value is greater than yours at end of round (but still under 21)

The value of cards is determined as follows:
    1) 2 thru 10 count at face value (i.e. 2 = 2, 3 = 3, and so on.)
    2) Face cards (Jack, Queen, King) = 10 points
    3) An Ace can count as a 1 or 11, depending on which value benefits the player (or dealer) more.    

To play:

Before the cards are dealt, both the player and the dealer are assigned a balance of XXXXX points, and both the player and the dealer can choose to bet any amount of points less than or equal to their full balance. 
The player that loses will have their entire bet given to the winning player, adding to their balance.

1) At the start of the game, the player is dealt two cards. The dealer also receives two cards, but only one card is revealed; the other card is face-down.

2) Players can choose to "hit" and receive additional cards in order to improve their hand, or "stand" and keep their current hand.

3) If a player's hand exceeds 21 points, they "bust" and lose the game immediately.

4) After the player is finished taking their turn, the dealer reveals their face-down card and must hit until their hand has a value of 17 or more.

5) If the dealer busts, the player wins. If the dealer doesn't bust, the player's hand is compared to the dealer's hand, and whoever has the higher hand value without busting wins.

6) A "blackjack" is a two-card hand consisting of an ace and any 10-point card. A blackjack beats all other hands, except for another blackjack, which results in a tie or "push".

"""

def main():

    # ------------------------------------------------------------------------------------------------------------------
    # ############################################### FUNCTIONS ########################################################
    # ------------------------------------------------------------------------------------------------------------------


    ########################################### ALLOWS USER TO QUIT ANYTIME ###########################################

    def user_input(text):
        player = input(text)
        if player == "bye":
            print("Oh... See you next time!")
            sys.exit()
        else:
            return player
        
    # ################################################ BETTING FUNCTIONS ###############################################

    # INITIAL GAME BALANCES
    global player_balance
    global dealer_balance
    player_balance = 500
    dealer_balance = 500

    # ------------------------------ Define Player Betting ---------------------------------------

    def get_bet_amount():
        while True:
            if player_balance != 0:
                try:
                    player_bet = int(user_input(f"\nHow much would you like to bet? (Current balance: {player_balance}) "))
                    if player_bet == 0:
                        print("\nReally? C'mon, dude. You gotta place a bet...") 
                    elif player_bet <= player_balance and player_bet != 0:
                        return player_bet
                    else:
                        print("You don't have enough balance.")
                except ValueError:
                    print("Please enter a valid integer.")
            else:
                print("Your wallet is empty! Come back after when you have more points.")
                input("Press Enter to quit..")
                sys.exit()

    # ---------------------- Define Dealer Betting --------------------------------------------

    def get_dealer_bet_amount():
        if dealer_balance == 0:
            print("The dealer's broke! Come back to play again some other time.")
            input("Press Enter to quit..")
            sys.exit()
        else:
            return random.randint(1, dealer_balance)
    
    # ################################################ CALCULATE ACES ##################################################

    def calculate_hand(hand):
        hand_value = sum(card[2] for card in hand)
        for card in hand:
            if card[0] in 'Ace':
                if hand_value >= 11:
                    card[2] = 1                
                elif hand_value <= 10:
                    card[2] = 11
            hand_value = sum(card[2] for card in hand)
        return hand_value

    ################################################# FORMAT CARDS IN HAND ############################################

    def format_cards(hand):
        card_strings = []
        for card in hand:
            card_string = f"{card[0]} of {card[1]}"
            card_strings.append(card_string)
        return ', '.join(card_strings)

    # ####################################### DEALER-SPECIFIC FUNCTIONS ###############################################

    # --------------------------------- Dealer Turn ------------------------------------------- 

    def dealer_turn():
        global dealer_hand_value
        global dealer_hand
        print(f"The dealer's hand is revealed to be {format_cards(dealer_hand)}, totaling {dealer_hand_value}")
        while dealer_hand_value < 17:
                dealer_hand_value = dealer_hit()
        if dealer_hand_value >= 17 and dealer_hand_value < 21:
                dealer_decision()        
        return dealer_hand_value

    # -------------------------- Dealer Decision Algorithm ------------------------------------

    def dealer_decision():
        if dealer_hand_value >= 17:
            random_number = random.randint(1,6)
            weight = (dealer_hand_value)-17
            random_number -= weight 
            if random_number > 4:
                dealer_hit()
            elif random_number < 4:
                dealer_stay()          

    # ------------------------------------ Dealer Hit Function ---------------------------------

    def dealer_hit():
        dealer_pull = random.choice(deck)
        dealer_hand.append(dealer_pull)
        dealer_hand_value = calculate_hand(dealer_hand)
        print(f"\nThe dealer has chosen to hit! The dealer pulled a {dealer_pull[0]} of {dealer_pull[1]}!")
        print(f"\nThe dealer's hand is now {format_cards(dealer_hand)}, totaling to {dealer_hand_value}.")
        return dealer_hand_value

    # -------------------------- Dealer Stays function -----------------------------------------

    def dealer_stay():
        global player_balance
        global dealer_balance
        print("The dealer has chosen to stay.")
        return
            
    # ######################################## WIN, LOSE, and PLAY AGAIN FUNCTIONS ######################################

    # -------------------------- PLAYER LOSES!! :( -------------------------------------------------

    def you_lose():
        global player_balance
        global dealer_balance
        print("\nThe dealer wins!")
        print(f"\nYou lost, so the dealer earns your bet of {player_bet_amount}.")
        dealer_balance += (player_bet_amount + dealer_bet_amount)
        print(f"Your new balance is {player_balance}, and the dealer's balance is {dealer_balance}.")
        play_again()

    # ------------------------------- PLAYER WINS!! :D ----------------------------------------------

    def you_win():
        global player_balance
        global dealer_balance
        print("\nCongrats! You win!")
        print(f"\nYou get to take the dealer's bet of {dealer_bet_amount}.")
        player_balance += (player_bet_amount + dealer_bet_amount)
        print(f"Your new balance is {player_balance}, and the dealer's balance is {dealer_balance}.")
        play_again()

    # ---------------------------- CHECKS TO PLAY ANOTHER ROUND ---------------------------------------

    def play_again():
        while True:
            player = user_input("\nDo you want to play another round? (Y/N): ")
            if player.upper() in ['YES', 'YAS', 'YUP', 'Y', 'YA', 'YE']:
                os.system('cls')
                break
            elif player.upper() in ['NO', 'NAY', 'N', 'NAH', 'NA', 'NOPE']:
                print("Okay! See ya later!")
                sys.exit() # quits game
            else:
                print("Invalid input. Please enter Y or N.")



    # ------------------------------------------------------------------------------------------------------------------------
    ################################################## GAME START ############################################################
    # ------------------------------------------------------------------------------------------------------------------------


    # ------------------------------------------------ DEFINE THE CARD DECK -------------------------------------------------

    suits = ['Hearts', 'Diamonds', 'Spades', 'Clubs']
    ranks = ['Two', 'Three', 'Four', 'Five', 'Six', 'Seven', 'Eight', 'Nine', 'Ten', 'Jack', 'Queen', 'King', 'Ace']
    values = {'Two': 2, 'Three': 3, 'Four': 4, 'Five': 5, 'Six': 6, 'Seven': 7, 'Eight': 8, 'Nine': 9, 'Ten': 10,
              'Jack': 10, 'Queen': 10, 'King': 10, 'Ace': 0}
    deck = [[rank,suit,values[rank]] for suit in suits for rank in ranks]



    # ################################################# INTRO ###############################################################


    # ------------------------------------- ASK PLAYER ABOUT PLAYING BLACKJACK --------------------------------------------

    play_blackjack = True
    while play_blackjack == True:
        player = user_input('Want to play blackjack?\n')  
        player = player.upper()
        if player in ['YES', 'YAS', 'YUP', 'Y', 'YA', 'YE']:
            print("Great!")
            break  
        elif player in ['NO', 'NAY', 'N', 'NAH', 'NA', 'NOPE']:
            print("Respectable choice. Goodbye.")
            sys.exit()  
        else:
            print("Choose a valid answer please...")
    
    # --------------------------------------- ASK PLAYER ABOUT READING THE RULES? -----------------------------------------

    read_rules = True
    while read_rules == True: 
        player = user_input('Would you like to read the rules before playing? \n ')
        player = player.upper()
        if player in ['YES', 'YAS', 'YUP', 'Y', 'YA', 'YE']:
            os.system('cls') 
            print(rules)
            player = user_input("When you're finished reading the rules, press Enter when ready to begin.....")
            os.system('cls') 
            break  
        elif player in ['NO', 'NAY', 'N', 'NAH', 'NA', 'NOPE']:
            player = user_input("Time to play, press Enter to get started! \n")
            os.system('cls') 
            break
        else:
            print("Choose a valid answer please...")



    # ############################################## BETTING ########################################################



    # ------------------------------------------ PLAYER BETTING -----------------------------------------------------

    player_betting = True
    while player_betting == True:
        player_bet_amount = get_bet_amount()

        check_bet = True
        while check_bet == True:
            answer = input(f"Are you sure you want to bet {player_bet_amount}? ")
            if answer.upper() in ['YES', 'YAS', 'YUP', 'Y', 'YA', 'YE' 'Y']:
                os.system('cls')
                print(f"\nAlright! You've chosen to bet {player_bet_amount}.")
                break
            elif answer.upper() in ['NO', 'NAY', 'N', 'NAH', 'NA', 'NOPE']:
                player_bet_amount = get_bet_amount()
            else:
                print("Invalid input. Please try again.")

        player_balance -= player_bet_amount

        #----------------------------------------------- DEALER BETTING ----------------------------------------------
        
        dealer_bet_amount = get_dealer_bet_amount()
        print(f"\nThe dealer has chosen to bet {dealer_bet_amount}.")

        # Deduct dealer's bet amount from dealer's balance
        dealer_balance -= dealer_bet_amount
    
        player = user_input("\nThe bets have been placed! Press Enter to continue...")
        os.system('cls') 



        # ############################################### DEALING HANDS #####################################################

        # PLAYER HAND 
        player_hand = []

        # DEALER HAND
        global dealer_hand
        dealer_hand = []

        # DEALING CARDS 
        player_pull = random.choice(deck) # Player pulls a card
        dealer_pull = random.choice(deck) # House pulls a card
       
        # FIRST DEALINGS 
        for i in range(4): 
            if i % 2 == 0:
                # Dealer deals a card to self
                dealer_pull = random.choice(deck)
                dealer_hand.append(dealer_pull) 
            else:
                # Player is dealt a card
                player_pull = random.choice(deck)
                player_hand.append(player_pull)

        global dealer_hand_value
        dealer_hand_value = calculate_hand(dealer_hand)

        print(f"\nThe dealer has one card face up, and one card face down. The face-up card is {dealer_hand[1][0]} of {dealer_hand[1][1]}.") 

        player_hand_value = calculate_hand(player_hand)
        print(f"\nYou have {format_cards(player_hand)}, so your hand value is {player_hand_value}.")

        ################################################# PLAYER AND DEALER TURNS #################################################

        # ------------------------------------------------- PLAYER TURN -----------------------------------------------------------
       
        player_turn = True
        while player_turn == True:
            player_hand_value = calculate_hand(player_hand)
            player = user_input('\nWould you like to Hit or Stand?\n')
            os.system('cls')
            if player_hand_value == 21:
                you_win()
            elif player in['Hit','hit']:
                os.system('cls')
                player_pull = random.choice(deck)
                player_hand.append(player_pull)
                print(f"You pulled a {player_pull[0]} of {player_pull[1]}!")
                player_hand_value = calculate_hand(player_hand)
                print(f"\nYour current hand is now {format_cards(player_hand)}, totaling to {player_hand_value}.")
                if player_hand_value > 21:
                        print("\nWhoops! Your hand busted.")
                        you_lose()
                        break
                elif player_hand_value == 21:
                        you_win()
                        break
            elif player in['Stand','stand']:
                    print('You decided to stay.')
                    user_input("Press Enter to move on to the dealer's turn.")
                    os.system('cls')

                    # ------------------------------ DEALER TAKES THEIR TURN --------------------------------------

                    dealer_hand_value = dealer_turn()
                    if dealer_hand_value > 21 or (player_hand_value < 21 and (player_hand_value > dealer_hand_value)):
                            you_win()
                    elif dealer_hand_value == 21 or (dealer_hand_value < 21 and (player_hand_value < dealer_hand_value)):
                            you_lose()      
                    break           
            else:
                print('Please choose to Hit or Stand')
                          
main()

