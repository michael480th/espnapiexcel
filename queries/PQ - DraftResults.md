# Draft Results Query

**Query Name:** `DraftResults`

Extracts draft picks, rounds, and draft details.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"mDraftDetail"}),
    draftDetail = Lib[SafeNestedRecord](data, {"draftDetail"}),
    
    // Check if draft exists
    isDrafted = Lib[SafeLogical](Lib[SafeRecordField](draftDetail, "drafted", false)),
    isInProgress = Lib[SafeLogical](Lib[SafeRecordField](draftDetail, "inProgress", false)),
    completeDate = Lib[SafeDateTime](Lib[SafeRecordField](draftDetail, "completeDate")),
    
    // Get draft picks
    picks = Lib[SafeNestedList](draftDetail, {"picks"}),
    
    // Process each draft pick
    processedPicks = if isDrafted then List.Transform(picks, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        PickId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        
        // Pick information
        OverallPickNumber = Lib[SafeNumber](Lib[SafeRecordField](_, "overallPickNumber", 0)),
        RoundId = Lib[SafeNumber](Lib[SafeRecordField](_, "roundId", 0)),
        RoundPickNumber = Lib[SafeNumber](Lib[SafeRecordField](_, "roundPickNumber", 0)),
        
        // Team information
        TeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "teamId", 0)),
        MemberId = Lib[SafeText](Lib[SafeRecordField](_, "memberId", "")),
        NominatingTeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "nominatingTeamId", 0)),
        
        // Player information
        PlayerId = Lib[SafeNumber](Lib[SafeRecordField](_, "playerId", 0)),
        LineupSlotId = Lib[SafeNumber](Lib[SafeRecordField](_, "lineupSlotId", 0)),
        
        // Auction information
        BidAmount = Lib[SafeNumber](Lib[SafeRecordField](_, "bidAmount", 0)),
        
        // Keeper information
        IsKeeper = Lib[SafeLogical](Lib[SafeRecordField](_, "keeper", false)),
        KeeperValue = Lib[SafeNumber](Lib[SafeRecordField](_, "keeperValue", 0)),
        ReservedForKeeper = Lib[SafeLogical](Lib[SafeRecordField](_, "reservedForKeeper", false)),
        
        // Draft status
        AutoDraftTypeId = Lib[SafeNumber](Lib[SafeRecordField](_, "autoDraftTypeId", 0)),
        TradeLocked = Lib[SafeLogical](Lib[SafeRecordField](_, "tradeLocked", false)),
        
        // Owning teams
        OwningTeamIds = Lib[SafeText](Lib[SafeRecordField](_, "owningTeamIds", ""))
    ]) else {},
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedPicks),
    
    // Sort by overall pick number
    sortedTable = Table.Sort(table, {{"OverallPickNumber", Order.Ascending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "PickId", "OverallPickNumber", "RoundId", "RoundPickNumber",
        "TeamId", "MemberId", "NominatingTeamId", "PlayerId", "LineupSlotId",
        "BidAmount", "IsKeeper", "KeeperValue", "ReservedForKeeper",
        "AutoDraftTypeId", "TradeLocked", "OwningTeamIds"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
