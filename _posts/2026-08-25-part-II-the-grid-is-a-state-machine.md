---
layout: post
title: "A Bit of History – Part II - The Grid Is a State Machine ⚡"
subtitle: "Modeling Energy & Utilities workflows with a Groovy DSL"
date: 2026-08-25
categories:
  - software-architecture
tags:
  - java
  - swing
  - jfc
  - groovy
  - dsl
  - workflow
  - state-machine
  - declarative-ui
  - model-driven-ui
  - energy
  - utilities
excerpt: "From supply contracts and meters to outages: expressing state, guards, validation and transitions as a declarative UI workflow."
---

# ⚡ The Grid Is a State Machine

## Part II — Modeling Energy Workflows Declaratively

*Java 1.4/1.5 · Swing/JFC · Groovy · 2009-ish enterprise architecture*

---

Part I established the separation:

```text
Domain → Presentation Model → DSL → Swing
```

Now comes the fun part.

Energy & Utilities applications are full of things that **change state**.

A meter isn't just a record.

A contract isn't just a record.

A supply point isn't just a record.

An outage isn't just a record.

They have **lifecycles**.

---

# 🔌 A contract lifecycle

For example:

```text
                    ┌─────────┐
                    │  DRAFT  │
                    └────┬────┘
                         │ activate
                         ▼
                    ┌─────────┐
             ┌──────│ ACTIVE  │──────┐
             │      └────┬────┘      │
          suspend        │          terminate
             │           │             │
             ▼           │             ▼
        ┌──────────┐      │        ┌───────────┐
        │ SUSPENDED│──────┘        │ TERMINATED│
        └──────────┘   resume      └───────────┘
```

A conventional Swing implementation tends to distribute this knowledge across:

* buttons
* listeners
* controllers
* dialogs
* validation methods
* panel switching
* `setEnabled()`
* table models

The DSL can put the workflow in one place.

---

# 🧙‍♂️ A workflow DSL

Imagine:

```groovy
workflow "SupplyContract" {

    state "draft" {

        screen "ContractEditor"

        command "activate" {

            enabledWhen {
                model.canActivate()
            }

            validate {
                required model.customer
                required model.supplyPoint
                required model.tariff
            }

            action {
                model.activate()
            }

            goto "active"
        }
    }

    state "active" {

        screen "ActiveContract"

        command "suspend" {

            enabledWhen {
                model.canSuspend()
            }

            action {
                model.suspend()
            }

            goto "suspended"
        }

        command "terminate" {

            enabledWhen {
                model.canTerminate()
            }

            confirm "Terminate supply contract?"

            action {
                model.terminate()
            }

            goto "terminated"
        }
    }

    state "suspended" {

        screen "SuspendedContract"

        command "resume" {
            action {
                model.resume()
            }

            goto "active"
        }
    }

    state "terminated" {
        screen "TerminatedContract"
    }
}
```

🔥 This is where the DSL starts becoming useful.

---

# 🧠 State ≠ screen

This is an important architectural detail.

Don't define:

```text
screen = state
```

Define:

```text
state
  │
  ├── screen
  ├── commands
  ├── guards
  ├── validation
  └── transitions
```

Because the same state could have different views.

For example:

```text
                 ACTIVE
                   │
          ┌────────┴────────┐
          │                 │
     Operator UI       Customer UI
          │                 │
     Swing screen       Web screen
```

The domain state is independent of presentation.

That's exactly the spirit of separated presentation: domain objects should remain independent of the presentation and support multiple presentations.

---

# ⚡ Now add the meter

Meters make the example more interesting.

```groovy
workflow "Meter" {

    state "planned" {
        screen "MeterInstallation"

        command "install" {

            validate {
                required model.serialNumber
                required model.location
            }

            action {
                model.install()
            }

            goto "installed"
        }
    }

    state "installed" {

        screen "MeterDashboard"

        command "read" {
            action {
                model.takeReading()
            }
        }

        command "replace" {

            action {
                model.startReplacement()
            }

            goto "replacement"
        }
    }

    state "replacement" {

        screen "MeterReplacement"

        command "complete" {

            action {
                model.completeReplacement()
            }

            goto "installed"
        }
    }
}
```

The UI language now expresses something meaningful:

