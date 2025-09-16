# Matchups By Week Query

Extracts weekly matchup scores and results for a specific scoring period.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Parameters[LeagueId], {"mMatchupScore"}, [
        scoringPeriodId = Parameters[ScoringPeriodId]
    ]),
    schedule = Lib[SafeNestedList](data, {"schedule"}),
    
    // Process each matchup for the specified week
    processedMatchups = List.Transform(schedule, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        ScoringPeriodId = Parameters[ScoringPeriodId],
        MatchupId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        MatchupPeriodId = Lib[SafeNumber](Lib[SafeRecordField](_, "matchupPeriodId", 0)),
        PlayoffTierType = Lib[SafeText](Lib[SafeRecordField](_, "playoffTierType", "")),
        IsPlayoff = Lib[SafeLogical](Lib[SafeRecordField](_, "isPlayoff", false)),
        
        // Home team details
        HomeTeamId = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"home", "teamId"}), 0),
        HomeTeamName = Lib[SafeText](Lib[SafeNestedRecord](_, {"home", "team", "name"})),
        HomeTeamLocation = Lib[SafeText](Lib[SafeNestedRecord](_, {"home", "team", "location"})),
        HomeTeamAbbrev = Lib[SafeText](Lib[SafeNestedRecord](_, {"home", "team", "abbrev"})),
        HomeScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"home", "totalPoints"}), 0),
        HomeProjectedScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"home", "projectedPoints"}), 0),
        HomeAdjustment = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"home", "adjustment"}), 0),
        
        // Away team details
        AwayTeamId = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"away", "teamId"}), 0),
        AwayTeamName = Lib[SafeText](Lib[SafeNestedRecord](_, {"away", "team", "name"})),
        AwayTeamLocation = Lib[SafeText](Lib[SafeNestedRecord](_, {"away", "team", "location"})),
        AwayTeamAbbrev = Lib[SafeText](Lib[SafeNestedRecord](_, {"away", "team", "abbrev"})),
        AwayScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"away", "totalPoints"}), 0),
        AwayProjectedScore = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"away", "projectedPoints"}), 0),
        AwayAdjustment = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"away", "adjustment"}), 0),
        
        // Matchup results
        Winner = Lib[SafeText](Lib[SafeRecordField](_, "winner", "")),
        IsTie = Lib[SafeLogical](Lib[SafeRecordField](_, "isTie", false)),
        IsBye = Lib[SafeLogical](Lib[SafeRecordField](_, "isBye", false)),
        MarginOfVictory = Lib[SafeNumber](Lib[SafeRecordField](_, "marginOfVictory"), 0),
        
        // Status and timing
        MatchupDate = Lib[SafeDateTime](Lib[SafeRecordField](_, "date")),
        MatchupStatus = Lib[SafeText](Lib[SafeRecordField](_, "status", "")),
        IsMatchupComplete = Lib[SafeLogical](Lib[SafeRecordField](_, "isMatchupComplete", false))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedMatchups),
    
    // Sort by matchup ID
    sortedTable = Table.Sort(table, {{"MatchupId", Order.Ascending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "ScoringPeriodId", "MatchupId", "MatchupPeriodId", "PlayoffTierType", "IsPlayoff",
        "HomeTeamId", "HomeTeamName", "HomeTeamLocation", "HomeTeamAbbrev", "HomeScore", "HomeProjectedScore", "HomeAdjustment",
        "AwayTeamId", "AwayTeamName", "AwayTeamLocation", "AwayTeamAbbrev", "AwayScore", "AwayProjectedScore", "AwayAdjustment",
        "Winner", "IsTie", "IsBye", "MarginOfVictory", "MatchupDate", "MatchupStatus", "IsMatchupComplete"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
