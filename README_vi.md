* [English](./README.md) | * [Tiếng Việt](./README_vi.md)


# Thư viện xiaozhi-mcp

Thư viện này là thư viện máy khách MCP cho nền tảng ESP32 Xiaozhi. Nó kết nối các thiết bị ESP32 với nền tảng Xiaozhi thông qua plugin MCP. Nó hỗ trợ đăng ký và gọi công cụ, và có thể điều khiển thiết bị thông qua loa Xiaozhi AI.

## Các tính năng
- Hỗ trợ kết nối WebSocket và WebSocket Secure (WSS)

- Cơ chế kết nối lại tự động để đảm bảo kết nối ổn định

- Hỗ trợ giao tiếp giao thức JSON-RPC

- Đăng ký công cụ và hệ thống cuộc gọi

- Cơ chế chức năng gọi lại linh hoạt

- Hỗ trợ cho nền tảng ESP32

## Hướng dẫn cài đặt

### Phương pháp 1: Sử dụng Trình quản lý thư viện Arduino

1. Mở Arduino IDE

2. Nhấp vào "Công cụ" -> "Quản lý thư viện..."

3. Nhập "xiaozhi_mcp" vào hộp tìm kiếm

4. Nhấp vào "Cài đặt"

### Phương pháp 2: Cài đặt thủ công

1. Tải xuống tệp ZIP của thư viện

2. Mở Arduino IDE

3. Nhấp vào "Dự án" -> "Nhập thư viện" -> "Thêm . Thư viện ZIP..."

4. Chọn tệp ZIP đã tải xuống

## Bắt đầu nhanh

Sau đây là một ví dụ hoàn chỉnh cho thấy cách kết nối với máy chủ MCP và đăng ký công cụ:
```cpp
#include <WiFi.h>
#include <WebSocketMCP.h>

// WiFi configuration
const char* ssid = "your-ssid";
const char* password = "your-password";

// MCP server configuration
const char* mcpEndpoint = "ws://your-mcp-server:port/path";

// Create a WebSocketMCP instance
WebSocketMCP mcpClient;

// Connection status callback function
void onConnectionStatus(bool connected) {
if (connected) {
Serial.println("[MCP] connected to the server");
// Register the tool after successful connection
registerMcpTools();
} else {
Serial.println("[MCP] disconnected from the server");
}
}

// Tool callback function - control the LED
ToolResponse ledControl(const String& params) {
// Parse parameters
ToolParams toolParams(params);
if (!toolParams.isValid()) {
return ToolResponse(true, "Invalid parameter");
}

// Get LED state parameter
String state = toolParams.getString("state");
if (state.isEmpty()) {
return ToolResponse(true, "Missing state parameter");
}

// Control LED
if (state == "on") {
digitalWrite(LED_BUILTIN, HIGH);
return ToolResponse(false, "LED is on");
} else if (state == "off") {
digitalWrite(LED_BUILTIN, LOW);
return ToolResponse(false, "LED is off");
} else {
return ToolResponse(true, "Invalid state value, can only be 'on' or 'off'");
}
}

// Register MCP tool
void registerMcpTools() {
// Register LED control tool
mcpClient.registerTool(
"led_control",
"Control ESP32 onboard LED",
"{\"type\":\"object\",\"properties\":{\"state\":{\"type\":\"string\",\"description\":\"LED state: on/off\"}},\"required\":[\"state\"]}",
ledControl
);

// Register a simple tool (simplified version)
mcpClient.registerSimpleTool(
"say_hello",
"Say hello to the person with the specified name",
"name",
"Name of the person to greet",
"string",
[](const String& params) {
ToolParams p(params);
String name = p.getString("name");
return ToolResponse(false, "Hello, " + name + "!");
}
);
}

void setup() {
Serial.begin(115200);

// Initialize the LED pin
pinMode(LED_BUILTIN, OUTPUT);
digitalWrite(LED_BUILTIN, LOW);

// Connect to WiFi
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("WiFi connected successfully");

// Initialize MCP client
if (mcpClient.begin(mcpEndpoint, onConnectionStatus)) {
Serial.println("MCP client initialization successful");
} else {
Serial.println("MCP client initialization failed");
}
}

void loop() {
// Handle MCP client events
mcpClient.loop();
delay(10);
}
```

