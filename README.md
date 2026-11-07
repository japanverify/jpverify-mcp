# JPVerify MCP

<img src="https://www.jpverify.com/logo-512.png" alt="JPVerify" width="96" align="right">

Remote MCP server for Japanese business identity checks and Japanese document OCR: NTA corporate-number and qualified-invoice (T-number) verification, address normalization and validation, invoice / receipt / business-card OCR, supplier resolution, and EU VAT / Peppol counterparty checks.

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
| `jp_company_verify` | $0.005 | Verify a Japanese company by name, 13-digit corporate number (法人番号), or address against the NTA registry. |
| `jp_invoice_registration_verify` | $0.01 | Verify a Japanese qualified-invoice registration number (T + 13 digits) against the NTA invoice registry. |
| `jp_invoice_issuer_check` | $0.03 | Verify a T-number and compare the issuer name and address with Japan's NTA registries. |
| `jp_invoice_extract` | $0.01 | OCR a Japanese billing invoice (text, HTTPS file URL, or small Base64) with local date, tax, arithmetic and T-number checks; no registry lookup. |
| `jp_invoice_extract_and_verify` | $0.05 | OCR a Japanese billing invoice, validate it locally, and resolve the issuer against NTA registries. |
| `jp_receipt_extract` | $0.03 | OCR a Japanese receipt with local amount, tax and T-number checks; detected T-numbers are verified with the NTA registry. |
| `jp_business_card_extract` | $0.03 (default) / $0.10 (enrich) | Read a Japanese business card (front/back HTTPS image URL or small Base64); enrich=true adds a registry pre-check of the organization. |
| `jp_address_normalize` | $0.005 | Normalize a Japanese address into a canonical hierarchy with an English rendering. |
| `jp_address_validate` | $0.005 | Validate a Japanese address with parser, postal and geocoder evidence and return exact/partial/ambiguous/conflict/unverified. |
| `jp_supplier_resolve` | $0.01 | One coverage-aware supplier check combining company, qualified-invoice and address evidence. |
| `jp_counterparty_check` | $0.05 | Check a known EU VAT ID and/or Peppol participant: VAT validity, name match, and Peppol discoverability. |

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

## 日本語

日本企業の照合と日本語書類OCRのためのリモートMCPサーバーです。国税庁の法人番号・適格請求書（T番号）照合、住所の正規化・検証、請求書・領収書・名刺OCR、仕入先確認、EU VAT／Peppolの取引先確認に対応します。

- **エンドポイント（Streamable HTTP）：** `https://api.jpverify.com/mcp`（アカウント・APIキー不要）
- **決済：** x402（Base上のUSDC）。成功した呼び出しのみ課金
- `initialize`、`tools/list`、`jpverify_guide` は無料です。

| ツール | 料金（USDC） | 内容 |
|---|---|---|
| `jpverify_guide` | 無料 | ツール一覧・料金・入力例。質問に合うツールを推薦 |
| `jp_company_verify` | $0.005 | 会社名・法人番号・住所を国税庁の法人番号公表データと照合します。 |
| `jp_invoice_registration_verify` | $0.01 | 適格請求書発行事業者登録番号（T+13桁）を国税庁データと照合します。 |
| `jp_invoice_issuer_check` | $0.03 | T番号を検証し、発行者名・住所を国税庁データと比較します。 |
| `jp_invoice_extract` | $0.01 | 日本語請求書をOCRし、日付・税額・金額・T番号をローカル検証します（法人照合なし）。 |
| `jp_invoice_extract_and_verify` | $0.05 | 日本語請求書をOCR・ローカル検証し、発行者を国税庁データと照合します。 |
| `jp_receipt_extract` | $0.03 | 日本語領収書をOCRし、金額・税額・T番号を検証します。 |
| `jp_business_card_extract` | $0.03 (default) / $0.10 (enrich) | 日本語の名刺を読み取ります。enrich=trueで組織の法人照合も行います。 |
| `jp_address_normalize` | $0.005 | 日本の住所を正規化し、階層と英語表記を返します。 |
| `jp_address_validate` | $0.005 | 住所解析・郵便番号・ジオコーディング根拠で日本住所を検証します。 |
| `jp_supplier_resolve` | $0.01 | 会社・適格請求書・住所の根拠をまとめて仕入先を確認します。 |
| `jp_counterparty_check` | $0.05 | 既知のEU VAT IDやPeppol参加者IDの有効性・名称一致・到達性を確認します。 |

不正な入力は支払い前に拒否され、失敗した呼び出しは課金されません。照合結果は公開データに基づく事前確認であり、KYB、AML、制裁リスト審査、法務・税務上の助言ではありません。

## License

Documentation and example configuration in this repository: MIT. The JPVerify service is subject to its [Terms of Service](https://api.jpverify.com/terms) and [Privacy Policy](https://api.jpverify.com/privacy).
