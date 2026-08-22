# Launch Checklist — StuntListing Contract Guide

Pre-launch to-do list for shipping this as a public side project.
Interactive version: https://claude.ai/code/artifact/278d55cf-b959-40d8-a5a6-b0e6f047db72

The project is one static `index.html` (2,880 lines) plus two PDFs in `docs/`.
No build step, no backend. That's a good place to launch from — the work below
is mostly about the three things that appear to work and don't: the map key,
the submission forms, and how the link renders when someone shares it.

---

## 1. Google Maps — 7 items, all blocking

The Studio Zone map is the only feature that costs money and can be abused.

**What's there now:** the loader at `index.html:1850` hardcodes
`AIzaSyBFw…U17R8` and pulls in `libraries=geometry`. That key was committed in
`4e088b8` and is a string that circulates widely in public Maps examples.
First thing to establish: **does that key belong to a Cloud project you can log
into?** If not, there's no console, no quota, no restrictions, and no warning
when it stops resolving.

- [ ] **Create your own Google Cloud project and attach billing.** Maps needs a
      billing account even inside the free tier. Until the key traces back to a
      project you own, the map is borrowed infrastructure.
- [ ] **Enable exactly two APIs: Maps JavaScript API and Geocoding API.** The
      map, circles, and distance math come from Maps JavaScript. The address box
      is the one people miss — `new google.maps.Geocoder()` at `index.html:1814`
      bills against the **Geocoding API** even though it's called from the
      browser. Enable nothing else.
- [ ] **Restrict the key by HTTP referrer and by API.** A browser key is public
      by definition and that's fine. The referrer restriction is the *only*
      thing stopping another site from spending your quota.

      Application restriction  → HTTP referrers
        https://yourdomain.com/*
        https://*.yourdomain.com/*
        http://localhost:*/*
      API restriction          → Restrict key
        Maps JavaScript API
        Geocoding API

- [ ] **Cap the spend: budget alert + daily quota ceiling.** Geocoding bills per
      request past the free allowance. A public address field with no rate limit
      and no debounce is exactly the shape that produces a surprise invoice. A
      capped map that stops working beats an uncapped one that keeps going.
      Cheap mitigation in-page: debounce the input, skip the geocode for strings
      under ~5 characters.
- [ ] **Swap the key, never reuse it server-side.** Replace the literal at
      `index.html:1850`. Don't try to hide it — a Maps JS key can't be hidden and
      doesn't need to be. The rule: browser-only and referrer-locked forever.
      Anything server-side later gets its own separate restricted key.
- [ ] **Add a failure state to the map container.** If the Maps script fails —
      bad key, quota hit, blocked network — `#sz-map` at `index.html:1155` sits
      on "Loading map…" permanently. Add an `onerror` to the injected script tag
      that swaps in a plain-text fallback naming the three zone centers and the
      30-mile radius. The rule is still usable without a map.
- [ ] **Verify the three zone centers and radius against the current CBA.** The
      map outputs "INSIDE STUDIO ZONE" in green — a legal test people will act on
      when arguing for travel pay and per diem. The centers in `index.html:1777`
      read as Columbus Circle, Beverly & La Cienega, and downtown SF at 48,280 m
      each. Confirm each against the contract text and note the source.

**Not blocking, but decide before styling the map further:** `google.maps.Marker`
at `index.html:1825` is superseded by `AdvancedMarkerElement`, and the inline
`styles` array at `index.html:1786` is the legacy styling route (the current one
is a cloud-based Map ID). The two are mutually exclusive — pick a lane before
hand-tuning more colors.

---

## 2. Everything else that blocks launch — 5 items

Two of these are features that appear to work and don't. That's worse than a
missing feature, because someone spends real effort before finding out.

- [ ] **Story submissions go nowhere.** `submitRealTalk()` at `index.html:2266`
      and its SAG On Record twin write to `localStorage` and nothing else.
      Someone types out a real story about a pay dispute, hits submit, watches it
      render — and it never leaves their laptop. You never receive it.
      Two honest fixes: wire it to a real endpoint (Supabase, Formspree, even a
      Google Form) with moderation before publish, or relabel the panel as a
      private draft ("Draft your story here, then email info@stuntlisting.com")
      and ship the backend later.
- [ ] **Contract-idea voting is per-browser.** Votes persist to `ci-votes` at
      `index.html:2527`. "23 votes" means one person clicked 23 times on one
      machine, and clears when they clear site data. Publishing fake social proof
      on a site whose pitch is accuracy costs more than having no voting at all.
- [ ] **No share metadata and no favicon.** Not one `og:` tag, `twitter:` card,
      `meta description`, or favicon in the file. This tool spreads by being
      texted between performers and dropped into stunt Facebook groups — every
      one of those links currently renders as a naked grey URL. Highest return
      per minute on this list. Add `og:title`, `og:description`, `og:image`
      (1200×630), `twitter:card=summary_large_image`, a meta description, a
      canonical URL, and an inline SVG favicon.
