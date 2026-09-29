# Changelog

Format: **v2**

<!--
Latest on top. One entry per release, written by /scope done.

The entry REFERENCES the feature document — it never describes the feature. The title is
transcribed verbatim from the document's `title` frontmatter field; the agent does not compose
prose here. The description lives in the archived document under scope/done/.

Entry format (v2, release tagging on):
- **DATE** · `RELEASE` · `ID` · Title · [`filename`](scope/done/filename)
  - Customer: release_note, verbatim

The `Customer:` line is there only when the document carries a release_note. A public changelog
page reads the entries that have one and ignores the rest.

The release is `v<major>.<minor>.<patch>`. F###: transcribed from the document's `release:`, the
second digit being the feature number, so entries need not be in number order. Q###: the highest
existing tag with the third digit raised by one.

Format v1 (release tagging off) is the same line without the release and without a customer line:
- **DATE** · `ID` · Title · [`filename`](scope/done/filename)

The ID is the link to the commit: every commit carries it as its conventional-commit scope, so
`git log --grep="(SP-F012)"` finds the change. No SHA is recorded here — it would have to be
written after the commit that contains this line, which would cost a second commit per feature.
The ID survives rebases and squashes; a SHA does not.

F### came from AxiomCore, Q### was drafted locally — so the origin of every change is readable
at a glance.
-->

- **2026-08-04** · `v0.12.1` · `SP-Q001` · Change card border colour · [`sp-q001-card-border.md`](scope/done/sp-q001-card-border.md)
- **2026-08-03** · `v0.12.0` · `SP-F012` · Favorites page · [`sp-01-f012-favorites.md`](scope/done/sp-01-f012-favorites.md)
  - Customer: Mark a speech as a favorite and find it again on its own page.
