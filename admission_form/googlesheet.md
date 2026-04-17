# Google Sheets Webhook Setup Guide

To link your HTML form to Google Sheets, follow these step-by-step instructions.

## Step 1: Create a Google Sheet
1. Open [Google Sheets](https://sheets.google.com) and create a new blank spreadsheet.
2. Name your spreadsheet (e.g., "Admission Form Submissions").
3. In the first row (A1, B1, C1, etc.), create headers for **every `name` attribute** used in the HTML form. For example:
   - `app_no`, `session`, `submission_date`, `class_admitted`
   - `child_name`, `dob`, `calculated_age`, `gender`, `blood_group`, `religion`, `category`, `child_aadhar`, `prev_school`, `prev_school_name`, `disability`
   - `f_name`, `f_aadhar`, `f_mob`, `f_edu`, `f_occ_inc`
   - `m_name`, `m_aadhar`, `m_mob`, `m_edu`, `m_occ_inc`
   - `address`, `city`, `state`, `pin`
   - `sibling1_name`, `sibling1_gender`, `sibling1_age`, `sibling1_class`, `sibling1_school` (and duplicate for sibling 2, 3, 4)

## Step 2: Create the Apps Script
1. In your Google Sheet, go to **Extensions > Apps Script**.
2. Delete any existing code in the code editor, and copy/paste the following script:

```javascript
const sheetName = 'Sheet1'; // Change this if your sheet has a different name

function doPost(e) {
  try {
    const doc = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = doc.getSheetByName(sheetName);
    const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    
    // Process form data
    const newRow = headers.map(header => {
      // Find the value associated with the header name
      let value = e.parameter[header] === undefined ? "" : e.parameter[header];
      return value;
    });

    // Add row to our spreadsheet
    sheet.appendRow(newRow);

    // Return a success JSON response properly configured for CORS
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': sheet.getLastRow() }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': error }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Click the **Save** icon (💾) at the top.

## Step 3: Deploy as a Web App
1. Click the **Deploy** button at the top right, then select **New deployment**.
2. Under "Select type" (the gear icon), choose **Web app**.
3. Fill in the details:
   - **Description**: Add an optional description (e.g., "Admission Webhook V1").
   - **Execute as**: Select **Me**.
   - **Who has access**: Change this to **Anyone**. *(Important! Otherwise the form will get blocked)*
4. Click **Deploy**.
5. Google will ask you to authorize the app. Click **Authorize access**, select your account, click **Advanced**, and then click "Go to [Your Script Name] (unsafe)".
6. Allow the requested permissions.

## Step 4: Link Script to HTML Form
1. Once deployed, you will be given a **Web app URL**. Click the **Copy** button to copy this long URL.
2. Open your `admission_form.html` in a text or code editor.
3. Scroll down to the script section at the bottom to `line 944` (or where the `YOUR_GOOGLE_APPS_SCRIPT_URL_HERE` comment is).
4. Locate the following code:
```javascript
const scriptURL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
```
5. Replace `'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE'` with the **Web app URL** you just copied. Ensure the URL is enclosed in quotes.
6. Save the HTML file.

Your form is now completely hooked up to your Google Sheet! When a user submits the form, a new row will be created continuously.
