---
summary: "Homebrew Cask release steps for CodexBar (Sparkle-disabled builds)."
read_when:
  - Publishing a CodexBar release via Homebrew
  - Updating the Homebrew tap cask definition
---

# CodexBar Homebrew Release Playbook

Homebrew is for the UI app via Cask. When installed via Homebrew, CodexBar disables Sparkle and shows a "update via brew" hint in About.

## Prereqs
- Homebrew installed.
- Access to the tap repo: `../homebrew-tap`.

## 1) Release CodexBar normally
Follow `docs/RELEASING.md` to publish `CodexBar-<version>.zip` to GitHub Releases.

## 2) Update the Homebrew tap cask
In `../homebrew-tap`, add/update the cask at `Casks/codexbar.rb`:
- `url` points at the GitHub release asset: `.../releases/download/v<version>/CodexBar-<version>.zip`
- Update `sha256` to match that zip.
- Keep `depends_on arch: :arm64` and `depends_on macos: ">= :sonoma"` (CodexBar is macOS 14+).

## 2b) Update the Homebrew tap formula (Linux CLI)
In `../homebrew-tap`, add/update the formula at `Formula/codexbar.rb`:
- `url` points at the GitHub release assets:
  - `.../releases/download/v<version>/CodexBarCLI-v<version>-linux-aarch64.tar.gz`
  - `.../releases/download/v<version>/CodexBarCLI-v<version>-linux-x86_64.tar.gz`
- Update both `sha256` values to match those tarballs.

## 3) Verify install
```sh
brew uninstall --cask codexbar || true
brew untap steipete/tap || true
brew tap steipete/tap
brew install --cask steipete/tap/codexbar
open -a CodexBar
```

## 4) Push tap changes
Commit + push in the tap repo.

---

## Build-from-source formula (vshuraeff/tap)

The `vshuraeff/tap` hosts a build-from-source formula at `Formula/codexbar.rb` that supports both Intel and Apple Silicon Macs.

### Updating the formula after a new release

1. Tag the release in the fork:
   ```bash
   git tag v<version>
   git push origin v<version>
   ```

2. Get the commit SHA for the tag:
   ```bash
   git rev-parse v<version>
   ```

3. Update `tag:` and `revision:` in `homebrew-tap/Formula/codexbar.rb`:
   ```ruby
   url "https://github.com/vshuraeff/CodexBar.git",
       tag:      "v<version>",
       revision: "<full-sha>"
   ```

4. Test the formula:
   ```bash
   brew uninstall codexbar || true
   brew install --verbose vshuraeff/tap/codexbar
   codexbar --help
   open "$(brew --prefix)/opt/codexbar/CodexBar.app"
   ```

5. Commit and push the tap:
   ```bash
   cd /path/to/homebrew-tap
   git add Formula/codexbar.rb
   git commit -m "codexbar: update to v<version>"
   git push origin main
   ```
