# DAX measures

127 measures in the `# Measures` table. Generated from the model.

### Sel Indicator
```dax
SELECTEDVALUE ( DimIndicator[indicator_id], "Y1" )
```

### Sel Sex
```dax
SELECTEDVALUE ( DimSex[Sex], "All" )
```

### Sel Indicator Name
```dax
VAR i = [Sel Indicator] RETURN CALCULATE ( MAX ( DimIndicator[Indicator] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i )
```

### Direction
```dax
VAR i = [Sel Indicator] RETURN CALCULATE ( MAX ( DimIndicator[need_direction] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i )
```

### Rate
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### Rate Female
```dax
VAR i = [Sel Indicator]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = "Female" )
```

### Rate Male
```dax
VAR i = [Sel Indicator]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = "Male" )
```

### People
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[est_count] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### People Female
```dax
VAR i = [Sel Indicator]
RETURN CALCULATE ( SUM ( FactDistrict[est_count] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = "Female" )
```

### Need
```dax
[Rate] * [Direction]
```

### Basis
```dax
SELECTEDVALUE ( 'Actuals-vs-PctChange'[Top-Bottom-Field], "Rate" )
```

### Rank Metric
```dax
IF ( [Basis] = "Number affected" && [Direction] = 1, [People], [Need] )
```

### National Rate
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactArea[value] ), FactArea[geo_level] = "National", REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### National People
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactArea[est_count] ), FactArea[geo_level] = "National", REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### Province Rate
```dax
VAR p = SELECTEDVALUE ( DimDistrict[Province] )
RETURN IF ( ISBLANK ( p ), [National Rate], VAR i = [Sel Indicator] VAR s = [Sel Sex] RETURN CALCULATE ( SUM ( FactArea[value] ), FactArea[geo_level] = "Province", FactArea[geo_key] = p, REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s ) )
```

### Rank
```dax
IF ( HASONEVALUE ( DimDistrict[District] ) && NOT ISBLANK ( [Rate] ), RANKX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Rank Metric], , DESC ) )
```

### Is Separator
```dax
VAR N = 'Top-Bottom-N'[Top-Bottom-N Value]
VAR r = [Rank]
VAR tot = COUNTROWS ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ) )
RETURN NOT ISBLANK ( r ) && ( r IN { N + 1, N + 2 } ) && ( r <= tot - N )
```

### Rank Filtering Dynamics
```dax
VAR N = 'Top-Bottom-N'[Top-Bottom-N Value]
VAR r = [Rank]
VAR tot = COUNTROWS ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ) )
RETURN IF ( ISBLANK ( r ), 0, IF ( ( r <= N ) || ( r > tot - N ) || [Is Separator], 1, 0 ) )
```

### Province Label
```dax
IF ( [Is Separator], "⋮", SWITCH ( SELECTEDVALUE ( DimDistrict[Province] ), "Khyber Pakhtunkhwa", "KP", SELECTEDVALUE ( DimDistrict[Province] ) ) )
```

### Table Value
```dax
IF ( [Basis] = "Number affected", FORMAT ( [People], "#,0" ), FORMAT ( [Rate], "0.0" ) )
```

### Table vs Pakistan
```dax
VAR d = [Rate] - [National Rate] RETURN IF ( ISBLANK ( [Rate] ), BLANK (), FORMAT ( d, "+0.0;-0.0;0.0" ) )
```

### Table Bar
```dax
IF ( [Is Separator], BLANK (), ( [National Rate] - [Rate] ) * [Direction] )
```

### CF Table
```dax
IF ( [Is Separator], "#FFFFFF", "#252423" )
```

### Rank CF Value
```dax
IF ( [Is Separator], "#FFFFFF", "#252423" )
```

### Rank CF vs Pakistan
```dax
IF ( [Is Separator], "#FFFFFF", IF ( ( [Rate] - [National Rate] ) * [Direction] > 0, "#DC143C", "#1E8C4E" ) )
```

