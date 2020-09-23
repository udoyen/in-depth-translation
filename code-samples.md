```
export TABLE_NAME=$(echo bq_vpcflows.$(bq query --nouse_legacy_sql --format=prettyjson 'SELECT * FROM bq_vpcflows.INFORMATION_SCHEMA.TABLES' | grep table_name | cut -d":" -f2 | cut -d"," -f1 | cut -d'"' -f2)) && \
bq query --nouse_legacy_sql <<< 'SELECT
      jsonPayload.src_vpc.vpc_name,
      SUM(CAST(jsonPayload.bytes_sent AS INT64)) AS bytes,
      jsonPayload.src_vpc.subnetwork_name,
      jsonPayload.connection.src_ip,
      jsonPayload.connection.src_port,
      jsonPayload.connection.dest_ip,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      FROM ' \
      "${TABLE_NAME}" \
      ' GROUP BY
      jsonPayload.src_vpc.vpc_name,
      jsonPayload.src_vpc.subnetwork_name,
      jsonPayload.connection.src_ip,
      jsonPayload.connection.src_port,
      jsonPayload.connection.dest_ip,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      ORDER BY
      bytes DESC
      LIMIT
      15'

```

```
export TABLE_NAME=$(echo bq_vpcflows.$(bq query --nouse_legacy_sql --format=prettyjson 'SELECT * FROM bq_vpcflows.INFORMATION_SCHEMA.TABLES' | grep table_name | cut -d":" -f2 | cut -d"," -f1 | cut -d'"' -f2)) && \
echo 'SELECT
      jsonPayload.src_vpc.vpc_name,
      SUM(CAST(jsonPayload.bytes_sent AS INT64)) AS bytes,
      jsonPayload.src_vpc.subnetwork_name,
      jsonPayload.connection.src_ip,
      jsonPayload.connection.src_port,
      jsonPayload.connection.dest_ip,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      FROM ' \
      ${TABLE_NAME} \
      ' GROUP BY
      jsonPayload.src_vpc.vpc_name,
      jsonPayload.src_vpc.subnetwork_name,
      jsonPayload.connection.src_ip,
      jsonPayload.connection.src_port,
      jsonPayload.connection.dest_ip,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      ORDER BY
      bytes DESC
      LIMIT
      15' | bq query --nouse_legacy_sql
      
```

```      
export TABLE_NAME=$(echo bq_vpcflows.$(bq query --nouse_legacy_sql --format=prettyjson 'SELECT * FROM bq_vpcflows.INFORMATION_SCHEMA.TABLES' | grep table_name | cut -d":" -f2 | cut -d"," -f1 | cut -d'"' -f2)) && \
echo 'SELECT
      jsonPayload.connection.src_ip,
      jsonPayload.connection.dest_ip,
      SUM(CAST(jsonPayload.bytes_sent AS INT64)) AS bytes,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      FROM ' \
      ${TABLE_NAME} \
      ' WHERE jsonPayload.reporter = "DEST"
      GROUP BY
      jsonPayload.connection.src_ip,
      jsonPayload.connection.dest_ip,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      ORDER BY
      bytes DESC
      LIMIT
      15' | bq query --nouse_legacy_sql
      
      ````
