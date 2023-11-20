# TCP 

## 如何保证可靠性的

* 数据分块：应用数据被分割成 TCP 认为最适合发送的数据块。

* 序列号和确认应答：TCP 给发送的每一个包进行编号，在传输的过程中，每次接收方收到数据后，都会对传输方进行确认应答，
即发送 ACK 报文，这个 ACK 报文当中带有对应的确认序列号，告诉发送方成功接收了哪些数据以及下一次的数据从哪里开始发。
除此之外，接收方可以根据序列号对数据包进行排序，把有序数据传送给应用层，并丢弃重复的数据。

* 校验和： TCP 将保持它首部和数据部分的检验和。
这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。
如果收到报文段的检验和有差错，TCP 将丢弃这个报文段并且不确认收到此报文段。

* 流量控制： TCP 连接的双方都有一个固定大小的缓冲空间，发送方发送的数据量不能超过接收端缓冲区的大小。
当接收方来不及处理发送方的数据，会提示发送方降低发送的速率，防止产生丢包。
TCP 通过滑动窗口协议来支持流量控制机制。

* 拥塞控制： 当网络某个节点发生拥塞时，减少数据的发送。

* ARQ协议： 也是为了实现可靠传输的，它的基本原理就是每发完一个分组就停止发送，等待对方确认。在收到确认后再发下一个分组。

* 超时重传： 当 TCP 发出一个报文段后，它启动一个定时器，等待目的端确认收到这个报文段。如果超过某个时
间还没有收到确认，将重发这个报文段。

### UDP 为什么是不可靠的？bind 和 connect 对于 UDP 的作用是什么

UDP 只有一个 socket 接收缓冲区，没有 socket 发送缓冲区，即只要有数据就发，不管对方是否可以正确接收。
而在对方的 socket 接收缓冲区满了之后，新来的数据报无法进入到 socket 接受缓冲区，此数据报就会被丢弃，因此 UDP 不能保证数据能够到达目的地，
此外，UDP 也没有流量控制和重传机制，故UDP的数据传输是不可靠的。

和 TCP 建立连接时采用三次握手不同，UDP 中调用 connect 只是把对端的 IP 和 端口号记录下来，
并且 UDP 可多多次调用 connect 来指定一个新的 IP 和端口号，或者断开旧的 IP 和端口号（通过设置 connect 函数的第二个参数）。
和普通的 UDP 相比，调用 connect 的 UDP 会提升效率，并且在高并发服务中会增加系统稳定性。

当 UDP 的发送端调用 bind 函数时，就会将这个套接字指定一个端口，若不调用 bind 函数，系统内核会随机分配一个端口给该套接字。
当手动绑定时，能够避免内核来执行这一操作，从而在一定程度上提高性能。

## TCP 超时重传的原理

发送方在发送一次数据后就开启一个定时器，在一定时间内如果没有得到发送数据包的 ACK 报文，那么就重新发送数据，在达到一定次数还没有成功的话就放弃重传并发送一个复位信号。
其中超时时间的计算是超时的核心，而定时时间的确定往往需要进行适当的权衡，因为当定时时间过长会造成网络利用率不高，定时太短会造成多次重传，
使得网络阻塞。在 TCP 连接过程中，会参考当前的网络状况从而找到一个合适的超时时间。

## TCP 的停止等待协议是什么
停止等待协议是为了实现 TCP 可靠传输而提出的一种相对简单的协议，该协议指的是发送方每发完一组数据后，直到收到接收方的确认信号才继续发送下一组数据。


## TCP中有七种计时器，分别为：

* 建立连接定时器：顾名思义，该定时器是在建立 TCP 连接的时候使用的，在 TCP 三次握手的过程中，发送方发送 SYN 时，会启动一个定时器（默认为 3 秒），若 SYN 包丢失了，那么 3 秒以后会重新发送 SYN 包，直到达到重传次数。

* 重传定时器：该计时器主要用于 TCP 超时重传机制中，当TCP 发送报文段时，就会创建特定报文的重传计时器，并可能出现两种情况：

1. 若在计时器截止之前发送方收到了接收方的 ACK 报文，则撤销该计时器；

2. 若计时器截止时间内并没有收到接收方的 ACK 报文，则发送方重传报文，并将计时器复位。

* 坚持计时器：TCP 通过让接受方指明希望从发送方接收的数据字节数（窗口大小）来进行流量控制，当接收端的接收窗口满时，接收端会告诉发送端此时窗口已满，请停止发送数据。
此时发送端和接收端的窗口大小均为0，直到窗口变为非0时，接收端将发送一个 确认 ACK 告诉发送端可以再次发送数据，但是该报文有可能在传输时丢失。
若该 ACK 报文丢失，则双方可能会一直等待下去，为了避免这种死锁情况的发生，发送方使用一个坚持定时器来周期性地向接收方发送探测报文段，以查看接收方窗口是否变大。

* 延迟应答计时器：延迟应答也被称为捎带 ACK，这个定时器是在延迟应答的时候使用的，为了提高网络传输的效率，当服务器接收到客户端的数据后，不是立即回 ACK 给客户端，
而是等一段时间，这样如果服务端有数据需要发送给客户端的话，就可以把数据和 ACK 一起发送给客户端了。

* 保活定时器：该定时器是在建立 TCP 连接时指定 SO_KEEPLIVE 时才会生效，当发送方和接收方长时间没有进行数据交互时，该定时器可以用于确定对端是否还活着。

