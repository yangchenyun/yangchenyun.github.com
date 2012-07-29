---
layout: post
title: How to Measure and Monitor Network Performance
date: 2012-02-17 15:54
comments: true
keywords: iperf, vnstat, ntop, traffic, LAN, bandwidth, network, speed, measurement
---

When a network is set up it is also necessary to measure the real performance. Does the real WAN bandwidth match ISP's promise? What's the real bandwidth of the 1Gbit ethernet LAN? When is the busiest hours of the network? How the traffic is distributed across different IPs and hosts?

In this article, I will introduce the usage of three different programs to solve above problems.

`iperf` is a easy tool to calculate the raw bandwidth between two machines. `vnstat` could track traffic with a low resource cost. For more detailed information about network usage, `ntop` plays its role well.

* Table of Content
{:toc}

## Measure the Effective Bandwidth with iperf
`iperf` is a program used to perform network throughput tests. It will **move as much data as possible** using available connections established between two computers.

### Installation
![Iperf Install][link_to_iperf_install]

On Linux, you do:

    sudo aptitude install iperf

On Mac OS X, you could install it with [homebrew][link_to_homebrew]{:target="blank"}.

    brew install iperf

On windows, you could download the executables from [linhost.info][link_to_iperf_win].

### Two Commands, One Test
![Iperf mode][link_to_iperf_mode]

You need to have **at least** one pair of `iperf` client and server to perform the test.
To run `iperf` as server:

    iperf -s

To run `iperf` as client:

    iperf -c 192.168.1.87

This is the result table from client's terminal:

    ------------------------------------------------------------
    Client connecting to 192.168.1.87, TCP port 5001
    TCP window size: 65.0 KByte (default)
    ------------------------------------------------------------
    [  4] local 192.168.1.88 port 56752 connected with 192.168.1.87 port 5001
    [ ID] Interval       Transfer     Bandwidth
    [  4]  0.0-10.0 sec   128 MBytes   107 Mbits/sec

By default, `iperf` will establish TCP connection on 5001 port and transfer as much data as possble in 10 seconds.

### More Options
The format of bandwidth could be specified by `-f`, the time to transmit could be specified in `-t` and the interval to send data could be set with `-i`.
For example, to report tests with 10 seconds intervals in 50 seconds under the format of MBytes could be written as:

    iperf -c 192.168.1.87 -t 50 -i 10 -f M

The result likes:

    ------------------------------------------------------------
    Client connecting to 192.168.1.87, TCP port 5001
    TCP window size: 0.06 MByte (default)
    ------------------------------------------------------------
    [  4] local 192.168.1.88 port 56924 connected with 192.168.1.87 port 5001
    [ ID] Interval       Transfer     Bandwidth
    [  4]  0.0-10.0 sec   126 MBytes  12.6 MBytes/sec
    [  4] 10.0-20.0 sec   133 MBytes  13.3 MBytes/sec
    [  4] 20.0-30.0 sec   135 MBytes  13.5 MBytes/sec
    [  4] 30.0-40.0 sec   138 MBytes  13.8 MBytes/sec
    [  4] 40.0-50.0 sec   135 MBytes  13.5 MBytes/sec
    [  4]  0.0-50.0 sec   667 MBytes  13.3 MBytes/sec

Sometimes it is also necessary to measure two-way transmission. You could use `-d` to do bidirectional test at the same time or use `-r` to do each transmission individually.

![Iperf mode][link_to_iperf_mode_d]

![Iperf mode][link_to_iperf_mode_r]

    iperf -c 192.168.1.87 -d
    iperf -c 192.168.1.87 -r

If you want to use run `iperf` as a daemon on the server, you could use `-D`

    iperf -s -D

To specify the port through which the client and server communicate use `-p`

    iperf -s -p 9393
    iperf -c 192.168.1.87 -p 9393

You could find more practical examples [here][link_to_openmaniak_iperf].

## Monitor Bandwith Usage with vnstat

`vnstat` use a **database file to keep a log of network traffic** through certain interfaces. It provide hourly/daily/weekly/monthly traffic information. And as it doesn't sniffing packets, it is very efficient.

To use `vnstat` there are two things needed to be done:
1. Install the program;
2. Update the databases regularly.