```text
INSTALL
   ↓
INSTALLED
   ├── READ
   └── REPLACE
          ↓
      REPLACEMENT
          ↓
       INSTALLED
```

The Swing widgets are merely the projection.

---

# 📈 Measurements introduce time

And here's where a utility domain starts getting really interesting.

Measurements aren't just:

```java
double value;
```

They have:

```text
timestamp
quality
unit
register
source
estimation status
```

So the presentation model could expose:

```groovy
measurementTable {

    column "Time"
        bind "timestamp"

    column "Value"
        bind "value"

    column "Unit"
        bind "unit"

    column "Quality"
        bind "quality"

    style "estimated"
        when { row.estimated }
}
```

The domain remains responsible for:

```text
Measurement
   │
   ├── value
   ├── timestamp
   ├── unit
   ├── quality
   └── estimated
```

The UI DSL decides how that semantic information becomes visible.

---

# 🚨 Outages

Now imagine an outage-management workflow:

```text
             DETECTED
                │
                ▼
             CONFIRMED
                │
                ▼
           DISPATCHED
                │
                ▼
           IN_PROGRESS
                │
                ▼
             RESTORED
                │
                ▼
             CLOSED
```

DSL:

```groovy
workflow "Outage" {

    state "detected" {
        screen "OutageAlert"

        command "confirm" {
            action {
                model.confirm()
            }

            goto "confirmed"
        }
    }

    state "confirmed" {
        screen "OutageDetails"

        command "dispatch" {
            enabledWhen {
                model.availableCrew()
            }

            action {
                model.dispatch()
            }

            goto "dispatched"
        }
    }

    state "dispatched" {
        screen "CrewTracking"

        command "startWork" {
            action {
                model.startWork()
            }

            goto "inProgress"
        }
    }

    state "inProgress" {

        command "restore" {
            enabledWhen {
                model.canRestore()
            }

            action {
                model.restore()
            }

            goto "restored"
        }
    }

    state "restored" {

        command "close" {
            action {
                model.close()
            }

            goto "closed"
        }
    }
}
```

Suddenly:

```text
Swing
```

isn't where the workflow lives anymore.

The DSL is the workflow.

---

# 🧩 The runtime

The runtime could be tiny:

```text
WorkflowDefinition
       │
       ▼
WorkflowEngine
       │
       ├── currentState
       ├── model
       ├── availableCommands()
       └── fire(command)
               │
               ▼
         PresentationModel
               │
               ▼
           Swing View
```

Something like:

```java
workflow.fire("suspend");
```

causes:

```text
1. locate command
2. evaluate guard
3. validate
4. execute action
5. update state
6. notify presentation model
7. refresh Swing
```

The UI becomes an interpreter for a little language.

---

# 🧪 Which is actually a very nice test boundary

You can test:

```groovy
def "active contract can be suspended"() {

    given:
        contract.status == ACTIVE

    when:
        workflow.fire("suspend")

    then:
        contract.status == SUSPENDED
}
```

No JFrame.

No `Robot`.

No clicking.

No `Thread.sleep()`.

No:

```java
SwingUtilities.invokeLater(...)
```

😂

---

# 🔭 The deeper idea

The DSL is effectively a **finite-state interaction language**:

```text
                 DOMAIN
                   │
                   ▼
             DOMAIN STATE
                   │
                   ▼
          PRESENTATION STATE
                   │
                   ▼
            UI STATE MACHINE
                   │
                   ▼
              SWING VIEW
```

This is why a rich utility domain is such a good candidate.

The UI isn't merely displaying objects.

It is guiding users through **legal transformations of those objects**.

---

# 🧮 And it can become data-driven

Eventually you could imagine:

```groovy
command "changeTariff" {

    requires Role.OPERATOR

    enabledWhen {
        model.contract.active
    }

    action {
        model.changeTariff(selectedTariff)
    }

    goto "tariffChanged"
}
```

Now the DSL knows about:

```text
roles
permissions
guards
validation
commands
workflow
navigation
```

At that point you have something considerably closer to a tiny **application language** than a UI builder.

And that, retrospectively, is probably the most interesting part of the 2009 idea.

---

*Next: Part III — from internal DSL to a real model-driven UI architecture: binding, metadata, generated screens, Swing/JFC adapters, and where the idea stops being clever and starts becoming dangerous.*
