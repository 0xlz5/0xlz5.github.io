---
layout: post
title: "When Swing Met the Grid ⚡"
subtitle: "A 2009 experiment in declarative UI, Groovy and energy-domain modeling"
date: 2026-08-25
categories:
  - software-architecture
tags:
  - java
  - swing
  - jfc
  - groovy
  - dsl
  - declarative-ui
  - model-driven-ui
  - presentation-model
  - energy
  - utilities
excerpt: "Reconstructing a 2009 idea: using Groovy to describe workflows and presentation models for a complex Energy & Utilities domain on top of Java Swing."
---

# ⚡ When Swing Met the Grid

## Part I — A Declarative UI for a Rich Energy & Utilities Domain

*Reconstructing a 2009 idea: Java 1.4/1.5 + Swing/JFC + Groovy DSL*

---

There was a particular class of enterprise application that made Swing feel... suspiciously imperative.

You had a serious domain model:

```text
Customer
   │
   ├── Contract
   │      │
   │      ├── Tariff
   │      └── Meter
   │
   └── Connection
          │
          └── SupplyPoint
                 │
                 ├── Measurements
                 ├── Outages
                 └── NetworkAsset
```

And then someone said:

> "We need a screen for this."

💀

Suddenly the beautiful OO model was surrounded by:

```java
button.addActionListener(...);
combo.addActionListener(...);
table.getModel()...
panel.add(...)
cardLayout.show(...)
if (...)
    button.setEnabled(...)
```

The domain was object-oriented.

The UI was becoming **event-oriented spaghetti**.

So why not describe the UI itself as a model?

---

## 🧠 The key distinction

The mistake would be:

```text
DOMAIN OBJECT
     │
     ▼
AUTO-GENERATED SWING FORM
```

A rich domain isn't a database schema.

Consider:

```java
class SupplyContract {
    Customer customer;
    SupplyPoint supplyPoint;
    Tariff tariff;
    Date start;
    Date end;

    void activate();
    void suspend();
    void terminate();

    boolean canActivate();
    boolean canTerminate();
}
```

A generic UI generator sees fields.

A domain expert sees:

```text
                  CONTRACT
                     │
       ┌─────────────┼─────────────┐
       │             │             │
     DRAFT       ACTIVE        SUSPENDED
       │             │             │
       └── activate ─┘             │
                     └── resume ───┘
```

That's a **state machine**, not a form.

And that difference is crucial.

---

## 🏗️ Three models, not one

The architecture I would propose is:

```text
                 ⚡ DOMAIN MODEL
              "What is electricity?"
                       │
                       │
                       ▼
              PRESENTATION MODEL
          "What does the operator
              need to see/do?"
                       │
                       │
                       ▼
               WORKFLOW MODEL
          "What can happen next?"
                       │
                       ▼
                SWING / JFC
             "How is it rendered?"
```

This is very close to the Presentation Model idea: move presentation state and behavior out of widgets and into an independent model.

The important bit:

> **The domain model does not become the UI model.**

---

# ⚡ Energy & Utilities makes this interesting

Imagine an operator handling a supply contract.

The domain might contain:

```text
Customer
   │
   └── SupplyContract
          │
          ├── SupplyPoint
          │      └── Meter
          │
          ├── Tariff
          │
          ├── Measurements
          │
          └── Events
```

But the UI might need:

```text
┌──────────────────────────────────────────────┐
│ ⚡ SUPPLY CONTRACT                            │
├──────────────────────────────────────────────┤
│ Customer:       ACME Industries               │
│ Supply Point:   IT001E12345678               │
│ Meter:          MTR-88392                    │
│ Tariff:         Industrial T2                 │
│                                              │
│ Status:         ● ACTIVE                     │
│                                              │
│ ┌─────────┐ ┌──────────┐ ┌──────────────┐   │
│ │ Suspend │ │ Readings │ │ Change Tariff│   │
│ └─────────┘ └──────────┘ └──────────────┘   │
└──────────────────────────────────────────────┘
```

Those buttons aren't simply methods discovered through reflection.

