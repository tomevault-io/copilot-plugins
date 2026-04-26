## credit-market-simulator

> use the following as a reference plus search the web for your own information.  I want an interactive web-based credit simulatory that models the real world and helps the user learn how the credit system in the US works.

use the following as a reference plus search the web for your own information.  I want an interactive web-based credit simulatory that models the real world and helps the user learn how the credit system in the US works.






// credit_system.ts
// A tiny, self-contained model of the U.S. consumer credit market for developers.
// No dependencies. Run with: `ts-node credit_system.ts` (or compile with tsc).

/* =========================
 * 1) Shared Types & Utils
 * ========================= */

type Money = number; // dollars; keep it simple

const round2 = (n: number) => Math.round(n * 100) / 100;

// Deterministic PRNG for repeatable sims
class PRNG {
  private s = 42;
  rand() {
    // xorshift-ish
    let x = this.s ^= this.s << 13;
    x ^= x >> 17;
    x ^= x << 5;
    this.s = x;
    return Math.abs(x % 10000) / 10000;
  }
}

/* =========================
 * 2) Domain Entities
 * ========================= */

// Borrower/Consumer
interface BorrowerProfile {
  id: string;
  name: string;
  monthlyIncome: Money;
  monthlyExpenses: Money;
  cashOnHand: Money;
}

type TradelineType = "CreditCard" | "PersonalLoan" | "Auto" | "Mortgage" | "BNPL";

interface TradelineSnapshot {
  lenderId: string;
  type: TradelineType;
  openedAtMonth: number;
  creditLimit?: Money;
  principalOriginal: Money;
  principalCurrent: Money;
  status: "Current" | "Late30" | "Late60" | "Defaulted" | "Closed";
}

interface BureauRecord {
  borrowerId: string;
  score: number; // FICO-like 300-850
  tradelines: TradelineSnapshot[];
  lastUpdatedMonth: number;
}

// Simple loan
interface LoanTerms {
  apr: number; // e.g., 0.24 = 24% APR
  termMonths: number;
  originationFeePct: number; // taken up-front
  type: TradelineType;
}

interface Loan {
  id: string;
  borrowerId: string;
  originatorId: string;
  terms: LoanTerms;
  principalOriginal: Money;
  principalCurrent: Money;
  monthlyPayment: Money; // fixed payment for simplicity
  status: "Active" | "Defaulted" | "PaidOff" | "Sold";
  openedAtMonth: number;
}

// Lenders (Originators) & Servicers
interface OriginatorConfig {
  id: string;
  name: string;
  targetROA: number; // baked into pricing/fees
  servicingPartnerId: string;
}

interface ServicerConfig {
  id: string;
  name: string;
  annualServicingFeePct: number; // extracted from cash flows
  daysToReportToBureau: number; // latency
}

// Investors & Securitization
interface Security {
  id: string;
  name: string;
  collateralLoanIds: string[];
  couponAPR: number; // what the security promises to pay investors
  servicerId: string;
  creationMonth: number;
  status: "Active" | "Matured";
  poolPrincipalCurrent: Money;
}

interface InvestorPosition {
  investorId: string;
  securityId: string;
  invested: Money;
  accruedInterest: Money;
}

interface Investor {
  id: string;
  name: string;
  costOfCapitalAPR: number; // hurdle/expectation
  cash: Money;
  positions: InvestorPosition[];
}

// Regulators (modeled as constraints/validators)
type RegulatoryEvent =
  | { type: "ECOA_NoticeLogged"; borrowerId: string; originatorId: string; month: number }
  | { type: "FCRA_AdverseActionLogged"; borrowerId: string; originatorId: string; month: number; reason: string }
  | { type: "TILA_DisclosureProvided"; borrowerId: string; originatorId: string; apr: number; month: number };

class Regulator {
  events: RegulatoryEvent[] = [];
  log(e: RegulatoryEvent) {
    this.events.push(e);
  }
}

/* =========================
 * 3) Messaging & Eventual Consistency
 * ========================= */

