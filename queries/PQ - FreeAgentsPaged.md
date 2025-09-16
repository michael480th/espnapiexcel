# Free Agents Paged Query

Extracts available free agents with pagination support for large datasets.

```powerquery-m
let
    Lib = Lib_Espn,
    
    // Build URL function for pagination
    makeUrl = (offset, pageSize) =>
        Lib[BuildUrl](Parameters[Season], Parameters[LeagueId], {"kona_player_info"}, [
            scoringPeriodId = Parameters[ScoringPeriodId],
            limit = pageSize,
            offset = offset
        ]),
    
    // Extract players from response
    extractPlayers = (data) => Lib[SafeNestedList](data, {"players"}),
    
    // Get all free agents using pagination
    allPlayers = Lib[Paginate](0, Parameters[PageSize], makeUrl, extractPlayers),
    
    // Process each player
    processedPlayers = List.Transform(allPlayers, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        ScoringPeriodId = Parameters[ScoringPeriodId],
        
        // Player information
        PlayerId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        PlayerName = Lib[SafeText](Lib[SafeRecordField](_, "fullName", "")),
        PlayerFirstName = Lib[SafeText](Lib[SafeRecordField](_, "firstName", "")),
        PlayerLastName = Lib[SafeText](Lib[SafeRecordField](_, "lastName", "")),
        DefaultPositionId = Lib[SafeNumber](Lib[SafeRecordField](_, "defaultPositionId", 0)),
        EligibleSlots = Lib[SafeText](Lib[SafeRecordField](_, "eligibleSlots", "")),
        
        // Player status
        IsActive = Lib[SafeLogical](Lib[SafeRecordField](_, "active", false)),
        IsInjured = Lib[SafeLogical](Lib[SafeRecordField](_, "injured", false)),
        IsDroppable = Lib[SafeLogical](Lib[SafeRecordField](_, "droppable", false)),
        InjuryStatus = Lib[SafeText](Lib[SafeRecordField](_, "injuryStatus", "")),
        
        // Professional team
        ProTeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "proTeamId", 0)),
        
        // Ownership information
        PercentOwned = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"ownership", "percentOwned"}), 0),
        PercentStarted = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"ownership", "percentStarted"}), 0),
        
        // Stats for the week
        AppliedStatTotal = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"stats", 0, "appliedTotal"}), 0),
        ProjectedPoints = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"stats", 0, "projectedTotal"}), 0),
        
        // Additional player data
        UniverseId = Lib[SafeNumber](Lib[SafeRecordField](_, "universeId", 0)),
        Status = Lib[SafeText](Lib[SafeRecordField](_, "status", ""))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedPlayers),
    
    // Sort by percent owned (most owned first)
    sortedTable = Table.Sort(table, {{"PercentOwned", Order.Descending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "ScoringPeriodId", "PlayerId", "PlayerName", "PlayerFirstName", "PlayerLastName",
        "DefaultPositionId", "EligibleSlots", "IsActive", "IsInjured", "IsDroppable", "InjuryStatus",
        "ProTeamId", "PercentOwned", "PercentStarted", "AppliedStatTotal", "ProjectedPoints",
        "UniverseId", "Status"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