## Hướng dẫn

### số 1. Kết nối với máy chủ MCP

1. Định cấu hình thông tin mạng WiFi

2. Đặt URL điểm cuối của máy chủ MCP

3. Tạo một phiên bản `WebSocketMCP`

4. Gọi phương thức `begin()` để khởi tạo và kết nối với máy chủ.

5. Gọi `mcpClient.loop()` trong hàm `loop()` để xử lý các sự kiện.
### số 2. Đăng ký một công cụ

Công cụ là một giao diện chức năng do thiết bị cung cấp cho máy chủ MCP. Nó có thể được đăng ký theo hai cách:

#### Phương pháp 1: Hoàn thành đăng ký (với các định nghĩa tham số chi tiết)
```cpp
mcpClient.registerTool(
"tool_name",
"Tool Description",
"{\"type\":\"object\",\"properties\":{\"param1\":{\"type\":\"string\"}},\"required\":[\"param1\"]}",
toolCallback
);
```

#### Phương pháp 2: Đăng ký đơn giản (đối với các công cụ một tham số)```cpp
mcpClient.registerSimpleTool(
"tool_name",
"Tool Description",
"param_name",
"Parameter Description",
"param_type",

toolCallback
);
```

### số 3. Chức năng gọi lại công cụ
Hàm gọi lại công cụ nhận các tham số và trả về phản hồi:
```cpp
ToolResponse toolCallback(const String& params) {
// Parse parameters
ToolParams toolParams(params);
if (!toolParams.isValid()) {
return ToolResponse(true, "Invalid parameters");
}

// Process business logic
// ...

// Return result
return ToolResponse(false, "Operation successful");
}
```

### số 4. Tương tác với Diễn giả Xiaozhi AI
1. Đảm bảo thiết bị được kết nối thành công với máy chủ MCP.

2. Đánh thức Loa AI Xiaozhi và nói một mệnh lệnh, ví dụ: "Xiaozhi, bật đèn LED trên ESP32 của tôi."

3. Loa gửi lệnh đến máy chủ MCP.

4. Máy chủ gọi công cụ tương ứng được đăng ký trên thiết bị.

5. Thiết bị thực thi công cụ và trả về kết quả.

6. Người nói thông báo kết quả thực hiện.

### số 5. Mẹo gỡ lỗi

1. Sử dụng `Serial.println()` để xuất thông tin gỡ lỗi

2. Kiểm tra xem kết nối WiFi có hoạt động bình thường không

3. Xác nhận rằng địa chỉ và cổng máy chủ MCP là chính xác

4. Kiểm tra đầu ra cổng nối tiếp để tìm thông báo lỗi

5. Đảm bảo rằng mã đăng ký công cụ được gọi sau khi kết nối thành công
## Tham chiếu API

### Lớp WebSocketMCP

#### Nhà xây dựng
```cpp
WebSocketMCP();
```

#### Phương pháp khởi tạo
```cpp
bool begin(const char *mcpEndpoint, ConnectionCallback connCb = nullptr);
```
- `mcpEndpoint`: WebSocket server address (ws://host:port/path)
- `connCb`: Connection status change callback function
- Return value: Initialization success

#### Gửi tin nhắn
```cpp
bool sendMessage(const String &message);
```
- `message`: The JSON string to send
- Return value: Whether the send was successful

#### Đăng ký công cụ
```cpp
bool registerTool(const String &name, const String &description, const String &inputSchema, ToolCallback callback);
bool registerSimpleTool(const String &name, const String &description, const String &paramName, const String &paramDesc, const String &paramType, ToolCallback callback);
```
- `name`: Tool name
- `description`: Tool description
- `inputSchema`: Input parameter definition in JSON format
- `callback`: Tool callback function
- Return value: Whether the registration was successful

#### Quản lý công cụ
```cpp
bool unregisterTool(const String &name);
void clearTools();
size_t getToolCount();
```

#### Trạng thái kết nối
```cpp
bool isConnected();
void disconnect();
```

### Lớp phản hồi công cụ
Used to create a tool call response:
```cpp
// Create a text response
ToolResponse(bool isError, const String& message);

