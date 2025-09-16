# ESPN Fantasy Football Power Query Tool - Requirements

## Objectives

### Primary Goals
- Create a comprehensive Power Query (M) library for ESPN Fantasy Football data extraction
- Provide Excel users with easy access to fantasy football data without programming knowledge
- Support both public and private league access with proper authentication
- Enable data refresh and analysis workflows in Excel

### Non-Objectives
- Real-time data streaming (refresh-based only)
- Data modification/write operations (read-only)
- Advanced analytics beyond basic data extraction
- Support for other fantasy sports (football only)

## Parameters

### Core Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `LeagueId` | Text | Required | ESPN Fantasy Football League ID |
| `Season` | Number | Required | Fantasy season year (e.g., 2024) |
| `ScoringPeriodId` | Number | 1 | Week number (1-18) |
| `BaseUrl` | Text | `https://lm-api-reads.fantasy.espn.com/apis/v3/games/ffl/seasons/` | ESPN API base URL |

### Authentication Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Cookies_espn_s2` | Text | "" | ESPN S2 authentication cookie |
| `Cookies_swid` | Text | "" | ESPN SWID authentication cookie |
| `UsePublicOnly` | Logical | false | Restrict to public league data only |

### Performance Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `PageSize` | Number | 200 | Records per page for paginated endpoints |
| `MaxRetries` | Number | 3 | API request retry attempts |
| `TimeoutSeconds` | Number | 30 | Request timeout in seconds |

## Data Domains

### Confirmed Data Tables
| Table Name | Primary Key | Dependencies | Description |
|------------|-------------|--------------|-------------|
| `LeagueSettings` | LeagueId, Season | None | League configuration and scoring |
| `Teams` | LeagueId, Season, TeamId | LeagueSettings | Team information and owners |
| `Standings` | LeagueId, Season, TeamId | Teams | Current standings and records |
| `Schedule` | LeagueId, Season, MatchupId | Teams | Full season schedule |
| `MatchupsByWeek` | LeagueId, Season, ScoringPeriodId, MatchupId | Teams, Schedule | Weekly matchup results |
| `BoxscoresByWeek` | LeagueId, Season, ScoringPeriodId, MatchupId | Teams, Schedule | Detailed player statistics |
| `RostersByWeek` | LeagueId, Season, ScoringPeriodId, TeamId | Teams | Weekly roster compositions |
| `Transactions` | LeagueId, Season, TransactionId | Teams | Transaction history |
| `DraftResults` | LeagueId, Season, PickId | Teams | Draft picks and details |
| `FreeAgentsPaged` | LeagueId, Season, ScoringPeriodId, PlayerId | None | Available free agents |
| `PlayersDirectory` | PlayerId | None | Complete player database |
| `RecentActivity` | LeagueId, Season, ActivityId | Teams | League activity feed |
| `ProSchedule` | Season, ProTeamId | None | NFL team schedules |

### Advanced Optional Tables
| Table Name | Primary Key | Dependencies | Description |
|------------|-------------|--------------|-------------|
| `WaiversAndFAAB` | LeagueId, Season, ScoringPeriodId | Teams, Transactions | Waiver wire and FAAB data |
| `InjuriesAndNews` | PlayerId, Date | PlayersDirectory | Player injury and news updates |
| `PlayerCards` | LeagueId, Season, PlayerId | PlayersDirectory | Detailed player statistics |

## Refresh Strategy

### Refresh Order
1. **LeagueSettings** (foundation data)
2. **Teams** (depends on LeagueSettings)
3. **Standings** (depends on Teams)
4. **Schedule** (depends on Teams)
5. **ProSchedule** (independent)
6. **PlayersDirectory** (independent)
7. **Weekly Data** (ScoringPeriodId dependent):
   - MatchupsByWeek
   - BoxscoresByWeek
   - RostersByWeek
8. **Transaction Data**:
   - Transactions
   - RecentActivity
9. **Draft Data**:
   - DraftResults
10. **Free Agent Data**:
    - FreeAgentsPaged

### Paging Strategy
- **Sequential Pages**: Use offset-based pagination for large datasets
- **Error Handling**: Retry failed requests with exponential backoff
- **Rate Limiting**: Implement delays between requests to avoid overwhelming API
- **Incremental Refresh**: Only fetch new/changed data where possible

## Error Handling

### Null-Safe Access
- **Record Fields**: Use `Record.FieldOrDefault()` for optional fields
- **List Fields**: Use `List.Count()` checks before accessing elements
- **Nested Objects**: Implement recursive null checking
- **Type Coercion**: Convert unknown types to text with fallbacks

### Stable Schemas
- **Column Definitions**: Explicit column lists for all tables
- **Data Types**: Consistent type mapping (text, number, datetime, logical)
- **Missing Data**: Use appropriate defaults (empty strings, 0, false, null)
- **Schema Evolution**: Handle API changes gracefully with version detection

### Error Recovery
- **Network Errors**: Retry with exponential backoff
- **Authentication Errors**: Clear error messages for invalid cookies
- **Data Errors**: Log errors but continue processing other data
- **Schema Errors**: Fall back to text representation for unknown fields

## Security Considerations

### Private vs Public Leagues
- **Public Leagues**: No authentication required, limited data access
- **Private Leagues**: Require valid `espn_s2` and `SWID` cookies
- **Cookie Security**: Treat cookies as sensitive credentials
- **Data Sensitivity**: Some league data may be private/sensitive

### Cookie Management
- **Storage**: Store cookies securely in Excel parameters
- **Validation**: Verify cookie format before use
- **Expiration**: Handle expired cookies gracefully
- **Privacy**: Never log or expose cookie values

### Data Privacy
- **Personal Information**: Handle owner names and contact info carefully
- **League Settings**: Some settings may be private league-specific
- **Transaction History**: May contain sensitive trade/waiver information
- **Compliance**: Ensure compliance with data privacy regulations

## Implementation Notes

### Power Query Limitations
- **No Custom Headers**: Limited ability to set custom HTTP headers
- **JSON Parsing**: Complex nested JSON requires careful handling
- **Error Messages**: Provide clear, actionable error messages
- **Performance**: Large datasets may require optimization

### Excel Integration
- **Parameter Management**: Use Excel's Parameters feature for configuration
- **Refresh Scheduling**: Leverage Excel's data refresh capabilities
- **Data Model**: Structure data for optimal Excel analysis
- **User Experience**: Provide clear instructions and error handling
