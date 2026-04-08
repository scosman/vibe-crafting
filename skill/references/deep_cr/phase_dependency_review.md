# Dependency Review

## What to Look For

### License Compatibility
- Check the license of every newly added dependency
- GPL, AGPL, LGPL, or any copyleft license is a **critical** issue — these licenses can require the entire project to be open-sourced
- LGPL is acceptable only if the dependency is dynamically linked (not bundled/compiled in)
- Dual-licensed packages: verify the project is using the permissive license option
- Transitive dependencies also matter — a permissively licensed package that depends on a GPL package still creates a licensing problem

### Dependency Reputation and Maintenance
- Is the package actively maintained? Check last commit date, open issues, release cadence
- Does it have meaningful adoption? (downloads, stars, known users — not vanity metrics, but signals of real usage)
- Is the author/org reputable? Single-maintainer packages with no community are a supply chain risk
- Check for signs of abandonment: years-old open PRs, unanswered issues, deprecated notices
- Typosquatting risk: is the package name very similar to a popular package? (`lodash` vs `lodashs`, `colors` vs `colour`)

### Version Pinning
- Dependencies should be pinned to specific versions or tight ranges (not `*` or `latest`)
- Lock files (package-lock.json, yarn.lock, poetry.lock, Cargo.lock) are updated and committed
- Version ranges should be intentional: `^1.2.3` allows minor updates, `~1.2.3` allows patches only — the choice should match the dependency's stability
- Pre-release versions (alpha, beta, rc) should not be used in production unless there's a documented reason

### Duplicate Functionality
- Does the new dependency overlap with something already in the project? (Adding `axios` when `fetch` wrapper already exists, adding `lodash` when utility functions exist)
- Could the needed functionality be achieved with the standard library or an existing dependency?
- Is the new dependency worth the added weight for the functionality used? (Adding a full utility library for one function)

### Security Advisories
- Run the language-specific audit tool (`npm audit`, `pip audit`, `cargo audit`, `bundle audit`)
- Check if the specific version being added has known vulnerabilities
- If vulnerabilities exist: are they relevant to how the dependency is used? (A vulnerability in a feature the project doesn't use is lower severity)

### Size Impact
- For frontend/client-side projects: check the bundle size impact of new dependencies
- Tree-shakeable? If the project uses only 1 function from a package, does the bundler include only that function or the entire package?
- Are there lighter alternatives that provide the same functionality?
- Transitive dependency count: adding one package that pulls in 50 sub-dependencies is a maintenance and security surface area concern

### Transitive Dependencies
- Review what the new dependency itself depends on
- Deep dependency trees increase supply chain attack surface
- Conflicting versions of shared transitive dependencies can cause build or runtime issues
- Transitive dependencies with known vulnerabilities are still vulnerabilities

## Common Issues

- **Adding a dependency for trivial functionality**: Pulling in a package to do something that's 5 lines of code. The maintenance and security cost of the dependency exceeds the cost of writing it.
- **Unpinned versions in production**: Using `*`, `latest`, or very wide ranges. A breaking change or malicious version can silently break the build.
- **Missing lock file updates**: Package manifest updated but lock file not regenerated. Builds may get different versions than development.
- **Abandoned dependencies**: Package hasn't been updated in 2+ years, has dozens of open security issues, and is the only option anyone can find. Consider forking or finding an alternative.
- **Frontend bundle bloat**: Adding moment.js (300KB) when date-fns or Temporal API would suffice. Or importing all of lodash for `_.get`.
- **Conflicting dependencies**: Two packages require incompatible versions of a shared dependency. May cause build failures or subtle runtime bugs.
- **Dev dependency in production**: Test frameworks, linters, or build tools listed as production dependencies. Increases deployment size unnecessarily.
- **Ignoring audit results**: `npm audit` shows 15 high-severity issues but nobody addresses them. Audit fatigue is real, but ignoring audit results entirely is worse.

## Severity Guidance

**Critical:**
- GPL, AGPL, or copyleft licensed dependency (or transitive dependency) added
- Dependency with a known critical CVE that's relevant to how it's used
- Typosquatting or obviously suspicious package (malware vector)
- Unpinned dependency version in production configuration

**Moderate:**
- Dependency that duplicates existing functionality in the project
- Abandoned dependency (no updates in 2+ years, unaddressed security issues)
- Missing lock file update for newly added dependency
- Dev dependency incorrectly listed as production dependency
- Dependency with known moderate vulnerabilities

**Mild:**
- Large dependency that could be replaced with a lighter alternative
- Slightly wide version range that could be tightened
- Deep transitive dependency tree (concern but not immediate issue)
- Missing justification comment for non-obvious dependency choice
