# MSL3DVR2NS-migration
Migration tool for Akamai Media Services Live 3 Stream Packaging (DVR) to NetStorage (->Static Packaging)

## Overview
This tool will help you migrate your content delivered today from Akamai Media Services Live 3 Stream Packaging (DVR) Stream Packaging to your prefered Netstorage (Akamai Storage) group.  
As MSL3+SP is part of an End Of Life program, you have few months left to migrate your LIVE DVR library.  
The current workflow is quite simple, it leverages Netstorage as the downloader:  

* The input is an Akamai Stream Packaging MSL3+DVR stream url (legacy MSL3+DVR)  
* We inspect the manifests (master and child). 
* We modify the urls as relative  
* We send commands to the destination NetStorage to download all the segments, so that we do not have to download locally and upload again (less bandwidth consumption then).  

## Requirements

* Perl script to be run from a terminal (Linux/Mac...) on a server/laptop
* Akamai MSL3 Stream Packaging LIVE (MSL3+SP+DVR) stream url as HLS matching:  

```
https://myHostname-lh.akamaihd.net/i/myPath/myFilename_angle@streamID/master.m3u8
```

* A Netstorage configuration on Akamai platform, that will be used as the destination
* A Netstorage upload account on that same Netstorage with write access
* A SSH key deployed on that upload account and on the server/laptop running the script

## Disclaimer

That tool is not officially supported by Akamai Technologies, so use it at your own risk.  
**WARNING: using this tool might impact your costs on Akamai platform because of the footprint of the static assets migrated to the destination Netstorage!**

For instance, if you have a list of 4 bitrates, so 4 mp4 on the Netstorage source, we will generate all the m3u8 and segments on the destination Netstorage. You can easily double the footprint of the source mp4 with the ts segments generated by Stream Packaging.

Akamai and the team I work for can address custom requirements and also provide support on such large migration in case you do not wish the do it yourself route.

## Downloader Usage

Show inline documentation:

```
perl ./NS-hlsdownloader-LIVE.pl -h
```

All options:

```
perl ./NS-hlsdownloader-LIVE.pl -url=streamUrl -nsHost=NetStorage_Hostname -nsPath=NetStorage_Destination_Directory -nsKey=NetStorage_SSH_Public_Key [-tokenKey=key] [-log] [-debug]

Usage:
    perl NS-hlsdownloader-LIVE.pl -url=streamUrl -nsHost=NetStorage_Hostname
    -nsPath=NetStorage_Destination_Directory
    -nsKey=NetStorage_SSH_Public_Key [-tokenKey=key] [-log] [-debug]

      --url,-u       Stream URL
      --nsHost,-h    Netstorage hostname
      --nsPath,-p    Netstorage path
      --nsKey,-k'    Netstorage Key path
      --tokenKey,-t  Token key (optional)
      --debug,-d     Run in debug mode
      --help,-h      Print this help

  Example:
        perl NS-hlsdownloader-LIVE.pl \
            -url="https://myHostname-lh.akamaihd.net/i/myPath/myFilename_angle@streamID/master.m3u8" \
            -nsHost="myAccount.upload.akamai.com" \
            -nsPath="myRootNSdirectory" \
            -nsKey="/Users/myUser/.ssh/netstorage/myPublicNetStorageKey.pub" \
            [-tokenKey=myTokenKey] \
            [-log] \
            [-debug]
```

Output:

- [x] Generate a CSV file output to keep track of what the tool is doing with Action/Url/Status.   


## Batch Downloader Usage

Show inline documentation:

```
perl ./processListOfStreams.pl -h
```

All options:

```
perl processListOfStreams.pl -file=listOfStreamUrls.txt -nsHost=NetStorage_Hostname -nsPath=NetStorage_Destination_Directory -nsKey=NetStorage_SSH_Public_Key [-tokenKey=key] [-log] [-debug] [-preview]

Usage:
    perl processListOfStreams.pl -file=listOfStreamUrls.txt -nsHost=NetStorage_Hostname
    -nsPath=NetStorage_Destination_Directory
    -nsKey=NetStorage_SSH_Public_Key [-tokenKey=key] [-log] [-debug] [-preview]

    --file,-f 	 file to load (stream urls list)
    --start,-s     Start index for resume (optional)
    --end,-e       End index (optional)
    --reverse,-r   Reverse the list file (optional)
    --nsHost,-h    Netstorage hostname
    --nsPath,-p    Netstorage path
    --nsKey,-k'    Netstorage Key path
    --tokenKey,-t  Token key (optional)
    --preview,-p   Run in preview mode without running the download sub-process
    --debug,-d     Run in debug mode
    --help,-h      Print this help

  Example:
        perl processListOfStreams.pl \
        --file="listOfStreams.txt" \
        --nsHost="myAccount.upload.akamai.com" \
        --nsPath="myRootNSdirectory" \
        --nsKey="/Users/myUser/.ssh/netstorage/myPublicNetStorageKey.pub" \
        --preview \
        --start=30 \
        --end=40

        This example will load the text file and select stream urls between lines 30 and 40, if valid, it will display the stream urls as preview without running the downloader command.
        You can restart the command from a specific line in the input file, define an end or reverse the list if needed to start multiple process in different shells
```


## Todo

Some corner cases are currently in development:

- [ ] Generate a batch script to remove source files when the process is over so you could clean up source files on the source NetStorage to reduce the costs.  
- [ ] Migrate Akamai hostnames into AMD config and implement logic to reuse legacy urls on AMD.

Then we will automate the migration and also allow migration of additional assets referenced in the master or child playlists.


## Contributors

* Frederic Beleteau / @Fred3d
* 