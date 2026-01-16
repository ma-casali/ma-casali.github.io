---
layout: post
title: "Training an AI to play the Palace Card Game"
description: "Training framework using PyTorch to train a neural net to play the card game 'Palace' against two other players."
date: 2025-12-30
categories: software
title-image: "/assets/images/Palace-title.png"
featured: true
---
{% include mathjax.html %}

* Table of Contents
{:toc}

# The goal

Since I was taught how to play Palace by my girlfriend, I think my record when playing against her stands at 1 - 49. In order to fix my losing streak, I thought it would be the perfect opportunity to use my experience in developing deep learning models to create an AI Palace player so that I can practice against them and learn about the strategies that they develop. 

# Neural net description

For this game, I decided to use a combination neural net with two Long Short Term Memory (LSTM) neurons, and a parallel Multi-Layer Perceptron (MLP) network. I decided to implement this after training for some time with a simple MLP network and having the network take too much time to learn simple strategies (like saving higher cards for later play). The neural network contains 93,000 degrees of freedom, which compared to AlphaZero's millions of degrees of freedom (for a much more complex game), is fairly lean. 

# Game rules implementation

Focusing only on the output for now, the AI has the following options:

1. **Action Index (0 - 12), (13 - 25), (26 - 38), (39 - 51).** Each of these sets of action indices corresponds to playing either 1, 2, 3, or 4 of a card of rank 0 - 12 (2 - Ace). 
2. **Action Index (52-64).** These actions correspond to playing a card of rank 0 - 12 from the player's face up pile.
3. **Action Index (65-77).** These actions correspond to playing a card of rank 0 - 12 from the player's face down pile.
4. **Action Index 78.** This action corresponds to picking up the pile.

The neural net has the opportunity to pick any of these actions, no matter the conditions imposed upon it by the available cards in its hand or the top card of the discard pile. *However*, the conditions are imposed by a masking vector. This masking vector imposes the game logic on the vector of possible outputs from the neural net. That logic is shown below:

```python
def get_valid_mask(self):
    mask = torch.zeros((self.batch_size, 6, 13), dtype=torch.bool, device= self.device)
    batch_ids = torch.arange(self.batch_size, device=self.device)

    # get the current player's piles
    hand = self.hands[batch_ids, self.active_players]
    faceup_pile = self.face_up_piles[batch_ids, self.active_players]
    facedown_pile = self.face_down_piles[batch_ids, self.active_players]

    is_hand_empty = (hand.sum(dim=1) == 0)
    is_faceup_empty = (faceup_pile.sum(dim=1) == 0)

    # 1. Play from hand
    for num in range(1, 5):
        mask[:, num - 1, :] = (hand >= num) & ~is_hand_empty.unsqueeze(1)

    # 2. Play from face-up pile
    mask[:, 4, :] = (faceup_pile > 0) & is_hand_empty.unsqueeze(1) & ~is_faceup_empty.unsqueeze(1)

    # 3. Play from face-down pile
    mask[:, 5, :] = (facedown_pile > 0) & is_hand_empty.unsqueeze(1) & is_faceup_empty.unsqueeze(1)

    # 4. Apply restrictions from table
    ranks = torch.arange(13, device = self.device)
    is_wild = (ranks == 0) | (ranks == 1) | (ranks == 5) | (ranks == 8)

    # three is top card
    invalid_3 = (self.top_cards == 1).unsqueeze(1) & (ranks.unsqueeze(0) != 1)
    mask[:, :5, :] &= ~invalid_3.unsqueeze(1)
    # top card on discard pile
    invalid = (ranks.unsqueeze(0) < self.top_cards.unsqueeze(1)) & (~is_wild.unsqueeze(0))
    # seven is top card
    invalid = torch.where((self.top_cards == 5).unsqueeze(1), (ranks.unsqueeze(0) >= 6) & (~is_wild.unsqueeze(0)), invalid)

    mask[:, :5, :] &= ~invalid.unsqueeze(1)
    flat_mask = mask.view(self.batch_size, 78)

    #5. Pickup option
    mask_pickup = (self.top_cards != -1) & (self.discard_counts.sum(dim=1) > 0)
    pickup_tensor = mask_pickup.unsqueeze(1)

    return torch.cat([flat_mask, pickup_tensor], dim=1)
```

