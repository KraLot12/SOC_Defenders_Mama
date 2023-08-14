# Captain S.Impson
## To bill or not to bill?

Imagine this: It's a fresh new year, and citizens of Digitown are in an uproar. ` Their water and electricity bills have inexplicably doubled, despite no changes in their consumption`. To make matters worse, the upcoming mayoral election amplifies the urgency to resolve this issue promptly.

#Question
`What is the total bills amount due in April?`
Here we need to find the actual total cost in April month

so let dig into data and will figure out whats wrong in the caluclations

Firstly we have 2 different tabels `Costs` and `Consumption`.
first we will dig in to `Consumption` tabel, According to the case description `Their water and electricity bills have inexplicably doubled` this is the issue, lets figure out if htere are any duplciates present in the data.
```
Consumption
| summarize count() by Timestamp, HouseholdId, MeterType, Consumed
| where count_ >1
```
the above query shows that there is a duplicate data, it reflects that few customers got the duplicate bills.
to remove this and lets find only the distinct values. we will use the below query 
```
Consumption
| distinct  Timestamp, HouseholdId, MeterType, Consumed
```
Next lets check the other tabel `Costs` 
```
Costs
```
lets add this tabel data to our consumption code, we can use Lookup/ join/ extend(hard code the cost details) operators 

### Lookup operator
```
Consumption
| distinct  Timestamp, HouseholdId, MeterType, Consumed
| lookup Costs on MeterType
| take 10
```
### Join operator
```
Consumption
| distinct  Timestamp, HouseholdId, MeterType, Consumed
| join kind=inner (
Costs
) on MeterType
| take 10
```
### Extend
```
Consumption
| distinct  Timestamp, HouseholdId, MeterType, Consumed
| extend Cost=iff(MeterType == "Water",0.001562,0.3016)
| take 10
```
Now lets caluclat the total cost
```
Consumption
| distinct  Timestamp, HouseholdId, MeterType, Consumed
| lookup Costs on MeterType  
| extend  totalcost = Consumed*Cost
| take 10
```
Now the final result is lets add total sum of total cose for each customer to get the total bill in April
```
Consumption
| distinct  Timestamp, HouseholdId, MeterType, Consumed
| lookup Costs on MeterType  
| extend  totalcost = Consumed*Cost
| summarize sum(totalcost)
```
When I submitted this answer its says wrong, so I further dig into the `train me` section and I looked at max and min value of each meter type
```
Consumption  
| summarize max(Consumed), min(Consumed) by MeterType
```
Here we can see the minimum consumption is in negitive value, WHy the consumption will be in negitive, this negitive data is a  garbage data, we have to remove in our data. Here is the modified query

```
Consumption
| where Consumed >0
| distinct  Timestamp, HouseholdId, MeterType, Consumed
| lookup Costs on MeterType  
| extend  totalcost = Consumed*Cost
| summarize sum(totalcost)
```

here we go, the task is completed. waiting for the next case.
