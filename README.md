# Commerce Admin

An independent, store-agnostic operations workspace for running an online retail business. The product direction is a calm, app-like workspace with a Buffer-inspired focus on clear navigation, schedules, and useful at-a-glance information, extended to commerce operations.

The current application is an early Flutter admin panel. It is still connected directly to Mazeduneh APIs; the roadmap describes the staged move to a multi-store product and does not imply those capabilities exist today.

The default Mazeduneh branch contained non-text Dart entrypoints in commit `96d16a3`. To make the independent app source-readable, `dev` restores those four entrypoints from the last readable predecessor, Mazeduneh commit `d9cba6a`, then applies generic Commerce Admin names. Mazeduneh itself is unchanged; the source history remains available in the Mazeduneh repository.

## Start here

- [Product and development roadmap](docs/ROADMAP.md)
- [Modular product architecture decision](docs/adr/0001-modular-commerce-admin-architecture.md)
- [Initial source](#origin)

Development work and task status live on `dev`. Merge a module to `main` only after its acceptance criteria are complete. A module's workspace entitlement controls what the organization can use; an admin's profile preference only controls which entitled modules appear in that person's navigation.

## Demo store

Mazeduneh stays in the repository as the first reference integration and regression target. Demo scenarios must use synthetic data and must not require Mazeduneh credentials. Keep its connector isolated so the product can be run against the demo store, Mazeduneh, and other supported stores independently.

## Origin

Initial source: `apps/admin` from [Hamrez95/Mazeduneh](https://github.com/Hamrez95/Mazeduneh), default branch `dev` at commit `0c17bf43a280f1c5d75d90aaf96616bc1f234216`.

This repository was initialized to continue the admin panel refactor independently.
