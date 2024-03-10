# Create Bridge with two name space and ping google IP
First we have to install these tools in our system

ok start.....

```sh
sudo apt update
```

```sh
sudo apt install iproute2
```

```sh
sudo apt install net-tools
```

```sh
sudo apt install iputils-ping
```


```sh
sudo apt install iptables
```


Now we are creating two name space

```sh
sudo ip netns add red
```

```sh
sudo ip netns add green
```

We can check our namespace list

```sh
sudo ip netns list
```

Now we create bridge

```sh
sudo ip link add my-bridge type bridge
```

Now configure bridge
```sh
sudo ip link set my-bridge up
```

```sh
sudo ip addr add 192.168.0.1/16 dev my-bridge
```


this step we are creating two virtual eth cable

```sh
sudo ip link add veth-red type veth peer name veth-red-br

```

```sh
sudo ip link add veth-green type veth peer name veth-green-br
```

We have created two virtual cable. Both cable has two side. Now we will be trying to connect one side with namespace
then another side we will try to connect with bridge like this.

now set
```sh
sudo ip link set veth-red netns red
```

```sh
sudo ip link set veth-red-br master my-bridge
```

```sh
sudo ip link set veth-green netns green
```

```sh
sudo ip link set veth-green-br master my-bridge
```

now we UP both side of cable

```sh
sudo ip netns exec red ip link set veth-red up
```

```sh
sudo ip netns exec green ip link set veth-green up
```


```sh
sudo ip link set veth-red-br up
```

```sh
sudo ip link set veth-green-br up
```

we are setting a IP address each of namespace

```sh
sudo ip netns exec red ip addr add 192.168.0.2/16 dev veth-red
```

```sh
sudo ip netns exec green ip addr add 192.168.0.3/16 dev veth-green
```

we are checking veth is down or up

```sh
sudo ip addr
```
we hope everything is UP

now we can ping namespace to another name space like this 

```sh
sudo ip netns exec red ping 192.168.3 -c 3
```

```sh
sudo ip netns exec red ping 192.168.1 -c 3
```

```sh
sudo ip netns exec green ping 192.168.2 -c 3
```


```sh
sudo ip netns exec green ping 192.168.1 -c 3
```

now are are trying to ping our root IP address

```sh
sudo ip netns exec red ping 10.1.42.245 -c 4
```

if stuck or network unreachable, now have to check our IP table

```sh
sudo ip netns exec red route
```

```sh
sudo ip netns exec green route
```

we have to set default network gateway

```sh
sudo ip netns exec red bash
```

add default route

```sh
sudo ip route add default via 192.168.0.1
```


```sh
route
```


exit commands
```sh
exit
```

```sh
sudo ip netns exec green bash
```


add default route
```sh
sudo ip route add default via 192.168.0.1
```
exit command for exit from green namespace
```sh
exit
```


again are are trying to ping our root IP address

```sh
sudo ip netns exec red ping 10.1.42.245 -c 4
```


To add SNAT Rule at Host side

```sh
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/16  -j MASQUERADE
```

or 

```sh
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth1 -j MASQUERADE
```
Explaining this code

```sh
-t nat: Specifies the "nat" table in iptables.
-A POSTROUTING: Appends the rule to the POSTROUTING chain.
-s 192.168.0.0/24: Specifies the source IP addresses (your local network).
-o eth1: Specifies the outgoing interface (connected to the internet).
-j MASQUERADE: Specifies the action to take, which is MASQUERADE (NAT).
```


```sh
sudo iptables -t nat -L -n -v
```
Putting it all together, the command sudo iptables -t nat -L -n -v is asking iptables to list all the NAT rules in the "nat" table, displaying numerical addresses, and providing verbose output with detailed information about each rule, such as packet and byte counts. This can be helpful for understanding the current configuration and activity of the NAT rules on your system.



```sh
sudo iptables --append FORWARD --in-interface br0 --jump ACCEPT
sudo iptables --append FORWARD --out-interface br0 --jump ACCEPT
```

These rules enabled traffic to travel across the br0 virtual bridge.These are useful to allow all traffic to pass through the br0 interface without any restrictions. However, keep in mind that using such rules without any filtering can expose your system to potential security risks. But for now we re good to ping!



