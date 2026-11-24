# WhatsApp setup: your steps (24 Nov 2026)

The product side is built and tested:
- sending through approved templates (A1.2, `3c15e34`);
- delivery status by webhook (A1.1, `46e1c0a`);
- farmers' replies by webhook (A1.3, `bc35ed9`).

Decisions: CHAT-55. What is left are the steps only you can do in your accounts. **Never paste a token, secret or friend's number into chat, git or a report.**

## 1. Get the version 2 texts reviewed (needed before anything is sent)
Meta allows no line break inside a value, so the two message texts have a version 2. It has the same words as version 1; only the sale lines are joined with ` ; `.
- Send the six texts below (2 kinds × English, Punjabi, Hindi) to the reviewers who checked version 1 on 28 Oct.
- When they confirm, tell me how to name them and the date. I record the review in the product. Until then, nothing goes out through WhatsApp: the send button answers "not reviewed".
- If they change any word, tell me. That becomes a version 3, and the Meta templates must match it.

## 2. In the Meta developer app (developers.facebook.com → your app → WhatsApp → API Setup)
1. **Phone number ID** of the test number: digits, shown under "From".
2. **API version**: shown in the sample request, for example `v23.0`.
3. **Recipients:** "To" → *Manage phone number list*. Add up to 5 friends who agreed. Each confirms with a code that arrives on WhatsApp.
4. **Permanent token:**
   - business.facebook.com → Settings → Users → *System users* → Add (Admin).
   - *Assign assets*: your app and your WhatsApp account, full control.
   - *Generate token*: your app, never expires, permissions `whatsapp_business_messaging` and `whatsapp_business_management`.

   The 24-hour token on API Setup also works for a first try.
5. **App secret:** App settings → Basic → App secret → Show.
6. **Verify token:** a long random string you make up, for example the output of `openssl rand -hex 24` on your Mac.

## 3. Create the six templates (WhatsApp Manager → Message templates → Create template)
- Category **Utility**.
- Use the name and language exactly as below.
- Paste the body exactly, with its line breaks.
- Enter the sample values.
- Submit. Meta usually answers within minutes to a day.
- Whether Meta accepts the Punjabi and Hindi language codes is still UNKNOWN. If Meta refuses one, tell me the exact message.

