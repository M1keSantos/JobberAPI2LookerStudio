`function fetchJobs() {`  
  `const url = "https://api.getjobber.com/api/graphql";`  
  `const accessToken = getStoredAccessToken(); // Automatically fetch the stored access token`

  `if (!accessToken) {`  
    `Logger.log('No access token found. Please refresh the token.');`  
    `return;`  
  `}`

  `let hasNextPage = true;`  
  `let afterCursor = null;`  
  `let sheet = SpreadsheetApp.openById('1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8').getSheetByName('Jobs');`  
   
  `if (!sheet) {`  
    `sheet = SpreadsheetApp.openById('1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8').insertSheet('Jobs');`  
  `} else {`  
    `sheet.clear();`  
  `}`  
   
  `// Append headers to the sheet`  
  `sheet.appendRow([`  
    `'Job ID', 'Title', 'Job Number', 'Job Status', 'Instructions', 'Start At', 'End At',`  
    `'Job Type', 'Jobber Web URI', 'Job Created At', 'Job Updated At', 'Client ID',`  
    `'Client First Name', 'Client Last Name', 'Client Company Name', 'Client Is Company',`  
    `'Client Created At', 'Client Updated At', 'Client Phone', 'Client Email',`  
    `'Client Street 1', 'Client Street 2', 'Client City', 'Client Province',`  
    `'Client Postal Code', 'Client Country', 'Line Item Name', 'Line Item Unit Price',`  
    `'Line Item Total Price', 'Line Item Quantity', 'Line Item Taxable',`  
    `'Line Item Created At', 'Line Item Updated At'`  
  `]);`

  `while (hasNextPage) {`  
    `// Define the query to fetch jobs with pagination and detailed job information`  
    `const query = {`  
      ``"query": `query FetchJobs($afterCursor: String) {``  
        `jobs(first: 10, after: $afterCursor) {`  
          `nodes {`  
            `id`  
            `title`  
            `jobNumber`  
            `jobStatus`  
            `instructions`  
            `startAt`  
            `endAt`  
            `jobType`  
            `jobberWebUri`  
            `createdAt`  
            `updatedAt`  
            `client {`  
              `id`  
              `firstName`  
              `lastName`  
              `companyName`  
              `isCompany`  
              `createdAt`  
              `updatedAt`  
              `phones {`  
                `number`  
              `}`  
              `emails {`  
                `address`  
              `}`  
              `billingAddress {`  
                `street1`  
                `street2`  
                `city`  
                `province`  
                `postalCode`  
                `country`  
              `}`  
            `}`  
            `lineItems {`  
              `nodes {`  
                `name`  
                `unitPrice`  
                `totalPrice`  
                `quantity`  
                `taxable`  
                `createdAt`  
                `updatedAt`  
              `}`  
            `}`  
          `}`  
          `pageInfo {`  
            `endCursor`  
            `hasNextPage`  
          `}`  
        `}`  
      ``}`,``  
      `"variables": {`  
        `"afterCursor": afterCursor`  
      `},`  
      `"operationName": "FetchJobs"`  
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
          `break;`  
        `} else if (jsonResponse.data) {`  
          `const jobs = jsonResponse.data.jobs.nodes;`  
          `hasNextPage = jsonResponse.data.jobs.pageInfo.hasNextPage;`  
          `afterCursor = jsonResponse.data.jobs.pageInfo.endCursor;`

          `jobs.forEach(job => {`  
            `const jobId = job.id;`  
            `const title = job.title;`  
            `const jobNumber = job.jobNumber;`  
            `const jobStatus = job.jobStatus;`  
            `const instructions = job.instructions || '';`  
            `const startAt = job.startAt;`  
            `const endAt = job.endAt;`  
            `const jobType = job.jobType;`  
            `const jobberWebUri = job.jobberWebUri;`  
            `const jobCreatedAt = job.createdAt;`  
            `const jobUpdatedAt = job.updatedAt;`

            `const client = job.client || {};`  
            `const clientId = client.id || '';`  
            `const clientFirstName = client.firstName || '';`  
            `const clientLastName = client.lastName || '';`  
            `const clientCompanyName = client.companyName || '';`  
            `const clientIsCompany = client.isCompany ? 'Yes' : 'No';`  
            `const clientCreatedAt = client.createdAt || '';`  
            `const clientUpdatedAt = client.updatedAt || '';`  
            `const clientPhone = client.phones && client.phones.length > 0 ? client.phones[0].number : '';`  
            `const clientEmail = client.emails && client.emails.length > 0 ? client.emails[0].address : '';`  
            `const billingAddress = client.billingAddress || {};`  
            `const clientStreet1 = billingAddress.street1 || '';`  
            `const clientStreet2 = billingAddress.street2 || '';`  
            `const clientCity = billingAddress.city || '';`  
            `const clientProvince = billingAddress.province || '';`  
            `const clientPostalCode = billingAddress.postalCode || '';`  
            `const clientCountry = billingAddress.country || '';`

            `const lineItems = job.lineItems.nodes || [];`  
            `if (lineItems.length > 0) {`  
              `lineItems.forEach(lineItem => {`  
                `const lineItemName = lineItem.name || '';`  
                `const lineItemUnitPrice = lineItem.unitPrice || '';`  
                `const lineItemTotalPrice = lineItem.totalPrice || '';`  
                `const lineItemQuantity = lineItem.quantity || '';`  
                `const lineItemTaxable = lineItem.taxable ? 'Yes' : 'No';`  
                `const lineItemCreatedAt = lineItem.createdAt || '';`  
                `const lineItemUpdatedAt = lineItem.updatedAt || '';`

                `// Append job and line item data to the sheet`  
                `sheet.appendRow([`  
                  `jobId, title, jobNumber, jobStatus, instructions, startAt, endAt,`  
                  `jobType, jobberWebUri, jobCreatedAt, jobUpdatedAt, clientId,`  
                  `clientFirstName, clientLastName, clientCompanyName, clientIsCompany,`  
                  `clientCreatedAt, clientUpdatedAt, clientPhone, clientEmail,`  
                  `clientStreet1, clientStreet2, clientCity, clientProvince,`  
                  `clientPostalCode, clientCountry, lineItemName, lineItemUnitPrice,`  
                  `lineItemTotalPrice, lineItemQuantity, lineItemTaxable,`  
                  `lineItemCreatedAt, lineItemUpdatedAt`  
                `]);`  
              `});`  
            `} else {`  
              `// Append job data without line items`  
              `sheet.appendRow([`  
                `jobId, title, jobNumber, jobStatus, instructions, startAt, endAt,`  
                `jobType, jobberWebUri, jobCreatedAt, jobUpdatedAt, clientId,`  
                `clientFirstName, clientLastName, clientCompanyName, clientIsCompany,`  
                `clientCreatedAt, clientUpdatedAt, clientPhone, clientEmail,`  
                `clientStreet1, clientStreet2, clientCity, clientProvince,`  
                `clientPostalCode, clientCountry, '', '', '', '', '', ''`  
              `]);`  
            `}`  
          `});`

          `Logger.log('Jobs data successfully written to the sheet.');`  
        `}`  
      `} else {`  
        `Logger.log('Request failed with response code: ' + responseCode);`  
        `break;`  
      `}`  
    `} catch (error) {`  
      `Logger.log('Error fetching jobs data: ' + error.message);`  
      `break;`  
    `}`

    `// Sleep to avoid hitting the rate limit`  
    `Utilities.sleep(1000);`  
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

