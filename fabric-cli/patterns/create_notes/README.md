# Create Notes

Generate detailed and accurate notes from a transcript or an article covering a technical topic.

## Usage

```bash
cat transcript.txt | fabric -p create_notes
```


## Example

!!!: Still working on / experimenting with this pattern.
- It seems like when I capture the transcript first and then `cat` it into the pattern, I get a better result.
- The best notes I generated was from The OpenWebUI using the chatgpt-4o-latest model.
- The second best notes I generated were from Copilot.  In fact, this pattern is supposed to imitate that prompt.


### Input:

```bash
fabric --youtube=https://youtu.be/I2mWnh66Bkg | fabric -p create_notes --model gpt-4o | tee 12-logging-best-practices.md
```


### Output:
```
# Summary of High-Level Ideas

- **Importance of Structured and Contextual Logging**: Effective logging requires a structured approach with context to aid in debugging, especially during critical times.
- **Establishing a Logging Strategy**: Define clear objectives for logging, prioritize critical information, and periodically review and adjust your logging practices.
- **Utilizing Log Levels**: Use different log levels (Info, Warning, Error, Fatal) to categorize the severity of events and adjust verbosity as needed.
- **Structured Logging**: Implement structured logging to facilitate easy filtering, searching, and analysis of log data.
- **Log Sampling**: Employ log sampling to manage storage costs, especially in high-traffic systems, while ensuring critical logs are retained.
- **Centralized Log Management**: Aggregate logs from multiple services into a centralized system to simplify troubleshooting and correlation.
- **Retention Policies**: Develop retention policies to manage log storage costs while keeping necessary logs accessible.
- **Log Security**: Protect sensitive data in logs through encryption and access controls and avoid logging sensitive information.
- **Performance Considerations**: Optimize logging to minimize performance impacts and use efficient logging libraries.
- **Metrics vs. Logs**: Use logs for debugging and metrics for real-time monitoring to assess system health.

# Detailed Notes on Key Concepts

## Importance of Structured and Contextual Logging
Logging should not be an afterthought or a haphazard activity. Without structure and context, logs become difficult to interpret, especially during a production issue at odd hours. Structured logging provides clarity and enables quick identification of problems.

## Establishing a Logging Strategy
- **Game Plan**: Before logging, consider the application's main goals, critical operations, and key performance indicators (KPIs).
- **Error Logging**: Aim to provide sufficient context to resolve issues, not just indicate failure.
- **Periodic Review**: Regularly assess logging practices to filter out noise and focus on capturing relevant information.

## Utilizing Log Levels
- **Info**: Routine operations like successful logins or transactions.
- **Warning**: Indicators of potential issues, e.g., slow payment processing.
- **Error**: Significant problems such as failed payments or crashed services.
- **Fatal**: Critical failures that may crash the application.

Adjust log levels according to the situation, increasing verbosity for troubleshooting.

## Structured Logging
Structured logs, unlike basic text logs, categorize information into fields, enhancing machine-readability and enabling advanced filtering and analysis. Utilize logging frameworks or tools like Vector to implement structured logging.

## Log Sampling
- **Purpose**: To reduce storage costs by capturing only a representative sample of logs.
- **Selective Sampling**: Prioritize critical logs over routine ones, especially during error spikes or high-traffic scenarios.
- **Implementation**: Use frameworks like OpenTelemetry for built-in sampling capabilities.

## Centralized Log Management
Aggregate logs from various services to a centralized location. This approach simplifies the process of diagnosing issues across different services by allowing comprehensive searches and correlations.

## Retention Policies
Develop strategies for managing log storage:
- **Recent Logs**: Keep them readily available for immediate debugging.
- **Older Logs**: Move to cheaper storage.
- **Log Deletion**: Remove logs that are no longer necessary.
Tailor retention periods based on log types, e.g., error logs vs. debug logs.

## Log Security
- **Encryption**: Protect logs both in transit and at rest.
- **Access Controls**: Restrict log access based on user roles and responsibilities.
- **Sensitive Data**: Avoid logging sensitive information like passwords or personal data. Use filters to redact sensitive data before it reaches storage.

## Performance Considerations
- **Efficient Libraries**: Choose performant logging libraries (e.g., Go's `slog` over `logus`).
- **Sampling**: Use sampling in high-traffic areas to reduce the load.
- **Separate Disk Partition**: Log to a separate partition to avoid impacting application performance.
- **Load Testing**: Identify and address logging bottlenecks early.

## Metrics vs. Logs
- **Logs**: Best for detailed post-incident analysis and debugging.
- **Metrics**: Provide real-time insights into system health and trends.
Developers should use both logs and metrics to maintain robust observability.

By implementing these best practices, developers can significantly enhance their logging strategies, making system maintenance and issue resolution much more efficient and effective.
```

