# Joomla 3.x (JoomlaWorks Security Distribution) — Agent Onboarding & Patch Log

---

## For the AI Agent — Read This First

This file is the primary onboarding document for resuming work on this project. Read it fully before making any changes.

### What this project is

A security-hardened, PHP 8.x-compatible fork of Joomla 3.10.20 eLTS, maintained by JoomlaWorks. It is **not** an official Joomla release. The goal is to keep Joomla 3.x sites running safely on modern PHP and MySQL/MariaDB versions, with backported CVE fixes from Joomla 4/5/6.

- **GitHub repo:** https://github.com/joomlaworks/joomla-3.x
- **Current version:** Joomla 3.15.0 (released July 9, 2026) [pending — code done, version bump not yet performed, see 3.15.0 patch log]
- **Minimum PHP:** 7.4 — **all code changes must remain compatible with PHP 7.4**. This is the *recommended/tested* baseline, chosen because it's broadly available across modern hosting (e.g. AlmaLinux 8 with cPanel, or Ubuntu 22.04+ with Ondřej's PHP repositories) — not an arbitrary cutoff. Separately, `JOOMLA_MINIMUM_PHP` (the hard parse-safety gate in the three `index.php` entry points) and the update feed's `<php_minimum>` are both set to `7.1.0` — the codebase's actual verified floor (see AGENTS.md's F-5/I-5 entries) — so PHP 7.1–7.3 sites still receive security updates even though they're below the recommended baseline. Don't conflate the two numbers: 7.4 = what we target/test; 7.1 = the lowest we won't outright refuse.
- **Tested up to PHP:** 8.5
- **Database support:** MySQL 5.7+, MySQL 8.x, MariaDB, PostgreSQL, SQL Azure

### Key conventions to follow

- **PHP 7.4 minimum:** The `?Type` nullable syntax is fine (valid since PHP 7.1). The `#[\AllowDynamicProperties]` attribute is fine (parsed as a comment on PHP 7.x). Never use PHP 8.0+ syntax that would be a parse error on 7.4.
- **No comments unless the WHY is non-obvious.** Do not narrate what code does.
- **Security patches:** Always check `libraries/vendor/joomla/filter/src/InputFilter.php` (vendor-level) AND `libraries/src/Filter/InputFilter.php` (CMS-level) — they are separate implementations and both may need patching.
- **SQL changes** affect three files: `installation/sql/mysql/joomla.sql`, `installation/sql/postgresql/joomla.sql`, `installation/sql/sqlazure/joomla.sql`. Schema migration SQL goes in `administrator/components/com_admin/sql/updates/{mysql,postgresql,sqlazure}/`. File naming: `{version}-{YYYY-MM-DD}.sql` (e.g. `3.12.0-2026-05-21.sql`).
- **Changelog:** `CHANGELOG.md` is the detailed log; `README.md` has a brief per-version summary. Always update both.
- **AGENTS.md** (this file): append a new section for every version worked on, documenting every change made. This is how future sessions resume without losing context.

### Open considerations for future sessions

- **Language-file version fingerprinting:** As of 3.14.0, `administrator/language/en-GB/{en-GB.xml,install.xml}`, `language/en-GB/{en-GB.xml,install.xml}`, and `administrator/manifests/packages/pkg_en-GB.xml` correctly report the running version (previously stuck at the stale `3.10.20` from the original eLTS base, which caused false-positive "vulnerable old version" flags from some scanners). The tradeoff: unauthenticated vulnerability scanners/bots commonly fingerprint Joomla version by reading these language metafiles (no login required), and an accurate `3.14` value now lets them pinpoint the exact version precisely — and a version number with no corresponding official Joomla release could itself signal "this is a patched fork," inviting more targeted probing. Consider whether to deliberately desynchronize or obscure the language-file version from `JVERSION` in a future release, without breaking the update system's version comparisons. Not yet decided or implemented — raised by the maintainer on 2026-07-04, no fix applied.

### What already exists and must NOT be re-applied

The following fixes are already in the codebase. Do not duplicate them:

- CVE-2025-54476 + H-1 extension (whitespace stripping + `\r\v\f` in `checkAttribute()`)
- CVE-2025-63083 (pagebreak toc.php htmlspecialchars)
- CVE-2026-21629 (com_ajax guest check)
- H-2 through H-8 (joomlaupdate ACL, TOTP timing, Yubikey HMAC, restore.php, RSS escaping, password reset timing, eval() removal)
- U-1 through U-3 (MediaHelper blocklist, sniff offset, images/.htaccess)
- P-1 through P-16 (all PHP 8.x compat fixes documented in the 3.11 section below)
- CVE-2025-63082 (data: URI blocking in checkAttribute)
- CVE-2024-40747 (ModuleHelper chrome attribs escaping)
- CVE-2025-25226 (quoteNameStr null byte/backslash rejection)
- CVE-2026-21631 (com_associations edit.php data-title escaping)
- All `#[\AllowDynamicProperties]` additions (Table, CMSObject, idna_convert)
- All remaining implicit-nullable fixes in fof/, vendor/joomla/session, vendor/joomla/data, vendor/joomla/di, vendor/google/recaptcha, vendor/symfony/yaml, vendor/joomla/filesystem, plugins/privacy/*
- All `(boolean)` → `(bool)` casts (libraries/src/ and libraries/joomla/, 13 files)
- `Uri::getInstance()` null guard (null → 'SERVER' coercion)
- `Uri::getInstance()` `HTTP_HOST` absent-in-CLI guard (`$httpHost` variable with `'localhost'` fallback before the SERVER URI branch — both Apache and IIS paths)
- Q-3 through Q-11 PHP 8.x fixes (see 3.13 patch log): null guards in `HtmlView::escape()`, `Date::__construct()`, `ListModel::populateState()` (×2), `utf8_ltrim/rtrim/trim`, `Json::stringToObject()`; declared properties `$registeredurlparams` on `CMSApplication`, `$itemTags` on `TagsHelper`, `$empty` and `$dates` on `FinderIndexerQuery`
- Q-12/Q-13 PHP 8.5 fixes (see 3.14.0 patch log): `HtmlDocument::getBuffer()`/`setBuffer()` null-array-offset normalization; `Image::destroy()`/`Backgroundfill::execute()` no longer call the deprecated `imagedestroy()`
- T-1 through T-8 (see 3.15.0 patch log): LFI in `HtmlView::setLayout()` (CVE-2026-40383); SSL-downgrade on password/username reset links (CVE-2026-48902); nested-entity-decode bypass in `InputFilter::checkAttribute()`/`cleanAttributes()` (CVE-2026-48905/-48903); language-override XSS, no separate fix needed (CVE-2026-48954); com_installer update-list XSS (CVE-2026-48952); com_contact vCard access-control bypass (CVE-2026-48948); generic image layout XSS (CVE-2026-48953); `InputFilter::getInstance()` cache-key fix (CVE-2026-48901)
- `fixSchemas()` in `com_admin/script.php` (auto-runs SQL migrations + syncs `#__schemas` + syncs `manifest_cache.version` on upgrade)
- `administrator/manifests/files/joomla.xml` version and `<updateservers>` URL (updated; do not revert)
- `deleteUnexistingFiles()` 3.12 entries (beez3, hathor, eos310, phpversioncheck directories and language files)
- `com_joomlaupdate` OPcache fix (`cleanup()` reads version from `joomla.xml`, `complete.php` uses session value)

### Update server

The update server uses a **two-file architecture** that mirrors `update.joomla.org` exactly. The update site type in `#__update_sites` is `collection`, which means Joomla uses `CollectionAdapter` to parse `list.xml`. `CollectionAdapter` only understands `<extensionset>/<extension>` tags — it will silently return nothing if given an `<updates>` file.

- **`docs/list.xml`** — `<extensionset>` collection. Each `<extension>` entry has a `targetplatformversion` regex matched against `JVERSION` on the visitor's site, and a `detailsurl` pointing to `extension.xml`. Add one entry per supported minor version.
- **`docs/extension.xml`** — `<updates>` details file. Contains the `<update>` block with the download URL. `com_joomlaupdate` fetches this via `detailsurl` to display the "Update Now" button and download the package.
- **Download URL:** `https://github.com/joomlaworks/joomla-3.x/releases/download/rolling/joomla-latest.zip`
- **GitHub Action:** `.github/workflows/rolling-release.yml` — rebuilds the zip on every push to `main` using `git archive`

**To release a new version (e.g. 3.14.0):**
1. Bump `MINOR_VERSION` (and `RELEASE`, `DEV_LEVEL`) in `libraries/src/Version.php`
2. In `docs/list.xml`: bump `version="3.13.0"` → `version="3.14.0"` in all `<extension>` entries, and add a new entry for `targetplatformversion="3.14"`
3. In `docs/extension.xml`: bump `<version>3.13.0</version>` → `<version>3.14.0</version>` and update `<infourl>`
4. In `administrator/manifests/files/joomla.xml`: bump `<version>` to match and update `<creationDate>`
5. Update `CHANGELOG.md` and `README.md`, push

### Removed in 3.12 (do not re-add)

- `plugins/quickicon/eos310` and all its language/media files
- `plugins/quickicon/phpversioncheck` and its language files
- `templates/beez3` and its language files
- `administrator/templates/hathor` and its language files

### CVEs confirmed NOT applicable to 3.x (do not re-investigate)

| CVE | Reason |
|-----|--------|
| CVE-2026-21630 | SQL injection in com_content webservice — 4.0.0+ only |
| CVE-2026-21632 | XSS in article title outputs — 4.0.0+ only |
| CVE-2026-23898 | Arbitrary file deletion in com_joomlaupdate — 4.0.0+ only |
| CVE-2026-23899 | Improper access check in webservice endpoints — 4.0.0+ only |

> **Note:** CVE-2025-63082 and CVE-2026-21631 were initially marked "4.0.0+ only" in the 3.11 review but were later confirmed to affect 3.x and have been patched in 3.12.

---

# Joomla 3.11.0 — Security Patch Log

**Date:** April 20, 2026
**Base version:** Joomla 3.10.20 eLTS
**Patched version:** Joomla 3.11.0

---

## Summary

This document records the security patches backported from Joomla 5.x/6.x into the Joomla 3.10.20 eLTS codebase, and the version bump to 3.11.0. All CVEs listed below were confirmed to affect the 3.x branch post-3.10.20 release. CVEs affecting only Joomla 4.0.0+ were reviewed and determined not applicable.

---

## CVEs Patched

### CVE-2025-54476 — XSS via input filter attribute bypass
- **Severity:** High
- **Affected:** Joomla 3.0.0–3.10.20-elts
- **Fixed upstream in:** Joomla 4.4.14 / 5.3.4 (September 30, 2025); joomla-framework/filter v2.0.6 (PR #84)
- **File patched:** `libraries/vendor/joomla/filter/src/InputFilter.php`
- **Change:** Added whitespace/null-byte stripping inside `checkAttribute()` before the XSS pattern match. Prevents bypass vectors such as `java\tscript:alert()` or `java&#x09;script:alert()` from evading the existing regex check.

```php
// Before (vulnerable):
$attrSubSet[1] = html_entity_decode(strtolower($attrSubSet[1]), $quoteStyle, 'UTF-8');
return (strpos($attrSubSet[1], 'expression') !== false && $attrSubSet[0] === 'style')
    || preg_match('/(?:(?:java|vb|live)script|behaviour|mocha)(?::|&colon;|&column;)/', $attrSubSet[1]) !== 0;

// After (patched):
$attrSubSet[1] = html_entity_decode(strtolower($attrSubSet[1]), $quoteStyle, 'UTF-8');

// Remove common XSS-evasion characters (CVE-2025-54476)
$attrSubSet[1] = str_replace(["\t", "\n", " ", "\0"], '', $attrSubSet[1]);

return (strpos($attrSubSet[1], 'expression') !== false && $attrSubSet[0] === 'style')
    || preg_match('/(?:(?:java|vb|live)script|behaviour|mocha)(?::|&colon;|&column;)/', $attrSubSet[1]) !== 0;
```

---

### CVE-2025-63083 — XSS in pagebreak plugin table-of-contents output
- **Severity:** Medium
- **Affected:** Joomla 3.9.0–5.4.1
- **Fixed upstream in:** Joomla 5.4.2 / 6.0.2 (January 6, 2026)
- **File patched:** `plugins/content/pagebreak/tmpl/toc.php`
- **Change:** Wrapped `$listItem->title` in `htmlspecialchars(..., ENT_QUOTES, 'UTF-8')` to prevent injection of arbitrary HTML/JS via crafted article page-break titles in the table-of-contents navigation block.

```php
// Before (vulnerable):
<?php echo $listItem->title; ?>

// After (patched):
<?php echo htmlspecialchars($listItem->title, ENT_QUOTES, 'UTF-8'); ?>
```

---

### CVE-2026-21629 — Improper ACL check in administrator com_ajax
- **Severity:** Medium
- **Affected:** Joomla 3.0.0–5.4.3
- **Fixed upstream in:** Joomla 5.4.4 / 6.0.4 (March 31, 2026)
- **File patched:** `administrator/components/com_ajax/ajax.php`
- **Change:** Added a guest-user check at the top of the administrator-side `com_ajax` entry point. Unauthenticated requests now receive a 403 before any AJAX dispatch logic runs. Without this patch, a guest user could invoke AJAX handler methods (`getAjax()`, `postAjax()`) in admin-area modules and plugins that assumed login-wall protection.

```php
// Added after defined('_JEXEC') or die:
if (JFactory::getUser()->guest)
{
    throw new RuntimeException(JText::_('JERROR_ALERTNOAUTHOR'), 403);
}
```

> **Note:** This CVE alone is not sufficient for remote code execution or webshell upload on a stock Joomla installation. Exploitation would require chaining with a secondary vulnerable extension that exposes a dangerous AJAX method.

---

---

## Proactive Security Hardening — April 20, 2026

The following issues were identified by static analysis of the codebase and fixed. None carry a CVE number but each represents a real exploitable condition or weakens defence-in-depth.

---

### H-1 — CVE-2025-54476 patch incomplete: `\r` / `\v` / `\f` bypass
- **Severity:** High
- **File:** `libraries/vendor/joomla/filter/src/InputFilter.php`
- **Issue:** The original patch stripped `\t`, `\n`, space, and `\0` but omitted carriage return (`\r`, 0x0D), vertical tab (`\v`, 0x0B), and form feed (`\f`, 0x0C). The WHATWG HTML5 URL parser strips all ASCII whitespace before scheme evaluation, so `java\rscript:alert(1)` bypassed the regex and executed in Firefox/Chrome.
- **Fix:** Extended the `str_replace` character list to include `"\r"`, `"\v"`, `"\f"`.

```php
// Before:
$attrSubSet[1] = str_replace(["\t", "\n", " ", "\0"], '', $attrSubSet[1]);

// After:
$attrSubSet[1] = str_replace(["\t", "\n", "\r", "\v", "\f", " ", "\0"], '', $attrSubSet[1]);
```

---

### H-2 — Missing `core.admin` check in com_joomlaupdate controller
- **Severity:** Medium
- **File:** `administrator/components/com_joomlaupdate/controllers/update.php`
- **Issue:** `finalise()` (line 138), `cleanup()` (line 182), and `purge()` (line 235) only validated the CSRF token. Any backend user (Manager role) could invoke them. The sibling methods `upload()`, `confirm()`, etc. already check `core.admin`.
- **Fix:** Added `JFactory::getUser()->authorise('core.admin')` guard to each of the three methods immediately after the token check.

---

### H-3 — Non-constant-time TOTP comparison
- **Severity:** Medium
- **File:** `plugins/twofactorauth/totp/totp.php`
- **Issue:** Six `===` comparisons of 6-digit OTP codes leaked timing information, reducing the effective brute-force search space.
- **Fix:** All six sites changed to `hash_equals((string) $code, (string) $input)`.

---

### H-4 — Yubikey plugin: no HMAC verification, hardcoded demo client ID
- **Severity:** Medium/High
- **Files:** `plugins/twofactorauth/yubikey/yubikey.php`, `yubikey.xml`, `en-GB.plg_twofactorauth_yubikey.ini`
- **Issue:** The Yubico API response `h=` HMAC field was never verified, allowing a MITM to forge `status=OK` and bypass 2FA. The plugin also used the hardcoded public demo `id=1`, which has no associated secret key, making HMAC verification structurally impossible.
- **Fix:**
  - Added `clientid` (integer) and `clientsecret` (password) plugin parameters.
  - Requests are now HMAC-SHA1-signed when a secret is configured.
  - Response `h=` signature is verified with `hash_equals()` before `status=OK` is trusted.
  - OTP/nonce response fields compared with `hash_equals()`.
  - Plugin returns `false` immediately if no client ID is configured.
- **Action required:** Obtain a free API key at `https://upgrade.yubico.com/getapikey/` and enter the Client ID and Secret in the plugin's configuration.

---

### H-5 — Unauthenticated `AKFactory::unserialize()` in restore.php
- **Severity:** High (conditional on restoration.php existing)
- **File:** `administrator/components/com_joomlaupdate/restore.php`
- **Issue:** The `factory` request parameter path called `AKFactory::unserialize($_REQUEST['factory'])` without the AES password check that guards the sibling `json` path. During an active update window (while `restoration.php` exists) an attacker could trigger PHP object injection via POP gadgets in Composer dependencies.
- **Fix:** Added a guard that terminates with `Invalid login` if the `json` path password check was not satisfied before `factory` is processed.

---

### H-6 — Unescaped RSS feed output in com_newsfeeds
- **Severity:** Low/Medium
- **File:** `components/com_newsfeeds/views/newsfeed/tmpl/default.php`
- **Issue:** Feed description, image URI, image title, item titles, and item links were echoed without escaping. A compromised or hostile upstream RSS feed could inject HTML/JS.
- **Fix:** All feed-sourced values wrapped with `$this->escape()` or `htmlspecialchars(..., ENT_QUOTES, 'UTF-8')`.

---

### H-7 — Non-constant-time password-reset token comparison
- **Severity:** Low
- **File:** `components/com_users/models/reset.php`
- **Issue:** `$user->activation !== $token` used `!==` (short-circuit) instead of a timing-safe comparison.
- **Fix:** Changed to `!hash_equals((string) $user->activation, (string) $token)`.

---

### H-8 — `eval()` in HtmlDocument::countModules
- **Severity:** Low
- **File:** `libraries/src/Document/HtmlDocument.php`
- **Issue:** A module-count expression built from whitelist-split tokens was evaluated with `eval()`. Not currently exploitable via user input, but any future caller forwarding user data would yield RCE.
- **Fix:** Replaced `eval()` with an explicit `switch`-based integer expression evaluator covering all operators the method supports (`+`, `-`, `*`, `/`, `==`, `!=`, `<>`, `<`, `>`, `<=`, `>=`, `and`, `or`, `xor`).

---

## PHP 8.x Compatibility Fixes — April 20, 2026

The following changes were made to ensure the codebase runs cleanly on PHP 8.0–8.3 without deprecation notices, `TypeError` exceptions, or fatal errors. All changes are backward-compatible with PHP 7.4.

---

### P-1 — `utf8_encode()` / `utf8_decode()` removed in PHP 8.2
- **Severity:** Fatal (function not found on PHP 8.2+)
- **Files patched:**
  - `libraries/vendor/joomla/filter/src/InputFilter.php` (×3 `utf8_encode()` calls)
  - `administrator/components/com_finder/helpers/indexer/stemmer/fr.php` (×6 calls)
  - `components/com_users/models/profile.php`
  - `administrator/components/com_admin/models/profile.php`
  - `libraries/fof/string/utils.php`
- **Fix:** All calls replaced with `mb_convert_encoding($value, 'UTF-8', 'ISO-8859-1')` / `mb_convert_encoding($value, 'ISO-8859-1', 'UTF-8')` as appropriate.

---

### P-2 — `FILTER_SANITIZE_STRING` removed in PHP 8.1
- **Severity:** Fatal (`E_ERROR: Undefined constant`)
- **File:** `libraries/src/Application/DaemonApplication.php`
- **Fix:** Replaced `filter_var($value, FILTER_SANITIZE_STRING)` with `strip_tags($value)`.

---

### P-3 — `null` passed as `$flags` to `htmlentities()` deprecated in PHP 8.1
- **Severity:** `TypeError` on PHP 8.1+
- **File:** `libraries/fof/string/utils.php`
- **Fix:** Replaced `null` flag argument with explicit `ENT_COMPAT`.

---

### P-4 — `Serializable` interface deprecated in PHP 8.1
- **Severity:** `E_DEPRECATED` notice at class-load time; cascades to fatal session startup failure when `display_errors` routes output before `session_name()` is called
- **Files patched:**
  - `libraries/vendor/joomla/input/src/Input.php` (base framework class)
  - `libraries/src/Input/Input.php` (CMS subclass)
  - `libraries/src/Input/Cli.php` (CLI subclass)
  - `libraries/src/Input/Cookie.php` — no change required; inherits new magic methods from CMS `Input` parent
- **Fix:** Added `__serialize(): array` and `__unserialize(array $data): void` magic methods alongside the existing `serialize()`/`unserialize()` methods. On PHP 8.1+ the magic methods take precedence and the deprecation is suppressed. On PHP 7.4 the magic methods are simply unused extra methods — no conflict. Old `Serializable` methods retained for PHP 7.x compatibility.

---

### P-5 — `count()` missing `int` return type (PHP 8.1 `Countable` signature)
- **Severity:** `E_DEPRECATED` notice
- **File:** `libraries/vendor/joomla/input/src/Input.php` line ~170
- **Fix:** Changed `public function count()` to `public function count(): int`. The `: int` return type declaration is valid PHP 7.0+ syntax — no compatibility shim required.

---

### P-6 — `stripos()` / `substr_replace()` / `strlen()` called with possible `null` argument (PHP 8.1)
- **Severity:** `TypeError` on PHP 8.1+
- **File:** `libraries/src/Application/WebApplication.php` line ~1305
- **Issue:** `$this->get('uri.request')` and `$this->get('uri.base.full')` can return `null` when the registry key is absent. PHP 8.1 no longer silently coerces `null` to `''` for string functions.
- **Fix:** Cast both values to `(string)` at the call sites for `stripos()`, `substr_replace()`, and `strlen()`.

---

### P-7 — `session_name()` / `session_cache_limiter()` called after headers sent
- **Severity:** `E_WARNING`; can prevent session startup
- **File:** `libraries/joomla/session/handler/native.php` lines ~128 and ~235
- **Root cause:** Any earlier deprecation notice output (e.g. from P-4 above) marks headers as sent before the session handler runs, causing PHP to emit warnings and potentially abort session start.
- **Fix:** Wrapped both calls in `if (!headers_sent()) { ... }` guards. The session name / cache limiter settings are silently skipped rather than triggering warnings if headers have already been sent.

---

### P-8 — Implicitly nullable typed parameters across Application class hierarchy (PHP 8.1)
- **Severity:** `E_DEPRECATED` notice on every bootstrap request/CLI invocation
- **Files patched:**
  - `libraries/src/Application/BaseApplication.php` — `__construct()`, `loadDispatcher()`, `loadIdentity()`
  - `libraries/src/Application/CliApplication.php` — `__construct()`
  - `libraries/src/Application/DaemonApplication.php` — `__construct()`
  - `libraries/src/Application/CMSApplication.php` — `__construct()`, `loadSession()`
  - `libraries/src/Application/WebApplication.php` — `__construct()`, `loadDocument()`, `loadLanguage()`, `loadSession()`
  - `libraries/src/Application/SiteApplication.php` — `__construct()`
  - `libraries/src/Application/AdministratorApplication.php` — `__construct()`
- **Issue:** All 11 signatures used the pattern `TypeName $param = null` without `?`. PHP 8.1 deprecated this implicit nullable form. With `error_reporting(E_ALL)` and `display_errors=1` set by `finder_indexer.php` and `deletefiles.php`, the notices print to stdout at bootstrap, polluting CLI output and re-triggering the session "headers already sent" cascade (see P-7).
- **Fix:** All affected parameters changed to explicitly nullable `?TypeName $param = null`. Valid PHP 7.1+ syntax — fully backward-compatible with PHP 7.4.

---

### P-9 — `posix_getuid()` / `posix_getgid()` called with spurious argument in DaemonApplication
- **Severity:** `E_WARNING` on every `changeIdentity()` call (unsuppressed)
- **File:** `libraries/src/Application/DaemonApplication.php` lines ~468 and ~476
- **Issue:** `posix_getuid()` and `posix_getgid()` take **zero** arguments. Both calls incorrectly passed `$file` (the PID file path), copied from the adjacent `fileowner()`/`filegroup()` calls. PHP 8.0+ emits an unsuppressed `E_WARNING` for unexpected arguments to these functions. The logic was also subtly wrong — the intent is to compare the *current process* UID/GID against the target, which is what the no-argument forms return.
- **Fix:** Removed the spurious `$file` argument from both calls. Correct on all PHP versions.

---

## Upload Security Hardening — April 20, 2026

The following vulnerabilities were identified by auditing all file upload code paths. None carry a CVE number but each represents a real exploitable condition.

---

### U-1 — Incomplete executable extension blocklist in `JHelperMedia::canUpload()`
- **Severity:** High
- **File:** `libraries/src/Helper/MediaHelper.php`
- **Issue:** The `$executable` array — which blocks dangerous extensions in ALL dot-separated segments of a filename (e.g. `shell.php.jpg` is caught because `php` appears in a non-final segment) — was missing several extensions that PHP or Apache will execute:

  | Missing | Why dangerous |
  |---|---|
  | `phar` | PHP executes `.phar` files natively as PHP code on all modern PHP versions |
  | `php3`, `php4`, `php5`, `php7`, `php8` | Commonly mapped to PHP by Apache; `php5` and `php7` particularly common on shared hosts |
  | `phps` | PHP source display; some hosts execute it, all hosts leak code |
  | `shtml` | Apache SSI execution — allows `<!--#exec cmd="..." -->` |

- **Fix:** Added `php3`, `php4`, `php5`, `php7`, `php8`, `phps`, `phar`, and `shtml` to the blocklist.

---

### U-2 — XSS content-sniffing check reads last 1 byte instead of first 256
- **Severity:** Medium
- **File:** `libraries/src/Helper/MediaHelper.php`
- **Issue:** The check that looks for embedded HTML/script tags in uploaded file content used `file_get_contents($file['tmp_name'], false, null, -1, 256)`. On PHP 7.1+, a negative offset counts from the *end* of the file, so this read only the final ~1 character. The check was completely bypassable by placing any HTML payload (`<script>`, `<?php`, etc.) anywhere except the very last byte of the file.
- **Fix:** Changed offset from `-1` to `0` so the first 256 bytes are checked as originally intended.

---

### U-3 — No server-level PHP execution guard in `images/` upload directory
- **Severity:** Medium (defence in depth)
- **File:** `images/.htaccess` *(created)*
- **Issue:** No `.htaccess` existed in the primary media upload destination. If any file with an executable extension reached disk (misconfigured allowlist, future bypass, direct FTP placement), Apache would execute it as PHP or a CGI script.
- **Fix:** Created `images/.htaccess` that:
  - Denies HTTP requests for files matching PHP/script extensions (`php`, `php[0-9]`, `phtml`, `phar`, `shtml`, `cgi`, `pl`, `py`, `asp`, `aspx`, `exe`, `sh`, `bash`) using both Apache 2.4 and 2.2 syntax.
  - Disables `php_flag engine` for mod_php 5, 7, and 8.
  - Removes `ExecCGI` and directory indexing (`-Indexes`).

---

## CVEs Reviewed but Not Applicable to 3.x (at 3.11 review time)

| CVE | Reason not patched |
|-----|--------------------|
| CVE-2025-63082 | ~~XSS for data URLs — affects 4.0.0+ only~~ **CORRECTION: patched in 3.12** |
| CVE-2026-21630 | SQL injection in com_content webservice — 4.0.0+ only (no webservice API in 3.x) |
| CVE-2026-21631 | ~~XSS in com_associations comparison view — 4.0.0+ only~~ **CORRECTION: patched in 3.12** |
| CVE-2026-21632 | XSS in article title outputs — 4.0.0+ only |
| CVE-2026-23898 | Arbitrary file deletion in com_joomlaupdate — 4.0.0+ only |
| CVE-2026-23899 | Improper access check in webservice endpoints — 4.0.0+ only |

---

## Version Bump

**File:** `libraries/src/Version.php`

| Constant | Before | After |
|----------|--------|-------|
| `MINOR_VERSION` | `10` | `11` |
| `PATCH_VERSION` | `20` | `0` |
| `EXTRA_VERSION` | `'elts'` | `''` |
| `RELEASE` (deprecated) | `'3.10'` | `'3.11'` |
| `DEV_LEVEL` (deprecated) | `'20-elts'` | `'0'` |

The installation now reports itself as **Joomla! 3.11.0**.

---

## PHP 8.x Compatibility Fixes — Session 2 — April 20, 2026

The following changes were identified in a second-pass audit targeting PHP 8.1–8.5 compatibility. All fixes are backward-compatible with PHP 7.4.

---

### P-10 — Implicitly nullable typed parameters across the full codebase (PHP 8.1)
- **Severity:** `E_DEPRECATED` on every bootstrap/CLI invocation; cascades to stdout pollution, "headers already sent" fatal errors, and session startup failures
- **Scope:** 118 files changed
- **Files patched (representative list):**
  - `libraries/src/` — all classes in Application, Cache, Database, Document, Event, Extension, Factory, Filter, Form, HTML, Http, Image, Installer, Language, Log, Mail, Menu, MVC, Plugin, Profiler, Router, Schema, Session, Table, Uri, User, Utility, Version
  - `libraries/joomla/` — legacy session, form, database, application, archive classes
  - `libraries/vendor/typo3/phar-stream-wrapper/src/` — Manager, PharStreamWrapper, Resolver classes
  - `libraries/vendor/joomla/application/src/` — AbstractApplication and related classes
- **Issue:** The pattern `TypeName $param = null` without a leading `?` was used throughout. PHP 8.1 deprecated this implicit nullable form and emits `E_DEPRECATED` for every such signature that is invoked. At scale, hundreds of notices per request made stdout unusable for CLI scripts.
- **Fix:** All affected parameters changed to `?TypeName $param = null` using an automated Python state-machine script. The script extracted function parameter lists and applied the transformation only within those lists — property declarations of the form `public $foo = null` were intentionally left unchanged (they are already valid on all PHP versions and do not accept the `?` prefix). The `?` prefix is valid PHP 7.1+ syntax — fully backward-compatible with PHP 7.4.

---

### P-11 — `session_set_save_handler()` individual-callback form deprecated (PHP 8.1)
- **Severity:** `E_DEPRECATED` on every session start
- **Files patched:**
  - `libraries/joomla/session/storage.php` (base class)
  - `libraries/joomla/session/storage/apc.php`
  - `libraries/joomla/session/storage/apcu.php`
  - `libraries/joomla/session/storage/database.php`
  - `libraries/joomla/session/storage/xcache.php`
- **Issue:** `session_set_save_handler()` was called with 6 individual callbacks (the pre-PHP-5.4 style). PHP 8.1 deprecated this form. The recommended replacement is to pass an object that implements `SessionHandlerInterface`.
- **Fix:**
  - `JSessionStorage` declared as `abstract class JSessionStorage implements \SessionHandlerInterface`.
  - `register()` updated to call `session_set_save_handler($this, true)`.
  - All 6 interface methods (`open`, `close`, `read`, `write`, `destroy`, `gc`) annotated with `#[\ReturnTypeWillChange]` in the base class and each subclass to suppress PHP 8.1 return-type mismatch deprecations. On PHP 7.4 the `#` starts a comment, making the attribute a no-op — fully backward-compatible.
  - `read()` bare `return;` statements fixed to `return ''` in the base class and `xcache.php` to satisfy the `string` return type contract.

---

### P-12 — Dynamic property creation deprecated (PHP 8.2)
- **Severity:** `E_DEPRECATED` on first assignment to undeclared property; becomes `E_ERROR` in PHP 9
- **Files patched:**
  - `libraries/src/User/User.php`
  - `libraries/src/Cache/CacheStorage.php`
- **Issues and fixes:**
  - `User::$aid` — assigned by `bind()` and legacy code but never declared in the class body. Added `public $aid = 0;` after the existing `$requireReset` declaration.
  - `CacheStorage::$_threshold` — assigned in the base-class constructor but not declared. Added `public $_threshold;` after the existing `$_hash` declaration. The notice appeared on concrete subclasses (e.g. `MemcachedStorage`) because PHP reports the class being instantiated, not the assigning class.

---

### P-13 — `${var}` string interpolation removed (PHP 8.3)
- **Severity:** Fatal parse error on PHP 8.3+
- **File:** `libraries/vendor/leafo/lessphp/lessc.inc.php`
- **Issue:** PHP 8.2 deprecated the `"${varName}"` and `"${expr}"` string interpolation forms; PHP 8.3 made them a fatal parse error. Two occurrences were present:
  - Line 1366: `"${name}expecting..."` (simple variable form)
  - Line 1748: `"op_${ltype}_${rtype}"` (complex expression form)
- **Fix:** Both changed to the `"{$var}"` curly-brace-first form, which has been valid since PHP 4 and remains the correct form on all PHP versions.

```php
// Before (fatal on PHP 8.3):
"${name}expecting ..."
"op_${ltype}_${rtype}"

// After:
"{$name}expecting ..."
"op_{$ltype}_{$rtype}"
```

---

### P-14 — `mhash()` removed (PHP 8.1)
- **Severity:** Fatal (`Call to undefined function mhash()`) on PHP 8.1+
- **File:** `libraries/src/User/UserHelper.php`
- **Issue:** Four calls to the removed `mhash()` function were present, used to produce raw binary SHA1 and MD5 digests for legacy password hashing schemes (`SHA1`, `MD5`, `SHA1Salted`, `MD5Salted`).
- **Fix:** Replaced with `hash($algo, $data, true)`. The third argument `true` returns raw binary output, matching `mhash()`'s byte-for-byte output — this is critical for verifying existing stored passwords.

```php
// Before:
base64_encode(mhash(MHASH_SHA1, $plaintext))
base64_encode(mhash(MHASH_MD5, $plaintext))
base64_encode(mhash(MHASH_SHA1, $plaintext . $salt) . $salt)
base64_encode(mhash(MHASH_MD5, $plaintext . $salt) . $salt)

// After:
base64_encode(hash('sha1', $plaintext, true))
base64_encode(hash('md5', $plaintext, true))
base64_encode(hash('sha1', $plaintext . $salt, true) . $salt)
base64_encode(hash('md5', $plaintext . $salt, true) . $salt)
```

---

### P-15 — `strftime()` deprecated (PHP 8.1) / removed (PHP 9)
- **Severity:** `E_DEPRECATED` on PHP 8.1; fatal on PHP 9
- **Files patched:**
  - `libraries/src/HTML/HTMLHelper.php`
  - `libraries/joomla/form/fields/calendar.php`
- **Issue:** `strftime()` was used to format date values for display in calendar fields. PHP 8.1 deprecated `strftime()` and PHP 9 will remove it.
- **Fix:**
  - Added a new `public static function strftimeToDateFormat(string $strftimeFormat): string` method to `HTMLHelper`. It uses `strtr()` with a static map to convert `strftime` format specifiers (e.g. `%Y`, `%m`, `%d`) to their `date()` equivalents (e.g. `Y`, `m`, `d`). All standard `%`-specifiers are covered.
  - Both call sites updated to `date(static::strftimeToDateFormat($format), strtotime($value))` / `date(JHtml::strftimeToDateFormat($this->format), strtotime($this->value))`.
  - The method is declared `public` (not `protected`) because `calendar.php` calls it externally via the `JHtml` alias registered in `libraries/classmap.php`.

---

### P-16 — `utf8_encode()` in Twitter library (PHP 8.2)
- **Severity:** Fatal (`Call to undefined function utf8_encode()`) on PHP 8.2+
- **File:** `libraries/joomla/twitter/statuses.php`
- **Issue:** Two remaining calls to the removed `utf8_encode()` were present in the Twitter API status update methods; the first-pass fix (P-1) had not covered this file.
- **Fix:** Both replaced with `mb_convert_encoding($status, 'UTF-8', 'ISO-8859-1')`.

---

# Joomla 3.12.0 — Patch Log

**Date:** May 21, 2026
**Base version:** Joomla 3.11.0
**Patched version:** Joomla 3.12.0

---

## PHP 8.x Compatibility Fixes — Session 3

### Q-1 — `#[\AllowDynamicProperties]` on base classes (PHP 8.2)
- **Files:** `libraries/src/Table/Table.php`, `libraries/src/Object/CMSObject.php`, `libraries/idna_convert/idna_convert.class.php`
- **Issue:** PHP 8.2 deprecated dynamic property creation. `Table` uses `bind()` to set arbitrary database column names as properties; `CMSObject` uses `set()` for arbitrary property names. Both are intentional by design. `idna_convert` has internal dynamic properties.
- **Fix:** Added `#[\AllowDynamicProperties]` attribute to all three classes. On PHP 7.x the `#` is a line comment so the attribute line is silently ignored — fully backward-compatible. The attribute propagates to all subclasses, fixing every concrete Table subclass and every JObject descendant in one change.

### Q-2 — Remaining implicit-nullable typed parameters (PHP 8.1/8.5)
- **Scope:** 17 files missed by the P-10 automated pass
- **Files:**
  - `libraries/fof/form/header.php`, `libraries/fof/encrypt/aes.php`, `libraries/fof/encrypt/aes/interface.php`, `libraries/fof/encrypt/aes/mcrypt.php`, `libraries/fof/encrypt/aes/openssl.php`, `libraries/fof/database/query.php`, `libraries/fof/database/factory.php` — FOF library
  - `libraries/vendor/joomla/session/Joomla/Session/Session.php`
  - `libraries/vendor/joomla/data/src/DataObject.php`, `DataSet.php`, `DumpableInterface.php`
  - `libraries/vendor/joomla/di/src/Container.php`
  - `libraries/vendor/google/recaptcha/src/ReCaptcha/ReCaptcha.php`
  - `libraries/vendor/symfony/yaml/Exception/ParseException.php`
  - `libraries/vendor/joomla/filesystem/src/Exception/FilesystemException.php`
  - `plugins/privacy/actionlogs/actionlogs.php`, `message/message.php`, `content/content.php`, `contact/contact.php`, `consents/consents.php`, `user/user.php` (3 occurrences)
- **Fix:** `Type $p = null` → `?Type $p = null` on all affected signatures.

---

## Security Patches — Session 3 (backported from Joomla 4/5/6 via TLWebdesign/Joomla-3-EOL-Security-Fixes audit)

### S-1 — CVE-2025-63082 — Data URI XSS in InputFilter
- **File:** `libraries/vendor/joomla/filter/src/InputFilter.php`
- **Change:** After the existing whitespace stripping in `checkAttribute()`, added a check that rejects any `data:` URI unless it matches `^data:image/(png|gif|jpe?g|webp);base64,`. Previously marked "4.0.0+ only" in 3.11 — this was incorrect; the filter class is shared and the vulnerability is present in 3.x.

### S-2 — CVE-2024-40747 — XSS via module chrome attributes
- **File:** `libraries/src/Helper/ModuleHelper.php`
- **Change:** In `renderModule()`, all string values in `$attribs` are now run through `htmlspecialchars(ENT_QUOTES, 'UTF-8')` after the `onRenderModule` event fires and before they are passed to `modChrome_*` template functions, which echo them directly into HTML.

### S-3 — CVE-2025-25226 — SQL injection via database identifier names
- **File:** `libraries/joomla/database/driver.php`
- **Change:** `quoteNameStr()` now iterates over all identifier parts before quoting and throws `InvalidArgumentException` if any part contains a null byte (`\x00`) or a backslash (`\`). Both can break out of the quoting delimiters.

### S-4 — CVE-2026-21631 — XSS in com_associations side-by-side editor
- **File:** `administrator/components/com_associations/views/association/tmpl/edit.php`
- **Change:** `$this->referenceTitle`, `$this->referenceTitleValue`, and `$this->targetTitle` were echoed raw into `data-*` HTML attributes. All three are now wrapped with `$this->escape()`. Previously marked "4.0.0+ only" — this was incorrect; `com_associations` has been in core since Joomla 3.7.

---

## Update Server Infrastructure

Two-file architecture (mirrors `update.joomla.org` exactly — see "Update server" in the guidance section above for the full explanation):

- **`docs/list.xml`** — `<extensionset>` collection. Each `<extension>` entry has `version`, `targetplatformversion` (regex against `JVERSION`), and `detailsurl` pointing to `extension.xml`. Served at `https://joomlaworks.github.io/joomla-3.x/list.xml`.
- **`docs/extension.xml`** — `<updates>` details file with the actual `<update>` block (download URL, `<targetplatform>`, `<php_minimum>`, etc.). Served at `https://joomlaworks.github.io/joomla-3.x/extension.xml`. This is what `com_joomlaupdate` fetches to display the update and get the package URL.
- **`docs/.nojekyll`** — Prevents GitHub Pages from running Jekyll on the `docs/` folder (required so XML files are served raw).
- **`.github/workflows/rolling-release.yml`** — On every push to `main`: runs `git archive --format=zip HEAD -o joomla-latest.zip`, deletes and recreates the `rolling` GitHub Release. The download URL `…/releases/download/rolling/joomla-latest.zip` is permanent. `git archive` produces a root-level zip (no subdirectory prefix), which is required by Kickstart inside `com_joomlaupdate`.
- **Installation SQL** (all three DB variants) — `#__update_sites` row 1 (`type='collection'`) now points to `https://joomlaworks.github.io/joomla-3.x/list.xml` instead of `update.joomla.org`.
- **`com_joomlaupdate/models/default.php`** — All `$updateURL` cases (default, next, testing) now point to the GitHub Pages feed.

---

## Removed Extensions & Templates

The following items were removed from the distribution entirely in 3.12:

| Item | Type | Reason |
|------|------|--------|
| `plugins/quickicon/eos310` | Plugin | Joomla 3.x end-of-service notice — irrelevant in this distribution |
| `plugins/quickicon/phpversioncheck` | Plugin | PHP version nag — irrelevant given our active PHP 8.x support |
| `templates/beez3` | Frontend template | Legacy, unmaintained; protostar is the maintained default |
| `administrator/templates/hathor` | Backend template | Legacy, unmaintained; isis is the maintained default |

Also removed: associated language files in `administrator/language/en-GB/` and `language/en-GB/`, and the `media/plg_quickicon_eos310/` JS directory.

**Installation SQL** — all four extension rows, their `#__template_styles` entries (style IDs 4 and 5), and the hathor `#__postinstall_messages` entry were removed from all three installation SQL files.

**Migration SQL** — `administrator/components/com_admin/sql/updates/{mysql,postgresql,sqlazure}/3.12.0-2026-05-21.sql` runs automatically on upgrade via `com_joomlaupdate`. It:
1. Reassigns any site using beez3/hathor as their global default template to protostar/isis (using a derived-subquery `EXISTS` check to avoid MySQL's "can't update target table" restriction)
2. Deletes all `#__template_styles` rows for beez3 and hathor
3. Deletes the four `#__extensions` records
4. Deletes the hathor `#__postinstall_messages` record
5. Cleans up any orphaned `#__update_sites_extensions` rows

---

## Post-Release Bug Fixes — May 22, 2026

Three bugs discovered after the 3.12.0 release was published, fixed before the next version bump.

### B-1 — Update feed format wrong: `<updates>` instead of `<extensionset>` in `list.xml`
- **Files:** `docs/list.xml` (rewritten), `docs/extension.xml` (new file)
- **Root cause:** The `#__update_sites` entry for the Joomla core has `type='collection'`. This means Joomla uses `CollectionAdapter` to parse `list.xml`. `CollectionAdapter` only handles `<extensionset>/<extension>` elements — it silently ignores `<updates>/<update>` content entirely. The original `list.xml` was in `<updates>` format, so every update check returned nothing.
- **Fix:** Rewrote `docs/list.xml` as an `<extensionset>` collection with one `<extension>` entry per supported minor version (3.10, 3.11, 3.12), each pointing `detailsurl` to `docs/extension.xml`. Created `docs/extension.xml` as the `<updates>` details file containing the download URL and `<targetplatform>` check. This matches the `update.joomla.org/core/list.xml` + `extension.xml` architecture exactly.

### B-2 — `MINOR_VERSION = 11` in `Version.php` (should be 12)
- **File:** `libraries/src/Version.php`
- **Root cause:** When bumping from 3.11 to 3.12, `MINOR_VERSION` was not updated. Since `JVERSION` is computed as `MAJOR_VERSION . '.' . MINOR_VERSION . '.' . PATCH_VERSION` (via `getShortVersion()` in `libraries/cms.php`), the constant `JVERSION` evaluated to `'3.11.0'` instead of `'3.12.0'`. The deprecated `RELEASE` constant was correctly set to `'3.12'`, but nothing used it for the version constant.
- **Impact:** After upgrading to 3.12, the site would still report `JVERSION = '3.11.0'`. The `com_joomlaupdate` "hasUpdate" check (`version_compare($latest, JVERSION, '>')`) would then evaluate `version_compare('3.12.0', '3.11.0', '>') = true` — meaning the update notification would reappear on every check even after a successful upgrade.
- **Fix:** `MINOR_VERSION` changed from `11` to `12`.

### B-4 — PHP 8.5: `(boolean)` cast deprecated
- **Files:** `libraries/src/Date/Date.php`, `libraries/src/Helper/TagsHelper.php`, `libraries/src/Table/ContentHistory.php`, `libraries/src/Table/Table.php`, `libraries/src/Log/Logger/FormattedtextLogger.php`, `libraries/src/Language/Text.php`, `libraries/src/Language/Associations.php`, `libraries/src/Layout/BaseLayout.php`, `libraries/src/Microdata/Microdata.php`, `libraries/src/Access/Rule.php`, `libraries/joomla/database/iterator.php`, `libraries/joomla/database/exporter.php`, `libraries/joomla/database/importer.php`
- **Root cause:** PHP 8.5 deprecated the non-canonical `(boolean)` cast alias in favour of `(bool)`. 17 occurrences across 13 files. The first deprecation notice to appear (from `Date.php` during CLI bootstrap) contributes to the "headers already sent" cascade.
- **Fix:** `sed -i 's/(boolean)/(bool)/g'` across all 13 files.

### B-5 — PHP 8.5: `null` used as array offset in `Uri::getInstance()`
- **File:** `libraries/src/Uri/Uri.php`
- **Root cause:** `Uri::getInstance($uri)` uses `$uri` as a key in `static::$instances[$uri]`. In CLI context, callers such as `WebApplication` pass `$this->get('uri.request')` which returns `null` when the registry key is unset. PHP 8.5 deprecated using `null` as an array offset. The resulting deprecation notice is the first line of output in a CLI script, which triggers the fatal "headers already sent" session startup failure (same cascade as P-7/P-8).
- **Fix:** Added `if ($uri === null) { $uri = 'SERVER'; }` guard at the top of `getInstance()`, before the array key is first used (line 59, 119, 122 in original). `'SERVER'` is the default value already used when no argument is passed, making this semantically correct.

### B-3 — Migration SQL used wrong column name `language_key` on `#__postinstall_messages`
- **Files:** `administrator/components/com_admin/sql/updates/mysql/3.12.0-2026-05-21.sql`, `…/postgresql/…`, `…/sqlazure/…`
- **Root cause:** The `#__postinstall_messages` table has no `language_key` column. The correct column for the language key is `title_key`. The migration SQL's `DELETE FROM #__postinstall_messages WHERE language_key = 'TPL_HATHOR_MESSAGE_POSTINSTALL_TITLE'` triggered MySQL error 1054 ("Unknown column 'language_key' in 'where clause'") mid-migration on first upgrade attempt.
- **Fix:** Column name changed from `language_key` to `title_key` in all three SQL variants. Sites that hit this error mid-migration need to run the corrected DELETE manually: `DELETE FROM #__postinstall_messages WHERE title_key = 'TPL_HATHOR_MESSAGE_POSTINSTALL_TITLE';`

---

# Joomla 3.13.0 — Patch Log

**Date:** May 31, 2026
**Base version:** Joomla 3.12.0
**Patched version:** Joomla 3.13.0

---

## PHP 8.x Compatibility Fixes — Session 4

The following fixes were identified from a community report posted in GitHub Discussions (user MaghSamana). All are backward-compatible with PHP 7.4.

### Q-3 — `htmlspecialchars(null)` in `HtmlView::escape()` (PHP 8.1)
- **File:** `libraries/src/MVC/View/HtmlView.php`
- **Issue:** `escape()` passed `$var` directly to `htmlspecialchars()` or a custom escape function. When `$var` is `null` (common for optional database fields), PHP 8.1 deprecated passing `null` to `htmlspecialchars()`. This fires on every view that renders a null field through `escape()`.
- **Fix:** Added `if ($var === null) { return ''; }` guard at the top of `escape()` before any function call.

### Q-4 — `strtoupper(null)` in `ListModel::populateState()` (PHP 8.1)
- **File:** `libraries/src/MVC/Model/ListModel.php` (two occurrences, lines 568 and 616)
- **Issue:** `$value` comes from `getUserStateFromRequest()` which returns `null` when the key is absent and no default is set. `strtoupper(null)` was deprecated in PHP 8.1.
- **Fix:** Changed both to `strtoupper((string) $value)`.

### Q-5 — `DateTime::__construct(null)` in `Date::__construct()` (PHP 8.1)
- **File:** `libraries/src/Date/Date.php`
- **Issue:** The parent `DateTime::__construct()` was called with `$date` which can be `null` if a caller passes `null` explicitly (e.g. `new JDate(null)` from third-party extensions). PHP 8.1 deprecated passing `null` where a `string` is expected.
- **Fix:** Changed to `parent::__construct($date ?? 'now', $tz)`.

### Q-6 — Dynamic property `$registeredurlparams` on `CMSApplication` (PHP 8.2)
- **File:** `libraries/src/Application/CMSApplication.php`
- **Issue:** `BaseController` (and the FOF controller) assign `$app->registeredurlparams` directly on the application object in both frontend and backend contexts. The property was never declared in any application class, triggering PHP 8.2 dynamic property deprecation on every cacheable request from any controller that registers URL params.
- **Fix:** Declared `public $registeredurlparams = null;` in `CMSApplication` (the common base for `SiteApplication` and `AdministratorApplication`).

### Q-7 — `trim(null)` / `ltrim(null)` / `rtrim(null)` in phputf8 (PHP 8.1)
- **File:** `libraries/vendor/joomla/string/src/phputf8/trim.php`
- **Issue:** `utf8_ltrim()`, `utf8_rtrim()`, and `utf8_trim()` all call the native PHP trim functions with `$str` which can be `null`. PHP 8.1 deprecated passing `null` to these string functions.
- **Fix:** Added `(string)` cast: `ltrim((string) $str)`, `rtrim((string) $str)`, `trim((string) $str)` in the short-circuit early-return paths of all three functions.

### Q-8 — `trim(null)` in `Registry/Format/Json::stringToObject()` (PHP 8.1)
- **File:** `libraries/vendor/joomla/registry/src/Format/Json.php`
- **Issue:** `$data` can be `null` when passed from upstream code. PHP 8.1 deprecated `null` to `trim()`.
- **Fix:** Changed to `$data = trim((string) $data);`.

### Q-9 — Dynamic property `$itemTags` on `TagsHelper` (PHP 8.2)
- **File:** `libraries/src/Helper/TagsHelper.php`
- **Issue:** `getItemTags()` assigns `$this->itemTags = $db->loadObjectList()` without the property being declared in the class. Triggers PHP 8.2 dynamic property deprecation whenever tags are loaded.
- **Fix:** Declared `public $itemTags = array();` in the class body, before `$typeAlias`.

### Q-10 — Dynamic properties `$empty` and `$dates` on `FinderIndexerQuery` (PHP 8.2)
- **File:** `administrator/components/com_finder/helpers/indexer/query.php`
- **Issue:** Both `$this->empty` and `$this->dates` are assigned in `__construct()` without class-level declarations, triggering PHP 8.2 dynamic property deprecation whenever a Smart Search query is constructed.
- **Fix:** Declared `public $empty = false;` and `public $dates;` in the class body after `$when2`.

---

## CLI Compatibility Fix

### Q-11 — Undefined `HTTP_HOST` warning in CLI context in `Uri::getInstance()`
- **File:** `libraries/src/Uri/Uri.php`
- **Issue:** When building the server URI (`$uri == 'SERVER'`), both the Apache path (line 95) and the IIS/fallback path (line 106) accessed `$_SERVER['HTTP_HOST']` directly. `HTTP_HOST` is never set in CLI context. The resulting PHP warning printed to stdout, poisoning CLI output and cascading into the "headers already sent" session startup failure. This is separate from the B-5 fix (which guarded against `null` as the `$uri` argument itself).
- **Fix:** Added `$httpHost = isset($_SERVER['HTTP_HOST']) ? $_SERVER['HTTP_HOST'] : 'localhost';` before the Apache/IIS conditional, and replaced both raw `$_SERVER['HTTP_HOST']` accesses with `$httpHost`.

---

## Infrastructure & Update Fixes

### I-1 — Auto-fix schema mismatch on upgrade
- **File:** `administrator/components/com_admin/script.php`
- **Issue:** The `update()` method called `updateDatabase()` (MySQL engine change + plugin cleanup only) but never ran the SQL migration files in `sql/updates/`. After every upgrade, the Extensions → Manage → Database view showed "Database update version (old) does not match CMS version (new)", requiring a manual "Fix" click that ran `JSchemaChangeset::fix()` + updated `#__schemas` + updated `manifest_cache.version`.
- **Fix:** Added protected method `fixSchemas()` and called it from `update()` after `updateDatabase()`. The method: (1) instantiates `JSchemaChangeset` over `sql/updates/`; (2) calls `fix()` to apply all pending migration files; (3) updates the `#__schemas` row for extension_id 700 to the latest SQL file version; (4) syncs `manifest_cache.version` for extension_id 700 to `$cmsVersion->getShortVersion()`.

### I-2 — `joomla.xml` version stale + wrong update server URL
- **File:** `administrator/manifests/files/joomla.xml`
- **Issue:** The manifest `<version>` field was still `3.10.20-elts` (never updated from the original eLTS value). Since `updateManifestCaches()` reads this file and writes to `manifest_cache`, every upgrade stored `3.10.20-elts` as the Joomla extension version — the actual root cause of the "database update version does not match" banner. Additionally, `<updateservers>` still pointed to `https://update.joomla.org/core/list.xml`; the JInstaller processes this during an upgrade and would silently revert `#__update_sites` row 1 back to the official Joomla server, breaking future update notifications.
- **Fix:** Updated `<version>` to the current release version and `<creationDate>` to the release date. Changed `<updateservers>` URL to `https://joomlaworks.github.io/joomla-3.x/list.xml`.
- **Ongoing:** This file must be updated on every version bump (added to the release checklist as step 4).

### I-3 — Filesystem cleanup missing for 3.12 removed items
- **File:** `administrator/components/com_admin/script.php` (`deleteUnexistingFiles()`)
- **Issue:** The 3.12 migration SQL correctly removed DB records for `beez3`, `hathor`, `eos310`, and `phpversioncheck` — but Kickstart (the zip extractor used by `com_joomlaupdate`) only overlays files and never deletes anything absent from the package. Files were left on disk. `deleteUnexistingFiles()` is the correct mechanism but the 3.12 entries were never added.
- **Fix:** Added a `// Joomla 3.12.0` block to `$files` (language files for all four items) and `$folders` (`/templates/beez3`, `/administrator/templates/hathor`, `/plugins/quickicon/eos310`, `/plugins/quickicon/phpversioncheck`, `/media/plg_quickicon_eos310`).

### I-4 — `com_joomlaupdate` complete screen shows old version (OPcache)
- **Files:** `administrator/components/com_joomlaupdate/controllers/update.php`, `administrator/components/com_joomlaupdate/views/default/tmpl/complete.php`
- **Issue:** After an upgrade, the "Your Joomla version is now X.Y.Z" message on the complete screen used `JVERSION`. PHP-FPM runs multiple worker processes each with their own OPcache. `opcache_reset()` is called during `finalise()` but only resets the current worker; the `cleanup()` and `complete` requests can land on different workers with stale `Version.php` still cached, so `JVERSION` evaluates to the pre-upgrade value.
- **Fix:** In `cleanup()`, after `$model->cleanUp()`: read the new version from `administrator/manifests/files/joomla.xml` on disk using `simplexml_load_file()` (XML files are never bytecode-cached by OPcache); store it in `com_joomlaupdate.newversion` user state. In `complete.php`: read from that session key (falling back to `JVERSION`) and clear it immediately after use.

---

# Joomla 3.14.0 — Patch Log

**Date:** July 4, 2026
**Base version:** Joomla 3.13.0
**Patched version:** Joomla 3.14.0

---

## PHP 8.x Compatibility Fixes — Session 5

Both fixes below were contributed via GitHub PR #13 (github.com/joomlaworks/joomla-3.x/pull/13) by community member @raramuridesign. Verified against upstream PHP 8.5 deprecation notices before merging; both are backward-compatible with PHP 7.4.

### Q-12 — `null` used as array offset in `HtmlDocument::getBuffer()` / `setBuffer()` (PHP 8.5)
- **File:** `libraries/src/Document/HtmlDocument.php`
- **Issue:** `getBuffer($type, $name, $attribs)` and `setBuffer()` index `parent::$_buffer[$type][$name][$title]` directly. `$name` defaults to `null` and `$title` (pulled from `$attribs['title']`) is `null` whenever no title is set — the common case for most modules/components. PHP 8.5 deprecates using `null` as an array offset (same class of issue as B-5/Q-11 against `Uri::getInstance()`), so this fired on nearly every page render.
- **Fix:** In `getBuffer()`, added local `$bufferName`/`$bufferTitle` set to `''` when `$name`/`$title` are `null`, and used them for the buffer-indexing reads (the `isset()` check, the cached-`$data` read, and the final return). In `setBuffer()`, `$options['type']`/`['name']`/`['title']` are read through `?? ''` before being used as array keys, so both the write path and the read path converge on `''` as the actual stored key.

### Q-13 — `imagedestroy()` deprecated (PHP 8.5); stray `(boolean)` cast
- **Files:** `libraries/vendor/joomla/image/src/Image.php`, `libraries/vendor/joomla/image/src/Filter/Backgroundfill.php`
- **Issue:** `Image::destroy()` called `imagedestroy($this->getHandle())`. Since PHP 8.0, GD uses refcounted `GdImage` objects instead of resources, so `imagedestroy()` has had no effect for several major versions; PHP 8.5 deprecates calling it at all. `Backgroundfill::execute()` had a second occurrence — `imagedestroy($bg)` on a local temporary handle — found via codebase-wide grep after fixing the first (confirmed `imagedestroy` had no other call sites anywhere in the tree). Also found one remaining `(boolean)` cast in `Image::setThumbnailGenerate()`, missed by the B-4 sweep in 3.12 (that pass's file list didn't include this vendor file).
- **Fix:** `Image::destroy()` now sets `$this->handle = null` and returns `true` directly instead of calling `imagedestroy()` — `isLoaded()` (via `isValidImage($this->handle)`) still correctly reports `false` afterward, preserving the method's observable contract. In `Backgroundfill::execute()`, the `imagedestroy($bg)` call was simply removed — `$bg` is a local variable that goes out of scope at the end of the method, so the `GdImage` object is reclaimed by normal refcounting with no explicit call needed. `(boolean)` → `(bool)` in `setThumbnailGenerate()`.

---

## Language File Version Fix

### L-1 — Stale `3.10.20` version in en-GB language metafiles
- **Files:** `administrator/language/en-GB/en-GB.xml`, `administrator/language/en-GB/install.xml`, `language/en-GB/en-GB.xml`, `language/en-GB/install.xml`, `administrator/manifests/packages/pkg_en-GB.xml`
- **Issue:** These files still carried `<version>3.10.20</version>` (and `3.10.20.1` for the package manifest) — the original eLTS base version, never updated across the 3.11/3.12/3.13 bumps. Some vulnerability scanners fingerprint the Joomla version from these language metafiles (readable pre-auth, unlike `JVERSION` which requires more work to expose), so sites running this distribution were being flagged as "Joomla 3.10.20" and matched against CVEs already patched here.
- **Fix:** Bumped `<version>` to `3.14` (`pkg_en-GB.xml` to `3.14`, dropping the redundant `.1` patch suffix) and `<creationDate>` to `July 4th, 2026` in all five files, aligning them with the actual running version.
- **Open question:** see "Open considerations for future sessions" at the top of this file — reporting the true version here is more accurate but also lets scanners precisely fingerprint this fork. Not resolved in this release.

---

## Version Bump

**File:** `libraries/src/Version.php`

| Constant | Before | After |
|----------|--------|-------|
| `MINOR_VERSION` | `13` | `14` |
| `PATCH_VERSION` | `0` | `0` |
| `RELEASE` (deprecated) | `'3.13'` | `'3.14'` |
| `RELDATE` | `'31-May-2026'` | `'4-July-2026'` |

Also updated per the release checklist: `administrator/manifests/files/joomla.xml` (`<version>3.14.0</version>`, `<creationDate>July 4th, 2026</creationDate>`), `docs/list.xml` (all `<extension>` rows bumped to `version="3.14.0"`, new row added for `targetplatformversion="3.14"`), `docs/extension.xml` (`<version>3.14.0</version>`, name/description/infourl updated to 3.14).

The installation now reports itself as **Joomla! 3.14.0**.

---

# Joomla 3.15.0 — Patch Log

**Date:** July 9, 2026
**Base version:** Joomla 3.14.0
**Status:** Code changes for all 8 confirmed backports are done and verified (see below). Version bump (`Version.php`, `joomla.xml`, `docs/list.xml`, `docs/extension.xml`, language files) has **not** been performed yet — CHANGELOG.md/README.md headers are marked `[pending]` intentionally. Do the version bump only when the maintainer confirms the review round below is closed (they are independently sourcing upstream diffs for the "uncertain" items first).

---

## Source: official Joomla security advisories review

At the maintainer's request, the full [Joomla Security Centre feed](https://developer.joomla.org/security-centre.html?format=feed) was reviewed — 25 advisories published May 26 and July 7, 2026 (CVE-2026-35223 through CVE-2026-48958) — to determine which also affect 3.x. The maintainer explicitly does not trust the "Affected Installs" ranges published by the Joomla security team at face value, since 3.x is no longer tested by them. Every advisory was independently verified against this codebase (component/file/pattern presence) rather than trusting the stated range.

**Result: 8 confirmed applicable (fixed below), 4 flagged uncertain (not fixed — see "Deferred" below), 13 confirmed not applicable** (mostly because the Joomla 4.0+ webservices/API layer, com_scheduler, com_workflow, and the 4.2+ MFA method-management redesign do not exist anywhere in this codebase — verified by grep/find, not assumed). Full per-CVE reasoning for all 25 was given to the maintainer in-conversation; only the actionable subset is logged here to keep this file focused. If revisiting: re-run the same review methodology (fetch advisory text + grep the codebase for the named component/method) rather than trusting version ranges.

Two of the 8 backported CVEs were labelled "affects 4.0.0+ only" upstream but were confirmed to affect 3.x anyway by finding the actual vulnerable code present and unpatched here — see T-7 and T-8 below. This validates the maintainer's skepticism.

---

## Security Patches — Session 4 (backported from official Joomla advisories)

### T-1 — CVE-2026-40383 — LFI via view layout parameter (High)
- **File:** `libraries/src/MVC/View/HtmlView.php`, `setLayout()`
- **Issue:** `BaseController::display()` reads the `layout` request parameter through the `'string'` filter (not `'cmd'`, so `:`, `.`, `/` survive). `HtmlView`'s constructor passes it to `setLayout()`, which splits on `:` and stores the pre-colon segment as `$this->_layoutTemplate` with zero sanitization. `loadTemplate()` then does an unguarded `str_replace()` of the current template name for `$layoutTemplate` inside the template search path (`$this->_path['template']`), before `JPath::find()` + `include`. Only the trailing filename portion is separately regex-sanitized; the directory portion was not.
- **Fix:** `$this->_layoutTemplate = preg_replace('/[^A-Z0-9_\-]/i', '', $temp[0]);` — restricts the template-override segment to safe identifier characters, closing the traversal at the source. No other `setLayout()` implementation in the codebase (`fof/view/view.php`, `joomla/view/html.php`, `Layout/FileLayout.php`) has this template-switching mechanic, so only this one file needed the fix.

### T-2 — CVE-2026-48902 — Password/username reset links downgraded to plain HTTP (Low)
- **Files:** `components/com_users/models/reset.php`, `components/com_users/models/remind.php`
- **Issue:** Both built their confirmation link with `$mode = $config->get('force_ssl', 0) == 2 ? 1 : (-1);`. Per `libraries/src/Router/Route.php`, `-1` legacy-maps to `TLS_DISABLE` (force `http://`), so unless "Force SSL: Entire Site" was explicitly enabled, the link was forced to plain HTTP even on a site served entirely over HTTPS (e.g. via reverse proxy/CDN TLS termination) — leaking the reset token in cleartext.
- **Fix:** Changed the fallback from `(-1)` to `0` (`Route::TLS_IGNORE` — "use the protocol currently used in the request"), the natural/correct default. The `force_ssl == 2` branch (explicit HTTPS force) is unchanged.

### T-3 — CVE-2026-48905 — Nested/double-encoded entity bypass in the attribute filter (Moderate) — extra hardening, not the real upstream fix
- **File:** `libraries/vendor/joomla/filter/src/InputFilter.php` — `checkAttribute()` and `cleanAttributes()`
- **Issue (as hypothesized before the real diff was available):** Both methods called `html_entity_decode()` exactly once, which could theoretically let nested/double-encoded entities survive detection.
- **Fix applied:** Both methods now decode entities in a capped fixed-point loop before pattern-matching, and `cleanAttributes()` stores the same decoded value it validates rather than the raw original. Verified via direct execution: legitimate content preserved, `javascript:`/numeric-entity/double-encoded colon variants all correctly rejected, sub-millisecond performance.
- **CORRECTION (2026-07-09, after obtaining the real upstream diff — see "Real upstream diff obtained" below):** This was **not** the actual CVE-2026-48905 fix. The real upstream fix (joomla-framework/filter PR #87) adds an explicit `data:text/html` block in `cleanAttributes()`. That exact protection was **already present in our codebase** before this session, via the broader **S-1 (CVE-2025-63082)** fix, which blocks *all* `data:` URIs except safe base64 images (`libraries/vendor/joomla/filter/src/InputFilter.php`, `checkAttribute()`) — confirmed by directly testing the official upstream test case (`<a href="data:text/html,<script>alert(1)</script>">Data link</a>` → correctly reduces to `<a>Data link</a>`). **CVE-2026-48905 was already fixed pre-session.** The decode-loop hardening above is kept as a harmless additional defense-in-depth measure (verified not to break anything) but should not be cited as "the CVE-2026-48905 fix" going forward.
- **Bug caught during implementation (kept for the historical record — still relevant to anyone touching this file):** `cleanAttributes()` has an outer `for ($i = 0; $i < $count; $i++)` loop over attribute pairs. The first draft of the inner decode loop also used `$i` as its counter — PHP `for` loops don't have block scope, so this silently clobbered the outer loop's iterator on every attribute processed, causing it to hang indefinitely (confirmed via direct `timeout php ...` testing, which is what caught it — a plain `php -l` syntax check does not). Renamed the inner counter to `$decodePass`. **Lesson for future edits to this file: never introduce a `for`/`foreach` loop variable named `$i`, `$j`, `$key`, etc. inside `cleanAttributes()` without checking it doesn't collide with the outer attribute-iteration loop.**

### T-3b — CVE-2026-48903 — checkAttribute filter code (Moderate) — RESOLVED: already fixed pre-session, no action needed
- **Real upstream diff obtained** (see "How the real diffs were found" below): joomla-framework/filter commit `6735e7051f` ("added cariiage return to xss filter blacklist", May 14 2026) adds `"\r"` to the whitespace-stripping list in `checkAttribute()`: `str_replace(["\t", "\n", " ", "\0"], ...)` → `str_replace(["\t", "\n", "\r", " ", "\0"], ...)`.
- **Our codebase already strips `\r` — and more.** The **H-1** hardening pass (documented in the 3.11 patch log, applied long before this CVE was even reported) already extended this exact `str_replace` call to `["\t", "\n", "\r", "\v", "\f", " ", "\0"]` — a strict superset of the official fix (we also strip `\v` vertical-tab and `\f` form-feed, which upstream's May 2026 fix does not). **No code change was needed or made for this CVE.**

### T-4 — CVE-2026-48954 — Stored XSS via language overrides (Moderate) — RESOLVED (2026-07-09)
- **CORRECTION history:** The 2026-07-04 session's claim that this was "fixed transitively by T-3" was wrong (retracted). The real vulnerability has nothing to do with `InputFilter`'s attribute-value filtering.
- **Root cause, confirmed by tracing an actual exploit chain (not just reading the advisory):** A non-`core.admin` user with edit rights on `com_languages` overrides (e.g. a delegated "Translator" ACL role — realistic in multi-admin sites; Super Users don't need this exploit since they're already fully trusted) can inject a quote character into an override value. Checked our own admin override list/edit views (`administrator/components/com_languages/views/{overrides,override}/tmpl/*.php`) and confirmed **those specific screens already escape output correctly** — the real sink is elsewhere: overridden strings are rendered via `JText::_()` **unescaped** almost everywhere across Joomla core and third-party extensions, by long-standing convention (language strings assumed trusted). Confirmed a concrete example: `administrator/components/com_tags/views/tags/tmpl/default.php:94` — `title="<?php echo JText::_('COM_TAGS_COUNT_PUBLISHED_ITEMS'); ?>"`. `InputFilter::cleanAttributes()`/`checkAttribute()` (already hardened via S-1/H-1/T-3) only sanitizes HTML **tag structures found inside the override text** — a bare payload like `x" onmouseover="fetch(...)` contains no `<`/`>` at all, so InputFilter's tag parser finds nothing to process and the payload passes through completely untouched, then breaks out of the surrounding hardcoded attribute at render time. This is a real, low-effort privilege-escalation path (plant once, fires for any admin who visits an ordinary screen like Components → Tags), not a theoretical one — auditing every `echo JText::_()` call across core plus every installed extension isn't feasible, which is exactly why the fix belongs at the input boundary.
- **Upstream fix (5.4.7):** New file `administrator/components/com_languages/src/Rule/DisallowQuotesRule.php` (a `FormRule` rejecting override text containing `"`/`'` unless the user has `core.admin`), wired via `validate="DisallowQuotes"` + `addruleprefix="Joomla\Component\Languages\Administrator\Rule"` on the field in `forms/override.xml`.
- **Ported to 3.x:**
  - New `administrator/components/com_languages/models/rules/languageoverridequotes.php` — class `JFormRuleLanguageoverridequotes extends JFormRule`, mirroring `test()` from the existing `components/com_users/models/rules/loginuniquefield.php` pattern (the working 3.x convention for component-scoped custom rules — confirmed by reading it, not assumed), using `?Registry`/`?JForm` nullable typed params (matching the base `FormRule::test()` signature exactly, and this project's PHP 7.4-safe convention) rather than the older example file's non-nullable style.
  - `administrator/components/com_languages/models/forms/override.xml` — added `addrulepath="administrator/components/com_languages/models/rules"` on `<form>`, and `validate="languageoverridequotes"` + `message="COM_LANGUAGES_ERROR_QUOTES_IN_TEXT"` on the `override` field.
  - New string `COM_LANGUAGES_ERROR_QUOTES_IN_TEXT` in `administrator/language/en-GB/en-GB.com_languages.ini`.
- **Side-effect safety (specifically checked, since custom-programmed extension fields like K2's were a concern):** `validate="..."` only fires for the one field it's declared on — nothing else changes. `addrulepath` adds an *additive* fallback path to a process-wide static search list (`FormHelper::$paths['rule']`), which only matters if some unrelated form also references the exact same rule name — mitigated by choosing a long, component-specific class/rule name (`languageoverridequotes`, not e.g. `quotes`) instead of copying upstream's generic `DisallowQuotes`; confirmed via repo-wide grep that nothing else uses this name. No K2/third-party field types, rendering, or validation are touched by this change in any way.
- **Verified with a standalone logic test** (mocking `JFormRule`/`JFactory`, since full framework bootstrap isn't practical for a quick check): non-admin + no quotes → valid; non-admin + `"` or `'` → invalid; Super User (`core.admin`) + quotes → valid. All four cases passed. `php -l` and `simplexml_load_file()` both clean on the new/modified files.

### T-5 — CVE-2026-48952 — XSS in com_installer update list view (Moderate)
- **File:** `administrator/components/com_installer/views/update/tmpl/default.php`
- **Issue:** `client_translated`, `type_translated`, `current_version`, `version`, `folder_translated`, `install_type`, and `detailsurl` were all echoed raw — all sourced from the `#__updates` table, populated by fetching XML manifests from (often third-party) extension update servers. The `infourl` anchor's `href` attribute was also unescaped (only the visible link text used `$this->escape()`).
- **Fix:** Wrapped all of the above in `$this->escape()`.

### T-6 — CVE-2026-48948 — Access control bypass in com_contact vCard export (Low)
- **File:** `components/com_contact/views/contact/view.vcf.php`
- **Issue:** The `&format=vcf` export builds and echoes the vCard directly from `$this->get('Item')` without ever checking `$item->access`/`$item->category_access` — unlike `view.html.php`, which enforces this via `getAuthorisedViewLevels()` before rendering. Any unauthenticated user could retrieve a restricted contact's name/email/phone/address by requesting `&format=vcf` directly.
- **Fix:** Added the identical access check used in `view.html.php` (confirmed `category_access` is populated by the shared `ContactModel::getItem()` used by both views) before building the vCard; returns HTTP 403 via `$app->setHeader('status', 403, true)` on failure, matching the HTML view's behavior.

### T-7 — CVE-2026-48953 — XSS in the generic image output layout (Moderate) — ⚠️ advisory range was wrong for 3.x
- **File:** `layouts/joomla/html/image.php`
- **Issue:** Only `src` and `alt` were escaped before the full `$displayData` array was handed to `ArrayHelper::toString()`, which concatenates every other key (`title`, `class`, `data-*`, etc.) into `key="value"` with zero escaping. Not triggered by core's own templates today (the actively-used content-image layouts already escape everything), but this is a public, documented layout API (`JLayoutHelper::render('joomla.html.image', ...)`) that third-party extensions call directly with arbitrary attributes.
- **Fix:** Escape every scalar value in `$displayData` in a loop before rendering, instead of only `src`/`alt`.
- **Advisory said "affects 4.0.0+ only" — this was wrong for our purposes.** The file and the vulnerable pattern are identical in this 3.x codebase (present since the original eLTS import). Backported anyway per the maintainer's standing instruction to verify ranges independently.

### T-8 — CVE-2026-48901 — Incorrect cache key construction for InputFilter instances (Low) — ⚠️ advisory range was wrong for 3.x
- **File:** `libraries/src/Filter/InputFilter.php`, `getInstance()`
- **Issue:** `$sig = md5(serialize(array($tagsArray, $attrArray, $tagsMethod, $attrMethod, $xssAuto)));` omitted the security-sensitive `$stripUSC` constructor parameter from the cache-key signature. A caller requesting a filter instance with `$stripUSC` disabled could silently receive an already-cached instance created by an earlier caller with it enabled, or vice versa — an unintended filtering configuration served from cache.
- **Fix:** Added `$stripUSC` to the array before serializing: `md5(serialize(array($tagsArray, $attrArray, $tagsMethod, $attrMethod, $xssAuto, $stripUSC)))`.
- **Advisory said "affects 4.0.0+ only" — this was wrong for our purposes.** The exact same code, with the same bug, is present in 3.x. Backported anyway.

---

## How the real diffs were found (2026-07-09) — technique for future sessions

Searching the Joomla CMS GitHub commit log, we couldn't locate a specific CVE patch by browsing `github.com/joomla/joomla-cms/commits/5.4-dev`, because apparently **Joomla develops security fixes privately and squashes them directly into the release tag**, not as visible individual commits on the public dev branch (the dev-branch history only shows an anticlimactic "Reset to dev" version-bump commit around each security release date). The reliable way to get the real diff:

1. Find the two release tags that bracket the fix (e.g. a CVE "Fixed: 2026-05-26" → compare the tag just before and just after, `5.4.5` and `5.4.6`).
2. `gh api "repos/joomla/joomla-cms/compare/5.4.5...5.4.6"` — the `.files[].patch` field gives the real, final diff regardless of how it was merged internally.
3. For framework-level packages (`joomla-framework/filter`, etc., "Project: Joomla! Framework" in the advisory rather than "Core") — check `composer.lock`'s diff for the package version bump; it often points to a transient `dev-*-release-*` branch name. That branch is usually deleted after release, but its merge commit persists in the package repo's git history (`gh api "repos/joomla-framework/filter/commits?sha=3.x-dev&since=...&until=..."` — look for a "Merge pull request #N from .../3.x-release-*" commit and diff it directly).

This is dramatically more reliable than guessing root causes from the advisory's terse prose (see T-3's corrected entry above for what happens when we guessed) or than browsing the dev branch commit log directly. **Use this technique first** for any future CVE investigation on this project.

---

## Security Patches — Session 4, continued (page 2 of the advisories feed, May 26 2026 batch)

At the maintainer's request, page 2 of the security feed (`?start=25`) was reviewed for anything published before May 2026 that might have been missed — everything older than May 2026 was already confirmed handled in prior sessions (cross-checked against the existing "CVEs Reviewed" tables and "already exists" list). Page 2 turned out to contain 7 *additional* advisories from the **same May 26, 2026 batch** as T-1 through T-8 above (paginated past the first page's cutoff), with real upstream diffs obtained via the technique above (all part of the `5.4.5...5.4.6` compare).

### T-9 — CVE-2026-352212 — SQL injection in com_tags (Moderate/High)
- **File:** `components/com_tags/models/tags.php`, `getListQuery()`
- **Issue:** `$orderDirection = $this->state->params->get('all_tags_orderby_direction', 'ASC');` was concatenated directly into `$query->order(...)` with no validation — `administrator/components/com_tags/config.xml`'s `all_tags_orderby_direction` radio field has no `validate="options"` constraint (matching upstream's pre-fix state exactly). The sibling single-tag model (`components/com_tags/models/tag.php`) already whitelists its equivalent value against `array('ASC', 'DESC', '')` — this second model was missed.
- **Fix:** Added the same whitelist check: `$orderDirection = in_array(strtoupper($orderDirection), array('ASC', 'DESC')) ? $orderDirection : 'ASC';` immediately after reading the param, mirroring `tag.php`'s existing pattern instead of upstream's form-level `validate="options"` (equivalent protection, applied at the point of query construction).

### T-10 — CVE-2026-25901 — XSS in com_associations (Moderate) — additional gap beyond the earlier S-4 fix
- **File:** `administrator/components/com_associations/views/association/tmpl/edit.php`
- **Issue:** The earlier **S-4 (CVE-2026-21631)** fix only escaped `referenceTitle`, `referenceTitleValue`, and `targetTitle`. This *separate* CVE covers the *other* unescaped `data-*` attributes on the same two `<iframe>` elements: `data-associatedview` (on the `<form>` tag), `typeName` (used in both `data-item` attributes), `referenceId`/`targetId` (`data-id`), `referenceLanguage`/`targetLanguage` (`data-language`), and `targetAction` (`data-action` on the target iframe — the reference iframe's `data-action="edit"` is a hardcoded literal, not affected).
- **Fix:** Wrapped all of the above in `$this->escape()`.

### T-11 — CVE-2026-30894 — XSS in com_contenthistory preview (Moderate)
- **File:** `administrator/components/com_contenthistory/views/preview/tmpl/preview.php`
- **Issue:** `$this->item->version_note`, `$subValue->value`, and `$value->value` (version-history field values, potentially attacker-influenced via a saved article/content revision) were echoed raw.
- **Fix:** Wrapped all three in `$this->escape()`, matching upstream exactly.

### T-12 — CVE-2026-25900 — XSS in feed modules (Moderate)
- **Files:** `modules/mod_feed/tmpl/default.php`, `administrator/modules/mod_feed/tmpl/default.php`
- **Issue:** Feed title, image `src`/`alt`, and per-item title/link were echoed raw or under-escaped in both the site and admin variants of `mod_feed` — content sourced from external, untrusted RSS/Atom feeds. The admin variant was worse than upstream's own pre-fix state: its `href` used `str_replace('&', '&amp;', $rssurl)` (only escapes `&`, not `<`/`>`/`"`) instead of `htmlspecialchars()`.
- **Fix:** All feed-sourced values now wrapped in `htmlspecialchars(..., ENT_QUOTES, 'UTF-8')` in both files (upgraded from `ENT_COMPAT` to `ENT_QUOTES` where already partially escaped, to also cover single-quote attribute contexts). `feed->description` deliberately left unescaped in both files, matching upstream — feed descriptions are treated as trusted HTML-formatted content by design.

### T-13 — CVE-2026-30895 — XSS in readmore links (Moderate)
- **Files:** `layouts/joomla/content/readmore.php`, `modules/mod_articles_category/tmpl/default.php`
- **Issue:** `JHtml::_('string.truncate', $item->title, $params->get('readmore_limit'))` was called with `$allowHtml` left at its default of `true` (see `libraries/cms/html/string.php:36`) — meaning any `<script>`/HTML embedded in an article title (e.g. by a lower-privileged author) is preserved and closed by `truncate()`, not stripped, then echoed raw into the readmore link text. (Titles were already escaped elsewhere in these same files, e.g. in `aria-label` attributes — this was specifically the visible link-text path.)
- **Fix:** Added explicit `true, false` as the 3rd/4th args (`$noSplit = true` unchanged, `$allowHtml = false` newly set) to all 7 call sites across both files (2 in `readmore.php`, 5 in `mod_articles_category/tmpl/default.php` — upstream's equivalent fix only touched one file, `mod_articles_category/tmpl/default_items.php`, because 5.x split this template differently; 3.x's `default.php` has the same vulnerable calls inline and needed the same fix applied at every call site, found via a codebase-wide grep for the pattern rather than assuming upstream's file list was complete for our layout).

### Investigated and confirmed NOT applicable
- **CVE-2026-35221 — SQL injection in com_finder.** Upstream fix swaps an array key/value pair in `Query.php` (`$this->filters[$modifier][$return->title] = (int) $return->id` → keyed by id instead of title) to stop a title string from later reaching a query as an array *key*. Our legacy-structured `administrator/components/com_finder/helpers/indexer/query.php` has the identical-*looking* pattern in 3 places (lines ~574, ~663, ~860) — but traced every downstream consumer of `Query::$filters` (`components/com_finder/models/search.php:265`, the only place it's used to build a query) and confirmed 3.x uses `implode(',', $groups[$i])` — `implode()` operates on array *values* (already `(int)`-cast IDs), never on the keys. The title-as-key structure is inert here; the vulnerable code path this CVE targets (title used as a key that reaches SQL) doesn't exist in 3.x's simpler, non-webservices search flow. Also consistent with the advisory's own stated range (`5.4.0+` only — does not even claim to affect 4.x or 3.x, unlike the other CVEs in this batch where we independently overrode an inaccurate range).
- **CVE-2026-35220 — CSRF in admin user-activation endpoint.** Advisory range is `6.0.0–6.1.0` only (not even 4.x/5.x) — confirmed accurate; the affected code (`administrator/components/com_users/src/Dispatcher/Dispatcher.php` task-inference logic) is 6.x-specific application-dispatcher architecture with no 3.x equivalent.

---

## Resolution of the 3 previously-deferred items (2026-07-09) — all 3 closed out using the real-diff technique

The 3 items left over from the first review pass were resolved by pulling the real upstream diffs (same `gh api compare` technique) rather than continuing to guess. Two are genuinely not applicable; one was a real, previously-unnoticed vulnerability and is now fixed — and turned out to be *broader* in 3.x than upstream's own fix.

### T-9-resolution — CVE-2026-48899 (sample data plugins ACL) — CONFIRMED NOT APPLICABLE
- Real upstream diff (`5.4.5...5.4.6`) touches 3 files: `plugins/sampledata/{blog,multilang,testing}/src/Extension/*.php`. Pattern: every `onAjaxSampledataApplyStepN()` handler was missing a `authorise('core.edit'|'core.manage'|..., 'com_X')` check alongside its existing `ComponentHelper::isEnabled('com_X')` check — meaning a user without edit rights on the target component could still get sample data created there via the sample-data installer's AJAX endpoints.
- **3.x has only the `blog` plugin** (`multilang` and `testing` don't exist — confirmed by directory listing). Read `plugins/sampledata/blog/blog.php` directly (not just re-trusting the earlier subagent summary): all 3 of its steps (`Step1`→com_content, `Step2`→com_menus, `Step3`→com_modules) **already** combine `JComponentHelper::isEnabled(...)` with `Factory::getUser()->authorise('core.create', ...)` on every step (lines 97, 320, 621) — predating this CVE. The specific gap upstream fixed (an unguarded "tags" step) doesn't exist in 3.x's simpler 3-step version. No code change needed.

### T-10-resolution — CVE-2026-48951 (modalreturn layouts XSS) — CONFIRMED NOT APPLICABLE
- Real upstream diff (`5.4.6...5.4.7`) touches 7 files, all named `modalreturn.php`, across com_categories/com_contact/com_content/com_menus/com_modules/com_newsfeeds/com_plugins — a full-page Bootstrap-5 "item saved, close this window" confirmation screen shown after a modal-picker save, echoing an unescaped `$title`.
- `find /home/fevangelou/Dropbox/WWW/Projects/joomla-3.x -iname "modalreturn.php"` returns **nothing** — this UI pattern doesn't exist anywhere in 3.x (3.x's older modal-picker flow doesn't have a dedicated post-save confirmation page). Not applicable; no equivalent file to fix.
- Note: the earlier session's tentative candidates (`com_modules/views/modules/tmpl/modal.php:110`, `positions/tmpl/modal.php:81,85`) are a *different*, coincidentally-similarly-named file class (the modal *picker list* itself, not a *modalreturn* confirmation page) and are **not** part of this CVE. Left uninvestigated — if the maintainer wants those looked at, it should be scoped as a separate, self-directed hardening item, not framed as a CVE backport.

### T-14 — CVE-2026-48950 — LFI-adjacent XSS in com_templates file manager (Moderate) — FIXED, broader than upstream
- **Files:** `administrator/components/com_templates/views/template/tmpl/default.php` (view), `administrator/components/com_templates/models/template.php` (root cause)
- **Real root cause (upstream diff, `5.4.6...5.4.7`):** the `file` GET parameter is base64-encoded and decoded server-side to resolve a template source/image/font file path. The earlier session's assumption ("`file` GET param is `cmd`-filtered, so no attacker-reachable output") missed that **`cmd`-filtering the outer base64 string does not constrain what the *decoded* bytes can contain** — PHP's `base64_decode()` is lenient enough that a `cmd`-safe input string (letters/digits/underscore/dot/hyphen only — no `+`/`/`/`=` needed) can still decode to arbitrary bytes, including a full `<script>` tag. Verified directly: `base64_decode('PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg')` → `<script>alert(1)</script>`.
- **3.x has this bug in *three* places, not the one upstream fixed:** `models/template.php` independently calls `base64_decode($app->input->get('file'))` with **zero** subsequent filtering in `getSource()` (line 434 → `$item->filename`), `getImage()` (line 1107 → `$image['path']`), and `getFont()` (line 1292 → `$relPath`/`$font['rel_path']`) — three parallel, unsanitized decode paths (contrast with the *view*'s own `$this->fileName` at `view.html.php:100`, which correctly runs the decoded value through `JFilterInput::getInstance()->clean(..., 'string')` — that sanitized copy just isn't what the template actually displays). All three raw values are echoed in `tmpl/default.php` (both inside the `COM_TEMPLATES_TEMPLATE_FILENAME` sprintf and in a separate hidden `<p class="lead path hidden">` element) — 6 echo sites total across the file/image/font branches. Upstream's fix only touched the "file" case's two echo sites; found the other 4 (image ×2, font ×2) via the "grep the whole codebase for the same vulnerable pattern, don't just trust upstream's file list" methodology established earlier this session (same approach that found com_finder's 3 occurrences and `mod_articles_category`'s 5).
- **Fix:** wrapped all 6 echo sites in `$this->escape()` — the minimal, output-side fix consistent with this project's established pattern (escape at the point of output rather than mutating the model), verified with `php -l` and the base64-payload test above.

---

## Functional Bug Fixes (reported by the maintainer, non-security)

### F-1 — Misleading "Refresh Manifest Cache failed: X Extension is not currently installed" warning on every Joomla Update run
- **File:** `administrator/components/com_admin/script.php`, `updateManifestCaches()`
- **Reported symptom:** after every successful core update via `com_joomlaupdate`, sites that have deliberately removed a stock core extension (e.g. the `protostar` template, or `com_banners`/`mod_banners` — removed by the site admin, not by this distribution) show a yellow "Warning: Refresh Manifest Cache failed: X Extension is not currently installed." box for each one, on every single future update. Confusing/alarming for something the admin already knows and intended.
- **Root cause, traced to the exact line:** `updateManifestCaches()` iterates every `#__extensions` row matching `JExtensionHelper::getCoreExtensions()` (the full stock-Joomla extension list) and calls `Installer::refreshManifestCache($extension->extension_id)` unconditionally. That method (`libraries/src/Installer/Installer.php:799-815`) immediately aborts with the exact message text `JLIB_INSTALLER_ABORT_REFRESH_MANIFEST_CACHE` (`administrator/language/en-GB/en-GB.lib_joomla.ini:538`) whenever the loaded extension's `state` column equals `-1` — confirmed as the *only* place this exact string appears anywhere in the codebase. `Installer::abort()` logs via `JLog::add($msg, JLog::WARNING, 'jerror')`, which is what surfaces as an on-screen admin Warning box.
- **Fix:** added a guard in the `updateManifestCaches()` loop — `if ((int) $extension->state === -1) { continue; }` — before calling `refreshManifestCache()`, skipping extensions already flagged this way instead of re-triggering (and letting `Installer::abort()` re-log) the same known, intentional, non-actionable condition on every future update. Does not touch the shared `Installer::refreshManifestCache()` method itself, since that's core JInstaller and other legitimate callers (`com_installer`'s manual "Manage → Refresh cache" action, the initial-install flow) should still see real failures when the admin explicitly requests a refresh.
- **Not fully root-caused:** did not conclusively determine *how* an already-installed core extension's row ends up with `state = -1` after its files are manually removed (traced the `-1` value's normal usage to `com_installer`'s Discover-scan placeholder rows, `models/discover.php`, which doesn't obviously match "previously-installed, now file-deleted" — but the fix is correct and sufficient regardless of the exact mechanism, since it eliminates the *exact* confirmed trigger condition for this *exact* confirmed message). If revisiting: worth checking whether `deleteUnexistingFiles()`/upgrade routines ever leave rows in this state, or whether it's purely from an admin having used Extensions → Discover at some point.

### F-2 — beez3/hathor rows never actually removed on upgrade (GitHub issue #7) — RESOLVED, fix shipped
- **Reported symptom:** [github.com/joomlaworks/joomla-3.x/issues/7](https://github.com/joomlaworks/joomla-3.x/issues/7) — after upgrading an existing site through this distribution (confirmed on 3.13, PHP 8.5.7), the removed `beez3` template is still listed in Extensions → Templates → Templates, and clicking it throws a PHP error, `0 uksort(): Argument #1 ($array) must be of type array, false given`. Same for `hathor` in the backend template area.
- **Root cause #1 — the 3.12 migration SQL never actually ran, on any site, ever:** `fixSchemas()` runs our migration files through `Joomla\CMS\Schema\ChangeSet`/`ChangeItem` (`libraries/src/Schema/ChangeItem/MysqlChangeItem.php::buildCheckQuery()`), which is Joomla core's mechanism for detecting whether a **DDL** statement (`ALTER TABLE ...`, `CREATE TABLE ...`) has already been applied — it recognizes nothing else. Any other statement type is left with `checkStatus = -1` ("skipped"), and `ChangeItem::fix()` only ever executes a query when `checkStatus === -2` ("failed"). `administrator/components/com_admin/sql/updates/mysql/3.12.0-2026-05-21.sql` (the beez3/hathor removal file) is **entirely `UPDATE`/`DELETE` DML — zero DDL** — so every statement in it is silently skipped and never runs via `fixSchemas()`, on any site, on any version, since the file was introduced in 3.12. Meanwhile `fixSchemas()` still bumps `#__schemas` to reflect the file as "applied" regardless, since it only checks `$schema !== $current` after calling `$changeSet->fix()` — it doesn't check whether any individual statement actually succeeded. All ~130 pre-existing stock Joomla migration files in that folder are genuine DDL and are correctly handled by this same mechanism — this is specific to the one DML file this distribution added.
- **Root cause #2 — this then actively corrupts the leftover rows:** because the `#__extensions`/`#__template_styles` rows for beez3/hathor are never deleted, `updateManifestCaches()` (which runs on every subsequent update, unconditionally, for every `#__extensions` row matching the stock core extension list) keeps trying to refresh their manifest cache. `TemplateAdapter::refreshManifestCache()` (`libraries/src/Installer/Adapter/TemplateAdapter.php:617-639`) looks for `templateDetails.xml` on disk — which `deleteUnexistingFiles()` has already removed — gets `false` back from `Installer::parseXMLInstallFile()`, and then unconditionally does `$this->parent->extension->manifest_cache = json_encode($manifest_details);` (i.e. `json_encode(false)`, storing the literal string `"false"`) and `$this->parent->extension->name = $manifest_details['name'];` (array access on `false`, silently `NULL` on PHP 8) before calling `store()`. This is what corrupts the row. When Template Manager's list view later `json_decode()`s that `"false"` string back to boolean `false` and passes it somewhere expecting an array (e.g. a param `uksort()`), you get exactly the reported crash.
- **Fix:** added `runDataMigrations()` to `administrator/components/com_admin/script.php`, called from `update()` **before** `updateManifestCaches()` (order: `deleteUnexistingFiles()` → `runDataMigrations()` → `updateManifestCaches()` → ...). It directly reads and executes an explicit list of this distribution's own DML-only migration files (currently just `3.12.0-2026-05-21.sql`) via `JDatabaseDriver::splitSql()` + `$db->execute()`, bypassing `JSchemaChangeset` entirely for these files. Deliberately does **not** track "already applied" state — every statement in these files is written to be naturally idempotent (`DELETE`s on rows that may already be gone; `UPDATE ... WHERE EXISTS (...)` guards that become false once already applied), so it's safe and cheap to re-attempt on every single update. This is also why it works as a **retroactive fix**: sites already stuck with orphaned/corrupted beez3/hathor rows (like the reporter's) self-heal automatically on their next update to this distribution — no manual DB cleanup or special one-time migration needed.
- **Convention going forward:** any future migration SQL file that is pure data-cleanup DML (not schema DDL) must be added to the `$dataMigrationFiles` array in `runDataMigrations()`, or it will suffer the exact same silent-no-op fate `fixSchemas()`/`JSchemaChangeset` gives it. Pure DDL files (`ALTER TABLE`/`CREATE TABLE`) should continue to rely on `fixSchemas()` alone — don't add those to this list too, since `JSchemaChangeset` already handles them correctly and re-running a non-idempotent DDL statement like `ADD COLUMN` a second time will error.
- **Possibly related:** this may also explain part of F-1's unresolved "how does state end up -1" question — worth revisiting together if similar reports surface.
- **Status: fix shipped and confirmed necessary. One side-mystery investigated thoroughly but deliberately left unresolved — read before reopening.** The core finding (`JSchemaChangeset` structurally cannot run `UPDATE`/`DELETE` statements) is not a theory — it was empirically confirmed by loading the real `MysqlChangeItem`/`ChangeItem` classes from this codebase and running them directly against the real, unmodified contents of `3.12.0-2026-05-21.sql`: all 9 statements come back `checkStatus = -1` ("skipped"), `checkQuery = NULL`. `ChangeItem::fix()` only ever executes a statement when `checkStatus === -2`, so none of them can run via `fixSchemas()`. This is settled.
  - What's *not* settled: the maintainer separately ran the confirming DB query (`SELECT ... FROM #__extensions/#__template_styles WHERE element IN ('beez3','hathor')`) against a real site currently on v3.14, and it returned **zero rows in both tables** — full, genuine deletion — even though that site never had my fix applied (it was queried in its pre-fix state). That directly contradicts "this can never run automatically."
  - Ruled out as explanations: the site's initial jump to v3.12 was a manual `tar.gz` extraction (update-feed infrastructure didn't exist pre-3.12, so the Joomla Update UI wasn't an option for that specific transition — confirmed by the maintainer), meaning `update()`/`postflight()` never ran for that step, and the physical files/DB rows would have been untouched at that point. Every version bump *after* reaching 3.12 (3.12→3.13→3.14) was confirmed done purely via "Update Now" in the Joomla Update component, no manual file or database intervention.
  - Checked the full git history of `com_admin/script.php` and the migration SQL file: `fixSchemas()` (commit `72cb824`) was a **pure addition** to the `update()` call chain — nothing was removed or replaced when it was introduced. So no earlier version of this project ever had a *different*, working, automated mechanism for running this file either. The gap has existed since the migration file itself was introduced in v3.12.
  - Also checked and ruled out: `FileAdapter` (the installer adapter for the "Joomla" `type="file"` package itself) has no generic sub-extension pruning logic; `ExtensionHelper::getCoreExtensions()` still lists both `beez3` and `hathor`, so `updateManifestCaches()` would still attempt to touch their rows if present; `com_templates`'s list/style models have no auto-delete-on-missing-files logic (`TemplatesHelper::parseXMLTemplateFile()` always returns a valid, if empty, `JObject` — a row with missing files still renders in the list, just with blank metadata, which if anything argues an orphaned row *should* be visible, not hidden); `administrator/manifests/files/joomla.xml`'s `<scriptfile>` does correctly point at `com_admin/script.php` (verified, not assumed); no other reference to `beez3`/`hathor` anywhere in the codebase does anything beyond file-path cleanup in `deleteUnexistingFiles()` or the stock Joomla runtime fallback in `SiteApplication::getTemplate()` (which only matters if `beez3` is the *active* template, and is a rendering fallback, not a DB operation).
  - **Conclusion:** the maintainer chose to close this out and ship the fix rather than continue investigating. The `JSchemaChangeset`-can't-run-DML gap is proven and the fix for it is correct and necessary regardless (the GitHub issue #7 report proves the gap is real for at least some upgrade paths). How the maintainer's own specific site ended up clean despite predating the fix remains genuinely unexplained. If this resurfaces (e.g. another report of orphaned rows, or new information about what happened on that site), the two open threads worth checking are: (a) something in the `com_joomlaupdate` component's own model/controller flow, prior to it handing off to `JInstaller`, that hasn't been traced end-to-end; (b) an undocumented manual action on that specific site (Extensions Manager's Warnings/Discover screens, etc.) that the site owner didn't connect to this at the time.
  - The fix (`runDataMigrations()`) is safe and beneficial regardless of how this resolves — it's a no-op if the rows are already gone, and closes the confirmed code-level gap if they're not. Do not revert it while this is unresolved.

### F-3 — `lcg_value()` deprecated in PHP 8.4 (GitHub issue #12)
- **Reported symptom:** [github.com/joomlaworks/joomla-3.x/issues/12](https://github.com/joomlaworks/joomla-3.x/issues/12) — `Deprecated: Function lcg_value() is deprecated since 8.4, use \Random\Randomizer::getFloat() instead` in `plugins/system/sessiongc/sessiongc.php` on line 56.
- **File:** `plugins/system/sessiongc/sessiongc.php`, `onAfterRespond()` — two occurrences (only one, line 56, was named in the report; the second, line 69, is byte-for-byte the same pattern for the session-metadata GC branch and would have hit the same deprecation).
- **Issue:** `$random = $divisor * lcg_value(); if ($probability > 0 && $random < $probability) { ... }` — a manual reimplementation of the classic `gc_probability`/`gc_divisor` percentage-chance trigger, using the now-deprecated `lcg_value()` (returns a float in `[0, 1)`) to build a `[0, $divisor)` random value.
- **Why the suggested replacement doesn't apply here:** the deprecation notice's own suggested fix, `\Random\Randomizer::getFloat()`, is PHP 8.2+ only — not usable, since this distribution's floor is PHP 7.4.
- **Fix:** replaced the float-based comparison with `random_int(1, $divisor) <= $probability` — an integer-uniform equivalent of the same `probability/divisor` chance (textbook-equivalent to how PHP's own session extension implements gc_probability/gc_divisor internally), using `random_int()` (available since PHP 7.0, not deprecated on any supported version, cryptographically sourced so no `srand()`-seeding concerns either). Added an explicit `$divisor > 0` guard before calling `random_int()`, since `random_int(1, 0)` throws a `ValueError` — the original `lcg_value()`-based code never crashed on a misconfigured `gc_divisor <= 0` (it just always/never triggered depending on sign), so the guard preserves that same no-crash behavior for a degenerate config rather than introducing a new fatal error.
- **Verified:** `php -l` clean; ran 200,000 trials with default settings (`probability=1`, `divisor=100`) — observed trigger rate 0.973%, matching the expected ~1% within statistical noise; confirmed `divisor=0` short-circuits via `&&` without calling `random_int()` (no fatal). Confirmed via repo-wide grep that `lcg_value()` doesn't appear anywhere else in the codebase — this was the only file affected.

### F-4 — Literal `_QQ_` text leaking into rendered UI, repo-wide (reported directly by the maintainer, no GitHub issue link)
- **Reported symptom:** the "Add Install from Web tab" alert on Extensions: Install shows literal `"_QQ_"` text instead of quote marks (e.g. `By selecting "_QQ_"Add Install from Web tab"_QQ_" below, ...`) when the `plg_installer_webinstaller` plugin is missing/disabled; also reported leaking into a link shown against extensions with an available update (exact string not pinned down — see below).
- **Root cause:** `_QQ_` is a legacy Joomla language-file placeholder for an escaped double quote, dating back to a `parse_ini_file()` bug in PHP 5.2 (long since irrelevant to this project's PHP 7.4+ floor — maintainer traced the history to [docs.joomla.org's archived language-pack guide](https://docs.joomla.org/Archived:Making_a_Language_Pack_for_Joomla)). Officially deprecated since Joomla 3.9.0 in favour of a real escaped quote (`\"`), but the substitution back to `"` was never fully wired into the primary runtime path: `LanguageHelper::parseIniFile()` (`libraries/src/Language/LanguageHelper.php:425-470`), which is what `Language::parse()` actually calls for every `JText::_()` lookup, only performs the `_QQ_` → `"` substitution in its **fallback** branch — used only when `parse_ini_file()` itself is unavailable, which is essentially never in practice. On any normal PHP setup, `_QQ_` tokens in a language string are loaded and rendered completely untouched.
- **Why this went unnoticed for so long:** `_QQ_` doesn't fully break the *page* — PHP's INI parser is lenient enough to keep parsing a value across embedded, unescaped `"` characters as long as they're not followed by whitespace/EOL, so a malformed line doesn't throw a parse error, it just silently produces a value containing stray literal quotes and the literal text `_QQ_`, which are otherwise inert as plain HTML text (not treated as markup) — cosmetically wrong, but not fatal, so it's easy to overlook amid competing priorities.
- **Scope, discovered via repo-wide grep (not just the 2 reported strings):** 80 `.ini` files, 1436 total occurrences, spanning both `en-GB` core language files (29 files) and third-party-translation-style `installation/language/*/*.ini` packs for other languages (51 files).
- **Pattern discovered while fixing (important for any future occurrence):** an initial naive `_QQ_` → `\"` substitution was tried first and was **wrong** — empirically verified (via `parse_ini_file()` against the real, unmodified original text) that 1434 of the 1436 occurrences are actually the 3-part group `"_QQ_"` (a real literal quote, then `_QQ_`, then another real literal quote) — i.e. someone had already attempted a partial migration away from `_QQ_` at some point and left stray literal quotes flanking each token instead of replacing them. A naive `_QQ_`-only substitution would have left those stray quotes in place, producing broken output like `class="\""alert-link"\""`. The correct fix collapses the whole 11-character `"_QQ_"` group into a single `\"` (2 characters). Only one line, in `installation/language/af-ZA/af-ZA.ini`, used the "correct" bare `_QQ_` (no flanking quotes) form.
- **Fix:** two-pass `sed` across all 80 files — first `"_QQ_"` → `\"` (handles 1434 of 1436), then a fallback bare `_QQ_` → `\"` pass to catch the one remaining outlier — followed by full validation: every one of the 477 `.ini` files in the repository (not just the 80 touched) parses cleanly via a real `parse_ini_file()` call, zero `_QQ_` remain anywhere, and the `COM_INSTALLER_INSTALL_FROM_WEB_INFO`/`_TOS` strings (the confirmed Extensions: Install match) were spot-checked end-to-end (parsed + the `\"`→`"` unescape `LanguageHelper::parseIniFile()` performs) to confirm they render as clean HTML with proper quote marks.
- **Confirmed post-fix:** the maintainer supplied a real-world example from an actual v3.14 site (pre-fix) that pins down the exact key behind the second reported symptom — `COM_INSTALLER_MSG_UPDATE_SITES_COUNT_CHECK` (the "Some update sites are disabled. You may want to check the Update Sites Manager." warning on Extensions → Update, triggered from `administrator/components/com_installer/controllers/update.php:98`). The broken link rendered as `https://sitehost/administrator/_QQ_%22/administrator/index.php?option=com_installer&view=updatesites%22_QQ_` — i.e. the literal `_QQ_"..."_QQ_` text from the unconverted language string was treated by the browser as one continuous relative URL (not parsed as separate HTML attributes), with the embedded literal `"` characters percent-encoded to `%22` when the browser resolved it against the site's base URL. Already fixed by the same 80-file sweep; confirmed the corrected value in the repo now reads `<a href=\"%s\">Update Sites Manager</a>`.

### F-5 — Hardcoded `JOOMLA_MINIMUM_PHP` gate raised from stale `5.3.10` to the codebase's real syntactic floor, `7.1.0`
- **Context:** grew directly out of a maintainer question about what happens if a pre-3.12 site (PHP <7.4) tries to update, and whether any code in this distribution would actually break on very old PHP. Investigated both the update-server-side gate (`docs/extension.xml`'s `<php_minimum>7.4.0</php_minimum>`, enforced by `libraries/src/Updater/Adapter/ExtensionAdapter.php:137-156` — a site below 7.4.0 simply never gets the update offered, with a warning message explaining why) and, separately, whether the *code itself* would even parse on older PHP if manually copied onto such a site (bypassing the update-server gate entirely).
- **Finding:** searched the entire codebase (including `vendor/`) for genuinely PHP 7.4-exclusive syntax — arrow functions (`fn() =>`), null coalescing assignment (`??=`), typed properties, numeric literal separators. Found **zero** instances of any of them. But this project's own PHP 8.1-deprecation fixes (P-8, P-10, Q-2 in the 3.11/3.12 patch log — "118 files changed" for P-10 alone) introduced nullable type declarations (`?Type $param = null`) throughout the codebase — **273 occurrences repo-wide** at last count. That syntax requires **PHP 7.1 minimum**; it's a hard parse error, not a warning, on PHP 5.6 or 7.0. Since these are scattered through core, always-loaded files (Application bootstrap, Filter, Form classes), a sub-7.1 site with the code manually copied on wouldn't partially work — it would white-screen immediately with a raw, unhelpful parse error, never reaching Joomla's own version-check logic.
- **The bug:** `JOOMLA_MINIMUM_PHP` — the hardcoded gate meant to catch exactly this scenario and show a friendly "Your host needs to use PHP X or higher" message — was still set to the original stock-Joomla value `5.3.10` in all three entry points (`index.php`, `administrator/index.php`, `installation/index.php`). Since the true floor is 7.1.0, this gate was completely ineffective for PHP 5.6/7.0 hosts: `version_compare(PHP_VERSION, '5.3.10', '<')` is false for those versions, so the check passes, execution continues, and the site hits the nullable-type parse error instead of the friendly message the gate exists to show. Confirmed the check itself (in all three files) is the very first executable statement, before any nullable-type-using file is loaded, so raising the constant is sufficient to make the friendly-error path actually trigger for these hosts.
- **Fix:** `JOOMLA_MINIMUM_PHP` raised from `'5.3.10'` to `'7.1.0'` in all three definition sites. `administrator/components/com_joomlaupdate/models/default.php` and `installation/model/setup.php` both only *read* the constant (used as a fallback/display value), so they pick up the new value automatically — not edited directly.
- **Deliberately NOT changed:** `docs/extension.xml`'s `<php_minimum>7.4.0</php_minimum>` — that's a separate, higher bar representing what this distribution is actually *tested and supported* on, not the absolute floor below which the code can't even parse. The two numbers serve different purposes and should stay different: 7.1.0 = "won't fatal-crash," 7.4.0 = "actually supported." Also left alone: `libraries/fof/utils/installscript/installscript.php`'s `$minimumPHPVersion = '5.3.3'` (a FOF framework *default* property meant to be overridden by third-party extensions using FOF for their own install scripts — nothing to do with Joomla core's own minimum) and `libraries/src/Http/Transport/StreamTransport.php`'s `version_compare(PHP_VERSION, '5.6.0')` (runtime feature-detection for an SSL stream option, not a version gate — harmless no-op now that the floor is above 5.6 anyway).
- **Verified:** `php -l` clean on all three edited files; ran `version_compare()` against a spread of versions (5.6.40, 7.0.33, 7.1.0, 7.1.33, 7.4.0, 8.5.0) confirming exactly 5.6.x/7.0.x are blocked and everything 7.1.0+ proceeds.
- **UPDATE (same session, immediately after):** the "deliberately not changed" call above was reversed on maintainer instruction. `docs/extension.xml`'s `<php_minimum>` was lowered from `7.4.0` to `7.1.0` to match — see the new entry immediately below (I-5) for the reasoning. The distinction drawn above (7.1.0 = won't fatal-crash vs. 7.4.0 = actually supported) was the *reasoning* for leaving it alone, not a hard rule; the maintainer's call was that since the code is confirmed to run on 7.1+, sites in the 7.1–7.3 range shouldn't be denied security patches just because they haven't hit the recommended baseline.

### F-6 — Module "Use Global" caching silently ignored Global Configuration's Cache Time, repo-wide (reported directly by the maintainer, no GitHub issue link)

- **Reported symptom:** mod_menu instances kept refreshing every 900 seconds (15 minutes) regardless of what Global Configuration's "Cache Time" was set to (e.g. 180 seconds/3 minutes), even though the module's own Caching field was set to "Use Global". Maintainer explicitly flagged this might be a broader bug, not mod_menu-specific.
- **Root cause, confirmed via `ModuleHelper::moduleCache()` (`libraries/src/Helper/ModuleHelper.php`, formerly line 563):**
  ```php
  $cache->setLifeTime($moduleparams->get('cache_time', $conf->get('cachetime') * 60) / 60);
  ```
  This reads the module's *own* `cache_time` param (stored in seconds), falling back to Global Configuration's `cachetime` (stored in minutes, ×60 to match units) **only if `$moduleparams->get('cache_time', ...)` treats the stored value as unset**. `$moduleparams` is a `Registry` instance, and `Registry::get()` (`libraries/vendor/joomla/registry/src/Registry.php:218`) has a specific quirk here: `return (isset($this->data->$path) && $this->data->$path !== '') ? $this->data->$path : $default;` — it treats a genuinely **absent key** *and* an explicitly-stored **empty string** (`''`) identically, both triggering the fallback; but a stored `0`/`'0'` is neither absent nor `''`, so it does **not** fall back — it's returned as a literal zero-second lifetime. Verified empirically with the real `Registry` class (not the XML's own field-render default, which is a separate, unrelated fallback path in `JForm::loadField()`/`JFormField::setup()` that only ever affects what's *displayed* when reopening the edit screen — not what `ModuleHelper` reads at render time).
  In practice, the fallback essentially never fires: every core module's `cache_time` field ships with `default="900"` in its XML (25 of 26 core modules checked; the one exception, `mod_breadcrumbs`, defaults to `0`, matching its `cache="0"` no-caching setup). HTML forms submit *all* rendered fields on save, not just touched ones, so the first time a module is saved through the Module Manager, `cache_time=900` gets written into its params — this is also baked directly into `installation/sql/mysql/joomla.sql`'s sample `#__modules` rows (mod_menu's "Main Menu" instance ships with `"cache":"1","cache_time":"900"` from install, never actually unset). Since the fallback only fires on a genuinely *absent* key or an explicit empty string, it's effectively dead code for any module that's ever been saved via the UI or shipped as a default sample — clearing the field via the UI and saving is the *one* way to actually reach the fallback, since `JForm::filterField()`'s default case (`libraries/src/Form/Form.php`, the `filter="integer"`/non-required branch) stores a blank field as literal `''`, not `0` and not the XML default. Typing an explicit `0` into the field, however, does *not* reach the fallback — it stores a literal `0`, which the pre-fix code would have divided into a `0`-minute lifetime.
  Deeper issue: every core module's `cache` field (`administrator's per-module XML, e.g. `modules/mod_menu/mod_menu.xml`) is a strict binary choice — `1` = "Use Global", `0` = "No caching" — there is no third "own value" mode anywhere in this codebase (confirmed via repo-wide grep: `ModuleHelper.php` is the only runtime consumer of `cache_time` for lifetime purposes, and the only `cache` param comparisons anywhere check for `0`/`'0'`). So the `cache_time` field was never meaningfully "the module's own override" for any stock module — it was purely UI filler that happened to permanently defeat "Use Global" the moment a module was saved.
- **Scope:** confirmed via grep across `modules/` and `administrator/modules/` — **26 core modules** define a `cache_time` field (frontend: mod_menu, mod_custom, mod_banners, mod_search, mod_feed, mod_stats, mod_wrapper, mod_breadcrumbs, mod_articles_archive/category/categories/latest/news/popular, mod_related_items, mod_users_latest, mod_whosonline, mod_tags_popular/similar; admin: mod_custom, mod_feed, mod_latestactions, mod_stats_admin, mod_privacy_dashboard, mod_quickicon). This is not a mod_menu-specific bug — it's a systemic issue affecting "Use Global" caching for essentially every cacheable core module, confirming the maintainer's suspicion.
- **Fix:** changed the central chokepoint in `ModuleHelper::moduleCache()` so that whenever the module's `cache` param isn't explicitly disabled (i.e. "Use Global" or unset), it honors Global Configuration's `cachetime` directly, bypassing the module's own (almost-always-baked-to-900) `cache_time` value entirely. The old fallback logic is preserved for the `cacheDisabled` branch (where the lifetime value is moot anyway since caching itself is off) to avoid touching behavior outside the reported bug's scope:
  ```php
  if ($cacheDisabled)
  {
      $cache->setLifeTime($moduleparams->get('cache_time', $conf->get('cachetime') * 60) / 60);
  }
  else
  {
      $cache->setLifeTime($conf->get('cachetime'));
  }
  ```
  Fixed at this single chokepoint rather than touching 26 XML files or existing sites' stored params — this way sites with already-saved modules (stale baked-in `cache_time:900`) are fixed immediately on upgrade with zero data migration needed, and any current/future core or third-party module using the standard binary `cache` field benefits automatically.
- **Verified empirically:** `php -l` clean; wrote and ran a standalone simulation of the fixed branch logic (`cache=1`, baked `cache_time=900`, Global Config=3 minutes → correctly yields `setLifeTime(3)`; `cache=0` path unchanged, still yields the old `15`).
- **Caveat:** this is an intentional behavior change for any site that manually hacked a module's stored params to set a nonstandard `cache_time` while leaving `cache=1` — that override is now ignored in favor of Global Configuration, matching the "Use Global" label's actual meaning. No stock core module or code path in this repo ever exercised that combination on purpose (see root-cause note above), so this is considered a correctness fix, not a breaking change. The `cache_time` XML fields themselves were deliberately left in place (unused now for `cache=1`, still meaningful for a hypothetical third-party module implementing its own genuine "own value" cache mode) to minimize blast radius.

### F-7 — Added "sort by Author" to Extensions: Manage (requested directly by the maintainer, no GitHub issue link)

- **Request:** the maintainer wanted to sort the Extensions: Manage list by the "Author" column (already displayed, just not sortable) to make it easy to spot third-party extensions vs. core Joomla ones on a large site — e.g. to review for stale/abandoned third-party extensions.
- **Why "Author" isn't a normal sortable column:** `#__extensions` has no `author` column at all. What's displayed comes from `InstallerModel::translate()` (`administrator/components/com_installer/models/extension.php`), which `json_decode()`s each row's `manifest_cache` blob (the extension's manifest metadata, cached at install time) and copies keys like `author`/`authorEmail`/`authorUrl`/`version`/`creationDate` onto the item — **after** the SQL query has already run. `InstallerModelManage::getListQuery()` uses `select('*')` with no `author` column to select in the first place.
- **How ordering already handled this for `name` (the template for this fix):** `InstallerModel::_getList()` has a `$customOrderFields` array (`name`, `client_translated`, `type_translated`, `folder_translated`) that, when the requested ordering matches one of them, skips the normal SQL `ORDER BY` + `LIMIT` path entirely: it fetches the **full** filtered result set (no `LIMIT`), calls `translate()` to decode/populate every row, then sorts the in-memory array with `ArrayHelper::sortObjects()` (case-insensitive, locale-aware) before finally slicing out the requested page. This exists because `name` itself gets re-translated via `JText::_()` inside `translate()`, so a raw SQL sort on the untranslated column wouldn't match the displayed order — the same root problem `author` has (it doesn't exist in SQL at all until `translate()` runs).
- **Fix — extended the existing pattern to `author` rather than inventing a new one:**
  - `models/extension.php`: added `'author'` to `$customOrderFields`.
  - `models/manage.php`: added `'author'` to `InstallerModelManage`'s `filter_fields` whitelist — required, since `JModelList::populateState()` (`libraries/src/MVC/Model/ListModel.php:538/561/605`) silently rejects any `list.ordering`/`fullordering` value not present in this array and falls back to the default, regardless of what's requested via the URL/form.
  - `models/forms/filter_manage.xml`: added `author ASC`/`author DESC` options to the `fullordering` select field.
  - `views/manage/tmpl/default.php`: the Author `<th>` was static text (`JText::_('JAUTHOR')`); changed to a clickable `JHtml::_('searchtools.sort', 'JAUTHOR', 'author', ...)` link, matching every other sortable column header on this screen.
  - `language/en-GB/en-GB.com_installer.ini`: added `COM_INSTALLER_HEADING_AUTHOR_ASC`/`_DESC` (used by the sort-by dropdown's option labels; `JAUTHOR` itself already existed for the column header).
  - `models/extension.php`'s `translate()`: also added `$item->author = isset($item->author) ? (string) $item->author : '';` right after the manifest_cache decode. Not every extension's `manifest_cache` actually includes an `author` key (older/malformed manifests, or a blank `manifest_cache`) — before this normalization, `$item->author` could be entirely undefined on some rows, and `ArrayHelper::sortObjects()` (`libraries/vendor/joomla/utilities/src/ArrayHelper.php:636`) accesses the sort-key property directly (`$a->{$key[$i]}`), which throws a PHP warning on an undefined property. `client_translated`/`type_translated`/`folder_translated` were already unconditionally set for every row for the same reason — `author` was the odd one out because the template's own display code already defensively used `@$item->author`.
- **Verified:** `php -l` clean on all touched PHP files, `xmllint` clean on the XML, `parse_ini_file()` clean on the language file. Also ran a standalone simulation using the real `Joomla\Utilities\ArrayHelper::sortObjects()` against a small mixed-data set (including one item with `author = ''`, mimicking the missing-manifest-cache-key case) confirming case-insensitive ascending/descending sort works correctly with no warnings once the normalization is applied.
- **Scope note:** intentionally did not touch the sibling Update view's list (`views/update`) — it lists *available updates*, not installed extensions, and doesn't display an Author column at all; the maintainer's ask was specifically about Extensions: Manage.

### I-5 — Lowered the update-server `<php_minimum>` gate to match the real floor (7.1.0), so PHP 7.1–7.3 sites keep receiving updates
- **Context:** direct follow-up to F-5. Once F-5 established that this codebase's actual parse-safe floor is PHP 7.1.0 (not 7.4.0), the maintainer decided sites on PHP 7.1 through 7.3 shouldn't be locked out of receiving this distribution's security patches — `docs/extension.xml`'s `<php_minimum>7.4.0</php_minimum>` was the thing actually preventing that (see F-5's own writeup of `ExtensionAdapter.php:137-156` for the exact mechanism: a site below the declared `php_minimum` never gets the update set as "latest available," so "Update Now" simply doesn't offer it, regardless of what the local `JOOMLA_MINIMUM_PHP` gate says).
- **Fix:** `docs/extension.xml`'s `<php_minimum>` changed from `7.4.0` to `7.1.0`. Verified the file is still well-formed XML (`simplexml_load_file()`).
- **README updated to avoid contradictory messaging:** the "PHP COMPATIBILITY" section still recommends PHP 7.4+ as the baseline (unchanged — that recommendation stands), but now explicitly clarifies that PHP 7.1–7.3 sites will still be offered updates through Joomla Update, so a reader doesn't infer "must be on 7.4 to update" from the baseline recommendation.
- **Important caveat, not fully resolved:** F-5's PHP-7.4-exclusive-syntax search only proves the codebase *parses* on 7.1+; it is not a claim that everything has been functionally verified correct on PHP 7.1/7.2/7.3 specifically (no PHP 7.1–7.3 interpreter was available in-session to actually run anything, only `php -l`/static analysis on whatever PHP version this environment has). If a PHP-7.1-specific runtime bug ever surfaces (as opposed to a parse error), this is the first place to look — the "7.4 is recommended, 7.1 is the advertised floor" gap is exactly where an undiscovered version-specific runtime quirk (not caught by syntax analysis) would live.

### I-6 — Removed `README.txt`, renamed `LICENSE.txt` → `LICENSE.md` (requested directly by the maintainer)

- **Request:** drop the stock, unedited upstream `README.txt` (generic Joomla boilerplate pointing to joomla.org docs, fully superseded by this project's own `README.md`) and rename the root `LICENSE.txt` to `LICENSE.md`, checking first whether either filename is referenced anywhere in the CMS or install/update process.
- **Functional references found and fixed** (as opposed to the ~3,039 files carrying just the cosmetic `@license ... see LICENSE.txt` comment-header boilerplate — see below):
  - `administrator/manifests/files/joomla.xml` — the `files_joomla` pseudo-extension's `<fileset>` lists every file this "package" owns. Removed the `<file>README.txt</file>` entry outright and changed `<file>LICENSE.txt</file>` to `<file>LICENSE.md</file>`; also updated the manifest's own `<license>` description text to match.
  - **Caught before it mattered:** this `<fileset>` only controls what gets *tracked/added* on install — it does **not** delete files that existed under the old manifest but are absent from the new one. Without an explicit cleanup step, sites upgrading from a pre-3.15 release would be left with an orphaned `README.txt` and a stale, unused `LICENSE.txt` sitting next to the new `LICENSE.md` forever. Fixed by adding both old filenames to `com_admin/script.php`'s `deleteUnexistingFiles()` array (`administrator/components/com_admin/script.php`, new `// Joomla 3.15.0` section, mirroring the exact same pattern already used for 3.12's beez3/hathor language-file cleanup): `/README.txt` and `/LICENSE.txt`. Confirmed this array is genuinely wired up — `deleteUnexistingFiles()` runs first thing inside `update()` (`script.php:84`, ahead of even the F-2 `runDataMigrations()` call), and each entry is guarded with `JFile::exists(...) && !JFile::delete(...)`, so it's a safe no-op on any site where the file's already gone (fresh 3.15+ installs, or a second run of the same update).
  - **Self-caught mistake:** on the first pass, the new `deleteUnexistingFiles()` entry read `/LICENSE.md` instead of `/LICENSE.txt` — a bulk find/replace (see below) blindly rewrote it along with everything else. That would have been a real bug: it must reference the *old* filename to clean up (the thing that needs deleting is the leftover `.txt`), not the new one — pointing it at `/LICENSE.md` would have deleted the brand-new license file immediately after every future update. Caught by re-reading the diff before considering this done, not by any tooling.
- **`docs/`, `.github/`, `composer.json`/`package.json`, `installation/` install screens:** grepped all of these — no references to either filename outside the manifest.
- **The cosmetic comment-header question:** ~3,039 files across the repo carry the standard Joomla copyright header `@license GNU General Public License version 2 or later; see LICENSE.txt` (pure comment text, zero functional effect either way). Asked the maintainer whether to bulk-update these too rather than silently doing (or silently skipping) a 3,000-file mechanical sweep; maintainer chose to update them.
  - Bulk `sed 's/LICENSE\.txt/LICENSE.md/g'` across every file containing the string. This is broader than just the comment header — it also unavoidably matches *any* literal occurrence of `LICENSE.txt`, so it had to be checked for collateral damage afterward rather than assumed safe.
  - **Found and reverted 3 incorrect matches after the fact** (all confirmed by grepping for every non-`"see LICENSE.txt"`-shaped occurrence before running the sed, then diffing after): `administrator/manifests/libraries/fof.xml`'s `<file>LICENSE.txt</file>` (the bundled FOF framework's *own* license file, `libraries/fof/LICENSE.txt` — a genuinely different, still-`.txt`-named file, confirmed still present on disk); `com_admin/script.php`'s `/libraries/simplepie/LICENSE.txt` entry inside the Joomla 3.5–3.6 historical file-removal list (a historical cleanup entry — rewriting it to `.md` would make it silently never match anything, defeating the point of it being there); and the I-6 `deleteUnexistingFiles()` entry described above.
  - The sed also correctly caught two format variants that a naive `see LICENSE\.txt` regex would have missed: tab-separated instances (`administrator/manifests/packages/pkg_weblinks.xml`, `plugins/system/highlight/highlight.xml`, both use `see\tLICENSE.txt`) and three translated installation-language manifests embedding the filename mid-sentence in non-English text (`installation/language/{ga-IE,sk-SK,sl-SI}/*.xml`) — using a blind substring replace rather than a narrower `see LICENSE\.txt`-only pattern is what caught these without needing per-language handling.