type DomainEvent =
  | { type: "CreditApplied"; borrowerId: string; month: number }
  | { type: "CreditDecisioned"; borrowerId: string; originatorId: string; loanId: string; approved: boolean; apr: number; month: number }
  | { type: "FundsDisbursed"; loanId: string; amount: Money; feesTaken: Money; month: number }
  | { type: "PaymentPosted"; loanId: string; borrowerId: string; amount: Money; interest: Money; principal: Money; servicerId: string; month: number }
  | { type: "DelinquencyUpdated"; loanId: string; status: TradelineSnapshot["status"]; month: number }
  | { type: "BureauReportReady"; borrowerId: string; tradeline: TradelineSnapshot; scheduledMonth: number }
  | { type: "BureauUpdated"; borrowerId: string; score: number; month: number }
  | { type: "LoansSecuritized"; securityId: string; loanIds: string[]; couponAPR: number; month: number }
  | { type: "InvestorSubscribed"; investorId: string; securityId: string; amount: Money; month: number }
  | { type: "SecurityDistribution"; securityId: string; investorId: string; interest: Money; principal: Money; month: number };

class EventBus {
  private queue: { deliverAtMonth: number; event: DomainEvent }[] = [];
  subscribe(fn: (e: DomainEvent) => void) {
    this.handlers.push(fn);
  }
  schedule(event: DomainEvent, deliverAtMonth: number) {
    this.queue.push({ deliverAtMonth, event });
  }
  tick(currentMonth: number) {
    const due = this.queue.filter(x => x.deliverAtMonth <= currentMonth);
    this.queue = this.queue.filter(x => x.deliverAtMonth > currentMonth);
    for (const { event } of due) for (const h of this.handlers) h(event);
  }
  private handlers: ((e: DomainEvent) => void)[] = [];
}

/* =========================
 * 4) Bureaus & Scoring
 * ========================= */

class Bureau {
  private records = new Map<string, BureauRecord>();
  constructor(private bus: EventBus) {
    bus.subscribe(e => {
      if (e.type === "BureauReportReady") {
        this.applyReport(e.borrowerId, e.tradeline, e.scheduledMonth);
      }
    });
  }

  get(borrowerId: string): BureauRecord {
    if (!this.records.has(borrowerId)) {
      this.records.set(borrowerId, { borrowerId, score: 680, tradelines: [], lastUpdatedMonth: 0 });
    }
    return this.records.get(borrowerId)!;
  }

  private computeScore(rec: BureauRecord): number {
    // Toy score model:
    // Start at 700, subtract based on utilization & delinquency, add for age.
    let score = 700;
    let totalLimit = 0;
    let totalBalance = 0;
    let worst = "Current" as TradelineSnapshot["status"];
    let ageBonus = 0;

    for (const tl of rec.tradelines) {
      totalBalance += tl.principalCurrent;
      totalLimit += tl.creditLimit ?? tl.principalOriginal;
      if (tl.status === "Defaulted") worst = "Defaulted";
      else if (tl.status === "Late60" && worst !== "Defaulted") worst = "Late60";
      else if (tl.status === "Late30" && !["Defaulted", "Late60"].includes(worst)) worst = "Late30";
      const age = Math.max(0, rec.lastUpdatedMonth - tl.openedAtMonth);
      ageBonus += Math.min(40, age * 0.5); // up to +40
    }

    const util = totalLimit > 0 ? totalBalance / totalLimit : 0;
    score -= Math.min(120, util * 200); // heavy penalty for high util
    if (worst === "Late30") score -= 40;
    if (worst === "Late60") score -= 90;
    if (worst === "Defaulted") score -= 180;
    score += Math.min(60, ageBonus);

    return Math.max(300, Math.min(850, Math.round(score)));
  }

  private applyReport(borrowerId: string, tradeline: TradelineSnapshot, month: number) {
    const rec = this.get(borrowerId);
    // upsert tradeline by (lenderId + openedAtMonth)
    const idx = rec.tradelines.findIndex(
      t => t.lenderId === tradeline.lenderId && t.openedAtMonth === tradeline.openedAtMonth
    );
    if (idx >= 0) rec.tradelines[idx] = tradeline; else rec.tradelines.push(tradeline);
    rec.lastUpdatedMonth = month;
    rec.score = this.computeScore(rec);
    this.bus.schedule({ type: "BureauUpdated", borrowerId, score: rec.score, month }, month);
  }
}

