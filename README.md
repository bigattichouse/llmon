# 🍋-llmon
LLM-based syslog and journalctl monitoring and analysis.

# LLMon syslog and journalctl
LLMon is a tool that can diagnose syslog and journalctl entries with a mix of inference and RAG activity.  

# Retrieval Augmented Generation (RAG)
The RAG entries come from the dataset (https://github.com/bigattichouse/llmon_dataset)[https://github.com/bigattichouse/llmon_dataset] created by analyzing open source projects and locating possible error messages that could be surfaced in the code. The LLM analyzes each potential message during creation of the dataset to determine the reason for the message, as well as possible resolutions. This data is stored in JSON files in the llmon_dataset folder, so it can be easily updated with git pulls. It can also be limited by creating a custom branch and only pulling the projects you need.

Here's an example from the `apache/httpd` project in the `mod_proxy.c` file:
```json
  {
    "function": "proxy_connect_handler",
    "message_template": "DNS lookup failure for: ",
    "reason": "Logs a detailed error message when DNS lookup fails for the specified hostname",
    "resolution":"Review DNS settings and resolve any issues related to host resolution"
  }
```
# Pipeline

1. Read a batch of `journalctl` or `systemd` logs into a log snapshot
2. (possibly) filter to specific services (or ignore certain services)
3. generate embeddings for each entry (or the whole block)
4. lookup embeddings for related modules (TBD: per JSON object? or the whole thing?)
5. run inference to decide what's happening

# Enhancements
We're also creating embeddings and summaries of each model, to hopefully provide more detail for the inferring LLM to find and retrieve background info about a particular block of code.  These are basic at the moment "Summarize this file", but I believe we can alter them to be more to creating a summary designed to help a troubleshooter. So we might have multiple RAG steps and agents discussing a particular stretch of log entries.

# Deficiencies
Currently, I'm running on a local LLM that can handle a 32k context size (may 5G of VRAM and 64G of system RAM). This is absolutely missing some major context as that window slides over the input content. Once I'm able to run on something like an H100, perhaps I can find 128k or larger context system... but I think I should try some chunking strategies before I spend that much money on a service.  Inference of apache/httpd is likely going to take a few days in my setup.  The first dataset will be a bit clunky, but I imagine will still be useful to the AI reading the logs.


