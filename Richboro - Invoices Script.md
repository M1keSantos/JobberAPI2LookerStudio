`function fetchInvoices() {`  
  `const url = "https://api.getjobber.com/api/graphql";`  
  `const accessToken = getStoredAccessToken();`

  `if (!accessToken) {`  
    `Logger.log('No access token found. Please refresh the token.');`  
    `return;`  
  `}`

  `let hasNextPage = true;`  
  `let endCursor = null;`  
   
  `let sheet = SpreadsheetApp.openById('1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8').getSheetByName('Invoices');`  
   
  `if (!sheet) {`  
    `sheet = SpreadsheetApp.openById('1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8').insertSheet('Invoices');`  
  `} else {`  
    `sheet.clear();`  
  `}`  
   
  `sheet.appendRow([`  
    `'Invoice ID', 'Invoice Number', 'Issued Date', 'Due Date', 'Subject', 'Total', 'Discount Amount',`  
    `'Client First Name', 'Client Last Name', 'Client Company Name', 'Client Is Company',`  
    `'Client Created At', 'Client Updated At', 'Client Phone Numbers', 'Client Emails',`  
    `'Billing Address Street1', 'Billing Address Street2', 'Billing Address City',`  
    `'Billing Address Province', 'Billing Address Postal Code', 'Billing Address Country',`  
    `'Line Item Name', 'Line Item Description', 'Line Item Quantity', 'Line Item Unit Price',`  
    `'Line Item Total Price', 'Line Item Taxable', 'Line Item Created At', 'Line Item Updated At'`  
  `]);`

  `while (hasNextPage) {`  
    `const query = {`  
      ``"query": `query FetchInvoices($afterCursor: String) {``  
        `invoices(first: 10, after: $afterCursor) {`  
          `nodes {`  
            `id`  
            `invoiceNumber`  
            `issuedDate`  
            `dueDate`  
            `subject`  
            `total`  
            `discountAmount`  
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
                `id`  
              `}`  
              `emails {`  
                `address`  
                `description`  
                `id`  
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
                `description`  
                `quantity`  
                `unitPrice`  
                `totalPrice`  
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
        `"afterCursor": endCursor`  
      `},`  
      `"operationName": "FetchInvoices"`  
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
       
      `if (responseCode === 200) {`  
        `const jsonResponse = JSON.parse(responseText);`  
        `if (jsonResponse.errors) {`  
          `Logger.log('Error: ' + jsonResponse.errors[0].message);`  
        `} else if (jsonResponse.data) {`  
          `const invoices = jsonResponse.data.invoices.nodes;`  
          `hasNextPage = jsonResponse.data.invoices.pageInfo.hasNextPage;`  
          `endCursor = jsonResponse.data.invoices.pageInfo.endCursor;`

          `invoices.forEach(invoice => {`  
            `const client = invoice.client || {};`  
            `const lineItems = invoice.lineItems.nodes || [];`

            `lineItems.forEach(lineItem => {`  
              `sheet.appendRow([`  
                `invoice.id,`  
                `invoice.invoiceNumber,`  
                `invoice.issuedDate,`  
                `invoice.dueDate,`  
                `invoice.subject,`  
                `invoice.total,`  
                `invoice.discountAmount,`  
                `client.firstName || 'N/A',`  
                `client.lastName || 'N/A',`  
                `client.companyName || 'N/A',`  
                `client.isCompany ? 'Yes' : 'No',`  
                `client.createdAt,`  
                `client.updatedAt,`  
                `client.phones.map(phone => phone.number).join(', '),`  
                `client.emails.map(email => email.address).join(', '),`  
                `client.billingAddress.street1 || 'N/A',`  
                `client.billingAddress.street2 || 'N/A',`  
                `client.billingAddress.city || 'N/A',`  
                `client.billingAddress.province || 'N/A',`  
                `client.billingAddress.postalCode || 'N/A',`  
                `client.billingAddress.country || 'N/A',`  
                `lineItem.name,`  
                `lineItem.description,`  
                `lineItem.quantity,`  
                `lineItem.unitPrice,`  
                `lineItem.totalPrice,`  
                `lineItem.taxable ? 'Yes' : 'No',`  
                `lineItem.createdAt,`  
                `lineItem.updatedAt`  
              `]);`  
            `});`  
          `});`

          `Logger.log('Invoices data successfully written to the sheet.');`  
        `}`  
      `} else {`  
        `Logger.log('Request failed with response code: ' + responseCode);`  
      `}`  
    `} catch (error) {`  
      `Logger.log('Error fetching invoices data: ' + error.message);`  
      `hasNextPage = false;`  
    `}`  
  `}`  
`}`

`function getStoredAccessToken() {`  
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8';`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

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
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8';`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

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
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8';`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

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
  `const sheetId = '1Ju5U4Tr03x_D4vFsBWT-TujbeLR3LhJG_gb2dyTNJO8';`  
  `const spreadsheet = SpreadsheetApp.openById(sheetId);`  
  `const sheet = spreadsheet.getSheetByName('Tokens');`

  `if (!sheet) {`  
    `Logger.log('No Tokens sheet found');`  
    `return null;`  
  `}`

  `return sheet.getRange('C1').getValue();`  
`}`

