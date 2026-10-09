# Dragon Con planner: the beta site

One tagged commit of [dragoncon-planner](https://github.com/KilgoreTrout853/dragoncon-planner), for a few friends, at https://kilgoretrout853.github.io/dragoncon-planner-beta/. The live site is https://kilgoretrout853.github.io/dragoncon-planner/ and publishes from that repository's `main`; the next site, https://kilgoretrout853.github.io/dragoncon-planner-next/, follows its `next` branch. Nothing here touches either.

There is no source code in this repository. `.github/workflows/deploy.yml`, "Deploy beta", is started by hand with a tag of the app's repository whose name starts `beta-`. It checks the app out at that tag's commit, builds it (`npm ci && npm run build`) and pushes the result to `gh-pages`. `deployed.txt` records the tag that is live and its commit.

## How it is pinned

The site shows the tag it was last deployed with until "Deploy beta" is run with another. There is no timer and no push trigger: a pull request merged into `next` does not move it, and neither does a new tag until the workflow is run with it. A branch's name, a bare sha or a tag that is not in the app's repository stops a run before anything is built.

The build is the app's own, with these set (the app's README, "The next site"; its DECISIONS #15 and #99):

- `DC_CHANNEL=beta`: the corner mark, and a worker cache and storage keys of the beta's own. The three sites share one origin, and the channel is what keeps the beta's picks and caches apart from the other two.
- `DC_BUILD`, the tag: the mark and the device readout under Settings, Advanced say which beta a screenshot came from.
- `DC_NOW=2026-09-01T10:00`: the page opens on the Tuesday before the con, the phone's own wall time, until a reader sets a time of their own.
- `DC_EMAIL=off`: no Keep your plan.
- `DC_SUPABASE_URL` and `DC_SUPABASE_KEY`, from this repository's variables: the dev project's address and its public key, the two values the next site has. They are variables and not secrets, since the build inlines both in a page anyone can read. Crews run against that project.

A tag from before the app read `DC_NOW` (its PR #117) would build at the real clock with the email step on, so the workflow stops such a build before it is published.

The deploy also points the page's four link-preview tags (`og:url`, `og:image`, `og:image:secure_url`, `twitter:image`) at this site: the app's page names the live site there, and a link to the beta would otherwise preview with the live site's image.

## Bringing the beta forward

Two steps, each by hand.

1. Tag the commit in the app's repository. The tag is a ref made on GitHub, so no working copy is touched:

   ```bash
   gh api repos/KilgoreTrout853/dragoncon-planner/git/refs -f ref=refs/tags/beta-1 -f sha=FULL_SHA_OF_THE_COMMIT
   ```

2. Run "Deploy beta" with the tag, from the Actions tab or:

   ```bash
   gh workflow run deploy.yml -R KilgoreTrout853/dragoncon-planner-beta -f tag=beta-1
   ```

Then read the build back: the page's `dc-build` meta and its corner mark name the tag, and `deployed.txt` holds the tag and its commit.

Friends keep their picks across a new beta: the storage keys carry the channel, not the build. A run for the tag already deployed publishes it again, which is how a build is redone; a run with an older tag takes the site back to it.
