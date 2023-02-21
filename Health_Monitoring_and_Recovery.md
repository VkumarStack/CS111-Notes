# Health Monitoring and Recovery
- Deadlock detection in real systems is not feasible to implement due to difficulties associated with identifying all blocked processes, determining the resources of blocked processes and their ownership, and determining if the associated dependency graph contains any loops
    - Even if deadlock *is* detected, responding to such deadlock is also difficult since killing an arbritary process to stop the deadlock may be incredibly dangerous
## Health Monitoring
- A **health monitoring** approach involves checking whether the system is making any progress
    - An internal monitoring agent can watch message traffic or a transaction log to determine if work is continuing
        - It may be difficult to monitor this log at a reasonable rate
    - Servers can send periodic heart-beats to the central monitoring service
        - Heartbeats only indicate whether the application is *running*, however, and does not give any information about whether the application is properly serving requests
    - An external health monitoring service can occassionally send test requests to the service that is being monitored to determine if they are being responded to in a timely and correct manner
        - This may not indicate whether *other* requests are being served or are wedged, however
- Most systems use a combination of these approaches, usually with the first line of monitoring being an internal monitoring agent, which sends heart-beats, followed by an external test service
## Managed Recovery
- Services should be designed for restart, recovery, and fail-over in instances where monitoring has determined that they are not working properly
    - Software should be designed to be able to be killed and restarted at any time, with the expectation of resuming with minimal disruption after the restart
    - Software should be able to support multiple levels of restart
        - A warm-start simply involves restoring from the last saved state (i.e. from a database)
        - A cold-start involves ignoring any saved state and starting from scratch
    - Software should be able to restart in different *scopes*
        - A single process or multiple processes could only be restarted
        - A single node or multiple nodes could only be restarted
    - Restarts may also be necessary for upgrades
        - With rolling upgrades, it is common for some nodes to be taken down, updated, and reintegrated while other nodes continue to run
            - This requires mechanisms to fall back if an update does not work as well as backwards compatibility so that new updates are able to work in tandem with old updates
    - Periodic restarts may be necessary to ensure efficiency, as some systems may become slower the longer they run (due to issues such as unforseen memory leaks)
## False Reports
- With *external monitoring*, there is the question of false reports on the health of a particular node
    - With a heartbeat approach, for example, it is entirely possible that a missed heartbeat is not necessarily due to failure but rather a network delay 
    - It is important, then, to account for when to actually declare failure - as restarting a node is expensive on false reports, but allowing a real failure to continue could also have devastating consequences
- It is ideal for a system (or server, or whatever) to determine failure *internally* and restart itself upon such detection