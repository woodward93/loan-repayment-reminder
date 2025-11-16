# loan-repayment-reminder

**📣 Loan Repayment Workflow (n8n)**
Automate loan-repayment reminders with push notifications via OneSignal, driven by a Google Sheets source of active loans. The workflow supports English & French audiences with tailored message templates and multiple repayment states (late, due today, due in 3 or 10 days, >10 days overdue).


**🔎 What this workflow does**

- Schedules every 3 days @ 11:00.
- Reads loan records from Google Sheets.
- Normalizes key fields (Edit Fields node).
- Filters by Year Disbursed and loan Status.
- Fetches the user’s OneSignal ID using their external_id (for this use case the external_id is the app's username).
- Routes users by language (English vs French).
- Segments users into repayment states:
  	- Past due
  	- Due today
  	- Due in 3 days
  	- Due in 10 days
  	- >10 days overdue (warning)
- Sends localized push notifications via OneSignal for each segment.


**🧩 Integrations**

**Google Sheets (OAuth2):** loan data source

**OneSignal (REST API):**
- Get user onesignal_id using the external_id
- Send push notifications with localized content and the OneSignal_id


**🚦 Segmentation logic (high level)**
**Include only rows where:**
  Year Disbursed == 2025 AND Status == "Good"

**Then branch by language:**
If properties.language == "fr" → French flow
  Else → English flow

Then segment by due state:
  - Past due: Days to due date < 0
  - Due today: Days to due date == 0
  - Due in 3 days: Days to due date == 3
  - Due in 10 days: Days to due date == 10
  - >10 days overdue (warning): Days to due date <= -10

**📨 Message templates (OneSignal)**

All notifications are sent via POST https://api.onesignal.com/notifications?c=push with:

**_{
  "app_id": "Your_One_Signal_App_ID",
  "headings": { "en": "..." },
  "contents": { "en": "..." },
  "include_aliases": { "onesignal_id": ["{{ onesignal_id }}"] },
  "target_channel": "push"
}_**


**English (examples)**

**Past due**

_headings.en: Your loan repayment is late_

_contents.en: Your loan repayment of {{ Amount Due }} was due {{ Days to due date (actual) }} day(s) ago. Please make repayment now to avoid late fees._

**French (examples)**

**Past due**

_headings.fr: Votre remboursement de prêt est en retard_

_contents.fr: Votre remboursement de prêt de {{ Amount Due }} était dû il y a {{ Days to due date (actual) }} jour(s). Veuillez effectuer le paiement maintenant afin d’éviter des frais de retard._


**🛠 Setup**

- Import the workflow JSON into n8n.

- Create Google Sheets OAuth2 credentials and update the Get repayment data node:

		- Connect your Google spreadsheet
		- Select preferred sheet with the loan data

- Set OneSignal credentials in all HTTP nodes:

	- **Header Authorization:** Your_One_Signal_API_KEY

	- **Payload app_id:** Your_One_Signal_App_ID

- Confirm field names in your sheet match those referenced in Edit Fields.

- Activate the workflow and monitor the first run via Executions in n8n.


**🔒 Compliance & privacy**
-	Do not store PII in logs.
- Keep API keys in n8n credentials or environment variables.
- Confirm you have consent to send push notifications and that your content complies with local regulations.
