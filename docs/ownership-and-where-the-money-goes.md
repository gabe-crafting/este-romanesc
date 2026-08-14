# Foreign ownership and where the money actually goes

Why a Romanian company owned by a foreign group still pays Romanian tax, what leaves the
country and what doesn't, and what that means for how the app words its answers.

> General explanation of how the system works, not tax advice. Rates change — the 2025
> fiscal package moved several of them. Verify before quoting any figure in the product.

## The principle everything rests on

**A company is taxed where it is resident, not where its owner lives.**

OMV Petrom SA is a Romanian legal person with a Romanian CUI, registered at ONRC. That OMV
Aktiengesellschaft in Vienna holds the majority of its shares does not move its tax residence
to Austria. Profit earned in Romania is taxed in Romania — **16% corporate income tax** —
before a single euro is available to send anywhere.

This is the answer to the core of your question: ownership and taxation are separate
questions. Foreign ownership does not mean the money bypasses Romania.

## What stays in Romania regardless of who owns it

- **Corporate income tax** — 16% on the Romanian entity's profit.
- **Payroll taxes and social contributions** — income tax, pension (CAS), health (CASS) and
  the employer's labour contribution. For a large employer these **dwarf the profit tax**, and
  they are entirely domestic.
- **VAT** on domestic sales — economically borne by the consumer, but state revenue.
- **Excise duties** — on fuel, alcohol, tobacco. Large for an energy company.
- **Royalties (`redevențe`)** on extracted resources, plus the windfall levies applied to
  energy producers.
- **Local taxes** on property and land.
- **The wage bill itself**, and payments to Romanian suppliers — not tax, but the largest
  channel by which a company's activity stays in the local economy.

For scale: OMV Petrom paid **more than €48 billion in taxes and dividends between 2005 and
2025**, making it one of the largest single contributors to the Romanian budget.

## What can leave

### Dividends — and the exemption that matters

After-tax profit distributed to shareholders leaves with the shareholder. Romania raised
dividend withholding tax to **16% from 1 January 2026** (Law 141/2025).

But the **EU Parent-Subsidiary Directive** exempts it. Dividends paid by a Romanian company to
a company resident in another EU/EEA state are **free of withholding tax** where the parent has
held at least **10% of the shares for at least one year**. OMV has held ~51% for two decades,
so it qualifies comfortably.

The asymmetry is worth stating plainly:

| Shareholder | Withholding tax on the same dividend |
| --- | --- |
| A Romanian individual | 16% |
| The Austrian parent company | **0%** |

That is EU law working as designed — it prevents the same profit being taxed twice inside the
single market — not a loophole. But it does mean the dividend leg leaves Romania untaxed at
source, while a domestic small shareholder is taxed on it.

### Transfer pricing — the real argument

The part people actually worry about isn't dividends, it's charges between group companies
that reduce Romanian taxable profit before it is ever taxed:

- management and service fees paid to the parent
- royalties for the group's brand and IP
- interest on shareholder loans
- markups on goods bought through a group procurement entity

These are legal and often legitimate — the services may be real. They become avoidance when
priced above market. Romania constrains this with **arm's-length rules and a mandatory
transfer pricing file**, and ATAD-derived limits cap interest deductibility. ANAF audits it.
Whether the constraint bites in any given case is an empirical question, not something our
data can settle.

## Petrom is the example that breaks the binary

Your instinct was that Petrom is Austrian. The shareholder register says something more
interesting:

| Holder | Stake |
| --- | --- |
| OMV Aktiengesellschaft (Austria) | **51.2%** |
| Romanian state, via the Ministry of Energy | **20.7%** |
| Romanian pension funds, ~500,000 individuals, other Romanian entities | **24.4%** |

So roughly **45% of Petrom's dividends stay in Romania** — a fifth of them going straight to
the Romanian state, and a large slice to Romanian pension funds, meaning ordinary Romanians'
retirement savings.

Petrom is not "an Austrian company". It is **Austrian-controlled and about 45% Romanian-owned**.
Our `effective_ro` walk would return ≈0.45 — which is the truthful answer, and one a boolean
cannot express. This case is a good regression test for the scoring code.

## Control and ownership are different things

51.2% buys **control** — who appoints the board, sets strategy, decides where to invest.
Ownership percentage determines **where the money goes**. They don't have to agree, and both
are legitimate things for a user to care about:

- Someone worried about *decisions being made abroad* cares about control (>50%).
- Someone worried about *profit leaving the country* cares about the ownership percentage.

We store the percentages, so we can answer either. We should not silently pick one and call
it "Romanian".

## The uncomfortable implication for the product

If a user's real question is **"does my money stay in Romania?"**, ownership percentage is a
weaker proxy than it feels like.

Most of what a company contributes locally — wages, payroll taxes, VAT, corporate tax on local
profit, payments to local suppliers — accrues in Romania **whether the owner is Austrian or
Romanian**. Dividends are the part that leaves, and dividends are a fraction of value added.
A foreign-owned factory employing 2,000 people in Ploiești very likely keeps more money in
Romania than a Romanian-owned importer with eight staff.

That points at something we should be honest about in the UI: **for the "money stays here"
question, "made here" is usually the stronger signal, and ownership the weaker one.** Users
will expect the opposite. The app can lead them to the better answer without lecturing, by
showing the axes side by side rather than collapsing them.

## Sources

- [PwC — Romania, corporate withholding taxes](https://taxsummaries.pwc.com/romania/corporate/withholding-taxes)
- [PwC — Romania, taxes on corporate income](https://taxsummaries.pwc.com/romania/corporate/taxes-on-corporate-income)
- [Clearstream — Romanian dividend withholding tax increase from 1 January 2026](https://www.clearstream.com/clearstream-en/securities-services/asset-services/tax-and-certification/a25059-4660024)
- [European Commission — Parent-Subsidiary Directive](https://taxation-customs.ec.europa.eu/taxation/business-taxation/parent-subsidiary-directive_en)
- [EY — Romanian tax changes introduced by new fiscal and budgetary measures](https://www.ey.com/en_gl/technical/tax-alerts/romanian-tax-changes-introduced-by-new-fiscal-and-budgetary-measures)
- [OMV Petrom — shareholder structure](https://www.omvpetrom.com/en/investors/shares-and-dividends/shareholder-structure)
