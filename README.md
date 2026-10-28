# Family Document Vault

A phone-friendly app for tracking a family's immigration and ID documents (passports, green cards, I-797s, I-94s, EAD cards and driver's licenses) and getting renewal alerts 3 months before anything expires.

## What it does

- **Family groups**: people are grouped by family; records with the same first and last name in a family are treated as one person.
- **Per person**: first name, last name, date of birth, place of birth, and any number of documents (type, ID number, issued by, expiry date).
- **Renewal alerts**: anything expired or expiring within 3 months stays on the Alerts tab until its expiry date is updated.
- **Fill from a photo**: take or upload a photo of a document and the details are read into the form for review.
- **Weekly email + phone alert**: every Monday at 7:35 AM Central, a scheduled task emails the alert list (with location-based renewal steps and official links) to the addresses saved in Settings, and sends a Claude app notification.

## How it runs

The app is a single self-contained file, `index.html`, published as a private **Claude artifact** on claude.ai. It relies on the claude.ai artifact runtime (`window.claude`) for:

| Feature | Runtime capability |
| --- | --- |
| Saving records, settings and renewal steps | `db` (owner-only access rule on `vault/`) |
| Knowing the signed-in owner | `user` |
| Reading details from photos | `sample` with images |

Opened anywhere else (for example GitHub Pages or a local browser), the page still loads, but it runs in a temporary mode: nothing is saved and photo reading is unavailable.

### Data layout (artifact database)

| Path | Contents |
| --- | --- |
| `vault/main/people/<personId>` | `group, firstName, lastName, dob, pob, documents[{id, type, customType, issuedBy, number, expiry, noExpiry}]` |
| `vault/settings` | `emails[]`, `location{city, state, zip}` |
| `vault/main/guidance/<personId>__<documentId>` | renewal steps written by the weekly task: `summary, steps[], links[], location, expiry, checkedAt` |

Records stay in the claude.ai artifact database. **No personal data is stored in this repository.**

### Weekly reminder

`scheduled-task/weekly-renewal-check.md` holds the instructions the Monday scheduled task runs (with personal details replaced by placeholders). It reads the vault, researches current renewal steps on official sites for each document that is due, writes them back for the app to show, and sends one Gmail alert.

## Updating the app

1. Edit `index.html`.
2. Republish it to the same artifact (ask Claude to publish the updated file to the vault's artifact link) so the link and saved data stay the same.
3. Commit the change here.

## Document types

| Key | Shown as |
| --- | --- |
| `passport` | Passport |
| `greencard` | Green Card (I-551) |
| `i797` | I-797 Approval Notice |
| `i94` | I-94 Arrival/Departure Record |
| `dl` | Driver's License |
| `ead` | EAD Card (I-766) |
| `other` | custom name |
