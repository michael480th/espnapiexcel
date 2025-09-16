# Transactions Query

**Query Name:** `Transactions`

Extracts transaction history including adds, drops, trades, and waivers.

```powerquery-m
let
    Lib = Lib_Espn,
    // Get current scoring period from league settings
    leagueSettings = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"mSettings"}),
    currentScoringPeriod = try leagueSettings[status][latestScoringPeriod] otherwise 1,
    
    data = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"mTransactions2"}, [
        scoringPeriodId = currentScoringPeriod
    ]),
    transactions = Lib[SafeNestedList](data, {"transactions"}),
    
    // Process each transaction
    processedTransactions = List.Transform(transactions, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        ScoringPeriodId = currentScoringPeriod,
        
        // Transaction information
        TransactionId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        TransactionType = Lib[SafeText](Lib[SafeRecordField](_, "type", "")),
        Status = Lib[SafeText](Lib[SafeRecordField](_, "status", "")),
        
        // Team information
        TeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "teamId", 0)),
        
        // Date information
        ProcessDate = Lib[SafeDateTime](Lib[SafeRecordField](_, "processDate")),
        ProposedDate = Lib[SafeDateTime](Lib[SafeRecordField](_, "proposedDate")),
        
        // Bid information
        BidAmount = Lib[SafeNumber](Lib[SafeRecordField](_, "bidAmount", 0)),
        
        // Transaction items (players involved)
        Items = Lib[SafeNestedList](_, {"items"}),
        ItemCount = List.Count(Items),
        
        // Extract first item details (most transactions have one item)
        FirstItemType = if List.Count(Items) > 0 then 
            Lib[SafeText](Lib[SafeRecordField](Items{0}, "type", "")) 
        else 
            "",
        FirstItemPlayerId = if List.Count(Items) > 0 then 
            Lib[SafeNumber](Lib[SafeRecordField](Items{0}, "playerId", 0)) 
        else 
            0,
        
        // Additional transaction data
        IsWaiver = Lib[SafeLogical](Lib[SafeRecordField](_, "isWaiver", false)),
        IsTrade = Lib[SafeLogical](Lib[SafeRecordField](_, "isTrade", false)),
        IsFreeAgent = Lib[SafeLogical](Lib[SafeRecordField](_, "isFreeAgent", false))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedTransactions),
    
    // Sort by process date (most recent first)
    sortedTable = Table.Sort(table, {{"ProcessDate", Order.Descending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "ScoringPeriodId", "TransactionId", "TransactionType", "Status",
        "TeamId", "ProcessDate", "ProposedDate", "BidAmount", "ItemCount",
        "FirstItemType", "FirstItemPlayerId", "IsWaiver", "IsTrade", "IsFreeAgent"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
