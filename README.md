diesel-static-postgres
======================

A statically compiled version of Diesel with libpq. You can download it easily
from GitHub into your CI directly.

Use Cases
---------

 *  If you have multiple projects that share the same database schema
    and you want to centralize the migrations in one place and then create
    an empty database in your CI for testing purpose.

    ### Example with CircleCI

    ```yaml
    version: 2.1
    jobs:
      build:
        docker:
          - image: circleci/rust
            user: root
          - image: circleci/postgres:10.5-alpine
        working_directory: ~/repo
        steps:
          - run:
              name: Checkout infrastructure
              command: git clone git@github.com:your/infrastructure.git ~/infrastructure
          - run:
              name: Initialize database
              command: |
                curl -o /usr/local/bin/diesel https://github.com/cecton/diesel-static-postgres/releases/download/v1.4.0/diesel
                chmod +x /usr/local/bin/diesel
                diesel migration --migration-dir ~/infrastructure/migrations --database-url postgresql://postgres@localhost:5432/postgres run
          - checkout:
              path: ~/repo
          - restore_cache:
              keys:
                - cache-v1-{{ .Branch }}-{{ .Revision }}
                - cache-v1-{{ .Branch }}-
                - cache-v1-
          - run:
              name: Check formatting
              command: |
                rustup component add rustfmt
                rustfmt --check **/*.rs
          - run:
              name: Run tests
              command: |
                cargo test
          - save_cache:
              key: cache-v1-{{ .Branch }}-{{ .Revision }}
              paths:
                - /usr/local/cargo/registry
                - /usr/local/cargo/bin/rustfmt
                - ./target
    ```

 *  If you have a single repository that doesn't necessary rely on diesel
    except for the migrations.

    ### Example with CircleCI

    ```yaml
    version: 2.1
    jobs:
      build:
        docker:
          - image: circleci/rust
            user: root
          - image: circleci/postgres:10.5-alpine
        working_directory: ~/repo
        steps:
          - checkout:
              path: ~/repo
          - restore_cache:
              keys:
                - cache-v1-{{ .Branch }}-{{ .Revision }}
                - cache-v1-{{ .Branch }}-
                - cache-v1-
          - run:
              name: Initialize database
              command: |
                curl -o /usr/local/bin/diesel https://github.com/cecton/diesel-static-postgres/releases/download/v1.4.0/diesel
                chmod +x /usr/local/bin/diesel
                diesel migration --database-url postgresql://postgres@localhost:5432/postgres run
          - run:
              name: Check formatting
              command: |
                rustup component add rustfmt
                rustfmt --check **/*.rs
          - run:
              name: Run tests
              command: |
                cargo test
          - save_cache:
              key: cache-v1-{{ .Branch }}-{{ .Revision }}
              paths:
                - /usr/local/cargo/registry
                - /usr/local/cargo/bin/rustfmt
                - ./target
    ```

Available Versions
------------------

Check the
[releases](https://github.com/cecton/diesel-static-postgres/releases).