// Create a JSON response
ToolResponse(const String& json, bool isError = false);

// Create a response from a JSON object
static ToolResponse fromJson(const JsonObject& json, bool error = false);
```

### Lớp ToolParams

Used to parse tool parameters:
```cpp
ToolParams(const String& json);
bool isValid() const;
String getString(const String& key) const;
int getInt(const String& key, int defaultValue = 0) const;
bool getBool(const String& key, bool defaultValue = false) const;
float getFloat(const String& key, float defaultValue = 0.0f) const;
```

## Các ví dụ
- **BasicExample**: Ví dụ đăng ký công cụ và kết nối cơ bản
- **SmartSwitchExample**: Ví dụ điều khiển công tắc thông minh

## Các dự án liên quan

Nếu bạn cần một giải pháp nhà thông minh hoàn chỉnh hơn, chúng tôi đề xuất dự án ha-esp32.

- Triển khai HomeAssistant trên ESP32, tích hợp với các nền tảng như Xiaomi, Xiaodu, Tuya và Tmall Genie.

- Cung cấp giao diện MCP, hỗ trợ các cuộc gọi mô hình quy mô lớn và cho phép điều khiển thống nhất các thiết bị gia đình.

- Địa chỉ dự án: https://gitee.com/panzuji/ha-esp32

## Lịch sử phiên bản

- v1.0.0: Phiên bản ban đầu, hỗ trợ các kết nối WebSocket cơ bản và đăng ký công cụ.

## Giấy phép

Thư viện xiaozhi-mcp được cấp phép theo Giấy phép Công cộng GNU v3.0 (GPLv3).

GPLv3 là giấy phép phần mềm mã nguồn mở copyleft cho phép bạn tự do sử dụng, sao chép, sửa đổi, hợp nhất, xuất bản và phân phối phần mềm, tuân theo các điều kiện sau:

1. Bất kỳ tác phẩm sửa đổi nào cũng phải được phát hành theo GPLv3.

2. Bản gốc bản quyền và thông báo giấy phép phải được giữ lại.

3. Nếu bạn phân phối phần mềm ở dạng nhị phân, bạn cũng phải cung cấp mã nguồn tương ứng.

PHẦN MỀM NÀY ĐƯỢC CUNG CẤP "NGUYÊN TRẠNG", KHÔNG CÓ BẢO ĐẢM DƯỚI BẤT KỲ HÌNH THỨC NÀO, RÕ RÀNG HAY NGỤ Ý, BAO GỒM NHƯNG KHÔNG GIỚI HẠN Ở CÁC BẢO ĐẢM VỀ KHẢ NĂNG BÁN HÀNG, SỰ PHÙ HỢP CHO MỘT MỤC ĐÍCH CỤ THỂ VÀ KHÔNG VI PHẠM. TRONG BẤT KỲ TRƯỜNG HỢP NÀO, TÁC GIẢ HOẶC CHỦ SỞ HỮU BẢN QUYỀN SẼ KHÔNG CHỊU TRÁCH NHIỆM PHÁP LÝ ĐỐI VỚI BẤT KỲ KHIẾU NẠI, THIỆT HẠI HOẶC TRÁCH NHIỆM PHÁP LÝ NÀO KHÁC, CHO DÙ TRONG MỘT HÀNH ĐỘNG CỦA HỢP ĐỒNG, SAI LẦM HOẶC CÁCH KHÁC, PHÁT SINH TỪ, PHÁT SINH TỪ HOẶC THEO BẤT KỲ CÁCH NÀO LIÊN QUAN ĐẾN PHẦN MỀM HOẶC VIỆC SỬ DỤNG HOẶC CÁC GIAO DỊCH KHÁC TRONG PHẦN MỀM.

Để xem toàn văn GPLv3, hãy truy cập https://www.gnu.org/licenses/gpl-3.0.html
