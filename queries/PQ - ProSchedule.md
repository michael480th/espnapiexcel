# Pro Schedule Query

**Query Name:** `ProSchedule`

Extracts NFL team schedules and game information.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"proTeamSchedules_wl"}),
    proTeams = Lib[SafeNestedList](data, {"settings", "proTeams"}),
    
    // Process each pro team
    processedTeams = List.Transform(proTeams, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        
        // Team information
        ProTeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        ProTeamName = Lib[SafeText](Lib[SafeRecordField](_, "name", "")),
        ProTeamAbbrev = Lib[SafeText](Lib[SafeRecordField](_, "abbrev", "")),
        
        // Schedule information
        ProGamesByScoringPeriod = Lib[SafeText](Lib[SafeRecordField](_, "proGamesByScoringPeriod", ""))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedTeams),
    
    // Sort by team ID
    sortedTable = Table.Sort(table, {{"ProTeamId", Order.Ascending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "ProTeamId", "ProTeamName", "ProTeamAbbrev", "ProGamesByScoringPeriod"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
