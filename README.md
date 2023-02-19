# 🦈 EffiShark

## A straightforward packet conversation capture tool

If you want to keep a log of which of your devices may have been involved in suspicious outbound activities, effishark will keep a very lean log of all conversions. Every minute it will track all outbound connections with the respective source MAC address, the destination IP and the PORT. This should be the minimal information needed to identify, from which device some suspicious activity started (e.g. lots of SMTP spamming, connecting to a blacklisted IP, etc.).

## Installation

We installed EffiShark on a Raspberry Pi 4 with a 32GB SD card, which should be sufficient to capture a few days worth of traffic.

```
  sudo apt-get install wireshark -y # Select YES to allow capturing by non-root users
  sudo useradd -G wireshark your-user
  pip install pyshark
```

Add the following line to your crontab with `crontab -e`

```
  # Run EffiShark on boot, interface: eth0, sqlite file: ~/shark.db, buffer: 60 seconds
  @reboot /path/to/effishark -i eth0 -f ~/shark.db -b 60
```

If you like, there is a systemd service template provided in `effishark.service`. You can install it like this:

```
  sudo cp ./effishark.service /etc/systemd/system/
  sudo systemctl daemon-reload
  sudo systemctl enable effishark
  sudo service effishark start
```

## Running

```
  ./effishark --help

  usage: effishark [-h] [-f FILE] [-b BUFFER] [-i INTERFACE] [--after AFTER] [--before BEFORE] [--device DEVICE]

  options:
    -h, --help            show this help message and exit
    -f FILE, --file FILE  Path to the sqlite database file
    -b BUFFER, --buffer BUFFER
                          How many seconds of traffic to aggregate
    -i INTERFACE, --interface INTERFACE
                          Network interface (for Macs: 'en0')
    --after AFTER         Do not capture, only display conversations starting after 'yyyy-mm-dd hh:mm'
    --before BEFORE       Do not capture, only display conversations before 'yyyy-mm-dd hh:mm'
    --device DEVICE       Do not capture, only display conversations from 'mac:add:ress'
    --clean DAYS          Remove all conversations which are older than DAYS days
```

In order to query for all conversations during a certain timeframe:

```
  effishark -f ./shark.db --device ac:c9:06:13:72:9a --after "2023-02-18 12:00"--before "2023-02-18 12:01"
```

If you like to regularly delete old conversations (a good practice to minimize personal data storage), add the following cronjob

```
  # Run at midnight and clean effishark conversations older than 7 days
  0 0 * * * /path/to/effishark --clean 7
```

## Caveats

- We may miss a few packets during dumping to the database, not exactly sure how `pyshark.sniff_continuously()` is implemented
- During our testing, `pyshark.sniff_continuously()` sometimes randomly got stuck and would not return further packages. We implemented a regular restart job
