# Release and Moodle Marketplace

## GitHub release
1. On the change branch:
   - bump `$plugin->version` and `$plugin->release`;
   - write the `## <release> - <date>` section in `CHANGELOG.md`;
   - update the documentation.
2. Merge into the default branch with CI green on every matrix combination.
3. Run the **Prepare release** workflow (`release.yml`, `workflow_dispatch`).
   - It checks the version, the changelog and the CI status of the commit.
   - It builds `<component>-<release>.zip` with the right root folder (`<x>/version.php`).
   - It creates a **draft** release, with the notes taken from the changelog.
   - Use `dry-run` to only validate.
4. Publishing the draft is the maintainer's decision: ask first.
   - `gh release edit v<release> --draft=false` (keep `--prerelease` if the maturity is not stable).
   - The tag is created on publication, and the release becomes immutable.
   - Never run `gh release create` over a published release (`Cannot upload asset to an immutable release`).
5. **Verify published release** (`release-check.yml`) checks:
   - tag against `version.php`;
   - the attached ZIP equals the tagged code;
   - pre-release flag consistent with the maturity;
   - green CI.

   If it fails: delete, fix and release the next version. The tag of an immutable release cannot be reused.

## Sending to the Marketplace
- Automatic (`moodle-release.yml`, triggered by the `v*` tag created on publication): only after the plugin is approved. The workflow is skipped until all three are in place.
  - Repository variable: `gh variable set MARKETPLACE_ENABLED -b true -R <repo>`.
  - Secret `MOODLE_MARKETPLACE_TOKEN`, with a token from https://marketplace.moodle.com/account/security. The maintainer creates and stores it (`gh secret set MOODLE_MARKETPLACE_TOKEN -R <repo>`, or once per organisation); an agent never handles the token.
  - Environment `marketplace` with required reviewers, so each upload takes one click.
- Manual, on marketplace.moodle.com:
  1. Open *Dashboard > plugin > Edit page > Versions > Add new version*.
  2. Upload the release ZIP (`gh release download v<release> -p '*.zip'`). The form fills in type, component, version, release name, maturity and Moodle versions.
  3. Fill in the tag (`v<release>`) and the release notes (the fixes for the reviewer).
  4. The Marketplace automated test takes about 10 minutes.

## Review statuses
| Status | Can add a version or edit details? |
|---|---|
| Submitted for review (in the queue) | Yes |
| In review (a reviewer is checking it) | No: wait |
| Changes needed | Yes: fix and upload a new version |
| Review passed | Yes: check every detail, then publish |

- Reviews take 2 to 4 weeks, more when changes are requested.
- Reviewer comments and status changes appear on the Jira Service Management ticket (`CONTRIB-<n>`); the maintainer needs a JSM account to read them.
- After a new version is uploaded during "Changes needed", the status goes back to "Submitted for review".

## Plugin still under review ("Changes needed")
- `/plugins/<component>/edit/*` returns 403. Use *Dashboard > View submission* (`/plugins/submit/step3/<component>`).
- *Add a new version*: upload the release ZIP. The Marketplace "always checks the most recent file", so the upload is the resubmission; there is no other resubmit button.
- In the version form, check the *Repository URL*: it comes from the old submission and may point to a renamed or transferred repository. Fill in the tag and release notes, quoting the ticket (`CONTRIB-<n>`) and each finding.
- *Update plugin details* (name, description and screenshots):
  - the description is a `textarea` with HTML, limited to 7000 characters; a description already over the limit blocks saving;
  - do not paste the converted README.
- Commenting on the Jira ticket speaks for the maintainer: only with their approval.

## Plugin page (marketplace.moodle.com)
- **Overview:**
  - Name.
  - Icon of at least 200x200.
  - Short description up to 256 characters.
  - Screenshots.
- **Description:** HTML in TinyMCE (`tinymce.get('plugin_edit_description_form_description').setContent(...)` followed by `.save()`). Follow the description checklist in [review-checklist.md](review-checklist.md).
- **Support:**
  - Repository URL and bug tracker (taken from the upload).
  - Documentation: the plugin's GitHub Pages.
  - Website: the organisation site.
  - *Potential privacy issues*: what leaves Moodle and where it goes.
- **Publish:** the button opens a modal with "Publish listing". Publishing is an external action: only with the maintainer's approval.

Form pitfalls:
- FilePond sometimes uploads screenshots twice. Check and remove the copies before saving.
- Browser automation may only upload files from allowed folders: copy images and ZIPs there first.
- Page-reading tools may block output that contains URLs or cookies. Read element `innerText` instead.
- Reviewer e-mails (MMR-* tickets): one issue per finding in the plugin repository. Some listing fields can only be edited after approval ("Update the listing after approval").
