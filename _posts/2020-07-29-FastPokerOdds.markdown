---
layout: post
author: Danilo Vilardi
title: Insights on Mobile App Development and Poker Combinatorics
permalink: InsightsAndCombinatorics
---

![](/FastPokerOdds/assets/InsightsPost2020/00-title.png)


Summary: I developed a Poker Odds Calculator App ([download on the iOS app store](/download/)) and wrote this post to present some insights and details of its development, as well as some Poker-related combinatory facts.


## Introduction


A few months ago, I got back to playing online Texas Hold'em Poker. I noticed my mind is awful at evaluating my odds of winning a table. You'd think that, after reading Kahneman's "[Thinking, Fast And Slow](https://www.amazon.com/Thinking-Fast-Slow-Daniel-Kahneman/dp/0374533555)", a person would be immune to cognitive biases and get better at evaluating poker table odds. But we always tend to overestimate our chances.


In this decade-long era of "there's an app for that", I searched for a mobile application that would do the math for me. The idea is: I'd present the app with the current state of the table (my cards, the community cards and the number of players active in that table) and the app would spit out the odds of me winning that particular table at that moment (see the visual representation example below).


![](/FastPokerOdds/assets/InsightsPost2020/01-example.png)


There are a few solutions on the iOS App Store and Google Play Store. However, they all have usability downsides which bear them unfit as a helper tool in an online poker match. Some matches give you no more than 20 seconds to decide your next move, so an app journey cannot be slower than that.


Failing to find an available solution, I decided to make my own app. As an enthusiast programmer with no formal CS background, mobile App development has been a "distant utopic dream" of sorts: it is something I've always wanted to experiment but never felt like up for the task. There's nothing like 4 months of social distancing to push you out of boredom and face the challenge.


As simple as it may sound, there are only two main requirements for a well-built poker calculator app:


* **The entire usage journey must be quick:** From selecting all the cards to receiving the odds results, the user must take no more than 15 seconds. This requires a well-designed UI and a fast probability code.


* **Results must be reliable:** the app must provide probabilities within a small margin of error, or else it will jeopardize the player's decision-making process.


## Calculating the odds of winning a poker match


Imagine the following scenario: 3 players are active. You are player 1 and you know the cards of player 2, but you don't know the cards of player 3. You're on the pre-flop stage (no community cards are shown yet):


<p align="center"><img width="350" src="/FastPokerOdds/assets/InsightsPost2020/02-example.png"></p>


How do you code an app that calculates your odds of winning at this point? The answer may seem obvious at first. Your intuitive response might be "make the app loop through all remaining possibilities and check how many times player 1 wins".


However, bear in mind that there are nearly 150 nonillion (150 followed by 30 zeroes) possible 9-player table card combinations (excluding inner permutations). We couldn't possibly run them all.


![](/FastPokerOdds/assets/InsightsPost2020/03-totalPokerCombins.png)


