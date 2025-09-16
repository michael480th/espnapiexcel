# Teams Query

Extracts team information, owners, and basic team data.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Parameters[LeagueId], {"mTeam"}),
    teams = Lib[SafeNestedList](data, {"teams"}),
    
    // Process each team
    processedTeams = List.Transform(teams, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        TeamId = Lib[SafeNumber](Lib[SafeRecordField](_, "id", 0)),
        TeamName = Lib[SafeText](Lib[SafeRecordField](_, "name", "")),
        Location = Lib[SafeText](Lib[SafeRecordField](_, "location", "")),
        Abbrev = Lib[SafeText](Lib[SafeRecordField](_, "abbrev", "")),
        Logo = Lib[SafeText](Lib[SafeRecordField](_, "logo", "")),
        DivisionId = Lib[SafeNumber](Lib[SafeRecordField](_, "divisionId", 0)),
        
        // Owner information
        OwnerId = Lib[SafeText](Lib[SafeNestedRecord](_, {"owners", 0, "id"})),
        OwnerDisplayName = Lib[SafeText](Lib[SafeNestedRecord](_, {"owners", 0, "displayName"})),
        OwnerFirstName = Lib[SafeText](Lib[SafeNestedRecord](_, {"owners", 0, "firstName"})),
        OwnerLastName = Lib[SafeText](Lib[SafeNestedRecord](_, {"owners", 0, "lastName"})),
        
        // Team record
        Wins = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "wins"}), 0),
        Losses = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "losses"}), 0),
        Ties = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "ties"}), 0),
        WinPercentage = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "percentage"}), 0),
        
        // Points
        PointsFor = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "pointsFor"}), 0),
        PointsAgainst = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"record", "overall", "pointsAgainst"}), 0),
        
        // Standings
        Standing = Lib[SafeNumber](Lib[SafeRecordField](_, "standing", 0)),
        FinalStanding = Lib[SafeNumber](Lib[SafeRecordField](_, "finalStanding", 0)),
        
        // Additional team data
        WaiverPosition = Lib[SafeNumber](Lib[SafeRecordField](_, "waiverPosition", 0)),
        FAABRemaining = Lib[SafeNumber](Lib[SafeRecordField](_, "faabRemaining", 0)),
        TransactionCount = Lib[SafeNumber](Lib[SafeRecordField](_, "transactionCount", 0)),
        TradeCount = Lib[SafeNumber](Lib[SafeRecordField](_, "tradeCount", 0))
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedTeams),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "TeamId", "TeamName", "Location", "Abbrev", "Logo", "DivisionId",
        "OwnerId", "OwnerDisplayName", "OwnerFirstName", "OwnerLastName",
        "Wins", "Losses", "Ties", "WinPercentage", "PointsFor", "PointsAgainst",
        "Standing", "FinalStanding", "WaiverPosition", "FAABRemaining", "TransactionCount", "TradeCount"
    },
    finalTable = Lib[EnsureColumns](table, requiredColumns)
in
    finalTable
```
