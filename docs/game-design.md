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
| Sigils | 5 slots per shopping player, 10 for you in single-player; both partners' sigils apply to their side's contracts; all sigils are public |
| Engravings | Seven types, applied immediately to an owned card, one per card |
| Cards | Bought cards are dealt to their owner every deal; owned cards are private |
| Shop | 2 sigils, 2 engravings, 2 cards; unlimited purchases; Balatro-style random offers |
| Archetypes | Five strains (♣ ♦ ♥ ♠ NT) and four minors (Aces, Doubles, Exact, Fit) |
| Prototype | Single-player: you sit South with a non-shopping AI partner against two shopping AI opponents |

## Design pillars

1. **Real bridge underneath.** Without sigils, a deal plays and scores as
   duplicate bridge, so the game also teaches SAYC bidding and declarer play.
2. **Direct scoring.** Rogue Spades had many sigils that were fun alone but
   added up to no game plan. Here most sigils say "you get points for doing X"
   in one of four categories, Balatro-style, and builds come from stacking
   them.
3. **Each strain plays differently.** Every strain archetype aims to bid slam
   in its strain every time, but each one gets there through a different bridge
   technique, the way a Flush build in Balatro plays differently from High Card.
4. **Symmetric seats.** Human and AI seats follow identical rules, so one
   design serves single-player and multiplayer. The one exception is an AI
   partner of a human, which never shops; its human partner gets double gold
   and limits instead ([§8](#ai-partners-do-not-shop)).

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
- Defense bonuses and multipliers come only from dedicated defense sigils
  ([Doubles](#doubles)).

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

- Each player holds up to **5 sigils**, or **10** for a human with an AI
  partner ([§8](#ai-partners-do-not-shop)). Buying one needs a free slot, so a
  player with a full row must sell one first. Utility sigils can add slots.
- A player is never offered a sigil they already own.
- A sigil sells for half its price, rounded down to a multiple of 5.
- Sigils never have activated abilities. Selling a sigil may trigger it, as
  with Balatro's Luchador.

### Scope: "you" means your side

- Both partners' sigils apply to any contract their side declares, whichever
  partner is declarer. Up to 10 sigils feed one contract, whether from two
  partners or from one human with an AI partner.
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

| Category | Balatro analog | Example |
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
  conversions. Strain-agnostic point sigils carry builds through the gaps.
- Sigils and cards can be sold at any shop for half price.

### Gold

- Each player starts with **100 gold**.
- After each deal, each player first gains **interest**, 10 per 50 gold held up
  to 50, and then gains **100 base income plus 10 per trick their side won**,
  whether declaring or defending.
- Gold from sigils and engravings arrives when its trigger happens.
- A typical round earns about 195 gold, or about 1,450 spendable over the run.
  Score leads do not turn into gold leads; earning more is the job of economy
  sigils, Gold engravings, and the [Diamonds](#-diamonds-treasury) archetype.

### AI partners do not shop

A human with an AI partner is their side's only buyer. In single-player that
is you, and North never shops.

- The AI partner has no gold, sigils, cards, or engravings. It is dealt 13
  cards from the shuffled remainder every deal.
- The human gets the whole side's economy: **200 starting gold**, **200 base
  income plus 20 per trick** their side won, and interest of 10 per 50 gold held
  up to **100**. That is about 390 gold a round, or about 2,900 over the run.
- The human holds up to **10 sigils**, so the side keeps its 10-sigil ceiling.
- Shop offers, prices, and reroll costs are unchanged. The extra gold mostly
  buys more rerolls, so the human sees about as many offers as two partners
  would.
- Cards and engravings still go to the human's own hand. Owning more than 13
  cards becomes more common, with the usual cost ([§7](#dealing)).
- Gold effects already count the whole side ("you" means your side), so they
  are not doubled.

## 9. Archetypes

There are five strain archetypes and four minor archetypes. Strain archetypes
score only in their strain's contracts. Minor archetypes cut across strains and
combine with a strain build or stand alone. Strain-agnostic generic sigils
support every build.

| Archetype | Plan in one line | Main scoring |
| --- | --- | --- |
| ♣ The Long Suit | Win with club length, not honors | Per-trick bonuses scaling with length |
| ♦ Treasury | Get rich and turn the bank into points | Flat bonuses scaling with gold held |
| ♥ The Run | Win an unbroken string of tricks | Per-trick bonuses and a trick multiplier that climb |
| ♠ Trump Power | Make voids and crossruff | Bonuses on ruffs and voids |
| NT Honors | Own aces and kings in every suit | Flat bonuses per HCP, multipliers per ace |
| Aces | Collect aces and win with them | Trick multiplier on ace tricks |
| Doubles | Profit from the opponents' greed | Penalty bonuses and multipliers on defense |
| Exact | Take exactly the contract | Multipliers for no overtricks |
| Fit | Share a strain with your partner | Multipliers per trump between both hands |

Example sigils are tagged by rarity (C, U, R, L) and category. Names and exact
numbers are placeholders for the sigil-writing pass.

### ♣ Clubs: The Long Suit

**Plan.** Load the hand with clubs, mostly cheap spot cards. Bid clubs to slam
on length, draw trumps, and let small clubs take the last tricks after the
opponents run out.

- **Buys:** club spot cards at 15 gold each. This build owns more cards than
  any other and often more than 13.
- **Engraves:** Convert ♣ on cheap off-suit spot cards; Raise on club spot cards.
- **Bids:** 1♣ openings and 3♣ preempts early; jumps to 5♣ and 6♣ on length.
- **Plays:** draw trumps, run the clubs, and count the opponents' clubs so the 9
  and 8 become winners.
- **Threat:** clubs is the lowest suit, so any other strain outbids it at the
  same level, and game needs 11 tricks. Preempt high and use length to make
  five, six, and seven.
- **Pairs with:** Fit, Exact.

Examples:

- [C, Per-trick] Tricks in your clubs contracts are worth +5 for each club in
  declarer's hand.
- [C, Flat] Your clubs contracts are worth +100 if declarer holds seven or more
  clubs.
- [C, Trick ×] Tricks you win with a club ranked 9 or lower have +2× trick
  multiplier.
- [U, Contract ×] Your clubs contracts have +2× contract multiplier.
- [R, Contract ×] Your clubs contracts have +1× contract multiplier for each club
  beyond six in declarer's hand.

### ♦ Diamonds: Treasury

**Plan.** Diamonds earn gold, and diamond contracts score from the gold you
hold. Every purchase lowers your score, so the build is about when to stop
spending and start banking.

- **Buys:** diamonds and economy sigils early.
- **Engraves:** Gold on cards played every deal; Convert ♦.
- **Bids:** like any minor, toward 5♦ and 6♦.
- **Plays:** ordinary diamond play. Gold engravings pay on defense too.
- **Threat:** building lowers the bank, and diamonds is outbid by the majors. Buy
  early, bank late, and raise the interest cap.
- **Pairs with:** anything, because gold helps every build; Doubles.

Examples:

- [C, Utility] When you play a diamond, gain 10 gold.
- [C, Flat] Your diamonds contracts are worth +1 for every 2 gold you hold.
- [C, Per-trick] Tricks in your diamonds contracts are worth +10, or +20 while
  you hold 200 or more gold.
- [U, Contract ×] Your diamonds contracts have +2× contract multiplier.
- [R, Contract ×] Your diamonds contracts have +1× contract multiplier for every
  150 gold you hold.

### ♥ Hearts: The Run

**Plan.** Hearts payoffs climb with each consecutive trick your side wins, so
every hand is a sequencing puzzle: lose your losers early, draw trumps, then
win everything to the end.

- **Buys:** top hearts for trump control and side-suit aces for entries.
- **Engraves:** Raise on hearts and side kings; While held on a card saved for
  late.
- **Bids:** Jacoby transfers, Jacoby 2NT, and limit raises; from 4♥ to 6♥ through
  "Aces? (4NT)".
- **Plays:** duck early, then run, and never lose the lead once the run starts.
  This is the opposite technique to spades' crossruff.
- **Threat:** one lost trick resets the climb. Hold aces in every side suit.
- **Pairs with:** Aces, Fit.

Examples:

- [C, Per-trick] In your hearts contracts, each trick your side wins is worth +10
  for each trick it won in a row before it.
- [C, Flat] Your hearts contracts are worth +150 if your side wins the last five
  tricks.
- [U, Trick ×] In your hearts contracts, each trick your side wins in a row has
  +1× trick multiplier more than the one before.
- [U, Contract ×] Your hearts contracts have +2× contract multiplier.
- [R, Contract ×] Your hearts contracts have +3× contract multiplier if your side
  wins every trick after the first one it loses.

### ♠ Spades: Trump Power

**Plan.** Convert short side suits into spades, which adds trumps and opens
voids at once, then crossruff instead of drawing trumps.

- **Buys:** spades of any rank.
- **Engraves:** Convert ♠ on short-suit cards; Raise on spades to win trump fights.
- **Bids:** weak 2♠ and preempts early; from 4♠ to 6♠ on shape more than points.
- **Plays:** crossruff, ruff losers in the short hand, and save high trumps for
  overruffs.
- **Threat:** opponents lead trumps to cut down ruffs, or overruff. Long trumps,
  raised top spades, and voids in both hands answer them.
- **Pairs with:** Fit, Doubles.

Examples:

- [C, Per-trick] Tricks you win by trumping in spades contracts are worth +20.
- [C, Flat] Your spades contracts are worth +75 for each void in declarer's or
  dummy's hand.
- [C, Trick ×] Tricks you win by trumping in spades contracts have +2× trick
  multiplier.
- [U, Contract ×] Your spades contracts have +2× contract multiplier.
- [R, Contract ×] Your spades contracts have +1× contract multiplier for each
  trick your side wins by trumping, up to +4×.

### NT: Honors

**Plan.** Aces and kings in every suit make NT safe against any lead, and NT
payoffs scale with your side's high-card points.

- **Buys:** aces and kings in all four suits. This is the most expensive build.
- **Engraves:** Raise to turn queens into kings and kings into aces; Wild for a
  stopper in every suit.
- **Bids:** 1NT and 2NT openings, Stayman and transfers, "Aces? (4♣)", and
  "Slam invite (4NT)"; from 3NT to 6NT and 7NT.
- **Plays:** count winners, cash honors, and establish a long suit with entries.
- **Threat:** a suit with no stopper runs against you. Wild cards and aces in
  every suit answer it.
- **Pairs with:** Aces, Exact.

Examples:

- [C, Flat] Your NT contracts are worth +15 for each HCP in declarer's and
  dummy's hands.
- [C, Per-trick] Tricks you win with an honor in NT contracts are worth +15.
- [U, Contract ×] Your NT contracts have +2× contract multiplier.
- [U, Utility] Your queens count as kings for HCP.
- [R, Contract ×] Your NT contracts have +1× contract multiplier for each suit
  in which your side holds the ace.

### Aces

**Plan.** Collect aces, both natural ones and raised kings, and win tricks with
them. This works in every strain and is strongest in NT, where aces can't be
trumped.

- **Buys:** aces, and kings to raise.
- **Bids:** "Aces? (4NT)" and "Aces? (4♣)" to reach slam.
- **Plays:** cash aces before the opponents can trump them.

Examples:

- [C, Trick ×] Tricks you win with an ace have +2× trick multiplier.
- [C, Flat] Your contracts are worth +40 for each ace in declarer's and dummy's
  hands.
- [U, Contract ×] +1× contract multiplier if your side holds four or more aces.
- [U, Utility] Your kings count as aces for your sigils.
- [R, Contract ×] +1× contract multiplier for each ace your side holds beyond
  three.

### Doubles

**Plan.** Penalty-double overreaching contracts and sacrifices. The opponents'
own multipliers raise the penalty ([§4](#failed-contracts)), and Doubles
sigils raise it again. This is the only archetype that scores on defense, with a
smaller offensive line in redoubled contracts.

- **Bids:** penalty doubles; competing to push opponents a level higher;
  redoubling with a sure contract.
- **Category note:** its per-trick and multiplier sigils are the defense versions,
  a per-undertrick bonus and a penalty multiplier.

Examples:

- [C, Per-trick] +50 for each undertrick you collect.
- [C, Contract ×] Penalties you collect from contracts you doubled have +1×
  multiplier.
- [U, Contract ×] If your side redoubles and makes the contract, +2× contract
  multiplier.
- [U, Utility] Each undertrick you collect pays you 10 gold.
- [R, Contract ×] Penalties you collect from contracts you doubled have +3×
  multiplier.

### Exact

**Plan.** Take exactly the contract, no more and no less. Exact rewards accurate
bidding and gives declarer a reason to give up an overtrick at the end.

Examples:

- [C, Flat] Your contracts are worth +150 if you make exactly your contract.
- [C, Contract ×] +1× contract multiplier if you make exactly your contract.
- [U, Contract ×] +2× contract multiplier if you make exactly a game or slam.
- [R, Contract ×] +1× contract multiplier for each exact contract your side has
  made this run.

### Fit

**Plan.** Fit payoffs scale with the trumps your side holds between both hands
and with tricks won by dummy, so both partners pile into one suit. In
single-player North owns nothing, so you build the fit alone: own a long trump
suit and rely on North's random share for the rest.

Examples:

- [C, Per-trick] Tricks won with dummy's cards are worth +20.
- [C, Contract ×] +1× contract multiplier if your side holds ten or more trumps.
- [U, Contract ×] +1× contract multiplier for each trump your side holds beyond
  eight.
- [R, Contract ×] +1× contract multiplier for each trump in dummy beyond three.

### Generic

Strain-agnostic point sigils fit any build and carry runs while a strain comes
together.

- [C, Per-trick] Tricks in your contracts are worth +5.
- [C, Per-trick] Tricks in your slams are worth +20.
- [U, Contract ×] Your vulnerable contracts have +1× contract multiplier.
- [R, Flat] Your grand slams are worth +1,000.
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
  10s count as honors; penalties against your side are halved.

## 10. Sigil pool skeleton

The first pool has 145 sigils. About 70% of commons, 50% of uncommons, 75% of
rares, and 60% of legendaries score points. Commons are mostly simple build-around
scoring, uncommons are the main home of utility and of contract multipliers,
rares return to exciting scoring, and legendaries are the splashiest effects.
Within trick-level scoring, per-trick points dominate and trick multipliers are
rare signature effects.

### By rarity and category

| Rarity | Flat | Per-trick | Trick × | Contract × | Utility | Total |
| --- | --- | --- | --- | --- | --- | --- |
| Common | 12 | 24 | 3 | 3 | 18 | 60 |
| Uncommon | 6 | 4 | 2 | 18 | 30 | 60 |
| Rare | 3 | 1 | 1 | 10 | 5 | 20 |
| Legendary | — | — | — | 3 | 2 | 5 |
| **Total** | **21** | **29** | **6** | **34** | **55** | **145** |

The six trick multipliers are Clubs' low clubs (C), Spades' ruffs (C), Aces'
ace tricks (C), Hearts' climbing run (U), Aces (U), and one generic rare.

### Point sigils by archetype

Each cell gives flat / per-trick / trick × / contract ×.

| Archetype | Common | Uncommon | Rare | Total |
| --- | --- | --- | --- | --- |
| ♣ The Long Suit | 2 / 3 / 1 / 0 | 1 / 0 / 0 / 2 | 0 / 0 / 0 / 1 | 10 |
| ♦ Treasury | 2 / 3 / 0 / 0 | 1 / 0 / 0 / 2 | 0 / 0 / 0 / 1 | 9 |
| ♥ The Run | 2 / 3 / 0 / 0 | 1 / 0 / 1 / 2 | 0 / 0 / 0 / 1 | 10 |
| ♠ Trump Power | 2 / 3 / 1 / 0 | 1 / 0 / 0 / 2 | 0 / 0 / 0 / 1 | 10 |
| NT Honors | 2 / 3 / 0 / 0 | 1 / 0 / 0 / 2 | 0 / 0 / 0 / 1 | 9 |
| Aces | 1 / 1 / 1 / 0 | 0 / 0 / 1 / 1 | 0 / 0 / 0 / 1 | 6 |
| Doubles | 0 / 2 / 0 / 1 | 0 / 0 / 0 / 2 | 0 / 0 / 0 / 1 | 6 |
| Exact | 1 / 0 / 0 / 1 | 0 / 0 / 0 / 2 | 0 / 0 / 0 / 1 | 5 |
| Fit | 0 / 1 / 0 / 1 | 0 / 0 / 0 / 2 | 0 / 0 / 0 / 1 | 5 |
| Generic | 0 / 5 / 0 / 0 | 1 / 4 / 0 / 1 | 3 / 1 / 1 / 1 | 17 |
| **Total** | **42** | **30** | **15** | **87, plus 3 legendary = 90** |

Each strain's two uncommon multipliers are a plain "+2× in this strain" and one
conditional on the strain's technique.

### Utility by family

| Family | Common | Uncommon | Rare | Legendary | Total |
| --- | --- | --- | --- | --- | --- |
| Economy and shop | 8 | 10 | 1 | — | 19 |
| Deal and card control | 6 | 10 | 2 | — | 18 |
| Rule benders | 4 | 10 | 2 | 2 | 18 |
| **Total** | **18** | **30** | **5** | **2** | **55** |

About 14 utility sigils are aimed at an archetype, roughly two per strain and
one per minor, such as Treasury's diamond gold or Aces' kings-as-aces. The rest
are generic.

### Budgets

These are starting sizes on a made contract of the right kind, before the
contract multiplier:

| Rarity | Flat | Per-trick | Trick × | Contract × |
| --- | --- | --- | --- | --- |
| Common | +100 in one strain, +50 broad | +10 in one strain, +5 broad | +2× on a narrow kind of trick | +1× on a narrow condition met in about a third of made contracts |
| Uncommon | +200 in one strain, +100 broad | +20 in one strain, +10 broad | +1× that climbs | +2× in one strain, +1× broad |
| Rare | +400, or scaling | +30, or scaling | scaling | +3× on a strain slam, +2× broad, or scaling |
| Legendary | — | — | — | ×2 compounding |

Checks against par:

- **Deal 4 (par 1,450):** a hearts partnership with two commons and an uncommon
  makes 4♥ vul with 11 tricks: 5 × 40 + 500 + 150 = 850, × 3 = 2,550 in its
  strain. Off-strain contracts and failures pull the average toward par.
- **Deal 8 (par 8,000):** six point sigils (two strain commons, a generic common,
  a strain +2×, a generic +1×, and a conditional strain +3×), plus engravings,
  make 6♥ vul with 12 tricks: about 6 × 60 + 1,500 = 1,860, × 4 to × 7 =
  7,400–13,000.

### How often a player sees support

With one reroll per shop, a player sees about 30 sigil offers per run: about 21
commons, 7.5 uncommons, 1.5 rares, and one legendary every three or four runs.
Each player therefore sees about 3 sigils aimed at any given strain per run.
An opposing pair shares a strain, so it sees about 6, plus about 3 generic
point sigils each. In single-player your double gold buys about three rerolls
per shop, so you alone see about 60 offers and a similar count. That is enough
for Balatro-style pivoting, but it is a risk to watch
([§14](#14-risks-and-tuning-levers)).

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
- **Each opposing pair settles on one strain** from its early offers and builds
  it together.
- Opposing AI seats buy, reroll, and sell under the same rules and prices as a
  human with a human partner.

## 13. Multiplayer

The first prototype is single-player only. Because every rule is symmetric
apart from AI partners, multiplayer needs only these additions:

- **Two humans** play as partners against two AI seats. **Four humans** play
  two against two. Human partners each shop with the normal economy and 5
  slots; only a human with an AI partner gets the doubled economy.
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
| Fit is weak in single-player because North's hand is random | Fit counts the whole side's cards; let you assign bought cards to North |
| Double gold inflates gold-scaling sigils, such as Treasury's "per 2 gold held" | Halve gold scaling for a doubled-economy seat, or raise Treasury thresholds |
| One buyer with 10 slots outbuilds two shopping opponents, or falls behind them | Income and slot count for the doubled-economy seat |
| Convert is 4 of 10 engraving offer types, 40% of engraving offers | Offer weights |
| Treasury's spend-or-bank tension is too weak or too harsh | Gold thresholds on Treasury sigils, interest cap |
| Full SAYC bidding plus competent card play is the largest implementation cost | Start with SAYC core bidding and sampling-based play, then widen |

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
| 18 | Strain identities | Long Suit, Treasury, The Run, Trump Power, Honors |
| 19 | Minor archetypes | Aces, Doubles, Exact, Fit |
| 20 | Per-trick bonuses vs trick multipliers | Mostly per-trick bonuses; about six trick multipliers |
| 21 | Utility families | Economy and shop, deal and card control, rule benders; no information |
| 22 | Convention coverage | The full SAYC booklet, labeled by meaning |
| 23 | Bidding suggestion | A subtle marker in the suggested call's hover card |
| 24 | AI bidding | SAYC meanings with sigil-aware judgment |
| 25 | AI shopping | Your AI partner never shops; opposing pairs share a strain |
| 26 | Multiplayer | Specified here; the first prototype is single-player |
| 27 | GDD scope | Systems, skeleton, and examples; the full sigil list comes later |
| 28 | Location | This file |
| 29 | Single-player buying | You are your side's only buyer, with double gold, income, interest cap, and sigil slots; opponents still shop |

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
