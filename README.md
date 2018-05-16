# docker-elasticsearch-5.2.2-cgroups2
Fix support of cgroup version 2 hierarchy in ES 5.2 (https://github.com/elastic/elasticsearch/issues/23486).

ES 5.2 have problem during the startup on newer kernels (e.g. the default kernel of Ubuntu 18.04 LTS) that supports cgroups v2 (https://www.kernel.org/doc/Documentation/cgroup-v2.txt).

The exception when starting ES is:
```
[2018-05-14T11:51:42,338][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.IllegalStateException: No match found
    at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:125) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:112) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.cli.SettingCommand.execute(SettingCommand.java:54) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:122) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.cli.Command.main(Command.java:88) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:89) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:82) ~[elasticsearch-5.2.2.jar:5.2.2]
Caused by: java.lang.IllegalStateException: No match found
    at java.util.regex.Matcher.group(Matcher.java:536) ~[?:1.8.0_92-internal]
    at org.elasticsearch.monitor.os.OsProbe.getControlGroups(OsProbe.java:216) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.monitor.os.OsProbe.getCgroup(OsProbe.java:414) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.monitor.os.OsProbe.osStats(OsProbe.java:466) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.monitor.os.OsService.<init>(OsService.java:45) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.monitor.MonitorService.<init>(MonitorService.java:45) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.node.Node.<init>(Node.java:345) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.node.Node.<init>(Node.java:232) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.bootstrap.Bootstrap$6.<init>(Bootstrap.java:241) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:241) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:333) ~[elasticsearch-5.2.2.jar:5.2.2]
    at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:121) ~[elasticsearch-5.2.2.jar:5.2.2]
    ... 6 more
```

this is because the cgroups in the new kernel is:
```
> cat /proc/self/cgroup 
(...)
1:name=systemd:/user.slice/user-1000.slice/user@1000.service/gnome-terminal-server.service
0::/user.slice/user-1000.slice/user@1000.service/gnome-terminal-server.service
```

the last one `0::/user.sli(...)` does not match the regex `"\\d+:([^:]+):(/.*)"` found on https://github.com/elastic/elasticsearch/blob/v5.2.2/core/src/main/java/org/elasticsearch/monitor/os/OsProbe.java#L191

There is already a bug about this: https://github.com/elastic/elasticsearch/issues/23486 and it `wontfix` in `5.2` (it is fixed in `5.3.1`).

Actually this container just applies the patch (https://github.com/elastic/elasticsearch/commit/ae6331f27e9237c0fbdf2a7a175026fbf91fccd7) into tag `v5.2`

# Build
 - `docker build -t "elasticsearch-5.2-cgroups2" .`

# Use
 - You can use it just like regular `docker.elastic.co/elasticsearch/elasticsearch:5.2.2` image: `docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" ptanov/elasticsearch-5.2-cgroups2:5.2.2`
 - For more information: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

# TODO
 - use ARG (when https://github.com/docker/hub-feedback/issues/508 is fixed and ARG is supported in hub.docker.com)