### dummy
```dax
" "
```

### Districts Worse
```dax
VAR nat = [National Rate] * [Direction]
RETURN COUNTROWS ( FILTER ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Need] > nat ) ) + 0
```

### Pct Districts Worse
```dax
DIVIDE ( [Districts Worse], COUNTROWS ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ) ) )
```

### Card1 Detail1
```dax
REPT ( " ", 2 ) & "of " & COUNTROWS ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ) ) & " districts"
```

### Card1 Detail2
```dax
REPT ( " ", 2 ) & [Sel Indicator Name]
```

### SVG Card 1
```dax
VAR v = MIN ( MAX ( [Pct Districts Worse], 0 ), 1 )
VAR mk = MIN ( MAX ( 0.5, 0 ), 1 )
VAR lo = IF ( [Pct Districts Worse] > 0.5, "#F7C7D0", "#C6E3FF" )
VAR hi = IF ( [Pct Districts Worse] > 0.5, "#DC143C", "#118DFF" )
RETURN "data:image/svg+xml;utf8, <svg width='300' height='30' xmlns='http://www.w3.org/2000/svg'><linearGradient id='g' x1='0%' y1='0%' x2='100%' y2='0%'><stop offset='0%' style='stop-color:" & lo & "'/><stop offset='100%' style='stop-color:" & hi & "'/></linearGradient><rect x='0' y='7.5' width='260' height='15' rx='7' ry='7' fill='#e3e3e3'/><rect x='0' y='7.5' width='" & FORMAT ( v * 260, "0" ) & "' height='15' rx='7' ry='7' fill='url(#g)'/><line x1='" & FORMAT ( mk * 260, "0" ) & "' y1='2.5' x2='" & FORMAT ( mk * 260, "0" ) & "' y2='27.5' stroke='black' stroke-dasharray='4,3'/></svg>"
```

### Avg District Rate
```dax
AVERAGEX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Rate] )
```

### Card2 Detail1
```dax
VAR d = ( [Avg District Rate] - [National Rate] ) * [Direction]
RETURN REPT ( " ", 2 ) & IF ( d > 0, "▲ worse by ", "▼ better by " ) & FORMAT ( ABS ( d ), "0.0" ) & " pts"
```

### SVG Card 2
```dax
VAR v = MIN ( MAX ( [Avg District Rate] / 100, 0 ), 1 )
VAR mk = MIN ( MAX ( [National Rate] / 100, 0 ), 1 )
VAR lo = IF ( ( [Avg District Rate] - [National Rate] ) * [Direction] > 0, "#F7C7D0", "#C6E3FF" )
VAR hi = IF ( ( [Avg District Rate] - [National Rate] ) * [Direction] > 0, "#DC143C", "#118DFF" )
RETURN "data:image/svg+xml;utf8, <svg width='300' height='30' xmlns='http://www.w3.org/2000/svg'><linearGradient id='g' x1='0%' y1='0%' x2='100%' y2='0%'><stop offset='0%' style='stop-color:" & lo & "'/><stop offset='100%' style='stop-color:" & hi & "'/></linearGradient><rect x='0' y='7.5' width='260' height='15' rx='7' ry='7' fill='#e3e3e3'/><rect x='0' y='7.5' width='" & FORMAT ( v * 260, "0" ) & "' height='15' rx='7' ry='7' fill='url(#g)'/><line x1='" & FORMAT ( mk * 260, "0" ) & "' y1='2.5' x2='" & FORMAT ( mk * 260, "0" ) & "' y2='27.5' stroke='black' stroke-dasharray='4,3'/></svg>"
```

### Worst District Rate
```dax
MAXX ( TOPN ( 1, FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Need], DESC ), [Rate] )
```

### Worst District Name
```dax
CONCATENATEX ( TOPN ( 1, FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Need], DESC ), DimDistrict[District], ", " )
```

### Best District Rate
```dax
MAXX ( TOPN ( 1, FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Need], ASC ), [Rate] )
```

