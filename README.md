# [Truck Contributor and Developer Book](https://bigmark222.github.io/truck_contributor_and_developer_book/)

This repo is now structured as an [mdBook](https://github.com/rust-lang/mdBook). The source chapters live under `src/` and can be built or served locally.

## Prerequisites

- Install `mdbook` (via `cargo install mdbook` or your package manager).

## Usage

- `mdbook serve --open` – run a live-reloading dev server.
- `mdbook build` – generate the static site into `book/`.

The original HTML exports remain in `docs/english` for reference while the mdBook becomes the new source of truth. Translators can fork this book if they want to publish another language.
