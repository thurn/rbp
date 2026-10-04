# Bridge of Rogues: game design

Bridge of Rogues is a roguelike built on contract bridge. Four seats play eight
deals of duplicate-scored bridge with Standard American Yellow Card (SAYC)
bidding. Before each deal, players shop for **sigils** (up to five
scoring and rule-changing pieces), **engravings** (permanent changes to cards
they own), and **cards** (which are dealt to their owner every deal). In
single-player you are the only buyer on your side, with double gold and sigil
slots. The partnership with the higher score after deal 8 wins.

This document records the design agreed in the design interview of
2026-10-03; [Appendix A](#appendix-a-decision-log) lists each decision. The
predecessor experiment is Rogue Spades (`~/rsp`); its
[rules](../../rsp/docs/game-overview.md),
[archetypes](../../rsp/docs/archetypes.md), and
[scoring model](../../rsp/docs/sigils/scoring.md) are referenced where an idea
carries over.

## At a glance

| Topic | Rule |
| --- | --- |
| Run | 8 rounds; one round is one deal, with a shop before it |
| Bridge | Duplicate scoring; vulnerability follows boards 1–8; SAYC bidding |
| Scoring | Trick values sum, the six lowest are book, bonuses add, contract multipliers sum, doubling multiplies the result |
| Failure | Undertricks scale with the declaring side's contract multiplier |
| Sigils | 5 slots per shopping player, 7 for you in single-player; both partners' sigils apply to their side's contracts; all sigils are public; every sigil here is an example pending simulation |
| Engravings | Seven types, applied immediately to an owned card, one per card |
| Cards | Bought cards are dealt to their owner every deal; owned cards are private |
| Shop | 2 sigils, 2 engravings, 2 cards; unlimited purchases; Balatro-style random offers |
| Archetypes | Majors: five strains built from shared sigil cycles, plus Trumps, Long Suits, Slams, and Ranks; minors: Low Cards, Voids, Rainbow, Timing |
| Prototype | Single-player: you sit South with a non-shopping AI partner against two shopping AI opponents |

## Design pillars

1. **Real bridge underneath.** Without sigils, a deal plays and scores as
   duplicate bridge, so the game also teaches SAYC bidding and declarer play.
2. **Direct scoring.** Rogue Spades had many sigils that were fun alone but
   added up to no game plan. Here most sigils say "you get points for doing X"
   in one of four categories, Balatro-style, and builds come from stacking
   them.
3. **Every plan works in every strain.** Cycles of identical sigils make the
   beginner plan, collecting one suit or its top honors and bidding it, equally
   viable in all five strains. Strains differ through bridge itself, and
   cross-strain archetypes such as Trumps and Ranks give builds a second axis,
   the way Balatro jokers mix hand types with card ranks.
4. **Symmetric seats.** Human and AI seats follow identical rules, so one
   design serves single-player and multiplayer. The one exception is an AI
   partner of a human, which never shops; its human partner gets double gold
   and 7 sigil slots instead ([§8](#ai-partners-do-not-shop)).
5. **Every sigil is simulation-backed.** A sigil ships only when simulated runs
   show that a player who picks it can build around it, trigger it, and win
   because of it ([§15](#15-sigil-validation)).

## 1. The table, the run, and victory

- Four seats form two partnerships, North-South and East-West, with partners
  opposite.
- A run is 8 rounds. A round is one deal: shop, deal, auction, play, score, and
  income.
- Scores belong to partnerships. Each deal awards points to exactly one side,
  so totals only grow.
- Gold, sigils, cards, and engravings belong to individual players.
- After deal 8, the side with the higher total wins. Equal totals are a draw.
- In single-player you sit South, partnered with an AI North, against AI East
  and West. North never shops, so you are your side's only buyer. Multiplayer
  is covered in [§13](#13-multiplayer).

## 2. The lifecycle of a round

1. **Shop.** Every shopping player shops at once
   ([§8](#8-shops-and-economy)). The first shop opens before deal 1 with each
   player's starting gold.
2. **Deal.** Hands are built from owned cards and the shuffled remainder
   ([§7](#7-cards)).
3. **Auction.** The dealer calls first and calls proceed clockwise. If all four
   players pass, the same dealer redeals with the same vulnerability and no
   shop.
4. **Play.** Declarer's left-hand opponent leads, dummy is exposed, and
   declarer plays both hands for 13 tricks.
5. **Score.** One side scores the deal ([§4](#4-scoring)).
6. **Income.** Each shopping player collects interest, then base and trick
   income.
7. **Next round.** After deal 8 the run ends without a shop.

The eight deals use duplicate boards 1–8, so each side is vulnerable on four
deals and each seat deals twice:

| Deal | Dealer | Vulnerable |
| --- | --- | --- |
| 1 | North | None |
| 2 | East | North-South |
| 3 | South | East-West |
| 4 | West | Both |
| 5 | North | North-South |
| 6 | East | East-West |
| 7 | South | Both |
| 8 | West | None |

## 3. Bridge rules and modified cards

### Auction

- The auction is standard bridge: bids from 1♣ to 7NT, double, redouble, and
  pass. Every legal call is always available; SAYC is guidance
  ([§11](#11-bidding-ui-and-learning-aids)).
- Declarer is the first player of the winning side to have named the final
  strain.

### Who plays declarer

- In single-player you play declarer whenever North-South declares. If North
  is the nominal declarer, you play from North's seat and your hand becomes
  dummy, as in BBO robot play. The opening lead and dummy follow the nominal
  declarer.
- With human partners ([§13](#13-multiplayer)), the nominal declarer plays and
  the human dummy watches, as in real bridge.

### Following suit and winning tricks

- Players must follow suit if able; a void player may trump or discard.
- The highest trump wins; otherwise the highest card of the led suit wins.
- **Ties:** suit conversions and rank increases can create two cards of the
  same effective suit and rank. Between equal cards, the **first played wins**:
  a card must beat another card, not match it.
- **Effective cards:** a modified card counts at its new suit and rank
  everywhere, including following suit, winning tricks, HCP in the bidding
  guidance, and sigil conditions. A king raised to an ace is an ace for every
  purpose. Ranks cap at ace.

### Wild cards

The Wild engraving makes a card count as every suit:

- It is always a legal play and counts as the suit led. When its holder leads
  it, they name its suit.
- It never forces its holder to follow suit. A player holding a wild can still
  be void, trump, or discard.
- It wins only as the led suit, never as a trump unless trumps were led. In NT
  it acts as a stopper in every suit.
- For sigils, it counts as every suit whenever a sigil checks a card's suit or
  counts cards of a suit: a trick won with a wild is a hearts trick and a
  spades trick. Voids are judged on non-wild cards only.

Example: you hold no hearts, ♠7 2, and a wild Q. When hearts are led you may
play the wild as the Q♥, trump with the ♠7, or discard.

## 4. Scoring

### Base values

These are standard duplicate values:

| Item | Not vulnerable | Vulnerable |
| --- | --- | --- |
| Trick in ♣ or ♦ | 20 | 20 |
| Trick in ♥ or ♠ | 30 | 30 |
| Trick in NT | 30, plus a flat +10 per NT contract | same |
| Part-score bonus (below game) | 50 | 50 |
| Game bonus (3NT, 4♥, 4♠, 5♣, 5♦, or higher) | 300 | 500 |
| Small slam bonus (level 6) | 500 | 750 |
| Grand slam bonus (level 7) | 1,000 | 1,500 |

NT's usual "40 for the first trick" becomes the flat +10 because tricks carry
individual values (below).

Undertricks, paid to the defenders:

| Undertricks | NV | NV doubled | NV redoubled | Vul | Vul doubled | Vul redoubled |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 50 | 100 | 200 | 100 | 200 | 400 |
| 2 | 100 | 300 | 600 | 200 | 500 | 1,000 |
| 3 | 150 | 500 | 1,000 | 300 | 800 | 1,600 |
| Each further | +50 | +300 | +600 | +100 | +300 | +600 |

Rubber-bridge honors are not scored.

### Trick values and the book

Every trick the declaring side wins has a **trick value**:

```
trick value = (base + per-trick bonuses) × (1 + trick multipliers on that trick)
```

The declaring side's **six lowest-value tricks are book** and score 0. Every
other trick it won is a scoring trick, whether it counts toward the contract
or is an overtrick. With no sigils every trick is worth the same, so this
reproduces standard bridge scoring exactly. The round summary greys out the
book tricks, which teaches the concept.

Game and slam status depends only on the contract's level, never on trick
values. Sigil bonuses never turn a part-score into a game.

### Made contracts

```
deal score = ( sum of scoring trick values
             + part-score, game, or slam bonus
             + flat contract bonuses )
             × (1 + sum of contract multipliers)
             × doubling factor
```

- **Trick multipliers** add up within each trick and **contract multipliers**
  add up within the contract: two "+2× contract multiplier" sigils make ×5.
  Because the layers multiply each other, growth across categories is still
  multiplicative, and the order of sigils never matters.
- A few legendaries compound, such as "double your side's contract
  multiplier". They apply after the sum.
- **Doubling factor:** 1 undoubled, ×2 doubled, ×4 redoubled. It applies on top
  of everything, so doubling a sigil-heavy contract is a large, readable gamble.
  Standard bridge's insult bonus, doubled overtrick values, and doubled-into-game
  rule are dropped.

### Failed contracts

The declaring side scores nothing: its trick values, flat bonuses, and trick
multipliers are lost. The defenders score:

```
defense score = ( undertrick penalty × (1 + declaring side's contract multipliers)
                + defense bonuses )
                × (1 + defense multipliers)
```

- The undertrick penalty comes from the table above, including doubles and
  redoubles.
- **Multipliers cut both ways.** The declaring side's contract multipliers
  count whenever their condition is met, except conditions that require making
  the contract. "Your hearts contracts have +2× contract multiplier" counts on
  a failed hearts contract; "+2× if you make exactly your contract" does not.
- Overreaching therefore costs a strong build more than a weak one, defenders
  profit from greed, and sacrificing is cheap for weak builds and expensive for
  strong ones.
- The slam decision stays near bridge's usual 50–55% threshold, because the safe
  game is multiplied by the same amount.
- Defense bonuses and multipliers come only from a few one-off defense sigils
  ([Appendix C](#appendix-c-one-offs-not-archetypes)).

### Worked examples

| Situation | Calculation | Score |
| --- | --- | --- |
| 4♥ NV, 10 tricks, no sigils | 4 × 30 + 300 | 420 to N-S |
| 3NT NV, 10 tricks, no sigils | 4 × 30 + 10 + 300 | 430 to N-S |
| 4♥ NV, 11 tricks; "+10 per trick in hearts contracts" and "tricks won with an ace have +2× trick multiplier"; one ace trick | Ace trick (30 + 10) × 3 = 120; ten others at 40, six of them book; 120 + 4 × 40 + 300 | 580 to N-S |
| Same, contract multiplier ×3 | 580 × 3 | 1,740 to N-S |
| Same, doubled; redoubled | 580 × 3 × 2; 580 × 3 × 4 | 3,480; 6,960 to N-S |
| 6♥ vul, down 1; N-S multiplier ×4 | 100 × 4; doubled 200 × 4 | 400; 800 to E-W |
| E-W sacrifice in 7♠X NV, down 4; E-W multiplier ×2 | 800 × 2 | 1,600 to N-S |
| E-W double 4♠ NV, down 2; N-S multiplier ×2; E-W hold "+50 per undertrick you collect" and "penalties from contracts you doubled have +1× multiplier" | (300 × 2 + 2 × 50) × 2 | 1,400 to E-W |
| Late run: 6♥ vul, 12 tricks; +30 per trick from sigils; +300 flat; contract multiplier ×4 | (6 × 60 + 500 + 750 + 300) × 4 | 7,640 to N-S |

### Par curve

Par is the typical score when a reasonably built partnership declares,
averaged over makes, failures, and contracts outside its strain. It grows
about ×1.5 per deal and ×20 over the run:

| Deal | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Par | 400 | 600 | 950 | 1,450 | 2,200 | 3,400 | 5,200 | 8,000 |

Par assumes single-player, with 7 sigils on your side. A side with two
human buyers holds up to 10 sigils and scores higher; that is accepted.

Deal 1 is a plain game, and deal 8 is a boosted slam. Sigil budgets
([§10](#10-sigil-pool-skeleton)) are calibrated to this curve, as Rogue Spades
calibrated its pool.

Because only one side declares each deal and builds grow every round,
whoever declares the late deals scores the most. A side declaring deals 2, 4,
6, and 8 at par scores 13,450, against 8,750 for deals 1, 3, 5, and 7. This
is accepted: the auctions on deals 7 and 8 are the climax, the stronger build
usually wins them, and sacrifices and multiplied penalties give the other
side real counterplay.

## 5. Sigils

### Slots and ownership

- Each player holds up to **5 sigils**, or **7** for a human with an AI
  partner ([§8](#ai-partners-do-not-shop)). Buying one needs a free slot, so a
  player with a full row must sell one first. Utility sigils can add slots.
- A player is never offered a sigil they already own.
- A sigil sells for half its price, rounded down to a multiple of 5.
- Sigils never have activated abilities. Selling a sigil may trigger it, as
  with Balatro's Luchador.

### Scope: "you" means your side

- Both partners' sigils apply to any contract their side declares, whichever
  partner is declarer. Up to 10 sigils feed one contract from two shopping
  partners, or 7 from one human with an AI partner.
- "You" and "your" in sigil text mean your partnership. Tricks won with dummy's
  cards count as your tricks.
- "Your hearts contracts" means contracts your side declares in hearts.
- Contract and trick sigils work only while your side declares. On defense,
  only defense sigils and utility sigils work. Defense is mostly damage
  control, as in real bridge: in a head-to-head race, setting the opponents'
  slam is worth as much as scoring one.

### Visibility

- Every player sees every player's sigils, like Balatro jokers on the table.
  Reading the opponents' build is part of the strategy ("they're a hearts
  build, so lead trumps").
- Each player's gold and both sides' scores are public.

### Categories

| Category | Balatro analog | Example (not final) |
| --- | --- | --- |
| Flat contract bonus | Chips | "Your hearts contracts are worth +100." |
| Per-trick bonus | Chips per card | "Tricks in your hearts contracts are worth +10." |
| Trick multiplier | +Mult on specific cards | "Tricks you win with an ace have +2× trick multiplier." |
| Contract multiplier | ×Mult | "Your hearts contracts have +2× contract multiplier." |
| Utility | Economy and rule jokers | "The first reroll in each shop is free." |

Utility comes in three families. Information effects, such as peeking at
another hand, are excluded because they undercut the bidding inference that
teaches bridge.

- **Economy and shop:** bonus gold, a higher interest cap, sell value that
  grows, cheaper or free rerolls, discounts, extra offers, and better rarity
  odds.
- **Deal and card control:** choosing which owned cards are dealt when you own
  more than 13, guaranteeing a kind of card, cheaper cards, better card offers,
  and engraving tools.
- **Rule benders:** analogs of Balatro's Smeared Joker and Pareidolia, such as
  "hearts and diamonds count as one suit for your sigils", "your kings count as
  aces for your sigils", "your side's book is five tricks", and +1 sigil slot.

### Rarity and price

| Rarity | Offer odds | Price | Pool size |
| --- | --- | --- | --- |
| Common | 69% | 50 | 60 |
| Uncommon | 25% | 75 | 60 |
| Rare | 5% | 100 | 20 |
| Legendary | 1% | 150 | 5 |

### Timing

Gold effects resolve as they happen, for example when a card is played. Scoring
effects resolve at scoring. Within the formula, sums make sigil order
irrelevant; compounding legendaries apply last.

## 6. Engravings

Engravings permanently modify a card you own. They are separate from sigils
and take no slot.

| Engraving | Effect | Price |
| --- | --- | --- |
| Trick value | +20 to the trick it is played to, if your side wins it* | 30 |
| Contract value | +50 contract value when played* | 40 |
| Wild | Counts as every suit ([wild card rules](#wild-cards)) | 60 |
| Raise | +1 rank, up to ace | 30 |
| Gold | +20 gold when played | 30 |
| While held | While it is in hand, each trick your side wins is worth +10* | 40 |
| Convert ♣ / ♦ / ♥ / ♠ | Becomes that suit; four separate offers | 30 |

\* Scores only when your side declares, because cards are often played on
defense.

Rules:

- An engraving is applied as soon as it is bought, to an owned card of the
  buyer's choice. It cannot be stored.
- Each card holds at most one engraving.
- An engraving cannot be bought without an eligible owned card. Raise needs a
  card below ace, Convert needs a card of another suit, and every engraving needs
  an unengraved card.
- An engraving works for **whoever holds the card**, including an opponent who
  is dealt your extra card ([§7](#7-cards)).
- An engraving is visible whenever its card is: in its holder's hand, in dummy,
  or once played.
- Selling a card removes its engraving without refunding it.
- Engraving offers are drawn uniformly from ten offer types: the six
  single-version engravings and the four Converts.

## 7. Cards

- Players start the run owning no cards.
- Each card exists once. Card offers are drawn uniformly from cards that nobody
  owns and nobody else is currently being offered, so two players can never be
  offered or own the same card.
- Prices follow rank:

  | Rank | A | K | Q | J | 10 | 2–9 |
  | --- | --- | --- | --- | --- | --- | --- |
  | Price | 100 | 75 | 50 | 35 | 25 | 15 |

- Owned cards are private, even from your partner. Partners learn about each
  other's hands only through the auction, as in bridge.
- A card sells for half its price, rounded down to a multiple of 5. It returns
  to the unowned pool without its engraving.

### Dealing

1. Each player receives their owned cards. A player who owns more than 13 cards
   receives a random 13 of them.
2. All other cards, both unowned cards and anyone's undealt extras, are shuffled
   and dealt to fill every hand to 13.

Extras keep their engravings, and those engravings work for whoever receives
them. Owning more than 13 cards therefore costs consistency and risks handing
an opponent your engraved ace for a deal.

## 8. Shops and economy

### Shop

- Each shop offers **2 sigils, 2 engravings, and 2 cards**. Utility sigils can
  add offers.
- Purchases are unlimited. A bought offer leaves an empty slot until the next
  reroll.
- A **reroll** refreshes all six offers for 50 gold, plus 10 for each further
  reroll in the same shop. The cost resets at the next shop.
- Offers are random within their rarity, as in Balatro. Builds come together by
  pivoting toward what appears and by using rerolls, card purchases, and suit
  conversions. Cross-strain archetypes and generic point sigils carry builds
  through the gaps.
- Sigils and cards can be sold at any shop for half price.

### Gold

- Each player starts with **100 gold**.
- After each deal, each player first gains **interest**, 10 per 50 gold held up
  to 50, and then gains **100 base income plus 10 per trick their side won**,
  whether declaring or defending.
- Gold from sigils and engravings arrives when its trigger happens.
- A typical round earns about 195 gold, or about 1,450 spendable over the run.
  Score leads do not turn into gold leads; earning more is the job of economy
  sigils and Gold engravings.

### AI partners do not shop

A human with an AI partner is their side's only buyer. In single-player that
is you, and North never shops.

- The AI partner has no gold, sigils, cards, or engravings. It is dealt 13
  cards from the shuffled remainder every deal.
- The human gets the whole side's economy: **200 starting gold**, **200 base
  income plus 20 per trick** their side won, and interest of 10 per 50 gold held
  up to **100**. That is about 390 gold a round, or about 2,900 over the run.
- The human holds up to **7 sigils**, fewer than a two-buyer side's 10.
  Single-player scores are lower than multiplayer scores as a result.
- Shop offers, prices, and reroll costs are unchanged. The extra gold mostly
  buys more rerolls, so the human sees about as many offers as two partners
  would.
- Cards and engravings still go to the human's own hand. Owning more than 13
  cards becomes more common, with the usual cost ([§7](#dealing)).
- Gold effects already count the whole side ("you" means your side), so they
  are not doubled.

## 9. Archetypes

An archetype is a plan you can build a run around. Archetypes come in two
weights:

- **Major archetypes** have sigils at every rarity and can carry a run alone.
  The five strains are majors, and so are four cross-strain archetypes:
  Trumps, Long Suits, Slams, and Ranks.
- **Minor archetypes** have about five sigils each and usually join a major:
  Low Cards, Voids, Rainbow, and Timing.

Only strain sigils care which strain you declare. Every other archetype scores
in any strain where its plan works, so a hearts build that pivots to spades
keeps its Trumps and Ranks sigils. Generic point sigils support every build,
and ideas too narrow for an archetype become one-off sigils
([Appendix C](#appendix-c-one-offs-not-archetypes)).

| Archetype | Weight | Plan in one line | Strain lean |
| --- | --- | --- | --- |
| Strains | Major ×5 | Collect one suit, or its top honors, and bid it | — |
| Trumps | Major | Hold many trumps, then draw them or ruff with them | ♠ |
| Long Suits | Major | Own one very long suit and run it | ♣ |
| Slams | Major | Bid six or seven whenever it is close | Any |
| Ranks | Major | Collect one rank, usually aces, and win with it | ♦ |
| Low Cards | Minor | Win tricks with 2s through 10s | ♣ |
| Voids | Minor | Start with a void or make one fast | ♠ |
| Rainbow | Minor | Win tricks with all four suits | NT |
| Timing | Minor | Win tricks in a row, early, or late | ♥ |

> **Every sigil in this document is an example, not a final design.** Names,
> wording, and numbers are placeholders. Each one must pass simulation
> ([§15](#15-sigil-validation)) before it enters the pool, and archetypes whose
> sigils keep failing are cut.

Example sigils are tagged by rarity (C, U, R, L) and category.

### Ways to score: bid, hold, lead, win

Each archetype rewards one idea through several triggers, so its sigils stack
instead of competing. Hold sigils pay for what you bought; Lead and Win sigils
pay for how you play it.

| Trigger | What it checks | Example |
| --- | --- | --- |
| Bid | The final contract's strain or level | Your slams have +2× contract multiplier. |
| Hold | Declarer's and dummy's hands as dealt | Your contracts are worth +40 for each ace your side holds. |
| Lead | Cards your side leads to tricks | Tricks you lead an ace to have +2× trick multiplier. |
| Win | The card that wins a trick for your side | Tricks you win with an ace are worth +20. |
| Ruff | Winning a trick by trumping a side suit | Tricks you win by trumping have +2× trick multiplier. |

- Hold sigils count both of your side's hands, because in single-player your
  hand may be dummy ([§3](#who-plays-declarer)).
- Per-trick bonuses and trick multipliers reach only tricks your side wins, so
  a per-trick Lead sigil needs the lead to hold. A flat Lead sigil pays either
  way.

### Cycles

A **cycle** is a set of sigils that are identical apart from one symbol, a
suit or a strain. Learn one member and you know them all, and the beginner
plan works the same way in every strain. There are four cycles, one for each
beginner plan, and they take 18 sigils, or about a fifth of the point sigils.

| Cycle | Rarity, category | Members | Plan | Text, shown for ♥ |
| --- | --- | --- | --- | --- |
| Strain tricks | C, Per-trick | ♣ ♦ ♥ ♠ NT | Bid hearts | Tricks in your ♥ contracts are worth +10. |
| Suit holding | C, Flat | ♣ ♦ ♥ ♠ | Get lots of hearts | Your contracts are worth +10 for each ♥ your side holds. |
| Strain multiplier | U, Contract × | ♣ ♦ ♥ ♠ NT | Bid hearts | Your ♥ contracts have +2× contract multiplier. |
| Top honors | U, Contract × | ♣ ♦ ♥ ♠ | Get the A, K, and Q of hearts | +1× contract multiplier if your side holds the A, K, and Q of ♥. |

- Cycles stay identical where bridge does not: minor-suit tricks are worth 20
  and minor-suit game takes 11 tricks. This is accepted for readability, and
  the gap shrinks as sigil bonuses outgrow base values
  ([§14](#14-risks-and-tuning-levers)).
- NT has no suit, so it has fewer cycle members. Ranks and Rainbow fill the
  gap.
- Suit holding and Top honors score in any contract, so collecting hearts
  still pays when the auction lands in NT.

### Strains

**Plan.** Pick a suit or NT. Buy its cards, Convert other cards into it, take
its cycle sigils, and bid it every deal you can, up to slam. Collecting only
the A, K, and Q of a suit is the same plan with three cards instead of ten.

That plan is identical in every strain, and no strain has sigils of its own
beyond the cycles. Strains feel different through bridge itself, which makes
some cross-strain archetypes a natural fit:

| Strain | Feel | Natural lean |
| --- | --- | --- |
| ♣ | Cheapest to build with 15-gold low cards. Outbid by every strain, so preempt high and run long clubs. | Long Suits, Low Cards |
| ♦ | Like clubs, game takes 11 tricks; side-suit aces stop the opening leads that beat 5♦ and 6♦. | Ranks |
| ♥ | Lose your losers early, draw trumps, then win everything. | Timing |
| ♠ | Outbids every strain at its level. Convert short suits into spades and crossruff. | Trumps, Voids |
| NT | Game in nine tricks, but every suit needs a stopper. Trumps sigils score nothing here. | Rainbow, Ranks |

- **Buys:** the strain's suit, or aces and kings in every suit for NT.
- **Engraves:** Convert into the suit; Raise toward its top honors; Wild for an
  NT stopper.
- **Bids:** the strain at every chance, preempting early in clubs and spades.
- **Threat:** opponents read your strain sigils and outbid or sacrifice. A
  cross-strain archetype keeps scoring when you can't declare your strain.
- **Pairs with:** any cross-strain archetype, most naturally its lean.

### Trumps

**Plan.** Own the trumps in whatever suit you declare. Trumps pays two ways:
drawing trumps by leading them, and ruffing side-suit losers. Most hands suit
one line or the other, so a Trumps build leans one way.

| Trigger | Example |
| --- | --- |
| Hold | [C, Flat] Your suit contracts are worth +15 for each trump your side holds. |
| Lead | [C, Per-trick] Tricks you lead a trump to are worth +15. |
| Win | [C, Per-trick] Tricks you win with a trump are worth +10. |
| Ruff | [C, Trick ×] Tricks you win by trumping have +2× trick multiplier. |
| Hold | [U, Flat] Your suit contracts are worth +50 for each trump your side holds beyond eight. |
| Win | [U, Contract ×] +2× contract multiplier if your side wins five or more tricks with trumps. |
| Ruff | [U, Contract ×] +1× contract multiplier if your side wins three or more tricks by trumping. |
| Ruff | [R, Contract ×] +1× contract multiplier for each trick you win by trumping, up to +4×. |
| Hold | [R, Contract ×] +1× contract multiplier for each trump your side holds beyond eight. |

- **Buys:** cards of one suit, and high trumps for overruffs.
- **Bids:** your long suit as trumps, never NT.
- **Threat:** opponents lead trumps to cut down ruffs, or overruff. Raised top
  trumps answer both.
- **Pairs with:** Voids, Long Suits, Low Cards, ♠.

### Long Suits

**Plan.** Own one very long suit, draw out the opponents' cards in it, and win
the remaining tricks with small cards. Your **longest suit** is the suit in
which declarer and dummy together hold the most cards as dealt, and ties count
every tied suit. It can be trumps or, in NT, a side suit you run.

| Trigger | Example |
| --- | --- |
| Hold | [C, Flat] Your contracts are worth +100 if declarer or dummy holds seven or more cards in one suit. |
| Lead | [C, Per-trick] Tricks you lead from your longest suit are worth +10. |
| Win | [C, Per-trick] Tricks you win with a card of your longest suit are worth +10. |
| Win | [C, Contract ×] +1× contract multiplier if your side wins the last four tricks with your longest suit. |
| Hold | [U, Flat] Your contracts are worth +40 for each card beyond six in your longest suit. |
| Win | [U, Per-trick] Each trick you win with your longest suit is worth +5 more than the one before. |
| Win | [U, Contract ×] +1× contract multiplier if your side wins eight or more tricks with cards of one suit. |
| Hold | [R, Contract ×] +1× contract multiplier for each card beyond six that declarer or dummy holds in one suit. |
| Hold | [R, Per-trick] Tricks you win with your longest suit are worth +10 for each card your side holds in it beyond seven. |

- **Buys:** cheap low cards of one suit. This build often owns more than 13
  cards.
- **Engraves:** Convert into the suit; Raise on its low cards.
- **Bids:** preempts and jumps on length; 3NT on a long running minor.
- **Plays:** count the opponents' cards so the 9 and 8 become winners.
- **Threat:** a bad split or a missing entry strands the suit.
- **Pairs with:** Trumps, Low Cards, ♣, NT.

### Slams

**Plan.** Bid six or seven whenever it is close. Slams pays for bidding slams
as well as making them, so its risk lives in the auction.

| Trigger | Example |
| --- | --- |
| Bid | [C, Utility] Gain 40 gold whenever your side bids a slam. |
| Make | [C, Flat] Your slams are worth +300. |
| Make | [C, Per-trick] Tricks in your slams are worth +20. |
| Make | [C, Flat] Your grand slams are worth +1,000. |
| Bid | [U, Contract ×] Your slams have +2× contract multiplier. |
| Make | [U, Flat] Your slams are worth +100 for each slam your side has made this run. |
| Bid | [U, Flat] Your slams are worth +500 if your side holds 30 or fewer HCP. |
| Bid | [R, Contract ×] +1× contract multiplier for each slam your side has bid this run, made or not. |
| Bid | [R, Contract ×] Your grand slams have +4× contract multiplier. |

- **Buys:** aces and kings for control, in any strain.
- **Bids:** "Aces? (4NT)", "Aces? (4♣)", "Kings? (5NT)", and jumps.
- **Threat:** a failed slam pays the opponents your whole contract multiplier
  ([§4](#failed-contracts)), and opponents sacrifice against slams they can't
  beat.
- **Pairs with:** Strain multipliers, Ranks, Timing.

### Ranks

**Plan.** Collect one rank and win tricks with it. Most Ranks sigils are about
aces, which win in every strain and cost the most; a few pay for cheaper kings,
queens, and jacks.

| Trigger | Example |
| --- | --- |
| Win | [C, Per-trick] Tricks you win with an ace are worth +20. |
| Win | [C, Per-trick] Tricks you win with a king are worth +20. |
| Hold | [C, Flat] Your contracts are worth +40 for each ace your side holds. |
| Hold | [C, Flat] Your contracts are worth +15 for each queen and jack your side holds. |
| Lead | [U, Trick ×] Tricks you lead an ace to have +2× trick multiplier. |
| Hold | [U, Contract ×] +1× contract multiplier if your side holds four or more aces. |
| Hold | [U, Utility] Your kings count as aces for your sigils. |
| Win | [R, Contract ×] +1× contract multiplier for each trick you win with an ace beyond two. |
| Hold | [R, Contract ×] +1× contract multiplier for each rank of which your side holds all four cards. |

- **Buys:** aces, and kings to Raise. Raised kings make five or more aces
  possible.
- **Engraves:** Raise on kings; Wild on an ace, which can be led as the ace of
  any suit.
- **Plays:** cash aces before the opponents can trump them. NT keeps them safe.
- **Threat:** aces cost 100 gold each, and they get trumped once a suit runs
  out.
- **Pairs with:** Slams, Timing, ♦, NT.

### Low Cards

**Plan.** Win tricks with 2s through 10s, the cheapest cards in the shop. Low
cards win through length, by trumping, and after the honors are gone. Raise
cuts both ways: a raised 10 is a jack.

- [C, Per-trick] Win: tricks you win with a 10 or lower are worth +15.
- [C, Flat] Hold: your contracts are worth +100 if your side holds 20 or fewer
  HCP.
- [C, Per-trick] Lead: tricks you win after leading a 10 or lower are worth
  +15.
- [U, Trick ×] Win: tricks you win with a 10 or lower have +1× trick
  multiplier.
- [U, Contract ×] Win: +1× contract multiplier if your side wins four or more
  tricks with a 10 or lower.
- [R, Per-trick] Win: tricks you win with a 2 are worth +200.

**Pairs with:** Long Suits, Trumps, ♣.

### Voids

**Plan.** Start with a void, or make one fast, then ruff or discard. Convert
engravings create voids, and wild cards never break one
([§3](#wild-cards)). Voids sigils score in NT, but a void there is a
liability.

- [C, Flat] Hold: your contracts are worth +75 for each void in declarer's or
  dummy's hand as dealt.
- [C, Contract ×] Early: +1× contract multiplier if declarer or dummy becomes
  void in a suit by the end of trick 3.
- [C, Per-trick] Early: tricks you win while declarer or dummy is void in a
  suit are worth +10.
- [U, Contract ×] Hold: +2× contract multiplier if declarer or dummy is dealt a
  void.
- [R, Contract ×] Early: +1× contract multiplier for each void declarer and
  dummy have after trick 4, up to +3×.

**Pairs with:** Trumps, ♠.

### Rainbow

**Plan.** Win tricks with cards of all four suits. NT does this naturally; in a
suit contract it means cashing side-suit winners as well as trumps. A trick
won with a wild card counts as every suit ([§3](#wild-cards)), so one wild
completes the rainbow and Wild is the build's key engraving.

- [C, Contract ×] Win: +1× contract multiplier if your side wins tricks with
  cards of all four suits.
- [C, Per-trick] Win: the first trick you win with each suit is worth +30.
- [C, Flat] Hold: your contracts are worth +100 if your side holds an ace or
  king in every suit.
- [U, Contract ×] Hold: +1× contract multiplier if neither declarer nor dummy
  is dealt a void.
- [R, Trick ×] Lead: tricks you lead in a suit your side has not yet led this
  deal have +2× trick multiplier.

**Pairs with:** NT, Ranks. It pulls against Voids.

### Timing

**Plan.** Win tricks in a row, early, or late. Each hand becomes a sequencing
puzzle: take your losses at the right moment, then keep the lead. Early and
late sigils pull opposite ways, since one wants winners cashed at once and the
other wants losers given up first. The defenders lead to trick 1, so winning
early means winning the opening lead.

- [C, Per-trick] Sequence: each trick your side wins is worth +10 for each
  trick it won in a row before it.
- [C, Trick ×] Late: the last trick has +2× trick multiplier if your side wins
  it.
- [C, Per-trick] Early: each of the first four tricks is worth +20 if your side
  wins it.
- [U, Trick ×] Sequence: each trick your side wins in a row has +1× trick
  multiplier more than the one before.
- [U, Flat] Early: your contracts are worth +150 if your side wins the first
  four tricks.
- [R, Contract ×] Late: +3× contract multiplier if your side wins every trick
  after the first one it loses.

**Pairs with:** ♥, Slams, Ranks.

### Generic

Generic point sigils fit any build and carry runs while a plan comes together.

- [C, Per-trick] Tricks in your contracts are worth +5.
- [C, Flat] Your contracts are worth +50.
- [U, Contract ×] Your vulnerable contracts have +1× contract multiplier.
- [L, Contract ×] Double your side's contract multiplier.
- [L, Utility] +1 sigil slot.
- [L, Utility] Your side's book is five tricks.

Utility examples:

- **Economy and shop:** the first reroll in each shop is free; your shops offer a
  third sigil; sigils cost you 10 less; your interest cap rises by 30; this
  sigil's sell value rises by 10 after each deal.
- **Deal and card control:** when you own more than 13 cards, choose which 13 are
  dealt to you; cards cost you 5 less; one of your cards may hold a second
  engraving; your shops offer a third card.
- **Rule benders:** hearts and diamonds count as one suit for your sigils; your
  jacks count as low cards; penalties against your side are halved.

## 10. Sigil pool skeleton

The first pool has 145 sigils. About 70% of commons, 50% of uncommons, 75% of
rares, and 60% of legendaries score points. Commons are mostly simple
build-around scoring, including two of the four cycles; uncommons are the
main home of utility and of contract multipliers; rares return to exciting
scoring; and legendaries are the splashiest effects. Within trick-level
scoring, per-trick points dominate and trick multipliers are rare.

### By rarity and category

| Rarity | Flat | Per-trick | Trick × | Contract × | Utility | Total |
| --- | --- | --- | --- | --- | --- | --- |
| Common | 15 | 22 | 2 | 3 | 18 | 60 |
| Uncommon | 6 | 3 | 3 | 18 | 30 | 60 |
| Rare | 1 | 3 | 2 | 9 | 5 | 20 |
| Legendary | — | — | — | 3 | 2 | 5 |
| **Total** | **22** | **28** | **7** | **33** | **55** | **145** |

The seven trick multipliers are Trumps' ruffs (C), Timing's last trick (C),
Low Cards' wins (U), Timing's climbing run (U), Ranks' ace leads (U),
Rainbow's new-suit leads (R), and one generic rare.

### Point sigils by archetype

| Archetype | Common | Uncommon | Rare | Total |
| --- | --- | --- | --- | --- |
| Strain cycles | 9 | 9 | — | 18 |
| Trumps | 4 | 3 | 2 | 9 |
| Long Suits | 4 | 3 | 2 | 9 |
| Slams | 3 | 3 | 2 | 8 |
| Ranks | 4 | 2 | 2 | 8 |
| Low Cards | 3 | 2 | 1 | 6 |
| Voids | 3 | 1 | 1 | 5 |
| Rainbow | 3 | 1 | 1 | 5 |
| Timing | 3 | 2 | 1 | 6 |
| Generic and one-offs | 6 | 4 | 3 | 13 |
| **Total** | **42** | **30** | **15** | **87, plus 3 legendary = 90** |

Each suit therefore has four point sigils of its own (two common, two
uncommon) and NT has two. Cycles are 18 of the 90 point sigils.

### Utility by family

| Family | Common | Uncommon | Rare | Legendary | Total |
| --- | --- | --- | --- | --- | --- |
| Economy and shop | 8 | 10 | 1 | — | 19 |
| Deal and card control | 6 | 10 | 2 | — | 18 |
| Rule benders | 4 | 10 | 2 | 2 | 18 |
| **Total** | **18** | **30** | **5** | **2** | **55** |

About 8 utility sigils are aimed at an archetype, roughly one per
cross-strain archetype, such as Ranks' kings-as-aces or Slams' gold for
bidding a slam. The rest are generic.

### Budgets

These are starting sizes on a made contract of the right kind, before the
contract multiplier:

| Rarity | Flat | Per-trick | Trick × | Contract × |
| --- | --- | --- | --- | --- |
| Common | +100 in one strain, +50 broad | +10 in one strain, +5 broad | +2× on a narrow kind of trick | +1× on a narrow condition met in about a third of made contracts |
| Uncommon | +200 in one strain, +100 broad | +20 in one strain, +10 broad | +1× that climbs | +2× in one strain, +1× broad |
| Rare | +400, or scaling | +30, or scaling | scaling | +4× on grand slams, +2× broad, or scaling |
| Legendary | — | — | — | ×2 compounding |

Checks against par:

- **Deal 4 (par 1,450):** a hearts partnership with the hearts Strain tricks,
  Suit holding, and Strain multiplier makes 4♥ vul with 11 tricks and nine
  hearts: 5 × 40 + 500 + 90 = 790, × 3 = 2,370 in its strain. Off-strain contracts and
  failures pull the average toward par.
- **Deal 8 (par 8,000):** six point sigils (two strain commons, a generic common,
  a Strain multiplier, a generic +1×, and Slams' +2×), plus engravings,
  make 6♥ vul with 12 tricks: about 6 × 60 + 1,500 = 1,860, × 4 to × 6 =
  7,400–11,200.

### How often a player sees support

With one reroll per shop, a player sees about 30 sigil offers per run: about 21
commons, 7.5 uncommons, 1.5 rares, and one legendary every three or four runs.
That is about 1 sigil aimed at a chosen suit per run, plus about 2 from each
cross-strain major, which score in any strain. In single-player your double
gold buys about three rerolls per shop, so you see about 60 offers and twice
those counts. Cross-strain archetypes are what make this enough: a build that
misses its strain's sigils still scores through Trumps, Ranks, or Long Suits.
It is still a risk to watch ([§14](#14-risks-and-tuning-levers)).

## 11. Bidding UI and learning aids

The AI bids exactly the system the UI describes, so labels never lie. The
reference is the ACBL SAYC System Booklet: every conventional call in it gets a
label, and every natural call gets a hover card.

- **Labels by meaning.** Conventional calls show what they ask or show, not the
  convention's name: "Majors? (2♣)" rather than "Stayman". The name appears in
  the hover card.
- **Labels depend on context.** The same call changes label with the auction:
  2♣ reads "Strong (2♣)" as an opening bid and "Majors? (2♣)" over 1NT.
- **Hover cards on every call** give the meaning in plain words, the convention
  name if any, the HCP and shape it promises, and whether it is forcing. For
  example, "1♥: 5+ hearts, 12–21 HCP."
- **Subtle suggestion.** The hover card of the call SAYC recommends for your
  hand carries a small "textbook bid" marker. The button itself looks like
  every other button.
- **Seat summaries.** Each seat has a tooltip listing what its calls revealed,
  updated through the auction and available during play. Example: "North:
  15–17 HCP, balanced (1NT). No 4-card major (2♦ over Majors?)."
- **Hand facts.** Hovering your hand shows HCP and suit lengths, counted with
  effective cards.

Representative labels:

| Label | Meaning |
| --- | --- |
| Strong (2♣) | Opening; 22+ HCP or 9+ tricks, artificial and forcing |
| Waiting (2♦) | Reply to Strong 2♣ |
| Weak (2♥) | Opening; 6 hearts, 5–11 HCP |
| Feature? (2NT) | Asks a weak-two opener for an outside ace or king |
| Majors? (2♣) | Stayman over 1NT |
| Request Hearts (2♦) | Jacoby transfer over 1NT |
| Request Spades (2♥) | Jacoby transfer over 1NT |
| Game raise (2NT) | Jacoby 2NT over 1♥ or 1♠ |
| Aces? (4NT) | Blackwood; Kings? (5NT) follows |
| Aces? (4♣) | Gerber over NT openings |
| Slam invite (4NT) | Quantitative raise over NT |
| Takeout (X), Negative (X), Penalty (X) | The three kinds of double |
| Both majors (2♦) | Michaels cue bid over 1♦ |
| Minors (2NT) | Unusual 2NT |

## 12. AI

AI seats follow the same rules as humans and see exactly what a human in their
seat would see. They never see another player's owned cards.

### Bidding

- Every AI call means what its SAYC label says, so the hover cards and seat
  summaries stay trustworthy.
- Within those meanings, the AI uses sigil-aware judgment. It values its hand
  with effective cards, prefers its side's build strain when SAYC offers a
  choice,
  pushes toward game and slam when its side's multipliers pay, and decides
  doubles and sacrifices by expected score.

### Card play

- AI seats play competently as declarer and defender, for example by sampling
  deals consistent with the auction and evaluating them double-dummy. Rogue
  Spades' sampling AI is a starting point.
- Defenders use standard SAYC opening leads and signals, so you can read them.
- In single-player the AI never declares for North-South, because you play
  declarer for your side.

### Shopping

- **Your partner never shops** ([§8](#ai-partners-do-not-shop)). It still bids
  and plays toward your build, which it reads from your public sigils.
- **Each opposing pair settles on one strain** and one cross-strain archetype
  from its early offers and builds them together.
- Opposing AI seats buy, reroll, and sell under the same rules and prices as a
  human with a human partner.

## 13. Multiplayer

The first prototype is single-player only. Because every rule is symmetric
apart from AI partners, multiplayer needs only these additions:

- **Two humans** play as partners against two AI seats. **Four humans** play
  two against two. Human partners each shop with the normal economy and 5
  slots, so a two-human side holds 10 sigils and outscores the single-player
  par curve. Only a human with an AI partner gets the doubled economy and 7
  slots.
- With a human partner, the nominal declarer plays the hand and the human dummy
  watches.
- Shops run simultaneously. Card offers never overlap, so purchases never
  conflict.

## 14. Risks and tuning levers

| Risk | Lever |
| --- | --- |
| Late deals decide the game (accepted) | Growth rate through multiplier budgets |
| AI sacrifices against every big contract, so builds rarely play their slams | Penalty scaling, and the AI's sacrifice threshold |
| Strain builds rarely come together with random offers | More strain commons, a third sigil offer, cheaper rerolls; affinity weighting held in reserve |
| ×2 and ×4 doubling swings decide too many games | Doubling as +1× and +3× instead |
| Identical cycles leave minor strains behind, since their tricks score 20 and game needs 11 tricks | A minor-only +5 per trick in the Strain tricks cycle |
| About one strain sigil per run is too thin for a pure strain build | A fifth cycle, a third sigil offer, cheaper rerolls |
| One wild card completes Rainbow | Wild counts as one named suit for Rainbow |
| Your 7 slots fall behind the 10 held by the two shopping AI opponents | Slot count for the doubled-economy seat; opposing AI slot caps as a difficulty setting |
| Convert is 4 of 10 engraving offer types, 40% of engraving offers | Offer weights |
| Full SAYC bidding plus competent card play is the largest implementation cost | Start with SAYC core bidding and sampling-based play, then widen |

## 15. Sigil validation

Every sigil in this document is an example. No sigil enters the pool on design
intuition alone: each needs empirical evidence from simulated runs.

### The bar

A point sigil ships only if simulation shows that a player who picks it can:

1. **Build around it.** Assemble the cards, engravings, and supporting sigils
   it needs from normal shop offers, reliably rather than on lucky runs.
2. **Trigger it.** Score with it on a steady share of the deals their side
   declares.
3. **Win with it.** Win the run with some real probability, with the sigil
   contributing measurably to that win.

Put plainly: if you pick this sigil and play toward it, there is a measurable
chance that you win because of it.

### How it is measured

- **Harness.** Headless full runs with AI in all four seats, using the same
  bidding, play, and shopping AI as the game ([§12](#12-ai)). The shopping AI
  needs a build-around policy for each archetype, which opposing pairs use too.
- **Forced-pick trials.** A candidate is forced into one side's sigils at a
  fixed shop, such as before deal 1, 3, or 5, and that side builds around it.
  A control arm forces a plain baseline of the same rarity instead, such as
  "Tricks in your contracts are worth +5", on the same seeds.
- **Metrics:**
  - **Trigger rate:** the share of the side's declared deals on which the
    sigil scores, by deal.
  - **Contribution:** the sigil's share of the side's run score, found by
    rescoring each deal without it.
  - **Decisive wins:** the share of the side's wins that rescoring without the
    sigil turns into a loss or draw.
  - **Win-rate lift:** forced-pick win rate minus control win rate.
- **Thresholds** are tuning levers set once the harness runs. Placeholder
  starting points: a trigger rate of at least a third of declared deals from
  deal 4 on, decisive in at least 5% of wins, a lift of at least zero, and a
  ceiling on lift that flags overpowered sigils.

Rescoring holds the auction and play fixed, so it misses how a sigil changes
decisions. The forced-pick win-rate lift covers that.

### Utility sigils

Where proof is prohibitively hard, utility sigils may be judged heuristically.
Convert the effect into gold or offers (a free reroll is worth 50 gold),
compare that to the price, and check in the harness that runs holding it do
not lose more often than runs without it. Rule benders that change what scores,
such as "your kings count as aces for your sigils", are point sigils for this
purpose and need the full bar.

### Results

Each shipped sigil records its trigger rate, decisive-win share, and lift in
the sigil list, so later balance changes are compared against numbers. A sigil
that fails is retuned or cut, and an archetype whose sigils keep failing drops
to one-offs ([Appendix C](#appendix-c-one-offs-not-archetypes)).

## Appendix A: Decision log

| # | Question | Decision |
| --- | --- | --- |
| 1 | Base bridge scoring | Duplicate scoring; vulnerability from boards 1–8 |
| 2 | Whose sigils apply | Partnership: both partners' sigils apply; you play declarer for your side |
| 3 | Which tricks score | Trick values per trick; the six lowest are book |
| 4 | How multipliers stack | Sum within each layer; layers multiply; rare compounding legendaries |
| 5 | Doubling | ×2 or ×4 on the final made score |
| 6 | Failed contracts | Undertricks × the declaring side's contract multiplier |
| 7 | Sigils on defense | Only dedicated defense sigils and utility |
| 8 | Score growth | Moderate, about ×20 over the run |
| 9 | Visibility | Sigils public; owned cards private, even from partner |
| 10 | Undealt extra cards | Dealt to others; engravings work for whoever holds them |
| 11 | Tie between equal cards | First played wins |
| 12 | Wild cards | Optional follower; never trumps; every suit for sigils |
| 13 | Income | 100 base + 10 per trick your side won + interest |
| 14 | Draft focus | Balatro: random offers within rarity |
| 15 | Rarity odds | 69 / 25 / 5 / 1, legendaries in normal offers |
| 16 | Engraving catalogue | Only the seven types in the brief |
| 17 | Engraving specifics | The table in [§6](#6-engravings) |
| 18 | Strain identities | Revised: four identical sigil cycles across the strains and no strain-specific sigils beyond them |
| 19 | Other archetypes | Revised: majors Trumps, Long Suits, Slams, Ranks; minors Low Cards, Voids, Rainbow, Timing |
| 20 | Per-trick bonuses vs trick multipliers | Mostly per-trick bonuses; about eight trick multipliers |
| 21 | Utility families | Economy and shop, deal and card control, rule benders; no information |
| 22 | Convention coverage | The full SAYC booklet, labeled by meaning |
| 23 | Bidding suggestion | A subtle marker in the suggested call's hover card |
| 24 | AI bidding | SAYC meanings with sigil-aware judgment |
| 25 | AI shopping | Your AI partner never shops; opposing pairs share a strain |
| 26 | Multiplayer | Specified here; the first prototype is single-player |
| 27 | GDD scope | Systems, skeleton, and examples; the full sigil list comes later, after simulation |
| 28 | Location | This file |
| 29 | Single-player buying | You are your side's only buyer, with double gold, income, and interest cap and 7 sigil slots; opponents still shop |
| 30 | Rejected archetypes | Gold, doubles, exact contracts, fit, overtricks, and specialized technique are one-off sigils at most ([Appendix C](#appendix-c-one-offs-not-archetypes)) |
| 31 | Sigil slots by mode | 7 for a human with an AI partner, 5 per shopping player otherwise; multiplayer scores run higher |
| 32 | Sigil validation | Every point sigil needs simulation evidence that it can be built around, triggered, and won with; utility may be judged heuristically |

## Appendix B: Calls made without a dedicated question

- A passed-out deal is redealt by the same dealer with no shop.
- Rubber-bridge honors are not scored.
- Equal totals after deal 8 are a draw.
- You sit South.
- Selling a card removes its engraving.
- Engraving offers are uniform over ten offer types, with each Convert suit
  counted as its own type.
- In a defense score, defense bonuses are added after the declaring side's
  multiplier, then defense multipliers apply.
- Wild cards never break a void for sigil purposes.
- Each player's gold is public.
- Your hand's HCP and suit lengths appear on hover rather than permanently on
  screen.

## Appendix C: One-offs, not archetypes

These ideas work as a single sigil that rewards one moment, but are too narrow,
too swingy, or too hard to read to plan a run around. They get at most a few
sigils each from the generic budget ([§10](#10-sigil-pool-skeleton)).

- **Gold builds:** gold comes from economy utility and Gold engravings, and no
  sigil scores from gold held.
- **Doubles and defense:** a few sigils such as "+50 for each undertrick you
  collect" keep defense from being dead time.
- **Exact contracts:** making exactly the contract, no overtricks.
- **Fit:** dummy's tricks and trumps split between hands. Trumps' Hold sigils
  keep the useful part.
- **Overtricks:** tricks beyond the contract.
- **Specialized technique:** squeezes, sacrifices, finesses, ducking, and
  singletons, which are hard for new players to spot and for sigil text to
  define.
