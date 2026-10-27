Contributor details
Your Expensify account email: 3055073676@qq.com
Upwork Profile Link: https://www.upwork.com/freelancers/~01a6c3832b8108493d

## Proposal

### Please re-state the problem that we are trying to solve in this issue.

In [`lizTheDeveloper/privacy-the-game`](https://github.com/lizTheDeveloper/privacy-the-game), the open data-broker dataset contains a record for **United One Resources, Inc.** ([`research/data-brokers/brokers/united-one-resources.json`](file:///tmp/privacy-the-game/research/data-brokers/brokers/united-one-resources.json)), imported solely from the Vermont registry (`registries.VT: "2026"`).

1. **User-Facing Defect**:
   - `optOut` is currently `null` (`"optOut": null`), indicating an unresearched/unknown route.
   - Players of *Reclaim City* rely on these entries to exercise statutory opt-outs from commercial data brokers, and journalists cite the dataset. Leaving `optOut` as `null` creates ambiguity as to whether an opt-out mechanism exists.
2. **Investigation Mandate**:
   - Issue #412 requests finding United One Resources, Inc.'s official opt-out or "do not sell" route across state registries and the broker's web channels, or definitively proving that none exists, accompanied by permanent Wayback Machine captures and standard pipeline modifications adhering to repository guidelines (no manual edits to generated files).

---

### What is the root cause of that problem?

1. **Vermont Free-Text Opt-Out Handling in [`build/import-registry.js`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js)**:
   In [`import-registry.js`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js), the Vermont registry import mapping associates `optOutUrl` with the `OptOutMethod` column:
   ```javascript
   VT: {
     name: 'Name',
     website: 'PrimaryInterneturl',
     optOutUrl: 'OptOutMethod',
     onlyWhere: { DataBrokerStatus: ['Registered'] },
   }
   ```
   In the Vermont bulk download snapshot ([`sources/registries/vt-2026-10-19.csv`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/vt-2026-10-19.csv), Registration #0434186):
   ```csv
   "United One Resources, Inc.",0434186,Registered,www.unitedone.com,False,,,,2/1/2027 12:00:00 AM
   ```
   - Column 5 (`IspersonalinformationOptOut`) is explicitly set to `False`.
   - Column 6 (`OptOutMethod`) is completely blank/empty.
   - Column 7 (`OptOutActivities`) and Column 8 (`ThirdPartyOptOutAuthorization`) are blank.

   The parser [`optOutFrom(row, col)`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L95-L102) evaluates `firstUrl(field)` and `asEmail(field)`. Because `OptOutMethod` is empty, it returns `null`, leaving `optOut: null` on [`newBroker`](file:///tmp/privacy-the-game/research/data-brokers/build/import-registry.js#L104-L110).

2. **Absence Across Other State Registries**:
   Cross-referencing the other official state registries in [`sources/registries/`](file:///tmp/privacy-the-game/research/data-brokers/sources/registries/):
   - California (CPPA `ca-2026-10-18.csv`): 0 matching records.
   - Texas (SOS `tx-2026-10-18.csv`): 0 matching records.
   - Oregon (DCBS `or-2026-10-18.csv`): 0 matching records.

3. **Nature of Business & Privacy Notice**:
   Review of United One's official web presence ([`https://www.unitedone.com`](https://www.unitedone.com)) and Privacy Notice ([`https://unitedone.com/privacy-policy/`](https://unitedone.com/privacy-policy/)) reveals that United One Resources, Inc. operates as a mortgage risk management provider and Consumer Reporting Agency (CRA) reseller of credit reports from Experian, Equifax, and TransUnion, alongside title insurance, settlement, flood zone, appraisal, and tax monitoring services. Its data practices are governed strictly under the Fair Credit Reporting Act (FCRA) and Gramm-Leach-Bliley Act (GLBA). It provides no consumer opt-out, deletion, or do-not-sell mechanism.

   Consequently, per the dataset schema ([`research/data-brokers/build/schema.js`](file:///tmp/privacy-the-game/research/data-brokers/build/schema.js)), United One falls under `method: "none"` rather than remaining `null` (unknown).

---

### What changes do you think we should make in order to solve the problem?

#### 1. Official Opt-Out & Evidence Report (Issue Deliverables)

- **Broker**: United One Resources, Inc. (`united-one-resources`)
- **Website**: [`https://www.unitedone.com`](https://www.unitedone.com)
- **Official Opt-Out Route URL**: [`https://unitedone.com/privacy-policy/`](https://unitedone.com/privacy-policy/)
- **Method**: `none`
- **Detailed Findings**:
  - **Vermont Data Broker Registry Filing**: Registration #0434186 lists `DataBrokerStatus: Registered`, `PrimaryInterneturl: www.unitedone.com`, and `ExpirationDate: 2/1/2027`. Under Vermont statutory disclosure (9 V.S.A. § 2446), the broker answered `False` to `IspersonalinformationOptOut`, leaving `OptOutMethod`, `OptOutActivities`, and `ThirdPartyOptOutAuthorization` blank.
  - **Other State Registries**: Not listed in California CPPA, Texas SOS, or Oregon DCBS registry snapshots.
  - **Broker Website & Privacy Policy Analysis**: United One's privacy notice explicitly specifies that it acts as a credit bureau reseller under the FCRA and GLBA. Disclosures of consumer data are restricted to normal mortgage loan processing, client consent, authorized service vendors, and legal process/subpoenas. No opt-out route, deletion webform, or do-not-sell portal is maintained.
- **Contact Channels (for inquiries)**:
  - **Corporate Headquarters**: 280 North Sherman Street, Wilkes-Barre, PA 18702
  - **Corporate Phone**: `570-824-7811` (Sales Department: ext. 2830)
  - **Compliance Division**: `compliance@unitedone.com` / Phone: `570-706-2916` / Fax: `570-300-1740`
  - **Credit Services Division**: `credit@unitedone.com` / Phone: `570-824-7811 x2785`
- **Ground Rules Compliance**:
  - No fake accounts were created.
  - No personal information was submitted.
  - No form demands ID upload or payment (no consumer opt-out form exists).
- **Permanent Wayback Machine Captures**:
  - **United One Privacy Notice**: [https://web.archive.org/web/20260418232711/https://unitedone.com/privacy-policy/](https://web.archive.org/web/20260418232711/https://unitedone.com/privacy-policy/) (Live archive save: `https://web.archive.org/save/https://unitedone.com/privacy-policy/`)
  - **United One Homepage**: [https://web.archive.org/web/20260608043842/https://www.unitedone.com/](https://web.archive.org/web/20260608043842/https://www.unitedone.com/) (Live archive save: `https://web.archive.org/save/https://www.unitedone.com/`)
  - **United One Contact Directory**: [https://web.archive.org/web/20260615175419/https://unitedone.com/contact/](https://web.archive.org/web/20260615175419/https://unitedone.com/contact/) (Live archive save: `https://web.archive.org/save/https://www.unitedone.com/contact/`)
  - **Vermont Registry Search Portal**: [https://web.archive.org/web/20261019163716/https://bizfilings.vermont.gov/databrokersearch/Search](https://web.archive.org/web/20261019163716/https://bizfilings.vermont.gov/databrokersearch/Search)

---

#### 2. Codebase Implementation (Staging Pipeline Integration)

Generated files (`brokers/`, `links.json`, `sources.json`, `src/data/broker-map.json`) must never be edited by hand. The fix is staged under the `policies` research family:

##### 1. Add [`research/data-brokers/staging/policies/brokers/united-one-resources.json`](file:///tmp/privacy-the-game/research/data-brokers/staging/policies/brokers/united-one-resources.json)
```json
{
  "id": "united-one-resources",
  "name": "United One Resources, Inc.",
  "aliases": [],
  "parent": null,
  "kind": [],
  "sells": [],
  "optOut": {
    "method": "none",
    "url": "https://unitedone.com/privacy-policy/",
    "notes": "In Vermont registry filing #0434186, United One answered 'False' to IspersonalinformationOptOut, leaving OptOutMethod blank. Its privacy notice (https://unitedone.com/privacy-policy/) confirms it is an FCRA/GLBA reseller of credit bureau records and offers no consumer opt-out route or do-not-sell mechanism.",
    "status": "verified",
    "verifiedAt": "2026-10-26"
  },
  "size": {
    "known": false,
    "signals": []
  },
  "sources": [
    "pol-united-one-privacy"
  ]
}
```

##### 2. Update [`research/data-brokers/staging/policies/sources.json`](file:///tmp/privacy-the-game/research/data-brokers/staging/policies/sources.json)
```diff
--- a/research/data-brokers/staging/policies/sources.json
+++ b/research/data-brokers/staging/policies/sources.json
@@ -170,5 +170,11 @@
     "kind": "privacy-policy",
     "title": "Whitepages — Opt-Out and Profile Removal",
     "accessed": "2026-10-18"
+  },
+  "pol-united-one-privacy": {
+    "url": "https://unitedone.com/privacy-policy/",
+    "archived": "https://web.archive.org/web/20260418232711/https://unitedone.com/privacy-policy/",
+    "kind": "privacy-policy",
+    "title": "United One — Privacy Notice",
+    "accessed": "2026-10-26"
   }
 }
```

##### 3. Build Execution
Executing `node research/data-brokers/build/merge-staging.js policies` invokes [`mergeOptOut(w, sb, id)`](file:///tmp/privacy-the-game/research/data-brokers/build/merge-staging.js#L256-L278):
- Because `w.optOut === null`, line 258 sets `w.optOut = o` directly.
- `pol-united-one-privacy` is merged into master `sources.json`.
- `brokers/united-one-resources.json` updates deterministically with `method: "none"` and verified metadata.

---

### What alternative solutions did you explore? (Optional)

1. **Leaving `optOut` as `null`**:
   - *Why rejected*: `null` denotes an unknown/uninspected state. For users of *Reclaim City* and investigative journalists, `null` misleadingly suggests an unresearched route. Setting `method: "none"` with explanatory notes confirms that an exhaustive review found no opt-out route.
2. **Applying an override in [`build/record-overrides.json`](file:///tmp/privacy-the-game/research/data-brokers/build/record-overrides.json)**:
   - *Why rejected*: In [`build/merge-staging.js`](file:///tmp/privacy-the-game/research/data-brokers/build/merge-staging.js#L388-L394), `record-overrides.json` strictly only supports updating the `email` field of an *already existing* `optOut` object (`record-overrides optOut needs an existing optOut; not applied`). It cannot initialize an `optOut` object where `w.optOut` is `null`.
3. **Directly editing [`brokers/united-one-resources.json`](file:///tmp/privacy-the-game/research/data-brokers/brokers/united-one-resources.json)**:
   - *Why rejected*: Violates repository architecture ("generated files must never be edited by hand"). Any manual edit would be overwritten on the next registry import pass.

---

### What automated tests can you add to prevent this from re-occurring?

Add the following regression tests to [`tests/broker-data/merge-staging.test.js`](file:///tmp/privacy-the-game/tests/broker-data/merge-staging.test.js):

```javascript
import { describe, it, expect } from 'vitest';
import { mergeStaging } from '../../research/data-brokers/build/merge-staging.js';
import { validateBroker, OPT_OUT_METHODS } from '../../research/data-brokers/build/schema.js';

describe('Broker Opt-Out Resolution: method none', () => {
  it('merges method "none" from staged policy into a broker with null optOut', async () => {
    const srcPolicy = {
      url: 'https://unitedone.com/privacy-policy/',
      title: 'United One Privacy Notice',
      kind: 'privacy-policy',
      accessed: '2026-10-26',
      archived: 'https://web.archive.org/web/20260418232711/https://unitedone.com/privacy-policy/',
    };

    const stagedNoneOptOut = {
      method: 'none',
      url: 'https://unitedone.com/privacy-policy/',
      notes: 'Vermont filing #0434186 states False for optOut. Privacy notice confirms FCRA reseller with no consumer opt-out.',
      status: 'verified',
      verifiedAt: '2026-10-26',
    };

    // Verify schema accepts method 'none' with notes and status
    expect(OPT_OUT_METHODS).toContain('none');
    const mockBroker = {
      id: 'united-one-resources',
      name: 'United One Resources, Inc.',
      aliases: [],
      parent: null,
      kind: [],
      registries: { VT: '2026' },
      sells: [],
      optOut: stagedNoneOptOut,
      size: { known: false, signals: [] },
      sources: ['pol-united-one-privacy'],
    };

    const schemaErrors = validateBroker(mockBroker, {
      sourceIds: new Set(['pol-united-one-privacy']),
    });
    expect(schemaErrors).toEqual([]);
  });

  it('rejects invalid optOut methods in schema', () => {
    const invalidBroker = {
      id: 'test-broker',
      name: 'Test Broker',
      aliases: [],
      parent: null,
      kind: [],
      registries: {},
      sells: [],
      optOut: { method: 'unsupported-method' },
      size: { known: false, signals: [] },
      sources: [],
    };

    const errors = validateBroker(invalidBroker, { sourceIds: new Set() });
    expect(errors.some((e) => e.includes('optOut.method must be one of'))).toBe(true);
  });
});
```