They represent **available business transitions**.

---

# 🧩 Enter the Presentation Model

A presentation object could expose:

```groovy
class ContractPM {

    SupplyContract contract

    String customerName()
    String supplyPointId()
    String tariffName()

    boolean canSuspend()
    boolean canTerminate()
    boolean canChangeTariff()

    void suspend()
    void terminate()
    void changeTariff(Tariff tariff)
}
```

Swing doesn't need to understand the domain graph.

It sees:

```text
ContractPM
 ├── customerName
 ├── supplyPointId
 ├── tariffName
 ├── canSuspend
 ├── suspend()
 ├── canTerminate
 └── terminate()
```

That's much easier to bind to a UI.

Fowler's formulation is almost exactly this idea: the Presentation Model can coordinate with several domain objects while remaining an abstraction of the view rather than a GUI-specific facade for one domain object.

---

# 🧙‍♂️ And Groovy becomes the little language

Instead of:

```java
JPanel panel = new JPanel();
JButton suspend = new JButton("Suspend");

suspend.addActionListener(...);

if (...) {
    suspend.setEnabled(true);
}

panel.add(...);
```

you could write:

```groovy
screen "SupplyContract" {

    field "customer"
        label: "Customer"
        bind: "customerName"

    field "supplyPoint"
        label: "Supply Point"
        bind: "supplyPointId"

    field "tariff"
        label: "Tariff"
        bind: "tariffName"

    status "status"
        bind: "status"

    command "suspend" {
        label "Suspend"
        enabledWhen { model.canSuspend() }
        action { model.suspend() }
    }

    command "terminate" {
        label "Terminate"
        enabledWhen { model.canTerminate() }
        action { model.terminate() }
    }
}
```

That's no longer "Groovy UI programming".

It's an **internal DSL for interaction**.

---

# 🔥 The big idea

The Java code describes the **mechanism**.

The DSL describes the **interaction semantics**.

```text
JAVA

create JButton
register listener
change layout
change state
enable/disable
refresh widgets
show dialog
...


DSL

field
command
bind
enabledWhen
validate
transition
```

That's exactly the sort of abstraction an internal DSL is good at.

---

# 🧪 And it becomes testable

Instead of testing Swing:

```text
click button
wait
find component
inspect component
...
```

you can test:

```groovy
when {
    contract.status == ACTIVE
}

then {
    pm.canSuspend()
}

when {
    pm.suspend()
}

then {
    contract.status == SUSPENDED
}
```

The GUI becomes mostly a renderer.

That separation is valuable because UI code is notoriously awkward to test, while the presentation logic can be exercised without widgets. Fowler explicitly identifies this as one of the benefits of Presentation Model.

---

# 🧭 The 2009 architecture

I'd therefore reconstruct your proposal as:

```text
             ┌─────────────────────┐
             │   ENERGY DOMAIN     │
             │                     │
             │ Customer            │
             │ Contract            │
             │ Meter               │
             │ Tariff              │
             │ SupplyPoint         │
             │ Measurement         │
             └──────────┬──────────┘
                        │
                        ▼
             ┌─────────────────────┐
             │ PRESENTATION MODEL  │
             │                     │
             │ properties          │
             │ commands            │
             │ validation          │
             │ permissions          │
             └──────────┬──────────┘
                        │
                        ▼
             ┌─────────────────────┐
             │ GROOVY UI DSL       │
             │                     │
             │ screens             │
             │ fields              │
             │ commands            │
             │ states              │
             │ transitions         │
             └──────────┬──────────┘
                        │
                        ▼
                 ┌────────────┐
                 │ Swing/JFC  │
                 └────────────┘
```

And that is a much more interesting proposal than simply:

> "Let's use Groovy instead of Java."

It is:

> **"Let's introduce a declarative interaction language between a rich domain model and Swing."**

⚡ That's a genuinely respectable 2009 architecture.

---

*Next: Part II — turning the energy domain into a stateful DSL: meters, supply contracts, outages, tariff changes, validation and temporal workflows.*
