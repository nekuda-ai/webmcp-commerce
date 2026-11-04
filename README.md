# WebMCP Commerce skill

An agent skill that checks whether an AI agent can buy on a website before it tries.
It uses the [WebMCP Commerce](https://webmcp.com/commerce) API from webmcp.com. The API
is free and needs no key.

For any website, the API answers three questions:

- **Can an agent buy here?** Do the site's own WebMCP tools carry a purchase all the way
  through payment, or does a person need to step in (card details, a CAPTCHA, a sign-in)?
- **Will the agent get in?** Results from a local browser and from a remote (datacenter) browser.
- **Is it allowed?** The site's agent policy, from its robots.txt, llms.txt and terms.

## Install

```sh
npx skills add nekuda-ai/webmcp-commerce
```

Works with Claude Code, Codex, Cursor and the other agents the
[skills](https://www.npmjs.com/package/skills) CLI supports.

## What the agent does

Before buying, booking or checking out on a website, the agent calls

```
GET https://webmcp.com/api/v1/commerce/check?host=example.com
```

and follows the `verdict`, `advice`, `cautions`, `access` and `policy` fields. It always
confirms the item and the total with you before paying, and hands over to you for CAPTCHAs,
sign-ins and terms. See [`skills/webmcp-commerce/SKILL.md`](skills/webmcp-commerce/SKILL.md).

Results come from tests run by hand; each response's `notice` says when.

## Privacy

Before each purchase, the agent sends the website's domain (for example `allbirds.com`)
to webmcp.com, so webmcp.com can see which shops your agent checks. The request carries
only the domain: no page or product links and no personal details.

## License

MIT. Made by [nekuda](https://nekuda.ai).
