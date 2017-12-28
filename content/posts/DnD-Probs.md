---
title: "Probability Analysis for Dungeons and Dragons"
date: 2017-12-20T00:04:14-05:00
draft: False
featuredImage : "img/dndprob.jpg"
---


In DND, players roll a 20 sided dice to determine if their actions are successful. The number they roll is contested by a difficulty check. If the player rolls a number higher than the difficulty check, the action is successful. For example, attempting to sneak past a guard might call for a player to roll a 15 or higher.

Within the newest edition of DND, there is a mechanic called advantage and disadvantage. They let the player roll the dice two times and take either the higher or the lower of the two numbers as their roll, respectively.

Below we see the probability distributions for rolling a certain number given a roll with advantage, disadvantage, or under normal circumstances.

#### Probability of Player Dice Rolls

``` r
uni <- rep(.05,20)
norm <- ggplot(data.frame(uni),aes(x=seq_along(uni),y=uni)) + geom_bar(stat="identity") +
  ylab("Probability of Roll") + xlab("Dice Roll") + ylim(c(0,.45)) + ggtitle("Probablity of Normal Dice Roll")

advantage <- seq(1:20)
for (i in 1:20){
  advantage[i] <- ((2*i-1)/20^2)
}
adv <- ggplot(data.frame(advantage),aes(x=seq_along(advantage),y=advantage)) + geom_bar(stat="identity") +
  ylab("Probability of Roll") + xlab("Dice Roll") + ylim(c(0,.45)) + ggtitle("Probablity of Dice Roll with Advantage")

disadvantage <- rev(advantage)
disadv <- ggplot(data.frame(disadvantage),aes(x=seq_along(disadvantage),y=disadvantage)) + geom_bar(stat="identity") +
  ylab("Probability of Roll") + xlab("Dice Roll") + ylim(c(0,.45)) + ggtitle("Probablity of Dice Roll with Disadvantage")

grid.arrange(norm, adv, disadv, ncol=3)
```

![](/Dice_Distros-1.png) The above graphs show an even probability for a normal dice roll- each side of the dice has a 5% chance of being rolled. We can see with advantage the probabilities slightly shift and the player has a better chance of rolling higher numbers. Similarly, disadvantage gives the player a slightly higher chance of rolling lower numbers.

#### Probability of NPC Dice Rolls

Sometimes, you aren't rolling against a single number to determine if you are successful, but many numbers. In the case of a rogue sneaking past many guards, the rogue will roll a dice vs every single guards' roll.

We know the probability of a player rolling a certain number, now we need to determine given N number of guards what is the probability of at least one of the guards rolling a certain number. Below we will show the probabilities for rolling values given 5, 10 and, 15 guards.

``` r
num_guards <- 5
prob5 <- seq(1:20)
for (n in 1:20){
  prob5[n] <- (n^num_guards-(n-1)^num_guards)/20^num_guards
}
five <- ggplot(data.frame(prob5),aes(seq_along(prob5),prob5)) + geom_bar(stat="identity") +
  ylab("Probability of Gaurd Roll") + xlab("Highest Dice Roll") + ggtitle("Probablity of Dice Roll Given 5 Guards") + ylim(c(0,.6))

num_guards <- 10
prob10 <- seq(1:20)
for (n in 1:20){
  prob10[n] <- (n^num_guards-(n-1)^num_guards)/20^num_guards
}
ten <- ggplot(data.frame(prob10),aes(seq_along(prob10),prob10)) + geom_bar(stat="identity") +
  ylab("Probability of Gaurd Roll") + xlab("Highest Dice Roll") + ggtitle("Probablity of Dice Roll Given 10 Guards") + ylim(c(0,.6))

num_guards <- 15
prob20 <- seq(1:20)
for (n in 1:20){
  prob20[n] <- (n^num_guards-(n-1)^num_guards)/20^num_guards
}
twenty <- ggplot(data.frame(prob20),aes(seq_along(prob20),prob20)) + geom_bar(stat="identity") +
  ylab("Probability of Gaurd Roll") + xlab("Highest Dice Roll") + ggtitle("Probablity of Dice Roll Given 20 Guards") + ylim(c(0,.6))

grid.arrange(five, ten, twenty, ncol=3)
```

![](/guards-1.png) Given 5 guards, the probability of one of them rolling a 20 is around 22%, with 20 guards that probability goes up to around 53%- the more guards there are, the more chances there are of one of them rolling a 20.

#### Probability of Success

We have the probabilities of the player rolling certain values, and the probabilities of the guards rolling certain values. We need one more piece of information. Each character has a modifier to the rolls they make. This represents how good the character is at the activity they are doing- a rogue might have a high modifier for stealth but a low modifier for strength. We don't need to know the guards' and rogue's individual modifiers, but we do need to know how much better the rogue is at sneaking than the guards are at perception. The graph below shows the rogue's probability of success for every relative difference between the 5 guards' and the rogue's modifiers.

``` r
encounter <- expand.grid(1:20, 1:20)
encounter <- cbind(encounter, prob = rep(prob5, 20)*.05)
encounter <- cbind(encounter, base = rep(prob5, 20))
encounter <- arrange(encounter, Var1)
encounter <- cbind(encounter, adv = rep(advantage, 20))
encounter <- cbind(encounter, disadv = rep(disadvantage, 20))
encounter$adv_prob <- encounter[,4]*encounter[,5]
encounter$disadv_prob <- encounter[,4]*encounter[,6]
success <- vapply(0:20, function(x) sum(encounter[encounter[,1] < (encounter[,2] + x), 3]), numeric(1))
adv_success <- vapply(0:20, function(x) sum(encounter[encounter[,1] < (encounter[,2] + x), 7]), numeric(1))
disadv_success <- vapply(0:20, function(x) sum(encounter[encounter[,1] < (encounter[,2] + x), 8]), numeric(1))

g <- ggplot() +
  geom_line(aes(seq_along(success)-1,success, color="Normal"), data.frame(success),size=1) +
  geom_line(aes(seq_along(adv_success)-1,adv_success, colour="Advantage"), data.frame(adv_success),size=1) +
  geom_line(aes(seq_along(disadv_success)-1,disadv_success, colour="Disadvantage"), data.frame(disadv_success),size=1) +
  scale_color_discrete(name = "Probablity Of Success") + theme(legend.position=c(0.8, 0.2)) + xlab("Relative Modifier Difference") +ylab("Probablity of Success") + ggtitle("Probability of Success Given 5 Guards")
g
```

![](/success-1.png)

The graph above shows that if a player has a modifier of 10 more than the guards, the rogue has a 60% chance of sneaking by 5 guards, and that is under normal rolls- with advantage that probability goes up to 80%. Even if we increase the number of guards to a theoretical infinite, the rogue has a 50% chance of being successful. Pass without a trace is a level 2 spell that gives +10 to stealth for the duration. Given what we know above, pass without a trace grants an incredible benefit to low level parties.
