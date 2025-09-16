# All Players Paged Query

**Query Name:** `AllPlayersPaged`

Extracts all players (both rostered and free agents) with pagination support for large datasets.

```powerquery-m
let
    Lib = Lib_Espn,
    
    // Get current scoring period from league settings
    leagueSettings = Lib[JsonGetWithView](Parameters[Season], Text.From(Parameters[LeagueId]), {"mSettings"}),
    currentScoringPeriod = try leagueSettings[status][latestScoringPeriod] otherwise 1,
    
    // Use kona_player_info with exact header format from the comment
    baseUrl = "https://lm-api-reads.fantasy.espn.com/apis/v3/games/ffl/seasons/",
    playersUrl = baseUrl & Text.From(Parameters[Season]) & "/segments/0/leagues/" & Text.From(Parameters[LeagueId]) & "?view=kona_player_info",
    
    // Exact filter string from the working curl command
    filterString = "{""players"":{""limit"":1500,""sortDraftRanks"": {""sortPriority"": 100,""sortAsc"": true,""value"": ""STANDARD""}}}",
    
    // Headers with exact format
    headers = [
        #"espn_s2" = Parameters[Cookies_espn_s2],
        #"SWID" = Parameters[Cookies_swid],
        #"x-fantasy-filter" = filterString
    ],
    options = [
        Headers = headers,
        Timeout = #duration(0, 0, 0, 30)
    ],
    
    // Make API call
    response = Web.Contents(playersUrl, options),
    jsonData = Json.Document(response),
    allPlayers = try jsonData[players] otherwise {},
    
    // Process each player using the correct nested structure
    processedPlayers = List.Transform(allPlayers, each [
        LeagueId = Parameters[LeagueId],
        Season = Parameters[Season],
        ScoringPeriodId = currentScoringPeriod,
        
        // Player information - access nested player object
        PlayerId = try _[player][id] otherwise try _[id] otherwise 0,
        PlayerName = try _[player][fullName] otherwise "",
        PlayerFirstName = try _[player][firstName] otherwise "",
        PlayerLastName = try _[player][lastName] otherwise "",
        DefaultPositionId = try _[player][defaultPositionId] otherwise 0,
        EligibleSlots = try Text.From(_[player][eligibleSlots]) otherwise "",
        
        // Player status
        IsActive = try _[player][active] otherwise false,
        IsInjured = try _[player][injured] otherwise false,
        IsDroppable = try _[player][droppable] otherwise false,
        InjuryStatus = try _[player][injuryStatus] otherwise "",
        
        // Professional team
        ProTeamId = try _[player][proTeamId] otherwise 0,
        
        // Ownership information
        PercentOwned = try _[player][ownership][percentOwned] otherwise 0,
        PercentStarted = try _[player][ownership][percentStarted] otherwise 0,
        
        // Additional player data
        Jersey = try _[player][jersey] otherwise "",
        LastNewsDate = try _[player][lastNewsDate] otherwise 0,
        LastVideoDate = try _[player][lastVideoDate] otherwise 0,
        
        // Top-level fields
        OnTeamId = try _[onTeamId] otherwise 0,
        DraftAuctionValue = try _[draftAuctionValue] otherwise 0,
        KeeperValue = try _[keeperValue] otherwise 0,
        KeeperValueFuture = try _[keeperValueFuture] otherwise 0,
        LineupLocked = try _[lineupLocked] otherwise false,
        RosterLocked = try _[rosterLocked] otherwise false,
        TradeLocked = try _[tradeLocked] otherwise false,
        Status = try _[status] otherwise "",
        
        // Draft rankings
        DraftRankStandard = try _[player][draftRanksByRankType][STANDARD][rank] otherwise 0,
        DraftRankPPR = try _[player][draftRanksByRankType][PPR][rank] otherwise 0,
        DraftAuctionValueStandard = try _[player][draftRanksByRankType][STANDARD][auctionValue] otherwise 0,
        DraftAuctionValuePPR = try _[player][draftRanksByRankType][PPR][auctionValue] otherwise 0
    ]),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedPlayers),
    
    // Sort by percent owned (most owned first)
    sortedTable = Table.Sort(table, {{"PercentOwned", Order.Descending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "ScoringPeriodId", "PlayerId", "PlayerName", "PlayerFirstName", "PlayerLastName",
        "DefaultPositionId", "EligibleSlots", "IsActive", "IsInjured", "IsDroppable", "InjuryStatus",
        "ProTeamId", "PercentOwned", "PercentStarted", "Jersey", "LastNewsDate", "LastVideoDate",
        "OnTeamId", "DraftAuctionValue", "KeeperValue", "KeeperValueFuture", "LineupLocked", "RosterLocked", 
        "TradeLocked", "Status", "DraftRankStandard", "DraftRankPPR", "DraftAuctionValueStandard", "DraftAuctionValuePPR"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
