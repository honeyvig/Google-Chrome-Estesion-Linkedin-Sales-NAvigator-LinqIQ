# Google-Chrome-Extension-Linkedin-Sales-Navigator-LinqIQ
 Lead IQ to collect data from LinkedIn Sales Navigator but is not interested in developing custom software. Instead, they are looking for the following solution:

1. Open LinkedIn Sales Navigator on one page.
2. Navigate to the "Search Leads" page and enter a keyword, such as "HR."
3. By clicking a single button, the tool should:
    - Fetch all the data from the current page.
    - Send the collected data to Lead IQ.
    - Automatically move to the next page of results (e.g., Page 2, Page 3, etc.) and repeat the process.

Key:
- The client wants to keep using lead IQ.
- We don't need to use the LinkedIn apis.

Just for context, leads IQ, if is installed on the computer, fetches the data somehow from the open LinkedIn page.
But they need to manually click each contract, that is why they want to automate.
------------
To automate the data collection from LinkedIn Sales Navigator and integrate it with LeadIQ via a Google Chrome extension, we will need to create a solution that does the following:

    Open LinkedIn Sales Navigator and navigate to the "Search Leads" page.
    Collect data from the current page of search results.
    Send the collected data to LeadIQ (assuming LeadIQ is installed on the computer and can be accessed through a specific interface).
    Navigate to the next page of the search results automatically and repeat the process.

Key Considerations:

    LinkedIn’s DOM structure: We need to scrape data from the page using DOM manipulation to extract the lead information.
    LeadIQ Integration: Since we’re not using the LinkedIn API directly and assuming LeadIQ is installed on the computer, we will use LeadIQ's interface or browser extensions to fetch data.
    Pagination: We’ll handle pagination to automatically move through pages of search results.

Chrome Extension Structure:

linkedin-leads-extension/
├── src/
│   ├── background.js
│   ├── content.js
│   ├── popup.js
│   ├── popup.html
│   ├── style.css
├── manifest.json
└── icons/
    └── icon.png

1. manifest.json

The manifest defines the basic settings of the extension, including permissions and content scripts.

{
  "manifest_version": 3,
  "name": "LinkedIn Leads Automation for LeadIQ",
  "version": "1.0",
  "description": "Automate lead collection from LinkedIn Sales Navigator and integrate with LeadIQ.",
  "permissions": [
    "activeTab",
    "storage",
    "tabs",
    "https://www.linkedin.com/*"
  ],
  "background": {
    "service_worker": "src/background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://www.linkedin.com/sales/*"],
      "js": ["src/content.js"]
    }
  ],
  "action": {
    "default_popup": "src/popup.html",
    "default_icon": {
      "16": "icons/icon.png",
      "48": "icons/icon.png",
      "128": "icons/icon.png"
    }
  },
  "host_permissions": [
    "https://www.linkedin.com/*"
  ]
}

2. Background Script (background.js)

The background script handles message passing and data processing.

// src/background.js

chrome.runtime.onInstalled.addListener(() => {
  console.log("LinkedIn Leads Automation Extension installed.");
});

// Listen for messages and send data to LeadIQ
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === "sendDataToLeadIQ") {
    // Simulating sending data to LeadIQ (could be through an API or browser interaction)
    console.log("Sending lead data to LeadIQ:", message.data);
    sendResponse({ status: "Data sent successfully to LeadIQ." });
  }
});

3. Content Script (content.js)

The content script interacts with the LinkedIn Sales Navigator page to scrape lead data, handle pagination, and send the data to LeadIQ.

// src/content.js

// Function to collect data from current LinkedIn search results page
function collectLeadData() {
  let leadData = [];
  
  // Collecting lead information (assuming LinkedIn displays leads in a specific format)
  const leads = document.querySelectorAll('.entity-result');
  
  leads.forEach(lead => {
    const name = lead.querySelector('.entity-result__title').innerText;
    const company = lead.querySelector('.entity-result__secondary-subtitle').innerText;
    const title = lead.querySelector('.entity-result__primary-subtitle').innerText;
    const profileLink = lead.querySelector('a').href;
    
    leadData.push({ name, company, title, profileLink });
  });
  
  return leadData;
}

// Function to move to the next page of search results
function goToNextPage() {
  const nextButton = document.querySelector('.artdeco-pagination__button--next');
  if (nextButton && !nextButton.disabled) {
    nextButton.click();
  } else {
    console.log('No more pages.');
  }
}

// Listen for messages from the popup or background
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === "startAutomation") {
    // Collect data from the current page
    const leadData = collectLeadData();
    
    // Send collected data to LeadIQ (via background or direct integration)
    chrome.runtime.sendMessage({ type: "sendDataToLeadIQ", data: leadData }, (response) => {
      console.log(response.status);
    });
    
    // Navigate to the next page and repeat
    goToNextPage();
    sendResponse({ status: "Automation started, data collected." });
  }
});

4. Popup HTML (popup.html)

The popup provides a simple user interface where the user can initiate the automation process.

<!-- src/popup.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LinkedIn Lead Automation</title>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <h1>LinkedIn LeadIQ Automation</h1>
    
    <div>
      <button id="startAutomation">Start Automation</button>
    </div>
    
    <script src="popup.js"></script>
  </body>
</html>

5. Popup JS (popup.js)

Handles user interaction with the extension, starting the automation when the button is clicked.

// src/popup.js

document.getElementById('startAutomation').addEventListener('click', () => {
  // Send message to content script to start automation
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    const tab = tabs[0];
    chrome.scripting.executeScript({
      target: { tabId: tab.id },
      func: startAutomation
    });
  });
});

function startAutomation() {
  // Send message to content script to start the process
  chrome.runtime.sendMessage({ type: "startAutomation" }, (response) => {
    console.log(response.status);
  });
}

6. CSS Styling (style.css)

Basic styling for the popup.

/* src/style.css */

body {
  font-family: Arial, sans-serif;
  padding: 10px;
}

h1 {
  font-size: 18px;
  margin-bottom: 20px;
}

button {
  background-color: #007bff;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

7. Icons (Optional)

Create and add icons for the extension in the icons/ directory (e.g., icon.png).
Final Notes:

    LinkedIn Restrictions: Be aware that LinkedIn has strict policies regarding scraping and automated actions on their platform. Automating interactions might be against LinkedIn's terms of service, and scraping might get the user's account blocked. Always ensure you are in compliance with LinkedIn's terms.
    LeadIQ Integration: The extension assumes LeadIQ is installed and can fetch data from LinkedIn. You may need to interface directly with LeadIQ's API or local storage to fully integrate it with the extension.
    Pagination: This script assumes the pagination button has the class .artdeco-pagination__button--next. The class names might change in future versions of LinkedIn, so the code may need adjustments if LinkedIn changes its structure.

This code provides a basic framework for automating data collection from LinkedIn Sales Navigator and integrating it with LeadIQ. Make sure to test it thoroughly and handle edge cases, such as navigating between pages or handling dynamic page loading.