- **Verified:** `git mv` used for the rename (preserves history) and `git rm` for the deletion; `php -l` clean on every modified `.php` file; `xmllint` clean on every modified `.xml` file; batched `parse_ini_file()` clean across all 476 modified `.ini` files (single PHP process, not one process per file — the naive per-file loop timed out against 3,000+ files).

### F-8 — Added an explicit "Use custom cache time" module caching mode (follow-up to F-6, requested directly by the maintainer)

- **Context:** F-6 (earlier this session) fixed "Use Global" caching to always honour Global Configuration's Cache Time, ignoring the module's own `cache_time` param entirely. That closed the reported bug, but left no *supported* way to give an individual module its own, different TTL — the only lever left was manually editing the stored `cache_time` value with no UI affordance explaining what it did, which the maintainer flagged as unclear. Chosen fix (over silently restoring the old implicit-override behaviour): add a real third caching mode.
- **Discovery:** `administrator/language/en-GB/en-GB.com_modules.ini` already ships the string `COM_MODULES_FIELD_VALUE_CUSTOM_CACHE_TIME="Use custom cache time"` — confirmed via repo-wide grep that it was **completely unreferenced** anywhere in any module XML. Almost certainly inherited from upstream Joomla's own language file (later Joomla versions do offer a 3-state cache field); 3.x never wired it up. This meant no new language string was needed — only wiring the existing one in.
- **Scope:** the same 26 core modules from F-6 define a `cache`/`cache_time` field pair. Of those, 25 use the identical 2-option pattern (`value="1"` Use Global / `value="0"` No caching) and got the new option; `mod_whosonline` was deliberately excluded — its `cache` field only ever offers a single, hardcoded "No caching" option (`default="0"`, no "Use Global" choice at all), so a "custom" mode has no meaningful place there and touching it was out of scope.
- **XML changes, 25 files** (`modules/*/​*.xml`, `administrator/modules/*/​*.xml`):
  - Inserted `<option value="2">COM_MODULES_FIELD_VALUE_CUSTOM_CACHE_TIME</option>` between the existing `value="1"` and `value="0"` options on the `cache` field.
  - Added `showon="cache:2"` to the `cache_time` field, so it's only visible in the module edit form when "Use custom cache time" is selected — resolves the maintainer's core complaint ("not really clear to the user") by making the override an explicit, self-documenting choice instead of an implicit side-effect of a number field nobody's told the purpose of. `showon` confirmed already supported by this codebase's `JFormField`/`FormHelper` (pre-existing usage in `modules/mod_search/mod_search.xml`), so no framework changes needed.
  - **Mistakes caught and fixed during this edit, not before:** the bulk multi-file scripting for the `showon` insertion went through three broken iterations before landing correctly, each caught by writing a proper verification pass rather than trusting the diff on one sample file:
    1. First attempt anchored the new `showon` line's indentation to the *closing* `/>`'s indentation (4 tabs) instead of the sibling attributes' indentation (5 tabs) — caught by inspecting the raw diff, not by any validator (XML is indentation-insensitive, so `xmllint` stayed silent about it).
    2. Second attempt (after reverting and redoing) used a regex anchored only on `filter="integer"\n\t+/>` with no `name="cache_time"` context and no `/g` flag — on any module where an *earlier* field in the document happened to also end that way (e.g. an `items`/count field), the `showon` attribute landed on the wrong field entirely. Affected 14 of the 25 files. Caught by writing an actual verification script that extracts just the `<field name="cache_time">...</field>` block per file and checks `showon` is *inside* it — the first version of that verification script itself had a bug (checked "does `showon` appear anywhere in the file" rather than "inside the extracted block"), which produced a false "all clear" the first time it ran. Re-derived the check properly (capture `$&` from the anchored match, grep only within that captured substring) before trusting the result.
    3. The corrective re-insertion for those 14 files reintroduced the original 4-tab-vs-5-tab indentation bug from mistake #1. Fixed with one more targeted pass.
  - Final state confirmed via a 5-point check across all 25 files (not spot-checked): `showon` present inside the `cache_time` field block specifically; exactly one `showon="cache:2"` per file; exactly one new `<option>` per file; indentation matching sibling attributes; `xmllint` valid.
