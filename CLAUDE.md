# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file browser invoice tool: `invoices.html`. No build step, no dependencies, no package manager. Open the file directly in a browser to run it.

```bash
open invoices.html
```

## Architecture

Everything lives in `invoices.html` — inline `<style>`, HTML structure, and a `<script>` block. No bundler, framework, or external file.

### Key state variables

- `invoices` — array of invoice objects (mock data, lives only in memory during the session)
- `selectedId` — id of the currently selected invoice
- `draft` — working copy (clone) of the selected invoice being edited
- `nextSeq` — counter for auto-generated invoice numbers (`INV-YYYY-NNNN`)

### Data model (invoice object)

```js
{
  id, number, date, dueDate, paymentTerms, poNumber,
  status,           // 'draft' | 'sent' | 'paid' | 'overdue'
  currency,         // 'USD' | 'EUR' | 'GBP' | 'MXN' | 'ARS' | 'BRL'
  issuer: { name, address, taxId },  // persisted in localStorage
  client: { name, address, taxId },
  shipTo,
  items: [{ description, quantity, unitPrice }],
  notes, terms,
  discountPercent, taxRate, amountPaid
}
```

### Multi-currency

`CURRENCIES` maps each code to `{ symbol, locale }`. `formatCurrency(value, currency)` uses `Intl.NumberFormat` with the invoice's currency code.

### Totals formula

```
subtotal       = Σ (quantity × unitPrice)
discountAmount = subtotal × discountPercent / 100
taxableBase    = subtotal − discountAmount
taxAmount      = taxableBase × taxRate / 100
total          = taxableBase + taxAmount
balanceDue     = total − amountPaid
```

### UX behaviors

- Changing `paymentTerms` to "Net N" auto-fills `dueDate` (date + N days).
- Issuer fields are saved to `localStorage('invoiceIssuer')` on every keystroke and reloaded on new invoices.
- Toast messages auto-dismiss after 3 seconds.
- `window.print()` triggers a print-optimized layout (`@media print`) that hides the sidebar and action buttons, showing only the invoice document.

## Preferences

- Be concise. Lead with the answer or action.
- Do not add comments, docstrings, or type annotations to code that wasn't changed.
- Do not refactor or clean up code beyond what was explicitly requested.
- Do not create files unless necessary. Prefer editing existing files.

## Git workflow

Commit and push to GitHub after every meaningful change:

```bash
git add invoices.html
git commit -m "short imperative summary"
git push
```

Remote: `https://github.com/alevasquezv/invoice-generator` (branch `main`).
