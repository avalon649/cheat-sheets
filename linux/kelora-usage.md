# Kelora Usage Guide

Kelora is a CLI log processor that transforms unstructured logs into actionable data. It combines pattern matching with embedded Rhai scripting for complex log analysis.

**GitHub:** https://github.com/dloss/kelora
**Docs:** https://kelora.dev

---

## Installation

```bash
# Via Cargo
cargo install kelora

# Via Homebrew
brew tap dloss/kelora && brew install kelora
```

---

## Basic Usage

```bash
# Auto-detect format
kelora app.log

# JSON logs shortcut
kelora -j app.log

# Show only specific fields
kelora -j app.log -k timestamp,level,service,message

# Exclude fields
kelora -j app.log -K service,version

# Filter by log level
kelora -j app.log -l error,warn

# Exclude log levels
kelora -j app.log -L debug,info

# Limit number of lines
kelora -j app.log -n 20

# Brief mode (values only)
kelora -j app.log -b

# Show statistics summary
kelora -j app.log --stats
```

---

## Input Formats

```bash
# Auto-detect
kelora app.log

# Explicit JSON
kelora -f json app.jsonl

# Logfmt
kelora -f logfmt app.log

# Syslog
kelora -f syslog /var/log/syslog

# Plain text columns — ts(2) means timestamp spans 2 columns, *msg captures rest
kelora -f 'cols:ts(2) level *msg' app.log

# Typed columns
echo "200 1234 0.123" | kelora -f 'cols:status:int bytes:int response_time:float'

# CSV / TSV
kelora -f "csv status:int bytes:int duration_ms:int" access.csv
kelora -f "tsv: user_id:int success:bool" access.tsv

# Gzip compressed
kelora -j app.log.gz
```

---

## Output Formats

```bash
# JSON output
kelora -j app.log -J
# or
kelora -j app.log -F json

# CSV output
kelora -j app.log -F csv -k timestamp,level,service,message

# Logfmt output
kelora -j app.log -F logfmt

# Debug / type inspection
kelora -j app.log -F inspect

# Export to file
kelora -j app.log -F json -o output.json
```

---

## Filtering

```bash
# By log level (include)
kelora -j app.log -l error
kelora -j app.log -l error,warn

# By log level (exclude)
kelora -j app.log -L debug,info

# Rhai expression filters
kelora -j app.log --filter 'e.duration_ms > 1000'
kelora -j app.log --filter 'e.level == "ERROR"'
kelora -j app.log --filter 'e.level in ["ERROR", "WARN"] && e.service == "database"'
kelora -j app.log --filter 'e.message.contains("timeout")'
kelora -j app.log --filter 'e.status_code >= 500'
```

---

## Time Filtering

```bash
# Relative time
kelora -j app.log --since 1h
kelora -j app.log --since 30m
kelora -j app.log --since 2d

# Absolute time range
kelora -j app.log --since "2024-01-15T10:00:00Z" --until "2024-01-15T11:00:00Z"

# Time of day range
kelora -j app.log --since "10:00" --until "start+30m"
```

---

## Field Transformations (Rhai Scripting)

```bash
# Add derived field
kelora -j app.log -e 'e.duration_s = e.duration_ms / 1000'

# Uppercase a field
kelora -j app.log -e 'e.level = e.level.to_upper()'

# Classify events
kelora -j app.log -e 'e.severity = if e.level in ["ERROR", "CRITICAL"] { "high" } else if e.level == "WARN" { "medium" } else { "low" }'

# Safe type conversion (with fallback)
echo '{"id":"123","status":"200"}' | kelora -j -e 'e.id_num = e.id.to_int_or(-1)'

# Check field existence
kelora -j app.log -e 'e.slow = if e.has("duration_ms") { e.duration_ms > 1000 } else { false }'

# Combine filter + transform
kelora -j app.log \
  -e 'e.duration_s = e.duration_ms / 1000' \
  --filter 'e.duration_s > 1.0' \
  -k timestamp,service,duration_ms,duration_s
```

---

## Timestamp Operations

```bash
# Normalize timestamps
kelora -j app.log --normalize-ts

# Custom timestamp format
echo '2024-01-15 10:30:45,123 INFO Login' | \
  kelora -f 'cols:timestamp(2) level *message' \
  --ts-field timestamp \
  --ts-format '%Y-%m-%d %H:%M:%S,%3f'

# Timezone conversion
kelora -j app.log --input-tz UTC
kelora -j app.log --input-tz Europe/Berlin

# Extract hour/day from timestamp via script
echo '{"timestamp":"2024-01-15T10:30:45Z"}' | \
  kelora -j -e 'e.hour = meta.parsed_ts.hour(); e.day = meta.parsed_ts.day()'

# Age of each event in minutes
kelora -j app.log -e 'e.age_minutes = (now() - to_datetime(e.timestamp)).as_minutes()'

# Business hours filter (9am–5pm ET)
kelora -j app.log --input-tz America/New_York \
  -e 'e.hour = to_datetime(e.timestamp).hour()' \
  --filter 'e.hour >= 9 && e.hour < 17'
```

