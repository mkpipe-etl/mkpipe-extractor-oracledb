# mkpipe-extractor-oracledb

Oracle Database extractor plugin for [MkPipe](https://github.com/mkpipe-etl/mkpipe). Reads Oracle tables via JDBC.

## Documentation

For more detailed documentation, please visit the [GitHub repository](https://github.com/mkpipe-etl/mkpipe).

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

---

## Connection Configuration

```yaml
connections:
  oracle_source:
    variant: oracledb
    host: localhost
    port: 1521
    database: ORCL
    user: myuser
    password: mypassword
```

---

## Table Configuration

```yaml
pipelines:
  - name: oracle_to_pg
    source: oracle_source
    destination: pg_target
    tables:
      - name: MYSCHEMA.EVENTS
        target_name: stg_events
        replication_method: full
        fetchsize: 100000
```

### Incremental Replication

```yaml
      - name: MYSCHEMA.EVENTS
        target_name: stg_events
        replication_method: incremental
        iterate_column: UPDATED_AT
        iterate_column_type: datetime
        partitions_column: ID
        partitions_count: 8
        fetchsize: 50000
```

### Custom SQL

```yaml
      - name: MYSCHEMA.EVENTS
        target_name: stg_events
        replication_method: full
        custom_query: "SELECT ID, USER_ID, EVENT_TYPE FROM MYSCHEMA.EVENTS WHERE {query_filter}"
```

---

## Read Parallelism

Set `partitions_column` and `partitions_count` for parallel JDBC reads. Partitioning only applies to incremental replication.

### Performance Notes

- Oracle's `ROWID` range partitioning is the most efficient for parallel scans, but mkpipe uses column-range partitioning for simplicity. Use a numeric PK column for best distribution.
- Oracle JDBC `fetchsize` defaults are often low — explicitly set `fetchsize: 50000` or higher.

---

## All Table Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `name` | string | required | Oracle table name (include schema: `SCHEMA.TABLE`) |
| `target_name` | string | required | Destination table name |
| `replication_method` | `full` / `incremental` | `full` | Replication strategy |
| `iterate_column` | string | — | Column used for incremental watermark |
| `iterate_column_type` | `int` / `datetime` | — | Type of `iterate_column` |
| `partitions_column` | string | same as `iterate_column` | Column to split JDBC reads on |
| `partitions_count` | int | `10` | Number of parallel JDBC partitions |
| `fetchsize` | int | `100000` | Rows per JDBC fetch |
| `custom_query` | string | — | Override SQL with `{query_filter}` placeholder |
| `custom_query_file` | string | — | Path to SQL file (relative to `sql/` dir) |
| `tags` | list | `[]` | Tags for selective pipeline execution |
| `pass_on_error` | bool | `false` | Skip table on error instead of failing |



