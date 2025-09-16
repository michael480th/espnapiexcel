# ESPN Fantasy Football Parameters Setup

## Quick Setup Instructions

1. **Open Excel** and go to Data → Get Data → From Other Sources → Blank Query
2. **Paste the M code below** into the Advanced Editor
3. **Run the query** to create all parameters
4. **Configure your parameters** in Excel's Parameters pane

## Parameter Creation Code

```powerquery-m
let
    // Create Parameters for ESPN Fantasy Football
    Parameters = [
        // Core Parameters
        LeagueId = "169608",                    // Your ESPN League ID
        Season = 2025,                          // Fantasy season year
        BaseUrl = "https://lm-api-reads.fantasy.espn.com/apis/v3/games/ffl/seasons/",
        
        // Authentication Parameters
        Cookies_espn_s2 = "AEBH9dUFpFESaSZ41jrpJADdJ%2BgPtq5ve0Kd1X1ciZzCwFO1GXOLGvuBTjbak16T%2B8215edLGQSNfKvq5KrK4rTxFNA%2BFmWwntH8CBrHiFzRh3poPfiRZtWzsCX2EcnPG6Z5vwQzRa4U4iQnbNOUirY9lpkGn%2BYCiaO17v7L2bH9qRRguRFbWbdVFamY8yUhwUXVQiyA7FkqbuTy1W%2BFObPLJkZlZ0rSAMcrwYR5m9M8GQdL96lHlBQ5v6RpBh7LLBgJnWFhz%2FKZtkQCwR66S29n%2BS3SeEGTQ8%2F%2BrXzULYw1dA%3D%3D",                   // ESPN S2 cookie (for private leagues)
        Cookies_swid = "{D1E8976F-5D4C-4E75-AEAC-26ACCCFA7017}",                      // ESPN SWID cookie (for private leagues)
        UsePublicOnly = false,                  // Set to true for public leagues only
        
        // Performance Parameters
        PageSize = 200,                         // Records per page
        MaxRetries = 3,                         // API retry attempts
        TimeoutSeconds = 30                     // Request timeout
    ]
in
    Parameters
```

## Parameter Descriptions

### Required Parameters
- **LeagueId**: Your ESPN Fantasy Football League ID (found in the URL)
- **Season**: The fantasy season year (e.g., 2024)

### Optional Parameters
- **ScoringPeriodId**: Week number for weekly data (default: 1)
- **Cookies_espn_s2**: ESPN S2 authentication cookie for private leagues
- **Cookies_swid**: ESPN SWID authentication cookie for private leagues
- **UsePublicOnly**: Set to true if you only want public league data

### Performance Tuning
- **PageSize**: Number of records per API request (default: 200)
- **MaxRetries**: Number of retry attempts for failed requests (default: 3)
- **TimeoutSeconds**: Request timeout in seconds (default: 30)

## Finding Your League ID

1. Go to your ESPN Fantasy Football league
2. Look at the URL: `https://fantasy.espn.com/football/league?leagueId=123456`
3. The number after `leagueId=` is your League ID

## Getting Authentication Cookies (Private Leagues Only)

### Method 1: Browser Developer Tools
1. Open your ESPN Fantasy Football league in a web browser
2. Press F12 to open Developer Tools
3. Go to Application/Storage → Cookies
4. Find `espn_s2` and `SWID` cookies
5. Copy their values

### Method 2: Browser Extension
1. Install a cookie export extension
2. Export cookies from espn.com
3. Find `espn_s2` and `SWID` values

## Security Notes

- **Keep cookies secure** - they provide access to your private league
- **Don't share cookies** - they're tied to your ESPN account
- **Refresh cookies** if you get authentication errors
- **Use public mode** for public leagues to avoid cookie requirements

## Troubleshooting

### Common Issues
- **401 Unauthorized**: Check your cookies or try public mode
- **404 Not Found**: Verify your League ID and Season
- **Timeout Errors**: Increase TimeoutSeconds or check your internet connection
- **Empty Results**: Verify the ScoringPeriodId (week number)

### Getting Help
1. Check the API_REALITY.md documentation
2. Verify your parameters match the examples
3. Test with a public league first
4. Check Excel's Data → Queries & Connections for error details
