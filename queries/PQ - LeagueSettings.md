# League Settings Query

**Query Name:** `LeagueSettings`

Extracts league configuration, scoring settings, and basic league information.

```powerquery-m
let
    Lib = Lib_Espn,
    
    // Get league settings data using the library
    leagueData = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"mSettings"}),
    
    // Extract comprehensive league info using direct field access with error handling
    result = [
        // Basic Info
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        LeagueName = try leagueData[settings][name] otherwise "",
        SeasonId = try leagueData[seasonId] otherwise 0,
        GameId = try leagueData[gameId] otherwise 0,
        Size = try leagueData[settings][size] otherwise 0,
        IsPublic = try leagueData[settings][isPublic] otherwise false,
        
        // Current Status
        CurrentWeek = try leagueData[status][currentMatchupPeriod] otherwise 0,
        LatestScoringPeriod = try leagueData[status][latestScoringPeriod] otherwise 0,
        FinalWeek = try leagueData[status][finalScoringPeriod] otherwise 0,
        FirstScoringPeriod = try leagueData[status][firstScoringPeriod] otherwise 0,
        IsActive = try leagueData[status][isActive] otherwise false,
        IsFull = try leagueData[status][isFull] otherwise false,
        
        // Draft Info
        DraftCompleted = try leagueData[draftDetail][drafted] otherwise false,
        DraftInProgress = try leagueData[draftDetail][inProgress] otherwise false,
        DraftType = try leagueData[settings][draftSettings][type] otherwise "",
        DraftDate = try leagueData[settings][draftSettings][date] otherwise 0,
        AuctionBudget = try leagueData[settings][draftSettings][auctionBudget] otherwise 0,
        KeeperCount = try leagueData[settings][draftSettings][keeperCount] otherwise 0,
        
        // Financial Settings
        EntryFee = try leagueData[settings][financeSettings][entryFee] otherwise 0,
        MiscFee = try leagueData[settings][financeSettings][miscFee] otherwise 0,
        PerTrade = try leagueData[settings][financeSettings][perTrade] otherwise 0,
        
        // Roster Settings
        QB = try leagueData[settings][rosterSettings][lineupSlotCounts][0] otherwise 0,
        RB = try leagueData[settings][rosterSettings][lineupSlotCounts][2] otherwise 0,
        WR = try leagueData[settings][rosterSettings][lineupSlotCounts][4] otherwise 0,
        TE = try leagueData[settings][rosterSettings][lineupSlotCounts][6] otherwise 0,
        FLEX = try leagueData[settings][rosterSettings][lineupSlotCounts][16] otherwise 0,
        K = try leagueData[settings][rosterSettings][lineupSlotCounts][17] otherwise 0,
        DST = try leagueData[settings][rosterSettings][lineupSlotCounts][20] otherwise 0,
        BENCH = try leagueData[settings][rosterSettings][lineupSlotCounts][21] otherwise 0,
        
        // Schedule Settings
        RegularSeasonWeeks = try leagueData[settings][scheduleSettings][matchupPeriodCount] otherwise 0,
        PlayoffTeams = try leagueData[settings][scheduleSettings][playoffTeamCount] otherwise 0,
        PlayoffSeedingRule = try leagueData[settings][scheduleSettings][playoffSeedingRule] otherwise "",
        
        // Acquisition Settings
        WaiverHours = try leagueData[settings][acquisitionSettings][waiverHours] otherwise 0,
        AcquisitionType = try leagueData[settings][acquisitionSettings][acquisitionType] otherwise "",
        AcquisitionBudget = try leagueData[settings][acquisitionSettings][acquisitionBudget] otherwise 0,
        IsUsingAcquisitionBudget = try leagueData[settings][acquisitionSettings][isUsingAcquisitionBudget] otherwise false,
        
        // Trade Settings
        MaxTrades = try leagueData[settings][tradeSettings][max] otherwise 0,
        VetoVotesRequired = try leagueData[settings][tradeSettings][vetoVotesRequired] otherwise 0,
        TradeRevisionHours = try leagueData[settings][tradeSettings][revisionHours] otherwise 0,
        
        // Scoring Type
        ScoringType = try leagueData[settings][scoringSettings][scoringType] otherwise "",
        AllowOutOfPositionScoring = try leagueData[settings][scoringSettings][allowOutOfPositionScoring] otherwise false
    ],
    
    // Convert to table using library function
    table = Lib[TableFromListOfRecords]({result})
in
    table
```
