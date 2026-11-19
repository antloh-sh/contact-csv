# AAC Contact Converter

A single-page, browser-only tool that converts AAC's `DataListing-AACMembership.csv` export into a CSV formatted for Google Contacts import. No data leaves the browser — all parsing and conversion happens client-side with [PapaParse](https://www.papaparse.com/).

## Features

- **Filter by Modified Date** — restrict the export to records within a date range.
- **Filter by Membership Status** — choose any combination of `Joined`, `Pending`, `Terminated`, `Withdrawn` (Joined is selected by default).
- **Date filtering on `ModifiedOn`** — the range is inclusive of both the start and end day (time of day is ignored). Records with a blank or unparseable `ModifiedOn` are excluded when a date range is set.
- **Duplicate NRIC scan** — after filtering, the tool groups records by `NRIC` (trimmed, uppercased, whitespace-stripped) and reports any NRIC appearing more than once. Only the first occurrence of each duplicated NRIC is kept in the export; later duplicates are dropped. Blank NRICs are reported but never treated as duplicates of each other.
- **Status tagging** — non-`Joined` contacts get a `[Status]` suffix on their name (e.g. `John Tan [Terminated]`) so they're easy to spot after import.
- **Postal code routing** — assigns a centre suffix (`@SN`, `@JS`, `@MG`, `@PL`, or `@OB`) based on postal code ranges.
- **Client-side only** — the CSV is processed entirely in-browser; nothing is uploaded to a server.

## Usage

1. Open `index.html` in a browser.
2. (Optional) Set a "Modified From" / "Modified To" date range.
3. Select the membership statuses to include.
4. Click the upload area and select `DataListing-AACMembership.csv`.
5. Review the record count and the duplicate NRIC summary.
6. Click **Download Google Contacts CSV**.
7. Import the downloaded file into Google Contacts.

The status banner walks through each stage: *Reading and parsing file…* → *Scanning for duplicates by NRIC…* → *Scan complete* (with record and duplicate counts).

## Input

The source file must be the raw export from the AAC membership system, `DataListing-AACMembership.csv`, with headers intact (`MembershipStatus`, `FullName`, `Salutation`, `ToAddressAs`, `PersonAddressPostalCode`, `MobilePhone`, `HomePhone`, `NRIC`, `JoinedDate`, `RegistrationDate`, `TerminatedDate`, `WithdrawnDate`, `ModifiedOn`, etc.).

If the NRIC column is named differently in your export (e.g. `PersonNRIC`, `IdNumber`), update the `row.NRIC` references in `findNricDuplicates()` and the dedupe filter in `runTransformation()`.

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

## Duplicate NRIC Handling

- The scan runs on the **filtered** set (after status and date filters), since that's what is about to be exported.
- NRICs are normalised for comparison: leading/trailing whitespace removed, internal whitespace collapsed, and case folded to uppercase. `S1234567A`, `s1234567a`, and ` S1234567A ` are treated as the same person.
- **Default policy: keep the first occurrence** of each duplicated NRIC and drop later ones. This lets the download proceed without manual intervention.
- If you would prefer to **block the download** until duplicates are resolved, remove the `deduped` filter in `runTransformation()`, map over `filtered` as before, and set `actions.classList.add('hidden')` whenever `duplicates.size > 0`.
- Blank NRICs are counted and reported but never flagged as duplicates of each other; those rows are always kept.

## Notes

- All processing happens locally in the browser; the file is never transmitted anywhere.
- **NRIC is sensitive personal data.** The duplicate list — including names and NRICs — is rendered only in your browser and is never uploaded, logged, or persisted. Close the tab or click **Reset App** to clear it.
- Update the postal code ranges in `getSuffix()` if new centres are added.