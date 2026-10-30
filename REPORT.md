# ant-design: Table `clearFilters({ confirm: true })` ignores `filterResetToDefaultFilteredValue`

## Branch, commit and target

| | |
| --- | --- |
| Fork | `b84330808/ant-design` |
| Branch | `claude/sweet-ramanujan-2d6x71` |
| Commit | `79a34c6ef0ad167abd6b3484d8a8ae47a988ddc6` (one commit) |
| Built on | upstream `master` at `db7fb1c660` (`fix(Button): respect tabIndex when href is set (#59732)`) |
| Target base | `ant-design/ant-design` **`master`** |
| Author | `b84330808 <b84330808@gmail.com>`, trailer `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>` |

Files changed:

```
 components/table/__tests__/Table.filter.test.tsx   | 54 ++++++++++++++++++++++
 .../table/hooks/useFilter/FilterDropdown.tsx       | 13 +++---
 2 files changed, 60 insertions(+), 7 deletions(-)
```

To open the PR:

```bash
gh pr create --repo ant-design/ant-design --base master --head b84330808:claude/sweet-ramanujan-2d6x71 --title "fix(Table): restore default filter on clearFilters confirm" --body-file PR_BODY.md
```

(Copy the PR body below into `PR_BODY.md` first.)

## Policy check

- `.github/CONTRIBUTING.md` points to `docs/react/contributing.en-US.md`. Neither has an AI or agent policy, and neither asks the author to attest to personal testing. It says bug fixes go to `master`, features to `feature`. It also asks for tests and for `npm test` and `npm run lint` to pass.
- `AGENTS.md` / `CLAUDE.md` are guidance written for AI coding assistants, not a ban. They ask for:
  - PR titles in English, in the form `type: description`
  - the EN or CN PR template
  - relative imports in `__tests__`
  - a simulated merge into `feature` before targeting `master`. The rule was reworded upstream in #59728 (2026-10), and I re-read it at the current HEAD.
