---
layout: post
title: Git通过代理访问Github
category: Document
tags: git github
year: 2012
month: 05
day: 13
published: true
summary: 公司代理如何拥抱github，使用ssh协议代理喽
image: pirates.svg
comment: true
---

实现方法已更新，参见 2016-07-07 博客

网络代理下，用 git 通过 ssh 协议与 github 仓库对接，就是不好用，因为：

1. https
2. ssh

解决方法：

__ssh协议代理__

前提准备：

- git
- ssh

步骤如下：

1. 配置.ssh/config:

```
$ vi ~/.ssh/config
```

写下如下配置项：

```
Host github.com
Hostname ssh.github.com
Port 443
ProxyCommand ~/.ssh/ssh-https-tunnel %h %p
```

2. 下载ssh-https-tunel，可从http://zwitterion.org/software/ssh-https-tunnel/ssh-https-tunnel下载，保存到~/.ssh下，添加可执行权限。

3. 修改host，port，user，pass：

```
# Proxy details
my $host = "proxy.xxx.com";
my $port = 8000;

# Basic Proxy Authentication - leave empty if you don't need it
my $user = "usr";
my $pass = "passwd";

```

4. 用git 对接你的github仓库吧：

```
$ git clone git@github.com:fooyou/jekyll-bootstrap.git
```

## 其他

也可使用 haproxy

**ssh-https-tunel 源码**

```
#!/usr/bin/perl -T -w
# Copyright (C) 2001,2002,2008 Mark Suter <suter@humbug.org.au>
#
# This program tunnels a secure shell connection via a https proxy as
# the ProxyCommand program.  The destination secure shell server needs
# to be running on port 443 unless the proxy is very lenient.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#
# $Id: ssh-https-tunnel,v 2.8 2008/02/03 06:14:44 suter Exp suter $

use strict;
use IO::Select;
use IO::Socket;

################################
##  Start User Configuration  ##
################################

# Proxy details
my $host = "172.30.50.10";
my $port = 8080;

# Basic Proxy Authentication - leave empty if you don't need it
my $user = "";
my $pass = "";

# Add an entry to your ~/.ssh/config that so "ssh remote.example.org"
# uses this program to proxy the connection.
#
#    host remote.example.org
#        ProxyCommand /path/to/ssh-https-tunnel %h %p
#        Port 443
#        ServerAliveInterval 10
#
# The last option enables Keep Alives to avoid the problem of many
# proxies timing out inactive connections.  Check your ssh client's
# documentation for details.
#
# If you are behind a Microsoft ISA server, or similar proxy that uses
# NTLM, see http://www.google.com/search?q=ntlm+proxy+auth for ideas.

################################
##   End User Configuration   ##
################################

## Based on "MIME::Base64::old_encode_base64" to avoid that dependancy
sub auth_header($$) {
    my ( $user, $pass ) = @_;

    sub encode_base64 ($;$) {
        my $eol = $_[1];
        $eol = "\n" unless defined $eol;

        my $res = pack( "u", $_[0] );

        # Remove first character of each line, remove newlines
        $res =~ s/^.//mg;
        $res =~ s/\n//g;

        $res =~ tr|` -_|AA-Za-z0-9+/|;  # ` help syntax parsers

        # fix padding at the end
        my $padding = ( 3 - length( $_[0] ) % 3 ) % 3;
        $res =~ s/.{$padding}$/'=' x $padding/e if $padding;

        # break encoded string into lines of no more than 76 characters each
        if ( length $eol ) {
            $res =~ s/(.{1,76})/$1$eol/g;
        }
        return $res;
    }

    return "Proxy-Authorization: Basic " . encode_base64( "$user:$pass", "\015\012" );
}

## Tunnel the connection and return a handle for it
sub tunnel_connect($$$$$$) {
    my ( $host, $port, $user, $pass, $remote_host, $remote_port ) = @_;

    my $socket = IO::Socket::INET->new( PeerAddr => $host, PeerPort => $port )
        or die "$0: Can't connect to $host:$port: $!\n";

    $socket->print( "CONNECT $remote_host:$remote_port HTTP/1.0\015\012",
        $user ? auth_header( $user, $pass ) : "", "\015\012" )
        or die "$0: Can't write: $!\n";

    local $/ = "\012";
    my $response = $socket->getline() or die "$0: Can't read: $!\n";
    $response =~ /^HTTP\/... 2/i or die "$0: CONNECT failed: $response";
    do { $response = $socket->getline() or die "$0: Can't read: $!\n"; }
        until $response =~ /^\s+$/;

    return $socket;
}

## Move data from one handle to another
sub proxy_data($$) {
    my ( $source, $destination ) = @_;

    my ( $buffer, $length, $offset, $bytes ) = ( "", 0, 0, 0 );
    $length = sysread( $source, $buffer, 4096, $offset ) or return 0;
    while ($length) {
        $bytes = syswrite( $destination, $buffer, $length, $offset ) or return 0;
        $offset += $bytes;
        $length -= $bytes;
    }
    return 1;
}

## Check we have two arguments
defined $ARGV[0] and defined $ARGV[1] or die "Usage $0 <host> <port>\n";

## Setup the tunnel
my $proxy = tunnel_connect( $host, $port, $user, $pass, $ARGV[0], $ARGV[1] );

## Shift data around in each direction
my $sel = IO::Select->new( [ \*STDIN, $proxy ], [ $proxy, \*STDOUT ] );
SELECT: while ( my @ready = $sel->can_read() ) {
    foreach my $handle (@ready) {
        proxy_data( $$handle[0], $$handle[1] ) or last SELECT;
    }
}


```
