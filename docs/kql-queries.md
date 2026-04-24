# KQL Queries

All queries used in the Sentinel workbook.

## Common filters

Most queries include filters to exclude noise:

```kql
| where isnotempty(IpAddress)
| where IpAddress != "-"
| where IpAddress !startswith "10."
| where IpAddress !startswith "192.168."
```


## Attack volume over time

```kql
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| summarize FailedLogons = count() by bin(TimeGenerated, 1h)
| render timechart
```


##  Top attempted usernames

```kql
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| where isnotempty(TargetUserName)
| where TargetUserName != "-"
| summarize Attempts = count() by TargetUserName
| order by Attempts desc
| take 20
```

**Purpose:** Shows which usernames attackers try most often. Standard targets (`administrator`, `admin`, `root`, `user`) dominate.

## Top source IPs

```kql
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| where isnotempty(IpAddress)
| where IpAddress != "-"
| where IpAddress !startswith "10." and IpAddress !startswith "192.168."
| summarize Attempts = count() by IpAddress
| order by Attempts desc
| take 20
```

**Purpose:** Identifies the most active source IPs. 

## Attacks by country

```kql
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| where isnotempty(IpAddress)
| where IpAddress != "-"
| where IpAddress !startswith "10." and IpAddress !startswith "192.168."
| summarize Attempts = count() by IpAddress
| extend Country = tostring(geo_info_from_ip_address(IpAddress).country)
| where isnotempty(Country)
| summarize TotalAttempts = sum(Attempts) by Country
| order by TotalAttempts desc
```


## Authentication event type distribution

```kql
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID in (4624, 4625, 4648)
| summarize Count = count() by EventID
```