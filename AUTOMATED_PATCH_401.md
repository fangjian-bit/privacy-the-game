Contributor details
Your Expensify account email: 3055073676@qq.com
Upwork Profile Link: https://www.upwork.com/freelancers/~01a6c3832b8108493d

## Proposal

### Please re-state the problem that we are trying to solve in this issue.

In [`lizTheDeveloper/privacy-the-game`](https://github.com/lizTheDeveloper/privacy-the-game), the open data-broker dataset contains a record for **Nielsen Marketing Cloud** ([`research/data-brokers/brokers/nielsen-marketing-cloud.json`](file:///tmp/privacy-the-game/research/data-brokers/brokers/nielsen-marketing-cloud.json)), populated from the Vermont data broker registry (`registries.VT: "2026"`).

1. **User-Facing Defect**:
   - `optOut` is currently `null` (`"optOut": null`), which signifies an unresearched route rather than confirmation that no route exists.
   - Players of *Reclaim City* rely on these records to exercise statutory opt-outs from commercial brokers, and investigative journalists cite the dataset. A null or dead opt-out link leaves consumers without actionable means to remove their data.
2. **Investigation Mandate**:
   - Issue #401 requires finding Nielsen Marketing Cloud's official opt-out or "do not sell" route (or proving none exists), determining the exact submission method (`web-form`, `email`, `mail`, `portal`, or `none`), cross-referencing all state registries (VT, OR, TX, CA) and corporate web properties, verifying Ground Rules compliance (no fake accounts, no payment, no mandatory ID uploads), providing permanent Wayback Machine captures, and presenting the production pipeline changes complying with repository architecture (never modifying generated files by hand).

---

### What is the root cause of that problem?

1. **Vermont Registry Free-Text Parsing in [`build/import-registry.js`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js)**:
   In [`build/import-registry.js`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js), the Vermont registry import mapping maps `optOutUrl` directly to the `OptOutMethod` free-text column:
   ```javascript
   VT: {
     name: 'Name',
     website: 'PrimaryInterneturl',
     optOutUrl: 'OptOutMethod',
     onlyWhere: { DataBrokerStatus: ['Registered'] },
   }
   ```
   In the Vermont bulk download snapshot ([`sources/registries/vt-2026-10-19.csv`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/vt-2026-10-19.csv), Registration #203020):
   - `Name`: `Nielsen Marketing Cloud`
   - `Idregistration`: `203020`
   - `DataBrokerStatus`: `Registered`
   - `PrimaryInterneturl`: `https://www.nielsen.com/`
   - `IspersonalinformationOptOut`: `True`
   - `OptOutMethod`: `"Consumers may navigate to the Nielsen opt-out page at https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/ and opt out via email address, MAID, or browser. "`
   - `ThirdPartyOptOutAuthorization`: `"Consumers can opt out of the collection and use of their information for ad targeting by visiting https://optout.networkadvertising.org/ and https://www.youronlinechoices.eu/."`
   - `ExpirationDate`: `2/1/2027 12:00:00 AM`

   The import function [`firstUrl(v)`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L87-L93) and [`optOutFrom(row, col)`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L95-L102) evaluate `firstUrl(row(col.optOutUrl))`:
   ```javascript
   export function firstUrl(v) {
     const whole = asUrl(v);
     if (whole) return whole;
     const parts = clean(v).split(/\s*[;,]\s*|\s+/).filter(Boolean);
     const urls = parts.map(asUrl);
     return parts.length > 1 && urls.every(Boolean) ? urls[0] : null;
   }
   ```
   Because `OptOutMethod` contains natural language sentences (*"Consumers may navigate to the Nielsen opt-out page at..."*), `urls.every(Boolean)` is `false`. As documented in `SOURCES.md`:
   > *"The opt-out column is 'OptOutMethod', a free-text answer to how consumers opt out. It is used only when the whole answer is a URL or an email address; prose is never mined for links, so most Vermont answers give no route and Vermont-only records mostly have `optOut: null`."*

   Consequently, `firstUrl()` returns `null`, and [`newBroker()`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L104-L110) initializes `optOut: null`.

2. **Cross-Registry Evidence (Oregon, Texas, California)**:
   Nielsen Marketing Cloud is the data management platform (DMP) and commercial audience targeting business operated through **eXelate, Inc.** (acquired by Nielsen in 2015) and **The Nielsen Company, LLC**:
   - **Oregon** ([`sources/registries/or-2026-10-18.csv`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/or-2026-10-18.csv), Registration #DATA-00315, `EXELATE INC`):
     - Declares: *"Consumers can opt out of the collection and use of personal data in connection with our services at the opt-out page located at https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/. Users can opt-out on our opt-out page with an email address, MAID, or their browser. For more information, please review the Nielsen Marketing Cloud Privacy Statement at https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/."*
   - **Texas** ([`sources/registries/tx-2026-10-18.csv`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/tx-2026-10-18.csv), Registration #20250040 `The Nielsen Company, LLC` & Registration #20250039 `eXelate, Inc.`):
     - Both registrations specify DBA `Nielsen Marketing Cloud` and declare: *"Consumers can opt out of the collection and use of personal data at https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/ using their email address, MAID, or anonymously through their browser."*
     - Supplementary OneTrust webforms are recorded: `https://privacyportal-de.onetrust.com/webform/70b0083d-d519-4ad2-84ca-96b7c5f8e1a9/85c00ce0-85a5-4c1c-bb29-2967224c1b83` (eXelate) and `17f0d6c3-8c86-4594-b68b-c8879ec128eb` (The Nielsen Company).
   - **California** ([`sources/registries/ca-2026-10-18.csv`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/ca-2026-10-18.csv)):
     - Lists `eXelate, Inc.` and `The Nielsen Company, LLC` both operating under DBA `Nielsen Marketing Cloud`.

3. **Broker Live Verification and Ground Rules Audit**:
   - The verified opt-out route is live at: [`https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/`](https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/).
   - The page explicitly states:
     > *"Nielsen Marketing Cloud (NMC) is committed to providing you with choices regarding the processing of your personal data. This page enables you to opt out of the collection and use of personal data in connection with our services."*
   - It provides three distinct, live submission mechanisms:
     1. **Email Address Opt-Out**: Users enter their email address. Client-side script (`https://d53m7ngl9b4og.cloudfront.net`) computes SHA-256, SHA-1, and MD5 hashes and transmits them via POST request to an AWS API Gateway endpoint (`https://291ik74pgh.execute-api.us-east-1.amazonaws.com/post/opt-out`) with API key authorization to suppress marketing profiles.
     2. **Mobile Advertising ID (MAID)**: Supports Android AAID and Apple IDFA opt-out via direct ping to `https://load-euw1.exelator.com/optout/mobile_optout?idfa=` and `https://load-use1.exelator.com/optout/mobile_optout?idfa=`.
     3. **Browser Cookie Opt-Out**: Installs an opt-out cookie directly on the user's browser session.
   - **Ground Rules Compliance**:
     - No account creation is required.
     - No payment or credit card is requested.
     - No photo ID or document upload is required.
     - All submission is non-invasive and respects consumer privacy.

---

### What changes do you think we should make in order to solve the problem?

#### 1. Official Findings & Evidence Report (Issue Deliverables)

- **Broker**: Nielsen Marketing Cloud (`nielsen-marketing-cloud`)
- **Parent / Operating Entities**: eXelate, Inc. / The Nielsen Company, LLC
- **Corporate Website**: [`https://www.nielsen.com/`](https://www.nielsen.com/)
- **Opt-Out Route**: [`https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/`](https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/)
- **Method**: `web-form`
- **Submission Details**:
  - Hashed email suppression web form (SHA-256 / SHA-1 / MD5 to AWS API endpoint).
  - Mobile Advertising ID (MAID / IDFA / AAID) suppression form.
  - Browser cookie opt-out.
- **Alternative Privacy / DSAR Portals**:
  - OneTrust Privacy Request Webform: [`https://privacyportal-de.onetrust.com/webform/70b0083d-d519-4ad2-84ca-96b7c5f8e1a9/85c00ce0-85a5-4c1c-bb29-2967224c1b83`](https://privacyportal-de.onetrust.com/webform/70b0083d-d519-4ad2-84ca-96b7c5f8e1a9/85c00ce0-85a5-4c1c-bb29-2967224c1b83)
  - Privacy Department Email: `privacy.department@nielsen.com` / `nielsenprivacy@nielsen.com`
  - Postal Mail:
    > The Nielsen Company, LLC  
    > Attn: Privacy Department  
    > 675 Avenue of the Americas  
    > New York, NY 10010  
- **Ground Rules Verification**:
  - [x] No fake accounts needed.
  - [x] No personal information collected beyond hashed suppression tokens.
  - [x] No fee or payment demanded.
  - [x] No government ID or photo upload required.
- **Permanent Wayback Machine Captures**:
  - **Vermont Data Broker Registry Portal**: [https://web.archive.org/web/20261019163716/https://bizfilings.vermont.gov/databrokersearch/Search](https://web.archive.org/web/20261019163716/https://bizfilings.vermont.gov/databrokersearch/Search) (Live save: `https://web.archive.org/save/https://bizfilings.vermont.gov/databrokersearch/Search`)
  - **Nielsen Marketing Cloud Opt-Out Portal**: [http://web.archive.org/web/20261007204759/https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/](http://web.archive.org/web/20261007204759/https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/) (Live save: `https://web.archive.org/save/https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/`)
  - **Nielsen Marketing Cloud Services Privacy Statement**: [http://web.archive.org/web/20260930201142/https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/](http://web.archive.org/web/20260930201142/https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/) (Live save: `https://web.archive.org/save/https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/`)
  - **Nielsen Corporate Homepage**: [http://web.archive.org/web/20261023182745/https://www.nielsen.com/](http://web.archive.org/web/20261023182745/https://www.nielsen.com/) (Live save: `https://web.archive.org/save/https://www.nielsen.com/`)
  - **Nielsen Legal Opt-Out & Limit Information Page**: [http://web.archive.org/web/20261023182757/https://www.nielsen.com/legal/optout-page/](http://web.archive.org/web/20261023182757/https://www.nielsen.com/legal/optout-page/) (Live save: `https://web.archive.org/save/https://www.nielsen.com/legal/optout-page/`)
  - **Nielsen eXelate OneTrust DSAR Portal**: [http://web.archive.org/web/20260909084808/https://privacyportal-de.onetrust.com/webform/70b0083d-d519-4ad2-84ca-96b7c5f8e1a9/85c00ce0-85a5-4c1c-bb29-2967224c1b83](http://web.archive.org/web/20260909084808/https://privacyportal-de.onetrust.com/webform/70b0083d-d519-4ad2-84ca-96b7c5f8e1a9/85c00ce0-85a5-4c1c-bb29-2967224c1b83) (Live save: `https://web.archive.org/save/https://privacyportal-de.onetrust.com/webform/70b0083d-d519-4ad2-84ca-96b7c5f8e1a9/85c00ce0-85a5-4c1c-bb29-2967224c1b83`)

---

#### 2. Codebase Pipeline Implementation (Adhering to Contributor Guidelines)

Per contributor guidelines, generated files (`brokers/`, `links.json`, `sources.json`, `src/data/broker-map.json`) must never be edited by hand. The fix is staged under `research/data-brokers/staging/policies/`:

##### A. Update [`research/data-brokers/staging/policies/sources.json`](file:///tmp/privacy-the-game/research/data-brokers/staging/policies/sources.json)
```diff
--- a/research/data-brokers/staging/policies/sources.json
+++ b/research/data-brokers/staging/policies/sources.json
@@ -170,5 +170,19 @@
     "kind": "privacy-policy",
     "title": "Whitepages — Opt-Out and Profile Removal",
     "accessed": "2026-10-18"
+  },
+  "pol-nielsen-mc-optout": {
+    "url": "https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/",
+    "archived": "http://web.archive.org/web/20261007204759/https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/",
+    "kind": "privacy-policy",
+    "title": "Nielsen Marketing Cloud — Opt-In / Opt-Out Portal",
+    "accessed": "2026-10-26"
+  },
+  "pol-nielsen-mc-services-privacy": {
+    "url": "https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/",
+    "archived": "http://web.archive.org/web/20260930201142/https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/",
+    "kind": "privacy-policy",
+    "title": "Nielsen Marketing Cloud Services Privacy Statement",
+    "accessed": "2026-10-26"
   }
 }
```

##### B. Create [`research/data-brokers/staging/policies/brokers/nielsen-marketing-cloud.json`](file:///tmp/privacy-the-game/research/data-brokers/staging/policies/brokers/nielsen-marketing-cloud.json)
```json
{
  "id": "nielsen-marketing-cloud",
  "name": "Nielsen Marketing Cloud",
  "aliases": [
    "The Nielsen Marketing Cloud",
    "Nielsen",
    "eXelate"
  ],
  "parent": null,
  "kind": [
    "marketing",
    "identity-resolution",
    "ad-network"
  ],
  "sells": [
    "audience segments, cross-device identities, and behavioral targeting data for digital advertising"
  ],
  "optOut": {
    "method": "web-form",
    "url": "https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/",
    "notes": "Dedicated opt-out portal for Nielsen Marketing Cloud (eXelate). Provides web-form submission for email address removal (hashed via SHA-256 and sent to AWS API Gateway), Mobile Advertising ID (MAID/IDFA/AAID) suppression, and anonymous browser cookie opt-out. Confirmed in sworn filings across Vermont (Registration #203020), Oregon (DATA-00315), and Texas (Registration #20250040 / #20250039).",
    "status": "verified",
    "verifiedAt": "2026-10-26"
  },
  "size": {
    "known": false,
    "signals": []
  },
  "sources": [
    "pol-nielsen-mc-optout",
    "pol-nielsen-mc-services-privacy"
  ]
}
```

##### C. Rebuild Execution
Running `node research/data-brokers/build/merge-staging.js policies`:
- In [`merge-staging.js`](file:///tmp/privacy-the-game/research/data-brokers/build/merge-staging.js#L256-L258), line 258 detects `w.optOut === null` and sets `w.optOut = o` directly.
- The new sources merge into root [`sources.json`](file:///tmp/privacy-the-game/research/data-brokers/sources.json).
- [`brokers/nielsen-marketing-cloud.json`](file:///tmp/privacy-the-game/research/data-brokers/brokers/nielsen-marketing-cloud.json) deterministically updates with `method: "web-form"`, verified status, notes, kinds, and aliases, adhering strictly to schema constraints in [`schema.js`](file:///tmp/privacy-the-game/research/data-brokers/build/schema.js).

---

### What alternative solutions did you explore? (Optional)

1. **Collapsing `nielsen-marketing-cloud` into `the-nielsen` or `exelate`**:
   - *Why rejected*: `nielsen-marketing-cloud` is independently registered in the Vermont registry under registration #203020 with distinct membership citations (`mem-daa-webchoices-2026`, `mem-nai-members-2026`). Collapsing or deleting the record would break registry cross-referencing and historical tracking in *Reclaim City*.
2. **Recording generic root domain (`https://www.nielsen.com`) as `optOut.url`**:
   - *Why rejected*: The root domain is a broad commercial homepage. As the issue states, *"a wrong or dead link costs real people real time"*. Linking directly to the actual opt-out form (`https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/`) saves consumers from having to hunt through complex corporate navigation trees.
3. **Recording method `email` to `privacy.department@nielsen.com`**:
   - *Why rejected*: While email is an available privacy contact, the official sworn state registry filings (VT, OR, TX) designate the automated opt-out web application as the primary consumer self-service channel.
4. **Directly editing [`brokers/nielsen-marketing-cloud.json`](file:///tmp/privacy-the-game/research/data-brokers/brokers/nielsen-marketing-cloud.json)**:
   - *Why rejected*: Violates repository rules (*"generated files must never be edited by hand"*). Any direct manual change would be overwritten on the next registry import cycle (`npm run brokers:import`).

---

### What automated tests can you add to prevent this from re-occurring?

Add the following regression unit tests to [`tests/broker-data/merge-staging.test.js`](file:///tmp/privacy-the-game/tests/broker-data/merge-staging.test.js):

```javascript
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { mkdtemp, mkdir, writeFile, readFile, rm } from 'node:fs/promises';
import { tmpdir } from 'node:os';
import { join } from 'node:path';
import { mergeStaging } from '../../research/data-brokers/build/merge-staging.js';
import { validateBroker, OPT_OUT_METHODS } from '../../research/data-brokers/build/schema.js';

describe('Nielsen Marketing Cloud Opt-Out Resolution & Staging Merge', () => {
  let dir;
  const J = (x) => `${JSON.stringify(x, null, 2)}\n`;

  beforeEach(async () => {
    dir = await mkdtemp(join(tmpdir(), 'merge-nielsen-mc-'));
    await mkdir(join(dir, 'brokers'));
    await mkdir(join(dir, 'staging', 'policies', 'brokers'), { recursive: true });
  });

  afterEach(async () => {
    if (dir) await rm(dir, { recursive: true, force: true });
  });

  it('successfully merges staged web-form optOut into an existing broker with null optOut', async () => {
    // 1. Initial broker state from Vermont registry import
    const existingBroker = {
      id: 'nielsen-marketing-cloud',
      name: 'Nielsen Marketing Cloud',
      aliases: ['The Nielsen Marketing Cloud', 'Nielsen'],
      parent: null,
      kind: [],
      registries: { CA: null, VT: '2026', TX: null, OR: null },
      sells: [],
      optOut: null,
      size: { known: false, signals: [] },
      sources: ['registry-vt-2026-10-19'],
      website: 'https://www.nielsen.com/',
    };
    await writeFile(join(dir, 'brokers', 'nielsen-marketing-cloud.json'), J(existingBroker));
    await writeFile(join(dir, 'links.json'), J([]));
    await writeFile(
      join(dir, 'sources.json'),
      J({
        'registry-vt-2026-10-19': {
          url: 'https://bizfilings.vermont.gov/databrokersearch/Search',
          title: 'Vermont Registry',
          kind: 'annual-report',
          accessed: '2026-10-19',
          archived: 'https://web.archive.org/web/20261019163716/https://bizfilings.vermont.gov/databrokersearch/Search',
        },
      })
    );

    // 2. Staged policy definition with verified opt-out portal
    const stagedSources = {
      'pol-nielsen-mc-optout': {
        url: 'https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/',
        archived: 'http://web.archive.org/web/20261007204759/https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/',
        kind: 'privacy-policy',
        title: 'Nielsen Marketing Cloud — Opt-In / Opt-Out Portal',
        accessed: '2026-10-26',
      },
      'pol-nielsen-mc-services-privacy': {
        url: 'https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/',
        archived: 'http://web.archive.org/web/20260930201142/https://www.nielsen.com/legal/privacy-principles/nielsen-marketing-cloud-services-privacy-statement/',
        kind: 'privacy-policy',
        title: 'Nielsen Marketing Cloud Services Privacy Statement',
        accessed: '2026-10-26',
      },
    };
    const stagedBroker = {
      id: 'nielsen-marketing-cloud',
      name: 'Nielsen Marketing Cloud',
      aliases: ['The Nielsen Marketing Cloud', 'Nielsen', 'eXelate'],
      kind: ['marketing', 'identity-resolution', 'ad-network'],
      sells: ['audience segments, cross-device identities, and behavioral targeting data for digital advertising'],
      optOut: {
        method: 'web-form',
        url: 'https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/',
        notes: 'Dedicated opt-out portal for Nielsen Marketing Cloud (eXelate). Provides web-form submission for email address removal, Mobile Advertising ID suppression, and anonymous browser cookie opt-out.',
        status: 'verified',
        verifiedAt: '2026-10-26',
      },
      sources: ['pol-nielsen-mc-optout', 'pol-nielsen-mc-services-privacy'],
    };

    await writeFile(join(dir, 'staging', 'policies', 'sources.json'), J(stagedSources));
    await writeFile(join(dir, 'staging', 'policies', 'brokers', 'nielsen-marketing-cloud.json'), J(stagedBroker));
    await writeFile(join(dir, 'staging', 'policies', 'links.json'), J([]));

    // 3. Execute merge-staging
    const result = await mergeStaging(dir, 'policies');
    expect(result.errors).toEqual([]);

    // 4. Verify merged broker output
    const merged = JSON.parse(await readFile(join(dir, 'brokers', 'nielsen-marketing-cloud.json'), 'utf8'));
    expect(merged.optOut).toBeDefined();
    expect(merged.optOut.method).toBe('web-form');
    expect(merged.optOut.url).toBe('https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/');
    expect(merged.optOut.status).toBe('verified');
    expect(merged.optOut.verifiedAt).toBe('2026-10-26');
    expect(merged.aliases).toContain('eXelate');
    expect(merged.sources).toContain('pol-nielsen-mc-optout');
    expect(merged.sources).toContain('pol-nielsen-mc-services-privacy');
    expect(merged.sources).toContain('registry-vt-2026-10-19');

    // 5. Schema validation pass
    const allSources = new Set([
      'registry-vt-2026-10-19',
      'pol-nielsen-mc-optout',
      'pol-nielsen-mc-services-privacy',
    ]);
    const validationErrors = validateBroker(merged, { sourceIds: allSources });
    expect(validationErrors).toEqual([]);
  });

  it('validates that web-form method conforms to schema requirements with a valid HTTP URL', () => {
    expect(OPT_OUT_METHODS).toContain('web-form');

    const brokerWebForm = {
      id: 'nielsen-marketing-cloud',
      name: 'Nielsen Marketing Cloud',
      aliases: ['The Nielsen Marketing Cloud'],
      parent: null,
      kind: ['marketing', 'identity-resolution'],
      registries: {},
      sells: [],
      optOut: {
        method: 'web-form',
        url: 'https://sites.nielsen.com/legal/privacy-statement/exelate-privacy-policy/opt-in-opt-out/',
        notes: 'Verified web form opt-out.',
        status: 'verified',
      },
      size: { known: false, signals: [] },
      sources: [],
    };

    const errors = validateBroker(brokerWebForm, { sourceIds: new Set() });
    expect(errors).toEqual([]);
  });
});
```