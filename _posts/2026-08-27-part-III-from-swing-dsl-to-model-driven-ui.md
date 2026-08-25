---
layout: post
title: "From Swing DSL to Model-Driven UI ⚡"
subtitle: "Where the 2009 idea scales — and where it starts to break"
date: 2026-08-27
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
  - data-binding
  - workflow
  - energy
  - utilities
excerpt: "From declarative Swing workflows to model-driven UI: generating mechanics while keeping business semantics explicit."
---

# ⚡ From Swing DSL to Model-Driven UI

## Part III — Where the 2009 Idea Goes If You Push It

---

So far we have:

```text
        ⚡ Energy Domain
               │
               ▼
       Presentation Model
               │
               ▼
       Groovy UI DSL
               │
               ▼
          Swing / JFC
```

But there is a tempting next step:

> "Why write the UI DSL manually at all?"

Why not generate it?

😈

---

# 🏭 Model-driven UI

Imagine the domain metadata contains:

```text
SupplyContract
 ├── customer : Customer
 ├── supplyPoint : SupplyPoint
 ├── tariff : Tariff
 ├── startDate : Date
 ├── endDate : Date
 └── status : ContractStatus
```

You could generate the boring 80%:

```text
Customer       → CustomerSelector
SupplyPoint    → SupplyPointSelector
Tariff         → TariffSelector
Date           → DateField
Enum           → ComboBox
Boolean        → CheckBox
Collection     → JTable
```

But **not**:

```text
activate
suspend
terminate
```

Those belong to the application's interaction semantics.

So:

```text
              DOMAIN METADATA
                    │
             ┌──────┴──────┐
             ▼             ▼
        DATA MODEL      BEHAVIOR
             │             │
             ▼             ▼
       AUTO-BINDINGS   DSL RULES
             │             │
             └──────┬──────┘
                    ▼
              PRESENTATION
                    │
                    ▼
                 SWING
```

That hybrid is much more powerful than pure code generation.

---

# 🧬 Declarative + generated

For example:

```groovy
screen "ContractEditor" {

    bind model

    autoFields()

    command "activate"
    command "suspend"
    command "terminate"

    table "measurements"

    workflow "SupplyContract"
}
```

The engine can infer:

```text
String       → JTextField
Date         → DatePicker
Enum         → JComboBox
Collection   → JTable
Object       → nested editor
Command      → JButton
```

But you retain explicit control where the domain gets interesting.

---

# ⚡ Energy-specific metadata

Imagine:

```groovy
field "consumption" {

    bind "consumption"

    unit "kWh"

    format "#,##0.00"

    warningWhen {
        value > model.threshold
    }
}
```

Or:

```groovy
field "power" {

    bind "power"

    unit "kW"

    alarmWhen {
        value > model.maxPower
    }
}
```

Now the DSL starts expressing **domain-aware presentation semantics**.

That is much more interesting than generic form generation.

---

# 📊 Tables become semantic projections

Consider measurements:

```groovy
table "readings" {

    bind "measurements"

    column "timestamp"
        format "yyyy-MM-dd HH:mm"

    column "value"
        format "#,##0.000"

    column "unit"

    column "quality"

    rowStyle {
        estimated when row.quality == ESTIMATED
        invalid  when row.quality == INVALID
    }
}
```

The renderer could remain ordinary Swing:

```text
JTable
  ↑
TableAdapter
  ↑
PresentationModel
  ↑
Measurement[]
```

The DSL simply describes the projection.

---

# 🧠 The important architectural constraint

Don't let this happen:

```text
              DSL
               │
               ▼
          Domain Model
               │
               ▼
             Swing
```

where the DSL starts modifying domain objects arbitrarily.

Instead:

```text
                DSL
                 │
                 ▼
         Presentation Model
                 │
                 ▼
           Application API
                 │
                 ▼
            Domain Model
```

The presentation layer asks the application/domain layer to perform operations.

It doesn't become the domain.

Fowler's broader "presentation domain separation" argument is precisely about keeping UI concerns out of domain classes; among the benefits are multiple presentations and easier testing.

---

# 🧱 The 2009 technology stack

A plausible implementation could have been:

```text
Java 1.4/1.5
│
├── Domain
│    └── ordinary Java OO
│
├── Application services
│    └── Java
│
├── Presentation Model
│    └── JavaBeans / observable properties
│
├── Groovy
│    └── declarative DSL
│
└── Swing / JFC
     └── renderer
```

And Groovy is particularly attractive here because it doesn't require abandoning Java.

You can have:

```groovy
model.suspend()
```

calling:

```java
public void suspend() {
    ...
}
```