* FIN_WAIT_2 定时器：当主动请求关闭的一方发送 FIN 报文给接收端并且收到其对 FIN 的确认 ACK后进入 FIN_WAIT_2状态。
如果这个时候因为网络突然断掉、被动关闭的一端宕机等原因，导致请求方没有收到接收方发来的 FIN，主动关闭的一方会一直等待。该定时器的作用就是为了避免这种情况的发生。
当该定时器超时的时候，请求关闭方将不再等待，直接释放连接。

* TIME_WAIT 定时器： 我们知道在 TCP 四次挥手中，发送方在最后一次挥手之后会进入 TIME_WAIT 状态，不直接进入 CLOSE 状态的主要原因是被动关闭方万一在超时时间内没有收到最后一个 ACK，
则会重发最后的 FIN，2 MSL（报文段最大生存时间）等待时间保证了重发的 FIN 会被主动关闭的一段收到且重新发送最后一个 ACK 。
还有一个原因是在这 2 MSL 的时间段内任何迟到的报文段会被接收方丢弃，从而防止老的 TCP 连接的包在新的 TCP 连接里面出现。

## TCP 最大连接数限制

### Client 最大 TCP 连接数
client 在每次发起 TCP 连接请求时，如果自己并不指定端口的话，系统会随机选择一个本地端口（local port），该端口是独占的，不能和其他 TCP 连接共享。
TCP 端口的数据类型是 unsigned short，因此本地端口个数最大只有 65536，除了端口 0不能使用外，其他端口在空闲时都可以正常使用，这样可用端口最多有 65535 个。

### Server最大 TCP 连接数
server 通常固定在某个本地端口上监听，等待 client 的连接请求。不考虑地址重用（Unix 的 SO_REUSEADDR 选项）的情况下，即使 server 端有多个 IP，本地监听端口也是独占的，
因此 server 端 TCP 连接 4 元组中只有客户端的 IP 地址和端口号是可变的，因此最大 TCP 连接为客户端 IP 数 × 客户端 port 数，对 IPV4，在不考虑 IP 地址分类的情况下，
最大 TCP 连接数约为 2 的 32 次方（IP 数）× 2 的 16 次方（port 数），也就是 server 端单机最大 TCP 连接数约为 2 的 48 次方。

然而上面给出的是只是理论上的单机最大连接数，在实际环境中，受到明文规定（一些 IP 地址和端口具有特殊含义，没有对外开放）、机器资源、操作系统等的限制，特别是 sever 端，其最大并发 TCP 连接数远不能达到理论上限。
对 server 端，通过增加内存、修改最大文件描述符个数等参数，单机最大并发 TCP 连接数超过 10 万 是没问题的。

## TCP 拥塞控制采用的四种算法

### 慢开始
当发送方开始发送数据时，由于一开始不知道网络负荷情况，如果立即将大量的数据字节传输到网络中，那么就有可能引起网络拥塞。
一个较好的方法是在一开始发送少量的数据先探测一下网络状况，即由小到大的增大发送窗口（拥塞窗口 cwnd）。慢开始的慢指的是初始时令 cwnd为 1，即一开始发送一个报文段。
如果收到确认，则 cwnd = 2，之后每收到一个确认报文，就令 cwnd = cwnd* 2。

但是，为了防止拥塞窗口增长过大而引起网络拥塞，另外设置了一个慢开始门限 ssthresh。

1. 当 cwnd < ssthresh 时，使用上述的慢开始算法；

2. 当 cwnd > ssthresh 时，停止使用慢开始，转而使用拥塞避免算法；

3. 当 cwnd == ssthresh 时，两者均可。

### 拥塞避免
拥塞控制是为了让拥塞窗口 cwnd 缓慢地增大，即每经过一个往返时间 RTT （往返时间定义为发送方发送数据到收到确认报文所经历的时间）就把发送方的 cwnd 值加 1，通过让 cwnd 线性增长，防止很快就遇到网络拥塞状态。

当网络拥塞发生时，让新的慢开始门限值变为发生拥塞时候的值的一半,并将拥塞窗口置为 1 ,然后再次重复两种算法（慢开始和拥塞避免）,这时一瞬间会将网络中的数据量大量降低。

### 快重传
快重传算法要求接收方每收到一个失序的报文就立即发送重复确认，而不要等到自己发送数据时才捎带进行确认，假定发送方发送了 Msg 1 ~ Msg 4 这 4 个报文，已知接收方收到了 Msg 1，Msg 3 和 Msg 4 报文，
此时因为接收到收到了失序的数据包，按照快重传的约定，接收方应立即向发送方发送 Msg 1 的重复确认。
于是在接收方收到 Msg 4 报文的时候，向发送方发送的仍然是 Msg 1 的重复确认。
这样，发送方就收到了 3 次 Msg 1 的重复确认，于是立即重传对方未收到的 Msg 报文。
由于发送方尽早重传未被确认的报文段，因此，快重传算法可以提高网络的吞吐量。

### 快恢复
快恢复算法是和快重传算法配合使用的，该算法主要有以下两个要点：

1. 当发送方连续收到三个重复确认，执行乘法减小，慢开始门限 ssthresh 值减半；

2. 由于发送方可能认为网络现在没有拥塞，因此与慢开始不同，把 cwnd 值设置为 ssthresh 减半之后的值，然后执行拥塞避免算法，线性增大 cwnd。


## TCP 状态转换

### 三次握手

### 四次挥手