### Best District Name
```dax
CONCATENATEX ( TOPN ( 1, FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Need], ASC ), DimDistrict[District], ", " )
```

### Card3 Detail1
```dax
VAR d = [Worst District Rate] - [National Rate] RETURN REPT ( " ", 2 ) & FORMAT ( d, "+0.0;-0.0" ) & " pts vs Pakistan"
```

### Card3 Detail2
```dax
REPT ( " ", 2 ) & [Best District Name]
```

### SVG Card 3
```dax
VAR v = MIN ( MAX ( [Worst District Rate] / 100, 0 ), 1 )
VAR mk = MIN ( MAX ( [National Rate] / 100, 0 ), 1 )
VAR lo = IF ( TRUE (), "#F7C7D0", "#C6E3FF" )
VAR hi = IF ( TRUE (), "#DC143C", "#118DFF" )
RETURN "data:image/svg+xml;utf8, <svg width='300' height='30' xmlns='http://www.w3.org/2000/svg'><linearGradient id='g' x1='0%' y1='0%' x2='100%' y2='0%'><stop offset='0%' style='stop-color:" & lo & "'/><stop offset='100%' style='stop-color:" & hi & "'/></linearGradient><rect x='0' y='7.5' width='260' height='15' rx='7' ry='7' fill='#e3e3e3'/><rect x='0' y='7.5' width='" & FORMAT ( v * 260, "0" ) & "' height='15' rx='7' ry='7' fill='url(#g)'/><line x1='" & FORMAT ( mk * 260, "0" ) & "' y1='2.5' x2='" & FORMAT ( mk * 260, "0" ) & "' y2='27.5' stroke='black' stroke-dasharray='4,3'/></svg>"
```

### People Affected
```dax
SUMX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [People] )
```

### Share Of National
```dax
DIVIDE ( [People Affected], [National People] )
```

### People Affected Girls
```dax
SUMX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [People Female] )
```

### Card4 Detail2
```dax
REPT ( " ", 2 ) & FORMAT ( DIVIDE ( [People Affected Girls], SUMX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), CALCULATE ( [People], REMOVEFILTERS ( DimSex ), DimSex[Sex] = "All" ) ) ), "0%" ) & " of the total"
```

### SVG Card 4
```dax
VAR v = MIN ( MAX ( [Share Of National], 0 ), 1 )
VAR mk = MIN ( MAX ( 0, 0 ), 1 )
VAR lo = IF ( FALSE (), "#F7C7D0", "#C6E3FF" )
VAR hi = IF ( FALSE (), "#DC143C", "#118DFF" )
RETURN "data:image/svg+xml;utf8, <svg width='300' height='30' xmlns='http://www.w3.org/2000/svg'><linearGradient id='g' x1='0%' y1='0%' x2='100%' y2='0%'><stop offset='0%' style='stop-color:" & lo & "'/><stop offset='100%' style='stop-color:" & hi & "'/></linearGradient><rect x='0' y='7.5' width='260' height='15' rx='7' ry='7' fill='#e3e3e3'/><rect x='0' y='7.5' width='" & FORMAT ( v * 260, "0" ) & "' height='15' rx='7' ry='7' fill='url(#g)'/><line x1='" & FORMAT ( mk * 260, "0" ) & "' y1='2.5' x2='" & FORMAT ( mk * 260, "0" ) & "' y2='27.5' stroke='black' stroke-dasharray='4,3'/></svg>"
```

### Jitter X
```dax
VAR nm = SELECTEDVALUE ( DimDistrict[District] ) RETURN IF ( ISBLANK ( nm ), BLANK (), MOD ( UNICODE ( nm ) * 7 + UNICODE ( RIGHT ( nm, 1 ) ) * 13 + UNICODE ( MID ( nm, 2, 1 ) ) * 3 + LEN ( nm ) * 31, 97 ) / 97 )
```

### Map Lon
```dax
IF ( NOT ISBLANK ( [Rate] ), AVERAGE ( DimDistrict[Longitude] ) )
```

