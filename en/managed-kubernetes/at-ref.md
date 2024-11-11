# {{ at-full-name }} event reference

{{ at-name }} supports tracking [control plane](../audit-trails/concepts/format.md) events for {{ managed-k8s-full-name }}.

The general view of the `event_type` field value is as follows:

```text
{{ at-event-prefix }}.audit.k8s.<event_name>
```

## Management event reference {#control-plane-events}

{% include [managed-k8s-events](../_includes/audit-trails/events/managed-k8s-events.md) %}

