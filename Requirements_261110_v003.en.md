# Ceddie — Download Distribution Management System Requirements

[日本語](ダウンロード配布管理システム要件定義_261110_v003.md) | English

- Created: November 9, 2026
- Version: v003
- Status: v003 finalized. This version is the basis for starting development; implementation methods and adjustable values will be refined during development under Section 14.
- Numbering: v001–v009 before development; sequential three-digit versions from v010 after development begins.
- Updated: November 12, 2026
- Project name: Ceddie (セディ)

## Overview

| Area | Key points | Details |
| --- | --- | --- |
| Purpose and scope | Distribute digital content such as doujin works and purchase bonuses using download codes and QR codes. Tracking whether cards have been handed out is outside the scope. | [1. Purpose and scope](#scope) |
| Environment and data | Shared hosting / VPS, PHP + SQLite, desktop and smartphone support | [2. Environment and data management](#environment) |
| Installation | Web-based setup; administration remains unavailable until the installation folder is removed | [3. Installation](#installation) |
| Item management | Register, edit, delete, and replace files. Renaming or adding codes does not require re-uploading. Per-item distribution suspension and notices. | [4. Item management](#items) |
| Thumbnails | Separate crops: square for administration, A-series paper ratio for recipients. Display widths: 65 px / up to 150 px. | [5. Thumbnails](#thumbnails) |
| Code issuance and conditions | Issue additional random 13-character alphanumeric codes (specification under consideration). Automatically record optionally named issuance groups. Lock when the download limit or expiration is reached, whichever comes first. | [6. Download code issuance and usage conditions](#code-issuance) |
| Code lists and bulk operations | Per-item access counts, download counts, and status. Sort, search, select, disable, delete, reset counts, adjust limits, change expiration, and re-enable. | [7. Download code lists and operations](#code-management) |
| Lists and navigation | Item name, issuance group count, total code count, and cumulative downloads. Icons and a right-side menu. | [8. Item list and page structure](#screens) |
| Receiving and QR codes | Enter a code or use a QR code containing a dedicated URL. Generate QR codes internally. Separate page viewing from downloading. Administrator preview. | [9. Receiving page and QR codes](#receiving) |
| CSV and card printing | Export selected or newly issued codes to CSV; bulk QR output, print-ready card PDFs, and test printing with dummy codes | [10. CSV and card printing](#printing) |
| Print status | Track unprinted / printed per ID. Mark manually after printing; reprinting is always possible. | [11. Print status](#print-status) |
| Logs and IP information | Retain access and download counts. Individual ordinary download history is outside the initial release. Record IP addresses only when excessive access is detected. | [12. Logs and IP information](#history) |
| Security | Mandatory two-factor authentication; HTTPS, session, request, output, and SQL protection. Refine implementation during development. | [13. Basic security requirements](#security) |
| Remaining decisions | Policies finalized; distinguish development adjustments from features outside the initial release | [14. Status and development adjustments](#pending) |
| Backup and restore | Manual and optional automatic backups. Switch between cron and administrator-page access; retain the latest six generations. | [15. Backup and restore](#backup) |
| Deletion | Delete related data; offer CSV export and require staged warnings and final confirmation | [16. Data deletion and pre-deletion CSV export](#deletion) |
| Japanese and English | Initially Japanese and English; add language files after publication. Supplement UTC with a country/region's standard time (English uses UTC only). | [17. Languages](#languages) |
| License | Under consideration | [18. License](#licensing) |

<a id="scope"></a>

## 1. Purpose and scope

- Distribute digital content such as doujin works and purchase bonuses using download codes or QR codes printed on cards or similar media.
- Support item registration, download code issuance and management, card printing, and usage checks.
- Tracking whether cards have been distributed is outside the scope.
- Collecting recipients' email addresses and sending announcement emails are outside the initial release. Integration with an external delivery system is a future option.
- Implement and design the application independently, without reusing the original application's source code or images.

<a id="environment"></a>

## 2. Environment and data management

- Support installation and operation on ordinary shared hosting and VPS environments.
- Use PHP and SQLite as the basic configuration.
- Require PHP 8.3 or later. Recommend a newer, officially supported version that has passed compatibility testing, and use the latest patch release in each supported series.
- PHP 8.3 security support is scheduled to end on December 31, 2027. Review the support policy when publishing or updating the application.
- Do not guarantee compatibility with untested future versions. Verify supported PHP versions and required extensions before publication and document them under specifications and installation requirements.
- Normal operation on shared hosting must not require a persistent background process or server administrator privileges.
- Support desktop and smartphone browsers.
- Use SQLite to manage items, download codes, usage conditions, usage counts, print status, and related data.
- Store distribution files and thumbnails as files, with their relationships managed in the database.
- Generate QR codes from stored information when needed. Storing QR images in the database is not mandatory.
- Finalize required PHP extensions, upload limits, and other specific conditions through development and compatibility testing.

### 2.1 Storage protection and installation requirements

- Store the database, authentication secrets, distribution files, backups, and temporary uploads where they cannot be retrieved directly over the Web.
- Prefer storage outside the public web directory. If this is unavailable, require direct-access denial through .htaccess or server configuration.
- Verify that access restrictions actually work; .htaccess availability alone is insufficient. Design and test the verification method during implementation.
- Check protection conditions in the installer. If they cannot be verified, show the unmet requirements and do not complete installation.
- Ceddie cannot be used where these private storage areas cannot be protected.
- Deliver distribution files through the application after checking the download code and all usage conditions.
- Allow only authenticated administrators to register or replace distribution files.
- Generate and manage storage locations and internal filenames within the application. Do not use uploaded filenames or user-supplied paths directly as storage paths. Manage download filenames separately.
- Do not normally restrict distribution file extensions. Store files privately and never execute or load them as server-side programs.
- Read distribution files as data and deliver them as attachments. Do not execute uploaded PHP or other files through include, require, or external commands.
- Preserve the current distribution file if an upload exceeds limits or transfer/storage fails. Switch the associated information only after the replacement is saved successfully.
- Process display images separately from distribution files; validate supported formats and image content before regenerating them.
- Distinguish prevention of server-side execution from virus scanning: these protections do not guarantee the distributed content itself is harmless.
- Store processed item images and icons in img/, separate from distribution files and secrets.
- Remove temporary uploads after processing completes or fails.
- Clearly state under specifications and installation requirements that protected private storage is mandatory and that unsupported servers cannot run Ceddie.
- Provide examples of storage outside the public directory or effective access-denial configuration, plus troubleshooting checks for failed installation.

<a id="installation"></a>

## 3. Installation

- Provide a browser-based installer for initial setup.
- Check the environment, storage locations, and required write permissions during setup.
- Configure the site name, administrator account, and administrator password, and initialize the database.
- Include a configuration file with an empty initial setup key inside install/. The installer operator fills in and saves the key locally, then uploads the files using FTP or a similar method.
- Proceed with setup only after verifying the configured key. Use a file format and location that prevent Web retrieval of its contents.
- If the key is unset, explain why setup cannot proceed and do not initialize the database or register an administrator. Before setup is complete, uploading a replacement key file must allow setup to resume.
- Keep installation completion state independent of install/ and the key file. Changing the key or re-uploading install/ must not permit setup to run again.
- Invalidate the key after successful setup; deleting install/ must also remove the key file.
- Do not allow the installer to run again after setup is complete.
- Keep administration unavailable while the installation folder exists.
- At setup completion, instruct the operator to remove the installation folder; enable administration after removal.
- Adopt this installer and administration-access restriction as the basic policy for future self-hosted Web applications as well.
- Implement and verify setup-key checking, prevention of reinstallation, and blocking administration while install/ exists at the final development stage. Preserve the ability to test by moving and restoring install/ during development.
- Require these protections in externally distributed releases. Do not distribute a build with development-only protection bypasses enabled.

<a id="items"></a>

## 4. Item management

- Allow items to be registered, edited, and deleted.
- Allow a distribution file to be registered and replaced for each item.
- Configure and change the item name and the filename used when downloading independently.
- Renaming alone must not require re-uploading the distribution file.
- Preserve the existing file if no replacement is selected while editing.
- Show the current filename and file size on the edit screen.
- Changing a name or thumbnail must not change issued download codes.
- Allow additional download codes to be issued without re-uploading the distribution file.

### 4.1 Maintenance mode and temporary distribution suspension

- Allow administrators to suspend and resume distribution per item. Keep this state separate from individual code suspension.
- Reject new downloads for suspended items without increasing used or cumulative download counts.
- Show an administrator-defined notice, such as “A revised version is being prepared,” on the receiving page during suspension.
- Suspension and resumption must not change issued codes, dedicated URLs/QR codes, download limits, used counts, expiration, or individual code suspension.
- After resumption, allow new downloads only when individual code suspension, expiration, and remaining-download conditions all permit them.
- Let transfers already started continue after suspension. Reject new download, retry, and reconnection requests received after the switch.
- Provide system-wide and per-item maintenance modes. Treat per-item maintenance as the same feature as temporary distribution suspension.
- During system-wide maintenance, reject new downloads and retries for all items; during per-item maintenance, reject them for all issuance groups and codes of that item. Do not consume download allowances.
- Allow authenticated administrators to perform file replacements and other administrative work during maintenance.
- Keep system-wide and item-level suspension states independent. Ending system-wide maintenance must not lift item or code suspension, or change limits and expiration dates.

### 4.2 Distribution file replacement

- Preserve existing download codes and dedicated URLs/QR codes after replacement. New downloads must retrieve the latest file.
- Replacement must not change used counts, limits, expiration dates, code suspension, or item distribution suspension.
- To allow recipients to download a revision again, use the limit-increase feature for selected existing codes. Replacing a file alone must not reset every code.
- Let transfers started before replacement continue using the old file. Reject new retry requests for the old file; treat retrieval of the latest version as a new download and check its conditions. Remove the old file only after no active transfer uses it.

<a id="thumbnails"></a>

## 5. Thumbnails

- Allow an arbitrary image to be uploaded for each item.
- Allow items to be registered without a thumbnail.
- Use a 65 × 65 px square as the baseline list thumbnail size, adjustable during prototyping.
- Offer fitting the entire image while preserving its aspect ratio, or cropping a square portion.
- Allow padding when displaying the full image.
- Allow crop position and zoom adjustment.
- Allow zooming out far enough to fit the whole image, with a “Show entire image” action to reset the view.
- Preview the result at its actual list display size.
- Support crop adjustments on smartphones.
- Allow thumbnails to be added or changed independently after item registration.
- Save at a resolution higher than the display size to accommodate display-size changes.
- The initial release does not include side-by-side comparison of existing thumbnails; focus on cropping and previewing the result.

### 5.1 Item images, automatic resizing, and storage

- Generate separate administration-list icons and recipient-page item images from uploaded images.
- Allow administrators to specify two separate crops from one upload: one for recipients and one for the administration list.
- Use an A-series paper aspect ratio (width:height = 1:√2) for recipient images and a square (1:1) for administration icons. Adjust position and zoom without stretching the image.
- Use fixed-ratio crop frames regardless of the input aspect ratio. Do not add a separate cropping workflow triggered by extremely tall or wide images.
- Both frames must support fitting the entire image with padding or cropping a portion. Continue to check input dimensions, file size, and total pixel count.
- Display recipient images at a maximum width of 150 px, deriving height from the cropped aspect ratio.
- Do not enlarge smaller images beyond their native size. Allow padding where an image does not fill the fixed-ratio frame.
- Generate stored item images at dimensions somewhat larger than the 150 px display width, balancing quality and file size. Finalize storage limits separately.
- Automatically reduce images exceeding the limits. Avoid unnecessary enlargement of small originals.
- Control image size in pixels, independently of DPI metadata or physical paper dimensions.
- Changing the administration icon crop must not affect the recipient image.
- Handle the original temporarily and discard the large source image after both generated images and crop results have been saved successfully.
- Keep only processed lightweight images in img/. Do not permanently retain the unprocessed upload; manage associations with items in the database.
- If a later crop cannot be made from saved images after the original has been discarded, prompt for another upload.
- On conversion failure, retain existing registered images, remove temporary images, and show an error.
- Account for orientation metadata in smartphone photographs.
- Set input file-size and total-pixel limits. Explain why an image cannot be processed safely rather than accepting it. Finalize input limits, output formats, and compression quality separately.

<a id="code-issuance"></a>

## 6. Download code issuance and usage conditions

- Issue multiple download codes for one item.
- Add the required number of codes while retaining existing codes.
- Use random alphanumeric codes rather than sequential codes.
- Use 13 characters excluding separators; reconsider the display format during implementation. The final character is also random and is not a check character.
- Prevent duplicate issued codes.
- Use uppercase letters and exclude confusing characters such as 0/O and 1/I. Confirm the final permitted character set during development.
- Match input case-insensitively after removing hyphens and spaces. Validate normalized format and length on the server.
- Generate codes using secure, unpredictable randomness and prevent duplicates throughout the system, including across different items.
- Offer four issuance conditions: download limit only; expiration only with unlimited downloads; both limit and expiration; or unlimited downloads without expiration.
- Specify unlimited downloads and no expiration through explicit choices. Do not use zero or negative numbers to mean unlimited.
- If both unlimited downloads and no expiration are selected, confirm that there are no count or time restrictions.
- Record used and cumulative download counts even for unlimited codes, and enforce manual suspension. Enforce expiration if configured.
- When both a limit and expiration apply, stop availability as soon as either condition is reached.
- Reject negative, fractional, or out-of-range issuance quantities and download limits.
- Explain invalid inputs and allow correction.
- Do not implement Levenshtein-distance exclusion of similar codes in the initial release.

### 6.1 Issuance group management

- Automatically record each additional issuance operation as one group within its item.
- Each issuance group belongs to exactly one item and never spans items. Identically named groups for different items remain separate groups.
- Allow an optional administrator-entered group name; issuing without a name must be possible.
- Use the issuance date and time as the display name when no name is provided.
- Allow names to be added or changed later.
- Use unique internal IDs so matching names or timestamps cannot cause confusion.
- Normally show all codes for an item, with optional filtering by issuance group.
- Allow printing, CSV export, and disabling to target issuance groups.
- Do not use issuance groups to track whether cards have been distributed.

<a id="code-management"></a>

## 7. Download code lists and operations

- Provide a download code list for each item.
- Show the item name and thumbnail at the top of the page.
- Make the current item and screen identifiable from navigation context and the browser tab title.
- Give each code an ID that does not change when sorting or paging.
- Normally display zero-padded three-digit IDs; increase the digit count for 1,000 or more entries.
- Use ID, download code, and downloads used / limit as the basic columns.
- Show access count, expiration, status such as enabled/disabled or limit reached, and print status.
- Sort by ID, download code, and actual download count in either direction.
- Default to ascending ID order.
- Search codes and filter by unused, downloaded, expired, limit reached, disabled, and similar states.
- Search all codes for the item, not just the visible page.
- Add a checkbox to each row, including unused codes.
- Select or deselect all entries on the current page and show the selected count.
- Print, export CSV, delete, disable, reset download counts, and increase or decrease download limits for selected codes.
- Visually distinguish hovered rows and selected rows.
- Resetting code usage must not erase item cumulative downloads, administrator logs, or excessive-access logs.
- Show the item name and target count when confirming bulk operations.

### 7.1 Adding and changing download allowances

- Increase or decrease download limits individually or in bulk, only for checked codes.
- Offer “+1 download” and adjustment by an arbitrary integer.
- Do not change used downloads or item cumulative downloads. If ordinary download history is introduced later, preserve it as well.
- For limited codes, calculate remaining downloads as limit minus used downloads, with a minimum of zero. Show “Unlimited” for unlimited codes.
- Allow filtering by unused/downloaded status or issuance group before selection. Do not affect unselected codes.
- Before execution, show the item, target count, and each code's previous and resulting limit and remaining allowance.
- If the limit is reduced to or below the used count, set remaining downloads to zero and block downloads.
- Require a positive integer when setting a limit. Reject adjustments producing negative, fractional, or out-of-range limits.
- Record the change time, affected codes, adjustment amount, previous and resulting limits, and an optional reason.
- Adding downloads must not remove expiration or disabling.
- Allow issued codes to switch between limited and unlimited downloads.
- Preserve used and cumulative counts when switching to unlimited.
- When switching from unlimited to limited, require a new positive integer limit. If usage already meets or exceeds it, reject new downloads with zero remaining.
- Do not apply adjustments such as “+1” to unlimited codes; show that they are excluded because they are unlimited. Identify excluded codes in bulk-operation confirmation.
- Switching between limited and unlimited must not change expiration, code suspension, item suspension, or print status.
- Confirm before/after conditions and log the time, codes, and changed conditions. If the result is unlimited with no expiration, also confirm the absence of restrictions.

### 7.2 Changing expiration and suspending or re-enabling codes

- Change expiration for selected issued codes individually or in bulk.
- Allow manual suspension (disabling) and re-enabling of selected codes.
- Manage manual suspension, expiration, and limit reached as separate, visible states.
- Keep download limits, expiration, and manual suspension independent.
- Suspending or re-enabling must not change previously increased limits, usage, cumulative downloads, or expiration.
- Allow limits to be adjusted while suspended, without lifting suspension.
- Changing expiration must not alter limits, usage, cumulative downloads, or manual suspension.
- Re-enabling does not permit downloads if a code is expired or has no remaining downloads. Extending expiration does not permit downloads if no allowance remains or the code is suspended.
- Allow new downloads only when item and code suspension are absent and all configured expiration and count conditions are met.
- Resetting download counts must not change expiration or manual suspension.

### 7.3 Expiration specification and evaluation

- Specify and store expiration as a UTC date and time.
- In Japanese, show UTC and Japan Standard Time (UTC+9), including the correct date when it changes. In English, show UTC.
- Reject new downloads once the current time is equal to or later than expiration.
- Example: 2026-12-31 15:00 UTC is a deadline of 2027-01-01 00:00 in Japan.
- Change expiration through individual/bulk code editing; resetting counts or adding allowance must not change it.
- After code expiration, allow retries of an already-started download if its retry information remains valid and no suspension, deletion, or file replacement has occurred.

<a id="screens"></a>

## 8. Item list and page structure

- Show thumbnails and item names in the item list.
- Display item-level information in this order: item name, issuance group count, total code count, cumulative downloads.
- Include all issuance groups belonging to the item in those counts.
- Treat cumulative downloads as a total that does not decrease when code usage is reset. Follow Section 16 for counts after deletion.
- Provide access to editing, code management, and card printing from each item.
- Consider a right-side administration pane as the basic menu layout.
- On smartphones, consider a menu that opens and closes from the right.
- Place list operations near the relevant list.
- Allow the number of entries per page to be changed.
- Initially default to 15 entries, with choices of 10, 15, 30, 50, and 100.
- If prototyping shows that 15 is too many, change the default to 10 and choices to 10, 20, 30, 50, and 100.
- Provide pagination.
- Preserve sorting and filtering when changing page size or navigating pages.
- Retain separate page-size settings for item lists and code lists.

<a id="receiving"></a>

## 9. Receiving page and QR codes

- Allow files to be downloaded by entering a download code.
- Generate a dedicated URL for each code so QR scanning can skip manual entry.
- Generate QR codes within the system, without sending download codes to an external QR service.
- Encode the site's dedicated URL directly in each QR code.
- Opening a receiving page through a QR code must not consume a download. Require a separate download action.
- Use a URL structure that keeps printed QR codes usable.
- Ensure each printed code matches its QR code destination.
- Viewing the receiving page must not consume a download. Increment used downloads and item cumulative downloads once when the server starts delivery in response to the download action.
- Before delivery, validate the code, suspension, expiration, allowance, and file existence/readability. Failures before delivery must not consume an allowance.
- Retries, resumed transfers, and range requests belonging to the same download must not consume additional allowances. Starting a new download counts separately.
- Even when the initial request used the final allowance, allow verified resumptions and range requests for that same download without additional consumption.
- Keep short-lived information identifying the same download separate from ordinary download history.
- Provide an administrator setting for the download retry window: 30, 60, 120, 180, or 360 minutes, defaulting to 30 and capped at 360.
- Count the retry window from the first delivery start; retries and resumptions must not extend it.
- Verified retries within the window consume no additional allowance and bypass the per-code five-minute wait.
- After retry information expires, treat a new request as a new download and check allowance, expiration, and suspension. Do not terminate an active transfer merely because the retry window expires during it.
- Reject subsequent retry requests after code suspension, item suspension, system maintenance, or deletion. After replacement, reject retries for the old file. Apply restrictions to requests received afterward, without forcibly disconnecting transfers already started.
- Do not automatically refund a count after delivery starts and the connection fails, because successful saving on the recipient device cannot be reliably determined. Use same-download retries; otherwise an administrator can increase the affected code's allowance.
- Check and consume allowances atomically so concurrent requests cannot exceed limits, and do not count the same download twice.
- Allow guidance on the receiving page and cards explaining that the allowance is intended for the purchaser's own devices and re-downloads.

### 9.1 Per-code re-download waiting period

- Allow the five-minute wait to be enabled or disabled per code at issuance and afterward.
- When enabled, require five minutes from the start of one new download before starting another new download using the same code. This choice also applies to unlimited codes.
- Allow the wait to be disabled for uses such as time-limited public distribution with one shared code. Retain limits, expiration, suspension, and excessive-access protections.
- Default the wait to enabled. Administrators may explicitly disable it for shared distribution. Show waiting notices only when enabled.
- Show guidance such as “You can start another download with this code five minutes after the previous download started.”
- Failures before delivery starts must neither consume an allowance nor trigger the wait.
- Handle failures during delivery through verified same-download retries, resumptions, or range requests without a wait or extra consumption.
- Distinguish new downloads from retries; entering the same code alone does not establish a retry. Design identification during development, using the configured retry window.
- Do not automatically remove the wait or refund counts solely on a recipient's failure report, because the server cannot reliably determine successful saving. Provide same-download retries instead.
- Rejected new-download requests during the wait must not consume an allowance or extend the wait.
- Same-download retries do not roll back usage; they remain within the single download already counted.

### 9.2 Receiving page preview

- Allow administrators to preview the page recipients see after entering a code or scanning a QR code.
- Preview the item name, image, download filename, guidance, download button, and similar elements before publication or after changes.
- Keep this separate from the printed-card preview.
- Include previewing item suspension notices.
- Do not consume production codes or increase production access/download counts.
- Clearly identify preview mode and restrict it to authenticated administrators.

### 9.3 Unavailable-code messages

- Check availability on manual code entry, dedicated URL/QR access, download requests, and retries. Explain unavailability according to its cause.
- Provide Japanese and English messages in language files. Initial message meanings are:
  - Suspended code: “This download code is currently unavailable.”
  - Suspended item: “Distribution of this item is temporarily suspended.” An administrator notice may be added.
  - System maintenance: “Downloads are unavailable during maintenance. Please try again later.”
  - Deleted or never-issued code: “Download code not found. Please check your entry.” Do not require retained deletion history.
  - Expired code: “This download code has expired.”
  - Limit reached: “This download code has reached its download limit.”
- Apply the separate retry conditions to the same download. Do not incorrectly reject an allowed retry based only on the code's count limit or expiration.

<a id="printing"></a>

## 10. CSV and card printing

- Export an item's download codes to CSV in bulk.
- Allow CSV export and printing of selected codes only.
- Allow output of only the codes added in the current issuance operation.
- Export QR codes in bulk.
- Place item names, images, download codes, QR codes, and usage guidance on cards.
- Generate print-ready PDFs with cards arranged on a sheet.
- Allow cutting lines where needed.
- Consider a choice of faint cutting lines, crop marks, or no guides (not finalized).
- Consider light gray cutting lines placed clear of names, QR codes, and other content.
- Decide whether to include guides, their darkness and thickness, crop-mark placement, and required margins during later print-layout design.
- Refer to previous distribution cards when deciding the print design separately.
- Fix the initial paper size at A4 and recommend full-sheet, free-cut label paper. Also support ordinary A4 paper.
- Document using printer-compatible paper and printing at actual size / 100%.
- Refine card dimensions, physical QR size and margins, and layout through development and prototyping.
- Initial layouts are four approximately A6 cards (two columns × two rows) or eight approximately A7 cards (two columns × four rows) on A4.
- Allow actual card sizes to be slightly smaller than A6/A7 because of surrounding margins.
- Treat these layouts as provisional and revisit them after checking content volume, readability, and actual printed output.
- Verify QR readability at the production print size.

### 10.1 Test printing

- Allow test printing with dummy codes without issuing or consuming production codes.
- Use the production layout to check text sizes, margins, cutting positions, and QR readability.
- Clearly label test cards “Test print” to distinguish them from production cards.
- Test QR codes must open a dedicated verification page, not retrieve production distribution files.
- Test printing and test QR access must not affect production print status, access counts, or download counts.

### 10.2 Saving print settings

- Initially support saving and reusing adjustments to the provided A4 layouts: four approximately A6 cards or eight approximately A7 cards.
- Refine saved properties, adjustment ranges, and controls, including margins and text/QR placement, through development and prototyping.
- A free-form design editor is not mandatory for the initial release.
- Keep layout settings separate from selected items/codes. Loading settings alone must not issue codes or update print status.

<a id="print-status"></a>

## 11. Print status

- Track unprinted / printed status per ID.
- Allow filtering for unprinted codes.
- After actual printing, allow selected IDs to be marked printed together.
- Generating a PDF or viewing the print page alone must not automatically mark codes printed.
- Allow previously printed codes to be reprinted without a reprint limit.
- Explicitly state that printed codes can be reprinted.
- Print status must not affect download availability or remaining allowance.
- Do not track whether cards have been distributed.

<a id="history"></a>

## 12. Logs and IP information

- Show access and download counts per code.
- Count receiving-page views using valid codes as accesses. Repeated views and refreshes by the same person also count; this is not a count of people or unique users.
- Exclude administrator previews and test access from ordinary access counts.
- Keep access-count display/aggregation separate from abuse-prevention limits. Permit multiple people to use a shared code.
- Detect repeated invalid-code entry and excessive requests over short periods and temporarily restrict them. During development, verify criteria and thresholds considering the source and target code without disrupting normal shared-code usage.
- Disabling the per-code five-minute wait must not disable excessive-access detection or attempt limits. Rejected requests that never start delivery must not consume downloads.
- Defer individual ordinary download history, including download timestamps, from the initial release. Revisit if needed; primarily handle inquiries through code search, sorting, and filtering.
- Do not store IP addresses for ordinary access or downloads.
- Treat IP information needed for excessive-access detection as temporary management data.
- Record IP addresses when short-term excessive access is detected.
- Include timestamp, target operation, attempt count, and restriction result in detection records.
- Automatically remove detection records after their retention period.
- Retain administrator operation logs for 90 days and excessive-access records for 30 days.
- For administrative changes, deletion, recovery, and similar operations, record timestamp, administrator ID, target, changes, and result. Do not record IP addresses for ordinary administrative operations.
- For excessive access, record detection time, IP, target operation, attempts, and restriction result.
- Never log actual passwords, authenticator codes/secrets, recovery codes, or session identifiers.
- Delete expired records on the next authenticated administrator-page access. Document that deletion is delayed if the administration interface is not accessed.
- Separate retained logs from short-term information used for throttling. Log expiration must not incorrectly change restrictions or cumulative item downloads.
- Refine detection criteria and other system-error log fields during development.

<a id="security"></a>

## 13. Basic security requirements

- Store administrator passwords using a salted password-hashing scheme, never as plaintext.
- Protect administrator authentication, authorization, sessions, operations, and repeated attempts.
- Keep the database and distribution files inaccessible directly over the Web.
- Enforce download limits even under concurrent access.
- Validate issuance quantities, counts, storage locations, uploaded content, and other inputs on the server.
- Require the following protection policies; refine methods, values, and verification during development.
- Require HTTPS for public operation, including the installer. Never send setup keys, passwords, or authentication data in plaintext transport. Clearly separate local testing exceptions from public releases.
- Use Secure and HttpOnly authentication cookies and configure SameSite and related protections. Rotate session identifiers after authentication to prevent impersonation through pre-authentication identifiers.
- For state-changing operations such as deletion and settings updates, validate authentication, permissions, and CSRF protection on the server. A URL visit alone must not execute them; require the appropriate change request.
- Escape names, notices, and other output for their destination context so input cannot execute scripts. Restrict any permitted notice formatting.
- Use parameterized SQL. Do not concatenate inputs directly into SQL statements. Allowlist values such as sorting fields.
- Require additional identity verification for sensitive changes such as passwords, two-factor settings, and recovery codes. Use the dedicated recovery procedure rather than allowing normal operations without authentication.
- Show concise user-facing errors without internal paths, SQL, secrets, or detailed exceptions. Record necessary diagnostics privately.

### 13.1 Administrator accounts and future expansion

- Require one administrator account managing all of Ceddie for the initial release.
- Set the administrator ID and password during installation and allow later changes after login.
- Allow administrators to manage all items, codes, and settings. Recipients do not need accounts; they use download codes.
- Allow concurrent login from multiple devices using the same account. New logins must not automatically log out existing devices.
- After a password change, invalidate that account's sessions except the session making the change. Other devices must log in again before their next administrative operation.
- Provide “Log out of all devices,” invalidating every session for the account, including the current one.
- Normal logout must invalidate only the current session.
- Offer administrator idle timeouts of 30, 60, and 120 minutes, defaulting to 30.
- Expire sessions after the selected idle period and require login for the next operation. Enforce this server-side per session even if a cookie remains.
- Refine what counts as activity, handling of lengthy input/uploads, and preserving input across reauthentication during development.
- Give administrators unique database IDs and associate audit records with them, allowing future multi-administrator support.
- Additional accounts, account management, and differentiated permissions are not mandatory in the initial release.
- Decide whether and when to add multiple administrators at the application's β-1 release.
- Normally consider this for application v1.0.1 or later, but earlier inclusion in v1.0.0 is allowed if appropriate. Do not make its completion a prerequisite for v1.0.0.
- These application versions (β-1, v1.0.0, v1.0.1) are distinct from requirements-document versions such as v003 and v010.

### 13.2 Failed login restrictions and administrator recovery

- Temporarily restrict sources with concentrated login failures and release restrictions after time elapses. Apply behavior-based rules equally to administrator mistakes and third-party attempts.
- Do not lock the entire account solely because one source fails, thereby excluding administrators connecting from elsewhere.
- Use uniform errors such as “Incorrect ID or password” without revealing whether an ID is registered.
- Temporarily restrict each source after five failed logins. After release, another five failures escalate the restriction stage.
  - From the fourth stage onward, repeat the maximum restriction duration after each further five failures following release. The policy is not to impose a 24-hour restriction.
- Do not process requests during restriction as authentication attempts, increment failures/stages, or extend the release time.
- After successful login, reset that source's failure count, stage, and restriction duration. Start future failures at the first stage; do not release restrictions on other sources.
- Because restricted sources cannot attempt authentication, success-based resets occur only on a normal login after release.
- Refine the failure-counting window and stage retention/expiration other than successful login during development.
- Use two-line restriction messages: “The login attempt limit has been reached.” and “You cannot log in for [N] minutes.” Substitute the applicable stage duration. Provide corresponding Japanese and English text in language files.
- The displayed duration explains the stage's restriction period; redisplaying it must not extend the deadline. Do not show a countdown.
- Record IP, timestamp, attempt count, and restriction result when excessive attempts are detected. Retain for 30 days under Section 12.
- Do not collect administrator email addresses or provide recovery email in the initial release.
- Issue a single-use account recovery code during installation and ask the administrator to store it securely.
- Associate recovery codes with administrator accounts so future multiple administrators can be handled independently.
- Follow Section 13.3 for recovery using a valid code. Password reset alone must not issue a normal administrator session that bypasses two-factor authentication.
- After password reset, invalidate all existing sessions for the account and require login with the new password.
- Generate unpredictable recovery codes securely; store only verification hashes in the database and throttle recovery-code attempts.
- Keep password recovery and time-based release of source restrictions separate. Using a recovery code alone must not lift a source restriction.

### 13.3 Administrator two-factor authentication

- Require authenticator-app two-factor authentication from the initial release. Do not use email authentication.
- After checking the administrator ID and password, verify a time-based one-time password (TOTP). Grant ordinary administration access only after both succeed.
- During initial setup, enroll an authenticator and verify it with a code. Restrict enrollment operations to those needed for setup.
- Password success alone is not a successful login and must not reset failed-login restrictions.
- Independently limit authenticator-code attempts and validity periods. Apply controls per account or in-progress authentication flow to address attempts switching IPs. Refine values during development.
- Changing or resetting a password must not automatically disable enrolled two-factor authentication.
- If the authenticator device is lost, support: verify administrator ID plus a valid single-use recovery code → set a new password → enroll and verify a new authenticator → normal login.
- During recovery, allow only dedicated setup screens, with no access to items, codes, or normal administration settings.
- Prevent concurrent or duplicate recovery using the same code from the moment verification succeeds. Define resumption/expiration of interrupted enrollment during development.
- On recovery completion, invalidate the old password, authenticator secret, sessions, and used recovery code; issue a new recovery code.
- Do not open ordinary administration until the new authenticator has been verified.
- Continue requiring two-factor authentication after recovery; recovery must not create a permanent authentication bypass.
- Refine secret protection, prevention of code reuse, tolerated clock drift, and related implementation details during security design and testing.

<a id="pending"></a>

## 14. Status and development adjustments

### 14.1 Finalized policies

- This version finalizes the initial feature scope and basic security policies. Treat implementation details and numerical values as development adjustments below.
- Before changing agreed defaults or usage conditions, check related requirements and document the change in requirements v010 onward and the development history.
- Handle publication and licensing conditions separately under Section 18, independently of functional requirements finalization.

### 14.2 Matters to refine during development and prototyping

- Actual dimensions, margins, text/QR sizes, card design, and saved adjustment values/controls for four approximately A6 or eight approximately A7 cards on A4.
- Whether and where to provide faint cutting lines, crop marks, or no guides.
- Stored image dimensions, input file/pixel limits, formats, and compression quality.
- Undecided limits, session/retry identification, authentication-secret protection, and interrupted recovery. Preserve agreed times/defaults; assess and record impacts before changing them.
- PHP extensions, capacity limits, large-file delivery, consistent backups, restore compatibility/failure recovery, and whether to use encryption.
- Setup/private-storage verification, active-transfer tracking, cleanup overhead, and abnormal termination handling.
- List page sizes, right-side menus, confirmation screens, layout, and usability.

### 14.3 Outside the initial release / future options

- Individual ordinary download history with timestamps, search, and period-based aggregation.
- Recipient email collection/delivery and integration with external notification systems.
- Alignment and product-specific support for commercially pre-cut label sheets: consider for application v1.0.1 or later. A4 printing on full-sheet free-cut label paper is included initially.
- Starting partway through a partially used label sheet, business-card stock, and dedicated card printers.
- Side-by-side comparison of existing thumbnails.
- Automatic switching of backup execution methods.
- Language files beyond Japanese and English, added after a pre-release or official release.

<a id="backup"></a>

## 15. Backup and restore

- Create manual backups from administration and download them locally.
- Offer database/settings only or a full backup including distribution files and images.
- Restore from backups after confirming contents and creation time and saving the pre-restore state.
- Clearly explain that restoring also returns counts, disabling, and other states to their backed-up values.
- Allow optional automatic backups of the database and settings.
- Normally run automatic backups once daily, with manual switching between cron and administrator-page-access methods.
- For cron, configure periodic server calls and check the schedule and previous execution within the application. Explain how to register and remove the schedule.
- For administrator-page access, run a pending backup on the first administrator visit after the scheduled time. Do not guarantee scheduled execution without visits.
- Specify execution times in UTC and use UTC consistently for internal dates and execution decisions.
- Support Japanese and English displays: UTC only in English; UTC plus Japan Standard Time (UTC+9) in Japanese, e.g. 18:00 UTC (03:00 the next day in Japan).
- Display language must not change the scheduling time zone. Keep UTC execution independent of supplementary displays when adding languages.
- Retain the latest six automatic backup generations, including the newest. Delete excess oldest generations only after successful creation of a new backup.
- With daily backups, the oldest is five days before today. With visit-triggered backups, generation counts and elapsed days may differ.
- Preserve schedule time, existing backups, and six-generation retention when switching methods.
- Do not run automatic backups through the unselected method; prevent concurrent and duplicate execution.
- Explain that backups on the same server alone cannot protect against losing the server.
- Provide an operational example: use cron immediately after distribution starts; after about a month, download a full backup and manually switch to administrator-page-access backups.
- Automatic switching by operational age is outside the initial release; consider it if needed.

### 15.1 Backup and restore protection

- Restrict manual backup creation, downloads, and restore to authenticated administrators. Also protect automatic-backup entry points against arbitrary third-party execution.
- Reverify identity immediately before restore and confirm creation time, contents, and impact.
- Treat backups as sensitive because they contain authentication information; store them privately and document secure handling after download.
- Validate restore format, content, compatibility, and extraction destinations to prevent writes outside storage and execution of malicious files.
- During restore, use system-wide maintenance to stop downloads and ordinary updates, permitting only restore-related processing. Design conflict handling with active transfers during development.
- Save the current state before restore and provide a way to return to it on failure. Document manual recovery if automatic rollback is unavailable.
- After restore, invalidate all administrator sessions and temporary authentication states. Require authentication using the restored account and two-factor settings.
- Consider and test backup encryption, key handling, and detailed restoration of authentication information during development.

<a id="deletion"></a>

## 16. Data deletion and pre-deletion CSV export

- Do not provide a dedicated deletion menu; start confirmation from selected list entries.
- Separate delete buttons from ordinary operations and distinguish them with warning symbols and wording. Decide colors during screen design.
- Provide staged confirmation: entering deletion, targets/scope and CSV saving, then final deletion confirmation.
- Show the item, codes/count, and related data to be deleted on the CSV step, and offer pre-deletion CSV download.
- Include code IDs, codes, issuance groups, usage/limits, expiration, availability, and print status. Explain that CSV is not a full restorable backup including files/images.
- CSV export alone must not delete anything. Show the final delete button after the administrator chooses “I have confirmed the file is saved” or “Continue without saving.” Do not claim to automatically detect completed browser saving.
- Show a final warning such as “⚠ Are you sure you want to delete? This cannot be undone.” Execute only after explicit approval.
- Allow cancellation at each step before deletion. Do not impose the same repeated confirmations on every ordinary update.
- Item deletion removes related distribution files, images, groups, codes, and the item's cumulative downloads.
- Code deletion removes its settings, usage, print status, and related data, while preserving the item's cumulative downloads.
- After confirmation, show the target as deleted and reject new downloads and retries. Deleting a code alone must not remove files used by other valid codes.
- Keep affected distribution files privately for at least 12 hours after confirmed deletion. If an active transfer still uses a file afterward, defer physical removal until it is no longer in use.
- This retention is not a grace period for resuming use/retrying and is not a restore feature. Continue rejecting new downloads and retries after deletion.
- On the next authenticated administrator-page access at least 12 hours later, physically remove files after confirming they are not in use. Dedicated cron is not required.
- Skip files still being transferred and check again on later administrator visits. Files remain while administration is not accessed.
- Document that logical deletion is immediate, physical removal requires a later administrator visit after at least 12 hours, active transfers delay removal, and storage is consumed until removal.
- Instruct administrators to revisit administration at least 12 hours later, such as the following day.
- If a logically deleted file was already removed through FTP or similar means, confirm absence and mark cleanup complete. Do not treat absence alone as failure; distinguish permission and I/O errors.
- Do not recommend manual physical deletion. Explain that manually deleting an active file may prevent continued downloading, and that database/authentication/management files must not be deleted manually.
- Retain replaced old files while transfers use them; remove them afterward. Never overwrite a file and change an active transfer's contents.
- Later clean temporary retained files left by abnormal termination after verifying they are unused. Design transfer tracking and cleanup during development.
- Automatically remove an issuance group when all of its codes are deleted. Preserve it when only some codes are deleted.
- Do not automatically remove an item when all codes/groups are gone. Retain item information, files, images, and cumulative downloads so new codes can be added later.
- Require explicit item deletion from the item list. Explain the distinction from code/group deletion in confirmation screens and documentation.
- “Total code count” and “issuance group count” mean currently existing codes/groups, not historical issuance totals including deleted ones. Explain this in the interface and manual. Code/group deletion must not reduce item cumulative downloads.
- When deleting the last code in a group, include the resulting empty-group deletion in the scope confirmation.

<a id="languages"></a>

## 17. Japanese/English support and multilingual expansion

- Support Japanese and English from the initial release.
- Translate the installer, administration interface, recipient pages, guidance, confirmations, and errors into both languages.
- Manage system text in language files and allow display-language switching.
- Support additional languages by adding language files without changing application processing logic.
- Add other languages only after a pre-release or official release. Decide languages and timing separately.
- For additional languages, always show UTC and supplement it with the country/region standard time defined by the language file. English initially uses UTC only.
- Label supplementary times with a country, region, or time-zone name and correctly handle date changes. Decide multiple-zone and daylight-saving treatment when implementing each language file.
- Supplementary times affect display only. Switching language must not change the internal clock, saved timestamps, expiration evaluation, aggregate counts, or backup scheduling basis.
- Automatic translation of administrator-entered item names and notices is not required.
- Store and evaluate dates in UTC regardless of language. English shows UTC; Japanese shows UTC and Japan Standard Time. Switching languages must not change expiration, counts, or execution time.

<a id="licensing"></a>

## 18. License

The license is under consideration.
