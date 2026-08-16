# LikeC4 DSL reference

Source of truth: [likec4.dev/dsl](https://likec4.dev/dsl/specification) —
verify against the live docs if behavior here seems off, the language
evolves. This doc condenses the grammar an agent needs to write correct
`.c4`/`.likec4` source without round-tripping to the site.

## File basics

- Extensions: `.c4` or `.likec4` — no functional difference.
- A project is the set of source files under one workspace root (nearest
  `likec4.config.json`, else nearest `package.json`/`.git`). **All files
  merge into a single model** — no `import` statement; reference elements
  from any file by their qualified name.
- Four top-level block types, any number of each, spread across any number
  of files: `specification { }`, `model { }`, `deployment { }`, `views { }`.
  A fifth, `global { }`, holds shared style/predicate definitions reused
  across views (see [Global groups](#global-predicate-and-style-groups)).
- Minimal valid file: `specification { } model { } views { }`.

## Specification block

Declares every kind an agent is allowed to use elsewhere. **Nothing is
built in** — no default element kinds, relationship kinds, or deployment
node kinds.

### Element kinds

```
specification {
  element person
  element system
  element service
  element database

  element queue {
    title 'Kafka'
    description 'Kafka queue'
    technology 'kafka topic'
    notation 'Kafka Topic'   // label shown in the legend
    style {
      shape queue
    }
  }
}
```

### Relationship kinds

Optional — plain `->` relationships need no declared kind. Declare a kind to
give a category of relationships shared defaults.

**Verified against `likec4` CLI v1.46.0** (`likec4 validate`): a
`relationship <kind> { }` body only accepts `notation`, `technology`,
`color`, `line`, `head`, `tail` — unlike element/deployment-node kinds, it
does **not** accept `title`, `description`, `link`, `multiple`, a leading
`#tag`, or a nested `style { }` block. (The public docs page shows some of
these as valid; they did not parse against the installed CLI when tested —
re-verify with `likec4 validate` if targeting a different version.)

```
specification {
  relationship uses
  relationship async {
    notation 'Async'
    technology 'Kafka'
    color amber
    line dotted
  }
}
```

### Deployment node kinds

```
specification {
  deploymentNode environment
  deploymentNode zone
  deploymentNode vm {
    notation 'Virtual Machine'
    technology 'VMware'
  }
  deploymentNode kubernetes {
    style {
      color blue
      icon tech:kubernetes
      multiple true
    }
  }
}
```

### Tags

```
specification {
  tag deprecated
  tag epic-123
  tag team2

  tag deprecated {
    color #FF0000
  }
}
```

### Custom colors

```
specification {
  color custom-color1 #F00
  color custom-color2 #AABBCC
  color custom-color3 rgb(255, 0, 0)
  color custom-color4 rgba(44, 8, 128, 0.9)
}
```
Only 3/6/8-char hex or `rgb()`/`rgba()` are accepted.

## Model block

### Elements

Syntax: `<kind> <name>` or `<name> = <kind>`, optionally with a quoted title
and/or a body.

```
model {
  customer = person 'Customer'
  cloud = system 'Cloud Platform' {
    backend = service 'Backend API'
  }
}
```

- **Naming**: letters, digits, `-`, `_`; cannot start with a digit or contain
  a period (`.` is the namespace separator, e.g. `cloud.backend`).
- **No duplicate names within the same parent.**

### Element properties

```
saas = softwareSystem 'SaaS' {
  summary 'Brief text'
  description 'Detailed information'      // or ''' triple-quoted ''' for Markdown
  technology 'REST'                       // or auto-derived from `icon tech:...`

  #deprecated                             // tags MUST come first in the body

  link https://github.com/likec4/likec4 'Repository'
  link ssh://bastion.internal 'SSH'

  metadata {
    prop1 'value1'
    tags ['frontend', 'react', 'typescript']   // array value
  }

  style {
    shape rectangle
    color primary
    icon tech:apache-flink
  }
}
```

Markdown in `description`/`summary` needs triple quotes:
```
description '''
  ### Heading
  [Link text](url)
'''
```

### Nested elements

Elements are containers; nesting creates dotted namespaces:

```
service service1 {
  component backend { component api }
  component frontend
}
// -> service1, service1.backend, service1.backend.api, service1.frontend
```

### Relationships

Operator is `->` (source to target). Three forms:

```
model {
  customer -> cloud                              // bare
  customer -> cloud.backend 'opens in browser'   // inline title
  customer -> cloud.backend {                     // nested, full properties
    title 'opens in browser'
    description 'Customer opens the web app'
    technology 'HTTPS'
    #critical #api
    link https://docs.example.com
    navigateTo dashboard-request-flow             // jump to another view on click
    metadata { sla '99.9%' }
  }

  actor customer {
    -> frontend   // sourceless form: source is the parent element (customer)
  }
}
```

Custom relationship kind: `system1 -[async]-> system2` (kind must be declared
in `specification`).

### `extend` — add to an element/relationship from elsewhere

`extend` is only valid **nested inside a `model { }` block** — a bare
top-level `extend ...` outside `model { }` is a parse error (`Expecting
token of type 'EOF' but found 'extend'`, verified against CLI v1.46.0). It's
typically its own `model { }` block in a different file from where the
element/relationship was first declared.

```
model {
  extend cloud {                 // must use the fully-qualified name
    #additional-tag
    metadata { owner 'platform-team' }
    service2 = service 'Service 2'
  }
}

model {
  extend frontend -> api "Makes requests" {   // extend a relationship (logical model only)
    metadata { latency_p95 '150ms' }
  }
}
```
Relationship extension matches on source + target + kind + title — all must
match. Duplicate metadata keys across an element and its extensions merge
into de-duplicated arrays (a single remaining value stays a plain string).

## Views block

```
views {
  view index { }         // the `index` view is the default render target
  view { }                // unnamed view, auto-named

  view epic12 {
    #next, #epic-12
    title "Cloud System — Changes in Epic-12"
    description """High-level components and interactions"""
    link https://my.jira/epic/12 'Epic-12'
    include *
  }

  view of cloud.backend {         // scoped to an element; becomes its default view
    include api                    // resolves to cloud.backend.api
  }

  view view2 extends view1 {      // inherit predicates/styles, then layer more
    title 'Same as View1, but with more detail'
    style * { color muted }
    include some.backend
  }
}
```

### Include/exclude predicates

```
include backend
exclude messageBroker.emailsQueue
include backend, frontend, authService, messageBroker.**

include *          // top-level elements (or scoped element + children in a scoped view)
include cloud.*     // direct children of cloud
include cloud.**    // all descendants of cloud, with their relationships
include cloud._      // children of cloud, with relationships, omitting the rest

include element.kind != system
exclude element.tag = #deprecated

include cloud.backend with {
  title 'Backend components'
  color amber
  navigateTo view3
}
```

### Relationship predicates

```
include customer -> cloud          // directed
include customer -> cloud.*
include customer <-> cloud         // either direction
include -> backend                 // anything incoming to backend
include cloud.* ->                 // anything outgoing from cloud.*
include -> cloud.* ->              // both directions

include cloud.* <-> amazon.* with {
  color red
  line solid
}
include webApp -> backend.api with { navigateTo dashboardRequestFlow }
```

### `where` filters

```
include cloud.* where kind is microservice
exclude element.kind = webapp

include cloud.* where
  not (kind is microservice or kind is webapp)
  and tag is not #legacy

include cloud.* <-> amazon.* where tag is #messaging
include -> backend where kind is http-request
include cloud.* -> amazon.* where source.tag is #next
include -> * where target.kind is microservice

include cloud.* where metadata.environment is "production"
exclude * where not metadata.version
include * where metadata.critical is true
```

### Groups (visual grouping, not the model hierarchy)

```
group {
  include backend
}
group 'Frontend' {
  include frontend.*
}
group 'Service Bus' {
  color amber
  opacity 20%
  include messageBroker.*
}
group 'Third-parties' {
  group 'Integrations' {
    group 'Analytics' { }
  }
}
```

### Style predicates (within a view)

```
style * { color muted opacity 10% }
style singlePageApplication, mobileApp { color secondary size xlarge }
style apiApplication.* { color primary }
style apiApplication._ { color primary }
style element.tag = #deprecated { color muted }
```

### Global predicate and style groups

Define once in `global { }`, reuse across views:

```
global {
  predicateGroup microservices {
    include cloud.* where kind is microservice
    exclude * where tag is #deprecated
  }

  style mute_all * { color muted opacity 10% }

  styleGroup common_styles {
    style singlePageApplication, mobileApp { color secondary }
  }
}

views {
  view of newServices {
    global predicate microservices
    global style mute_all
    global style common_styles
  }
}
```

### Auto-layout

```
view {
  include *
  autoLayout LeftRight 120 110   // direction, nodeSpacing, rankSpacing (both optional)
}
```
Directions: `TopBottom`, `BottomTop`, `LeftRight`, `RightLeft`.

### Rank constraints

```
view checkoutFlow {
  include *
  rank same { cloud.backend.api, cloud.backend.billingApi }
  rank source { customer }
  rank sink { analytics }
}
```
Allowed rank values: `same`, `min`, `max`, `source`, `sink`.

`rank` and `group` are each fine on their own; combining both in the same
tiny view crashed the bundled dot-wasm layouter in local testing (CLI
v1.46.0, a layout-engine error, not a DSL syntax error). If a view using
both fails to lay out, try dropping one or splitting into two views before
assuming the source is wrong — `likec4 validate` reports this as a layout
failure, not an "Invalid" parse error.

## Deployment model

Nodes come from `deploymentNode` kinds declared in `specification`; logical
elements are placed into the topology with `instanceOf`.

```
deployment {
  environment prod 'Production' {
    #live #sla-customer
    technology 'OpenTofu'
    summary 'Production environment'
    description '''Detailed description with **Markdown**'''
    link https://likec4.dev

    zone eu 'EU Region' {
      zone zone1 {
        instanceOf frontend.ui
        ui = instanceOf frontend.ui
        api1 = instanceOf backend.api
        api2 = instanceOf backend.api

        db = instanceOf database {
          title 'Primary DB'
          technology 'PostgreSQL with streaming replication'
          icon tech:postgresql
          style { color red }
        }
      }
    }
  }
}
```

### Deployment relationships

```
deployment {
  environment prod {
    vm vm1 { db = instanceOf database 'Primary DB' }
    vm vm2 { db = instanceOf database 'Standby DB' }

    vm2.db -> vm1.db 'replicates'
    vm2.db -[streaming]-> vm1.db {
      #next #live
      title 'replicates'
      description 'Streaming replication'
    }
    vm2.db -> vm1.db.repl_log 'replicates'
  }
}
```

## Styling reference

Style blocks appear inside `specification { element ... }`, `model { ... }`,
`deployment { ... }`, or view `style` predicates.

| Property | Values |
|---|---|
| `shape` | `rectangle` (default), `component`, `storage`, `cylinder`, `browser`, `mobile`, `person`, `queue`, `bucket`, `document` |
| `color` | `primary` (default), `secondary`, `muted`, `amber`, `gray`, `green`, `indigo`, `red`, or a custom color declared in `specification` |
| `size` | `xsmall`/`xs`, `small`/`sm`, `medium`/`md` (default), `large`/`lg`, `xlarge`/`xl` |
| `padding` | space around the title |
| `textSize` | title font size (same scale as `size`) |
| `opacity` | percentage, mainly for group containers |
| `border` | `dashed` (default), `dotted`, `solid`, `none` |
| `multiple` | `true`/`false` — render as a stack (multiple instances) |
| `icon` | `tech:`/`aws:`/`azure:`/`gcp:`/`bootstrap:`-prefixed bundled icon, or a URL/local path |
| `iconColor`, `iconSize`, `iconPosition` | `iconPosition`: `left` (default), `right`, `top`, `bottom` |

Relationship-specific style properties: `line` (`dashed` default, `solid`,
`dotted`), `head`/`tail` arrow types (`normal`, `onormal`, `diamond`,
`odiamond`, `crow`, `vee`, `open`, `none`), `color`, `multiple`.

An `icon tech:*` on an element auto-derives its `technology` label if none is
set explicitly (bootstrap icons are excluded from this auto-derivation).
Element styles set in `model { }` override the kind's defaults from
`specification { }`.
