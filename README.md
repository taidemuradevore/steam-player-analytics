# Steam player data: what makes a game last?

I pulled two Kaggle datasets together to look at how games on Steam hold onto players after release. One dataset has the details of each game (price, tags, genres, reviews, release date), the other has monthly average and peak player counts going back to 2012. After joining and filtering, I had about 350,000 game-months covering roughly 3,300 games.

I started with three questions and ended up mostly proving myself wrong, which was the interesting part.

## 1. Does paying more for a game make people stick with it?

The idea was sunk cost. If you drop $40 on a game, maybe you keep playing it even when it's mediocre, because quitting means admitting you wasted the money. Free games should get abandoned fastest.

To test it I only looked at badly reviewed games (under 60% positive) and measured "half-life": how many months until a game's monthly peak falls below half of what it was at launch.

| Price tier | Median half-life | Mean | Games |
|---|---|---|---|
| Free | 2 months | 5.4 | 8,355 |
| Budget (under $15) | 3 months | 6.5 | 16,327 |
| Standard ($15+) | 2 months | 6.8 | 5,702 |

Budget games last a bit longer than free ones. But the trend doesn't keep going as price goes up, which is what sunk cost would predict. Expensive games die about as fast as free ones. My guess is that people are less patient with a bad game they paid real money for. A $5 indie game that's rough around the edges gets some goodwill; a $40 one that's rough gets refunded.

One thing I noticed along the way: there were no badly reviewed AAA games in the data at all. The worst-reviewed game over $60 still had 80% positive reviews. That's why the price buckets got redrawn partway through.

## 2. Do multiplayer games grow more after launch?

I assumed yes. Friends invite friends, the game snowballs.

Comparing each game's all-time peak to its launch peak, single-player games actually came out ahead (median growth of 3.4x versus 2.6x). But that's misleading, because multiplayer games launch much bigger. If you start with 50 players it's easy to multiply by ten. If you start with 5,000, it isn't.

So I switched to a metric that doesn't care about size: the share of months where a game gained players compared to the month before. Multiplayer games gained in 43.2% of months, single-player in 44.3%. Essentially no difference.

I then narrowed it to free multiplayer games, thinking no price tag plus friends should be the most viral combination possible. That was wrong too. Free multiplayer games had the largest launches of any group (median 741 players in the first month) and the weakest growth afterward. Paid multiplayer games launched smaller but grew far more, with the top 1% reaching 140x their launch peak against 61x for free ones.

## 3. Which genres actually outperform?

Raw growth numbers favor whatever launches small, so I needed to adjust for size. First I checked what the distribution of player counts looks like. I expected a power law, the usual rich-get-richer shape. A formal comparison said log-normal instead (likelihood ratio -11.8, p = 0.003), and a Q-Q plot on the logged data backed that up, apart from a heavier right tail than a clean log-normal would have.

That let me regress log(all-time peak) on log(launch peak). Launch size explains about 68% of where a game ends up (R² = 0.68). The leftover 32% is everything else: word of mouth, quality, updates, luck. I turned each game's residual into a z-score and called it a growth score, so a positive score means the game outgrew what its launch predicted.

Sorted by average growth score:

| Genre | Growth score | Median launch peak |
|---|---|---|
| Early Access | +0.15 | 470 |
| Action | +0.10 | 715 |
| Adventure | +0.04 | 534 |
| Indie | +0.03 | 426 |
| Racing | +0.01 | 321 |
| Casual | -0.02 | 382 |
| Simulation | -0.05 | 496 |
| Strategy | -0.07 | 608 |
| Sports | -0.13 | 465 |
| RPG | -0.15 | 775 |

Early Access on top makes sense: those games are supposed to grow as content gets added. RPGs at the bottom fit the opposite pattern, big launch, hype burns through the audience on day one. Sports games probably suffer from annual releases pulling players off last year's edition.

Worth noting that every genre has a negative *median* growth score even when its mean is positive. The averages are being carried by a handful of viral hits. The typical game in any genre underperforms its launch.

The same scoring applied to the multiplayer segments:

| Segment | Growth score | Median launch peak |
|---|---|---|
| Paid Multiplayer | +0.23 | 795 |
| Free Singleplayer | -0.06 | 366 |
| Free Multiplayer | -0.23 | 843 |
| Paid Singleplayer | -0.43 | 411 |

Paid multiplayer is the only segment that clearly beats expectations. Free multiplayer gets a big crowd on day one and loses it.

## Notes on the data

- Games with an all-time peak under 100 players are dropped. With tiny player counts, the growth ratio explodes on noise (going from 2 players to 40 is not virality).
- The regression and growth scores use a stricter cut: launch peak of at least 100, leaving 2,354 games.
- A few columns in the games dataset are unreliable. The `peak_ccu` field disagrees with the player-count data, so I dropped it and used the monthly data instead.
- Growth is measured against the first two months after release, not the first month, since release timing within a month varies.

## Files

- [data-cleaning.ipynb](data-cleaning.ipynb) — downloads both datasets, parses the games JSON, merges on app ID, builds the derived columns (months since launch, launch peak, gain consistency), writes `joined.csv`
- [analysis.ipynb](analysis.ipynb) — everything above
- `joined.csv` — the merged dataset, not committed (137 MB). Run the cleaning notebook to regenerate it.

## Running it

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Run `data-cleaning.ipynb` first. It downloads the data through `kagglehub`, so you'll need Kaggle credentials set up.

Sources: [fronkongames/steam-games-dataset](https://www.kaggle.com/datasets/fronkongames/steam-games-dataset) and [lunthu/steam-monthly-average-players](https://www.kaggle.com/datasets/lunthu/steam-monthly-average-players).