### Install Vnstat
Install `vnstat` with these commands:

```bash
# Linux
sudo aptitude install vnstat
# Mac OS
brew install vnstat
```

### Setup Scheduled Update
Now add a scheduled task with `crontab -e` to update the database every five minutes.

    # /etc/cron.d/vnstat: crontab entries for the vnstat package
    0-55/5 * * * *   root    if [ -x /usr/bin/vnstat ] && [ `ls /var/lib/vnstat/ | wc -l` -ge 1 ]; then /usr/bin/vnstat -u; fi

If you run `vnstatd` daemon instead of `vnstat`, there is no need to create the crontab as it will read the [configuration file](#detailed-configuration) and handle the updates accordingly.

### Simple Usage
Now wait a while for the database to be updated and you could see a traffic summary for one interface through:

    vnstat -i eth0

    Database updated: Sat Feb 18 16:30:01 2012

       eth0 since 02/17/2012

              rx:  1.75 GiB      tx:  4.45 GiB      total:  6.20 GiB

       monthly
                         rx      |     tx      |    total    |   avg. rate
         ------------------------+-------------+-------------+---------------
           Feb '12      1.75 GiB |    4.45 GiB |    6.20 GiB |   34.04 kbit/s
         ------------------------+-------------+-------------+---------------
         estimated      2.86 GiB |    7.30 GiB |   10.17 GiB |

       daily
                         rx      |     tx      |    total    |   avg. rate
         ------------------------+-------------+-------------+---------------
         yesterday      1.69 GiB |    4.22 GiB |    5.91 GiB |  573.44 kbit/s
             today     60.19 MiB |  242.29 MiB |  302.48 MiB |   41.71 kbit/s
         ------------------------+-------------+-------------+---------------
         estimated        87 MiB |     352 MiB |     439 MiB |

You could also specify the intervals with `--hours`, `--days`, `--weeks`, `--months`.

    vnstat -i eth0 --days

There is even a live mode triggered by `--live` to display current transfer rate.

    vnstat -i eth0 -l

### Detailed Configuration
`vnstat` use `$HOME/.vnstatrc` for user specific configuration and use `/etc/vnstat.conf` for default configuration.

You could specify the interfaces to use by default, intervals to update data and update the database file etc.

The configuration file is quite self-explained:

    sudo vim /etc/vnstat.conf

### Dump the Data for Scripting
`vnstat` also provides formated output of the database.

If you want a xml formatted database, use `vnstat --xml`.

There is also a parseable format with `vnstat --dumpdb`. The detailed explanation of the output format could be found at `man vnstat`. You could also use this format for traffic summary with `vnstat --oneline`.

You could find more about `vnstat` on [its official site][link_to_vnstat].

## Monitor Detailed Network Traffic with ntop
`vnstat` measures the overall traffic of interfaces but it fails to keep track of individual communications. For more detailed information, `ntop` comes into places.

`ntop` is a **traffic probe** and use a **web interface** to display certain information. It could show traffic distribution across protocols, IPs, hosts and more.

You could install `ntop` with:

    sudo aptitude install ntop

To initial `ntop` the first time run

    sudo ntop

Now the ntop could be accessed through 3000 port from HTTP request, you could use it through a browser.

    open http://127.0.0.1:3000

Along with the detailed information `ntop` collects, it uses much more system resources than `vnstat`.

I am just a beginner of webmaster work, I'd love to hear what's in your toolbox to measure network performance.

[link_to_vnstat]: http://humdi.net/vnstat/ "vnstat official site"
[link_to_openmaniak_iperf]: http://openmaniak.com/iperf.php "iperf detailed usage"
[link_to_homebrew]: mxcl.github.com/homebrew/ "Home Brew - Package Manager on Mac"
[link_to_iperf_win]: http://linhost.info/downloads/apps/iperf.exe "Iperf Executables for Windows"
[link_to_iperf_mode]: /images/iperf-mode.png "iperf mode image"
[link_to_iperf_mode_d]: /images/iperf-mode-d.png "iperf mode in d image"
[link_to_iperf_mode_r]: /images/iperf-mode-r.png "iperf mode in r image"
[link_to_iperf_install]: /images/iperf-install.png "install iperf on both machines"
