set ns [new Simulator]
set tf [open lana.tr w]
$ns trace-all $tf
set nf [open lana.nam w]
$ns namtrace-all $nf

set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]

$ns color 1 "blue"
$n1 label "Source"
$n2 label "Error Node"
$n3 label "Destination"

$ns make-lan "$n0 $n1 $n2" 10Mb 10ms LL Queue/DropTail Mac/802_3
$ns make-lan "$n3 $n4 $n5" 10Mb 10ms LL Queue/DropTail Mac/802_3

$ns duplex-link $n2 $n4 100Mb 10ms DropTail
$ns duplex-link-op $n2 $n4 color "green"

set udp1 [new Agent/UDP]
$ns attach-agent $n1 $udp1
set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp1
set null5 [new Agent/Null]
$ns attach-agent $n5 $null5

$udp1 set class_ 1

$ns connect $udp1 $null5

$cbr1 set packetSize_ 1000
$cbr1 set interval_ 0.001

set err [new ErrorModel]
$ns lossmodel $err $n2 $n4
$err set rate_ 0.1

proc finish { } {
global ns nf tf
$ns flush-trace
exec nam lana.nam &
close $tf
close $nf
exit 0
}
$ns at 0.1 "$cbr1 start"
$ns at 6.0 "finish"
$ns run
