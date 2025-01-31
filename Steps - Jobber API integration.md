

| Steps | Title | Description | Procedure |
| ----- | :---- | :---- | :---- |
| 1 | **Creating an app on Jobber API** | Begin by registering a new application in the Jobber Developer Portal. This will provide you with the necessary client\_id and client\_secret for API authentication. | [**Step-by-Step Guide: Creating an App on Jobber API**](https://docs.google.com/document/d/1hjQYXL8ISSHJl2_Fqk2PdtaoVdQa-LDn6qPkB-LnhGQ/edit?usp=sharing) |
| 2 | **Run Authorization process** | Use OAuth 2.0 to authorize your app to access Jobber’s data on behalf of a user. This involves generating an authorization URL, which the user will visit to grant access. | [**Step-by-Step Guide: Running the Authorization Process**](https://docs.google.com/document/d/17sW9l4FzZ-FzM1zKFJADLBsd4cZLN5QpScrScvY0UkE/edit?usp=sharing) |
| 3 | **Get the refresh token and Access token** | After authorization, exchange the authorization code for an access token and a refresh token. The access token is used for API requests, while the refresh token allows you to obtain new access tokens without user intervention. | [**Step-by-Step Guide: Getting the Refresh Token and Access Token**](https://docs.google.com/document/d/1n4O72fqjJ6_oW5i5P0vkAbNuG1oY0vMqRdlmgAPyg3g/edit?usp=sharing) |
| 4 | **Run Queries on GraphiQL** | Test and fine-tune your GraphQL queries using Jobber’s GraphiQL interface. This helps in retrieving the specific data needed for your reports. | [**Step-by-Step Guide: Running Queries on GraphiQL**](https://docs.google.com/document/d/12IGuh3TMkEMX2O6YyfyyJSvuEaW9x5SUwsRfa3At0f4/edit?usp=sharing) |
| 5 | **Create a script to fetch data from Jobber using API** | Write a script (e.g., in Google Apps Script) that automates the process of fetching data from Jobber's API using the access token. This script will execute the GraphQL queries to pull the data you need. | [**Step-by-Step Guide: Creating a Script to Fetch Data from Jobber Using API**](https://docs.google.com/document/d/1S-9Q9iiULxcRl3rTTYEVzwA9qubkZqWatDJ1pEQ-hu4/edit?usp=sharing) |
| 6 | **Automate by refresh token function and script triggers** | Enhance your script with functions to automatically refresh the access token using the refresh token. Set up triggers (e.g., time-based) to run your data-fetching script periodically. | [**Step-by-Step Guide: Automate by Refresh Token Function and Script Triggers**](https://docs.google.com/document/d/1zr9qUNMxSEM14YRLaTvQ0VbgZCQ_5PP_qr_7EP1M-hU/edit?usp=sharing) |
| 7 | **Connect the data and creating a report on looker studio** | Import the fetched data into Looker Studio using a suitable data connector (like Google Sheets). This will allow you to visualize the data from Jobber. | [**Step-by-Step Guide: Connecting the Data to Looker Studio**](https://docs.google.com/document/d/1bgpuouMk-6AhtGGJTwbK00OCsvoKYElThGcaLj61mDg/edit?usp=sharing) |
|  |  |  |  |
|  |  |  |  |
|  | **client\_id:** | 80a216eb-bccd-4534-8578-570f718b6fb5 |  |
|  | **client\_secret:** | aeb9a91b1e12613074693e4be8a9dc77b84ca2bb20cb0d653534ed93e259f983 |  |
|  | **Direct\_URI:** | [https://example.com/callback](https://example.com/callback) |  |
|  | **Auth\_URL:** | [https://api.getjobber.com/api/oauth/authorize?response\_type=code\&client\_id=80a216eb-bccd-4534-8578-570f718b6fb5\&redirect\_uri=https://example.com/callback\&state=\<STATE\>](https://api.getjobber.com/api/oauth/authorize?response_type=code&client_id=80a216eb-bccd-4534-8578-570f718b6fb5&redirect_uri=https://example.com/callback&state=\<STATE\>) |  |
|  | **Token\_Endpoint:** | [https://api.getjobber.com/api/oauth/authorize?response\_type=code\&client\_id=80a216eb-bccd-4534-8578-570f718b6fb5\&redirect\_uri=https://example.com/callback\&state=\<STATE\>](https://api.getjobber.com/api/oauth/authorize?response_type=code&client_id=80a216eb-bccd-4534-8578-570f718b6fb5&redirect_uri=https://example.com/callback&state=\<STATE\>) |  |
|  | **Jobber developer account:** | jesso@deptfordfence.com | \!Hjfence123\! |
|  | **Authorized with: Jobber\_Richboro** | jjmmarketing1983@gmail.com | hC\&YM9f?28B9V&8 |
|  | **Appscript token exchange:** | [function exchangeAuthCodeForTokens() {](https://docs.google.com/document/d/1FnGTgTPHSn26LT0VBYGul0LaXy24K_YrNeOLbwo_ID4/edit?usp=sharing) |  |