---

## Metrics & Aggregation

```bash
# Count events by field value
kelora -j app.log -e 'track_count(e.service)' --metrics

# Sum / min / max
kelora -j app.log -e '
  if e.has("duration_ms") {
    track_sum("total_duration", e.duration_ms);
    track_min("min_duration", e.duration_ms);
    track_max("max_duration", e.duration_ms);
  }
' --metrics

# Percentiles
kelora -j app.log -e 'track_percentiles("latency", e.duration_ms, [0.50, 0.95, 0.999])' --metrics

# Full stats (mean, stddev, percentiles)
kelora -j app.log -e 'track_stats("latency", e.duration_ms, [0.50, 0.90, 0.99])' --metrics

# Top N slowest services
kelora -j app.log -e 'track_top("slowest", e.service, 5, e.duration_ms)' --metrics

# Unique values / cardinality
kelora -j app.log -e 'track_unique("services", e.service)' --metrics

# Histogram / bucketing
kelora -j app.log -e '
  if e.has("duration_ms") {
    let bucket = (e.duration_ms / 1000) * 1000;
    track_bucket("latency_histogram", bucket);
  }
' --metrics

# Save metrics to file
kelora -j app.log -e 'track_count(e.service)' --metrics --metrics-file metrics.json
```

---

## Windowed / Span Processing

```bash
# Count-based spans (every 5 events)
kelora -j app.log -q --span 5 \
  --span-close 'print("Span " + span.id + " had " + span.size.to_string() + " events")'

# Time-based windows (every 1 minute)
kelora -j app.log -q --span 1m \
  --span-close 'print("Window: " + span.id + " (" + span.size.to_string() + " events)")'

# Error rate per window
kelora -j app.log -q \
  --exec 'track_count("requests"); if e.level == "ERROR" { track_count("errors"); }' \
  --span 1m \
  --span-close '
    let m = span.metrics;
    let requests = m.get_path("requests", 0);
    let errors = m.get_path("errors", 0);
    let rate = if requests > 0 { (errors * 100) / requests } else { 0 };
    print(`${span.start.to_iso()}: ${errors}/${requests} errors (${rate}%)`);
  '

# Spike detection
kelora -j app.log -q \
  --exec 'track_count("requests"); if e.level == "ERROR" { track_count("errors"); }' \
  --span 3 \
  --span-close '
    let span_rate = span.metrics.get_path("errors", 0) * 100 / span.size;
    let total_rate = metrics.get_path("errors", 0) * 100 / metrics.get_path("requests", 1);
    if span_rate > total_rate * 2 { eprint(`Spike in ${span.id}: ${span_rate}% (avg: ${total_rate}%)`); }
  '
```

---

## Real-World Pipelines

```bash
# Stream live errors
tail -f app.log | kelora -j -l error

# Kubernetes pod logs, errors only
kubectl logs -f deployment/api | kelora -f json -l error

# Filter and export errors to JSON
kelora -j app.log -l error -F json -o errors.json

# Web access log — tag server errors
kelora -f combined access.log \
  -e '
    let status = e.status.to_int_or(0);
    if status >= 500 { e.family = "server_error"; }
    else if status >= 400 { e.family = "client_error"; }
    else { e.family = "ok"; }
  ' \
  --filter 'e.family != "ok"'

# Parse and filter slow requests from raw logs
cat app.log | kelora -f 'cols:timestamp level service *message' \
  --ts-field timestamp \
  -e 'if e.message.contains("ms") { e.duration = e.message.extract_regex(r"(\d+)ms", 1).to_int() }' \
  --filter 'e.has_path("duration") && e.duration > 1000' \
  -k timestamp,service,duration,message
```

---

## Interactive REPL

```bash
# Launch REPL (no arguments)
kelora

# Then type commands directly:
# -j examples/app.jsonl -l error
# -f logfmt examples/*.log --stats
# :exit
```

---

## Help Reference

```bash
kelora --help            # Full CLI reference
kelora --help-examples   # More usage patterns
kelora --help-rhai       # Rhai scripting guide
kelora --help-functions  # All built-in Rhai functions
kelora --help-time       # Timestamp format reference
```