- [ ] **Date-stamp the rate data and say who maintains it.** The disclaimer bar
      at `index.html:326` covers affiliation but not staleness. The rate finder
      switches contract cycles and the calculators emit specific dollar figures.
      Add a visible "Rates current as of [cycle] — verify against the CBA" line
      next to the rate tables, and calendar the update.
- [ ] **Confirm you can host the two PDFs in `docs/`.** 5.8 MB of Jeffords Rules
      (2024 and East 2021) served from your domain is redistribution, not linking.
      Confirm it's permitted or link out — linking out is also better for users,
      who then get the current revision rather than a snapshot frozen at your
      last commit.

---

## 3. Before launch — 11 items

A single static file with hash routing deploys almost anywhere for free, so this
part is genuinely short.

- [ ] **Pick a host and point a subdomain at it.** Cloudflare Pages, Netlify,
      Vercel, or GitHub Pages all serve this free with automatic HTTPS. Something
      like `contracts.stuntlisting.com` borrows trust from the main brand.
- [ ] **Confirm unknown paths fall back to `index.html`.** Navigation is
      hash-based (`index.html:2189`), so every real URL is `/#rates`,
      `/#studio-zones` etc., all served by the one file — deep links survive by
      default. Just check a stray `/rates` lands on the page, not a host 404.
- [ ] **Add privacy-friendly analytics.** Nothing is instrumented, so you'd
      launch blind to which of the twenty-odd sections get used. Plausible or
      Fathom need no cookie banner; GA4 drags consent along. First questions:
      which calculators get used, and where mobile visitors drop.
- [ ] **Test every calculator on a real phone.** This gets used standing on set,
      one-handed, on cell service. Layout has had a mobile pass (`8fa8193`); what
      needs checking is the interactions — time inputs, the Exhibit G flow, the
      stunt-driving checkboxes, and the map address field with a software
      keyboard covering half the screen.
- [ ] **Spot-check the calculators against known-correct scenarios.** Meal
      penalty, rest period, late payment, fittings, overtime, and the rate finder
      all output figures people may take into a conversation with production. Run
      three or four real past workdays through each and confirm against what
      actually got paid. This is the credibility of the whole site in one task.
- [ ] **Keyboard and focus pass.** Navigation runs on `onclick` attributes.
      Verify every section is keyboard-reachable, focus is visible against the
      dark ground, and the mobile nav closes without a mouse.
- [ ] **Make sure `info@stuntlisting.com` is monitored.** It's the only feedback
      channel in the build. On a site full of numbers, corrections are the most
      valuable mail you'll get.
- [ ] **Write a README and choose a license.** The repo has neither. README needs
      three things: what this is, that it isn't legal advice or SAG-AFTRA
      affiliated, and how to update rates each cycle — write that last one for
      yourself six months from now.
- [ ] **Re-read the disclaimer with a lawyer's eye.** One line covers
      non-affiliation. Given the site interprets contract language and computes
      penalties, consider: informational only, not legal advice, verify against
      the CBA, contact SAG-AFTRA for disputes. The union contacts are already in
      the build — point at them from the disclaimer.
- [ ] **Add a print stylesheet.** People will want the meal-penalty rules and
      rate tables on paper. A small `@media print` block that drops the nav,
      un-hides inactive sections, and switches to dark-on-white makes the whole
      guide printable in one pass.
- [ ] **Seed the community sections with real content before opening them.** Real
      Talk, SAG On Record, and Contract Ideas are what make this feel alive.
      Empty ones read as abandoned. Get four or five genuine stories in each —
      with permission — before sending the link anywhere.

---

## 4. First month after launch — 6 items

- [ ] **Watch Maps usage daily for the first week.** The billing console tells you
      within days whether the quota caps are sensible or the address box is
      getting hammered. Only part of the site with a cost curve.
- [ ] **Distribute where stunt performers already are.** Stunt Facebook groups,
      the SAG-AFTRA stunt committee, coordinators you know, the StuntListing list.
      A tool this specific spreads by word of mouth, not search.
- [ ] **Set a recurring reminder for the next rate cycle.** Highest-risk
      maintenance task and the easiest to forget. Calendar it at launch, with the
      procedure in the README so it's a twenty-minute job.
- [ ] **Route corrections into a visible changelog.** Performers *will* write in
      to correct a number. A dated list of what changed converts the site's
      biggest liability — being wrong about pay — into proof it's maintained.
- [ ] **Split `index.html` once you're editing it weekly.** 2,880 lines and
      225 KB in one file is fine for launch. When it becomes a weekly edit, pull
      rate data into JSON and calculators into their own script — the rate data
      especially, since that's what changes on a schedule.
- [ ] **Decide whether this stays standalone or folds into StuntListing.** A free
      utility that builds trust with exactly the audience the main product serves.
      Worth deciding deliberately in month two, once traffic is visible.

---

## Shortest path to launch

The map key and restrictions, relabel or wire up the two submission features,
add the share metadata. That's a weekend, and it's the difference between
shipping something solid and shipping something that quietly loses people's
contributions.
