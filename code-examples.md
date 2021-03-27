# Примеры кода

## Ntp Client

```csharp
#!/usr/bin/env vala --pkg=Posix
struct NtpPacket
{
    uint8 li_vn_mode;      // Eight bits. li, vn, and mode.
                                // li.   Two bits.   Leap indicator.
                                // vn.   Three bits. Version number of the protocol.
                                // mode. Three bits. Client will pick mode 3 for client.

    uint8 stratum;         // Eight bits. Stratum level of the local clock.
    uint8 poll;            // Eight bits. Maximum interval between successive messages.
    uint8 precision;       // Eight bits. Precision of the local clock.

    uint32 rootDelay;      // 32 bits. Total round trip delay time.
    uint32 rootDispersion; // 32 bits. Max error aloud from primary clock source.
    uint32 refId;          // 32 bits. Reference clock identifier.

    uint32 refTm_s;        // 32 bits. Reference time-stamp seconds.
    uint32 refTm_f;        // 32 bits. Reference time-stamp fraction of a second.

    uint32 origTm_s;       // 32 bits. Originate time-stamp seconds.
    uint32 origTm_f;       // 32 bits. Originate time-stamp fraction of a second.

    uint32 rxTm_s;         // 32 bits. Received time-stamp seconds.
    uint32 rxTm_f;         // 32 bits. Received time-stamp fraction of a second.

    uint32 txTm_s;         // 32 bits and the most important field the client cares about. Transmit time-stamp seconds.
    uint32 txTm_f;         // 32 bits. Transmit time-stamp fraction of a second.
}

void main()
{
    const string hostname = "pool.ntp.org";
    const uint16 portno = 123; // NTP
    
    var packet = NtpPacket();
    assert( sizeof( NtpPacket ) == 48 );
    packet.li_vn_mode = 0x1b;

    unowned Posix.HostEnt server = Posix.gethostbyname( hostname ); // Convert URL to IP.
    if ( server == null )
    {
        error( @"Can't resolve host $hostname: $(Posix.errno)" );
    }
    print( @"Found $(server.h_addr_list.length) IP address(es) for $hostname\n" );
    
    var address = Posix.SockAddrIn();
    address.sin_family = Posix.AF_INET;
    address.sin_port = Posix.htons( portno );
    Posix.memcpy( &address.sin_addr, server.h_addr_list[0], server.h_length );
    var stringAddress = Posix.inet_ntoa( address.sin_addr );    
    print( @"Using $hostname IP address $stringAddress\n" );
    
    var sockfd = Posix.socket( Posix.AF_INET, Posix.SOCK_DGRAM, Posix.IPProto.UDP ); // Create a UDP socket.
    if ( sockfd < 0 )
    {
        error( @"Can't create socket: $(Posix.errno)" );
    }
    var ok = Posix.connect( sockfd, address, sizeof( Posix.SockAddrIn ) );
    if ( ok < 0 )
    {
        error( @"Can't connect: $(Posix.errno)" );
    }

    var written = Posix.write( sockfd, &packet, sizeof( NtpPacket ) );
    if ( written < 0 )
    {
        error( "Can't send UDP packet: $(Posix.errno)" );
    }
    
    var received = Posix.read( sockfd, &packet, sizeof( NtpPacket ) );
    if ( received < 0 )
    {
        error( "Can't read from socket: $(Posix.errno)" );
    }
    
    packet.txTm_s = Posix.ntohl( packet.txTm_s ); // Time-stamp seconds.
    packet.txTm_f = Posix.ntohl( packet.txTm_f ); // Time-stamp fraction of a second.
    const uint64 NTP_TIMESTAMP_DELTA = 2208988800ull;
    time_t txTm = ( time_t ) ( packet.txTm_s - NTP_TIMESTAMP_DELTA );
    var str = Posix.ctime( ref txTm );    

    print( @"Current UTC time is $str" );
}
```



## Weather Client

```csharp
#!/usr/bin/env vala --pkg=gio-2.0

void main()
{
    var host = "api.apixu.com";
    var port = 80;
    var key = "f4bddf887be54dc188a111908181801";
    var city = "Neu-Isenburg";
    var query = @"/v1/current.json?key=$key&q=$city";
    var message = @"GET $query HTTP/1.1\r\nHost: $host\r\n\r\n";

    try
    {
        var resolver = Resolver.get_default();
        var addresses = resolver.lookup_by_name( host, null );
        var address = addresses.nth_data( 0 );
        stderr.printf( @"Resolved $host to $address\n" );

        var client = new SocketClient();
        var addr = new InetSocketAddress( address, port );
        var conn = client.connect( addr );
        stderr.printf( @"Connected to $host\n" );

        conn.output_stream.write( message.data );
        stderr.printf( @"Wrote request $message\n" );

        var response = new DataInputStream( conn.input_stream );
        var status_line = response.read_line( null ).strip();
        stderr.printf( @"Got status line: '$status_line'\n" );

        if ( ! ( "200" in status_line ) )
        {
            error( "Service did not answer with 200 OK" );
        }

        var headers = new HashTable<string,string>( str_hash, str_equal );
        var line = "";
        while ( line != "\r" )
        {
            line = response.read_line( null );
            var headerComponents = line.strip().split( ":", 2 );
            if ( headerComponents.length == 2 )
            {
                var header = headerComponents[0].strip();
                var value = headerComponents[1].strip();
                headers[ header ] = value;
                stderr.printf( @"Got Header: $header = $value\n" );
            }
        }
        var contentLength = headers[ "Content-Length" ].to_int();

        var jsonResponse = new uint8[ contentLength ];
        size_t actualLength = 0;
        response.read_all( jsonResponse, out actualLength );
        stderr.printf( @"Got $contentLength bytes of JSON response: %s\n", jsonResponse );
        stdout.printf( @"%s", jsonResponse );

    }
    catch (Error e)
    {
        stderr.printf( @"$(e.message)\n" );
    }
}
```