### Map Lat
```dax
IF ( NOT ISBLANK ( [Rate] ), AVERAGE ( DimDistrict[Latitude] ) )
```

### Map X Min
```dax
60.588
```

### Map X Max
```dax
75.664
```

### Map Y Min
```dax
23.035
```

### Map Y Max
```dax
37.566
```

### Axis Min
```dax
VAR v = MINX ( ALL ( DimDistrict[District] ), [Rate] ) RETURN FLOOR ( v / 10, 1 ) * 10
```

### Axis Max
```dax
VAR v = MAXX ( ALL ( DimDistrict[District] ), [Rate] ) RETURN CEILING ( v / 10, 1 ) * 10
```

### Unit Suffix
```dax
VAR i = [Sel Indicator] RETURN IF ( CALCULATE ( MAX ( DimIndicator[unit] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i ) = "pp", " pts", "%" )
```

### Reliability
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[reliability] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### Is Caution
```dax
[Reliability] = "Use with caution"
```

### CI Low
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[ci_lo] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### CI High
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[ci_hi] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### Sample n
```dax
VAR i = [Sel Indicator]
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[n] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = i, REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### Province Short
```dax
SWITCH ( SELECTEDVALUE ( DimDistrict[Province] ), "Khyber Pakhtunkhwa", "KP", SELECTEDVALUE ( DimDistrict[Province] ) )
```

### Headline
```dax
VAR p = SELECTEDVALUE ( DimDistrict[Province] )
VAR area = IF ( ISBLANK ( p ), "Pakistan", SWITCH ( p, "Khyber Pakhtunkhwa", "KP", p ) )
VAR who = SWITCH ( [Sel Sex], "Female", " (girls and young women)", "Male", " (boys and young men)", "" )
VAR n = COUNTROWS ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ) )
RETURN [Sel Indicator Name] & who & ":  " & area & " " & FORMAT ( [Province Rate], "0.0" ) & [Unit Suffix]
    & "   ·   highest need: " & [Worst District Name] & " " & FORMAT ( [Worst District Rate], "0.0" ) & [Unit Suffix]
    & "   ·   " & [Districts Worse] & " of " & n & " districts worse than the national rate"