```text
========================================================================
Name: kisaniti_sale_v2    Language: English (en)    Category: Utility
Review status in the product: UNREVIEWED
Body (copy exactly, including the line breaks):

Namaste {{1}}. Your FPO manager has approved this plan for your {{2}} basmati paddy:
{{3}}
If anything looks wrong, please contact your FPO office.

Sample values:
  {{1}} = Gurpreet Singh
  {{2}} = 1121
  {{3}} = • 1250.000 kg at Mansa APMC on 1 December 2026 ; • 400.050 kg at Bathinda APMC on 3 December 2026
========================================================================
Name: kisaniti_sale_v2    Language: Punjabi (pa)    Category: Utility
Review status in the product: UNREVIEWED
Body (copy exactly, including the line breaks):

ਸਤਿ ਸ੍ਰੀ ਅਕਾਲ {{1}} ਜੀ। ਤੁਹਾਡੇ FPO ਮੈਨੇਜਰ ਨੇ ਤੁਹਾਡੇ {{2}} ਬਾਸਮਤੀ ਝੋਨੇ ਲਈ ਇਹ ਯੋਜਨਾ ਮਨਜ਼ੂਰ ਕੀਤੀ ਹੈ:
{{3}}
ਜੇ ਕੁਝ ਗ਼ਲਤ ਲੱਗੇ ਤਾਂ ਕਿਰਪਾ ਕਰਕੇ ਆਪਣੇ FPO ਦਫ਼ਤਰ ਨਾਲ ਸੰਪਰਕ ਕਰੋ।

Sample values:
  {{1}} = Gurpreet Singh
  {{2}} = 1121
  {{3}} = • 1250.000 ਕਿਲੋ, Mansa APMC ਵਿੱਚ, 1 ਦਸੰਬਰ 2026 ਨੂੰ ; • 400.050 ਕਿਲੋ, Bathinda APMC ਵਿੱਚ, 3 ਦਸੰਬਰ 2026 ਨੂੰ
========================================================================
Name: kisaniti_sale_v2    Language: Hindi (hi)    Category: Utility
Review status in the product: UNREVIEWED
Body (copy exactly, including the line breaks):

नमस्ते {{1}} जी। आपके FPO मैनेजर ने आपके {{2}} बासमती धान के लिए यह योजना मंज़ूर की है:
{{3}}
अगर कुछ गलत लगे तो कृपया अपने FPO कार्यालय से संपर्क करें।

Sample values:
  {{1}} = Gurpreet Singh
  {{2}} = 1121
  {{3}} = • 1250.000 किलो, Mansa APMC में, 1 दिसंबर 2026 को ; • 400.050 किलो, Bathinda APMC में, 3 दिसंबर 2026 को
========================================================================
Name: kisaniti_cancel_v2    Language: English (en)    Category: Utility
Review status in the product: UNREVIEWED
Body (copy exactly, including the line breaks):

Namaste {{1}}. The earlier plan for your {{2}} basmati paddy has been cancelled. Please do not act on it:
{{3}}
Please contact your FPO office for details.

Sample values:
  {{1}} = Gurpreet Singh
  {{2}} = 1121
  {{3}} = • 1250.000 kg at Mansa APMC on 1 December 2026 ; • 400.050 kg at Bathinda APMC on 3 December 2026
========================================================================
Name: kisaniti_cancel_v2    Language: Punjabi (pa)    Category: Utility
Review status in the product: UNREVIEWED
Body (copy exactly, including the line breaks):

ਸਤਿ ਸ੍ਰੀ ਅਕਾਲ {{1}} ਜੀ। ਤੁਹਾਡੇ {{2}} ਬਾਸਮਤੀ ਝੋਨੇ ਦੀ ਪਹਿਲਾਂ ਵਾਲੀ ਯੋਜਨਾ ਰੱਦ ਹੋ ਗਈ ਹੈ। ਕਿਰਪਾ ਕਰਕੇ ਇਸ ਮੁਤਾਬਕ ਕੁਝ ਨਾ ਕਰੋ:
{{3}}
ਵੇਰਵੇ ਲਈ ਆਪਣੇ FPO ਦਫ਼ਤਰ ਨਾਲ ਸੰਪਰਕ ਕਰੋ।

Sample values:
  {{1}} = Gurpreet Singh
  {{2}} = 1121
  {{3}} = • 1250.000 ਕਿਲੋ, Mansa APMC ਵਿੱਚ, 1 ਦਸੰਬਰ 2026 ਨੂੰ ; • 400.050 ਕਿਲੋ, Bathinda APMC ਵਿੱਚ, 3 ਦਸੰਬਰ 2026 ਨੂੰ
========================================================================
Name: kisaniti_cancel_v2    Language: Hindi (hi)    Category: Utility
Review status in the product: UNREVIEWED
Body (copy exactly, including the line breaks):

नमस्ते {{1}} जी। आपके {{2}} बासमती धान की पहले वाली योजना रद्द हो गई है। कृपया उसके अनुसार कुछ न करें:
{{3}}
विवरण के लिए अपने FPO कार्यालय से संपर्क करें।

Sample values:
  {{1}} = Gurpreet Singh
  {{2}} = 1121
  {{3}} = • 1250.000 किलो, Mansa APMC में, 1 दिसंबर 2026 को ; • 400.050 किलो, Bathinda APMC में, 3 दिसंबर 2026 को
========================================================================
Server setting, once Meta has approved all six:
KN_WA_TEMPLATES={"SALE_INSTRUCTION.v2.en":"kisaniti_sale_v2:en","SALE_INSTRUCTION.v2.pa":"kisaniti_sale_v2:pa","SALE_INSTRUCTION.v2.hi":"kisaniti_sale_v2:hi","CANCELLATION.v2.en":"kisaniti_cancel_v2:en","CANCELLATION.v2.pa":"kisaniti_cancel_v2:pa","CANCELLATION.v2.hi":"kisaniti_cancel_v2:hi"}
```

## 4. On the server (after the domain works and I have deployed)
Run these on your Mac (Terminal):
```bash
ssh -i ~/.ssh/kisaniti_azure kisaniti@172.128.33.73
```
```bash
nano ~/kisaniti/.env
```
Add these lines at the end with your own values. Keep the single quotes around the JSON. Save with Ctrl+O, Enter, then exit with Ctrl+X.
```text
KN_WA_TOKEN=<permanent token>
KN_WA_PHONE_NUMBER_ID=<phone number ID>
KN_WA_API_VERSION=<for example v23.0>
KN_WA_APP_SECRET=<app secret>
KN_WA_VERIFY_TOKEN=<your random verify token>
KN_WA_TEMPLATES='<the JSON from the last line printed above>'
```
Then tell me "WhatsApp settings are on the server". I restart the API without seeing the values. A deploy changes only the `KN_TAG` line, so these lines stay.

## 5. Webhook (Meta app → WhatsApp → Configuration)
- Callback URL: `https://app.kisaniti.in/api/v1/webhooks/whatsapp`.
- Verify token: the same one as `KN_WA_VERIFY_TOKEN`. Then *Verify and save*.
- Webhook fields: subscribe to **messages**. This gives delivery statuses and replies.

## 6. In KisanNiti
For each friend, a fictional farmer:
- enter the friend's number under Contacts;
- record WhatsApp consent (how they agreed, and the notice version).

Then: approve a plan → Messages → **Prepare the messages** → **Send through WhatsApp**. The status changes by itself (queued → sent → delivered → read). A friend's reply appears under Replies, labelled "from the farmer on WhatsApp". That is Gate 4.
