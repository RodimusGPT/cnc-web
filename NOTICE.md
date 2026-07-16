# Notices and provenance

This standalone browser product was split on 2026-07-16 from the
`browser-port` work developed in the RodimusGPT/Vanilla-Conquer fork.
Browser-port changes began on 2026-07-10 from Vanilla Conquer commit
`7f351dae`, which incorporates source released by Electronic Arts from the
Command & Conquer Remastered Collection. The extraction preserves the browser
checkpoints `cffd540` and `7b7fb83` as provenance; it intentionally starts
fresh history rather than copying unrelated upstream blobs.

The exact engine source used for a build is the commit-pinned
`vendor/vanilla-conquer` submodule. Until the reusable hosted-engine changes
are accepted upstream, that gitlink points to the small
RodimusGPT/Vanilla-Conquer fork branch containing those changes.

The original source, this modification, and distributed WebAssembly object code
are governed by the GNU General Public License version 3 and the additional
terms in [License.txt](License.txt). Modified versions must be identified as
modified, must preserve the applicable notices, and receive no Electronic Arts
trademark or publicity rights.

Electronic Arts, Command & Conquer, Tiberian Dawn, Red Alert, and related marks
are trademarks of their respective owners. The project uses a neutral working
identity and makes no claim of affiliation.

**EA has not endorsed and does not support this product.**

EA game content is not part of the source-code license and is never committed
to this repository. A free, noncommercial deployment may provide a separately
generated, hash-verified package derived from the Tiberian Dawn freeware
release under EA's current modding guidelines. That release package excludes
C&amp;C music and movies. Optional Remastered-derived packages remain local to
the user who creates and imports them.

Upstream sources:

- Electronic Arts source release:
  https://github.com/electronicarts/CnC_Remastered_Collection
- Vanilla Conquer portable engine:
  https://github.com/TheAssemblyArmada/Vanilla-Conquer
- This modified browser source:
  https://github.com/RodimusGPT/cnc-web
- Temporary hosted-engine fork:
  https://github.com/RodimusGPT/Vanilla-Conquer
- EA Command & Conquer franchise modding guidelines:
  https://www.ea.com/games/command-and-conquer/command-and-conquer-remastered/news/modding-faq
- OpenRA freeware package provenance and legal notice:
  https://www.openra.net/legal/