while the DSL handles:

```groovy
enabledWhen { ... }
goto "suspended"
```

That is an excellent division of labour.

---

# 🔌 An interesting contemporary parallel

This wasn't happening in a vacuum.

JFace Data Binding was explicitly designed to reduce repetitive UI event handling and automate synchronization between business models and presentation widgets.

Its architecture even anticipated adapters around model objects and reusable controllers between business and presentation layers.

So your proposed architecture sits in the same historical design space:

```text
      Rich domain
           │
           │
     abstraction
           │
           ▼
  presentation model
           │
      data binding
           │
           ▼
         widgets
```

Your twist was:

```text
        + WORKFLOW DSL
        + DECLARATIVE SCREEN DESCRIPTION
        + GROOVY AS META-LANGUAGE
```

That's the interesting bit.

---

# 🚀 Where I'd take it today

If I were resurrecting the idea now, I wouldn't necessarily use Groovy.

I'd preserve the **architecture**, not the technology:

```text
             DOMAIN
                │
                ▼
         APPLICATION API
                │
                ▼
       PRESENTATION MODEL
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
   WORKFLOW DSL      VIEW MODEL
        │                │
        └───────┬────────┘
                ▼
              UI
```

The DSL could be:

```text
Groovy
Kotlin DSL
TypeScript
JSON/YAML
Clojure
Scala
Rust macro DSL
```

depending on the environment.

---

# 🤖 And the really modern twist

Today an LLM could potentially generate the boring presentation layer from the domain metadata.

For example:

```text
Domain model
     ↓
metadata
     ↓
AI-assisted presentation model
     ↓
human-authored workflow DSL
     ↓
renderer
```

The human specifies the **semantics**:

```groovy
command "terminate" {
    requires Role.OPERATOR

    enabledWhen {
        model.canTerminate()
    }

    confirm "Terminate supply?"

    action {
        model.terminate()
    }
}
```

while tooling generates:

```text
button
dialog
binding
validation messages
keyboard shortcut
toolbar entry
menu entry
```

---

# 🧨 But there's a trap

Model-driven UI can become **metadata-driven UI hell**.

You don't want:

```text
@Entity
@Field
@Widget
@Layout
@Validation
@Visibility
@Workflow
@Permission
@Presentation
@...
```

until the domain model has become an accidental UI schema.

The sweet spot is:

```text
                    GENERATE
                       ↓
             boring CRUD / binding
                       +
                       │
                 DECLARE EXPLICITLY
                       ↓
          business workflow / interaction
```

In other words:

> **Generate mechanics. Declare semantics.**

That's probably the strongest formulation of your original idea.

---

# 🧭 The retrospective architecture

If I were documenting your 2009 proposal today, I'd draw it like this:

```text
                       ⚡ ENERGY DOMAIN
                  ┌──────────────────────┐
                  │ Customer             │
                  │ Contract             │
                  │ Meter                │
                  │ SupplyPoint          │
                  │ Tariff               │
                  │ Measurement          │
                  │ Outage               │
                  └──────────┬───────────┘
                             │
                       domain API
                             │
                             ▼
                  ┌──────────────────────┐
                  │ PRESENTATION MODEL   │
                  │                      │
                  │ properties           │
                  │ commands             │
                  │ validation           │
                  │ permissions          │
                  │ UI state             │
                  └──────────┬───────────┘
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
       ┌─────────────────┐      ┌─────────────────┐
       │  WORKFLOW DSL   │      │  VIEW METADATA  │
       │                 │      │                 │
       │ states          │      │ fields          │
       │ guards          │      │ tables          │
       │ transitions     │      │ formatting      │
       │ commands        │      │ bindings        │
       └────────┬────────┘      └────────┬────────┘
                │                        │
                └────────────┬───────────┘
                             ▼
                    ┌─────────────────┐
                    │ SWING / JFC     │
                    │                 │
                    │ JFrame          │
                    │ JPanel          │
                    │ JTable          │
                    │ JButton         │
                    └─────────────────┘
```

## 🧠 The 2009 verdict

So, looking back:

**Yes — I would have endorsed the idea.**

But I would have sharpened the terminology:

> ❌ *"Use Groovy to build the Swing UI."*

> 🟡 *"Create a model-driven UI."*

> 🟢 **"Introduce a Presentation Model and a declarative workflow DSL between a rich Energy & Utilities domain model and Swing."**

And the really good design principle is:

### **Domain objects model the energy system.**

### **Presentation models model the user's interaction with it.**

### **The DSL models the workflow.**

### **Swing renders the result.**

That is a surprisingly modern architecture for a proposal made around **2009**. ⚡
