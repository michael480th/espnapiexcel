# ESPN Fantasy Football API Reality Report

## Base URLs and Authentication

### Base Endpoints
- **Fantasy Base**: `https://lm-api-reads.fantasy.espn.com/apis/v3/games/ffl/seasons/{year}`
- **League Endpoint**: `https://lm-api-reads.fantasy.espn.com/apis/v3/games/ffl/seasons/{year}/segments/0/leagues/{league_id}`
- **Legacy Endpoint** (pre-2018): `https://lm-api-reads.fantasy.espn.com/apis/v3/games/ffl/leagueHistory/{league_id}?seasonId={year}`
- **News Endpoint**: `https://site.api.espn.com/apis/fantasy/v3/games/ffl/news/players`

### Authentication
- **Cookies Required**: `espn_s2` and `SWID` for private leagues
- **Public Leagues**: Work without cookies (limited data access)
- **Cookie Format**: 
  - `espn_s2`: Long alphanumeric string
  - `SWID`: UUID format like `{D0C25A4C-2A0D-4E56-8E7F-20A10B663272}`

## Confirmed API Endpoints and Views

| Endpoint/View | HTTP Method | Required Params | Key Response Keys | Notes |
|---------------|-------------|------------------|-------------------|-------|
| **League Data** | GET | `view=mTeam,mRoster,mMatchup,mSettings,mStandings` | `teams`, `schedule`, `settings`, `members` | Core league data |
| **League Settings** | GET | `view=mSettings` | `settings` | Scoring, roster, playoff settings |
| **Teams** | GET | `view=mTeam` | `teams` | Team info, owners, records |
| **Standings** | GET | `view=mStandings` | `teams` (with standings data) | Current standings |
| **Schedule** | GET | `view=mMatchup` | `schedule` | Full season schedule |
| **Matchups by Week** | GET | `view=mMatchupScore`, `scoringPeriodId={week}` | `schedule` | Weekly matchup scores |
| **Box Scores** | GET | `view=mMatchupScore,mScoreboard`, `scoringPeriodId={week}` | `schedule` | Detailed player stats |
| **Rosters by Week** | GET | `view=mRoster`, `scoringPeriodId={week}` | `teams[].roster` | Weekly roster data |
| **Draft Results** | GET | `view=mDraftDetail` | `draftDetail.picks` | Draft picks and details |
| **Transactions** | GET | `view=mTransactions2`, `scoringPeriodId={week}` | `transactions` | Recent transactions |
| **Free Agents** | GET | `view=kona_player_info`, `scoringPeriodId={week}` | `players` | Available players |
| **Player Cards** | GET | `view=kona_playercard` | `players` | Detailed player stats |
| **Recent Activity** | GET | `view=kona_league_communication` | `topics` | League activity feed |
| **Pro Schedule** | GET | `view=proTeamSchedules_wl` | `settings.proTeams` | NFL team schedules |
| **All Players** | GET | `view=players_wl` | Array of player objects | Complete player database |

## Pagination and Limits

### Pagination Mechanics
- **Free Agents**: Uses `limit` parameter (default 50, max ~200)
- **Recent Activity**: Uses `limit` and `offset` parameters
- **No Standard Pagination**: Most endpoints return all data in single request
- **Filter Headers**: Complex filtering via `x-fantasy-filter` header with JSON

### Known Limits
- **Free Agents**: ~200 players per request
- **Recent Activity**: 25 messages per request
- **Player Cards**: Limited by player ID list size
- **No Rate Limiting**: No evidence of API rate limits in code

## Query Parameters and Filters

### Standard Parameters
- `view`: Comma-separated list of data views
- `scoringPeriodId`: Week number (1-18 typically)
- `seasonId`: Year (for legacy endpoints)

### Advanced Filtering
- **Header**: `x-fantasy-filter` with JSON payload
- **Free Agents**: Filter by position, status, ownership percentage
- **Transactions**: Filter by transaction type
- **Activity**: Filter by message type, date range

## Response Structure Patterns

### Top-Level Keys
- `teams`: Array of team objects
- `schedule`: Array of matchup objects  
- `settings`: League configuration
- `members`: League member information
- `draftDetail`: Draft information
- `transactions`: Transaction history
- `players`: Player data arrays

### Data Drift Issues
- **List vs Record**: Some fields switch between array and object formats
- **Week-Dependent**: Player stats change structure by scoring period
- **Season-Dependent**: Pre-2018 vs post-2018 data structure differences
- **Null Safety**: Many fields can be null or missing

## Error Handling

### Status Codes
- **200**: Success
- **401**: Access denied (try alternate endpoint)
- **404**: League not found
- **Other**: Unknown error

### Endpoint Fallback
- Automatic switching between `/seasons/` and `/leagueHistory/` endpoints
- Retry logic for 401 errors with alternate endpoint

## Source Files Referenced
- `espn_api/requests/espn_requests.py` - Core API client
- `espn_api/requests/constant.py` - Base URLs and sport mappings
- `espn_api/football/league.py` - Football-specific API methods
- `espn_api/football/constant.py` - Position mappings and stat definitions
- `espn_api/base_league.py` - Base league functionality
- `tests/football/unit/test_league.py` - API usage examples
- `tests/football/unit/data/` - Sample API responses

## Security Notes
- **Public Leagues**: No authentication required, limited data access
- **Private Leagues**: Require valid `espn_s2` and `SWID` cookies
- **Cookie Security**: Cookies should be treated as sensitive credentials
- **No API Keys**: ESPN uses cookie-based authentication only
