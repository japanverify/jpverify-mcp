# JPVerify MCP

[![mcptry](https://mcptry.com/badge/com.jpverify/jpverify)](https://mcptry.com/servers/com.jpverify/jpverify)

<img src="https://www.jpverify.com/logo-512.png" alt="JPVerify" width="96" align="right">

Remote MCP server for Japanese business identity checks: identify a Japanese company from an English or brand name, English address, website domain or identifier, NTA corporate-number and qualified-invoice (T-number) verification, invoice issuer checks (registration on the invoice date, supplier master data), address normalization and validation, and supplier resolution. JPVerify does not read files: send the fields you read. Japan only.

- **Endpoint (Streamable HTTP):** `https://api.jpverify.com/mcp` — no account, no API key
- **Official MCP Registry:** `com.jpverify/jpverify`
- **Payment:** [x402](https://www.x402.org/) — USDC on Base mainnet (`eip155:8453`), per successful call
- **Website:** https://www.jpverify.com · **Contact:** service@jpverify.com

This repository holds documentation and connection examples only. The service itself is operated by DSテクニック at the endpoint above.

[日本語](#日本語)

## Connect

Claude Code:

```bash
claude mcp add --transport http jpverify https://api.jpverify.com/mcp
```

Generic `mcp.json` (Cursor, Claude Desktop with remote support, and other MCP clients):

```json
{
  "mcpServers": {
    "jpverify": { "type": "http", "url": "https://api.jpverify.com/mcp" }
  }
}
```

`initialize`, `tools/list` and the `jpverify_guide` tool are free, so you can inspect every tool and its input schema before paying for anything.

## Tools

| Tool | Price (USDC) | What it does |
|---|---|---|
| `jpverify_guide` | free | Tool catalog, prices, example inputs; recommends a tool for a question |
| `jp_company_resolve` | $0.25 | Identify a Japanese company from an English, brand or romanized name (with an English address, postal code or prefecture), a website or e-mail domain, T-number, corporate number, LEI or securities code; returns the legally binding Japanese name, kana, English name with its source, status and qualified-invoice registration, in English. |
| `jp_supplier_resolve` | $0.10 | One coverage-aware supplier check combining company, qualified-invoice and address evidence; each check reports English address completeness. |
| `jp_invoice_issuer_check` | $0.05 | Check a T-number against the issuer name on an invoice, the address (optional), the invoice date (optional: registered on that date, not cancelled or expired) and your supplier master number (optional: catches invoices from another company). JPVerify does not read files. |
| `jp_invoice_registration_verify` | $0.01 | Verify a Japanese qualified-invoice registration number (T + 13 digits) against the NTA invoice registry. |
| `jp_company_verify` | $0.01 | Verify a Japanese company by its registered (Japanese) name or 13-digit corporate number (法人番号) against the NTA registry; an address is comparison evidence only. An English name that is not found returns `english_name_not_in_registry_index` — use `jp_company_resolve` for English names. |
| `jp_address_normalize` | $0.005 | Normalize a Japanese address into a canonical hierarchy (prefecture to room) with the confirmed match level and an English rendering from Japan Post romanized data. |
| `jp_address_validate` | $0.005 | Validate a Japanese address with parser, postal and geocoder evidence and return exact/partial/ambiguous/conflict/unverified. |

`jp_company_resolve` was called `jp_entity_resolve` before 2026-11-10; the old name still works but is no longer listed.

All tools are read-only (`readOnlyHint: true`). Registry results are a public-data pre-check, not KYB, AML, sanctions screening, or legal or tax advice.

## How payment works

1. Call a paid tool without payment. The result is an x402 `PaymentRequired` object (`isError: true`, `structuredContent.accepts[]` with the exact USDC amount, `payTo` and network).
2. Sign the payment with an x402 client and call the same tool again with the payload in `params._meta["x402/payment"]`.
3. The tool runs; payment is settled only after a successful result. The settlement receipt is returned in `_meta["x402/payment-response"]`.

Invalid input is rejected before any payment is requested, and failed calls are not charged. MCP clients without an x402 wallet can still use `jpverify_guide` and the HTTP API documentation.

Example tool call:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"jp_invoice_registration_verify","arguments":{"registration_number":"T1013301046627"}}}
```

## Other entry points

- **A2A agent (v0.3):** AgentCard `https://api.jpverify.com/.well-known/agent-card.json`, JSON-RPC `https://api.jpverify.com/a2a` (`message/send`, `tasks/get`), payments via the a2a-x402 extension
- **HTTP API:** OpenAPI `https://api.jpverify.com/openapi.json`, x402 catalog `https://api.jpverify.com/.well-known/x402.json`, agent guide `https://api.jpverify.com/skill.md`
- Batch verification, async batch jobs, JP PINT validation, FSA regulated-entity checks and EDINET filings are available over the HTTP API only.

## Complementary checks

English addresses carry `completeness` (`complete`, `partial`, `unavailable`). A partial English address (for example, a building name that cannot be romanized) never replaces the Japanese `address`; use the Japanese original for mail and records.

JPVerify confirms who a Japanese counterparty is (corporate number, T-number, address). It does not screen sanctions lists. If your onboarding flow also needs sanctions screening, run it as a separate step on the resolved name:

- Use the **official English or romanized name** from the corporate-number record (`jp_company_resolve` returns it in `name_en`, with its source). Most sanctions lists are matched on Latin-script names and aliases, so Japanese script often finds nothing even when an entry exists.
- Check official sources such as [OFAC Sanctions List Search](https://sanctionssearch.ofac.treas.gov/), or a screening tool. One that runs on the same x402 rail (USDC on Base) is AgentReader's [`/sanctions-batch`](https://github.com/grokbot1233/agentreader#pairs-well-with), which checks OFAC, UK, UN and EU lists and has a free preview.

Third-party services are listed for convenience only. JPVerify is not affiliated with them and does not endorse or verify their results. Confirm any potential match with the issuing authority.

## 日本語

日本企業の照合のためのリモートMCPサーバーです。英語名や識別番号からの日本法人の特定、国税庁の法人番号・適格請求書（T番号）照合、請求書の発行者確認（請求書日付時点の登録・仕入先マスター照合）、住所の正規化・検証、仕入先確認に対応します。ファイルの読み取りは行いません。日本専用です。

- **エンドポイント（Streamable HTTP）：** `https://api.jpverify.com/mcp`（アカウント・APIキー不要）
- **決済：** x402（Base上のUSDC）。成功した呼び出しのみ課金
- `initialize`、`tools/list`、`jpverify_guide` は無料です。

| ツール | 料金（USDC） | 内容 |
|---|---|---|
| `jpverify_guide` | 無料 | ツール一覧・料金・入力例。質問に合うツールを推薦 |
| `jp_company_resolve` | $0.25 | 英語名・ブランド名・ローマ字（英語住所・郵便番号・都道府県付き）、Webサイト／メールのドメイン、T番号、法人番号、LEI、証券コードから日本の法人を特定し、正式な日本語商号・読み・出典付き英語名・状態・適格請求書登録を英語で返します。 |
| `jp_supplier_resolve` | $0.10 | 会社・適格請求書・住所の根拠をまとめて仕入先を確認します。英語住所の完全性も返します。 |
| `jp_invoice_issuer_check` | $0.05 | T番号を請求書の発行者名・住所（任意）・日付（任意。その日に登録済みで取消・失効前か）・仕入先マスターの番号（任意）と照合します。ファイルの読み取りは行いません。 |
| `jp_invoice_registration_verify` | $0.01 | 適格請求書発行事業者登録番号（T+13桁）を国税庁データと照合します。 |
| `jp_company_verify` | $0.01 | 登記上の会社名（日本語）または法人番号を国税庁の法人番号公表データと照合します。住所は比較用の根拠です。英語名は `jp_company_resolve` を使用してください。 |
| `jp_address_normalize` | $0.005 | 日本の住所を正規化し、都道府県から部屋番号までの階層・照合レベルと、日本郵便のローマ字データによる英語表記を返します。 |
| `jp_address_validate` | $0.005 | 住所解析・郵便番号・ジオコーディング根拠で日本住所を検証します。 |

不正な入力は支払い前に拒否され、失敗した呼び出しは課金されません。照合結果は公開データに基づく事前確認であり、KYB、AML、制裁リスト審査、法務・税務上の助言ではありません。

制裁リスト審査が必要な場合は、法人番号データの英語表記またはローマ字表記の名称を使い、別の手順として実施してください（多くの制裁リストはラテン文字の名称で照合されます）。審査先の例は英語版の [Complementary checks](#complementary-checks) を参照してください。第三者サービスは参考として掲載しており、JPVerifyとの提携や推奨を意味するものではありません。

## License

Documentation and example configuration in this repository: MIT. The JPVerify service is subject to its [Terms of Service](https://api.jpverify.com/terms) and [Privacy Policy](https://api.jpverify.com/privacy).
