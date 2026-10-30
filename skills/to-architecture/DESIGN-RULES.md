# Design rules

Adapted from the `codebase-design` skill in [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, © 2026 Matt Pocock), which builds on John Ousterhout's *A Philosophy of Software Design* and Michael Feathers' *Working Effectively with Legacy Code*.

The goal is **deep modules**: a lot of behaviour behind a small interface, placed at a clean seam, testable through that interface. That gives callers leverage, maintainers locality, and agents a codebase they can navigate and verify.

## Vocabulary

Use these terms exactly, in conversation and in every architecture doc. Do not substitute "component", "service", "API" or "boundary".

- **Module:** anything with an interface and an implementation. Scale-agnostic on purpose: a function, a class, a package, or a slice spanning several tiers.
- **Interface:** everything a caller must know to use the module correctly. Not just the type signature: also invariants, ordering constraints, error modes, required configuration and performance characteristics.
- **Implementation:** the code inside a module.
- **Depth:** leverage at the interface: how much behaviour a caller, or a test, can exercise per unit of interface it has to learn. A module is **deep** when a lot of behaviour sits behind a small interface, and **shallow** when its interface is nearly as complex as its implementation.
- **Seam:** a place where behaviour can be altered without editing in that place; the location where a module's interface lives. Where the seam goes is a design decision of its own, separate from what goes behind it.
- **Adapter:** a concrete thing that satisfies an interface at a seam. The word describes a role (which slot it fills), not what is inside.
- **Leverage:** what callers get from depth: more capability per unit of interface. One implementation pays back across every call site and every test.
- **Locality:** what maintainers get from depth: change, bugs, knowledge and verification concentrate in one place instead of spreading across callers.

## Deep versus shallow

```
Deep module                         Shallow module (avoid)
┌─────────────────────┐             ┌─────────────────────────────────┐
│   Small interface   │             │         Large interface         │
├─────────────────────┤             ├─────────────────────────────────┤
│                     │             │  Thin implementation            │
│  Deep implementation│             └─────────────────────────────────┘
│                     │
└─────────────────────┘
```

When shaping an interface, ask:

- Can it have fewer entry points?
- Can the parameters be simpler?
- Can more of the complexity move inside?

## Principles

- **Depth belongs to the interface, not the implementation.** A deep module may be built from small, swappable internal parts. Those parts are not its interface. A module can have **internal seams**, used only by its own tests, as well as the **external seam** at its interface. Never expose an internal seam through the interface just because a test uses it.
- **The deletion test.** Imagine deleting the module. If the complexity disappears, it was a pass-through and should go. If the complexity reappears in every caller, it was earning its keep.
- **The interface is the test surface.** Callers and tests cross the same seam. If a test needs to reach past the interface, the module has the wrong shape.
- **One adapter means a hypothetical seam; two adapters mean a real one.** Do not introduce a seam unless something actually varies across it, typically a production adapter and a test adapter.
- **Highest seam, fewest seams.** Prefer an existing seam to a new one, and the highest seam that still runs fast. The ideal number of seams for a feature is one.

## Designing for testability

1. **Accept dependencies, don't create them.**

   ```ts
   // Testable
   function processOrder(order: Order, gateway: PaymentGateway) {}

   // Hard to test
   function processOrder(order: Order) {
     const gateway = new StripeGateway();
   }
   ```

2. **Return results, don't produce side effects.**

   ```ts
   // Testable
   function calculateDiscount(cart: Cart): Discount {}

   // Hard to test
   function applyDiscount(cart: Cart): void {
     cart.total -= discount;
   }
   ```

3. **Small surface area.** Fewer entry points need fewer tests; simpler parameters need simpler setup.

## Dependency categories

Classify every dependency of a module. The category decides how the module is tested across its seam, and goes into the module doc's Seam and dependencies section.

1. **In-process:** pure computation or in-memory state, no I/O. Test directly through the interface. No adapter needed.
2. **Local-substitutable:** a dependency with a local stand-in that runs in the test suite (an in-memory or embedded database, an in-memory filesystem). Test with the stand-in. The seam is internal; no port at the external interface.
3. **Remote but owned:** your own services across a network boundary. Define a **port** at the seam; the module owns the logic and the transport is injected as an adapter: HTTP, gRPC or queue in production, in-memory in tests.
4. **True external:** third-party services you don't control (payment providers, email, SMS). The module takes the dependency as an injected port; tests use a fake adapter.

## Testing strategy: replace, don't layer

- Tests assert on observable outcomes through the interface, never on internal state.
- Tests must survive internal refactors. A test that has to change when the implementation changes is testing past the interface.
- When shallow modules are merged into a deep one, write tests at the new interface and delete the old tests on the shallow parts. Do not keep both layers.
- A spec rule's examples become tests at the seam recorded in the owning module's Rules owned table, titled with `[<domain>.<rule_name>]`.
