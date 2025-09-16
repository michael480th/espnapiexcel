# Rosters By Week Query

Extracts weekly roster compositions and player information for a specific scoring period.

```powerquery-m
let
    Lib = Lib_Espn,
    data = Lib[JsonGetWithView](Parameters[Season], Parameters[LeagueId], {"mRoster"}, [
        scoringPeriodId = Parameters[ScoringPeriodId]
    ]),
    teams = Lib[SafeNestedList](data, {"teams"}),
    
    // Process each team's roster
    processedRosters = List.Accumulate(teams, {}, (state, team) =>
        let
            teamId = Lib[SafeNumber](Lib[SafeRecordField](team, "id", 0)),
            teamName = Lib[SafeText](Lib[SafeRecordField](team, "name", "")),
            roster = Lib[SafeNestedRecord](team, {"roster", "entries"}),
            rosterList = if roster = null then {} else if Value.Type(roster) = type list then roster else {},
            
            // Process each player in the roster
            players = List.Transform(rosterList, each [
                LeagueId = Parameters[LeagueId],
                Season = Parameters[Season],
                ScoringPeriodId = Parameters[ScoringPeriodId],
                TeamId = teamId,
                TeamName = teamName,
                
                // Player information
                PlayerId = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "id"}), 0),
                PlayerName = Lib[SafeText](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "fullName"})),
                PlayerFirstName = Lib[SafeText](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "firstName"})),
                PlayerLastName = Lib[SafeText](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "lastName"})),
                DefaultPositionId = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "defaultPositionId"}), 0),
                EligibleSlots = Lib[SafeText](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "eligibleSlots"})),
                
                // Roster information
                LineupSlotId = Lib[SafeNumber](Lib[SafeRecordField](_, "lineupSlotId", 0)),
                AcquisitionDate = Lib[SafeDateTime](Lib[SafeRecordField](_, "acquisitionDate")),
                AcquisitionType = Lib[SafeText](Lib[SafeRecordField](_, "acquisitionType", "")),
                
                // Player status
                InjuryStatus = Lib[SafeText](Lib[SafeRecordField](_, "injuryStatus", "")),
                IsInjured = Lib[SafeLogical](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "injured"}), false),
                IsActive = Lib[SafeLogical](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "active"}), false),
                IsDroppable = Lib[SafeLogical](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "droppable"}), false),
                
                // Stats for the week
                AppliedStatTotal = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"playerPoolEntry", "appliedStatTotal"}), 0),
                ProjectedPoints = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"playerPoolEntry", "projectedPoints"}), 0),
                
                // Professional team
                ProTeamId = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"playerPoolEntry", "player", "proTeamId"}), 0),
                
                // Keeper information
                KeeperValue = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"playerPoolEntry", "keeperValue"}), 0),
                KeeperValueFuture = Lib[SafeNumber](Lib[SafeNestedRecord](_, {"playerPoolEntry", "keeperValueFuture"}), 0)
            ]),
            
            combinedState = List.Combine({state, players})
        in
            combinedState
    ),
    
    // Convert to table
    table = Lib[TableFromListOfRecords](processedRosters),
    
    // Sort by team ID and lineup slot
    sortedTable = Table.Sort(table, {{"TeamId", Order.Ascending}, {"LineupSlotId", Order.Ascending}}),
    
    // Ensure all columns exist
    requiredColumns = {
        "LeagueId", "Season", "ScoringPeriodId", "TeamId", "TeamName",
        "PlayerId", "PlayerName", "PlayerFirstName", "PlayerLastName", "DefaultPositionId", "EligibleSlots",
        "LineupSlotId", "AcquisitionDate", "AcquisitionType",
        "InjuryStatus", "IsInjured", "IsActive", "IsDroppable",
        "AppliedStatTotal", "ProjectedPoints", "ProTeamId", "KeeperValue", "KeeperValueFuture"
    },
    finalTable = Lib[EnsureColumns](sortedTable, requiredColumns)
in
    finalTable
```
