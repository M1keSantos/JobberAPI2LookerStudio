`const API_URL = 'https://api.getjobber.com/api/graphql';`  
`const SHEET_ID = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8'; // Your Google Sheets ID`

`function fetchAllQuotes() {`  
  `let accessToken = getStoredAccessToken(); // Automatically fetch the stored access token`

  `if (!accessToken) {`  
    `Logger.log('No access token found. Please refresh the token.');`  
    `return;`  
  `}`

  `let afterCursor = null;`  
  `let hasNextPage = true;`

  `// Initialize the sheet`  
  `let sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Quotes');`  
   
  `if (!sheet) {`  
    `sheet = SpreadsheetApp.openById(SHEET_ID).insertSheet('Quotes');`  
  `} else {`  
    `sheet.clear();`  
  `}`  
   
  `sheet.appendRow([`  
    `'Quote ID', 'Title', 'Created At', 'Updated At',`  
    `'Client ID', 'Client First Name', 'Client Last Name', 'Client Company Name', 'Client Phone', 'Client Email', 'Billing Address',`  
    `'Property Address', 'Line Item Name', 'Line Item Description', 'Line Item Unit Price', 'Line Item Total Price', 'Line Item Quantity', 'Line Item Created At', 'Line Item Updated At', 'Line Item Taxable',`  
    `'Message'`  
  `]); // Column headers`

  `while (hasNextPage) {`  
    `const query = {`  
      ``"query": `query FetchFirst10Quotes($afterCursor: String) {``  
        `quotes(first: 10, after: $afterCursor) {`  
          `nodes {`  
            `id`  
            `title`  
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
                `description`  
                `smsAllowed`  
              `}`  
              `emails {`  
                `address`  
                `description`  
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
            `property {`  
              `id`  
              `street1`  
              `street2`  
              `city`  
              `province`  
              `postalCode`  
              `country`  
            `}`  
            `lineItems {`  
              `nodes {`  
                `name`  
                `description`  
                `unitPrice`  
                `totalPrice`  
                `quantity`  
                `createdAt`  
                `updatedAt`  
                `taxable`  
              `}`  
            `}`  
            `message`  
          `}`  
          `pageInfo {`  
            `endCursor`  
            `hasNextPage`  
          `}`  
          `totalCount`  
        `}`  
      ``}`,``  
      `"variables": {`  
        `"afterCursor": afterCursor`  
      `},`  
      `"operationName": "FetchFirst10Quotes"`  
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
      `const response = UrlFetchApp.fetch(API_URL, options);`  
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
          `const quotes = jsonResponse.data.quotes.nodes;`  
          `hasNextPage = jsonResponse.data.quotes.pageInfo.hasNextPage;`  
          `afterCursor = jsonResponse.data.quotes.pageInfo.endCursor;`

          `quotes.forEach(quote => {`  
            `const quoteId = quote.id;`  
            `const title = quote.title;`  
            `const createdAt = quote.createdAt;`  
            `const updatedAt = quote.updatedAt;`  
            `const client = quote.client;`  
            `const property = quote.property;`  
            `const lineItems = quote.lineItems.nodes;`

            `const clientFirstName = client ? client.firstName : '';`  
            `const clientLastName = client ? client.lastName : '';`  
            `const clientCompanyName = client ? client.companyName : '';`  
            `const clientPhone = client && client.phones.length > 0 ? client.phones[0].number : '';`  
            `const clientEmail = client && client.emails.length > 0 ? client.emails[0].address : '';`

            ``const billingAddress = client && client.billingAddress ? `${client.billingAddress.street1}, ${client.billingAddress.city}, ${client.billingAddress.province}, ${client.billingAddress.postalCode}, ${client.billingAddress.country}` : '';``  
            ``const propertyAddress = property ? `${property.street1}, ${property.city}, ${property.province}, ${property.postalCode}, ${property.country}` : '';``

            `lineItems.forEach(item => {`  
              `sheet.appendRow([`  
                `quoteId, title, createdAt, updatedAt,`  
                `client.id, clientFirstName, clientLastName, clientCompanyName, clientPhone, clientEmail, billingAddress, propertyAddress,`  
                `item.name, item.description, item.unitPrice, item.totalPrice, item.quantity, item.createdAt, item.updatedAt, item.taxable,`  
                `quote.message`  
              `]);`  
            `});`  
          `});`

          `Logger.log('Quotes data successfully written to the sheet.');`  
        `}`  
      `} else {`  
        `Logger.log('Request failed with response code: ' + responseCode);`  
      `}`

    `} catch (error) {`  
      `Logger.log('Error fetching quotes data: ' + error.message);`  
    `}`  
  `}`  
`}`

`function getStoredAccessToken() {`  
  `const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Tokens');`  
  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return null;`  
  `}`  
  `const accessToken = sheet.getRange('B1').getValue();`  
  `if (!accessToken || isTokenExpired()) {`  
    `Logger.log('Access token is expired or not found, refreshing...');`  
    `refreshAccessToken();`  
    `return sheet.getRange('B1').getValue();`  
  `}`  
  `return accessToken;`  
`}`

`function isTokenExpired() {`  
  `const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Tokens');`  
  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return true;`  
  `}`  
  `const lastUpdated = sheet.getRange('A1').getValue();`  
  `const currentTime = new Date();`  
  `const timeDifference = currentTime.getTime() - new Date(lastUpdated).getTime();`  
  `const hourDifference = timeDifference / (1000 * 60 * 60);`  
  `return hourDifference >= 1;`  
`}`

`function refreshAccessToken() {`  
  `const clientId = '80a216eb-bccd-4534-8578-570f718b6fb5';`  
  `const clientSecret = 'aeb9a91b1e12613074693e4be8a9dc77b84ca2bb20cb0d653534ed93e259f983';`  
  `const refreshToken = getStoredRefreshToken();`  
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
    `updateJobberTokens(jsonResponse);`  
  `} else {`  
    `Logger.log('Error refreshing access token: ' + jsonResponse.error);`  
  `}`  
`}`

`function updateJobberTokens(tokens) {`  
  `const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Tokens');`  
  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return;`  
  `}`  
  `sheet.getRange('A1').setValue(new Date());`  
  `sheet.getRange('B1').setValue(tokens.access_token);`  
  `sheet.getRange('C1').setValue(tokens.refresh_token);`  
  `Logger.log('Tokens updated successfully in the Google Sheet.');`  
`}`

`function getStoredRefreshToken() {`  
  `const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Tokens');`  
  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return null;`  
  `}`  
  `return sheet.getRange('C1').getValue();`  
`}`