- **PHP change:** `ModuleHelper::moduleCache()` (`libraries/src/Helper/ModuleHelper.php`) — the F-6 branch (`if ($cacheDisabled) { use own cache_time } else { use Global }`) extended to a three-way split. First draft branched on `$cacheDisabled || $cacheMode == 2`; the maintainer proposed a cleaner version — default to Global Configuration unconditionally, then override only for the explicit custom case — which was applied as the final form (brace style corrected to match this file's Allman convention):
  ```php
  $moduleCacheState = $moduleparams->get('cache', 1);

  $cache->setLifeTime($conf->get('cachetime'));

  if ($moduleCacheState == 2)
  {
      $cache->setLifeTime($moduleparams->get('cache_time', $conf->get('cachetime') * 60) / 60);
  }
  ```
  This also incidentally closes a gap in the first draft: any unrecognised/legacy `cache` value (not `0`, `1`, or `2` - e.g. a hypothetical third-party module using its own convention) now safely defaults to Global Configuration instead of being interpreted as "must be disabled or custom."
- **Verified empirically:** `php -l` clean; ran a standalone simulation of all three branches — Use Global ignores a populated `cache_time` and returns Global Config's value; custom mode honours a distinct `cache_time` value with correct seconds→minutes conversion; custom mode with no `cache_time` set falls back to Global Config; disabled-caching path unchanged from F-6's behaviour.
- **`COM_MODULES_FIELD_VALUE_CUSTOM_CACHE_TIME` string:** added directly by the maintainer to `en-GB.com_modules.ini` (not by this agent) ahead of this work.
- **Confirmed `setLifeTime()` is provably inert whenever caching is disabled, not just "probably fine":** maintainer asked whether the `cacheDisabled` branch needs special handling for `setLifeTime()`, given `setCaching(false)` already fired earlier in the method. Traced the actual call chain: `\JFactory::getCache($cacheparams->cachegroup, 'callback')` returns a `CallbackController` (`libraries/src/Cache/Controller/CallbackController.php`), whose `get()` delegates straight to the wrapped `Cache::get()`/`Cache::store()` (`libraries/src/Cache/Cache.php`), and **both** bail out immediately via `if (!$this->getCaching()) { return false; }` before ever consulting `_options['lifetime']`. So no, no special-casing is needed — confirmed by reading the real call path, not assumed.

- **CRITICAL FOLLOW-UP, found by tracing the unrelated hidden `cachemode` XML field (maintainer asked about its purpose) — mode `2` never actually reached `moduleCache()` at all:**
  - `cachemode` (hidden field present in 15 core modules' XML, values `itemid`/`static`) is unrelated to the TTL question — it selects the *cache key strategy* for `moduleCache()`'s callback-cache mechanism, per that method's own docblock (`itemid`/`static`/`safeuri`/`id`/`oldstatic`). Traced every consumer in the codebase:
    - The 5 modules that call `ModuleHelper::moduleCache()` directly from their own `.php` (`mod_tags_popular`, `mod_tags_similar`, `mod_articles_category`, `mod_articles_categories`, `mod_related_items`) all **hardcode** `$cacheparams->cachemode` in PHP (`'safeuri'`/`'id'`) and don't define the hidden XML field at all — for these, the field would be irrelevant even if present.
    - `libraries/src/Document/Renderer/Html/ModuleRenderer.php:84` — `$cachemode = $params->get('cachemode', 'oldstatic');` — this is the real consumer. `ModuleRenderer` is the universal per-module renderer every `<jdoc:include type="modules">` in every template goes through (`ModulesRenderer::render()` → `$this->_doc->loadRenderer('module')` → this class), and it's the **only** place whole-module rendered-HTML output caching is actually applied — it wraps `ModuleHelper::renderModule()` itself as the cached callback.
  - **The bug:** that wrapping only happens inside `if ($params->get('cache', 0) == 1 && ...)` — a strict equality check against exactly `1`. With the new 3-mode system, `cache == 2` (Use custom cache time) fails this check and falls straight through to the plain, uncached `return ModuleHelper::renderModule($module, $attribs);` at the bottom of `render()` — every single page load. `moduleCache()` (all of the above work, both the maintainer's version and mine) is **never invoked** for mode 2 through the real rendering path, because this gate filters it out first. Mode 1 was never affected (this check has always correctly matched `== 1`), so F-6 held up fine; F-8 added a second "should cache" value without updating the one place that gates whether caching is attempted at all.
  - **Fix:** `ModuleRenderer::render()` changed from `$params->get('cache', 0) == 1` to a strict-equality `$cacheEnabled` flag (`!($cacheState === 0 || $cacheState === '0')`), mirroring the exact idiom `moduleCache()` already uses for its own `$cacheDisabled`. Now any non-`0` value enables the whole-output caching wrapper, future-proofing against further modes too. The `0`-when-absent default (`$params->get('cache', 0)`) was deliberately left unchanged — out of scope for this fix, and altering it wasn't asked for.
  - **Verified empirically:** `php -l` clean; ran a standalone simulation of the gate across 8 cases (cache 0/'0'/1/2/'2', global caching on/off, absent-defaults-to-0, and the reserved `cachemode == 'safeuri'` opt-out used by the 5 self-caching modules) — all passed, confirming mode 2 now reaches `moduleCache()` while mode 0 and global-caching-off still correctly don't.
- **UPDATE (same session, immediately after) — `mod_whosonline` brought into the same caching scheme:** originally excluded above since it shipped with a `cache` field offering only a single, hardcoded "No caching" option (no "Use Global" choice at all) — a deliberate historical decision, presumably because Who's Online shows live session data. Maintainer reconsidered: the underlying query (`ModWhosonlineHelper::getOnlineCount()`/`getOnlineUserNames()` in `modules/mod_whosonline/helper.php`) is a plain `#__session` table query with no special no-cache code path of its own — caching for this module was only ever disabled via its XML config, not enforced anywhere in PHP — so there's no technical reason it can't participate in the same Use Global / Use custom / No caching scheme as every other module, and doing so gives a real performance win on sites with many concurrent users. Updated `modules/mod_whosonline/mod_whosonline.xml`: `cache` field's `default` changed from `0` to `1` (matching every other module's convention) and given the same 3 options; existing `cache_time` field (already present, previously inert) given `showon="cache:2"`. No PHP changes needed. No migration needed either: unlike the F-6/I-6 cases, this is a pure XML-default change — Joomla's form binding only substitutes the XML default for a genuinely *unset* param, so any site with an existing Who's Online instance (or the "Learn" sample-data package, which ships one with `"cache":"0"` explicitly baked into its `#__modules` row) keeps its current, explicit setting untouched; only newly-created instances pick up the new default. Verified `xmllint` clean.
