# Catch the Phishermen!
The complaints are pouring in, and people are fed up with the sudden increase in phishing calls attempting to steal their identity details. We can't let these scammers get away with it, and we need your help to catch them!

lets start digging into the data we have

```
PhoneCalls
| take 10
```
The `phonecall` tabel has properties column where there was some more key values pairs are there, lets try to extend those values in to columns

```
PhoneCalls
| take 10
| extend origin = Properties.Origin, destination = Properties.Destination, ishidden = Properties.IsHidden, Disconnected = Properties.DisconnectedBy
```

Lets findout the Phisherman
To locate the phone call number, `we should examine the "Origin" field, as spammers typically initiate calls`. Additionally, `when users perceive suspicion, they tend to disconnect such calls`. Therefore, `our objective is to identify the highest frequency of calls based on the "Origin" field and the most frequent disconnections at the destination.` Furthermore, `we have another field, "IsHidden," which is often employed by spammers using concealed numbers.` This provides an additional criterion to consider for filtering purposes.

to findout the Hidden number
```
PhoneCalls
| where EventType == "Connect"
| extend origin = Properties.Origin, destination = Properties.Destination, ishidden = Properties.IsHidden, Disconnected = Properties.DisconnectedBy
| where ishidden == "true"
```
Now, we need to find the most frequent disconnections at the destination.
```
PhoneCalls
| where EventType != "Connect"
| extend origin = Properties.Origin, destination = Properties.Destination, ishidden = Properties.IsHidden, Disconnected = Properties.DisconnectedBy
| where Disconnected == "Destination"
```
Joining last 2 quires we will get our spammer by validating hte most number of calls.
```
PhoneCalls
| where EventType == "Connect"
| extend origin = Properties.Origin, destination = Properties.Destination, ishidden = Properties.IsHidden, Disconnected = Properties.DisconnectedBy
| where ishidden == "true"
| join kind=inner (
PhoneCalls
| where EventType != "Connect"
| extend origin = Properties.Origin, destination = Properties.Destination, ishidden = Properties.IsHidden, Disconnected = Properties.DisconnectedBy
| where Disconnected == "Destination"
) on CallConnectionId
| summarize count() by tostring(origin)
```
Finally we got the spammer, Enjoy Hunting #kql

