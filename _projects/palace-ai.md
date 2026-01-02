---
layout: post
title: "Training an AI to play the Palace Card Game"
description: "Training framework using PyTorch to train a neural net to play the card game 'Palace' against two other players."
date: 2025-12-30
categories: software
featured: true
---
{% include mathjax.html %}

# The Goal

Since I was taught how to play Palace by my girlfriend, I think my record when playing against her stands at 1 - 49. In order to fix my losing streak, I thought it would be the perfect opportunity to use my experience in developing deep learning models to create an AI Palace player so that I can practice against them and learn about the strategies that they develop. 

# Implementation of the game rules

Focusing only on the output for now, the AI has the following options:

1. **Action Index (0 - 12), (13 - 25), (26 - 38), (39 - 51).** Each of these sets of action indices corresponds to playing either 1, 2, 3, or 4 of a card of rank 0 - 12 (2 - Ace). 
2. **Action Index (52-64).** These actions correspond to playing a card of rank 0 - 12 from the player's face up pile.
3. **Action Index (65-77).** These actions correspond to playing a card of rank 0 - 12 from the player's face down pile.
4. **Action Index 78.** This action corresponds to picking up the pile.

The neural net has the opportunity to pick any of these actions, no matter the conditions imposed upon it by the available cards in its hand or the top card of the discard pile. *However*, the conditions are imposed by a masking vector. This masking vector imposes the game logic on the vector of possible outputs from the neural net. That logic is shown below:

```python
def get_valid_mask(hand, discard_pile, face_up_pile, face_down_pile):
    mask = np.zeros((6, 13), dtype=bool)
    
    # Check if the player possess each rank in any pile
    for rank in range(13):
        # Hand play (1-4 cards)
        for num_cards in range(1, 5):
            if hand[rank] >= num_cards:
                mask[num_cards - 1, rank] = True

        # Face-up play (Only if hand is empty)
        if np.sum(hand) == 0 and face_up_pile[rank] > 0:
            mask[4, rank] = True

        # Face-down play (Only if hand and face-up are empty)
        if np.sum(hand) == 0 and np.sum(face_up_pile) == 0 and face_down_pile[rank] > 0:
            mask[5, rank] = True

    mask_pickup = len(discard_pile.cards) > 0

    # discard pile restrictions
    if mask_pickup:
        top_card = discard_pile.cards[-1]

        # check for three first
        if top_card == 1:
            for rank in range(13):
                if rank != 1:
                    mask[:, rank] = False

        else: # normal restrictions
            for rank in range(13):
                is_wild_card = (rank == 0 or rank == 1 or rank == 5 or rank == 8)
                if not is_wild_card:
                    if top_card == 0: # 2 is on top
                        break # Everything already True from possession check stays True
                    elif top_card == 5: # 7 is on top
                        if rank >= 6: 
                            mask[:, rank] = False
                    elif rank < top_card:
                        mask[:, rank] = False

    # Flatten and add the pickup action (Index 78)
    return np.append(mask.flatten(), mask_pickup).astype(bool)
```

While this imposes restrictions on the possible actions that the neural net is able to take, it is up to a function with the game environment to determine the results of these actions. For example, while it is possible for the neural net to play any one of its face down cards, it is forced to make a random decision since it should not know what cards it has. 

# How is the neural net making its decision?

The input vector fed into this neural net is given as follows:

1. **Last Actions (0 - 12), (13 - 25), (26 - 38).** Each set of indices corresponds to the last three actions taken to the discard pile (where (0 - 12) is the most recent action.) Each rank can be populated by a number 1 through 4 representing the number of cards of that type that were put down. Including this information in the input vector allows the neural net to access some memory of how the game has been played up until the current turn. 
2. **Current Hand (39-51).** Each index corresponds to the rank available in the player's hand with a number 0 - 4 representing how many of that rank they have in their hand.
3. **Valid Mask (52-131).** Each index corresponds to the mask as described above. 

Using this input vector, the neural net generates a vector of probabilites for each action that it wants to take in the form of its output vector. These outputs are masked by the valid mask, the choice is made, positive and negative rewards are calculated for the choice, and the turn passes onto the next player. 

At the end of the game the rewards for each turn are aggregated and discounted based on their distance from the end of the game. Loss is calculated from the normalized returns that the player gets from these rewards (in combination with an entropy factor) and this loss is backpropagated through the neural network. Integrated into this loss is the valid mask that was used to help the player make the legal decision. As a result, during backpropagation, the neural net is predisposed to learning not to make an illegal move very quickly.

# The training scheme. 

Because Palace is a game influence by random chance as well as strategy, it is important to use batching of games played so that the neural net isn't influenced by an extremely lucky or extremely unlucky game. The batch is set to a relatively small number here, either 8 or 16 games. Only after a batch of games is played, the total loss and its backpropagation will be calculated for each neural net.

A generation will consist of 10 times the number of games in a batch. After a generation, the neural nets engage in a "tournament" to decide who the best player is. During this tournament losses (where a player is the last one left with cards) are recorded over games in the amount of 20 times the number of players. In order to make the training more stable, I made it hard to overthrow the best player (the king) and easy to remain the king. This means that if the king loses less than or equal to it's equal share of games (20) it remains the king. Additionally, even if it loses more than 20, if no other player loses less than 15 games, it still keeps its title. 

After the tournament, the king neural net has its weights saved so that for the next generation it remains static and plays with the other learning nets. This means that each learning net is always playing against the current best neural net and learning a strategy to beat them. 

# Next steps!

I want these players to be able to beat even the most experienced Palace player. In order to do so, I plan to do the following things:

1. **Increase Network Complexity.** Right now, the network is a fairly simple feed forward network. While this may be enough to learn a small amount of strategy, it might not allow the network to learn effective strategies for the different stages of the game. Additionally, integrating a structure like a (Long Short Term Memory) LSTM component might help the neural net keep track of what the other players have in their hands without actually seeing their current hand. 
2. **Make Changes to the Input Vector.** Right now, the neural net can not see what the other players have in their face-up piles. This should be a part of the input vector so that the neural net knows not to let them be able to place cards from this pile to easily. 
3. **Refine the reward system.** Right now, rewards are doled out based only on my ideas of what good and bad moves might be in the game and how good or bad they might be. However, this is the key to effective learning for this training scheme. It will be important to refine the weights given to the different kinds of decisions the neural net makes. 

# Can you beat these players?

It's easy to play the game! Download the python files [here](https://github.com/ma-casali/palace-ai) and you can run a game from your terminal using the pre-saved weights or you can retrain the AI yourself!

Here's an example of what the first turn in a game can look like:

![Palace Game Screenshot](/assets/images/PalaceAIGameScreenshot.png)

