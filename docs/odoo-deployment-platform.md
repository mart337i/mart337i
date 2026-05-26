# Odoo Deployment Platform

This is a sanitized implementation note for a self-hosted Odoo deployment platform I built around a custom Odoo control plane and a dedicated deployment API.

The original implementation documentation lives with the private platform module in `egeskov-group/me_odoo_deployment/README.md` and `egeskov-group/DEPLOYMENT_SETUP_NOTES.md`. This public note summarizes the architecture and engineering decisions without exposing host-specific operations, credentials, internal URLs, firewall rules, or production runbooks.

## What It Does

The platform manages Odoo instances through a control-plane UI backed by a deployment API. It is designed to support repeatable deployment, migration, proxying, backup, copy/import, and observability workflows for managed Odoo environments.

Core capabilities include:

- Instance lifecycle management for create, plan, redeploy, destroy, turn on, and turn off operations.
- Manifest-driven deployments for reproducible Docker Compose workspaces.
- Repository and addon configuration for custom, private, and OCA-style module sources.
- nginx proxy synchronization for managed domains and websocket routes.
- Certificate strategy handling for inherited, provided, and Let's Encrypt based certificates.
- Database and filestore copy/import flows for migrations between managed instances.
- Observability integration with metrics, logs, dashboards, and operational summaries.

## Architecture

The system is split into two main parts:

- **Odoo control plane**: Stores deployment metadata, instance definitions, repository configuration, addon entries, domains, certificate policies, jobs, manifests, and observability summaries.
- **Deployment API**: Executes host-level deployment tasks, renders workspaces, manages Docker Compose projects, writes proxy configuration, validates generated infrastructure state, and reports task results back to Odoo.

This split keeps business workflow and operator UX inside Odoo while isolating infrastructure execution inside a purpose-built API service.

## Deployment Model

Deployments are manifest-driven. The control plane builds a desired-state manifest from Odoo records, then submits it to the deployment API for planning or execution.

The API is responsible for:

- Rendering workspace files.
- Preparing Docker Compose projects.
- Applying repository and addon configuration.
- Managing current and previous workspace revisions.
- Updating reverse proxy configuration.
- Running validation before publishing changes.
- Returning structured task state and logs.

This creates a safer operational model than ad-hoc shell deployments because each instance has a generated, inspectable representation of its target state.

## Migration Support

The platform was built with real Odoo migrations in mind. Migration workflows include:

- Creating a managed target instance before public cutover.
- Copying database and filestore data between managed instances.
- Importing backup archives into a deployed target.
- Preserving or adjusting repository configuration during copy workflows.
- Supporting neutralization flows for non-production targets.
- Delaying public proxy cutover until the managed target is verified.

This is especially useful when moving legacy or manually operated Odoo installations into a managed platform model.

## Proxy And Certificate Handling

The platform uses nginx as the public routing layer for managed instances. It generates proxy configuration for standard Odoo HTTP traffic and websocket routes.

Certificate handling was designed to reduce fragile deployment failures. The platform supports inherited defaults and explicit overrides, and it validates certificate state before enabling generated proxy configuration.

The main goal is to prevent one bad domain or certificate configuration from silently breaking unrelated deployment work.

## Observability

The observability design combines lightweight operational summaries in Odoo with deeper metrics and logs in dedicated observability tools.

The platform integrates with:

- Grafana for dashboards.
- Loki for log exploration.
- VictoriaMetrics for metrics history.
- Promtail and vmagent for collection.

Odoo remains the operator-facing control plane, while Grafana is used for deeper operational analysis.

## Engineering Focus

The project demonstrates work across several engineering areas:

- Odoo module architecture and business workflow modeling.
- Python API design for infrastructure automation.
- Docker Compose based runtime management.
- nginx reverse proxy automation.
- PostgreSQL and filestore migration concerns.
- GitHub/private repository integration.
- Observability design for self-hosted systems.
- Operational safety around validation, cutover, and rollback paths.

The result is a practical platform for managing Odoo deployments with a stronger control-plane model than manual server operations.
