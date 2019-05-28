# ccn
# congestion
###############################################
# Name : Nataraju A B
#        Assistant Professor,
#			Dept. of ECE, AIT, Bangalore
############################################### 

# Simulate an Ethernet LAN using ‘n’ nodes 
# and set multiple traffic nodes and plot congestion 
# window for different source / destination. 

#Create Simulator
set myNS [new Simulator]

#Open trace and NAM trace file
set myTraceFile [open out.tr w]
$myNS trace-all $myTraceFile

set myNamFile [open out.nam w]
$myNS namtrace-all $myNamFile

set n0 [$myNS node]
set n1 [$myNS node]
set n2 [$myNS node]
set n3 [$myNS node]
set n4 [$myNS node]
set n5 [$myNS node]
set n6 [$myNS node]
set n7 [$myNS node]

# Label the nodes 
$n0 label "SRC-1"
$n7 label "DST-1"

$n4 label "SRC-2"
$n3 label "DST-2"

# Create a LAN of nodes
$myNS make-lan "$n0 $n1 $n2 $n3 $n4 $n5 $n6 $n7" 3Mb 30ms LL Queue/DropTail Mac/802_3

# Setup a pair of TCP sender and receiver (sender @ n0 , receiver @ n7)
set tcp1 [new Agent/TCP]
$myNS attach-agent $n0 $tcp1

set ftp1 [new Application/FTP]
$ftp1 attach-agent $tcp1

set sink1 [new Agent/TCPSink]
$myNS attach-agent $n7 $sink1

# Connect the TCP Source-1 and SINK-1
$myNS connect $tcp1 $sink1

# Setup another pair of TCP sender and receiver (sender @ n4, receiver @ n3)
set tcp2 [new Agent/TCP]
$myNS attach-agent $n4 $tcp2

set ftp2 [new Application/FTP]
$ftp2 attach-agent $tcp2

set sink2 [new Agent/TCPSink]
$myNS attach-agent $n3 $sink2

# Connect the TCP Source-2 and SINK-2
$myNS connect $tcp2 $sink2

$ftp1 set type_ FTP
$ftp2 set type_ FTP
$tcp1 set class_ 1
$tcp2 set class_ 2

set outFile1 [open congestion1.xg w]
set outFile2 [open congestion2.xg w]

# Populate the TCP window values and write to a file for later plotting
# procedure to plot the congestion window

$myNS at 0.0 "plotWindow $tcp1 $outFile1"

proc plotWindow {tcpSource outfile} {
	global myNS
	set curTime [$myNS now]
	set curCwnd [$tcpSource set cwnd_]

	# the data is recorded in a file called congestion.xg (this can be plotted 		
	# using xgraph or gnuplot. 
	# this example uses xgraph to plot the cwnd_
	puts  $outfile  "$curTime $curCwnd"
	$myNS at [expr $curTime+0.01] "plotWindow $tcpSource  $outfile"
}

$myNS color 1 "red"
$myNS color 2 "green"

# What to do at the end of simulation ???
proc finish { } {
	global myNS myTraceFile myNamFile outFile1 outFile2
	$myNS flush-trace
	close $myTraceFile
	close $myNamFile 

	exec xgraph congestion1.xg congestion2.xg -geometry 400x400 &
#		exec xgraph congestion1.xg  -geometry 400x400 &
#		exec xgraph congestion2.xg -geometry 400x400 &

	exit 0
}

$myNS at 0.0 "plotWindow $tcp1 $outFile1"
$myNS at 0.0 "plotWindow $tcp2 $outFile2"

$myNS at 0.1  "$ftp1 start"
$myNS at 0.1  "$ftp2 start"
$myNS at 49.5 "$ftp1 stop"
$myNS at 49.5 "$ftp2 stop"
$myNS at 50.0 "finish"

# Start the simulation....
$myNS run

######..... END .....#######

#case-1
$myNS at 0.1  "$ftp1 start"
$myNS at 49.5 "$ftp1 stop"

$myNS at 0.1  "$ftp2 start"
$myNS at 49.5 "$ftp2 stop"

#case-2
$myNS at 0.1  "$ftp1 start"
$myNS at 49.5 "$ftp1 stop"

#$myNS at 0.1  "$ftp2 start"
#$myNS at 49.5 "$ftp2 stop"

#case-3
$myNS at 0.1  "$ftp1 start"
$myNS at 49.5 "$ftp1 stop"
$myNS at 0.1  "$ftp2 start"
$myNS at 49.5 "$ftp2 stop"
$myNS at 0.1  "$ftp3 start"
$myNS at 49.5 "$ftp3 stop"

#	puts $outFile1 "#Title Text: congestion window plot"
#	puts $outFile1 "#X Unit Text: time (seconds)"
#	puts $outFile1 "#Y Unit Text: window size"

#	puts $outFile2 "#Title Text: congestion window plot"
#	puts $outFile2 "#X Unit Text: time (seconds)"
#	puts $outFile2 "#Y Unit Text: window size"

#exec xgraph congestion2.xg -geometry 400x400 &
#exec xgraph congestion2.xg -geometry 400x400 &

#exec xgraph congestion1.xg -geometry 400x400 &
#exec xgraph congestion2.xg -geometry 400x400 &
