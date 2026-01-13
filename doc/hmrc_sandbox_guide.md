# HMRC Sandbox Setup and Troubleshooting Guide

This guide captures the sandbox setup steps and fixes for common issues encountered while authorising and fetching VAT obligations with the `uk_vat` app.

## 1. Prerequisites
- HMRC Developer Hub test application with client ID/secret.
- VAT test user (from Developer Hub) with its assigned VRN (9 digits). Example format: `110061741` → store as `GB110061741` in ERPNext.
- ERPNext/Frappe bench with `uk_vat` installed.

## 2. Configure HMRC API Settings (sandbox)
1. Open **HMRC API Settings**.
2. Enable the API and set client ID/secret for the sandbox app.
3. In **Advanced**, set `api_base` to the sandbox endpoint: `https://test-www.tax.service.gov.uk`.
4. Save and click **Test HMRC Connectivity**.

## 3. Set the Company VAT number (Tax ID)
- In **Company**, set **Tax ID** to `GB` + 9 digits from the sandbox test user (no spaces). Example: `GB110061741`.
- Save after editing. The code now validates: must start with `GB`, length 11, last 9 chars all digits.

## 4. Register the redirect URI in HMRC Developer Hub
- Redirect URI (decoded): `https://erpdashingdisty.com/?cmd=uk_vat.uk_vat_return.doctype.hmrc_authorisations.hmrc_authorisations.hmrc_callback`
- Redirect URI (URL-encoded for the auth request): `https%3A%2F%2Ferpdashingdisty.com%2F%3Fcmd%3Duk_vat.uk_vat_return.doctype.hmrc_authorisations.hmrc_authorisations.hmrc_callback`
- The trailing slash before `?` matters. HMRC compares byte-for-byte with what is registered.

## 5. Authorise HMRC access (sandbox)
1. Go to **HMRC Authorisations** and create a record for the Company.
2. Click **Request above authorisations** and complete the HMRC OAuth flow.
3. Ensure status becomes **Authorised**. Tokens are tied to the VRN used during authorisation; if you change the Company Tax ID, re-authorise.

## 6. Fetch open obligations from HMRC
- On **UK VAT Return**, click **Fetch Open Returns from HMRC**.
- Requires: valid VRN in Company, sandbox token for that VRN, and `api_base` pointing to sandbox.

## 7. Common errors and fixes
- `redirect_uri is invalid`: Redirect URI registered in Developer Hub does not exactly match the decoded URI above (often missing the `/` before `?`). Register the exact URI and retry auth.
- `The provided VRN is invalid`: The VRN in Company does not match the sandbox test user VRN, or the token was issued for a different VRN. Set the correct `GB`+9-digit Tax ID, save, and re-authorise.
- `AttributeError: 'NoneType' object has no attribute 'upper'`: Company Tax ID was empty. Set the Tax ID correctly and save.
- HMRC message without `obligations` key: HMRC returned an error payload; check the message and ensure connectivity, VRN, and authorisation.

## 8. Checklist for sandbox success
- [ ] `api_base` = `https://test-www.tax.service.gov.uk`.
- [ ] Company Tax ID = `GB` + sandbox VRN (no spaces, 11 chars total).
- [ ] Redirect URI registered exactly as decoded above.
- [ ] Authorisation status = Authorised after running the OAuth flow with the same Company/VRN.
- [ ] “Fetch Open Returns” succeeds and shows obligations.

## 9. If issues persist
- Re-run authorisation after any change to Tax ID.
- Verify the test user VRN in Developer Hub → Test users → VAT user details.
- Ensure HTTPS for the redirect domain is valid and not redirecting to another host/path.
- Check browser allows the popup during the authorisation flow.
