### tcp开发

#### 服务端

```go
package main

import (
	"fmt"
	"net"
	"strconv"
)

func main() {
	err := RunServer(":8000")
	if err != nil {
		panic(err)
	}
}

func RunServer(port string) error {
	ln, err := net.Listen("tcp", port)
	if err != nil {
		return err
	}
	for {
		conn, err := ln.Accept()
		if err != nil {
			continue
		}
		go handlerConnection(conn)
	}

}

func handlerConnection(conn net.Conn) {
	defer conn.Close()
	buf := make([]byte, 1024)
	length, err := conn.Read(buf)
	if err != nil {
		panic(err)
	}
	bytes := buf[:length]
	data := string(bytes)
	fmt.Println(data)
	dataLen := strconv.Itoa(len(data))
	conn.Write([]byte(dataLen))
}
```

#### 客户端

```go
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
)

func main() {
	RunClient("127.0.0.1", 8000)
}

func RunClient(host string, port int) error {
	scanner := bufio.NewScanner(os.Stdin)
	for scanner.Scan() {
		line := scanner.Text()
		if line == "bye" {
			fmt.Println("bye")
			break
		}
		addr := fmt.Sprintf("%s:%d", host, port)
		conn, err := net.Dial("tcp", addr)
		defer conn.Close()
		if err != nil {
			return err
		}
		conn.Write([]byte(line))
		buf := make([]byte, 1024)
		len, err := conn.Read(buf)
		if err != nil {
			return err
		}
		bytes := buf[:len]
		fmt.Println(string(bytes))

	}
	return nil
}
```



###  时间计算

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	now := time.Now()
	fmt.Println(now)

	// m = minute , h = hour
	oneHour, _ := time.ParseDuration("1h")
	fmt.Println(oneHour)
	// 一小时后
	d := now.Add(oneHour)
	fmt.Println(d)

	// 一小时前
	oneHour, _ = time.ParseDuration("-1h")
	fmt.Println(oneHour)
	d = now.Add(oneHour)
	fmt.Println(d)

	// 一分钟后
	oneMinute, _ := time.ParseDuration("1m")
	fmt.Println(oneMinute)
	d = now.Add(oneMinute)
	fmt.Println(d)

	// 一分钟前
	oneMinute, _ = time.ParseDuration("-1m")
	fmt.Println(oneMinute)
	d = now.Add(oneMinute)
	fmt.Println(d)

	// 时间计算
	fiftyMinutes := 50 * time.Minute
	fmt.Println(fiftyMinutes)
	count := 30
	fiftyMinutes = time.Duration(count) * time.Minute
	fmt.Println(fiftyMinutes)
}
```

