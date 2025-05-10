# Tools

Welds has a sister project [Gumbo](https://crates.io/crates/gumbo) that is a command line tool (CLI) that can be extremely useful when writing welds code.

Reasons you might be interested in this CLI:
- basic project setup
- migration generation
- model generation
- code generation from database tables

## Gumbo setup

```bash
cargo install gumbo

# to use the database related commands, set the ENV `DATABASE_URL`
export DATABASE_URL=sqlite://./dev.sqlite
```

## Project Setup

You can initialize a basic empty welds project with the `init` command

```bash
gumbo init --welds-only project_name
```

## Migration Generation
```
gumbo generate migration ...
```

## Code Generation

To generate models from an existing database, Make sure that you have the ENV VAR `DATABASE_URL` set in your shell.
You can then generate models individually, or all at once. This command can be re-ran on your models after generation. 
Only the fields on your welds struct will be altered

```bash
# single table
gumbo db model-from-table table_name
# or all tables
gumbo db model-from-table -- --all-tables
```

