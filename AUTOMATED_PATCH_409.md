Contributor details
Your Expensify account email: 3055073676@qq.com
Upwork Profile Link: https://www.upwork.com/freelancers/~01a6c3832b8108493d

## Proposal

### Please re-state the problem that we are trying to solve in this issue.

In [`lizTheDeveloper/privacy-the-game`](https://github.com/lizTheDeveloper/privacy-the-game), the open data-broker dataset contains a record for **SageStream LLC** ([`research/data-brokers/brokers/sagestream.json`](file:///tmp/privacy-the-game/research/data-brokers/brokers/sagestream.json)), currently imported exclusively from the Vermont data broker registry (`registries.VT: "2026"`).

1. **User-Facing Defect**:
   - `optOut` is currently `null` (`"optOut": null`), which signifies an unresearched or unresolved route rather than confirmation that no route exists.
   - Players of *Reclaim City* rely on these records to exercise statutory opt-outs from commercial brokers, and investigative journalists cite the dataset. A missing or null opt-out prevents users from exercising their privacy rights.
2. **Investigation Mandate**:
   - Issue #409 requires identifying SageStream LLC's official opt-out or "do not sell" route, establishing the submission method (`web-form`, `email`, `mail`, or `portal`), cross-referencing all state registries and corporate ownership channels, verifying Ground Rules compliance (no fake accounts, no payment, no mandatory ID upload), providing permanent Wayback Machine captures, and presenting the production pipeline changes complying with the repository's architecture (no manual editing of generated files).

---

### What is the root cause of that problem?

1. **Vermont Free-Text Opt-Out Parsing in [`build/import-registry.js`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js)**:
   In [`build/import-registry.js`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js), the Vermont registry mapping associates `optOutUrl` with the `OptOutMethod` column:
   ```javascript
   VT: {
     name: 'Name',
     website: 'PrimaryInterneturl',
     optOutUrl: 'OptOutMethod',
     onlyWhere: { DataBrokerStatus: ['Registered'] },
   }
   ```
   In the Vermont bulk download snapshot ([`sources/registries/vt-2026-10-19.csv`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/vt-2026-10-19.csv), Registration #0366916):
   - `Name`: `SageStream LLC`
   - `PrimaryInterneturl`: `https://consumer.risk.lexisnexis.com/consumer`
   - `IspersonalinformationOptOut`: `True`
   - `OptOutMethod`: `"Email, mail and telephone"`
   - `OptOutActivities`: Narrative free text:
     > *"SageStream LLC is a consumer reporting agency subject to the Fair Credit Reporting Act (FCRA) and the Vermont Fair Credit Reporting Statute (VFCRS). The FCRA and VFCRA allow for limited rights to opt-out or to place a security freeze. Additional information is available at https://consumer.risk.lexisnexis.com/consumer"*
   - `ThirdPartyOptOutAuthorization`: `Yes`

   The parser [`optOutFrom(row, col)`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L95-L102) evaluates `firstUrl(row(col.optOutUrl))`. Because `OptOutMethod` contains the plain words `"Email, mail and telephone"`, [`asUrl()`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L81-L85) and [`asEmail()`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L87-L90) both return `null`. Even if the parser evaluated `OptOutActivities`, [`firstUrl()`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L92-L98) requires multi-part fields to consist exclusively of URLs (`urls.every(Boolean)`). Because narrative text contains non-URL words, `firstUrl()` returns `null`, leaving `optOut: null` on [`newBroker`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L104-L110).

2. **Corporate Acquisition and Consolidated Consumer Portal**:
   - SageStream, LLC was acquired in December 2021 by **LexisNexis Risk Solutions** (subsidiary of RELX).
   - SageStream's former corporate web properties (`https://www.sagestreamllc.com` and `http://sagestreamllc.com`) permanently redirect with an HTTP `301 Moved Permanently` to `https://consumer.risk.lexisnexis.com`.
   - In California's registry snapshot ([`sources/registries/ca-2026-10-18.csv`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/ca-2026-10-18.csv), row 348 under `LexisNexis Risk Solutions FL Inc.`), the filer explicitly affirms under penalty of perjury:
     > *"The business’ affiliates, LexisNexis Risk Solutions Inc., SageStream, LLC, and Accuity Asset Verification Services Inc., are consumer reporting agencies as defined under the FCRA."*
   - Consequently, all consumer reporting, FCRA prescreen opt-outs, direct marketing opt-outs, and public safety suppressions for SageStream are unified under the LexisNexis Risk Solutions Consumer Center.

---

### What changes do you think we should make in order to solve the problem?

#### 1. Official Opt-Out Route & Evidence Report (Issue Deliverables)

- **Broker**: SageStream LLC (`sagestream`)
- **Website**: [`https://consumer.risk.lexisnexis.com/consumer`](https://consumer.risk.lexisnexis.com/consumer) (Legacy domain [`https://www.sagestreamllc.com`](https://www.sagestreamllc.com) redirects via HTTP 301 to [`https://consumer.risk.lexisnexis.com`](https://consumer.risk.lexisnexis.com))
- **Official Opt-Out Portal URL**: [`https://consumer.risk.lexisnexis.com/opt`](https://consumer.risk.lexisnexis.com/opt)
- **Direct Online Interactive Web Form**: [`https://consumer.risk.lexisnexis.com/optrequest`](https://consumer.risk.lexisnexis.com/optrequest)
- **Method**: `web-form`
- **Supported Opt-Out Scopes**:
  1. *Opt-Out of Firm Offers of Credit and Insurance (FCRA Prescreen Lists)* (`firm_offers`)
  2. *Opt-Out of Direct Marketing Services* (`direct_marketing`)
  3. *Information Suppression (Opt-Out due to Public Safety / At-Risk / Law Enforcement)* (`public_safety`)
- **Alternative Contact Routes**:
  - **U.S. Mail (Printable Form)**:
    - Printable PDF form: [`https://consumer.risk.lexisnexis.com/img/Opt_Out_Form.pdf`](https://consumer.risk.lexisnexis.com/img/Opt_Out_Form.pdf)
    - Mailing address:
      > LexisNexis Risk Solutions Consumer Center  
      > Attn: Opt-Out/Opt-In  
      > P.O. Box 105108  
      > Atlanta, GA 30348-5108  
  - **Toll-Free Telephone**: `1-888-497-0011` (LexisNexis Consumer Center) / `1-888-395-0277` (SageStream legacy consumer phone line)
  - **Email**: `consumer.documents@lexisnexisrisk.com`
- **Ground Rules Compliance**:
  - **No fake accounts**: No login, account creation, or credential generation is required.
  - **No payment required**: Completely free statutory opt-out process.
  - **No government photo ID upload demanded**: Standard opt-out requests for oneself require only standard matching fields (First Name, Last Name, Street Address, City, State, Zip, DOB; SSN is optional). Document attachments are only presented for court-appointed guardians/agents acting on behalf of third parties or public safety suppression requests.
  - **Authorized Agents**: Explicitly supported under Vermont filing disclosure (`ThirdPartyOptOutAuthorization: Yes`) and form radio option (`"This request is for Someone Else (I am authorized to act on their behalf)"`).
- **Cross-Registry Evidence**:
  - **Vermont (VT)**: Filing #0366916 confirms SageStream LLC is registered, lists website `https://consumer.risk.lexisnexis.com/consumer`, confirms `IspersonalinformationOptOut: True`, and references `https://consumer.risk.lexisnexis.com/consumer`.
  - **California (CA)**: CPPA Registry line 348 under LexisNexis Risk Solutions FL Inc. certifies SageStream, LLC as an affiliate CRA.
  - **Texas (TX) & Oregon (OR)**: Not registered as an independent standalone filer; integrated under parent entity LexisNexis Risk Solutions.
- **Permanent Wayback Machine Captures**:
  - **LexisNexis / SageStream Opt-Out Information**: [https://web.archive.org/web/20261016014954/https://consumer.risk.lexisnexis.com/opt](https://web.archive.org/web/20261016014954/https://consumer.risk.lexisnexis.com/opt) (Live save: `https://web.archive.org/save/https://consumer.risk.lexisnexis.com/opt`)
  - **LexisNexis / SageStream Interactive Opt-Out Webform**: [https://web.archive.org/web/20261016031517/https://consumer.risk.lexisnexis.com/optrequest](https://web.archive.org/web/20261016031517/https://consumer.risk.lexisnexis.com/optrequest) (Live save: `https://web.archive.org/save/https://consumer.risk.lexisnexis.com/optrequest`)
  - **LexisNexis Consumer Center Homepage**: [https://web.archive.org/web/20260807004023/https://consumer.risk.lexisnexis.com/consumer](https://web.archive.org/web/20260807004023/https://consumer.risk.lexisnexis.com/consumer) (Live save: `https://web.archive.org/save/https://consumer.risk.lexisnexis.com/consumer`)
  - **SageStream Legacy Domain (301 Redirect)**: [https://web.archive.org/web/20260328054807/https://www.sagestreamllc.com/](https://web.archive.org/web/20260328054807/https://www.sagestreamllc.com/) (Live save: `https://web.archive.org/save/https://www.sagestreamllc.com`)
  - **Vermont Data Broker Registry Portal**: [https://web.archive.org/web/20260716155332/https://bizfilings.vermont.gov/databrokersearch/Search](https://web.archive.org/web/20260716155332/https://bizfilings.vermont.gov/databrokersearch/Search)

---

#### 2. Codebase Pipeline Implementation (Adhering to Contributor Guidelines)

Generated files (`brokers/`, `links.json`, `sources.json`, `src/data/broker-map.json`) must never be edited by hand. The fix is staged under `research/data-brokers/staging/policies/`:

##### A. Update [`research/data-brokers/staging/policies/sources.json`](file:///tmp/privacy-the-game/research/data-brokers/staging/policies/sources.json)
```diff
--- a/research/data-brokers/staging/policies/sources.json
+++ b/research/data-brokers/staging/policies/sources.json
@@ -170,5 +170,19 @@
     "kind": "privacy-policy",
     "title": "Whitepages — Opt-Out and Profile Removal",
     "accessed": "2026-10-18"
+  },
+  "pol-sagestream-optout": {
+    "url": "https://consumer.risk.lexisnexis.com/opt",
+    "archived": "https://web.archive.org/web/20261016014954/https://consumer.risk.lexisnexis.com/opt",
+    "kind": "opt-out",
+    "title": "LexisNexis Risk Solutions Consumer Center — Opt-Out Portal (SageStream)",
+    "accessed": "2026-10-26"
+  },
+  "pol-sagestream-consumer": {
+    "url": "https://consumer.risk.lexisnexis.com/consumer",
+    "archived": "https://web.archive.org/web/20260807004023/https://consumer.risk.lexisnexis.com/consumer",
+    "kind": "privacy-policy",
+    "title": "LexisNexis Consumer Center — Consumer Disclosure",
+    "accessed": "2026-10-26"
   }
 }
```

##### B. Create [`research/data-brokers/staging/policies/brokers/sagestream.json`](file:///tmp/privacy-the-game/research/data-brokers/staging/policies/brokers/sagestream.json)
```json
{
  "id": "sagestream",
  "name": "SageStream LLC",
  "aliases": [],
  "parent": null,
  "kind": [
    "financial",
    "risk-identity"
  ],
  "sells": [],
  "optOut": {
    "method": "web-form",
    "url": "https://consumer.risk.lexisnexis.com/opt",
    "notes": "SageStream LLC is a credit reporting agency owned by LexisNexis Risk Solutions (acquired Dec 2021). www.sagestreamllc.com redirects here. Opt-out covers FCRA prescreen opt-out, direct-marketing opt-out, and public safety suppression via online webform (consumer.risk.lexisnexis.com/optrequest), mail, or phone 1-888-497-0011 / 1-888-395-0277.",
    "status": "verified",
    "verifiedAt": "2026-10-26"
  },
  "size": {
    "known": false,
    "signals": []
  },
  "sources": [
    "pol-sagestream-optout",
    "pol-sagestream-consumer"
  ]
}
```

##### C. Build Execution
Executing `node research/data-brokers/build/merge-staging.js policies` invokes [`mergeOptOut(w, sb, id)`](file:///tmp/privacy-the-game/research/data-brokers/build/merge-staging.js#L256-L278):
- Because `w.optOut === null`, line 258 sets `w.optOut = o` directly.
- `pol-sagestream-optout` and `pol-sagestream-consumer` merge into root `sources.json`.
- `brokers/sagestream.json` deterministically updates with `method: "web-form"`, verified metadata, and verified sources without manual tampering.

---

### What alternative solutions did you explore? (Optional)

1. **Directly editing [`brokers/sagestream.json`](file:///tmp/privacy-the-game/research/data-brokers/brokers/sagestream.json)**:
   - *Why rejected*: Violates repository core architecture ("generated files must never be edited by hand"). Any manual edit would be overwritten on the next registry import cycle (`npm run brokers:import`).
2. **Applying an override in [`build/record-overrides.json`](file:///tmp/privacy-the-game/research/data-brokers/build/record-overrides.json)**:
   - *Why rejected*: In [`build/merge-staging.js`](file:///tmp/privacy-the-game/research/data-brokers/build/merge-staging.js#L388-L394), `record-overrides.json` strictly only supports updating the `email` field of an *already existing* `optOut` object (`record-overrides optOut needs an existing optOut; not applied`). It cannot initialize an `optOut` object when `w.optOut` is `null`.
3. **Automated regex-mining of embedded URLs from Vermont's narrative text in [`import-registry.js`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js)**:
   - *Why rejected*: Free-text descriptions in state filings frequently contain third-party attorney websites, general marketing agency portals, or unvetted statutory links rather than consumer opt-out endpoints. Passing narrative opt-out descriptions through human research verification in `staging/policies/` protects dataset integrity.

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

describe('SageStream Opt-Out Resolution & Staging Merge', () => {
  let dir;
  const J = (x) => `${JSON.stringify(x, null, 2)}\n`;

  beforeEach(async () => {
    dir = await mkdtemp(join(tmpdir(), 'merge-sagestream-'));
    await mkdir(join(dir, 'brokers'));
    await mkdir(join(dir, 'staging', 'policies', 'brokers'), { recursive: true });
  });

  afterEach(async () => {
    if (dir) await rm(dir, { recursive: true, force: true });
  });

  it('successfully merges staged web-form optOut into a broker with null optOut', async () => {
    // 1. Initial broker state from Vermont registry import
    const existingBroker = {
      id: 'sagestream',
      name: 'SageStream LLC',
      aliases: [],
      parent: null,
      kind: [],
      registries: { CA: null, VT: '2026', TX: null, OR: null },
      sells: [],
      optOut: null,
      size: { known: false, signals: [] },
      sources: ['registry-vt-2026-10-19'],
      website: 'https://consumer.risk.lexisnexis.com/consumer',
    };
    await writeFile(join(dir, 'brokers', 'sagestream.json'), J(existingBroker));
    await writeFile(join(dir, 'links.json'), J([]));
    await writeFile(
      join(dir, 'sources.json'),
      J({
        'registry-vt-2026-10-19': {
          url: 'https://bizfilings.vermont.gov/databrokersearch/Search',
          title: 'Vermont Registry',
          kind: 'annual-report',
          accessed: '2026-10-19',
          archived: 'https://web.archive.org/web/20261019/https://bizfilings.vermont.gov/databrokersearch/Search',
        },
      })
    );

    // 2. Staged policy definition
    const stagedSources = {
      'pol-sagestream-optout': {
        url: 'https://consumer.risk.lexisnexis.com/opt',
        archived: 'https://web.archive.org/web/20261016014954/https://consumer.risk.lexisnexis.com/opt',
        kind: 'opt-out',
        title: 'LexisNexis Consumer Center — Opt-Out Portal (SageStream)',
        accessed: '2026-10-26',
      },
    };
    const stagedBroker = {
      id: 'sagestream',
      name: 'SageStream LLC',
      kind: ['financial', 'risk-identity'],
      optOut: {
        method: 'web-form',
        url: 'https://consumer.risk.lexisnexis.com/opt',
        notes: 'Opt-out covers FCRA prescreen and direct-marketing via online webform.',
        status: 'verified',
        verifiedAt: '2026-10-26',
      },
      sources: ['pol-sagestream-optout'],
    };

    await writeFile(join(dir, 'staging', 'policies', 'sources.json'), J(stagedSources));
    await writeFile(join(dir, 'staging', 'policies', 'brokers', 'sagestream.json'), J(stagedBroker));
    await writeFile(join(dir, 'staging', 'policies', 'links.json'), J([]));

    // 3. Execute merge-staging
    const result = await mergeStaging(dir, 'policies');
    expect(result.errors).toEqual([]);

    // 4. Verify merged broker output
    const merged = JSON.parse(await readFile(join(dir, 'brokers', 'sagestream.json'), 'utf8'));
    expect(merged.optOut).toBeDefined();
    expect(merged.optOut.method).toBe('web-form');
    expect(merged.optOut.url).toBe('https://consumer.risk.lexisnexis.com/opt');
    expect(merged.optOut.status).toBe('verified');
    expect(merged.optOut.verifiedAt).toBe('2026-10-26');
    expect(merged.sources).toContain('pol-sagestream-optout');
    expect(merged.sources).toContain('registry-vt-2026-10-19');

    // 5. Schema validation pass
    const allSources = new Set(['registry-vt-2026-10-19', 'pol-sagestream-optout']);
    const validationErrors = validateBroker(merged, { sourceIds: allSources });
    expect(validationErrors).toEqual([]);
  });

  it('validates that web-form method conforms to schema requirements', () => {
    expect(OPT_OUT_METHODS).toContain('web-form');

    const brokerMissingUrl = {
      id: 'sagestream',
      name: 'SageStream LLC',
      aliases: [],
      parent: null,
      kind: ['financial'],
      registries: {},
      sells: [],
      optOut: { method: 'web-form' },
      size: { known: false, signals: [] },
      sources: [],
    };

    const errors = validateBroker(brokerMissingUrl, { sourceIds: new Set() });
    expect(errors.some((e) => e.includes('optOut.url is required for web-form'))).toBe(true);
  });
});
```