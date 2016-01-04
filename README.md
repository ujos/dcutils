#Dashcam utilities

This repository documents how the [Street Guardian SG9665GC
dashcam](https://streetguardian.info/sg9665gc-full-hd-gps-dvr.html)
embeds GPS logs within the Quicktime MOV files it generates. It also
contains a quick and dirty script to extract these logs into a GPX
file.

This information is just my best guess, looking at files generated by
an SG9665GC running the `SG20151109.V2` firmware. No guarantees about
its accuracy, and YMMV.

##Extracting GPX logs

I haven't provided a binary distribution - this stuff is a bit geeky
and probably best suited for programmer types at this point.

1. [Install go](https://golang.org/dl/) if you don't already have
it.
2. Install the script by running
```
$ go install github.com/kbsriram/dcutils/go/cmd/sggps
```
3. To extract logs, run the `sggps` command on `.MOV` files generated by
the dashcam.
```
$ $GOPATH/bin/sggps *.MOV
```
This creates a `.gpx` file alongside each `.MOV` file.


##SG9665GC GPS format

This dashcam generates
[MOV](https://developer.apple.com/library/mac/documentation/QuickTime/QTFF/QTFFPreface/qtffPreface.html)
containers, and stores GPS data once a second. The data is stored in a
"free" atom following each audio chunk. (The audio is split
into one second chunks.)

Two "free" atoms are written immediately following each audio chunk,
and the GPS data is located in the second "free" atom.

I could not figure how to get a direct offset to the GPS atom, but the
sizes of the audio and free atom chunks seem to be constant. So a
quick hack is to locate the start of each audio chunk, and skip
forward 64k bytes to locate the free atom containing the GPS data.

This go struct describes the format of the GPS data within the body of
the atom.

```
// Written as little endian
type GPSLog struct {
	Magic         [4]byte // 'GPS '
	_             [36]byte // Don't know what this is
	Hour          uint32 // 0-23
	Min           uint32
	Sec           uint32
	Year          uint32 // (year - 2000)
	Mon           uint32 // 1-12
	Day           uint32 // 1-31
	Unknown       byte   // I see 'A' in my data
	LatitudeSpec  byte   // Unsure - 'N/S' for north/south?
	LongitudeSpec byte   // Unsure - 'E/W' for east/west?
	_             byte   // Don't know - always 0
        // lat/long are written as IEEE 784 32-bit floats
        // This float is always positive, expressed as
        // dddmm.mmmmmm
        // dd is the degree (multiplied by 100)
        // mm.mmmmmm is a decimal minute (0-59.999999)
	Latitude      float32 
	Longitude     float32 
	Speed         float32 // knots
	Bearing       float32 // if the speed is very low, writes a 0
}
```

##Licensing

Copyright 2016 KB Sriram

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
