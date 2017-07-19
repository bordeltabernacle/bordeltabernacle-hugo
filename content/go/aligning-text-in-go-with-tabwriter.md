+++
topics = ["Go","golang","programming"]
tags = ["go","golang"]
date = "2017-07-19T19:52:07+01:00"
title = "Aligning Text in Go with text/tabwriter"
draft = true

+++

I wrote a small tool in Go, called [Trawl](https://github.com/robphoenix/trawl)
that prints out various information about the network interfaces on your
machine. The output is in a columnar fashion, much like a csv file, as so:

```sh
Name     IPv4 Address   IPv4 Mask      IPv4 Network    MTU   MAC Address        IPv6 Address
----     ------------   ---------      ------------    ---   -----------        ------------
wlp1s0   192.168.1.197  255.255.255.0  192.168.1.0/24  1500  10:02:b5:e4:de:8c  fe80::4043:757d:9f01:5082/64
docker0  172.17.0.1     255.255.0.0    172.17.0.0/16   1500  02:42:c8:0d:e0:f7  -
```

Getting the output into that format has gone through a few iterations; firstly
static, then more dynamic and then I stumbled across Go's
[`tabwriter`](https://golang.org/pkg/text/tabwriter/) package, which made
everything better.

Initially all the column widths were static. An IP address will be at most 15
characters wide, `192.168.100.200`, a MAC address will be 17,
`10:02:b5:e4:de:8c`, etc. etc.

More problematic were the interface names. I built the tool to work on Windows as well as Unix based systems, and Windows
interfaces have ridiculously long, but obvious, names like `Wireless Network Connection`,
rather than something short and less obvious like `wlp1s0`, the wireless
interface on my Ubuntu laptop. So I had to consider this variance in length, and
did so by just counting the longest interface names I encountered and setting
the width to that, which didn't take into interfaces I hadn't or wouldn't
encounter. I know, I know, this is awful design/code, and did lead to [issues](https://github.com/robphoenix/trawl/commit/99d61aceb9db64098db3e6ce254dcce0257bcfcb). Anyway, it looked something like this;

The `String` method used for printing:

```go
func (iface *Interface) String() string {
	ifaceString := osString()
	return fmt.Sprintf(
		ifaceString,
		iface.Name,
		iface.IPv4Addr,
		iface.IPv4Mask,
		iface.IPv4Network,
		strconv.Itoa(iface.MTU),
		iface.HardwareAddr,
		iface.IPv6Addr,
	)
}
```

The `osString` function to determine what operating system the tool is being
used on:

```go
func osString() (s string) {
	switch os {
	case win:
		s = windowsString
	case linux:
		s = linuxString
	case darwin:
		s = darwinString
	}
	return
}
```

The static strings:

```go
windowsString     = "%-35s  %-15s  %-15s  %-18s  %-5s  %-17s  %s\n"
linuxString       = "%-10s  %-15s  %-15s  %-18s  %-5s  %-17s  %s\n"
darwinString      = "%-10s  %-15s  %-15s  %-18s  %-5s  %-17s  %s\n"
```

And where the print statements:

```go
ifaces := getIfaces(loopback)

for _, iface := range ifaces {
    i, err := New(iface)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf(i.String())
}
```

Horrible I know, but it got the tool built. Thankfully, refactoring happened.

I knew when writing this that it just would not do, so next iteration I got
dynamic. I realised that it was the interface names that were the main problem,
the element I didn't know in advance, and so couldn't predict, and that differed
across operating systems. So I left the other columns with static numbers and
spent more time than I would have liked working out how to format the string to
use a width not known in advance. This was kinda buried in the `fmt` package
[documentation](https://golang.org/pkg/fmt/) and took a bit of messing around on the Go Playground to get
it right, maybe I'm just slow on the uptake (person shrugging emoji).

> Either or both of the flags may be replaced with the character '*', causing their values to be obtained from the next operand, which must be of type int.

Let's have [an example](https://play.golang.org/p/zbR3If0Zku):

```go
name := "rob"

// here we're saying we want the width to be 10 characters
// and we're going to pad with zeroes just so it's visually clearer
s1 := "%010s\n"
fmt.Printf(s1, name)

// now we're going to replace the 10 with an asterisk and provide the
// width we want in the Printf statement
s2 := "%0*s\n"
// Note the width comes first, to build the verb before it can accept
// the string
fmt.Printf(s2, 15, name)

// output
// 0000000rob
// 000000000000rob
```

Okay, easy enough, when you know how. So, armed with this knowledge I added an extra for loop to find the max width of all the interface names and [committed](https://github.com/robphoenix/trawl/commit/fd3899f562fb296ba67676608b9cb80e97819cee). My code now looked something like so:

The new dynamic string, the only one, no matter what your OS:

```go
outputString := "%-*s  %-15s  %-15s  %-18s  %-5s  %-17s  %s\n"
```

I replaced the `String` method with this function:

```go
func ifaceString(l int, i *Iface) string {
	return fmt.Sprintf(
		outputString,
		l,
		setMissingValue(i.Name),
		setMissingValue(i.IPv4Addr),
		setMissingValue(i.IPv4Mask),
		setMissingValue(i.IPv4Network),
		setMissingValue(i.MTU),
		setMissingValue(i.HardwareAddr),
		setMissingValue(i.IPv6Addr),
	)
}
```

`setMissingValue` just replaced an empty string with a `-`, and l is the maximum
string length, passed in here:

```go
var maxLen int
var ifs []*Iface

for _, iface := range getIfaces(loopback, filter) {
    i, err := New(iface)
    if err != nil {
        log.Fatal(err)
    }
    ifs = append(ifs, i)
    if nameLen := len(i.Name); nameLen > maxLen {
        maxLen = nameLen
    }
}

for _, i := range ifs {
    fmt.Printf(ifaceString(maxLen, i))
}
```

Super, I don't have to have sleepless nights now wondering if someone somewhere
is cursing me because they're output is ugly because the have a network
interface with a very very long name. Yay, win for software.

And then I somehow, don't remember how, found `tabwriter`, and all was good.

> Package tabwriter implements a write filter (tabwriter.Writer) that translates tabbed columns in input into properly aligned text.

This basically means it takes care of the boring formatting stuff for you. With
`tabwriter` you create a new writer, fill it with all your columns of data and
then flush it out, all nicely formatted. Sweet, shoulders of giants and all
that.

So, what does this mean? Well, I reverted back to having a `String` method but
replaced my static/semi-dymanic string with a simple tab separated one, like so:

```go
func (i *Iface) String() string {
	return fmt.Sprintf("%s\t%s\t%s\t%s\t%s\t%s\t%s",
		setMissingValue(i.Name),
		setMissingValue(i.IPv4Addr),
		setMissingValue(i.IPv4Mask),
		setMissingValue(i.IPv4Network),
		setMissingValue(i.MTU),
		setMissingValue(i.HardwareAddr),
		setMissingValue(i.IPv6Addr),
	)
}
```

I then create a `tabwriter.NewWriter`, load it up with all the interfaces, and
column headers (names) if needs be, and then flush it all out:

```go
w := tabwriter.NewWriter(os.Stdout, 0, 0, 2, ' ', 0)

if names {
    fmt.Fprintln(w, tabbedNames())
}

for _, iface := range validInterfaces(loopback, filter) {
    i, err := New(iface)
    if err != nil {
        log.Fatal(err)
        return
    }
    fmt.Fprintln(w, i)
}
w.Flush()
```