- PR template: `.github/PULL_REQUEST_TEMPLATE.md`, with an EN/CN changelog table. The body below follows it.
- Simulated merge with the latest `upstream/feature` (`20477fd4a1`): `git merge-tree --write-tree upstream/feature HEAD` exits 0 with no conflicts. Neither changed file differs between `master` and `feature`. So the target stays `master`.
- Our open PR, [ant-design/ant-design#59611](https://github.com/ant-design/ant-design/pull/59611) (`defaultSortOrder` on grouping columns), changes `useSorter.tsx` and `Table.sorter.test.tsx`. This change only touches `useFilter/FilterDropdown.tsx` and `Table.filter.test.tsx`, so they don't overlap.

## The bug

`FilterDropdown`'s `onReset` (the function a custom `filterDropdown` gets as `clearFilters`) did two inconsistent things when `filterResetToDefaultFilteredValue` was set:

```ts
if (confirm) {
  internalTriggerFilter([]);            // applies an EMPTY filter
}
...
if (filterResetToDefaultFilteredValue) {
  setFilteredKeysSync((defaultFilteredValue || []).map<string>(String));  // but the panel shows the defaults
} else {
  setFilteredKeysSync([]);
}
```

So `clearFilters({ confirm: true })` removed the filter completely: every row shows and `onChange` reports `null` for the column. Meanwhile the dropdown's `selectedKeys` were set to `defaultFilteredValue`. The applied filter and the panel disagree, and the documented meaning of `filterResetToDefaultFilteredValue` ("click the reset button, whether to restore the default filter") is lost exactly when the reset is confirmed.

The fix computes the reset keys once and uses them in both places. `clearFilters({ confirm: true })` now behaves like `clearFilters()` followed by `confirm()`. Without `filterResetToDefaultFilteredValue`, the keys are `[]` as before, so nothing else changes. The built-in reset button calls `onReset()` without `confirm`, so it's unaffected.

History: `clearFilters({ confirm, closeDropdown })` came in #34120 (Feb 2022) and `filterResetToDefaultFilteredValue` in #34355 (Mar 2022). These two features were never made to work together.

## PR title

```
fix(Table): restore default filter on clearFilters confirm
```

## PR body

```markdown
[中文版模板 / Chinese template](https://github.com/ant-design/ant-design/blob/master/.github/PULL_REQUEST_TEMPLATE_CN.md?plain=1)

### 🤔 This is a ...

- [ ] 🆕 New feature
- [x] 🐞 Bug fix
- [ ] 📝 Site / documentation improvement
- [ ] 📽️ Demo improvement
- [ ] 💄 Component style improvement
- [ ] 🤖 TypeScript definition improvement
- [ ] 📦 Bundle size optimization
- [ ] ⚡️ Performance optimization
- [ ] ⭐️ Feature enhancement
- [ ] 🌐 Internationalization
- [ ] 🛠 Refactoring
- [ ] 🎨 Code style optimization
- [ ] ✅ Test Case
- [ ] 🔀 Branch merge
- [ ] ⏩ Workflow
- [ ] ⌨️ Accessibility improvement
- [ ] ❓ Other (about what?)

### 🔗 Related Issues

No existing issue. I found this while reading the filter reset logic, and couldn't find an issue or PR about it.

### 💡 Background and Solution

A custom `filterDropdown` can call `clearFilters({ confirm: true })` to reset the filter and apply the reset right away. With `filterResetToDefaultFilteredValue: true`, the reset should restore `defaultFilteredValue`. But `onReset` in `FilterDropdown` always applied `[]` when `confirm` was set, and only reset the dropdown's own selected keys to `defaultFilteredValue`. So the table dropped the filter completely, `onChange` reported `null` for that column, and the dropdown's `selectedKeys` no longer matched the applied filter.

Steps to reproduce:

1. Add a column with `defaultFilteredValue: ['Jack']`, `filterResetToDefaultFilteredValue: true` and a custom `filterDropdown` whose reset button calls `clearFilters({ confirm: true, closeDropdown: true })`.
2. Filter by another value and confirm.
3. Click the reset button. All rows are shown, instead of only Jack.

`onReset` now works out the reset keys once and uses them both for the dropdown selection and for the filter applied on `confirm`. So `clearFilters({ confirm: true })` gives the same result as `clearFilters()` followed by `confirm()`. Without `filterResetToDefaultFilteredValue`, nothing changes. There's no API change.

The new test in `Table.filter.test.tsx` covers this. It fails on `master` and passes with this change. All Table tests pass, and ESLint, Biome and `tsc` are clean.

### 📝 Change Log

| Language   | Changelog |
| ---------- | --------- |
| 🇺🇸 English | Fix Table custom `filterDropdown` `clearFilters({ confirm: true })` clearing the filter instead of restoring `defaultFilteredValue` when `filterResetToDefaultFilteredValue` is enabled. |
| 🇨🇳 Chinese | 修复 Table 开启 `filterResetToDefaultFilteredValue` 时，自定义 `filterDropdown` 调用 `clearFilters({ confirm: true })` 会清空筛选而不是恢复 `defaultFilteredValue` 的问题。 |

I used Claude Code to write this change.
```

## Fail before (fix reverted, new test present)

Setup: commit `79a34c6ef0` with only `components/table/hooks/useFilter/FilterDropdown.tsx` checked out from `upstream/master`. That keeps the new test and reverts the fix. I got the same failure earlier, when I wrote the test before applying the fix.

Command: `npx jest --config .jest.ts --no-cache components/table/__tests__/Table.filter.test.tsx -t "clearFilters with confirm should apply defaultFilteredValue"`

```
FAIL components/table/__tests__/Table.filter.test.tsx (14.266 s)
  ● Table.filter › clearFilters with confirm should apply defaultFilteredValue when filterResetToDefaultFilteredValue
    expect(received).toEqual(expected) // deep equality
    - Expected  - 0
    + Received  + 3
      Array [
        "Jack",
    +   "Lucy",
    +   "Tom",
    +   "Jerry",
      ]
      2784 |     fireEvent.click(container.querySelector('span.ant-dropdown-trigger')!);
      2785 |     fireEvent.click(container.querySelector('#reset')!);
    > 2786 |     expect(renderedNames(container)).toEqual(['Jack']);
           |                                      ^
      2787 |     expect(onChange).toHaveBeenLastCalledWith(
      2788 |       expect.anything(),
      2789 |       { name: ['Jack'] },
      at Object.toEqual (components/table/__tests__/Table.filter.test.tsx:2786:38)
Test Suites: 1 failed, 1 total
Tests:       1 failed, 95 skipped, 96 total
Snapshots:   0 total
Time:        15.194 s
```

## Pass after (with the fix)

Same command:

```
Test Suites: 1 passed, 1 total
Tests:       95 skipped, 1 passed, 96 total
Snapshots:   0 total
Time:        14.151 s
```

## Wider checks (with the fix)

- **Table suite**, `npx jest --config .jest.ts --no-cache components/table`:
  ```
  Test Suites: 17 passed, 17 total
  Tests:       17 skipped, 464 passed, 481 total
  Snapshots:   166 passed, 166 total
  Time:        468.819 s
  ```
  The 17 skipped tests come from the demo `skip` lists in `components/table/__tests__/demo.test.ts` and `demo-extend.test.ts`, which `tests/shared/demoTest.tsx` turns into `test.skip`. The same lists are upstream. The new test is not skipped: it runs and passes, as the targeted run above shows.
- **ESLint**, `npx eslint components/table/hooks/useFilter/FilterDropdown.tsx components/table/__tests__/Table.filter.test.tsx`: exit 0, no output.
- **Biome**, `npx biome lint <same files>`: `Checked 2 files in 33ms. No fixes applied.`, exit 0.
- **tsc**, `npx tsc --noEmit` on the whole project: exit 0. It needed `NODE_OPTIONS=--max-old-space-size=6144`, because the default 2 GB heap ran out in this container.
- **Environment:** Node 22.22.0, npm 10.9.4, React 19.3.1 (jest setup), `npm install --ignore-scripts`. Run jest with `--no-cache`, as `npm test` does. Without it, the first run here hit a stale-transform error in `genStyleUtils`.

## Duplicate search (all states)

All searches were in `ant-design/ant-design`, open and closed, on 2026-10-30.

**Issues**
- "filterResetToDefaultFilteredValue clearFilters confirm": no match. The nearest hits are unrelated: #57957 (feature request: confirm/close on reset), #48580 (filterOnClose state), #32154.
- "Table custom filterDropdown clearFilters reset to defaultFilteredValue not applied": no match. Hits are #48580, #46583, #32154, #23182 (clearFilters and `currentDataSource`, 2020, closed) and #40435 (numeric `defaultFilteredValue` checkbox, closed).
- Chinese query "Table 自定义筛选 clearFilters 重置 默认筛选值 defaultFilteredValue": no match.

**PRs**
- `filterResetToDefaultFilteredValue`: #34353 and #34355, both closed, from 2022. These added the API.
- `clearFilters`: #57985, #51754, #51854, #34120, #35861, #34328, #19701 and #19205, all closed. None touches the confirm-with-reset-to-default path.
- All open PRs created since 2026-10-01 (66) were checked by title. Two open PRs touch the same files: #59735 changes `filterSearch(searchValue, …)` in `renderFilterItems`, and #59740 rewrites `getFilterData` in `useFilter/index.tsx`. I fetched both heads, and neither touches `onReset`. No textual overlap.

Candidates I checked and dropped because someone else already has them: Form `name={0}` label (#59678 / #59679), Table positional keys for columns without `key` (#59676 / #59677), DirectoryTree shift anchors (#59616 / #59617), Calendar header vs `validRange` (#59691 / #59692), TreeSelect `dropdownStyle` (#59736 / #59737), Form.List `noStyle` stale errors (#54119 / #59419).

## Notes and leads for later sessions (not part of this PR)

- **Doc mismatch, not fixed:** `components/table/demo/custom-filter-panel.md` says `clearFilters`' `confirm` and `closeDropdown` default to `true`. In the code (`onReset` default param) and in the `clearFilters should support params` test, both default to `false`. This could be a separate docs PR.
- **Upload, frozen list items crash picture list views. Unverified lead, no duplicate found:**
  - `components/upload/UploadList/index.tsx` (thumbnail effect) writes `file.thumbUrl` onto items without the `Object.isFrozen` guard that `Upload.tsx` uses elsewhere.
  - A frozen item (immer / RTK store) with `originFileObj` and `listType="picture-card"` should throw "object is not extensible".
  - Open Upload PRs to check for overlap: #59744, #59665.
- **InputNumber legacy `addonBefore`/`addonAfter`. Unverified lead:**
  - In `components/input-number/index.tsx`, `SpaceAddon` gets raw `props.disabled`/`props.variant`, and `Compact` gets no `size`.
  - So `ConfigProvider componentDisabled`, Form `disabled`, `bordered={false}` and `size` aren't reflected on the addon.
- **Transfer RTL `direction`. Low confidence:** `components/transfer/index.tsx` passes `direction` swapped under RTL, so `filterOption` gets `'right'` for the source list while `onSearch` gets `'left'`.
