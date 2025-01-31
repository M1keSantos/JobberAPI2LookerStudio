`function fetchTotalNumberOfSchedules() {`  
  `const url = "https://api.getjobber.com/api/graphql";`  
  `const accessToken = getStoredAccessToken(); // Automatically fetch the stored access token`

  `if (!accessToken) {`  
    `Logger.log('No access token found. Please refresh the token.');`  
    `return;`  
  `}`

  `// Define the query to fetch the total number of schedules`  
  `const query = {`  
    ``"query": `query GetTotalSchedules {``  
      `jobs {`  
        `totalCount`  
      `}`  
    ``}`,``  
    `"operationName": "GetTotalSchedules"`  
  `};`

  `const options = {`  
    `'method': 'post',`  
    `'contentType': 'application/json',`  
    `'headers': {`  
      `'Authorization': 'Bearer ' + accessToken,`  
      `'X-JOBBER-GRAPHQL-VERSION': '2024-06-10'`  
    `},`  
    `'payload': JSON.stringify(query),`  
    `'muteHttpExceptions': true`  
  `};`

  `Logger.log('URL: ' + url);`  
  `Logger.log('Headers: ' + JSON.stringify(options.headers));`  
  `Logger.log('Payload: ' + JSON.stringify(query));`

  `let sheet = SpreadsheetApp.openById('1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8').getSheetByName('TotalSchedules');`  
   
  `if (!sheet) {`  
    `sheet = SpreadsheetApp.openById('1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8').insertSheet('TotalSchedules');`  
  `} else {`  
    `sheet.clear();`  
  `}`  
   
  `sheet.appendRow(['Total Schedules']); // Column header`

  `try {`  
    `const response = UrlFetchApp.fetch(url, options);`  
    `const responseCode = response.getResponseCode();`  
    `const responseText = response.getContentText();`  
     
    `Logger.log('Response Code: ' + responseCode);`  
    `Logger.log('Response Body: ' + responseText);`  
     
    `if (responseCode === 200) {`  
      `const jsonResponse = JSON.parse(responseText);`  
      `if (jsonResponse.errors) {`  
        `Logger.log('Error: ' + jsonResponse.errors[0].message);`  
      `} else if (jsonResponse.data) {`  
        `const totalCount = jsonResponse.data.jobs.totalCount;`

        `sheet.appendRow([totalCount]);`

        `Logger.log('Total number of schedules successfully written to the sheet.');`  
      `}`  
    `} else {`  
      `Logger.log('Request failed with response code: ' + responseCode);`  
    `}`  
  `} catch (error) {`  
    `Logger.log('Error fetching total number of schedules: ' + error.message);`  
  `}`  
`}`

`function getStoredAccessToken() {`  
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8'; // Your Google Sheets ID`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return null;`  
  `}`

  `const accessToken = sheet.getRange('B1').getValue(); // Access token is in cell B1`

  `// Check if access token is expired`  
  `if (!accessToken || isTokenExpired()) {`  
    `Logger.log('Access token is expired or not found, refreshing...');`  
    `refreshAccessToken();`  
    `return sheet.getRange('B1').getValue(); // Return the newly refreshed access token`  
  `}`

  `return accessToken;`  
`}`

`function isTokenExpired() {`  
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8'; // Your Google Sheets ID`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return true; // If the sheet is not found, assume the token is expired`  
  `}`

  `const lastUpdated = sheet.getRange('A1').getValue(); // Timestamp of the last token update`  
  `const currentTime = new Date();`  
  `const timeDifference = currentTime.getTime() - new Date(lastUpdated).getTime();`  
  `const hourDifference = timeDifference / (1000 * 60 * 60);`

  `return hourDifference >= 1; // Return true if the token is older than 1 hour`  
`}`

`function refreshAccessToken() {`  
  `const clientId = '80a216eb-bccd-4534-8578-570f718b6fb5';`  
  `const clientSecret = 'aeb9a91b1e12613074693e4be8a9dc77b84ca2bb20cb0d653534ed93e259f983';`  
  `const refreshToken = getStoredRefreshToken(); // Fetch the stored refresh token`

  `if (!refreshToken) {`  
    `Logger.log('No stored refresh token found. Please add it to cell C1 of the "Tokens" sheet.');`  
    `return;`  
  `}`

  `const payload = {`  
    `'client_id': clientId,`  
    `'client_secret': clientSecret,`  
    `'grant_type': 'refresh_token',`  
    `'refresh_token': refreshToken`  
  `};`

  `const payloadString = Object.keys(payload)`  
    `.map(key => encodeURIComponent(key) + '=' + encodeURIComponent(payload[key]))`  
    `.join('&');`

  `const options = {`  
    `'method': 'post',`  
    `'contentType': 'application/x-www-form-urlencoded',`  
    `'payload': payloadString`  
  `};`

  `const url = 'https://api.getjobber.com/api/oauth/token';`  
  `const response = UrlFetchApp.fetch(url, options);`  
  `const jsonResponse = JSON.parse(response.getContentText());`

  `if (jsonResponse.access_token) {`  
    `Logger.log('Access token refreshed successfully.');`  
    `Logger.log('New Access Token: ' + jsonResponse.access_token);`  
    `Logger.log('New Refresh Token: ' + jsonResponse.refresh_token);`  
     
    `updateJobberTokens(jsonResponse);`  
  `} else {`  
    `Logger.log('Error refreshing access token: ' + jsonResponse.error);`  
  `}`  
`}`

`function updateJobberTokens(tokens) {`  
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8'; // Your Google Sheets ID`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return;`  
  `}`

  `sheet.getRange('A1').setValue(new Date()); // Timestamp of token update`  
  `sheet.getRange('B1').setValue(tokens.access_token); // Store the access token in cell B1`  
  `sheet.getRange('C1').setValue(tokens.refresh_token); // Store the refresh token in cell C1`

  `Logger.log('Tokens updated successfully in the Google Sheet.');`  
`}`

`function getStoredRefreshToken() {`  
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8'; // Your Google Sheets ID`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return null;`  
  `}`

  `return sheet.getRange('C1').getValue(); // Refresh token is in cell C1`  
`}`

