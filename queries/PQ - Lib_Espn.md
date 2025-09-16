let
    // =============================================================================
    // ESPN Fantasy Football Power Query Library
    // =============================================================================
    
    // Authentication Headers
    AuthHeaders = () =>
        let
            espn_s2 = Parameters[Cookies_espn_s2],
            swid = Parameters[Cookies_swid],
            headers = [
                #"espn_s2" = espn_s2,
                #"SWID" = swid
            ],
            // Only include cookies if they're not empty
            filteredHeaders = if espn_s2 <> "" and swid <> "" then headers else []
        in
            filteredHeaders,

    // Build URL with parameters
    BuildUrl = (year as number, league as text, views as list, optional qs as record) =>
        let
            baseUrl = Parameters[BaseUrl],
            leagueEndpoint = baseUrl & Text.From(year) & "/segments/0/leagues/" & league,
            viewParam = "?view=" & Text.Combine(views, ","),
            queryString = if qs = null then "" else 
                let
                    qsList = Record.ToList(qs),
                    qsKeys = Record.FieldNames(qs),
                    qsPairs = List.Transform(List.Zip({qsKeys, qsList}), each Text.From(_{0}) & "=" & Text.From(_{1})),
                    qsText = Text.Combine(qsPairs, "&")
                in
                    if qsText = "" then "" else "&" & qsText,
            fullUrl = leagueEndpoint & viewParam & queryString
        in
            fullUrl,

    // Generic JSON GET request
    JsonGet = (url as text) =>
        let
            headers = AuthHeaders(),
            options = [
                Headers = headers,
                Timeout = #duration(0, 0, 0, Parameters[TimeoutSeconds])
            ],
            response = Web.Contents(url, options),
            json = Json.Document(response)
        in
            json,

    // GET request with view parameters
    JsonGetWithView = (year as number, league as text, views as list, optional qs as record) =>
        let
            url = BuildUrl(year, league, views, qs),
            result = JsonGet(url)
        in
            result,

    // POST request with JSON body (if needed for advanced endpoints)
    PostJson = (url as text, body as record) =>
        let
            headers = AuthHeaders(),
            jsonBody = Json.FromValue(body),
            options = [
                Headers = headers,
                Content = jsonBody,
                Timeout = #duration(0, 0, 0, Parameters[TimeoutSeconds])
            ],
            response = Web.Contents(url, options),
            json = Json.Document(response)
        in
            json,

    // Convert list of records to table
    TableFromListOfRecords = (list as list) =>
        let
            table = if List.Count(list) = 0 then 
                #table({}, {}) 
            else 
                Table.FromRecords(list)
        in
            table,

    // Ensure table has required columns with defaults
    EnsureColumns = (table as table, cols as list) =>
        let
            existingCols = Table.ColumnNames(table),
            missingCols = List.Difference(cols, existingCols),
            addMissingCols = if List.Count(missingCols) = 0 then 
                table 
            else 
                let
                    colDefaults = List.Transform(missingCols, each {_, type text}),
                    addCols = Table.AddColumn(table, missingCols{0}, each null, type text)
                in
                    if List.Count(missingCols) = 1 then 
                        addCols 
                    else 
                        List.Accumulate(
                            List.Skip(missingCols), 
                            addCols, 
                            (state, col) => Table.AddColumn(state, col, each null, type text)
                        )
        in
            addMissingCols,

    // Pagination helper
    Paginate = (offset as number, pageSize as number, makeUrl as function, extract as function) =>
        let
            url = makeUrl(offset, pageSize),
            data = JsonGet(url),
            extracted = extract(data),
            hasMore = if Record.HasFields(data, "paging") then 
                Record.FieldOrDefault(data, "paging", [])[hasMore] 
            else 
                List.Count(extracted) = pageSize,
            nextPage = if hasMore then 
                Paginate(offset + pageSize, pageSize, makeUrl, extract) 
            else 
                {},
            allData = List.Combine({extracted, nextPage})
        in
            allData,

    // Safe record field access
    SafeRecordField = (record as record, field as text, default as any) =>
        if Record.HasFields(record, {field}) then 
            Record.Field(record, field) 
        else 
            default,

    // Safe list access
    SafeListAccess = (list as list, index as number, default as any) =>
        if List.Count(list) > index then 
            list{index} 
        else 
            default,

    // Convert to text with null handling
    SafeText = (value as any) =>
        if value = null then 
            "" 
        else 
            Text.From(value),

    // Convert to number with null handling
    SafeNumber = (value as any) =>
        if value = null then 
            0 
        else 
            try Number.From(value) otherwise 0,

    // Convert to logical with null handling
    SafeLogical = (value as any) =>
        if value = null then 
            false 
        else 
            try Logical.From(value) otherwise false,

    // Convert to datetime with null handling
    SafeDateTime = (value as any) =>
        if value = null then 
            null 
        else 
            try DateTime.From(value) otherwise null,

    // Extract nested record safely
    SafeNestedRecord = (record as record, path as list) =>
        let
            result = List.Accumulate(
                path, 
                record, 
                (state, field) => 
                    if state = null then 
                        null 
                    else if Record.HasFields(state, {field}) then 
                        Record.Field(state, field) 
                    else 
                        null
            )
        in
            result,

    // Extract nested list safely
    SafeNestedList = (record as record, path as list) =>
        let
            nested = SafeNestedRecord(record, path),
            result = if nested = null then {} else if Value.Type(nested) = type list then nested else {}
        in
            result,

    // Build filter header for advanced queries
    BuildFilterHeader = (filter as record) =>
        let
            jsonFilter = Json.FromValue(filter),
            headers = [
                #"x-fantasy-filter" = jsonFilter
            ]
        in
            headers,

    // Advanced GET with filters
    JsonGetWithFilters = (url as text, filters as record) =>
        let
            authHeaders = AuthHeaders(),
            filterHeaders = BuildFilterHeader(filters),
            allHeaders = List.Combine({authHeaders, filterHeaders}),
            options = [
                Headers = allHeaders,
                Timeout = #duration(0, 0, 0, Parameters[TimeoutSeconds])
            ],
            response = Web.Contents(url, options),
            json = Json.Document(response)
        in
            json,

    // Retry mechanism for failed requests
    RetryRequest = (request as function, maxRetries as number) =>
        let
            result = try request() otherwise null,
            retry = if result[HasError] and maxRetries > 0 then 
                RetryRequest(request, maxRetries - 1) 
            else 
                result
        in
            retry,

    // Main library export
    Lib_Espn = [
        AuthHeaders = AuthHeaders,
        BuildUrl = BuildUrl,
        JsonGet = JsonGet,
        JsonGetWithView = JsonGetWithView,
        PostJson = PostJson,
        TableFromListOfRecords = TableFromListOfRecords,
        EnsureColumns = EnsureColumns,
        Paginate = Paginate,
        SafeRecordField = SafeRecordField,
        SafeListAccess = SafeListAccess,
        SafeText = SafeText,
        SafeNumber = SafeNumber,
        SafeLogical = SafeLogical,
        SafeDateTime = SafeDateTime,
        SafeNestedRecord = SafeNestedRecord,
        SafeNestedList = SafeNestedList,
        BuildFilterHeader = BuildFilterHeader,
        JsonGetWithFilters = JsonGetWithFilters,
        RetryRequest = RetryRequest
    ]

in
    Lib_Espn