/* =========================
 * 5) Originator (Lender) + Simple Decisioning
 * ========================= */

class Originator {
  cfg: OriginatorConfig;
  private loans = new Map<string, Loan>();
  private prng = new PRNG();

  constructor(cfg: OriginatorConfig, private bus: EventBus, private regulator: Regulator, private bureau: Bureau) {
    this.cfg = cfg;
  }

  // Simple underwriting using bureau + cash flow
  underwrite(b: BorrowerProfile, month: number, requested: Money, type: TradelineType): Loan | null {
    const rec = this.bureau.get(b.id);
    const score = rec.score;
    const dti = (b.monthlyExpenses / Math.max(1, b.monthlyIncome)); // rough
    // Base APR by score bucket
    const baseAPR =
      score >= 760 ? 0.12 :
      score >= 700 ? 0.18 :
      score >= 640 ? 0.24 :
      score >= 580 ? 0.32 : 0.45;

    // Adjust for DTI
    const apr = Math.min(0.60, baseAPR + Math.max(0, (dti - 0.35)) * 0.5);

    // Simple affordability: max payment 20% income
    const maxMonthlyPay = b.monthlyIncome * 0.2;
    const term = type === "CreditCard" ? 36 : 24;
    const monthlyRate = apr / 12;
    const payment = monthlyRate === 0
      ? requested / term
      : requested * (monthlyRate / (1 - Math.pow(1 + monthlyRate, -term)));

    const approved = payment <= maxMonthlyPay;
    this.bus.schedule({
      type: "CreditDecisioned",
      borrowerId: b.id,
      originatorId: this.cfg.id,
      loanId: `${this.cfg.id}_${Date.now()}_${Math.floor(this.prng.rand()*1e6)}`,
      approved,
      apr,
      month
    }, month);

    this.regulator.log({ type: "ECOA_NoticeLogged", borrowerId: b.id, originatorId: this.cfg.id, month });
    if (!approved) {
      this.regulator.log({
        type: "FCRA_AdverseActionLogged",
        borrowerId: b.id,
        originatorId: this.cfg.id,
        reason: "Insufficient affordability (DTI/payment cap).",
        month
      });
      return null;
    }

    this.regulator.log({ type: "TILA_DisclosureProvided", borrowerId: b.id, originatorId: this.cfg.id, apr, month });

    const terms: LoanTerms = {
      apr,
      termMonths: term,
      originationFeePct: 0.04, // 4% fee
      type
    };

    const loan: Loan = {
      id: `${this.cfg.id}_${Math.floor(this.prng.rand()*1e9)}`,
      borrowerId: b.id,
      originatorId: this.cfg.id,
      terms,
      principalOriginal: requested,
      principalCurrent: requested,
      monthlyPayment: round2(payment),
      status: "Active",
      openedAtMonth: month
    };

    this.loans.set(loan.id, loan);

    // Disburse funds (net of origination fee)
    const fee = round2(requested * terms.originationFeePct);
    this.bus.schedule({ type: "FundsDisbursed", loanId: loan.id, amount: round2(requested - fee), feesTaken: fee, month }, month);

    // Initial bureau tradeline schedule (report later via servicer)
    return loan;
  }

  getLoan(id: string) { return this.loans.get(id); }
  allLoans() { return [...this.loans.values()]; }
}

/* =========================
 * 6) Servicer: payments, delinquency, bureau reporting
 * ========================= */

class Servicer {
  private loans: Map<string, Loan>;
  private statuses = new Map<string, TradelineSnapshot["status"]>();
  private prng = new PRNG();

  constructor(
    private cfg: ServicerConfig,
    private bus: EventBus,
    private bureau: Bureau,
    originator: Originator
  ) {
    this.loans = new Map(originator.allLoans().map(l => [l.id, l]));
  }

  attachLoans(loans: Loan[]) {
    for (const l of loans) this.loans.set(l.id, l);
  }