**Solution:** rather than exploring all 150 nonillion possibilities (which would take more time than the universe's age), we could loop through a smaller sample of possibilities, randomly placing cards on the remaining slots and evaluating which player wins each random match. This approach, known as a **Monte Carlo Method**, is broadly used in physics, engineering, economics, banking and other areas:


![](/FastPokerOdds/assets/InsightsPost2020/04-results.png)


Monte Carlo is a method based on the **Law of Large Numbers**, which states that, for a sufficiently large sample of events (event example: random simulations of a poker match), the perceived frequency of occurrences of an outcome (outcome example: player 1 wins) will approximate the theoretical probability of that outcome happening. One easy way to visualize the law of large numbers at play is to roll 100,000 dices (don't do that) and sum the number of times that the number 6 appears. You might not get exactly 16,667 appearances (the theoretical expected value), but the real number will be fairly close to it. Another way of observing the Law of Large Numbers is to play with a [Galton Board](https://www.youtube.com/watch?v=EvHiee7gs9Y), an office table toy which uses small pins to distribute marbles in a random, yet ordered manner.


At this point, you might be wondering: "100,000 samples is **too small** when compared to the original 150 nonillions, right? How large of a random sample is reliable enough?". Surprisingly, not much. A convergence test shows that the margin of error gets sufficiently small (below 2%) with as little as 2,000 simulations. Beyond 10,000 the margin is lower than 1%.


<p align="center"><img width="500" src="/FastPokerOdds/assets/InsightsPost2020/05-convergence.png"></p>


There is no need to go past 10,000 simulations, as the accuracy gain is asymptotic-logarithmic and the decision-making process certainly won't change due to small fluctuations between 10,000 and 100,000 simulations. It is wiser to sacrifice 0.5% of margin to have an app that is 10 times faster.


I find this uniquely beautiful: with 0.00000000000000000000000001% of all possible poker match combinations, a Monte Carlo algorithm is able to define the odds of all players with a precision of ±1%.


## Teaching Poker rules to a computer


Explaining the basic rules of Texas Hold'em Poker to a person is simple. Of course, knowledge of strategies based on player position, number of outs, equity and pot size comes with experience. But here are the basics:


* There might be up to 9 players on the table, and each player receives 2 cards.

* There is a community stack of 5 cards.

* There are 4 betting moments on the match: pre-flop (before the community cards are placed), post-flop (after 3 cards are placed on the community stack), post-turn (after the 4th cards is placed) and post-river (after the 5th cards is placed). If a player bets a value, the remaining players must either match or raise the bet to continue playing. Otherwise, the player "folds" and do not fight for the pot on that match. The betting order shifts after each match.

* In total, each player has 7 cards (their 2 own cards and the 5 cards on the community stack). Their hand strength is based on the best combination of 5 cards out of the 7.

* There are nine possible 5-card hands ranked by strength. There are tie breakers within the hand combinations based on the highest cards A > K > Q > J > T > 9 > 8 > 7 > 6 > 5 > 4 > 3 > 2:


![](/FastPokerOdds/assets/InsightsPost2020/06-hands.png)


The table above is fairly easy for a human to interpret and use when evaluating their hands. When programming an App to do the same task, it gets more complicated:


## Hand Scoring Method A - Control Flow Evaluation


The easiest way of programming a 5-card hand score is by evaluating it from the highest possible combination (Straight Flush) to the lowest (no combination):


![](/FastPokerOdds/assets/InsightsPost2020/07-flow.png)


An in-depth view of a hand strength scoring algorithm can be seen [here](http://www.mathcs.emory.edu/~cheung/Courses/170/Syllabus/10/pokerCheck.html).


Texas Hold'em Poker is a 7-card game, however. There are 21 (7 choose 5) possible combinations of 5 cards for each player (considering their 2 cards plus the 5 cards on the community stack). Thus, the control flow above must be evaluated 21 times to get the maximum score of all 21 combinations.


Considering the first requirement for the App (**The entire usage journey must be quick**), this method surely does not look promising, as it will need a ton of unnecessary repetition and nested conditionals.


## Hand Scoring Method B - Pre-computed Score Tables


One could loop through all possible 5-hand combinations, applying method A (control flow evaluation) and then storing the score result in a look-up table.


There are 311,875,200 possible 5-card combinations. Excluding permutations (5! = 120), this number reduces to 2,598,960. It is still a significantly large number to store in a look-up table. We can do better than this.


Take a look at the combinations below. They are different from one another. However, they still have the same strength (pair of aces, 10-5-3 kickers), thus they have the same score:


![](/FastPokerOdds/assets/InsightsPost2020/08-equivalence.png)


This shows something interesting: even though there are 2,598,960 different hand combinations, they all collapse into a much smaller number of hand scores. Once again, combinatorics surprises us and tells us that this final number is only 7,462.


<p align="center"><img width="350" src="/FastPokerOdds/assets/InsightsPost2020/09-scoreRank.png"></p>


The next step is to create a "hash" function that will translate all 2,598,960 hand combinations into only 7,462 without spending too much computing time. Otherwise, we would still take too long to apply this method in the Monte Carlo algorithm. To solve this issue, the app uses the cleverness of [this guy](http://suffe.cool/poker/evaluator.html), which uses a neat encoding trick with prime number correspondence and multiplications.


## Bringing the App to life


Development of a simple app such as this consists of 2 parts, mainly: user interface (UI) design and back-end programming. For those familiar with object-oriented programming, coding the app is quite easy once you get familiar with the tool (Xcode) and language (Swift). The real challenge is to design an intuitive and quick-to-use interface with no design experience. How can one fit an entire poker table into a single app screen and yet make everything readable and accessible?


The main drivers behind this app's interface design are:


* Minimize the number of clicks (taps) required to set up the poker table (players' cards, community cards, players' ranges)

* Facilitate the visualization of relevant data (active players, active cards, card deck, win-tie-loss odds)

* Provide secondary visualization of more in-depth simulated statistics (hand combination probability, tie & loss rates etc.)


After some dozen hours deep diving into online courses in app development, vector graphics and design ([iOS13 & Swift 5 course](https://www.udemy.com/course-dashboard-redirect/?course_id=1778502) / [Adobe XD course](https://www.udemy.com/course/curso-de-adobe-xd/learn/)) and another 100+ hours coding and setting up the layout, the main screens of the app were developed. Aiming for simplicity, the app has as little views as possible:


![](/FastPokerOdds/assets/InsightsPost2020/10-appScreens.png)


* **View 1**: main dashboard. Contains the player's cards, enable/disable buttons, win rate labels, calculate button etc.

* **View 2((: card selection dashboard. Aims for quick usage, as all cards appear at once on the screen. Some apps do a 2-step card selection (first value, then suit), which takes too much of the user's time.

* **View 3**: range selection dashboard. One necessary feature of all Poker calculators is the range selection, which limits your opponents' hand possibilities. It is safe to assume that most players wouldn't maintain 2-7 off-suit (the worst possible poker hand), so the Monte Carlo algorithm must consider that constraint when calculating the odds of all players winning the match. The range controller is a simple slider, ensuring quick action in defining weak hand thresholds.

* **View 4**: in-depth statistics. This view presents more details on the Monte Carlo simulation for the active players: win-tie-loss rates and probability of reaching a specific hand.


## Results and final remarks


After 3 weeks of development + 1 week of bureaucracy, the app is active and running on the iOS App Store ([iPhone link](http://127.0.0.1:4000/privacy-policy-iPhone/)).


![](/FastPokerOdds/assets/InsightsPost2020/11-appStoreScreen.png)


Surprisingly, the step I believed would be the hardest - programming all poker-related algorithms - was the easiest of the entire project, whereas the steps I thought would be easier - layouts, UI design, app store submission and bureaucracy - turned out to be the most tiresome.


As a final customer of my own project, I believe the app succeeded in serving its main purpose. Its usage journey is fast and the results are extremely reliable. Also, this project provided some interesting insights:


* **Don't reinvent the wheel**: deeply research your subject and learn from other people's projects before coding hundreds of lines of code you may end up throwing away. Of course, be sure to give other people credit when credit is due: [Cactus Kev's prime number hand encoding algorithm](http://suffe.cool/poker/evaluator.html) is responsible for making this app as quick to calculate odds as it is now.
Use the right tools for the right tasks: Apple's Xcode is a great coding platform, but not that great of a UI design software. In early iterations, I tried designing the interface within Xcode, which turned out to be a nightmare. I then used a proper UI mockup tool for that specific task (Adobe XD). After a [7-hour course](https://www.udemy.com/course/curso-de-adobe-xd/learn/), I was able to learn the operational basics and the results were much better than what I was achieving with Xcode.

* **Don't overestimate the complexity of your project**: when you haven't started the project, it is easy to feel overwhelmed by the amount of work required to complete it, and sometimes this discourages you from pursuing it. Break it down into small sub-projects and tasks and work one step at a time. I first started programming the entire Poker algorithm and without even knowing how to build a good UI ("that's a problem for me to worry about next week"), nor how to go through the entire process and bureaucracy of submitting the app for revision ("that's a problem for me to worry about two weeks from now").


## Next steps


![](/FastPokerOdds/assets/InsightsPost2020/12-iPadDev.png)


I am currently working on an iPad version, which leverages on its bigger screen size to reduce the number of screens and steps needed to calculate the odds. I expect it to be available on the App store mid-August.


What about Android? Since I chose swift and Xcode frameworks for this project, I would need to recreate the entire project from scratch if I wanted to offer an Android option. The new framework that "all cool kids are into right now" is [Flutter](https://flutter.dev/): a framework developed by google which theoretically closes the gap between Android and iOS development. I'll make sure to investigate it for the next projects.