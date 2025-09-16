# Standings Query

**Query Name:** `Standings`

Extracts current league standings and team records.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"mStandings"}),
    teams = Lib[SafeNestedList](data, {"teams"}),
    
    // Process each team for standings
    processedStandings = List.Transform(teams, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        TeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        TeamName = Lib[SafeText](Lib[SafeRecordField](_, "name", "")),
        Location = Lib[SafeText](Lib[SafeRecordField](_, "location", "")),
        Abbrev = Lib[SafeText](Lib[SafeRecordField](_, "abbrev", "")),
        DivisionId = Lib[SafeNumber](Lib[SafeRecordField](_, "divisionId", 0)),
        
        // Overall record
        OverallWins = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "wins"}), 0),
        OverallLosses = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "losses"}), 0),
        OverallTies = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "ties"}), 0),
        OverallWinPercentage = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "percentage"}), 0),
        
        // Division record
        DivisionWins = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "division", "wins"}), 0),
        DivisionLosses = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "division", "losses"}), 0),
        DivisionTies = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "division", "ties"}), 0),
        DivisionWinPercentage = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "division", "percentage"}), 0),
        
        // Points
        PointsFor = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "pointsFor"}), 0),
        PointsAgainst = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "pointsAgainst"}), 0),
        PointsDifferential = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "pointsFor"}), 0) - Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "pointsAgainst"}), 0),
        
        // Standings position
        Standing = Lib[SafeNumber](Lib[SafeRecordField](_, "standing", 0)),
        FinalStanding = Lib[SafeNumber](Lib[SafeRecordField](_, "finalStanding", 0)),
        
        // Streak information
        StreakType = Lib[SafeText](Lib[SafeNestedRecord](_, {"record", "overall", "streakType"})),
        StreakLength = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "streakLength"}), 0),
        
        // Additional metrics
        GamesBack = Lib[SafeNumber](Lib[SafeRecordField](_, "gamesBack", 0)),
        PlayoffSeed = Lib[SafeNumber](Lib[SafeRecordField](_, "playoffSeed", 0))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedStandings),
    
    // Sort by standing
    sortedTable = Table.Sort(table, {{"Standing", Order.Ascending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "TeamId", "TeamName", "Location", "Abbrev", "DivisionId",
        "OverallWins", "OverallLosses", "OverallTies", "OverallWinPercentage",
        "DivisionWins", "DivisionLosses", "DivisionTies", "DivisionWinPercentage",
        "PointsFor", "PointsAgainst", "PointsDifferential",
        "Standing", "FinalStanding", "StreakType", "StreakLength", "GamesBack", "PlayoffSeed"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
