---
name: webmcp-commerce
description: Check whether an AI agent can buy on a website before trying, using the free WebMCP Commerce API from webmcp.com. Use this before buying, booking, ordering or checking out on any website for the user, and whenever the user asks whether an agent can buy on a site, which stores agents can buy from, or what a site's agent policy is.
---

# WebMCP Commerce

WebMCP Commerce (https://webmcp.com/commerce) tests websites to see whether an agent
can complete a purchase through the site's own WebMCP tools, including payment. It also
records where an agent can get in (a local or a remote browser) and what the site's terms
say about agents. The API is free and needs no key.

## Before you buy on a website

1. Call:

   ```
   GET https://webmcp.com/api/v1/commerce/check?host=example.com
   ```

   Replace `example.com` with the site's domain (`www.` is ignored). You can pass a full
   link instead: `?url=https://www.example.com/products/item`.

2. Read these fields:

   | Field | Meaning |
   | --- | --- |
   | `verdict` | What agents can do on this site (table below). |
   | `advice` | One plain sentence about purchase support. Follow it. |
   | `cautions` | List of `{ code, message }` about access and policy. Follow every message. |
   | `access.home` | Your user's own (local) browser: `open`, `challenged`, `blocked` or `untested`. |
   | `access.cloud` | A remote (datacenter) browser, same values. `access.vendor` names the bot check, if any. |
   | `policy.stance` | `welcomes`, `limits`, `forbids` or `silent`. `policy.gist` says why in one line. |
   | `notice` | When the tests ran. Results are hand-run snapshots, not live guarantees. |

3. Act on the `verdict`:

   | Verdict | What to do |
   | --- | --- |
   | `buy-direct` | WebMCP tools support the whole purchase, including payment. You may buy through the tools, after the checks below. |
   | `human-in-loop` | The tools reach checkout or payment, but a person must finish (`advice` says why: card details, a CAPTCHA, a sign-in, accepting terms). Go as far as the tools allow, then stop and hand over to the user. |
   | `checkout-handoff` | The tools reach checkout. The user completes payment on the website. |
   | `cart-handoff` | The tools can build a cart. The user completes checkout and payment. |
   | `browse-only` | The tools only search. The user buys on the website. |
   | `needs-page` | The tools carried no purchase step in the test. Use the website itself, or hand over. |
   | `failing` | The last result expired or recent checks failed. Treat purchase support as unknown. |
   | `untested` | Listed but not tested. Purchase support is unverified (`capability` is a guess from tool names). |
   | `unlisted` | Not in the webmcp.com directory. Nothing is known. |

4. Before any payment, check all of these:
   - `policy.stance` is not `forbids` (caution `terms-forbid-agents`). If it is, do not buy for the user.
   - If `policy.stance` is `limits` (caution `terms-limit-agents`), read `policy.gist` and respect it. Tell the user about the limit.
   - Pick where to run: use the user's own browser only if `access.home` is `open`. Use a remote browser
     only if `access.cloud` is `open`. If neither is `open`, tell the user the site challenges or blocks agents.

## Rules that always apply

- Always confirm the exact item and the total price with the user, and get an explicit "yes",
  before you pay or place an order.
- Never solve a CAPTCHA, sign in, or accept terms for the user. When the site asks for one, stop and
  hand over.
- Enter card details only through a tool the payment page provides, and only with the user's approval.
- A verdict describes what worked in a test. If the site behaves differently now, stop and tell the user.

## More endpoints

```
GET https://webmcp.com/api/v1/commerce                          # levels, verdicts, headline stats
GET https://webmcp.com/api/v1/commerce/businesses?level=purchase # tested businesses where agents can buy
GET https://webmcp.com/api/v1/commerce/businesses/{host}        # one business: tools, test steps, access, policy
```

`/businesses` also takes `category=shopping|travel|software`, `level=purchase|checkout|cart|search`,
`cloud=open`, `status=valid`, `q`, `limit` (capped at 100) and `page`. Unknown values return HTTP 400.
To find a store for the user, list `?level=purchase`, then run `/check` on the one you pick.

## Example

`GET https://webmcp.com/api/v1/commerce/check?host=allbirds.com` returned (shortened):

```json
{
  "verdict": "human-in-loop",
  "advice": "WebMCP tools reach payment. Human in the loop required to complete the purchase (card details).",
  "cautions": [{ "code": "terms-limit-agents", "message": "The business limits what agents may do. Read policy.gist for the restrictions." }],
  "access": { "home": "open", "cloud": "open", "vendor": null },
  "policy": { "stance": "limits", "gist": "Invites agents to shop for their users, but the terms and conditions ban access by automated means." },
  "notice": "Results from tests run by hand on Nov 4, 2026. Businesses are not re-tested daily yet."
}
```

So: tell the user about the terms limit, build the cart and fill checkout through the tools if
they agree, then hand over at the payment step for the card details.