  processMonth(borrowers: Map<string, BorrowerProfile>, currentMonth: number) {
    for (const loan of this.loans.values()) {
      if (loan.status !== "Active") continue;
      const b = borrowers.get(loan.borrowerId)!;

      // Simple default/late behavior: if cash < payment, go late and maybe default
      let payment = loan.monthlyPayment;
      const monthlyRate = loan.terms.apr / 12;
      const interest = round2(loan.principalCurrent * monthlyRate);
      let principalPaid = 0;

      if (b.cashOnHand >= payment) {
        // make payment
        b.cashOnHand = round2(b.cashOnHand - payment);
        principalPaid = round2(payment - interest);
        loan.principalCurrent = round2(Math.max(0, loan.principalCurrent - principalPaid));
        this.statuses.set(loan.id, "Current");
        this.bus.schedule({
          type: "PaymentPosted",
          loanId: loan.id,
          borrowerId: b.id,
          amount: payment,
          interest,
          principal: principalPaid,
          servicerId: this.cfg.id,
          month: currentMonth
        }, currentMonth);
      } else {
        // Late: escalate with probability
        const prev = this.statuses.get(loan.id) || "Current";
        const next = prev === "Current" ? "Late30" : prev === "Late30" ? "Late60" : "Defaulted";
        // Random chance to default early if very cash-poor
        const p = b.cashOnHand < payment * 0.1 ? 0.6 : 0.2;
        const actuallyNext = (this.prng.rand() < p && next !== "Defaulted") ? next : prev;
        this.statuses.set(loan.id, actuallyNext);
        if (actuallyNext === "Defaulted") {
          loan.status = "Defaulted";
        }
        this.bus.schedule({ type: "DelinquencyUpdated", loanId: loan.id, status: this.statuses.get(loan.id)!, month: currentMonth }, currentMonth);
      }

      // Bureau reporting (eventually consistent)
      const tl: TradelineSnapshot = {
        lenderId: loan.originatorId,
        type: loan.terms.type,
        openedAtMonth: loan.openedAtMonth,
        creditLimit: loan.terms.type === "CreditCard" ? loan.principalOriginal : undefined,
        principalOriginal: loan.principalOriginal,
        principalCurrent: loan.principalCurrent,
        status: this.statuses.get(loan.id) || "Current"
      };
      const reportAt = currentMonth + Math.ceil(this.cfg.daysToReportToBureau / 30);
      this.bus.schedule({ type: "BureauReportReady", borrowerId: loan.borrowerId, tradeline: tl, scheduledMonth: reportAt }, reportAt);

      if (loan.principalCurrent <= 0) {
        loan.status = "PaidOff";
        this.statuses.set(loan.id, "Closed");
      }
    }
  }
}

/* =========================
 * 7) Securitization & Investor Flows
 * ========================= */

class Securitizer {
  private securities = new Map<string, Security>();

  constructor(private bus: EventBus) {}

  createSecurity(id: string, name: string, loans: Loan[], couponAPR: number, servicerId: string, month: number) {
    const poolPrincipal = round2(loans.reduce((sum, l) => sum + Math.max(0, l.principalCurrent), 0));
    const sec: Security = {
      id, name,
      collateralLoanIds: loans.map(l => l.id),
      couponAPR,
      servicerId,
      creationMonth: month,
      status: "Active",
      poolPrincipalCurrent: poolPrincipal
    };
    this.securities.set(id, sec);
    this.bus.schedule({ type: "LoansSecuritized", securityId: id, loanIds: sec.collateralLoanIds, couponAPR, month }, month);
    return sec;
  }

  get(id: string) { return this.securities.get(id); }

  // Pay investors pro-rata based on a simple coupon; principal flows through as collected.
  distribute(sec: Security, investors: Investor[], collectedInterest: Money, collectedPrincipal: Money, month: number) {
    const holders = investors.flatMap(inv => inv.positions.filter(p => p.securityId === sec.id)
                           .map(p => ({ inv, pos: p })));
    const totalInvested = holders.reduce((s, h) => s + h.pos.invested, 0);
    if (totalInvested <= 0) return;

    for (const h of holders) {
      const share = h.pos.invested / totalInvested;
      const iPay = round2(collectedInterest * share);
      const pPay = round2(collectedPrincipal * share);
      h.pos.accruedInterest = round2(h.pos.accruedInterest + iPay);
      h.inv.cash = round2(h.inv.cash + iPay + pPay);
      this.bus.schedule({ type: "SecurityDistribution", securityId: sec.id, investorId: h.inv.id, interest: iPay, principal: pPay, month }, month);
    }

    sec.poolPrincipalCurrent = round2(Math.max(0, sec.poolPrincipalCurrent - collectedPrincipal));
    if (sec.poolPrincipalCurrent <= 0) sec.status = "Matured";
  }
}

