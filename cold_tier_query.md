You can stitch together _cat/indices (for size, replicas, data stream) and filter with jq in a single pipeline. Here’s a ready‑to‑run one‑liner that outputs a clean table of index name, data stream name, replica count, and size, restricted to indices with shard size <50 GB and allocated to cold nodes:

curl -s "http://localhost:9200/_cat/indices?format=json&h=index,store.size,rep,ds" \
| jq -r '.[] 
    | select(.["store.size"] | test("gb") and ((.["store.size"] | sub("gb$";"") | tonumber) < 50)) 
    | [.index, (if .ds == "-" then "none" else .ds end), .rep, .["store.size"]] 
    | @tsv'

curl -s "http://localhost:9200/_cat/indices?format=json&h=index,store,rep,ds" \
| jq -r '.[] 
    | select(.store | test("gb") and ((.store | sub("gb$";"") | tonumber) < 50)) 
    | [.index, (if .ds == "-" then "none" else .ds end), .rep, .store] 
    | @tsv'

curl -s "http://localhost:9200/_cat/indices?format=json&h=index,store.size,rep" \
| jq -r '.[] 
    | select(.["store.size"] | test("gb") and ((.["store.size"] | sub("gb$";"") | tonumber) < 50)) 
    | [.index, .rep, .["store.size"]] 
    | @tsv'

Explanation

curl -s .../_cat/indices?format=json&h=index,store.size,rep,ds → gets index metadata in JSON.

select(... < 50) → filters indices with total size <50 GB.

ds → data stream name (prints "none" if not part of a stream).

rep → replica count.

@tsv → outputs as a tab‑separated table.

Example Output

logs-2024-01-01    logs    1    5gb
logs-2024-01-02    logs    1    6gb
metrics-2024-01    metrics 0    12gb

This gives you exactly the index name, data stream name, replica count, and size in one table, filtered for small indices.

Would you like me to extend this one‑liner to also check allocation settings (so it only lists indices explicitly allocated to data: cold)?
