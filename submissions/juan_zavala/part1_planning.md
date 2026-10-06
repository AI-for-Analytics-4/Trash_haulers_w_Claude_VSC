# Part 1: Understanding and Planning (No Code Yet)

**Data:** `data/trash_hauler_report_with_lat_lng.csv`. It has 20,226 hubNashville service requests, opened from Nov 1, 2017 to Nov 1, 2019. There are 13 columns, and every row has a unique `Request Number`.

**Prompt I gave Claude:**
> "I'm working in Python with pandas. I want to think through the approach to this problem. Do not write any code yet. Explain the logic, assumptions, and possible approaches in plain English only. Here is the README and the CSV."

Before answering, I had Claude look at the CSV itself, so its answers are based on the real data and not on guesses. The numbers below come from that first look. They are not final results.

---

## 1. How would you identify repeat missed pickups at the same address?

**First, decide which rows count as missed pickups.** The `Request` column has four values:

| Request type | Rows | Is it a missed pickup? |
|---|---:|---|
| Trash - Curbside/Alley Missed Pickup | 15,028 | **Yes.** This is the main category. |
| Trash - Backdoor | 2,629 | **Yes.** These are missed pickups for back-door service customers. About 77% of the descriptions say "missed" or "still out." |
| Trash Collection Complaint | 2,312 | **No by default.** Most are about crews, spills or rules, though 691 mention "miss." |
| Damage to Property | 257 | **No.** These are mailboxes, walls and fluid leaks. |

That gives **17,657 missed-pickup rows**. Keeping complaints out is a judgment call. I'll note it as an assumption rather than hide it.

**Then the logic is:**
1. Standardize the address so the same house is spelled one way (see question 2).
2. Sort the missed pickups by address, then by date.
3. Number each address's pickups in date order: 1st, 2nd, 3rd, and so on.
4. The 1st pickup at an address costs $0. Every pickup numbered 2 or higher costs $200.

pandas can number rows within each address all at once (a "cumulative count within each group"), so I don't need a loop or a custom function. This also sets up Part 4. For the $500 rule I'll need to count how many misses fall in the 6 months before each one, using a rolling window over the same sorted data.

---

## 2. What assumptions need to be made about how addresses are standardized?

The `Incident Address` column is the biggest problem in this data. The same house shows up written several different ways:

```
1007 ALICE ST, 37218
1007 ALICE ST, NASHVILLE, TENNESSEE, 37218
1007 ALICE ST, NASHVILLE, TN 37218, UNITED STATES
1007 ALICE ST B
```

```
101 AILEEN CT, ANTIOCH, TN 37013, USA
101 AILEEN CT, NASHVILLE, TN 37013, UNITED STATES
```

These are the assumptions I plan to make:

| Assumption | Why it matters |
|---|---|
| **Case and spaces don't matter.** Convert to upper case, trim, and turn repeated spaces into one. | This removes surface differences. |
| **Only the street part counts.** Keep the text before the first comma and drop the city, state, ZIP and country. | Some rows have the full mailing address and some have only the street. This step alone cuts the distinct missed-pickup addresses from **12,384 to 11,192**. |
| **Street suffixes mean the same thing.** ROAD becomes RD, DRIVE becomes DR, AVENUE becomes AVE, and so on. Periods are removed. | "Brush Hill Road" and "Brush Hill Rd" are one premises. |
| **Different units are different premises.** "320 OLD HICKORY BLVD 3024" and "...2111" are kept separate. | The contract fines repeat misses at the *same premises*. Two apartments aren't the same customer. (This is the one I'm least sure of. It could also be read as one building.) |
| **Same text means same place.** I won't try to fix typos or do fuzzy matching. | Fuzzy matching could merge two real neighbors, like 1000 N 14TH ST and 1000 N 7TH ST. |
| **The lat/long columns are a check, not the key.** | 119 coordinate points cover more than one address, such as apartment complexes, so coordinates alone would over-merge. |

---

## 3. What edge cases could cause over- or under-counting fines?

**These could cause over-counting (fines that are too high):**

- **Duplicate reports of one miss.** 295 missed-pickup rows repeat an address that already has a report on the same day, for example when a resident calls twice or two neighbors report it. A $200 fine should go with one missed pickup, not one phone call. **Plan:** count at most one miss per address per day.
- **The wrong hauler.** The fine is part of Metro's contract with **Red River**, but the data also has METRO (3,580 rows across "METRO" and "Metro") and WASTE IND (1,350 rows). Fining those would charge Red River for misses it didn't make. **Plan:** count only Red River rows, and make "Metro" and "METRO" the same value.
- **Counting complaints or damage reports as misses** would inflate the totals (see question 1).

**These could cause under-counting (fines that are too low):**

- **Unstandardized addresses.** The same house split across two spellings looks like two "first misses," which hides real repeats. This is the main reason question 2 matters.
- **A blank hauler.** 901 rows have no `Trash Hauler`. If I filter to Red River, these drop out, even though some are probably Red River routes. **Plan:** report how many there are. I could fill them in from another row at the same address or on the same `Trash Route`.
- **Whole-street misses.** About 1,900 descriptions say things like "the entire street was missed" or "whole court." One request then covers many houses, but it can only be fined once, at the one address it lists. The data can't fix this, so I'll name it as a limitation.
- **Missing addresses.** 9 rows have no address, and 48 have no house number (for example "XAVIER DR"). These can't be matched reliably and should be left out of the repeat counts and reported.

**Other cases:**

- **Same-day order.** `Date Opened` has only the date, with no time, so I can't tell which of two same-day misses came first. Removing same-day duplicates mostly settles this.
- **The data starts Nov 1, 2017.** A house's "first" miss in this file may not be its real first miss. I'll treat the data window as the whole history.
- **The 6-month window in Part 4.** It needs an exact definition, such as 182 days or 6 calendar months, and that choice moves a few fines across the $500 line.

---

## My planned approach (summary paragraph)

I'll start by keeping only the missed-pickup requests, "Trash - Curbside/Alley Missed Pickup" and "Trash - Backdoor." Then I'll filter to Red River, since the fines come from Metro's contract with that hauler, and report the 901 rows with no hauler separately. Next I'll standardize `Incident Address`: upper case, trimmed spaces, only the street part before the first comma, and abbreviated suffixes. Different units stay separate premises, and I won't do fuzzy matching. After that I'll drop duplicate reports of the same address on the same day, so one missed pickup is counted once even if it was called in twice. With clean data, I'll sort by address and date and number each address's misses in order. The first is $0 and every later one is $200. I'll use pandas' built-in group methods instead of a custom function. To check the work, I'll pick a few addresses with many misses, such as one with 15 or more, and compare the code's count and fine with a manual count of their rows. I'll also confirm that the totals add up: missed pickups = first misses + fined misses. Each assumption, especially the hauler filter, the complaint rows and the duplicate rule, will be written down, because each one changes the final dollar amount.
