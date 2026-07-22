## Why I Chose This Issue

I chose issue #25726 "chore: migrate from quickcheck to proptest for VRL Arbitrary impls" because it aligns with my experience in working on tools that generate "random" test cases and it aligns with my goal to do more work on repositories that span across more than a single repo/crate.

I'm interested in this because:
1. I've worked on smaller PRs in this repository before, and I think this will be a good marker of progression as it is a slightly more challenging task.
2. I've worked on projects related to randomized testing before (specifically, fuzzers), and I find this domain interesting.

From reading the issue thread, I understand the current state of quickecheck and proptest rely on different versions of the rand crate, which lead to two incompatible rand versions in the dependence tree, and the goal of this task is to remove quickcheck from VRL (upstream repo) and update Vector (the downstream repo) to mainly use proptest instead. 

I commented on the issue and asked a minor clarifying question to the maintainer. Since then, I have been given the go-ahead to work on this issue.

## Reproduction Process

### Environment Setup

Clone both repos side-by-side:

git clone https://github.com/vectordotdev/vrl.git

git clone https://github.com/vectordotdev/vector.git

First cargo build in Vector takes ~60 minutes cold. Subsequent builds are fast via incremental compilation.

Working branches:
- VRL: https://github.com/hjooh/vrl/tree/chore/migrate-quickcheck-to-proptest
- Vector: https://github.com/hjooh/vector/tree/chore/migrate-quickcheck-to-proptest

### Steps to Reproduce

1. In the Vector repo, run cargo tree -p vector-core --features generate-fixtures | grep rand
2. Expected: A single version of rand in the dependency tree
3. Actual: Two separate versions of rand appear — one pulled in by proptest, one by quickcheck via vrl's arbitrary feature

The chain: vector-core generate-fixtures feature → dep:quickcheck + vrl/generate-fixtures → vrl/arbitrary → dep:quickcheck → rand@X, while proptest simultaneously pulls in rand@Y.

### Solution Plan

Understand: Both VRL and Vector depend on quickcheck and proptest for property-based testing. Since these libraries pin to different versions of rand, the build graph ends up compiling two rand versions. VRL's arbitrary feature exposes quickcheck::Arbitrary for Value, KeyString, and BorrowedSegment as public API, which propagates quickcheck into Vector's production dep tree when the generate-fixtures feature is enabled.

**Match:** VRL already has a proptest feature providing proptest support for path types (OwnedSegment, OwnedValuePath, etc.) using proptest_derive. KeyString already derives proptest_derive::Arbitrary. The pattern for porting the remaining quickcheck impls is established in the same files.

**Plan:**

VRL PR (goes first):
1. Add proptest::Arbitrary for BorrowedSegment<'static> in src/path/borrowed.rs
2. Delete quickcheck::Arbitrary for KeyString in src/value/keystring.rs (proptest derive already present)
3. Delete quickcheck::Arbitrary for BorrowedSegment in src/path/borrowed.rs
4. Replace Value's quickcheck Arbitrary in src/value/value/arbitrary.rs with a proptest strategy
5. Rewrite quickcheck_value() test in src/value/value.rs as a proptest! block
6. Update Cargo.toml: remove dep:quickcheck from the arbitrary feature, update generate-fixtures to depend on proptest instead of arbitrary, drop quickcheck from optional dependencies

Vector PR (after VRL merges):
1. Update VRL workspace dep: features = ["arbitrary", ...] → features = ["proptest", ...]
2. Update vector-core generate-fixtures feature: drop dep:quickcheck, add dep:proptest, drop vrl/generate-fixtures
3. Remove quickcheck = { workspace = true, optional = true } from vector-core dependencies
4. Move arbitrary_impl module gate from cfg(any(test, feature = "generate-fixtures")) to cfg(test) only; fix the two VRL-dependent calls (ObjectMap::arbitrary(), Value::arbitrary(g)) with local implementations
5. Write lib/vector-core/src/event/proptest_strategies.rs with proptest strategies for Event, LogEvent, Metric, EventMetadata, Value, ObjectMap — JSON-safe floats, deterministic seeding
6. Widen metric/mod.rs gate to include generate-fixtures so existing proptest metric Arbitrary impls are available to the new strategies
7. Rewrite generate_fixtures.rs using proptest::test_runner::TestRunner with a fixed ChaCha seed
8. Run the binary, commit the regenerated 1024 fixture files in lib/codecs/tests/data/native_encoding/

**Review:** Check against Vector's CONTRIBUTING.md and .github/PULL_REQUEST_TEMPLATE.md. PR title follows Conventional Commits: chore(deps): migrate from quickcheck to proptest for VRL Arbitrary impls.

**Evaluate:**
- cargo tree -p vector-core --features generate-fixtures | grep rand shows a single rand version
- cargo tree -p vector-core --features generate-fixtures | grep quickcheck produces no output
- cargo test -p vector-core --features test passes
- cargo test -p codecs passes against the regenerated fixtures
- Running the generate-fixtures binary twice produces byte-identical output

##  Implementation Notes

### VRL: quickcheck to proptest migration
**What I built:**
- Created proptest.rs: contains new proptest strategies for Value with float_strategy() and datetime_strategy() helpers
- Updated keystring.rs to remove the manual quickcheck::Arbitrary impl and replaced with #[derive(proptest_derive::Arbitrary)]
- Updated value.rs with a proptest test (replacing a quickcheck test)
- Updated mod.rs to export the proptest module and deleted arbitrary.rs which contained the old quickcheck impl

**Challenges faced:**  
- Didn't realize that the new proptest module shadowed the name of the proptest crate (needed to use ::proptest::prelude::* to tell them apart)

**Commits:**
- 598377652: chore(value): migrate Arbitrary impls from quickcheck to proptest
- WIP PR linked [here](https://github.com/vectordotdev/vrl/pull/1864)

