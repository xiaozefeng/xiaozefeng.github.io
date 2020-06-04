# Java Socket

## BIO

### 服务端实现

```java
public class Server {

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(8888);
        System.out.println("server start at 8888 ...");
        while (true) {
            Socket accept = serverSocket.accept();
            new RequestHandler(accept).start();
        }
    }

    static class RequestHandler extends Thread {

        private final Socket socket;

        public RequestHandler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            try (BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {
                final String line = br.readLine();
                System.out.println("received client msg: " + line);
                try (PrintWriter out = new PrintWriter(socket.getOutputStream())) {
                    out.println(line);
                    out.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

### 客户端实现

```java
public class Client {
    public static void main(String[] args) {
        try (Socket client = new Socket(InetAddress.getLocalHost(), 8888)) {
            final PrintWriter printWriter = new PrintWriter(client.getOutputStream());
            printWriter.println("hello");
            printWriter.flush();
            final BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));
            final String line = br.readLine();
            System.out.println("received sever msg: " + line);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



## NIO

### 服务端

```java
public class NIOServer {

    public static void main(String[] args) {
        try (final Selector selector = Selector.open();) {
            final ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.bind(new InetSocketAddress(InetAddress.getLocalHost(), 8888));
            serverSocketChannel.configureBlocking(Boolean.FALSE);

            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            System.out.println("NIO server start at 8888 ....");
            while (true) {
                selector.select();
                final Set<SelectionKey> selectionKeys = selector.selectedKeys();
                final Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()) {
                    final SelectionKey key = iterator.next();
                    try (SocketChannel client = ((ServerSocketChannel) key.channel()).accept()) {
                        final ByteBuffer buffer = ByteBuffer.allocate(1024);
                        client.read(buffer);
                        buffer.flip();

                        final String content = StandardCharsets.UTF_8.newDecoder().decode(buffer.asReadOnlyBuffer()).toString();
                        System.out.println("received client msg: " + content);
                        client.write(StandardCharsets.UTF_8.encode(content));
                    }
                    iterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### NIO客户端

客户端可以使用BIO的客户端代码