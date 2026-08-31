# AAC Contact Converter

A single-page, browser-only tool that converts AAC's `DataListing-AACMembership.csv` export into a CSV formatted for Google Contacts import. No data leaves the browser — all parsing and conversion happens client-side with [PapaParse](https://www.papaparse.com/).

## Features

- **Filter by Modified Date** — restrict the export to records within a date range.
- **Filter by Membership Status** — choose any combination of `Joined`, `Pending`, `Terminated`, `Withdrawn` (Joined and Pending are selected by default).
- **Status-aware date filtering** — the date range is checked against the date field relevant to each status (`JoinedDate`, `RegistrationDate`, `TerminatedDate`, `WithdrawnDate`), falling back to `ModifiedOn` if that field is blank.
- **Status tagging** — non-`Joined` contacts get a `[Status]` suffix on their name (e.g. `John Tan [Terminated]`) so they're easy to spot after import.
- **Postal code routing** — assigns a centre suffix (`@SN`, `@JS`, `@MG`, `@PL`, or `@OB`) based on postal code ranges.
- **Client-side only** — the CSV is processed entirely in-browser; nothing is uploaded to a server.

## Usage

1. Open `index.html` in a browser.
2. (Optional) Set a "Modified From" / "Modified To" date range.
3. Select the membership statuses to include.
4. Click the upload area and select `DataListing-AACMembership.csv`.
5. Review the record count, then click **Download Google Contacts CSV**.
6. Import the downloaded file into Google Contacts.

## Input

The source file must be the raw export from the AAC membership system, `DataListing-AACMembership.csv`, with headers intact (`MembershipStatus`, `FullName`, `Salutation`, `ToAddressAs`, `PersonAddressPostalCode`, `MobilePhone`, `HomePhone`, `JoinedDate`, `RegistrationDate`, `TerminatedDate`, `WithdrawnDate`, `ModifiedOn`, etc.).

## Output

A CSV with the following Google Contacts columns:

| Column | Source |
|---|---|
| Name Prefix | `Salutation` |
| First Name | `FullName` (proper case), with `ToAddressAs` and non-Joined `[Status]` tags appended |
| Address 1 - Postal Code | `PersonAddressPostalCode`, zero-padded to 6 digits |
| Name Suffix | Centre code derived from postal code |
| Phone 1 - Value / Type | `MobilePhone` / "Mobile" |
| Phone 2 - Value / Type | `HomePhone` / "Home" |

## Postal Code → Centre Mapping

| Range | Suffix |
|---|---|
| 347467–349979 | `@SN` |
| 360001–362016 | `@JS` |
| 367819–369977 | `@MG` |
| 534068–539746 | `@PL` |
| Other / unrecognised | `@OB` |

## Notes

- All processing happens locally in the browser; the file is never transmitted anywhere.
- Update the postal code ranges in `getSuffix()` if new centres are added.