class InvestorLedger {
  subscribe(investor: Investor, security: Security, amount: Money, month: number) {
    if (investor.cash < amount) throw new Error("Insufficient cash");
    investor.cash = round2(investor.cash - amount);
    const pos = investor.positions.find(p => p.securityId === security.id);
    if (pos) pos.invested = round2(pos.invested + amount);
    else investor.positions.push({ investorId: investor.id, securityId: security.id, invested: amount, accruedInterest: 0 });
    return { type: "InvestorSubscribed" as const, investorId: investor.id, securityId: security.id, amount, month };
  }
}

/* =========================
 * 8) “Rails”: Payments & Collections
 * ========================= */

class PaymentsRail {
  // Return what portion of incoming cash is interest vs principal (toy calculator).
  static splitLoanPayment(loan: Loan): { interest: Money; principal: Money } {
    const monthlyRate = loan.terms.apr / 12;
    const interest = round2(loan.principalCurrent * monthlyRate);
    const principal = round2(Math.max(0, loan.monthlyPayment - interest));
    return { interest, principal };
  }
}

/* =========================
 * 9) Minimal “Market” Simulation
 * ========================= */

class MarketSim {
  private month = 0;
  private bus = new EventBus();
  private regulator = new Regulator();
  private bureau = new Bureau(this.bus);

  private borrowers = new Map<string, BorrowerProfile>();
  private originator: Originator;
  private servicer: Servicer | null = null;
  private securitizer = new Securitizer(this.bus);
  private investors: Investor[] = [];
  private ledger = new InvestorLedger();

  private collectedBySecurity = new Map<string, { interest: Money; principal: Money }>();

  constructor() {
    this.originator = new Originator(
      { id: "LENDR1", name: "Lendr Bank", targetROA: 0.03, servicingPartnerId: "SERV1" },
      this.bus, this.regulator, this.bureau
    );

    this.bus.subscribe(e => this.observe(e));
  }

  addBorrower(b: BorrowerProfile) {
    this.borrowers.set(b.id, b);
  }

  addInvestor(inv: Investor) {
    this.investors.push(inv);
  }

  applyAndFund(borrowerId: string, amount: Money, type: TradelineType) {
    const b = this.borrowers.get(borrowerId)!;
    this.bus.schedule({ type: "CreditApplied", borrowerId: b.id, month: this.month }, this.month);
    const loan = this.originator.underwrite(b, this.month, amount, type);
    if (!loan) return null;
    // Borrower receives net proceeds immediately
    const fee = round2(amount * loan.terms.originationFeePct);
    b.cashOnHand = round2(b.cashOnHand + amount - fee);

    // Attach servicer if needed
    if (!this.servicer) {
      this.servicer = new Servicer(
        { id: "SERV1", name: "Acme Servicing", annualServicingFeePct: 0.005, daysToReportToBureau: 35 },
        this.bus, this.bureau, this.originator
      );
    } else {
      this.servicer.attachLoans([loan]);
    }

    return loan.id;
  }

  securitize(name: string, loanIds: string[], couponAPR: number) {
    const loans = loanIds.map(id => this.originator.getLoan(id)!).filter(l => l && l.status === "Active");
    const sec = this.securitizer.createSecurity(`SEC_${Date.now()}`, name, loans, couponAPR, "SERV1", this.month);
    this.collectedBySecurity.set(sec.id, { interest: 0, principal: 0 });
    return sec.id;
  }

  investorSubscribe(investorId: string, securityId: string, amount: Money) {
    const inv = this.investors.find(i => i.id === investorId)!;
    const sec = this.securitizer.get(securityId)!;
    const evt = this.ledger.subscribe(inv, sec, amount, this.month);
    this.bus.schedule(evt, this.month);
  }

