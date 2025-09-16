# Players Directory Query

**Query Name:** `PlayersDirectory`

Extracts the complete player database with basic player information.

```powerquery-m
let
    Lib = Lib_Espn,
    
    // Get current scoring period from league settings
    leagueSettings = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"mSettings"}),
    currentScoringPeriod = try leagueSettings[status][latestScoringPeriod] otherwise 1,
    
    // Use players_wl endpoint
    data = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"players_wl"}),
    players = Lib[SafeNestedList](data, {"players"}),
    
    // Process each player
    processedPlayers = List.Transform(players, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        ScoringPeriodId = currentScoringPeriod,
        
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
        
        // Professional team
        ProTeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "proTeamId", 0)),
        
        // Ownership information
        PercentOwned = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"ownership", "percentOwned"}), 0),
        
        // Additional data
        UniverseId = Lib[SafeNumber](Lib[SafeRecordField](_, "universeId", 0))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedPlayers),
    
    // Sort by player name
    sortedTable = Table.Sort(table, {{"PlayerName", Order.Ascending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "ScoringPeriodId", "PlayerId", "PlayerName", "PlayerFirstName", "PlayerLastName",
        "DefaultPositionId", "EligibleSlots", "IsActive", "IsInjured", "IsDroppable",
        "ProTeamId", "PercentOwned", "UniverseId"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