```

### Card2 Compare
```dax
VAR p = SELECTEDVALUE ( DimDistrict[Province] )
RETURN IF ( ISBLANK ( p ), "median district " & FORMAT ( MEDIANX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Rate] ), "0.0" ), SWITCH ( p, "Khyber Pakhtunkhwa", "KP", p ) & " average " & FORMAT ( [Province Rate], "0.0" ) )
```

### Card2 Detail2
```dax
VAR p = SELECTEDVALUE ( DimDistrict[Province] )
VAR ref = IF ( ISBLANK ( p ), MEDIANX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Rate] ) ), [Rate] ), [Province Rate] )
VAR d = ( [Avg District Rate] - ref ) * [Direction]
RETURN REPT ( " ", 2 ) & IF ( d > 0, "▲ ", "▼ " ) & FORMAT ( ABS ( d ), "0.0" ) & " pts"
```

### Card4 Coverage
```dax
IF ( ISBLANK ( SELECTEDVALUE ( DimDistrict[Province] ) ), "all of Pakistan", FORMAT ( [Share Of National], "0%" ) & " of Pakistan's total" )
```

### Card4 Detail1
```dax
" "
```

### Table CI
```dax
IF ( [Is Separator] || ISBLANK ( [Rate] ), BLANK (), FORMAT ( [CI Low], "0.0" ) & " – " & FORMAT ( [CI High], "0.0" ) )
```

### Rank CF CI
```dax
IF ( [Is Separator], "#FFFFFF", IF ( [Is Caution], "#C77700", "#8A8886" ) )
```

### Map Colour
```dax
VAR lo = MINX ( ALL ( DimDistrict[District] ), [Need] )
VAR hi = MAXX ( ALL ( DimDistrict[District] ), [Need] )
VAR t = DIVIDE ( [Need] - lo, hi - lo )
VAR r = ROUND ( 156 + t * ( 220 - 156 ), 0 )
VAR g = ROUND ( 203 + t * ( 20 - 203 ), 0 )
VAR b = ROUND ( 245 + t * ( 60 - 245 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( [Is Caution], "#C3C8CE",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### Province Colour
```dax
VAR c = [Is Caution]
RETURN SWITCH ( SELECTEDVALUE ( DimDistrict[Province] ),
    "Punjab", IF ( c, "#A9D4FF", "#118DFF" ),
    "Khyber Pakhtunkhwa", IF ( c, "#F2A5B5", "#DC143C" ),
    "Sindh", IF ( c, "#F5C2A8", "#E66C37" ),
    "Balochistan", IF ( c, "#C9A3CF", "#6B007B" ),
    "#605E5C" )
```

### TT Title
```dax
SELECTEDVALUE ( DimDistrict[District] ) & ", " & SWITCH ( SELECTEDVALUE ( DimDistrict[Province] ), "Khyber Pakhtunkhwa", "KP", SELECTEDVALUE ( DimDistrict[Province] ) )
```

### TT Value Line
```dax
IF ( ISBLANK ( [Rate] ), "No estimate (sample too small)", FORMAT ( [Rate], "0.0" ) & [Unit Suffix] & "   " & [Sel Indicator Name] )
```

### TT CI Line
```dax
IF ( ISBLANK ( [Rate] ), " ", "95% CI " & FORMAT ( [CI Low], "0.0" ) & "–" & FORMAT ( [CI High], "0.0" ) & "   ·   n = " & FORMAT ( [Sample n], "#,0" ) & "   ·   " & [Reliability] )
```

### TT Priority Line
```dax
"Worst quarter nationally on " & MAX ( DimDistrict[Priority Count] ) & " of 10 indicators   ·   overall need rank " & MAX ( DimDistrict[Priority Rank] ) & " of 131"
```

### TT District
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), TREATAS ( VALUES ( 'Indicator List'[indicator_id] ), DimIndicator[indicator_id] ), REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### TT Province
```dax
VAR s = [Sel Sex]
VAR p = SELECTEDVALUE ( DimDistrict[Province] )
RETURN CALCULATE ( SUM ( FactArea[value] ), FactArea[geo_level] = "Province", FactArea[geo_key] = p, REMOVEFILTERS ( DimIndicator ), TREATAS ( VALUES ( 'Indicator List'[indicator_id] ), DimIndicator[indicator_id] ), REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### TT Pakistan
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactArea[value] ), FactArea[geo_level] = "National", REMOVEFILTERS ( DimIndicator ), TREATAS ( VALUES ( 'Indicator List'[indicator_id] ), DimIndicator[indicator_id] ), REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### TT CF
```dax
VAR dir = MAX ( 'Indicator List'[need_direction] )
VAR d = ( [TT District] - [TT Pakistan] ) * dir
RETURN IF ( ISBLANK ( [TT District] ), "#8A8886", IF ( d > 0, "#DC143C", "#1E8C4E" ) )
```

### Priority
```dax
MAX ( DimDistrict[Priority Count] )
```

### Gender Gap
```dax
IF ( NOT ISBLANK ( [Rate Female] ) && NOT ISBLANK ( [Rate Male] ), [Rate Female] - [Rate Male] )
```

### Gap Top 15
```dax
VAR g = [Gender Gap]
RETURN IF ( NOT ISBLANK ( g ) && RANKX ( FILTER ( ALL ( DimDistrict[District] ), NOT ISBLANK ( [Gender Gap] ) ), ABS ( [Gender Gap] ), , DESC ) <= 15, g )
```

### Gap Colour
```dax
IF ( [Gender Gap] > 0, "#B03A6A", "#2E5E8C" )
```

### Priority Districts
```dax
COUNTROWS ( FILTER ( VALUES ( DimDistrict[District] ), CALCULATE ( MAX ( DimDistrict[Priority Count] ) ) >= 4 ) ) + 0
```

### Province Bar Colour
```dax
SWITCH ( SELECTEDVALUE ( DimDistrict[Province] ), "Punjab", "#118DFF", "Khyber Pakhtunkhwa", "#DC143C", "Sindh", "#E66C37", "Balochistan", "#6B007B", "#605E5C" )
```

### Gap Title
```dax
"Largest girls–boys gaps:  " & [Sel Indicator Name] & "  (points; pink = girls higher)"
```

### Page2 Headline
```dax
VAR n = COUNTROWS ( FILTER ( VALUES ( DimDistrict[District] ), NOT ISBLANK ( CALCULATE ( MAX ( DimDistrict[Priority Count] ) ) ) ) )
RETURN [Priority Districts] & " of " & n & " districts are in the worst quarter nationally on 4 or more of the 10 indicators.   Sorted by number of priority indicators."
```

### P C1
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "C1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P C1 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "C1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P C1 Heat
```dax
VAR t = [P C1 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P C1 Font
```dax
IF ( [P C1 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P C2
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "C2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P C2 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "C2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P C2 Heat
```dax
VAR t = [P C2 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P C2 Font
```dax
IF ( [P C2 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P L1
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "L1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P L1 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "L1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P L1 Heat
```dax
VAR t = [P L1 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P L1 Font
```dax
IF ( [P L1 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P L2
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "L2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P L2 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "L2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P L2 Heat
```dax
VAR t = [P L2 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P L2 Font
```dax
IF ( [P L2 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P L3
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "L3", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P L3 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "L3", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P L3 Heat
```dax
VAR t = [P L3 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P L3 Font
```dax
IF ( [P L3 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P Y1
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "Y1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P Y1 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "Y1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P Y1 Heat
```dax
VAR t = [P Y1 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P Y1 Font
```dax
IF ( [P Y1 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P Y2
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "Y2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P Y2 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "Y2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P Y2 Heat
```dax
VAR t = [P Y2 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P Y2 Font
```dax
IF ( [P Y2 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P W1
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "W1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P W1 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "W1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P W1 Heat
```dax
VAR t = [P W1 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P W1 Font
```dax
IF ( [P W1 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P G1
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "G1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P G1 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "G1", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P G1 Heat
```dax
VAR t = [P G1 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P G1 Font
```dax
IF ( [P G1 Pct] > 0.7, "#FFFFFF", "#252423" )
```

### P G2
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( SUM ( FactDistrict[value_shown] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "G2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P G2 Pct
```dax
VAR s = [Sel Sex]
RETURN CALCULATE ( MAX ( FactDistrict[need_pctile] ), REMOVEFILTERS ( DimIndicator ), DimIndicator[indicator_id] = "G2", REMOVEFILTERS ( DimSex ), DimSex[Sex] = s )
```

### P G2 Heat
```dax
VAR t = [P G2 Pct]
VAR r = ROUND ( 255 + t * ( 220 - 255 ), 0 )
VAR g = ROUND ( 255 + t * ( 20 - 255 ), 0 )
VAR b = ROUND ( 255 + t * ( 60 - 255 ), 0 )
VAR hx = "0123456789ABCDEF"
RETURN IF ( ISBLANK ( t ), "#F3F3F3",
    "#" & MID ( hx, INT ( r / 16 ) + 1, 1 ) & MID ( hx, MOD ( r, 16 ) + 1, 1 ) & MID ( hx, INT ( g / 16 ) + 1, 1 ) & MID ( hx, MOD ( g, 16 ) + 1, 1 ) & MID ( hx, INT ( b / 16 ) + 1, 1 ) & MID ( hx, MOD ( b, 16 ) + 1, 1 ) )
```

### P G2 Font
```dax
IF ( [P G2 Pct] > 0.7, "#FFFFFF", "#252423" )
```