  private observe(e: DomainEvent) {
    // Aggregate cash flows earmarked for securities (toy: if loan in a security, its payment flows to that pool).
    if (e.type === "PaymentPosted") {
      // Identify which security holds the loan (naive: search)
      for (const [secId, bucket] of this.collectedBySecurity) {
        const sec = this.securitizer.get(secId)!;
        if (sec.collateralLoanIds.includes(e.loanId)) {
          bucket.interest = round2(bucket.interest + e.interest);
          bucket.principal = round2(bucket.principal + e.principal);
        }
      }
    }
  }

  stepMonth() {
    this.month += 1;

    // Borrower incomes/expenses accrue
    for (const b of this.borrowers.values()) {
      b.cashOnHand = round2(b.cashOnHand + b.monthlyIncome - b.monthlyExpenses);
    }

    // Servicing cycle
    if (this.servicer) this.servicer.processMonth(this.borrowers, this.month);

    // Deliver scheduled events (bureau updates, etc.)
    this.bus.tick(this.month);

    // Distribute to investors from pools
    for (const [secId, bucket] of this.collectedBySecurity) {
      const sec = this.securitizer.get(secId)!;
      if (!sec || sec.status !== "Active") continue;

      // Simple servicing skim (annual -> monthly)
      const servicingFee = 0.005 / 12;
      const skimInterest = round2(bucket.interest * (1 - servicingFee));
      const skimPrincipal = bucket.principal; // typically principal passes through
      this.securitizer.distribute(sec, this.investors, skimInterest, skimPrincipal, this.month);

      // reset bucket after distribution
      bucket.interest = 0;
      bucket.principal = 0;
    }
  }

  snapshot() {
    return {
      month: this.month,
      borrowers: [...this.borrowers.values()],
      loans: this.originator.allLoans(),
      regulatorEvents: this.regulator.events.slice(-6),
      investorBalances: this.investors.map(i => ({ id: i.id, cash: i.cash, positions: i.positions })),
    };
  }
}

/* =========================
 * 10) Demo Run (Optional)
 * ========================= */

async function main() {
  const sim = new MarketSim();

  // Borrowers
  sim.addBorrower({ id: "B1", name: "Alice", monthlyIncome: 6000, monthlyExpenses: 3800, cashOnHand: 2000 });
  sim.addBorrower({ id: "B2", name: "Bob", monthlyIncome: 3500, monthlyExpenses: 3200, cashOnHand: 200 });

  // Investors
  sim.addInvestor({ id: "I1", name: "Alpha Fund", costOfCapitalAPR: 0.08, cash: 100000, positions: [] });
  sim.addInvestor({ id: "I2", name: "Beta Insurance", costOfCapitalAPR: 0.05, cash: 200000, positions: [] });

  // Origination
  const loan1 = sim.applyAndFund("B1", 15000, "Auto");        // Alice gets an auto loan
  const loan2 = sim.applyAndFund("B2", 3000, "CreditCard");   // Bob gets a credit card-like installment

  // Securitize both loans into one pool and have investors subscribe
  const secId = sim.securitize("Consumer ABS 2025-1", [loan1!, loan2!], 0.12);
  sim.investorSubscribe("I1", secId, 50000);
  sim.investorSubscribe("I2", secId, 80000);

  // Run 12 months
  for (let i = 0; i < 12; i++) {
    sim.stepMonth();
  }

  // Print snapshot
  const snap = sim.snapshot();
  console.log("=== MONTH", snap.month, "SNAPSHOT ===");
  console.log("Borrowers:", snap.borrowers);
  console.log("Loans:", snap.loans.map(l => ({
    id: l.id, borrowerId: l.borrowerId, apr: l.terms.apr, principalCurrent: l.principalCurrent, status: l.status
  })));
  console.log("Recent Regulator Events:", snap.regulatorEvents);
  console.log("Investors:", snap.investorBalances);
}

// Run if executed directly
// (Uncomment the next line when running with ts-node)
// main().catch(console.error);

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
