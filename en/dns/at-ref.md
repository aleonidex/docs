---
title: '{{ dns-full-name }} event reference in {{ at-full-name }}'
description: This page gives a reference for {{ dns-name }} events tracked in {{ at-name }}.
---

# {{ at-full-name }} event reference

{{ at-name }} supports tracking [control plane](../audit-trails/concepts/format.md) and [data plane](../audit-trails/concepts/format-data-plane.md) events for {{ dns-full-name }}.

The `event_type` value follows this general format:

```text
{{ at-event-prefix }}.audit.dns.<event_name>
```

## Control plane event reference {#control-plane-events}

{% include [dns-events](../_includes/audit-trails/events/dns-events.md) %}

## Data plane event reference {#data-plane-events}

{% include [dns-events-dp](../_includes/audit-trails/events/dns-events-dp.md) %}