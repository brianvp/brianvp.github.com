---
id: 344
title: Generating JSON from CSV Using Powershell
date: 2015-10-08T19:12:58+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=344
permalink: /2015/10/08/generating-json-from-csv-using-powershell/
categories:
  - General
tags:
  - csv
  - Excel
  - JSON
  - Powershell
---
<span style="font-weight: 400;">Here is a technique I picked up while researching ways to generate JSON output from a CSV file.  My main use case is generating sample data quickly.  While JSON is easy to read and parse, I don’t feel like hand typing dozens of string/value pairs, I want to use excel like a sane person.  I also do not want to upload my JSON to some random internet site.  Thankfully Powershell (v3) has a built-in cmdlet that allows us to convert to JSON.  </span>

## <span style="font-weight: 400;">Step 1 &#8211; Generate Test Data using preferred spreadsheet</span>

[<img class="alignnone wp-image-345 size-full" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVExcelInput.png" alt="JsonCSVExcelInput" width="668" height="146" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVExcelInput.png)

## <span style="font-weight: 400;">Step 2 &#8211; Save File as either CSV or Tab-Delimited text file</span>

<img class="alignnone wp-image-346 size-full" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVSaveDialog.png" alt="JsonCSVSaveDialog" width="615" height="165" />

[<img class="alignnone size-full wp-image-356" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVInputFile.png" alt="JsonCSVInputFile" width="641" height="129" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVInputFile.png)

## <span style="font-weight: 400;">Step 3 &#8211; Parse the ouput file using the ConvertTo-Json powershell cmdlet, and output to a json file</span>

```powershell
import-csv "SampleInput.csv" | ConvertTo-Json | Add-Content -Path "output.json"
```

[<img class="alignnone wp-image-347 size-full" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVPowershellStep1.png" alt="JsonCSVPowershellStep1" width="648" height="73" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVPowershellStep1.png)

which produces the following output:

[<img class="alignnone wp-image-348 size-full" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVJsonOutput.png" alt="JsonCSVJsonOutput" width="400" height="402" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVJsonOutput.png)

### <span style="font-weight: 400;">variation #1 &#8211; Convert from tab-delimited file</span>

```powershell
import-csv -Delimiter "`t" "SampleInput.txt"  | ConvertTo-Json | Add-Content -Path "output.json"
```

### <span style="font-weight: 400;">variation #2 &#8211; Remove whitespace / carriage returns from output file</span>

```powershell
import-csv "SampleInput.csv" | ConvertTo-Json -Compress | Add-Content -Path "output.json"
```

which produces the following output:

[<img class="alignnone wp-image-349 size-full" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVJsonOutputCompressed.png" alt="JsonCSVJsonOutputCompressed" width="695" height="65" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/JsonCSVJsonOutputCompressed.png)