While this imposes restrictions on the possible actions that the neural net is able to take, it is up to a function with the game environment to determine the results of these actions. For example, while it is possible for the neural net to play any one of its face down cards, it is forced to make a random decision since it should not know what cards it has. 

# How is the neural net making its decision?

The static input vector is structured as follows (B will be used to represent the batch size): 

1. **The Player's Current States.** This includes the players hands as a frequency count vector of ranks from 0 - 12, the face up cards in the same frequency count vector format, and the number of face down cards remaining in their pile. This results in a tensor of shape (B, 27).

2. **The Opponent's Current States.** This is similar to the players hands for each opponent, but it hides the card ranks in the opponent's hand and only provides the number of cards in their hand. This results in a tensor of shape (B, 30) for two opponents.

3. **The Table State.** This includes things like the frequency count of the ranks within the discard pile, the top card on the discard pile, the current run count (number of cards of the same rank in a row on the discard pile), and the length of the draw pile. This results in a tensor of shape (B, 16)

Additionally, the neural net relies on a history of actions taken by all of the players. For the time being, I am limiting the length of this history vector to 6 items. Each action corresponds to an action index from the output vector as described above. 

Using this input vector, the neural net generates a vector of probabilites for each action that it wants to take in the form of its output vector. These outputs are masked by the valid mask, the choice is made, positive and negative rewards are calculated for the choice, and the turn passes onto the next player. 

At the end of the game the rewards for each turn are aggregated and discounted based on their distance from the end of the game. Loss is calculated from the normalized returns that the player gets from these rewards (in combination with an entropy factor) and this loss is backpropagated through the neural network. Integrated into this loss is the valid mask that was used to help the player make the legal decision. As a result, during backpropagation, the neural net is predisposed to learning not to make an illegal move very quickly.

# The training scheme

Because Palace is a game influence by random chance as well as strategy, it is important to use batching of games played so that the neural net isn't influenced by an extremely lucky or extremely unlucky game. The batch is set to a size of 256 in order to mitigate these random fluctuations. After 20 generations of playing, the best neural net is decided. This happens every 20 generations. These weights are then kept static throughout the remainder of training, so that the learning neural nets can play against it. If it is beaten in the future, the neural net that beat it will have its weights saved. 

The learning rate is attenuated according to a cosine annealing scheme with warm restarts while the entropy is attenuated according to a linear scheme with restarts at half its value. Other training meta-parameters like initial and final learning rate and entropy are decided by a sweeping search conducted by a Weights and Biases bayesian sweep focused on minimizing a patience criterion equivalent to a mix between the stalemate rate, a measure of how many games in the batch resulted in a stalemate, and a ratio of the average game length over the max game length. This is an ideal metric, because it balances both the desire for the neural net to learn how to finish games while also learning how to finish them quickly. 

# The reward system

One of the most important aspects of training is designing a reward system that ensures that the neural nets learn viable strategies at various points throughout the game. For example, you would not want to create a reward system that heavily penalizes a player for committing to a move that may not be good all the time, but a good idea at very precise times (like picking up the discard pile).

The reward scheme is as follows:

| Category of Reward | Type of Reward       | Reward / Penalty Value |
| ------------------ | --------------       | ---------------------- |
| Frequency/Urgency  | Step                 | -0.1                   |
|                    | Stalemate            | -20.0                  |
| Hand Management    | Card Played          | 0.2                    |
|                    | Per Card Bonus       | 0.02                   |
|                    | Hand Size            | -0.1                   |
| Game Milestones    | Burn Pile            | 2.0                    |
|                    | Pick up              | -0.05                  |
|                    | Pick up Per Card     | -0.02                  |
|                    | Reached Face Up      | 1.0                    |
|                    | Reached Face Down    | 1.0                    |
| Winning / Losing   | Win                  | 5.0                    |
|                    | Lose                 | -5.0                   |
|                    | Card Diff. (Win)     | 0.01                   |
|                    | Card Diff. (Loss)    | -0.05                  |
| Wildcard Bonuses   | Played 2             | 0.1                    |
|                    | Played 3             | 0.5                    |
|                    | Played 7             | 0.3                    |
|                    | Played 10            | 0.7                    |
| Misc.              | Forced a Pickup      | 0.5                    |

This reward scheme heavily penalizes a stalemate, with a penalty that is four times greater than a loss or a win. This in combination with a per-game-step penalty forces the neural nets to play faster, more conclusive games. Most of the other rewards are focused on penalizing the player for being in a losing state (where they have lots of cards in their hand and are having trouble playing cards) or rewards them for getting to different milestones and playing their wild cards to good effect. 

