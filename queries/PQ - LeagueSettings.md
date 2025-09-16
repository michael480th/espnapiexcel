# League Settings Query

Extracts league configuration, scoring settings, and basic league information.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Parameters[LeagueId], {"mSettings"}),
    settings = Lib[SafeNestedRecord](data, {"settings"}),
    
    // Extract core settings
    leagueInfo = [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        LeagueName = Lib[SafeText](Lib[SafeRecordField](settings, "name", "")),
        SeasonId = Lib[SafeNumber](Lib[SafeRecordField](settings, "seasonId", 0)),
        GameId = Lib[SafeNumber](Lib[SafeRecordField](settings, "gameId", 0)),
        Status = Lib[SafeText](Lib[SafeRecordField](settings, "status", "")),
        PreviousSeasons = Lib[SafeText](Lib[SafeNestedRecord](settings, {"status", "previousSeasons"})),
        CurrentMatchupPeriod = Lib[SafeNumber](Lib[SafeNestedRecord](settings, {"status", "currentMatchupPeriod"}), 0),
        FinalScoringPeriod = Lib[SafeNumber](Lib[SafeNestedRecord](settings, {"status", "finalScoringPeriod"}), 0)
    ],
    
    // Extract roster settings
    rosterSettings = Lib[SafeNestedRecord](settings, {"rosterSettings"}),
    rosterInfo = [
        RosterSize = Lib[SafeNumber](Lib[SafeRecordField](rosterSettings, "rosterSize", 0)),
        LineupSlotCounts = Lib[SafeText](Lib[SafeNestedRecord](rosterSettings, {"lineupSlotCounts"})),
        BenchCount = Lib[SafeNumber](Lib[SafeRecordField](rosterSettings, "benchCount", 0)),
        IRCount = Lib[SafeNumber](Lib[SafeRecordField](rosterSettings, "injuredReserveCount", 0))
    ],
    
    // Extract scoring settings
    scoringSettings = Lib[SafeNestedRecord](settings, {"scoringSettings"}),
    scoringInfo = [
        ScoringType = Lib[SafeText](Lib[SafeRecordField](scoringSettings, "scoringType", "")),
        ScoringItems = Lib[SafeText](Lib[SafeNestedRecord](scoringSettings, {"scoringItems"}))
    ],
    
    // Extract playoff settings
    playoffSettings = Lib[SafeNestedRecord](settings, {"scheduleSettings", "playoffSettings"}),
    playoffInfo = [
        PlayoffSeeds = Lib[SafeNumber](Lib[SafeRecordField](playoffSettings, "playoffSeeds", 0)),
        PlayoffRounds = Lib[SafeNumber](Lib[SafeRecordField](playoffSettings, "playoffRounds", 0)),
        PlayoffSeedTieRule = Lib[SafeText](Lib[SafeRecordField](playoffSettings, "playoffSeedTieRule", ""))
    ],
    
    // Combine all settings
    combinedSettings = Record.Combine({leagueInfo, rosterInfo, scoringInfo, playoffInfo}),
    
    // Convert to table
    table = Table.FromRecords({combinedSettings}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "LeagueName", "SeasonId", "GameId", "Status",
        "PreviousSeasons", "CurrentMatchupPeriod", "FinalScoringPeriod",
        "RosterSize", "LineupSlotCounts", "BenchCount", "IRCount",
        "ScoringType", "ScoringItems", "PlayoffSeeds", "PlayoffRounds", "PlayoffSeedTieRule"
    },
    finalTable = Lib[EnsureColumns](table, requiredColumns)
in
    finalTable
```
