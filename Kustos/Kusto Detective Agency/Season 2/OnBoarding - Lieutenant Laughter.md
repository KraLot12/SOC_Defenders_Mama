# Lieutenant Laughter

## Section 1 - Case Description:

Imagine our agency as a buzzing beehive, like StackOverflow on steroids. We have a crazy number of cases popping up every day, each with a juicy bounty attached (yes, cold, hard cash!). And guess what? We've got thousands of Kusto Detectives scattered across the globe, all itching to pick a case and earn their detective stripes. But here's the catch: `only the first detective to crack the case gets the bounty` and major street cred!

So, your mission, should you choose to accept it, is to dig into the vast archives of our system operation logs from the legendary year 2022. You're on a quest to unearth the absolute legend, the detective with the biggest impact on our business—the one who raked in the most moolah by claiming bounties like a boss!

Feeling a bit rusty or want to level up your Kusto skills? No worries, my friend. We've got your back with the "Train Me" section. It's like a power-up that'll help you sharpen your Kusto-fu to tackle each case head-on. Oh, and if you stumble upon a mind-boggling case and need a little nudge, the "Hints" are there to save the day!

Now, strap on your detective hat, embrace the thrill, and get ready to rock this investigation. The fate of the "Most Epic Detective of the Year" rests in your hands!

Good luck, rookie, and remember to bring your sense of humor along for this wild ride!

Lieutenant Laughter

## Section 2 - Question

Who is the detective that earned most money in 2022?

## Section 3 - Approach

> Highlighted points from Case Description - `only the first detective to crack the case gets the bounty`

```
DetectiveCases
| where EventType contains "CaseSolved"
| summarize arg_min(Timestamp,*) by CaseId
```
> Extracting bounty from CaseOpened events (Since bounty is available only in CaseOpened events)

```
DetectiveCases
    | where EventType contains "CaseOpened"
    | extend Bounty = tolong(Properties.Bounty)
    | distinct CaseId, Bounty
```

> Now that we have detective who solved the case first and bounty for each case, we need to append bounty to case solved using join operator.

```
DetectiveCases
| where EventType contains "CaseSolved"
| summarize arg_min(Timestamp,*) by CaseId
| join kind=leftouter (
    DetectiveCases
    | where EventType contains "CaseOpened"
    | extend Bounty = tolong(Properties.Bounty)
) on CaseId
```

> To calculate the total bounty collected by each detective, you can use the summarize operator with the sum() function. This will group the data by detective and calculate the sum of their bounties. Here's a simple statement to achieve that:

```
DetectiveCases
| where EventType contains "CaseSolved"
| summarize arg_min(Timestamp,*) by CaseId
| join kind=leftouter (
    DetectiveCases
    | where EventType contains "CaseOpened"
    | extend Bounty = tolong(Properties.Bounty)
) on CaseId
| summarize BountyCollected=sum(Bounty) by DetectiveId
```

<details>
<summary>Answer</summary>

## Section 4 - Kusto'd

```
DetectiveCases
| where EventType contains "CaseSolved"
| summarize arg_min(Timestamp,*) by CaseId
| join kind=leftouter (
    DetectiveCases
    | where EventType contains "CaseOpened"
    | extend Bounty = tolong(Properties.Bounty)
) on CaseId
| summarize BountyCollected=sum(Bounty) by DetectiveId
| top 1 by BountyCollected
```
</details>
