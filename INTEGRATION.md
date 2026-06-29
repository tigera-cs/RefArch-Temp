# Reference Architecture pages — integration guide for webdev

Three interactive reference-architecture diagrams. This note covers how to put them on
tigera.io with the least effort and zero CSS conflict, what is editable where, and how
copy changes get made.

## The three files

| Page | Hosted (always current) | File in repo |
|------|------|------|
| Architecture | https://tigera-cs.github.io/RefArch-Temp/Calico_Architecture_v1.html | `Calico_Architecture_v1.html` |
| Envision | https://tigera-cs.github.io/RefArch-Temp/Calico_Envision_v1.html | `Calico_Envision_v1.html` |
| Unified Platform | https://tigera-cs.github.io/RefArch-Temp/Calico_Unified%20Platform_v1.html | `Calico_Unified Platform_v1.html` |

Each is a single self-contained HTML file: inline CSS + JS, images embedded as data URIs,
no build step, no external dependencies, no network calls. The logic is finished and tested.

## Recommended approach: iframe embed

Drop each file into a page region via an iframe. This is the fast path (your 4-6h estimate)
and it removes the integration friction that drove the larger estimate, because:

- **Zero CSS conflict, both directions.** An iframe is its own document, so the diagram's
  dark theme cannot leak into the WordPress theme and the theme's global CSS cannot break
  the diagram. No scoping or namespacing work needed.
- **It now fills any container.** The diagrams size themselves to the iframe, not the
  browser window, so they look right at whatever width/height you give the iframe (verified
  inside a normal content column with header/footer above and below).

Suggested markup (responsive, keeps the landscape aspect):

```html
<div style="max-width:1180px;margin:0 auto;">
  <div style="position:relative;width:100%;aspect-ratio:16/9;">
    <iframe
      src="https://tigera-cs.github.io/RefArch-Temp/Calico_Unified%20Platform_v1.html"
      title="Calico Unified Platform"
      loading="lazy"
      style="position:absolute;inset:0;width:100%;height:100%;border:0;border-radius:12px;"
    ></iframe>
  </div>
</div>
```

Notes:
- `aspect-ratio:16/9` works well for all three. If you prefer a fixed height, ~640px on
  desktop is a good baseline. The content scales to whatever you set, it will not clip.
- Point `src` at the hosted GitHub Pages URL above (always the current version), or at a
  copy you host. If you host your own copy, just re-upload the file when we send an update.

## Alternative: inline ("real page")

If you want the diagram markup native in the page (e.g. for SEO of the diagram internals),
it can be inlined into a page template. Two things to know:

1. The scaling is now **container-aware**: it measures the element it sits in, so give that
   wrapper a width and height (or the aspect-ratio box above) and it fills it.
2. The diagram's CSS would then share the page, so it needs to be **scoped** so generic class
   names (`.card`, `.slab`, etc.) and the `html,body` reset do not collide with the theme.
   We have not scoped the files for inline use yet, to avoid churn on the working versions.
   If you decide on the inline route, tell us and we will hand you a scoped build (all rules
   namespaced under one root) so it drops in cleanly. That keeps your side to a thin template
   rather than a CSS-integration fight.

On SEO: the only thing an iframe hides from crawlers is the text *inside* the diagram (SVG
labels, wizard options). The page title, subtitle, and any surrounding WordPress copy stay on
the real page and crawlable. For an interactive tool, that diagram-internal text carries
little SEO weight, so the iframe's SEO cost is small.

## What is editable, and where

- **Page title + subtitle/intro** — editable in WordPress via ACF, as you noted.
- **Everything inside the tool** (diagram labels, tier tabs, wizard questions, layer content,
  capability builders, click-popups, walkthrough steps) lives in the component's code.

Making that in-tool text editable through the WP admin would mean externalizing hundreds of
strings into ACF fields and rewiring the JS, which is not worth it. You do **not** need to do
that. Those strings are edited directly in the source by our side and redeployed in minutes
(GitHub Pages), which is how all the recent content updates were made. So copy changes are
fast without any CMS rebuild:

> Copy-change workflow: send us the string(s) to change → we edit the file and redeploy →
> the hosted URL (and therefore the iframe) updates. If you host your own copy, we send you
> the updated file to re-upload.

If there is one specific section you do want editable in WP, point it out and that single
part can be wired to ACF without rebuilding the whole component.

## Mobile

These are tuned for desktop today. A responsive pass (so they reflow on phones/tablets) is a
separate piece of work that we can do on our side and fold into the same files, so it travels
with them wherever they are hosted. Flag if/when you want it prioritized.

## Summary

- Fastest, cleanest, zero-conflict: **iframe** the hosted URLs (snippet above). The files are
  ready for this now.
- Want it inline for SEO of the diagram internals: doable, ask us for a CSS-scoped build first.
- Title/subtitle: ACF. All other copy: we edit the code and redeploy, minutes, no CMS rebuild.
- Mobile: available as a follow-up, done on our side.
