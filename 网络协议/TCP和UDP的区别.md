<table border="2">
    <tr>
        <th></th>
        <th>TCP</th>
        <th>UDP</th>
    </tr>
    <tr>
        <td align="center">连接</td>
        <td align="center">面向有连接的(连接需要三次握手，断开需要四次挥手)</td>
        <td align="center">面向无连接的(通信之前不需要连接)</td>
    </tr>
    <tr>
        <td align="center">传输</td>
        <td align="center">面向字节流，把TCP数据看成一串无结构的数据</td>
        <td align="center">基于报文的，没有拥塞控制，网络拥塞不会导致源主机的发送速率降低</td>
    </tr>
    <tr>
        <td align="center">可靠性</td>
        <td align="center">可靠性传输，保证正确性和数据顺序(三大机制：流量控制、拥塞控制、三握四挥)</td>
        <td align="center">不可靠，不保证正确性与数据的顺序，尽最大努力交付，不保证可靠性交付。</td>
    </tr>
    <tr>
        <td align="center">连接方式</td>
        <td align="center">TCP的连接方式只能是点到点的</td>
        <td align="center">UDP的连接可以是一对一，一对多，多对多的交互通信</td>
    </tr>
    <tr>
        <td align="center">首部(报文)开销</td>
        <td align="center">20字节</td>
        <td align="center">8字节</td>
    </tr>
    <tr>
        <td align="center">系统资源</td>
        <td align="center">对系统资源要求高</td>
        <td align="center">对系统资源要求低</td>
    </tr>
    <tr>
        <td align="center">逻辑信息通道</td>
        <td align="center">全双工的可靠通道</td>
        <td align="center">不可靠通道</td>
    </tr>
</table>

全双工：服务器向客户端发送数据的同时，客户端也可以向服务器端发送数据。