# SHAP analysis of a game

Using Shapley values to evaluate the impact of different input variables on the resulting action index of the output, we can draw up a plot of what categories of information the neural network was most using to make its decision for a play. 

![SHAP Analysis of a Game](/assets/images/shap_analysis.png)

Here, we have a series of bar plots for each turn that Player 0 has taken where each depicts the portion of the decision making process that can be attributed to each input category. These categories are: 
 - the players hand and piles (cyan), 
 - opponent 1's hand and piles (yellow), 
 - opponent 2's hand piles (magenta), 
 - the table state (red), 
 - and the history series fed into the RNN (grey).

Below the plot are small descriptions describing the turns taken by the player (in bold) and opponent's turns. This plot is powerful because it illuminates how the neural net is making its decision and how the decision making structure changes throughout the course of a game. These decisions can show how the neural net has begun to learn the game rules (it makes sure the draw pile is empty before playing a face up card) or which opponent it's basing its decision on before playing a low card or punishing wild card.

One of the most surprising results from this analysis is the lack of reliance on the history of actions when making its decision. This may be due to most of the play history being included in the information from the discard pile, but as I refine the training of this neural net, it will be interesting to see whether or not reliance on the history sequence increases. I suspect it will increase because analyzing what one's opponents are playing and when they are playing it, allows a player to better understand what the opponent is planning for the future. Since this is a higher level strategy, I am assuming that the player has not reached that level of literacy within the game.

In addition to this plot, I also save a file that describes the top three specific inputs that influenced the player's action. Here's a sample of what that would like for one game:

1. **Played 2 Ks** because: the drawpile had 25 cards left, player had 2 K, and player had 2 faceup A.
2. **Pickup** because: the drawpile had 21 cards left, there was 2 K in the discard pile, and player had 2 8.
3. **Played 2 Ks** because: the drawpile had 19 cards left, player had 2 K, and player had 2 8.
4. **Played 2 8s** because: there was 2 K in the discard pile, player had 2 8, and there was 1 5 in the discard pile.
5. **Played a 7** because: there was 3 8 in the discard pile, there was 2 K in the discard pile, and the top card on the pile was a J.
6. **Played a 10** because: there was 4 8 in the discard pile, there was 2 K in the discard pile, and player had 1 10.
7. **Played a 6** because: the drawpile had 7 cards left, player had 2 faceup A, and player had 3 facedown cards.
8. **Played a 2** because: there was 1 5 in the discard pile, player had 1 4, and there was 1 6 in the discard pile.
9. **Played a 10** because: player had 1 10, the top card on the pile was a Q, and there was 1 5 in the discard pile.
10. **Played a 4** because: the drawpile had 0 cards left, player had 1 4, and there was 0 K in the discard pile.
11. **Played F-U Q** because: opponent 2 had 1 cards in their hand, there was 1 9 in the discard pile, and there was 1 2 in the discard pile.
12. **Played F-U A** because: the drawpile had 0 cards left, there was 0 6 in the discard pile, and there was 0 5 in the discard pile.
13. **Played F-U A** because: the drawpile had 0 cards left, player had 1 faceup A, and opponent 1 had 0 cards in their hand.
14. **Played F-D** because: the drawpile had 0 cards left, player had 0 faceup A, and opponent 1 had 0 cards in their hand.
15. **Played F-D** because: the drawpile had 0 cards left, opponent 2 had 10 cards in their hand, and player had 2 facedown cards.
16. **Played F-D** because: player had 1 facedown cards, the drawpile had 0 cards left, and player had 0 faceup A.

In this history, we get a little bit of a better idea of what cards specifically are influencing the player's decision. For most of these decisions, it seems that the game rules are what the player is paying the most attention to. It is interesting to examine their ninth turn, where they play a 10. For this turn, they are making their decision primarily on the inclusion of the 5 in the discard pile. 

# Can you beat these players?

It's easy to play the game! Download the python files [here](https://github.com/ma-casali/palace-ai) and you can run a game from your terminal using the pre-saved weights or you can retrain the AI yourself!

Here's an example of what the first turn in a game can look like:

![Palace Game Screenshot](/assets/images/PalaceAIGameScreenshot.png)

