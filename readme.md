# Reliable UDP protocol (ALPHA).

It is support:

 * Constant bandwith limitation per connection.
 * ping/pong logic to be sure that connection still alive
 * data's packet acknowledge (delivery confirmation)
 * reiteration of data packets when acknowledge is not received through some time
 * limitation of attempts of data send. It can close connection, when it has many lost packets.
 * asynchronous sending simple (1024 bytes) UDP packets without any acknowledge
 * sending data packets as messages. So recepient will be fired when it will receive message fully
 * limitation by send packets what are waiting for delivery confirmation
 * internal priority of traffic (QoS). (as examples: ping has real time priority, delivery confiration has high priority, data packets has low priority )
 * sync/async sending of data. Synchronous sending return control of flow when all packets with data are sended (they can be not confirmed)

## Roadmap:
 
 * flexible bandwith for connection. It should consider full bandwith of UDP channel.
 * using STUN to make UDP hole punching ( https://en.wikipedia.org/wiki/UDP_hole_punching )

## Options of application rudp:
p.s. Commented option is disabled, but can be enabled in the future.
```
    % { workers_for_each_port, 1 }, %% Workers for parallel process packets for each port
    { listener_options, [] }, %% Socket listener options. See http://erlang.org/doc/man/gen_udp.html#open-2
    { connection_timeout, 10000 }, %% Connection timeout, ms
    { ping_interval, 5000 }, %% Timeout to check good connection, ms
    { ping_packet_count, 4 }, %% Number of ping packets are sent before disconnection
    { delivery_timeout, 10000 }, %% Delivery timeout to check status of delivery of packet, ms
    { bandwith_max, 100000 }, %% Max bandwith of sender for one connection, bytes/second
    { bandwith_min, 100 }, %% Min bandwith of sender for one connection, bytes/second
    %%{ bandwith_step, 100 }, %% Bandwith step for one lost packet
    { max_attempts_to_send_packet, 500000 }, %% Max attempts to send packet when it can't get delivery confirmation
    %{ bandwith_step_increase, 10000 }, %% Step to increase bandwith, bytes
    %{ bandwith_check_interval, 5000 }, %% Interval to check right bandwith.
    %{ bandwith_step_decrease, 100 }, %% Step to decrease bandwith, bytes
    { send_buffer_size, 10 }, %% Number of packets inside send buffer. Sender will be stoped if this buffer is big.
    { udp_packet_size, 1024 } %% Max udp packet size, bytes
  ]}
```  

## Using:

All basic functions described in gen_rudp module:
```
  start_listener - start listener for port
  stop_listener - stop listener for port.
  connect -- connect to other host by port
  close -- close connection
  accept - start accept connection
  is_alive -- check is socket alive
  is_connected - check is socket connected to other host.
  controlling_process -- change controlling process for socket
  async_send_binary -- sends message asynchronously
  sync_send_binary -- sends message synchronously. This function will retunr ok only after full packet will be sended (delivery not confirmed)
  send_udp -- sends simple UDP message without confirmation of delivery. The maximu side for message is 1024 bytes.
```  

## Messages generated by rudp:

  * { rudp_connected, Socket, Address, Port} -- made connection from Address and Port
  * { rudp_closed, Socket, Reason } -- connection closed with Reason
  * { rudp_received, Socket, Binary } -- received Binary

