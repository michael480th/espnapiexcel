# Schedule Query

Extracts the full season schedule with matchup information.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Parameters[LeagueId], {"mMatchup"}),
    schedule = Lib[SafeNestedList](data, {"schedule"}),
    
    // Process each matchup
    processedSchedule = List.Transform(schedule, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        MatchupId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        MatchupPeriodId = Lib[SafeNumber](Lib[SafeRecordField](_, "matchupPeriodId", 0)),
        PlayoffTierType = Lib[SafeText](Lib[SafeRecordField](_, "playoffTierType", "")),
        IsPlayoff = Lib[SafeLogical](Lib[SafeRecordField](_, "isPlayoff", false)),
        
        // Home team
        HomeTeamId = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"home", "teamId"}), 0),
        HomeTeamName = Lib[SafeText](Lib[SafeNestedRecord](_, {"home", "team", "name"})),
        HomeTeamLocation = Lib[SafeText](Lib[SafeNestedRecord](_, {"home", "team", "location"})),
        HomeTeamAbbrev = Lib[SafeText](Lib[SafeNestedRecord](_, {"home", "team", "abbrev"})),
        HomeScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"home", "totalPoints"}), 0),
        HomeProjectedScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"home", "projectedPoints"}), 0),
        
        // Away team
        AwayTeamId = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"away", "teamId"}), 0),
        AwayTeamName = Lib[SafeText](Lib[SafeNestedRecord](_, {"away", "team", "name"})),
        AwayTeamLocation = Lib[SafeText](Lib[SafeNestedRecord](_, {"away", "team", "location"})),
        AwayTeamAbbrev = Lib[SafeText](Lib[SafeNestedRecord](_, {"away", "team", "abbrev"})),
        AwayScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"away", "totalPoints"}), 0),
        AwayProjectedScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"away", "projectedPoints"}), 0),
        
        // Matchup details
        Winner = Lib[SafeText](Lib[SafeRecordField](_, "winner", "")),
        IsTie = Lib[SafeLogical](Lib[SafeRecordField](_, "isTie", false)),
        IsBye = Lib[SafeLogical](Lib[SafeRecordField](_, "isBye", false)),
        
        // Date information
        MatchupDate = Lib[SafeDateTime](Lib[SafeRecordField](_, "date")),
        MatchupStatus = Lib[SafeText](Lib[SafeRecordField](_, "status", ""))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedSchedule),
    
    // Sort by matchup period and matchup ID
    sortedTable = Table.Sort(table, {{"MatchupPeriodId", Order.Ascending}, {"MatchupId", Order.Ascending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "MatchupId", "MatchupPeriodId", "PlayoffTierType", "IsPlayoff",
        "HomeTeamId", "HomeTeamName", "HomeTeamLocation", "HomeTeamAbbrev", "HomeScore", "HomeProjectedScore",
        "AwayTeamId", "AwayTeamName", "AwayTeamLocation", "AwayTeamAbbrev", "AwayScore", "AwayProjectedScore",
        "Winner", "IsTie", "IsBye", "MatchupDate", "MatchupStatus"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
